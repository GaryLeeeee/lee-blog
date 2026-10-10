---
title: 常用的Linux和macOS命令：表格速查版
date: 2026-10-10 16:39:40
tags: [Golang, Linux, macOS, 工具]
categories: [Golang]
summary: "以表格形式整理常用的Linux和macOS命令，适合排查接口、日志、进程、端口和机器资源问题时快速查阅。"
---

> 本文只记录日常工作中会反复使用的命令，不收录一次性安装、版本查看等低频命令。
>
> - 阅读方式：本文是{% post_link "golang/常用的Linux和macOS命令" "常用的Linux和macOS命令" %}的表格速查版，适合排查问题时快速查找。
> - 系统范围：Linux统一以CentOS为例；没有特别说明时，命令同时适用于CentOS和macOS。
> - 参数约定：`<PID>`、`<PORT>`、`<FILE>`和`<SERVICE>`需要替换为真实内容。
> - ⭐ 标记表示这一类中使用频率较高、建议优先记住的命令。

## 一、调用接口（curl）

| 场景 | 命令 | 关注点 |
| --- | --- | --- |
| 查看响应头 | `curl -I https://example.com/health` | 状态码、重定向、缓存和响应类型 |
| 查看完整请求过程 | `curl -v https://example.com/health` | 连接过程、请求头和响应头；分享前隐藏Token、Cookie |
| 设置超时 | `curl --connect-timeout 3 --max-time 10 https://example.com/health` | 连接最多3秒，整个请求最多10秒 |
| ⭐ 查看状态码和耗时 | `curl -sS -o /dev/null -w 'status=%{http_code} connect=%{time_connect}s total=%{time_total}s\n' https://example.com/health` | 区分连接慢还是业务处理慢 |
| 发送JSON | `curl -X POST https://example.com/api/orders -H 'Content-Type: application/json' -H 'X-Request-ID: local-debug-001' -d '{"product_id":1001,"count":1}'` | 主动传入`request_id`便于关联日志 |

## 二、查找文件（ls/find）

| 场景 | 命令 | 平台/说明 |
| --- | --- | --- |
| ⭐ 查看目录、权限和大小 | `ls -lah` | CentOS、macOS通用 |
| 按修改时间排序 | `ls -lht` | 最新文件在前 |
| 按文件大小排序 | `ls -lhS` | 大文件在前 |
| 按名称查找日志 | `find /var/log -type f -name '*.log'` | CentOS、macOS通用 |
| 忽略大小写查找 | `find /var/log -type f -iname '*order*.log'` | CentOS、macOS通用 |
| 查找最近一天修改的文件 | `find /var/log -type f -mtime -1` | CentOS、macOS通用 |
| 查找最近10分钟修改的文件 | `find /var/log -type f -mmin -10` | CentOS、macOS通用 |
| 查找大于1GB的文件 | `find /var/log -type f -size +1G -print` | 只查找，不直接删除 |

权限示例`-rwxr-x---`中，三组`rwx`分别属于文件所有者、同组用户和其他用户；`r`表示读取、`w`表示写入、`x`表示执行。

## 三、查看文件（tail/grep/less）

| 场景 | 命令 | 说明 |
| --- | --- | --- |
| 查看最后200行 | `tail -n 200 <FILE>` | 快速查看最近日志 |
| 持续查看 | `tail -f <FILE>` | 文件被轮转后可能需要重新执行 |
| 持续查看并跟随文件重建 | `tail -F <FILE>` | 业务日志长期跟踪优先使用 |
| 搜索并显示行号 | `grep -n 'ERROR' <FILE>` | 定位原文件中的具体位置 |
| 忽略大小写 | `grep -ni 'timeout' <FILE>` | 同时匹配大小写 |
| 匹配多个关键词 | `grep -n -e 'ERROR' -e 'panic' -e 'timeout' <FILE>` | 多个`-e`表示满足任意一个关键词 |
| 显示前后日志 | `grep -nC 3 'request_id=abc123' <FILE>` | 查看错误上下文 |
| 排除健康检查 | `grep -v '/health' <FILE>` | 去掉无关日志 |
| ⭐ 实时过滤错误 | <code>tail -F &lt;FILE&gt; &#124; grep --line-buffered -E 'ERROR&#124;panic&#124;timeout'</code> | 收到一行就输出一行 |
| 实时跟踪请求 | <code>tail -F &lt;FILE&gt; &#124; grep --line-buffered 'request_id=abc123'</code> | 按请求标识跟踪 |
| 分页查看日志 | `less <FILE>` | `/关键词`搜索，`n`下一个，`q`退出 |
| 过滤后分页 | <code>grep -n 'request_id=abc123' &lt;FILE&gt; &#124; less</code> | 结果较多时使用 |

