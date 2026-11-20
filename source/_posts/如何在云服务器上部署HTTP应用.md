---
title: 用 Caddy 部署 HTTP 应用
date: 2026-11-20 17:41:51
tags: [Http, Linux]
categories: [Http]
summary: "以 Ubuntu 上的工具箱为例，用 systemd 运行应用、用 Caddy 配置公网 IP 的 HTTPS 和整站密码，并记录证书配置与公网连接超时的排查过程。"
---

> 本文用一个自用工具箱举例，演示如何在云服务器上部署 HTTP 应用。这个案例运行在腾讯云轻量应用服务器的 Ubuntu 26.04 上，前端用 Vue，后端用 FastAPI。先通过公网 IPv4 和 HTTPS 访问，浏览器会弹出整站密码验证。
>
> 部署自己的应用时，按项目实际情况替换文中的构建和启动命令、服务名、安装目录与监听端口。

## 给 AI 的部署指令

在项目目录打开能使用终端的 AI 助手，填好前三项，再把整段指令发给它。

```text
请把当前项目部署到我的 Ubuntu 云服务器，并完成验收。
服务器连接 <SSH 别名，或我在云控制台终端操作>
公网入口 <公网 IP 或域名，可附端口>
访问权限 <公开，或整站加密码>

1. 先读项目说明、构建和启动脚本，再检查服务器上已有的服务、端口和 Caddy 配置。按实际环境确定命令。
2. 构建、上传并安装应用。需要常驻的后端用独立的 systemd 服务和本机端口。将新站点合并到 Caddy 配置，校验后重载。配好 HTTPS 和访问权限，域名入口先核对 DNS。
3. 在已授权的环境里直接执行。遇到需要我登录云控制台、放行端口或输入密码的步骤，给我这一环节的具体操作，完成后继续。
4. 依次验证服务状态、本机健康检查和公网访问。出错先看相关日志，修复后复测。最后告诉我访问地址、服务名、实际验收结果，以及启停和查看日志的命令。
```

## 一、Caddy 是什么？

Caddy 是一个开源 Web 服务器，可以直接提供网页、图片等静态文件，也可以做反向代理。反向代理就是由 Caddy 接收浏览器的请求，转交给后端应用，再把应用的响应送回浏览器。

在这次部署里，Caddy 是工具箱的公网入口。它负责申请和续期 HTTPS 证书、验证网站访问密码，再通过本机 HTTP 把请求交给 FastAPI。

```text
浏览器 → Caddy（80 跳转 HTTPS，443 负责 TLS 和访问密码）
           → HTTP 应用（127.0.0.1:8000，由 systemd 管理）
```

云防火墙放通 TCP 80 和 443，供网页访问和证书验证。应用端口 8000 只监听服务器本机。22 用于 SSH 和 SCP，按管理来源放行。部署前用 `sudo ss -lntp '( sport = :80 or sport = :443 or sport = :8000 )'` 看端口是否已被占用。

### Caddy 和 Nginx 有什么区别？

两者都能提供静态文件和做反向代理。

| 配置项 | Caddy | Nginx |
| --- | --- | --- |
| HTTPS | 内置自动 HTTPS，可申请、续期证书并设置 HTTP 跳转 | 常见配置用 `ssl_certificate` 加载证书，安装并配置 ACME 模块也能自动申请和续期 |
| 转发规则 | 站点地址对应域名或端口，`handle` 或 `handle_path` 匹配路径，`reverse_proxy` 指定后端 | `listen` 和 `server_name` 选择站点，`location` 匹配路径，`proxy_pass` 指定后端 |

## 二、把应用跑起来

### 1. 准备并上传部署包

本例先在本机构建前端，部署包放入 `frontend/dist`、后端源码和 `deploy/` 下的部署文件。服务器由 FastAPI 提供前端静态文件，运行时只需后端环境。

在工具箱项目根目录生成部署包。

