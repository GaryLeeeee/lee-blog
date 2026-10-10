---
title: 常用的Linux和macOS命令
date: 2026-10-10 16:13:44
tags: [Golang, Linux, macOS, 工具]
categories: [Golang]
summary: "整理常用的Linux和macOS命令，覆盖接口调用、日志排查、进程端口、定时任务、CPU和磁盘问题。"
---

> 本文只记录日常工作中会反复使用的命令，不收录一次性安装、版本查看等低频命令。
>
> - 系统范围：Linux统一以CentOS为例；没有特别说明时，命令同时适用于CentOS和macOS。
> - 参数约定：`<PID>`、`<PORT>`、`<FILE>`和`<SERVICE>`需要替换为真实内容。
> - 快速查阅：内容较多时可以直接查看{% post_link "golang/常用的Linux和macOS命令-表格速查版" "表格速查版" %}。
> - ⭐ 标记表示这一类中使用频率较高、建议优先记住的命令。

## 一、调用接口（curl）

### 1、查看响应头

```bash
curl -I https://example.com/health
```

适合确认HTTP状态码、重定向、缓存和响应类型。

### 2、查看完整请求过程

```bash
curl -v https://example.com/health
```

`-v`会显示DNS连接、请求头和响应头。输出中可能包含Token、Cookie等敏感数据，分享前需要脱敏。

### 3、设置超时时间

```bash
curl --connect-timeout 3 --max-time 10 \
  https://example.com/health
```

- `--connect-timeout 3`：3秒内没有建立连接就结束。
- `--max-time 10`：整个请求最多等待10秒。

接口卡住时，建议带上超时，避免终端一直等待。

### 4、只查看状态码和耗时 ⭐

```bash
curl -sS -o /dev/null \
  -w 'status=%{http_code} connect=%{time_connect}s total=%{time_total}s\n' \
  https://example.com/health
```

适合快速判断问题发生在连接阶段还是业务处理阶段。

### 5、发送JSON请求

```bash
curl -X POST https://example.com/api/orders \
  -H 'Content-Type: application/json' \
  -H 'X-Request-ID: local-debug-001' \
  -d '{"product_id":1001,"count":1}'
```

主动传入`request_id`后，可以继续在服务端日志中搜索同一个值，把客户端请求和业务日志串起来。

## 二、查找文件（ls/find）

### 1、查看目录和权限（ls） ⭐

```bash
# 显示隐藏文件、权限、大小和更新时间
ls -lah

# 按修改时间排序，最新文件在前
ls -lht

# 按文件大小排序
ls -lhS

# 查看指定目录
ls -lah /var/log
```

`ls -l`输出中的权限类似：

```text
-rwxr-x--- 1 app app 182312 Oct 10 15:30 my-service
```

- 第一位`-`表示普通文件，`d`表示目录。
- 后面三组权限分别属于文件所有者、同组用户和其他用户。
- `r`表示读取，`w`表示写入，`x`表示执行。

### 2、按名称查找（find）

```bash
find /var/log -type f -name '*.log'

# 忽略大小写
find /var/log -type f -iname '*order*.log'
```

### 3、按修改时间查找（find）

```bash
# 最近一天内修改过的文件
find /var/log -type f -mtime -1

# 最近10分钟内修改过的文件
find /var/log -type f -mmin -10
```

### 4、按文件大小查找（find）

```bash
# 查找大于1GB的文件
find /var/log -type f -size +1G -print

# 查找大于500MB的日志文件
find /var/log -type f -name '*.log' -size +500M -print
```

查找磁盘大文件时先使用`find`确认，不要直接拼接删除操作。

## 三、查看文件（tail/grep/less）

### 1、查看最近日志（tail）

```bash
# 查看最后200行
tail -n 200 <FILE>

# 持续查看新增日志
tail -f <FILE>

# 日志轮转或文件重建后继续跟踪
tail -F <FILE>
```

业务日志通常会轮转，长期跟踪时使用`tail -F`比`tail -f`更稳妥。

### 2、过滤日志（grep）

```bash
# 搜索关键词并显示行号
grep -n 'ERROR' <FILE>

# 忽略大小写
grep -ni 'timeout' <FILE>

# 匹配多个关键词
grep -nE 'ERROR|panic|timeout' <FILE>

# 显示匹配行前后各3行
grep -nC 3 'request_id=abc123' <FILE>

# 排除健康检查日志
grep -v '/health' <FILE>
```

