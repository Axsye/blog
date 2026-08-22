# Linux 学习记录 — Process（进程）

> 本节范围：Linux 进程基础、进程状态、父子进程、`fork/exec/wait`、退出状态、信号、前后台任务、systemd 服务管理、进程权限，以及 File Descriptor 与进程之间的衔接。
>
> 下一节将进入：**File Descriptor / System Call / Linux I/O**，重点包括 `open/read/write/close/ioctl`、返回值、阻塞与非阻塞 I/O 等。

---

# 1. Program 与 Process

## 【必记】

```text
Program = 程序
Process = 进程
PID = Process ID
```

程序是磁盘上的静态可执行文件；进程是程序运行起来后的实例。

```text
program → 静态
process → 跑起来的实例
```

同一个程序可以启动多次：

```text
fpga_capture
├── PID 1201
├── PID 1305
└── PID 1410
```

每一次运行都是独立进程，并拥有自己的 PID。

## 【记忆方法】

```text
PID = Process ID = 进程编号
```

---

# 2. 查看进程：ps / top / htop

## `ps`

```text
ps = process status
```

用于查看某一时刻的进程快照。

常见：

```bash
ps aux
ps -ef
```

---

## `ps aux`

BSD 风格选项，通常不写 `-`。

```bash
ps aux
```

常见列：

```text
USER
PID
%CPU
%MEM
VSZ
RSS
TTY
STAT
START
TIME
COMMAND
```

### 【理解】

可以把它粗略记成：

```text
ps aux → 更方便看用户、CPU、内存、进程状态等资源信息
```

其中：

```text
a → 包括其他用户的相关进程
u → user-oriented format
x → 包括没有控制终端的进程
```

不要求死背每个字母，知道它是常见的“看全部进程 + 资源信息”方式即可。

---

## `ps -ef`

Unix/System V 风格：

```bash
ps -ef
```

其中：

```text
-e = every process
-f = full-format
```

常见列：

```text
UID
PID
PPID
C
STIME
TTY
TIME
CMD
```

### 【记忆方法】

```text
ps aux → 看资源
ps -ef  → 看关系
```

这是方便记忆的工程化简化，不代表两者完全没有重叠。

---

## `top`

```bash
top
```

动态刷新进程、CPU、内存等状态。

```text
ps  → snapshot
top → dynamic monitor
```

`htop` 是更友好的可选工具。

---

# 3. PID 与 PPID

## 【必记】

```text
PID  = Process ID
PPID = Parent Process ID
```

例如：

```text
PID  = 2301
PPID = 2200
```

表示：

```text
2200 是父进程
2301 是它的子进程
```

Linux 进程天然形成父子关系和进程树。

---

# 4. PID 1

Linux 启动进入用户空间后，会有第一个用户空间进程：

```text
PID = 1
```

现代 Ubuntu 中通常是：

```text
systemd
```

可以查看：

```bash
ps -p 1 -f
```

## 【必记】

```text
PID 1
→ 第一个 userspace process
→ 现代 Ubuntu 通常为 systemd
```

---

# 5. 前台与后台进程

正常执行：

```bash
./fpga_capture
```

shell 会等待它结束。

后台运行：

```bash
./fpga_capture &
```

## 【必记】

```text
& = 让 shell 把该命令作为后台 job 运行
```

它**不会自动打开新的终端**。

即使进程在后台：

```text
stdout
stderr
```

仍然可能继续输出到当前终端。

例如：

```bash
./fpga_capture > capture.log 2>&1 &
```

含义：

```text
stdout → capture.log
stderr → capture.log
进程   → shell 后台运行
```

---

# 6. Shell Job Control

常见命令：

```bash
jobs
fg
bg
```

`jobs` 查看当前 shell 的后台/暂停任务。

```bash
fg %1
```

把 job 1 调回前台。

```bash
bg %1
```

让被暂停的 job 1 在后台继续运行。

## 【必记】

```text
job number != PID
```

例如：

```text
[1] 2301
```

这里：

```text
1    → shell job number
2301 → PID
```

---

# 7. Ctrl+C 与 Ctrl+Z

## Ctrl+C

通常给前台进程发送：