```bash
npm --prefix frontend ci
npm --prefix frontend run build
tar --exclude='__pycache__' --exclude='config.toml' -czf lee-toolbox-deploy.tar.gz \
  frontend/dist backend/app backend/pyproject.toml backend/requirements.lock.txt \
  deploy/configure.py deploy/lee-toolbox.service
```

把部署包传到服务器的 `/tmp`。这次走腾讯云网页终端的「文件管理」上传，公网 SSH 超时期间仍可继续部署。上传方式见 {% post_link codex/如何在云服务器上使用Codex 云服务器使用笔记 %}。

### 2. 安装后端和 Caddy

以下命令在服务器的管理账号下执行。代码放在 `/opt/lee-toolbox`，任务和下载结果放在 `/var/lib/lee-toolbox`，由专用账号写入。

```bash
sudo apt-get update
sudo apt-get install -y python3-venv ffmpeg curl gnupg
sudo install -d -m 0755 /opt/lee-toolbox
sudo tar --no-same-owner -xzf /tmp/lee-toolbox-deploy.tar.gz -C /opt/lee-toolbox
id lee-toolbox >/dev/null 2>&1 || sudo useradd --system --user-group \
  --home-dir /var/lib/lee-toolbox --shell /usr/sbin/nologin lee-toolbox
sudo install -d -o lee-toolbox -g lee-toolbox -m 0750 /var/lib/lee-toolbox
sudo python3 -m venv /opt/lee-toolbox/backend/.venv
sudo /opt/lee-toolbox/backend/.venv/bin/python -m pip install \
  -c /opt/lee-toolbox/backend/requirements.lock.txt /opt/lee-toolbox/backend
```

按文末 Caddy 官方的 Debian/Ubuntu 安装步骤装稳定版。执行 `caddy version` 确认版本，本例的配置脚本要求 Caddy 2.11 或更新版本。

### 3. 配置 HTTPS 和整站密码

把 `PUBLIC_IP` 设为云控制台显示的公网 IPv4。脚本会交互式设置网站账号和密码，生成 `/etc/lee-toolbox/Caddyfile` 与 `toolbox.env`。Caddyfile 里的密码用 bcrypt 哈希保存。

```bash
PUBLIC_IP='替换为服务器公网IPv4'
sudo python3 /opt/lee-toolbox/deploy/configure.py --site "https://$PUBLIC_IP"
sudo caddy validate --config /etc/lee-toolbox/Caddyfile --adapter caddyfile
```

给其他应用配置时，可以复用下面的 Caddyfile 结构。将 `PUBLIC_IPV4` 替换为实际公网 IP，`BCRYPT_HASH` 替换为 `caddy hash-password` 交互生成的值。

```text
{
    default_sni PUBLIC_IPV4
}

https://PUBLIC_IPV4 {
    tls {
        issuer acme {
            dir https://acme-v02.api.letsencrypt.org/directory
            profile shortlived
        }
    }
    basic_auth {
        toolbox BCRYPT_HASH
    }
    reverse_proxy 127.0.0.1:8000
}
```

浏览器用 IP 建立 TLS 连接时通常省略 SNI，云服务器经过公网 NAT 后看到的本地地址可能是内网 IP。`default_sni` 指定在这种情况下用哪个地址选择证书。

校验通过后安装配置并启用服务。下方复制命令适用于只部署这一站的 Caddy。已有其他站点时，将站点块和 `default_sni` 分别合并到原有站点配置和全局选项，校验完整的 Caddyfile 再重载。

```bash
sudo cp -n /etc/caddy/Caddyfile /etc/caddy/Caddyfile.before-toolbox
sudo install -o root -g caddy -m 0640 /etc/lee-toolbox/Caddyfile /etc/caddy/Caddyfile
sudo install -m 0644 /opt/lee-toolbox/deploy/lee-toolbox.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now lee-toolbox.service caddy
sudo systemctl reload caddy
```

