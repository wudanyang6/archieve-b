---
title: "Nginx Master 进程主流程"
date: 2021-04-13T21:07:19+08:00
draft: false
tags: ["nginx", "进程", "源码分析"]
---

# nginx master 进程主流程

之前有说到 [nginx 进程模型-整体架构](https://wudanyang.top/post/nginx/nginx-process/)，下面来看一下 `nginx master` 进程的主要工作

`nginx` 的入口 `main` 函数在 `nginx.c` 文件中

函数原型为：

```c
int ngx_cdecl
main(int argc, char *const *argv)
```

在这个函数中，`master` 做了一系列的初始化操作

最终在下面这个地方进入了主流程中：

```c
//...
    if (ngx_process == NGX_PROCESS_SINGLE) {
        ngx_single_process_cycle(cycle);

    } else {
        ngx_master_process_cycle(cycle);
    }
//...
```

因为我们主要看 `master-worker` 这种进程模型，所以进入 `ngx_master_process_cycle` 

## 设置信号屏蔽字，防止创建子进程过程中被信号中断

```c
// 先清空信号集
    sigemptyset(&set);
    sigaddset(&set, SIGCHLD);
    sigaddset(&set, SIGALRM);
    sigaddset(&set, SIGIO);
    sigaddset(&set, SIGINT);
    sigaddset(&set, ngx_signal_value(NGX_RECONFIGURE_SIGNAL));
    sigaddset(&set, ngx_signal_value(NGX_REOPEN_SIGNAL));
    sigaddset(&set, ngx_signal_value(NGX_NOACCEPT_SIGNAL));
    sigaddset(&set, ngx_signal_value(NGX_TERMINATE_SIGNAL));
    sigaddset(&set, ngx_signal_value(NGX_SHUTDOWN_SIGNAL));
    sigaddset(&set, ngx_signal_value(NGX_CHANGEBIN_SIGNAL));

// 设置信号屏蔽字，将 set 中的信号设置为阻塞状态，防止创建worker 的过程中，被进来的信号打断
    if (sigprocmask(SIG_BLOCK, &set, NULL) == -1) {
        ngx_log_error(NGX_LOG_ALERT, cycle->log, ngx_errno,
                      "sigprocmask() failed");
    }

// 将 set 清空
    sigemptyset(&set);
```

设置信号屏蔽字，防止在创建子进程的过程中被信号处理程序中断

关于信号屏蔽字，引用 `《UNIX 环境高级编程》 `中信号一节的部分内容：

>   进程可以选用`“阻塞信号递送”`。如果为进程产生了一个`阻塞`的信号,而且对该信号的动作是系统默认动作或捕捉该信号, 则为该进程将此信号保持为`未决状态`,直到该进程对此信号解除了阻塞, 或者将对此信号的动作更改为忽略。内核在`递送`一个原来被阻塞的信号给进程时(而不是在产生该信号时),才决定对它的处理方式。于是进程在信号递送给它之前仍可改变对该信号的动作。进程调用 `sigpending` 函数(见10.13节)来判定哪些信号是设置为阻塞并处于未决状态的。

>   每个进程都有一个`信号屏蔽字( signal mask)`,它规定了当前要阻塞递送到该进程的`信号集`。对于每种可能的信号,该屏蔽字中都有一位与之对应。对于某种信号,若其对应位已设置,则它当前是被阻塞的。进程可以调用 `sigprocmask`(在10.12节中说明)来检测和更改其当前信号屏蔽字。
>

当然，在下面创建完子进程之后，会使用 `sigsuspend` 解除信号屏蔽，并使 master 进程进入休眠

关于 `sigsuspend` 函数，简单来说，它是一个 `sigprocmask(SIG_SETMASK, &emptyset, NULL)` 和 `pause()` 函数的结合体，不过相对于使用两个函数完成上述操作，`sigsuspend` 是 `原子操作`。



## 设置 master 进程的 title

```c
static u_char  master_process[] = "master process";

    size = sizeof(master_process);

    for (i = 0; i < ngx_argc; i++) {
        size += ngx_strlen(ngx_argv[i]) + 1;
    }

    title = ngx_pnalloc(cycle->pool, size);
    if (title == NULL) {
        /* fatal */
        exit(2);
    }

    p = ngx_cpymem(title, master_process, sizeof(master_process) - 1);
    for (i = 0; i < ngx_argc; i++) {
        *p++ = ' ';
        p = ngx_cpystrn(p, (u_char *) ngx_argv[i], size);
    }

    ngx_setproctitle(title);
```

将title设置成：

1. 固定字符：`nginx:`
2. 主进程标志: `master process`
3. 命令行启动的命令，如：`/home/ubuntu/mydisk/var/nginx-debug-1/sbin/nginx`

所以进程刚开始是这样显示的：

```sh
$ ps aux | grep nginx | grep -v grep
root      183117  0.0  0.0   4324  2680 ?        ts   17:07   0:00 /home/ubuntu/mydisk/var/nginx-debug-1/sbin/nginx
```

设置了 title 之后，变成了下面这样：

```sh
$ ps aux | grep nginx | grep -v grep
root      183117  0.0  0.0   4324  2680 ?        ts   17:07   0:00 nginx: master process /home/ubuntu/mydisk/var/nginx-debug-1/sbin/nginx
```

## 根据配置启动相应数量的 `worker` 和 `cache` 管理进程

```c
ccf = (ngx_core_conf_t *) ngx_get_conf(cycle->conf_ctx, ngx_core_module);

ngx_start_worker_processes(cycle, ccf->worker_processes,
                           NGX_PROCESS_RESPAWN);
ngx_start_cache_manager_processes(cycle, 0);
```



## 监听信号，并作出响应

在 `ngx_init_signals` 函数中，对原始信号做了变量名映射，具体映射如下：

| 信号        | 对应进程中的标志位变量 | 含义                                                         |
| ----------- | ---------------------- | ------------------------------------------------------------ |
| QUIT        | ngx_quit               | 优雅关闭服务                                                 |
| TERM 或 INT | ngx_terminate          | 强制关闭服务                                                 |
| USR1        | ngx_reopen             | 重新打开服务中的所有文件                                     |
| WINCH       | ngx_noaccept           | 所有子进程不再接受处理新的连接，实际相当于对所有的子进程发送 QUIT 信号 |
| USR2        | ngx_change_binary      | 平滑升级到新版本的 Nginx 程序                                |
| HUP         | ngx_reconfigure        | 重新读取配置文件并使服务对新配置项生效                       |
| CHLD        | ngx_reap               | 有子进程意外结束<br />master 会监控所有子进程，并在子进程意外退出时调用 ngx_reap_children 方法重启子进程 |



master 并不是时刻不停的执行循环检测这些标志位，而是通过 `sigsuspend` 进入休眠，等待有信号唤醒进程时，再循环检测所有信号并处理。



# 参考

[nginx中的ngx_cdecl](https://blog.csdn.net/midion9/article/details/50605337)

[Nginx源码|Nginx信号处理](https://www.jianshu.com/p/33a7502de6e6)

[nginx 信号处理](https://knifefly.cn/2017/09/15/nginx-%E4%BF%A1%E5%8F%B7%E5%A4%84%E7%90%86/)

[LINUX C中sigprocmask()函数用法](https://blog.csdn.net/ShaoLiang_Ge/article/details/57984123)

[linux信号的阻塞和未决](https://blog.csdn.net/linux_ever/article/details/50344837)

《UNIX 环境高级编程》