```text
SIGINT
```

```text
SIGINT = Interrupt Signal
```

用于请求中断程序。

---

## Ctrl+Z

通常发送：

```text
SIGTSTP
```

让进程暂停。

之后可以：

```bash
bg
fg
```

恢复。

## 【必记】

```text
Ctrl+C → SIGINT  → 通常请求终止/中断
Ctrl+Z → SIGTSTP → 暂停
```

---

# 8. Linux Signal

```text
Signal = 信号
```

它是 Linux 向进程通知某种事件的重要机制。

---

## `kill`

```bash
kill PID
```

名字容易误导：

> `kill` 本质是“发送 signal”，并不等于必然强制杀死。

默认：

```text
SIGTERM
```

---

## SIGTERM

```text
SIGTERM = Terminate Signal
signal number = 15
```

通常用于：

> 请求程序正常退出。

程序可以捕获并处理 SIGTERM，例如：

```text
停止 DMA
关闭设备
flush 数据
关闭 socket
保存状态
然后退出
```

### 【必记】

```text
kill PID
≈ kill -15 PID
≈ 发送 SIGTERM
```

---

## SIGKILL

```bash
kill -9 PID
```

表示：

```text
SIGKILL
signal number = 9
```

### 【必记】

SIGKILL：

```text
不能被捕获
不能被忽略
不能被程序处理
```

由内核强制终止进程。

所以：

```text
SIGTERM → 先礼貌请求退出
SIGKILL → 最后手段
```

---

## SIGSTOP

和 SIGKILL 类似：

```text
SIGSTOP
```

也不能被进程捕获、忽略或处理。

它的作用是强制暂停进程。

---

## 查看所有 signal

```bash
kill -l
```

---

# 9. 进程状态 STAT

`ps aux` 中的：

```text
STAT
```

表示进程状态。

最重要的几个：

```text
R
S
D
T
Z
```

---

## R — Running / Runnable

```text
R = Running / Runnable
```

表示：

- 正在 CPU 上执行；
- 或已经准备好，正在等待 CPU 调度。

---

## S — Sleeping

```text
S = Sleeping
```

进程正在等待某个事件。

这是非常正常、非常常见的状态。

例如：

```text
read UART
等待 socket 数据
等待某个 event
```

过程可能是：

```text
process
  ↓
read()
  ↓
没有数据
  ↓
Sleep
  ↓
IRQ / event
  ↓
wake up
```

---

## D — Uninterruptible Sleep

```text
D = Uninterruptible Sleep
```

通常表示进程正在内核中等待某个不能立即被普通信号打断的事件，经常和 I/O、驱动、硬件等待有关。

可能原因：

```text
块设备 I/O
SD / eMMC
NFS
driver wait
DMA completion 没来
IRQ 没来
硬件异常
总线问题
```

### 【嵌入式关联】

例如：

```text
userspace
   ↓ read()
driver
   ↓
等待 DMA / IRQ
   ↓
FPGA
```

如果 FPGA、DMA 或 IRQ 永远没有完成：

```text
fpga_capture → 长期 STAT=D
```

这时应优先怀疑：

```text
driver
I/O
DMA
IRQ
hardware
storage
```

而不是首先认为：

```text
用户态 while(1) 死循环
```

### 【记忆方法】

```text
D → Device / Disk wait
```

这是帮助记忆，不是官方全称。

---

## T — Stopped

```text
T = Stopped
```

进程被暂停。

例如：

```text
Ctrl+Z
```

之后就可能看到：

```text
STAT = T
```

---

## Z — Zombie

```text
Z = Zombie
```

表示：

> 子进程已经退出，但父进程还没有读取并回收它的退出状态。

Zombie 并不是：

```text
“程序还在疯狂运行”
```

它实际上已经停止执行，只留下少量进程信息等待父进程回收。

### 【必记】

```text
child exit
   ↓
parent 尚未 wait()
   ↓
Zombie
```

---

## 【记忆方法】

```text
R → Run
S → Sleep
D → Device/Disk wait（助记）
T → sTopped
Z → Zombie
```

---

# 10. pidof / pgrep / pkill

## `pidof`