### 4. 证书怎么申请和续期？

**首次申请。** Caddy 启动或重载上面的配置后，会通过 ACME 向 Let’s Encrypt 申请公网 IP 证书。签发机构从公网访问 Caddy，通过 TCP 80 上的 HTTP 验证或 TCP 443 上的 TLS 验证，确认你控制这个地址。本例保持两个端口都可从公网访问，服务器也要能通过 HTTPS 访问 ACME 接口。签发成功后，Caddy 会自动启用证书。

**后续续期。** `shortlived` 的 IP 证书有效期为 160 小时，也就是 6 天 16 小时。Caddy 会在到期前自动续期并切换到新证书。保持 `caddy` 服务运行和开机自启，保留它可读写的证书存储目录。Caddy 官方 systemd 服务默认使用 `/var/lib/caddy/.local/share/caddy`。

在服务器上查看签发或续期日志，并读取 Caddy 当前提供的公网 IP 证书。`PUBLIC_IP` 沿用前面的设置。

```bash
sudo journalctl -u caddy -n 50 --no-pager
openssl s_client -connect 127.0.0.1:443 -noservername \
  -verify_ip "$PUBLIC_IP" -verify_return_error </dev/null |
  openssl x509 -noout -issuer -dates -ext subjectAltName
```

`issuer` 是签发机构，`subjectAltName` 应包含当前公网 IP，`notAfter` 是到期时间。续期成功后，到期时间会往后延。遇到签发或续期失败，按日志核对公网端口、到 ACME 接口的出站连接和证书目录权限。用域名时还要核对 DNS 解析。

## 三、从服务器本机验收到公网

先检查后端健康接口，再在服务器本机走 Caddy 的 HTTPS 入口。

```bash
systemctl is-active lee-toolbox.service caddy
curl -fsS http://127.0.0.1:8000/api/health
curl --noproxy '*' -sSI --connect-timeout 5 --max-time 10 \
  --resolve "$PUBLIC_IP:443:127.0.0.1" "https://$PUBLIC_IP/"
```

健康接口应返回 `"status":"ok"`，需要执行后台任务时还应确认 `worker_running` 为 `true`。最后一条把连接地址指向本机，仍按公网 IP 校验证书。返回 `401 Unauthorized` 且有 `WWW-Authenticate`，说明本机的 TLS 和密码保护都已响应。接着从电脑或手机的外部网络打开 `https://公网IP`，输入账号密码，确认应用首页能打开。

| 现象 | 优先检查 |
| --- | --- |
| 健康接口连不上，或 Caddy 返回 `502` | 检查 `lee-toolbox.service` 日志和 `127.0.0.1:8000` 的监听状态 |
| 公网提示证书错误 | 检查 Caddy 的签发日志、当前访问地址和证书对应的 IP |
| 本机返回 `401`，公网连接超时 | 检查公网 IP、云防火墙、系统防火墙，再换一个网络复测 |

查看应用日志用 `sudo journalctl -u lee-toolbox -n 40 --no-pager`。

## 四、这次实际踩到的两个坑

### 配置生成失败，查 `issuer acme` 的参数

模板起初把 ACME 服务地址写成了 `ca`，Caddy 配置检查失败。`issuer acme` 块里应使用上文的 `dir`。修正模板后重新生成配置，先跑 `caddy validate`，看到校验通过再安装。

### 本机 HTTPS 正常，公网 22 和 443 都超时

当时后端健康检查是 200，本机 HTTPS 是 401，云防火墙已放行，UFW 未启用。电脑和手机移动网络访问都超时。服务器抓到客户端的 `SYN` 和反复重发的 `SYN-ACK`，排查范围缩小到了公网连接握手，具体丢包位置留给云厂商核查。

复测时可按发起访问的客户端公网 IP 抓 30 秒 TCP 包，这样能同时核对第三次握手的 `ACK`。Ubuntu 缺少 `tcpdump` 时可以用 apt 安装。