## 四、查找进程和端口（ps/lsof）

| 场景 | 命令 | 关注点 |
| --- | --- | --- |
| 查看全部进程 | `ps aux` | PID、CPU、内存和启动命令 |
| ⭐ 按名称查找进程 | <code>ps aux &#124; grep '[m]y-service'</code> | 方括号避免匹配`grep`自身 |
| 查看进程详情 | `ps -p <PID> -o pid,ppid,user,%cpu,%mem,etime,command` | 父进程、资源占用和运行时长 |
| 查看进程打开的文件和连接 | `lsof -p <PID>` | 日志文件、动态库和网络连接 |
| 过滤进程连接 | <code>lsof -p &lt;PID&gt; &#124; grep -E 'log&#124;TCP&#124;LISTEN'</code> | 快速缩小输出范围 |
| ⭐ 查看端口监听者 | `sudo lsof -nP -iTCP:<PORT> -sTCP:LISTEN` | CentOS、macOS通用 |
| 查看8080端口 | `sudo lsof -nP -iTCP:8080 -sTCP:LISTEN` | 返回进程名和PID |
| 查看已删除但仍被占用的文件 | `sudo lsof +L1` | 处理`df`与`du`结果不一致 |

## 五、后台运行（nohup）

| 场景 | 命令 | 说明 |
| --- | --- | --- |
| ⭐ 后台运行服务 | `nohup ./my-service > app.log 2>&1 &` | 标准输出和错误输出写入同一日志 |
| 指定环境和日志 | `nohup ./my-service -env=test > logs/my-service-test.log 2>&1 &` | 不同环境使用不同日志文件 |
| 确认进程 | <code>ps aux &#124; grep '[m]y-service'</code> | 启动后确认进程存在 |
| 查看启动日志 | `tail -F app.log` | 检查初始化和端口监听是否成功 |

`nohup`适合临时运行和测试，线上长期运行应交给现有进程管理或发布系统。

## 六、查看定时任务和日志（crontab/tail/grep）

| 场景 | 命令 | 说明 |
| --- | --- | --- |
| ⭐ 查看当前用户任务 | `crontab -l` | 只显示当前用户，CentOS示例 |
| 查看root任务 | `sudo crontab -l` | 需要相应权限 |
| 查看系统任务 | `less /etc/crontab` | 系统级crontab |
| 查看cron目录 | `ls -lah /etc/cron.d` | 检查独立任务文件 |
| 查看周期任务目录 | `ls -lah /etc/cron.hourly /etc/cron.daily /etc/cron.weekly` | 检查按周期执行的脚本 |
| 查看最近调度日志 | `tail -n 200 /var/log/cron` | CentOS常见日志位置 |
| 实时查看调度日志 | `tail -F /var/log/cron` | 等待任务再次触发 |
| 查找指定任务 | `grep -ni 'job.sh' /var/log/cron` | 判断任务是否被调度 |
| 查找执行记录 | `grep -ni 'CROND' /var/log/cron` | 查看crond记录 |
| 查看crond进程 | <code>ps aux &#124; grep '[c]rond'</code> | 确认调度进程存在 |
| 给任务单独记录日志 | `*/5 * * * * /absolute/path/job.sh >> /absolute/path/job.log 2>&1` | cron中尽量使用绝对路径 |

`/var/log/cron`主要记录是否调度，脚本输出需要通过`>> ... 2>&1`单独保存。

## 七、修改权限（chmod）

| 场景 | 命令 | 权限结果 |
| --- | --- | --- |
| ⭐ 增加执行权限 | `chmod +x job.sh` | 所有现有权限组增加执行位 |
| 只给所有者增加执行权限 | `chmod u+x job.sh` | 不扩大其他用户权限 |
| 配置文件权限 | `chmod 640 app.conf` | 所有者读写、同组只读、其他无权限 |
| 服务文件权限 | `chmod 750 my-service` | 所有者读写执行、同组读执行 |
| 所有人只读 | `chmod 444 app.conf` | 无人可以直接写入 |
| 递归修改 | `chmod -R 750 /path/to/directory` | 风险较高，执行前先用`ls -lah`确认范围 |

数字权限中，`4`表示读取、`2`表示写入、`1`表示执行。

## 八、查看CPU和内存（top）