```bash
pidof fpga_capture
```

查找某个程序对应的 PID。

### 【记忆方法】

```text
pidof = PID of ...
```

---

## `pgrep`

```bash
pgrep fpga_capture
```

用于按进程名称等条件搜索 PID。

常见：

```bash
pgrep -a fpga_capture
```

同时显示匹配的进程及命令信息。

---

## `pkill`

```bash
pkill fpga_capture
```

按名字匹配进程并发送 signal。

默认也是：

```text
SIGTERM
```

---

# 11. fork()

Linux/Unix 进程创建的核心系统调用之一：

```c
pid_t pid = fork();
```

## 【必记】

```text
fork()
→ 创建一个子进程
```

成功后：

```text
一个进程
  ↓ fork()
两个进程
```

父子进程都会从 `fork()` 后面的代码继续执行。

---

## fork() 返回值

### 【必记】

```text
pid < 0 → fork 失败
pid = 0 → 当前是子进程
pid > 0 → 当前是父进程，并且值是子进程 PID
```

例如：

```text
Parent PID = 2000

fork()
  ├──────────────┐
  ▼              ▼
Parent          Child
PID 2000        PID 2100

pid = 2100      pid = 0
```

### 【记忆方法】

```text
子进程拿 0
父进程拿孩子的 PID
```

---

# 12. fork() 后的内存

从逻辑上看，子进程获得父进程地址空间的副本：

```text
Parent                Child
code                  code
stack                 stack
heap                  heap
globals               globals
```

父子进程之后通常拥有独立的用户空间地址空间。

现代 Linux 为了提高效率大量使用：

```text
COW = Copy-On-Write
```

即：

```text
写时复制
```

一开始不一定真的复制所有内存页，某一方真正修改时再复制。

### 【了解】

COW 的细节以后在虚拟内存部分继续学习。

---

# 13. exec()

```text
exec
→ 用新的程序替换当前进程映像
```

例如：

```text
child
PID 2100
   ↓ exec("./fpga_capture")
fpga_capture
PID 2100
```

## 【必记】

```text
exec() 不创建新的 PID
```

成功后 PID 通常保持不变。

### 【记忆方法】

```text
fork → 多出一个进程
exec → 当前进程换程序
```

---

## exec 成功后

例如：

```c
execl("./fpga_capture", "./fpga_capture", NULL);

perror("execl");
```

如果 `exec` 成功：

```text
原来的程序已经被替换
```

所以后面的：

```c
perror(...)
```

不会执行。

只有 `exec` 失败才会继续执行原代码。

---

# 14. wait() / waitpid()

父进程可以等待并回收子进程：

```c
wait(NULL);
```

或者：

```c
waitpid(pid, ...);
```

## 【必记】

```text
wait()/waitpid()
→ 等待并回收子进程退出状态
```

它和 Zombie 直接相关：

```text
child exit
   ↓
Zombie
   ↓
parent wait()/waitpid()
   ↓
彻底回收
```

---

# 15. fork / exec / wait 整体模型

### 【必记】

```text
fork → 创建进程
exec → 换成新程序
wait → 回收子进程
```

典型模型：

```text
Parent
  ↓
fork()
  ├───────────────┐
  │               │
Parent          Child
  │               ↓
  │             exec()
  │               ↓
  │          新程序运行
  │               ↓
  └──── wait() ← exit
```

Shell 启动一个程序时，可以概念性理解为使用这一套机制。

---

# 16. Exit Status

程序结束时会返回：

```text
exit status = 退出状态码
```

C 程序：

```c
int main(void)
{
    return 0;
}
```

### 【必记】

Unix/Linux 约定：

```text
0     → success
非 0  → failure / 特定错误状态
```

注意：

> 非 0 的具体数字含义取决于程序本身。

---

## `$?`

Shell 中：

```bash
echo $?
```

查看：

```text
上一条命令的 exit status
```

例如：

```bash
./myapp
echo $?
```

输出：

```text
0
```

通常表示执行成功。

---

# 17. `&&` 与 `||`

## `&&`

```bash
command1 && command2
```

表示：

```text
command1 成功
(exit status = 0)
        ↓
才执行 command2
```