### 3、实时查看并过滤（tail + grep） ⭐

```bash
# 实时查看错误日志
tail -F <FILE> \
  | grep --line-buffered -E 'ERROR|panic|timeout'

# 实时跟踪某个请求
tail -F <FILE> \
  | grep --line-buffered 'request_id=abc123'
```

`--line-buffered`可以让`grep`收到一行就输出一行，减少实时查看时的延迟。

### 4、分页查看（less）

```bash
less <FILE>

# 先过滤，再分页查看
grep -n 'request_id=abc123' <FILE> | less
```

在`less`中常用：

- `/keyword`：向下搜索。
- `?keyword`：向上搜索。
- `n`：下一个结果。
- `Shift+F`：持续跟踪新增内容。
- `Ctrl+C`：退出跟踪模式。
- `q`：退出`less`。

### 5、案例：根据request_id排查接口错误

先调用接口并指定`request_id`：

```bash
curl -X POST https://example.com/api/orders \
  -H 'Content-Type: application/json' \
  -H 'X-Request-ID: local-debug-001' \
  -d '{"product_id":1001,"count":1}'
```

再查看这个请求前后的日志：

```bash
grep -nC 5 'request_id=local-debug-001' app.log
```

需要等待问题再次出现时：

```bash
tail -F app.log \
  | grep --line-buffered 'request_id=local-debug-001'
```

日志可能包含Token、Cookie、手机号和订单信息，复制或提交前要先脱敏。

## 四、查找进程和端口（ps/lsof）

### 1、查找进程（ps） ⭐

```bash
# 查看全部进程
ps aux

# 按名称查找，方括号可以避免匹配grep自身
ps aux | grep '[m]y-service'

# 查看指定进程详情
ps -p <PID> -o pid,ppid,user,%cpu,%mem,etime,command
```

常用字段：

- `PID`：进程ID。
- `PPID`：父进程ID。
- `%CPU`：CPU占用。
- `%MEM`：内存占用比例。
- `ETIME`：进程已经运行了多长时间。
- `COMMAND`：启动命令和参数。

### 2、查看进程打开的文件和连接

```bash
lsof -p <PID>
```

输出可能很多，可以继续通过`grep`过滤：

```bash
lsof -p <PID> | grep -E 'log|TCP|LISTEN'
```

### 3、查看端口占用（lsof） ⭐

下面的命令同时适用于CentOS和macOS：

```bash
# 查看8080端口由谁监听
sudo lsof -nP -iTCP:8080 -sTCP:LISTEN

# 使用占位符查找指定端口
sudo lsof -nP -iTCP:<PORT> -sTCP:LISTEN
```

- `-n`：不解析域名，查询更快。
- `-P`：直接显示端口数字。
- `-iTCP:<PORT>`：只查看指定TCP端口。
- `-sTCP:LISTEN`：只查看监听状态。

### 4、案例：服务启动后端口没有监听

先找进程：

```bash
ps aux | grep '[m]y-service'
```

进程存在时查看它打开的网络连接：

```bash
lsof -p <PID> | grep TCP
```

再检查目标端口：

```bash
sudo lsof -nP -iTCP:8080 -sTCP:LISTEN
```

常见结果：

- 没有进程：程序可能启动失败，需要查看启动日志。
- 有进程但没有端口：程序可能卡在初始化或监听了其他地址。
- 端口被其他进程占用：根据`lsof`返回的PID确认占用者。

## 五、后台运行（nohup）

### 最常用写法 ⭐

让服务在后台运行，并把标准输出和错误输出写入同一个日志文件：

```bash
nohup ./my-service > app.log 2>&1 &
```

- `nohup`：退出当前终端后，程序仍然继续运行。
- `> app.log`：把标准输出写入`app.log`。
- `2>&1`：把错误输出也写入同一个文件。
- 最后的`&`：把任务放到后台执行。

启动后确认进程和日志：

```bash
ps aux | grep '[m]y-service'
tail -F app.log
```

需要区分不同环境时，可以使用不同日志文件：

```bash
nohup ./my-service -env=test \
  > logs/my-service-test.log 2>&1 &
```