| 场景 | 命令/按键 | 平台/说明 |
| --- | --- | --- |
| ⭐ 查看整机和进程 | `top` | CentOS |
| 只看指定进程 | `top -p <PID>` | CentOS |
| 展开CPU核心 | 进入`top`后按`1` | CentOS |
| 按CPU排序 | 进入`top`后按`P` | CentOS |
| 按内存排序 | 进入`top`后按`M` | CentOS |
| 显示线程 | 进入`top`后按`H` | CentOS |
| 显示完整命令 | 进入`top`后按`c` | CentOS |
| 按CPU排序 | `top -o cpu` | macOS |
| 按内存排序 | `top -o mem` | macOS |
| 只看指定进程 | `top -pid <PID>` | macOS |

### CentOS整机指标

| 指标 | 含义 | 数值较高时关注 |
| --- | --- | --- |
| `load average` | 最近1、5、15分钟平均负载 | 结合CPU核心数判断，不能只看绝对值 |
| `us` | 用户态CPU | 业务计算、循环、序列化、代码热点 |
| `sy` | 内核态CPU | 网络、文件操作和频繁系统调用 |
| `id` | 空闲CPU | 越低说明CPU越忙 |
| `wa` | IO等待 | 磁盘或存储响应慢 |
| `st` | 虚拟机被宿主机占用的CPU | 宿主机资源竞争 |
| Swap | 交换空间 | 持续使用可能伴随内存压力和性能下降 |

### 进程指标

| 指标 | 含义 | 关注方式 |
| --- | --- | --- |
| `S` | 进程状态 | `R`运行、`S`休眠、`D`不可中断等待、`Z`僵尸 |
| `%CPU` | CPU占用 | 多核机器中单个进程可以超过100% |
| `%MEM` | 内存占比 | 结合`RES`判断实际驻留内存 |
| `VIRT` | 虚拟地址空间 | 不能直接当作真实内存占用 |
| `RES` | 驻留物理内存 | 更接近进程当前物理内存占用 |
| `TIME+` | 累计CPU时间 | 判断进程是否长期消耗CPU |

### 常见case

| 现象 | 可能方向 | 下一步 |
| --- | --- | --- |
| `us`高且单个进程CPU高 | 业务计算或循环热点 | 按`P`排序，用`top -p <PID>`持续观察，按`H`查看线程 |
| `wa`高 | 磁盘或存储等待 | 使用`df`、`du`和`find`检查空间及大文件 |
| load高但`id`也高 | 任务可能处于IO等待 | 关注`wa`和进程状态中的`D` |
| 单进程显示250% | 大约使用2.5个CPU核心 | 多核环境中的正常统计方式，不是`top`错误 |
| `VIRT`很大但`RES`不大 | 预留了较大虚拟地址空间 | 重点观察`RES`、`%MEM`及其增长趋势 |

## 九、查看磁盘（df/du/find/lsof）

| 场景 | 命令 | 平台/说明 |
| --- | --- | --- |
| ⭐ 查看所有文件系统 | `df -h` | 关注`Avail`、`Use%`和挂载目录 |
| 查看指定目录所在文件系统 | `df -h /var/log` | 判断`/var/log`占用哪个分区 |
| ⭐ 查看目录总大小 | `du -sh /var/log` | CentOS、macOS通用 |
| 查看下一层目录 | `du -h --max-depth=1 /var/log` | CentOS |
| 查看下一层目录 | `du -h -d 1 /var/log` | macOS |
| 查看当前目录各项大小 | `du -sh ./*` | CentOS、macOS通用 |
| 查找大于1GB的文件 | `find /var/log -type f -size +1G -print` | 找到后先确认用途，不直接删除 |
| 检查已删除但仍被占用的文件 | `sudo lsof +L1` | 解释`df`很高但`du`很小的情况 |

`df`用于确认哪个文件系统满了，`du`用于确认空间被哪些目录或文件占用。

## 十、常见问题排查

| 问题 | 排查顺序 |
| --- | --- |
| 接口访问失败 | `curl`看状态码和耗时 → `tail + grep`按`request_id`看日志 → `ps`看进程 → `lsof`看端口 → `top`看资源 |
| 服务启动但端口不通 | `ps`确认进程 → `lsof -p <PID>`看连接 → `lsof -iTCP:<PORT>`看监听者 → `tail`看启动日志 |
| cron没有执行 | `crontab -l`确认用户 → `ps`看`crond` → `grep /var/log/cron`看调度 → `ls -l`看脚本权限 → `tail`看任务日志 |
| CPU占用高 | `top`看`us/sy/wa/id` → 按`P`找进程 → `top -p <PID>`观察 → 按`H`看线程 |
| 磁盘空间不足 | `df -h`找分区 → `du`找目录 → `find`找大文件 → `lsof +L1`找已删除未释放文件 |