例如：

```bash
gcc main.c -o myapp && ./myapp
```

只有编译成功才运行程序。

---

## `||`

```bash
command1 || command2
```

表示：

```text
command1 失败
(exit status != 0)
        ↓
才执行 command2
```

例如：

```bash
./fpga_capture || echo "capture failed"
```

### 【必记】

```text
&& → success then continue
|| → failure then fallback
```

---

# 18. systemd

现代 Ubuntu 中：

```text
PID 1 通常是 systemd
```

它不仅是一个普通启动脚本，而是现代 Linux 中的重要系统与服务管理器。

常见职责：

```text
启动服务
停止服务
管理依赖
开机自启
失败重启
管理日志
管理服务进程生命周期
```

---

# 19. systemctl

```text
systemctl
≈ system control
```

用于控制 systemd unit。

常见：

```bash
systemctl start fpga-capture
systemctl stop fpga-capture
systemctl restart fpga-capture
systemctl status fpga-capture
systemctl enable fpga-capture
systemctl disable fpga-capture
```

---

## start / stop / restart

```text
start   → 现在启动
stop    → 现在停止
restart → 停止后重新启动
```

---

## enable / disable

```text
enable
→ 配置以后在系统启动时自动启动

disable
→ 取消开机自动启动
```

### 【必记】

```text
start  != enable
stop   != disable
```

### 【记忆方法】

```text
start  → 现在干活
enable → 以后开机也干活
```

---

## `enable --now`

```bash
sudo systemctl enable --now fpga-capture
```

表示：

```text
enable
+
现在立即 start
```

---

# 20. `systemctl status`

查看服务：

```bash
systemctl status fpga-capture
```

通常能看到：

```text
Loaded
Active
Main PID
最近日志
```

### 【必记】

对 systemd service 排障时：

```text
systemctl status <service>
```

通常比单纯 `ps -ef` 更适合作为第一步。

因为：

```text
ps
→ 从进程视角看

systemctl status
→ 从 systemd service 生命周期视角看
```

---

# 21. systemd unit

systemd 管理的对象统称：

```text
unit
```

常见：

```text
.service
.socket
.target
.mount
.timer
```

当前最重要：

```text
.service
```

表示服务单元。

例如：

```text
fpga-capture.service
```

---

# 22. Service 文件基础结构

示例：

```ini
[Unit]
Description=FPGA Capture Service
After=network.target

[Service]
ExecStart=/usr/bin/fpga_capture
WorkingDirectory=/var/lib/fpga-capture
User=fpga
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

三个 section：

```text
[Unit]
→ 描述、依赖和顺序关系

[Service]
→ 服务进程怎么运行

[Install]
→ enable 时如何加入启动关系
```

---

# 23. ExecStart

```ini
ExecStart=/usr/bin/fpga_capture
```

### 【必记】

```text
ExecStart
→ 启动该 service 时实际执行什么
```

---

# 24. WorkingDirectory

```ini
WorkingDirectory=/var/lib/fpga-capture
```

表示服务启动后的：

```text
current working directory
```

这就是 `pwd` 所显示的目录。

例如程序：

```c
open("data.bin", ...);
```

如果：

```ini
WorkingDirectory=/var/lib/fpga-capture
```

那么相对路径：

```text
data.bin
```

会相对于：

```text
/var/lib/fpga-capture
```

解析。

---

# 25. User=

```ini
User=fpga
```

### 【必记】

```text
User=
→ 服务以哪个 Linux 用户身份运行
```

正式服务若没有必要，通常不应该全部以 root 运行。

---

# 26. Least Privilege

```text
least privilege = 最小权限原则
```

即：

> 程序只获得完成工作所需要的最少权限。

例如：

```text
fpga_capture
只需要：
- /dev/my_fpga
- 网络
- /var/lib/fpga-capture

