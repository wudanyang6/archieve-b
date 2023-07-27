---
title: "gdb 基本使用"
date: 2022-03-27T22:28:47+08:00
draft: false
tags: ["gdb", "c语言"]
---


# 1 什么是 gdb
gdb 全称：`"GNU symbolic debugger"`

是一个 gnu 项目孵化出来的调试工具

最常用来调试 c、c++ 程序

# 2 基本使用
## 2.1 生成调试信息
在使用 gdb 之前，可执行程序需要生成调试信息

```shell
g++ test.c -g -o test
```

-g 会在可执行文件中生成一个用于调试的符号表

## 2.2 启动 gdb

直接后面参数是可执行文件就可以启动调试
```shell
gdb test
```

如果要debug一个正在运行的程序，可以使用 -p 参数指定进程id

```
gdb -p $PID
```

效果如下图所示
![](Pasted%20image%2020220327233801.png)

## 2.3 调试程序

几个基本指令：
- file：从文件导入调试信息符号表
- start：从头开始并在首行进入断点执行
- r(un)：执行
- n(ext)：单步跳过
- s(tep)：单步进入
- fin(ish)：单步跳出
- c(ontinue)：继续，执行到下一个断点
- b(reak)：行号，在第几行打断点；function_name，在函数开始断点
- p(rint)：打印变量值
- ptype：打印变量类型
- quit：退出当前调试
- display：跟踪变量（相当于其他工具的监视 watch）
- bt：栈信息
- frame：激活栈，默认显示当前栈帧


一般的工作流是：


```shell
gdb 【带有调试信息的可执行程序】
```


【列出当前执行位置后面的代码】
```
(gdb) l
795	    /* NGX_TIMER_INFINITE == INFTIM */
796
797	    ngx_log_debug1(NGX_LOG_DEBUG_EVENT, cycle->log, 0,
798	                   "epoll timer: %M", timer);
799
800	    events = epoll_wait(ep, event_list, (int) nevents, timer);
801
802	    err = (events == -1) ? ngx_errno : 0;
803
804	    if (flags & NGX_UPDATE_TIME || ngx_event_timer_alarm) {
```

【在第 800 行设置断点】

```
(gdb) b 800  

(gdb) c
Continuing.
【这里程序会运行到800行断点处】
Breakpoint 1, ngx_epoll_process_events (cycle=0xaaaab5a70700, timer=60000, flags=1) at src/event/modules/ngx_epoll_module.c:800
800	    events = epoll_wait(ep, event_list, (int) nevents, timer);
```


【看一下刚才设置的断点信息】

```
(gdb) info b
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000aaaab0f35414 in ngx_epoll_process_events at src/event/modules/ngx_epoll_module.c:800
	breakpoint already hit 2 times
```


【看下当前在什么函数中】

```
(gdb) bt
#0  ngx_epoll_process_events (cycle=0xaaaab5a70700, timer=60000, flags=1) at src/event/modules/ngx_epoll_module.c:800
#1  0x0000aaaab0f29c58 in ngx_process_events_and_timers (cycle=cycle@entry=0xaaaab5a70700) at src/event/ngx_event.c:248
#2  0x0000aaaab0f3268c in ngx_worker_process_cycle (cycle=0xaaaab5a70700, data=<optimized out>) at src/os/unix/ngx_process_cycle.c:721
#3  0x0000aaaab0f30a80 in ngx_spawn_process (cycle=cycle@entry=0xaaaab5a70700, proc=0xaaaab0f32568 <ngx_worker_process_cycle>, data=0x0,
    name=0xaaaab0f8d7d8 "worker process", respawn=respawn@entry=0) at src/os/unix/ngx_process.c:199
#4  0x0000aaaab0f330a0 in ngx_reap_children (cycle=0xaaaab5a70700) at src/os/unix/ngx_process_cycle.c:598
#5  ngx_master_process_cycle (cycle=0xaaaab5a70700) at src/os/unix/ngx_process_cycle.c:174
#6  0x0000aaaab0f08574 in main (argc=<optimized out>, argv=<optimized out>) at src/core/nginx.c:383
```


【监视变量：events 与 event_list】

```
(gdb) display events
1: events = <optimized out>
(gdb) display event_list
2: event_list = (struct epoll_event *) 0xaaaab5a7d6f0
(gdb) n
802	    err = (events == -1) ? ngx_errno : 0;
1: events = 2
2: event_list = (struct epoll_event *) 0xaaaab5a7d6f0

```

【取消监视变量】

```
(gdb) undisplay 1
(gdb) n
802	    err = (events == -1) ? ngx_errno : 0;
2: event_list = (struct epoll_event *) 0xaaaab5a7d6f0
(gdb) undisplay 2
(gdb) n
804	    if (flags & NGX_UPDATE_TIME || ngx_event_timer_alarm) {
```

【查看变量值】

```
(gdb) p flags
$2 = 1
(gdb) p ngx_event_timer_alarm
$3 = 0
```

# 参考
- [3.9 Options for Debugging Your Program or GCC](https://gcc.gnu.org/onlinedocs/gcc-4.6.4/gcc/Debugging-Options.html)
- [GDB: The GNU Project Debugger](https://www.sourceware.org/gdb/documentation/)