`nohup`适合临时运行和测试。线上长期运行的服务仍应交给已有的进程管理或发布系统维护。

## 六、查看定时任务和日志（crontab/tail/grep）

### 1、查看CentOS定时任务（crontab） ⭐

```bash
# 当前用户的任务
crontab -l

# root用户的任务
sudo crontab -l

# 系统级任务
less /etc/crontab
ls -lah /etc/cron.d
ls -lah /etc/cron.hourly /etc/cron.daily /etc/cron.weekly
```

同一台机器可能同时存在普通用户、root和`/etc/cron.d`中的任务，排查时不要只看当前用户。

### 2、查看cron是否执行

CentOS通常把cron调度记录写入`/var/log/cron`：

```bash
# 最近200行
tail -n 200 /var/log/cron

# 持续查看新记录
tail -F /var/log/cron

# 查找指定任务
grep -ni 'job.sh' /var/log/cron

# 查找cron执行记录
grep -ni 'CROND' /var/log/cron

# 分页查看
less /var/log/cron
```

`/var/log/cron`主要记录任务是否被调度，不一定包含脚本自己的输出。需要查看业务执行结果时，应在定时任务中明确重定向日志：

```cron
*/5 * * * * /absolute/path/job.sh >> /absolute/path/job.log 2>&1
```

然后查看任务自己的日志：

```bash
tail -n 200 /absolute/path/job.log
tail -F /absolute/path/job.log
```

### 3、案例：cron配置了但没有执行

依次检查：

```bash
# 1.确认任务配置在哪个用户下
crontab -l
sudo crontab -l

# 2.确认crond进程存在
ps aux | grep '[c]rond'

# 3.查看调度记录
grep -ni 'job.sh' /var/log/cron

# 4.确认脚本有执行权限
ls -l /absolute/path/job.sh

# 5.查看脚本自己的日志
tail -n 200 /absolute/path/job.log
```

cron环境中的`PATH`通常比登录终端少，所以任务里的脚本、程序和日志建议全部使用绝对路径。

## 七、修改权限（chmod）

### 1、增加执行权限 ⭐

```bash
chmod +x job.sh

# 只给文件所有者增加执行权限
chmod u+x job.sh
```

### 2、使用数字权限

数字权限由下面三个值相加：

- `4`：读取。
- `2`：写入。
- `1`：执行。

```bash
# 所有者可读写，同组用户只读，其他用户无权限
chmod 640 app.conf

# 所有者可读写执行，同组用户可读执行，其他用户无权限
chmod 750 my-service

# 所有人只读
chmod 444 app.conf
```

### 3、递归修改要谨慎

```bash
chmod -R 750 /path/to/directory
```

`-R`会修改目录下的所有文件和子目录。文件通常不需要执行权限，直接递归设置可能扩大权限范围，生产环境使用前要先通过`ls -lah`确认目录内容。

## 八、查看CPU和内存（top）

### 1、CentOS查看资源 ⭐

```bash
# 查看整机和进程状态
top

# 只查看指定进程
top -p <PID>
```

进入`top`后常用按键：

- `1`：展开每个CPU核心。
- `P`：按CPU占用排序。
- `M`：按内存占用排序。
- `H`：显示线程。
- `c`：切换完整命令行。
- `q`：退出。

### 2、macOS查看资源

```bash
# 按CPU排序
top -o cpu

# 按内存排序
top -o mem

# 只查看指定进程
top -pid <PID>
```

### 3、重点关注的整机指标

CentOS的`top`顶部通常包含下面几组数据。

#### load average

```text
load average: 2.10, 1.80, 1.20
```

分别表示最近1分钟、5分钟和15分钟的平均负载。负载需要结合CPU核心数判断，8核机器的负载为2和2核机器的负载为2不是同一压力。

#### CPU

```text
%Cpu(s): 70.0 us, 10.0 sy, 0.0 ni, 15.0 id, 5.0 wa, 0.0 hi, 0.0 si, 0.0 st
```

| 指标 | 含义 | 数值较高时重点关注 |
| --- | --- | --- |
| `us` | 用户态CPU | 业务计算、循环、序列化、代码热点 |
| `sy` | 内核态CPU | 网络、文件操作、频繁系统调用 |
| `id` | 空闲CPU | 越低说明CPU越忙 |
| `wa` | 等待IO | 磁盘或存储响应慢 |
| `st` | 虚拟机被宿主机占用的CPU | 宿主机资源竞争 |