不需要：
- 任意修改 /etc
- 管理所有进程
- 任意访问整个系统
```

那么可以创建专用 service user，而不是让应用长期以 root 身份运行。

---

# 27. Restart=

```ini
Restart=on-failure
```

表示：

> 服务异常失败时 systemd 自动重新启动。

常见：

```text
Restart=no
Restart=always
Restart=on-failure
```

简化理解：

```text
no         → 不自动重启
always     → 退出后总是重启
on-failure → 失败时重启
```

---

# 28. After=

```ini
After=network.target
```

### 【必记】

```text
After=
→ 主要描述启动顺序
```

即：

```text
当前 unit 应排在 network.target 之后启动
```

但：

```text
After=B
```

不简单等于：

```text
“自动把 B 启动起来”
```

systemd 中：

```text
ordering dependency
```

和：

```text
requirement dependency
```

需要区分。

以后再学习：

```text
After
Before
Wants
Requires
```

---

# 29. journalctl

systemd 的日志系统常通过 journal 查看。

```bash
journalctl -u fpga-capture
```

其中：

```text
-u = unit
```

表示只查看某个 unit 的日志。

实时跟踪：

```bash
journalctl -u fpga-capture -f
```

其中：

```text
-f = follow
```

### 【记忆方法】

```text
tail -f
→ 跟踪普通文件

journalctl -f
→ 跟踪 journal
```

### 【嵌入式关联】

systemd 管理的程序：

```c
printf(...)
fprintf(stderr, ...)
```

stdout/stderr 常可被 journal 接收。

所以正式服务未必需要自己手写：

```text
/var/log/myapp.log
```

---

# 30. restart / reload / daemon-reload

这是一个容易混淆的重点。

---

## restart

```bash
systemctl restart myapp
```

表示：

```text
停止旧服务进程
+
重新启动服务
```

PID 通常会变化。

---

## reload

```bash
systemctl reload myapp
```

表示：

> 让正在运行的服务重新加载它自己的配置。

通常不会完整重启整个服务。

例如：

```text
/etc/myapp/config.ini
```

改变后，如果程序支持 reload，可以：

```bash
systemctl reload myapp
```

---

## daemon-reload

```bash
sudo systemctl daemon-reload
```

表示：

> systemd 自己重新读取 unit 文件。

例如修改：

```text
/etc/systemd/system/fpga-capture.service
```

之后应想到：

```bash
sudo systemctl daemon-reload
```

然后若要让正在运行的服务真正按新 `ExecStart` 运行：

```bash
sudo systemctl restart fpga-capture
```

### 【必记】

```text
restart
→ 重启服务进程

reload
→ 服务重新读取自己的配置

daemon-reload
→ systemd 重新读取 unit 配置
```

---

# 31. 修改 .service 的典型流程

例如修改：

```ini
ExecStart=/usr/bin/fpga_capture --mode dma
```

操作：

```bash
sudo systemctl daemon-reload
sudo systemctl restart fpga-capture
```

流程：

```text
修改 .service
     ↓
daemon-reload
     ↓
systemd 读取新 unit
     ↓
restart
     ↓
按新 ExecStart 创建服务进程
```

---

# 32. systemd vs `&`

开发测试：

```bash
./fpga_capture &
```

只是：

```text
shell job control
```

正式服务：

```text
systemd
 ↓
.service
 ↓
fpga_capture
```

systemd 可以管理：

```text
运行用户
工作目录
启动顺序
依赖关系
日志
失败重启
开机启动
停止方式
生命周期
```

### 【必记】

```text
&       → shell 层面的后台任务
systemd → 系统级服务生命周期管理
```

---

# 33. systemctl 辅助命令

查看 unit 内容：

```bash
systemctl cat fpga-capture
```

判断当前是否运行：

```bash
systemctl is-active fpga-capture
```

判断是否 enable：

```bash
systemctl is-enabled fpga-capture
```

现阶段知道即可。

---

# 34. 进程也有 UID/GID

之前学过文件拥有：

```text
owner
group
permissions
```

进程同样带有自己的身份凭据。

简化模型：

```text
Process
UID / GID / groups
       ↓
Kernel permission check
       ↓