```bash
CLIENT_IP='替换为客户端公网IPv4'
sudo timeout --signal=INT 30s tcpdump -nn -i any -s 96 -c 40 \
  "host $CLIENT_IP and tcp port 443"
```

抓包运行期间，从该客户端重新访问。提交网络工单时附上故障时间、防火墙规则、**本机返回 401** 和 **公网超时**的对比，以及抓包摘要。这次按腾讯云工单建议更换了公网 IP，新 IP 的 22、80、443 均通过连通性检查，随后验收了 HTTPS 和整站密码保护。

## 五、更换公网 IP 或接入域名

更换 IP 后更新 `PUBLIC_IP`，用 `--replace` 重新生成配置。这会同步更新应用允许的公网访问地址，并重新设置网站密码。配置校验并安装后，分别重启应用和重载 Caddy，让 Caddy 为新 IP 申请证书。SSH、SCP 也改用新 IP。下方的复制命令适用于单站点，同机有多个站点时，将变更合并到共享 Caddyfile 后再校验和重载。

```bash
sudo python3 /opt/lee-toolbox/deploy/configure.py --site "https://$PUBLIC_IP" --replace
sudo caddy validate --config /etc/lee-toolbox/Caddyfile --adapter caddyfile
sudo install -o root -g caddy -m 0640 /etc/lee-toolbox/Caddyfile /etc/caddy/Caddyfile
sudo systemctl restart lee-toolbox
sudo systemctl reload caddy
```

有域名后，把 `tools.example.com` 的 DNS A 记录指向公网 IPv4。解析生效后，改用 `--site https://tools.example.com` 生成配置，再按上面的步骤校验并重载。想同时保留 IP 入口，可以重复指定 `--site`。本次实测的是公网 IP 入口。

## 六、怎么单独启停应用？多个应用怎么访问？

### 按 systemd 服务名启停

部署包装好后，按对应的 systemd 服务名管理进程。工具箱的服务名是 `lee-toolbox.service`。装了多个应用时，可以先列出已安装的单元，再看目标服务的状态。

```bash
systemctl list-unit-files 'lee-*.service' 'lee-*.timer'
systemctl status lee-toolbox.service --no-pager
```

| 要做的事 | 命令 |
| --- | --- |
| 临时停止应用 | `sudo systemctl stop lee-toolbox.service` |
| 手动启动应用 | `sudo systemctl start lee-toolbox.service` |
| 立即停止并取消开机自启 | `sudo systemctl disable --now lee-toolbox.service` |
| 立即启动并设置开机自启 | `sudo systemctl enable --now lee-toolbox.service` |

停单个应用时操作它自己的服务，Caddy 作为共享入口继续运行。保留原代理配置时，暂停的应用在通过密码验证后通常会返回 `502`。若应用由 `.timer` 定时拉起，暂停后续调度时也用 `systemctl stop` 或 `disable --now` 操作对应的定时器。

### 公网入口可以按端口、路径或子域名分流

同机部署时，给每个应用分配独立的服务名、运行目录和本机监听端口。这里用工具箱的 `lee-toolbox.service` 和 `127.0.0.1:8000` 举例，再假设有一个笔记应用，使用 `lee-notes.service` 和 `127.0.0.1:8001`。这两个后端端口都只在本机监听，由同一个 Caddy 对外接收请求。

| 公网分流方式 | 浏览器怎么访问 | Caddy 怎么配置 |
| --- | --- | --- |
| 按端口 | `https://公网IP/` 和 `https://公网IP:8443/` | 默认 `443` 转发给工具箱，另设 `8443` 的 HTTPS 站点转发给第二个应用，同时放通公网 TCP 8443 |
| 按路径 | `https://公网IP/tools/` 和 `https://公网IP/notes/` | 两个应用共用 `443`，按 URL 里的路径选择后端。`handle_path /notes/*` 匹配后会去掉 `/notes` 前缀再转发 |
| 按子域名 | `https://tools.example.com/` 和 `https://notes.example.com/` | 两个域名解析到同一个公网 IP，在 Caddy 中分别写站点块，共用 `443` |

