---
title: ShellLab
date: 2020-10-23 14:53:26
tags: [异常控制流,进程,信号,csapp]
categories: [CSAPPLab]
---
## 前置知识

这篇文章主要围绕CMU的CSAPP中的ShellLab来讲（ShellLab实现一个简易的unix下的shell，光听就感觉很激动吧）

讲什么

- 完成ShellLab所需知识（对应CSAPP第八章，仅作复习使用，如果从未接触过请去看原书）
  - 进程
  - 信号
- ShellLab实验文件
- ShellLab中的注意点和坑点

注：本篇文章所有代码已上传至[github](https://github.com/nc-77/cmu_csapp_lab/tree/master/ShellLab)，lab须在Linux环境下运行

### 进程

进程可以说是计算机科学中最深刻，最成功的概念之一。打开你的任务管理器，是不是有一堆进程正在运行中，想象一下，如果没有进程，你的计算机每时每刻只能运行一个程序，是不是很崩溃的画面。

但是在现代操作系统中，我们正在使用的程序往往给我们这是系统中当前运行唯一的程序一样，处理器似乎就在单独执行我们这个程序一样。然而，这些都是通过进程模拟出来的假象。

进程提供给程序的关键抽象：

- 一个独立的逻辑控制流。它提供一个假象，好像我们的程序独占地使用处理器。
- 一个私有的地址空间。它提供一个假象，好像我们的程序独占地使用内存系统。

#### 逻辑控制流

首先，我们要有一个概念。进程是轮流使用CPU的。

如图，这三个进程的逻辑流是交错运行的。进程A运行了一会儿，接着轮到进程B，然后是进程B开始运行到结束。然后，进程C运行了一会儿，进程A紧接着运行到完成，最后，进程C可以运行到结束。

<img src="https://img.nc-77.top/20201022163214.png" style="zoom:67%;" />

一个逻辑流的执行在时间上与另一个流重叠，称为**并发流**，比如上图中的A与B，A与C，但是B与C并不是并发，因为在C开始运行前B已经结束运行。

#### 私有地址空间

进程为每个程序提供它的私有地址空间。一般而言，该地址空间中的内存是不能被其他进程读写的。

尽管每个私有地址空间存储内容不同，但是他们有相同的通用结构，如下图是x86-64 Linux进程的地址空间的组织结构。

<img src="https://img.nc-77.top/20201022163247.png" style="zoom:67%;" />

#### 创建和终止进程

进程总是处于以下三种状态：

- 运行
- 停止（被挂起，但是依然存在）
- 终止（永远地停止了，不再存在于内存中）

我们可以通过在父进程中调用fork函数来创建一个子进程。子进程将得到与父进程虚拟地址空间相同但是独立的副本，两者最大的区别在于他们拥有不同的PID。

fork函数调用一次，但是却返回两次——在父进程中，fork返回子进程的PID；在子进程中，fork返回0

<img src="https://img.nc-77.top/2020-10-22 16-34-35-1.png" style="zoom: 50%;" />

正如上文中所说，进程之间是并发执行的。因此对于父进程和子进程，我们绝不能对不同进程中指令的交替执行做任何假设。（这点很重要!!!）

当一个进程终止时，它会一直保持在一种已终止的状态中，直到被它的父进程回收。我们将一个终止了但还未被回收的进程称为**僵死进程**。

一个进程可以通过调用wait或waitpid函数来等待它的子进程终止或者停止（具体函数可在man中查看），然后父进程会回收这个子进程，内核将抹去被回收进程的所有痕迹。

### 信号

信号是不同进程中相互通信的方式，它允许进程和内核中断其他进程。

一般而言，每种信号类型都对应于某种系统事件。例如，平时使用shell的时候，当进程在前台运行时，键入Ctrl+c内核将会发送一个SIGINT信号给这个前台进程组的每个进程。（SIGINT默认行为为终止）当一个子进程终止或者停止时，内核也发送一个SIGCHLD信号给父进程。

#### 接收与阻塞信号

当进程收到内核发来的信号时，进程可以忽略，终止或者通过一个信号处理程序的用户层函数来捕获这个信号。

一个发出而没有被接收的信号叫做**待处理信号**。在任何时刻，一种类型至多只会有**一个**待处理信号。

也就是说，如果进程中有一个待处理信号，那么同类型的信号将不再被接受，也不会排队等待，而是直接被丢弃。但是，一个进程可以阻塞某种信号，当某种信号被阻塞时，它仍可以被发送，但是产生的待处理信号不会被接受，直到进程取消对这种信号的阻塞。

下图为阻塞SIGINT信号的例子。（具体相关sigprocmask等信号函数请查看man文档）

<img src="https://img.nc-77.top/2020-10-22 17-10-00-1.png" style="zoom:67%;" />

#### 编写信号处理程序

编写一个安全的信号处理程序很麻烦，因为主程序和这些信号处理程序是并发运行的。如果这些程序都需要访问同样的全局变量，但是由于我们无法预测这些程序指令的先后顺序，那么结果往往是不可预知的。因此在编写信号处理程序时往往有以下规则。

- 处理程序尽可能简单。
- 在处理程序中只调用异步信号安全（可重入，不能被信号处理程序打断）的函数（printf，malloc和exit都不在此列）
- 保存和恢复error。
- 阻塞所有信号，保护对共享全局数据结构的访问。
- 用volatile声明全局变量。
- 用sig_atmoic_t 声明标志。（原子操作，不可被打断）

- 由于同一类型信号不会排队。因此如果存在一个未处理的信号就表明至少有一个信号到达了
- 当有父子进程中某部分有明确的执行顺序时，注意阻塞信号来避免父子进程之间的竞争。

## ShellLab

### 实验文件

[实验文件下载](http://csapp.cs.cmu.edu/3e/labs.html)

在开始做这个lab之前请认真阅读[writeup](http://csapp.cs.cmu.edu/3e/shlab.pdf)和对应的csapp第8章。

实验要求完成tsh.c这个文件实现一个shell，大致框架已经搭好，我们只需完成其中的空白函数。

tshref是一个成品，我们需要让我们的tsh和tshref的输出完全一致（除PID外）。

tracexx.txt是测试文件，推荐trace一个一个测试过来逐步完善我们的tsh。

常用命令如下：

```bash
/* 编译生成tsh */
make clean
make
/* 
 *测试trace01 
 *test为tsh输出 
 *rtest为rtsh输出
 */
make test01
make rtest01
```

### eval

书本p525已经为我们展示了一个大致框架。但是还存在几个问题需要改进

- 后台子进程运行完没回收，会成为僵尸进程

- 需要维护一个jobs（表示正在运行的进程）

Hints:

- 维护jobs通过已给出的addjob和deletjob实现，但我们需要防止父子进程之间的竞争，防止子进程在父进程addjob前结束，触发SIGICHLD信号导致先执行了deletjob。因此需要在父进程fork前阻塞SIGCHLD信号
- 父进程fork每个子进程后需每个子进程设立一个新的进程组，避免之后的SIGINT信号终止所有子进程

```c
void eval(char *cmdline) 
{
    char buf[MAXLINE];
    strcpy(buf,cmdline);
    char *argv[MAXARGS];
    pid_t pid;
    int bg=parseline(buf,argv); /* run bg or fg */

    sigset_t mask_one;
    sigemptyset(&mask_one);
    sigaddset(&mask_one,SIGCHLD);

    if(argv[0]==NULL) 
        return;
    if(!builtin_cmd(argv)){ /*if builtin_cmd execute immediately,if not fork and execute*/
        sigprocmask(SIG_BLOCK,&mask_one,NULL);  /* 阻塞SIGCHLD防止竞争,确保正确addjob */
        
        if((pid=Fork())==0){    /* child */
            sigprocmask(SIG_UNBLOCK,&mask_one,NULL); /* 子进程继承父进程BOLCK,需要取消子进程阻塞 */
            setpgid(0,0); /* 为每个进程设立一个新的进程组 */
            if(execve(argv[0],argv,environ)<0){
                printf("%s: Commond not found\n",argv[0]);
                exit(0);
            }
        }
        /* parent wait for fg job to terminate */
        int state=(bg? BG:FG);
        
        addjob(jobs,pid,state,cmdline);
        
        /* 后台程序在printf后再解除CHLD阻塞防止在printf前被CHLD信号打断 */
        //sigprocmask(SIG_UNBLOCK,&mask_one,NULL);
        if(!bg) {
            sigprocmask(SIG_UNBLOCK,&mask_one,NULL);  /* 解除父进程CHLD阻塞 */
            waitfg(pid);
        }
        else{
            printf("[%d] (%d) %s",pid2jid(pid),pid,cmdline);
            sigprocmask(SIG_UNBLOCK,&mask_one,NULL);  /* 解除父进程CHLD阻塞 */
        }
    }
    
    return;
}
```

### builtin_cmd

处理内置命令的函数

```c
int builtin_cmd(char **argv) 
{
    if(!strcmp(argv[0],"quit"))
        exit(0);
    else if(!strcmp(argv[0],"jobs")){
        listjobs(jobs);
        return 1;
    }
    else if(!strcmp(argv[0],"bg")||!strcmp(argv[0],"fg")){
        do_bgfg(argv);
        return 1;
    }
    return 0;     /* not a builtin command */
}
```

### waitfg

等待前台进程结束

Hints:

- sigsuspend(&mask)函数用mask临时代替当前进程的阻塞集合，挂起该进程等待一个信号
- 用sleep（） 间隔太小会浪费循环 间隔太大又会导致程序太慢
- 用pause（）存在竞争的风险，即CHLD信号在while和pause之间被接受会导致永久的pause

```c
void waitfg(pid_t pid)
{
    sigset_t mask_empty;
    sigemptyset(&mask_empty);
    while(fgpid(jobs)==pid){  /* 等待fg结束向父进程发送SIGCHLD信号 */
        sigsuspend(&mask_empty);
    }
    return;
}
```

### sigchld_handler

Hints：

- waitpid函数第三个参数需设置WNOHANG|WUNTRACED，表示即使子进程STOP也直接返回
- 利用返回的status判断子进程目前状态并作相应处理
- 访问全局变量记得阻塞信号
- 保存之前errno并在返回前恢复

```c
void sigchld_handler(int sig) 
{
    int olderrno=errno;
    pid_t pid;
    sigset_t mask_all,prev;
    sigfillset(&mask_all);
    int status;
    while((pid=waitpid(-1,&status,WNOHANG|WUNTRACED))>0){   /* WNOHANG 所有子进程终止后返回 | WUNTRACED 即使进程被ST也立即返回 */
        if(WIFEXITED(status)){                       /* 进程正常终止 */
            sigprocmask(SIG_BLOCK,&mask_all,&prev);  /* 访问全局变量jobs上阻塞 */
            deletejob(jobs,pid);
            sigprocmask(SIG_SETMASK,&prev,NULL);  /* 解除阻塞 */
        }
        else if(WIFSIGNALED(status)){           /* 进程由于SIGINT信号终止 */
            sigprocmask(SIG_BLOCK,&mask_all,&prev);  
            printf("Job [%d] (%d) terminated by signal %d\n",pid2jid(pid),pid,SIGINT); /* 这里printf非异步函数,其实不应该用在此处 */
            deletejob(jobs,pid);
            sigprocmask(SIG_SETMASK,&prev,NULL); 
        }
        else if(WIFSTOPPED(status)){           /* 进程由于SIGTSTP信号挂起 */
            struct job_t *job=getjobpid(jobs,pid);
            sigprocmask(SIG_BLOCK,&mask_all,&prev);  
            printf("Job [%d] (%d) stoped by signal %d\n",pid2jid(pid),pid,SIGTSTP); /* 这里printf非异步函数,其实不应该用在此处 */
            job->state=ST;
            sigprocmask(SIG_SETMASK,&prev,NULL); 
        }
    }
    errno=olderrno;
    return;
}
```

### sigint_handler

实现ctrl+c 向当前前台进程组发送SIGINT信号

Hints:

- kill（pid_t pid,int _sig）发送sig信号到pid，pid<-1时表示发送到-pid的整个进程组

```c
void sigint_handler(int sig) 
{
    int olderrno=errno;
    pid_t pid=fgpid(jobs);
    if(pid){
        kill(-pid,SIGINT);
    }
    
    errno=olderrno;
    return;
}
```

### sigstp_handler

实现ctrl+z 向当前前台进程组发送SIGTSTP信号

```c
void sigtstp_handler(int sig) 
{
    int olderrno=errno;
    pid_t pid=fgpid(jobs);
    if(pid){
        struct job_t *job=getjobpid(jobs,pid);
        if(job->state==ST)  /* 已经停止了 */
            return;
        else 
            kill(-pid,SIGTSTP);
    }
    errno=olderrno;
    return;
}
```

### do_bgfg

Hints:

- 部分前后台程序可能被挂起，先发送SIGCONT信号
- 后台程序转到前台运行时需等待其运行完

```c
void do_bgfg(char **argv) 
{
    if(argv[1]==NULL){
        printf("%s command requires PID or %%jobid argument\n",argv[0]);
        return;
    }
    int id;
    struct job_t *job;
    /* 按&%d格式读取jid */
    if(sscanf(argv[1],"%%%d",&id)>0){
        job=getjobjid(jobs,id);
        if(job==NULL){
            printf("(%d): No such job\n",id);
            return;
        }
    }
    /* 按%d格式读取pid */
    else if(sscanf(argv[1],"%d",&id)>0){
        job=getjobpid(jobs,id);
        if(job==NULL){
            printf("%s: No such process\n",argv[1]);
            return;
        }
    }
    /* 格式错误 */
    else{
        printf("%s: argument must be a PID or %%jobid\n",argv[0]);
        return;
    }
    kill(-(job->pid),SIGCONT);
    if(!strcmp("bg",argv[0])){
        printf("[%d] (%d) %s",job->jid,job->pid,job->cmdline);
        job->state=BG;
    }
    else{   /*job转到前台运行并等待job运行完 */
        job->state=FG;
        
        waitfg(job->pid);
    }
    return;
}
```



## 总结

目前做了4个lab中这个ShellLab给我的感触是最深的，算上这篇blog ，这部分内容断断续续做了一周左右的时间。当然，其中的有些内容还是有点迷迷糊糊，希望能在之后再能系统地学习一次。但是，不管怎么说，个人认为CSAPP给我带来的帮助是很大的，尤其是配套的lab实验认认真真的做，相信会让你对计算机的认识有翻天覆地的变化。

最后，希望自己能一直保持初心，毕竟可能有只有这段大学时光能做到不那么功利，去读自己想读的书，做自己想做的事。

共勉。