File / Device
owner / group / permissions
```

例如：

```ini
User=alice
```

设备：

```text
crw-rw---- root fpga /dev/my_fpga
```

如果：

```text
alice 不是 root
alice 也不是 fpga group 成员
```

那么进程只能匹配：

```text
others = ---
```

因此：

```c
open("/dev/my_fpga", ...)
```

通常失败：

```text
Permission denied
```

对应常见 errno：

```text
EACCES
```

### 【记忆方法】

```text
EACCES
≈ access denied
```

---

# 35. Process Credentials

更准确地说，Linux 进程带有一组：

```text
credentials = 身份凭据
```

其中包括：

```text
UID
GID
supplementary groups
```

因此内核权限检查会综合这些身份信息。

---

# 36. Group 与设备访问

例如设备：

```text
crw-rw---- root fpga /dev/my_fpga
```

如果把用户：

```text
alice
```

加入：

```text
fpga
```

组：

```bash
sudo usermod -aG fpga alice
```

之后 alice 登录形成的新进程就可以通过 group 权限访问设备。

整体链：

```text
/etc/group
    ↓
alice ∈ fpga
    ↓
Process credentials
    ↓
/dev/my_fpga
root:fpga
rw-rw----
    ↓
open() permission check
```

---

# 37. root 与 sudo

```text
root = UID 0
```

root 在传统 Unix/Linux 权限模型中拥有极高权限。

所以：

```bash
sudo ./fpga_capture
```

可能绕过普通权限问题。

但：

### 【必记】

```text
“sudo 后能运行”
```

通常只说明：

```text
问题很可能和权限有关
```

并不意味着正确解决方案就是长期用 root 运行。

更合理的设计通常是：

```text
设备 group 权限
+
专用 service user
+
least privilege
```

---

# 38. udev：与进程权限的衔接

现代 Linux 中，设备节点通常不是手工维护的。

例如：

```text
/dev/ttyUSB0
```

设备插入后自动出现。

用户空间设备管理机制常涉及：

```text
udev
```

简化链：

```text
hardware
   ↓
kernel
   ↓
device event
   ↓
udev
   ↓
/dev/xxx
owner / group / mode / symlink
```

例如可通过规则让设备最终表现为：

```text
crw-rw---- root fpga /dev/my_fpga
```

---

# 39. 【嵌入式关联】从硬件到 userspace process 的完整链

对于 Zynq / PolarFire SoC，可以逐渐形成下面这条工程思维链：

```text
FPGA / SoC Peripheral
        ↓
Device Tree
描述硬件
        ↓
Kernel Driver
控制硬件
        ↓
/dev/my_fpga
用户空间访问入口
        ↓
udev / permissions
决定设备节点权限
        ↓
systemd
启动应用
        ↓
User=
确定进程身份
        ↓
Process credentials
UID/GID/groups
        ↓
open("/dev/my_fpga")
        ↓
Kernel permission check
        ↓
Driver
        ↓
Hardware
```

这条链是后续嵌入式 Linux 驱动、权限、部署、服务管理的核心基础。

---

# 40. File Descriptor 与 Process 的衔接

虽然 File Descriptor 将在下一节正式展开，但它和进程强相关，因此在 Process 章节保留最基本概念。

## 【必记】

```text
FD = File Descriptor
   = 文件描述符
```

例如：

```c
int fd = open("/dev/my_fpga", O_RDWR);
```

成功返回：

```text
fd = 3
```

这个 `3`：

```text
不是 FPGA 地址
不是设备号
不是文件内容
```

而是：

```text
当前进程中某个已打开对象的编号
```

---

# 41. FD 是 per-process 的

### 【必记】

```text
File Descriptor 是每个进程自己的编号空间。
```

例如：

```text
Process A             Process B