#### 内存和Swap

重点看可用内存、Swap使用量，以及进程列表中的`RES`和`%MEM`。`VIRT`包含虚拟地址空间，不能直接当作进程真实占用内存。

### 4、重点关注的进程指标

| 指标 | 含义 |
| --- | --- |
| `PID` | 进程ID |
| `S` | 进程状态，常见`R`运行、`S`休眠、`D`不可中断等待、`Z`僵尸 |
| `%CPU` | 进程CPU占用 |
| `%MEM` | 进程内存占比 |
| `VIRT` | 虚拟内存空间 |
| `RES` | 当前驻留在物理内存中的大小 |
| `TIME+` | 进程累计使用CPU的时间 |
| `COMMAND` | 进程名称或命令 |

多核机器中，单个进程的`%CPU`可能超过100%。例如显示250%，大致表示该进程正在使用2.5个CPU核心，并不代表`top`统计错误。

### 5、常见case

#### case一：us高，单个服务进程CPU高

```bash
top
```

按`P`确认高CPU进程，再只查看该进程：

```bash
top -p <PID>
```

按`H`展开线程。如果`us`长期较高，通常需要继续结合业务日志和项目已有的性能分析方式定位计算热点。

#### case二：wa高，但CPU仍有空闲

`wa`高通常表示任务在等待磁盘或存储。可以继续检查磁盘是否已满、日志是否快速增长：

```bash
df -h
du -h --max-depth=1 /var/log
find /var/log -type f -size +1G -print
```

#### case三：load很高，但id也很高

这种情况不一定是CPU计算不足，可能有大量任务处于`D`状态等待IO。可以在`top`进程列表中关注`S`列，并结合`wa`和磁盘使用情况判断。

## 九、查看磁盘（df/du/find/lsof）

### 1、查看文件系统（df） ⭐

```bash
# 查看所有文件系统
df -h

# 查看指定目录所在的文件系统
df -h /var/log
```

重点关注：

- `Size`：总空间。
- `Used`：已使用空间。
- `Avail`：剩余空间。
- `Use%`：使用率。
- `Mounted on`：挂载目录。

`df`回答的是“哪个磁盘分区满了”。

### 2、查看目录大小（du） ⭐

CentOS：

```bash
# 查看目录总大小
du -sh /var/log

# 查看下一层目录大小
du -h --max-depth=1 /var/log

# 查看当前目录下每个文件和目录的大小
du -sh ./*
```

macOS：

```bash
# 查看目录总大小
du -sh /var/log

# macOS使用-d控制目录深度
du -h -d 1 /var/log
```

`du`回答的是“空间被哪些目录或文件占用了”。

### 3、案例：删除日志后磁盘空间没有释放

先确认文件系统仍然很满：

```bash
df -h /var/log
```

再查看目录实际大小：

```bash
du -sh /var/log
```

如果`df`显示占用很高，但`du`统计明显更小，可能有文件已经被删除，却仍被进程持有：

```bash
sudo lsof +L1
```

找到对应PID后，再通过进程发布或日志切换流程处理。不要为了释放空间直接强制停止不认识的进程。

## 十、常见问题排查

### 接口访问失败

1. 使用`curl`确认状态码、连接时间和总耗时。
2. 使用`tail + grep`按`request_id`查看业务日志。
3. 使用`ps`确认服务进程是否存在。
4. 使用`lsof`确认端口是否监听。
5. 使用`top`确认CPU、内存或IO等待是否异常。

### 定时任务没有生效

1. 使用`crontab -l`确认任务属于哪个用户。
2. 使用`ps`确认`crond`进程存在。
3. 使用`grep`检查`/var/log/cron`是否有调度记录。
4. 使用`ls -l`确认脚本权限。
5. 使用`tail`查看任务自己的输出日志。

### 磁盘空间不足

1. 使用`df -h`确认哪个文件系统已满。
2. 使用`du`逐层确认占用空间的目录。
3. 使用`find`寻找大文件。
4. 使用`lsof +L1`检查已删除但仍被占用的文件。
