---
title: 如何在云服务器上使用 Codex？从安装到故障排查
date: 2026-11-20 11:06:45
tags: [Codex]
categories: [Codex]
summary: "在 Ubuntu 云服务器上安装 Codex CLI，用设备码登录 ChatGPT，并排查公网 SSH 超时、用户权限和定时任务找不到命令的问题。"
---

> 为了让金价简报在 Mac 关机时也能生成，我把任务搬到一台 Ubuntu 26.04 LTS（x86_64）云服务器上，沿用已有的 ChatGPT Pro 账号。下面先在服务器跑通 Codex，再定位部署时遇到的问题。

## 一、准备服务器和运行账号

普通 Linux 云服务器就能运行 Codex CLI。服务器负责执行命令和脚本，模型推理由 OpenAI 提供。选服务器时先确认访问地点在 OpenAI 支持地区、服务器能连通 OpenAI 和推送服务。助手主动调用飞书 Webhook，部署时可以先用公网 IP 管理服务器，省去域名配置。

从本机运行 `ssh ubuntu@服务器公网IP`，把地址替换成实例的公网 IP。也可以打开云厂商控制台的网页终端。本文用 Ubuntu 的管理账号 `ubuntu` 安装系统工具，再创建一个有独立家目录的普通用户 `lee-assistant`，供 Codex 和定时任务使用。

```bash
sudo apt-get update
sudo apt-get install -y curl ca-certificates git
sudo useradd --create-home --shell /bin/bash lee-assistant
sudo -iu lee-assistant
```

已创建用户时，直接执行最后一条。切换后可用 `whoami` 核对当前用户，用 `exit` 回到管理账号。

## 二、安装 Codex CLI 并用设备码登录

在 `lee-assistant` 的终端里执行。官方独立安装方式无需提前安装 Node.js。安装后可按提示确认可执行文件位置，这次装在 `~/.local/bin/codex`。

```bash
curl -fSL --connect-timeout 15 --max-time 120 https://chatgpt.com/codex/install.sh -o "$HOME/codex-install.sh" && sh "$HOME/codex-install.sh"
"$HOME/.local/bin/codex" --version
```

服务器通常没有浏览器，可以在本机浏览器完成设备码授权。

1. 在 ChatGPT 的「设置 → 安全」开启设备代码登录。托管工作区需要管理员允许。
2. 在服务器上用 `lee-assistant` 执行 `"$HOME/.local/bin/codex" login --device-auth`，保持终端打开。
3. 在本机浏览器打开终端显示的授权地址，登录自己的账号，输入一次性设备码。
4. 回到服务器执行 `"$HOME/.local/bin/codex" login status`。看到 `Logged in using ChatGPT` 就可以试跑。

这里有两次身份验证。SSH 密钥用来登录服务器，ChatGPT 授权用来调用 Codex。用 ChatGPT 登录时，Codex 用量计入对应订阅，服务器租金另外支付。用 API Key 登录时，按 API 侧的额度和计费规则结算。

## 三、在服务器上试跑

先做一次最小测试，确认服务器上的 CLI 能实际调用模型。这条命令可在家目录这类非 Git 目录里运行。

```bash
"$HOME/.local/bin/codex" exec --skip-git-repo-check --sandbox read-only --ephemeral '只回复 OK'
```

要分析代码，进入服务器上已经准备好的 Git 仓库，再运行 `"$HOME/.local/bin/codex" --sandbox read-only`，例如请它说明项目入口和启动命令。脚本和定时任务则用上面的 `codex exec`，让程序拿到结果后继续处理。

这次的定时任务用了 `--ephemeral`，平时从服务器上的简报文件和服务日志查看结果。需要保留 CLI 会话时，可以去掉这个参数，将会话文件留在服务器本地。

## 四、部署时踩到的坑

### `scp` 超时，先检查 SSH 连接

上传文件时，在 Mac 上运行 `scp ./assistant.tar.gz ubuntu@服务器公网IP:/tmp/`。这次遇到 `Operation timed out`，连接还没走到账号验证。先在云控制台核对实例的**公网 IP**，检查云防火墙或安全组是否放行 TCP 22，以及规则来源是否覆盖本机当前的公网 IP。再从网页终端以管理账号检查服务器。

```bash
sudo ss -lntp 'sport = :22'
sudo ufw status
```

当时确认了 SSH 正在监听、UFW 未启用、云防火墙有放行规则，公网超时的具体原因仍未定位。这次是从腾讯云网页终端打开「文件管理」，直接把部署包上传到 `/tmp`，随后在服务器上解压，继续完成了部署。

### 手动能运行，服务却找不到 Codex 或提示未登录

安装、登录和服务运行尽量用同一个 Linux 用户。例子中是 `lee-assistant`，systemd 服务设置 `User=lee-assistant`，调用 `/home/lee-assistant/.local/bin/codex`。绝对路径可以避开交互终端和 systemd 的 `PATH` 差异。

这次还碰到专用账号执行 `sudo` 被拒绝。查看防火墙、安装系统软件和配置 systemd 时，先用 `exit` 回到 `ubuntu`。日常留在管理终端，也可以用 `sudo -iu lee-assistant /home/lee-assistant/.local/bin/codex login status` 临时以运行用户检查。

### CLI 已登录，生成仍然失败

本机通过公网 SSH 登录依赖服务器的入站连接。Codex 调用模型、脚本推送消息都依赖服务器的出站 HTTPS。遇到生成失败，先用运行账号检查登录状态，失效就重新做设备码授权，再重跑第三节的最小测试。登录正常时，按报错检查出站网络、账号验证或剩余额度。

登录和试跑通过后，可以用 {% post_link codex/用Codex做定时推送小助手 金价简报的完整示例 %} 配置北京时间定时触发和飞书推送。

## 参考资料

- [Codex CLI 安装](https://learn.chatgpt.com/docs/codex/cli)
- [Codex 设备码登录](https://learn.chatgpt.com/docs/auth#login-on-headless-devices)
- [Codex 非交互运行](https://learn.chatgpt.com/docs/non-interactive-mode)
- [OpenAI 支持的国家和地区](https://help.openai.com/en/articles/7947663-chatgpt-and-openai-api-supported-countries-and-territories)
- [腾讯云轻量应用服务器防火墙](https://cloud.tencent.com/document/product/1207/44577)、[网页终端文件管理](https://cloud.tencent.com/document/product/1665/84580)