fd 3 → /dev/my_fpga   fd 3 → config.ini
```

两个进程中的 `fd 3` 完全可以指向不同对象。

因此应该说：

```text
“进程 A 的 fd 3”
```

而不能笼统说：

```text
“Linux 的 fd 3”
```

---

# 42. 标准 FD

进程启动时通常已有：

```text
stdin  = fd 0
stdout = fd 1
stderr = fd 2
```

### 【必记】

```text
0 → stdin
1 → stdout
2 → stderr
```

这也是为什么第一个额外 `open()` 经常返回：

```text
3
```

但程序不能假设一定如此。

---

# 43. `/proc/<PID>/fd`

Linux 可以通过：

```bash
ls -l /proc/<PID>/fd
```

查看某个进程当前持有的 FD。

例如：

```text
0 -> /dev/null
1 -> /var/log/capture.log
2 -> /var/log/capture.log
3 -> /dev/my_fpga
4 -> socket:[12345]
```

### 【必记】

```text
/proc/<PID>/fd/
→ 查看某个进程打开了哪些对象
```

### 【嵌入式排障】

例如：

```bash
systemctl status fpga-capture
pidof fpga_capture
ls -l /proc/<PID>/fd
```

可以依次判断：

```text
服务活着吗？
进程 PID 是多少？
它到底有没有打开 FPGA 设备？
```

---

# 44. fork() 与 FD

### 【必记】

```text
fork 后，子进程会继承父进程已经打开的文件描述符。
```

例如：

```text
Parent
fd 3 → log.txt

    fork()

Parent             Child
fd 3 → log.txt     fd 3 → log.txt
```

底层打开状态可能有关联，例如文件偏移量可能共享。

细节以后在 Linux I/O 部分继续学习。

---

# 45. Shell 重定向与 fork / exec / FD

之前学过：

```bash
./myapp > out.log 2>&1
```

现在可以理解得更深入：

```text
shell
  ↓
先准备 FD

fd 1 → out.log
fd 2 → 与 fd 1 相同目标

  ↓
fork()
  ↓
child 继承 FD
  ↓
exec("./myapp")
  ↓
myapp
stdout(fd 1) → out.log
stderr(fd 2) → out.log
```

所以：

```text
>
2>
2>&1
```

本质上都与进程的 FD 表有关。

---

# 46. 本节核心总图

```text
Executable Program
        ↓
     process
        ↓
     PID / PPID
        ↓
  ┌─────┴──────────────┐
  │                    │
fork/exec/wait       signal
  │                    │
父子关系             TERM/KILL/INT/TSTP
  │
exit status
  │
Zombie / wait
  │
systemd
  │
.service
  │
User / Restart / logs
  │
Process credentials
UID/GID/groups
  │
权限检查
  │
File Descriptor
  │
下一节：Linux I/O
```

---

# 47. 【必记】Process 章节最终速记

```text
Program  → 磁盘上的程序
Process  → 运行实例

PID      → Process ID
PPID     → Parent Process ID

ps aux   → 常用于看进程资源/状态
ps -ef   → 常用于看 PID/PPID 等关系
top      → 动态监控

R → Running/Runnable
S → Sleeping
D → Uninterruptible Sleep
T → Stopped
Z → Zombie

kill PID     → 默认 SIGTERM
kill -9 PID  → SIGKILL，最后手段
Ctrl+C       → SIGINT
Ctrl+Z       → SIGTSTP

fork → 创建子进程
exec → 当前进程换程序，PID 通常不变
wait → 等待并回收子进程

fork 返回：
<0 → 失败
=0 → 子进程
>0 → 父进程，值为 child PID

exit status:
0     → success
non-0 → failure / specific status

&& → 前一个成功才继续
|| → 前一个失败才继续

systemctl start   → 现在启动
systemctl stop    → 现在停止
systemctl restart → 重启
systemctl enable  → 开机自启
systemctl disable → 取消开机自启

reload        → 服务重新读自己的配置
daemon-reload → systemd 重新读 unit 文件

journalctl -u SERVICE
→ 查看 service 日志

Process credentials:
UID / GID / supplementary groups

least privilege
→ 最小权限原则

FD = File Descriptor
→ 当前进程中已打开对象的编号
```

---

# 48. 下一节

下一节进入：

```text
File Descriptor / System Call / Linux I/O
```

重点包括：

```text
open()
read()
write()
close()
ioctl()

read/write 返回值
短读 / 短写
EOF
阻塞 I/O
非阻塞 I/O
errno
EINTR
EAGAIN
设备文件 I/O
pipe/socket 与 FD
```

这部分会直接连接到：

```text
UART
SPI/I2C userspace
socket
pipe
字符设备驱动
DMA
FPGA device
```

是嵌入式 Linux 开发中非常重要的一章。