同一个公网 IP 的证书可用于不同 HTTPS 端口，ACME 验证仍走 `80` 或 `443`。

这里常说的按 URI 分流，具体匹配的是 `/tools/` 这样的路径。要把当前工具箱放到子路径下，需要一起调整前端路由、静态资源和 `/api/` 请求的前缀。眼下只有 IP，可以保留工具箱的 `443`，给其他应用分配 `8443` 这类额外的 HTTPS 入口。有域名之后，可以用子域名区分，各应用都从根路径访问。

### 怎么查看路由？

**看请求转发给哪个应用。** 在服务器上用 systemd 确认 `--config` 指向的文件。本例使用 `/etc/caddy/Caddyfile`。

```bash
systemctl show caddy -p ExecStart -p ExecReload
sudo less /etc/caddy/Caddyfile
```

在对应站点块里，看 `handle` 或 `handle_path` 匹配的路径，再看 `reverse_proxy` 指向的端口。遇到 `import` 就沿路径查看包含的文件。本例的 `reverse_proxy 127.0.0.1:8000` 转发整个工具箱站点。

如果想确认 Caddy 当前加载的配置，可以在服务器本机查询默认的管理接口。下面的命令用 `jq` 列出监听、匹配的主机和路径，以及代理目标。

```bash
curl --noproxy '*' -fsS http://localhost:2019/config/apps/http/servers |
  jq -r '.. | objects | (.listen[]?, .host[]?, .path[]?, .upstreams[]?.dial?)'
```

用 Nginx 时，`sudo nginx -T` 会校验并展开磁盘上的配置，包括 `include` 文件。按 `listen`、`server_name`、`location`、`proxy_pass` 顺序找对应规则。

**看工具箱有哪些接口。** 登录网站后打开 `https://公网IP/api/docs` 就能看 FastAPI 的接口列表，例如 `GET /api/health`。看源码时，接口注册在 `backend/app/main.py`，`/tasks` 这类前端页面路由在 `frontend/src/app/router.ts`。

## 参考资料

- [Caddy 官方介绍](https://caddyserver.com/)、[Caddy 在 Debian 和 Ubuntu 上的安装](https://caddyserver.com/docs/install#debian-ubuntu-raspbian)
- [Caddy 的 ACME 参数](https://caddyserver.com/docs/caddyfile/directives/tls#issuers)、[TLS 默认 SNI](https://caddyserver.com/docs/caddyfile/options#default_sni)、[Basic Auth](https://caddyserver.com/docs/caddyfile/directives/basic_auth)
- [Caddy 自动 HTTPS 与证书续期](https://caddyserver.com/docs/automatic-https)、[Caddy 服务与证书存储](https://caddyserver.com/docs/running#using-the-service)
- [systemctl 服务管理](https://www.freedesktop.org/software/systemd/man/latest/systemctl.html)、[Caddy 站点地址](https://caddyserver.com/docs/caddyfile/concepts#addresses)、[Caddy 按路径分流](https://caddyserver.com/docs/caddyfile/directives/handle_path)
- [Caddy 管理接口](https://caddyserver.com/docs/api)、[Nginx 请求匹配](https://nginx.org/en/docs/http/request_processing.html)、[Nginx 命令参数](https://nginx.org/en/docs/switches.html)、[Nginx ACME 模块](https://nginx.org/en/docs/http/ngx_http_acme_module.html)
- [Let’s Encrypt IP 证书和有效期](https://letsencrypt.org/2026/01/15/6day-and-ip-general-availability/)
- [腾讯云轻量实例防火墙](https://cloud.tencent.com/document/product/1207/44577)、[更换实例公网 IP](https://cloud.tencent.com/document/product/1207/71733)
