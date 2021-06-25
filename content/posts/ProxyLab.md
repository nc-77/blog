---
title: ProxyLab
date: 2020-12-16 16:29:57
tags: [csapp,计算机网络]
categories: [CSAPPLab]
---

## 实验说明

在这个实验中，我们将实现一个多线程带缓存的代理服务器，听起来是不是很cool，而且这个实验最终可以不只是以一个打分结束，这个完全手写的代理服务器还可以真正用在浏览器上。

先来一波实验效果图。推荐使用firefox进行测试，设置如下：

如果是在远程服务器上开启的服务，填服务器的公网ip+对应端口

本地就是loaclhost或者127.0.0.1+对应端口了

<img src="https://img.nc-77.top/20201216164921.png" style="zoom: 67%;" />

测试网站：http://home.baidu.com/home/index/contact_us

代理服务未启动前：

<img src="https://img.nc-77.top/20201216165016.png" style="zoom: 50%;" />

代理服务启动后：

<img src="https://img.nc-77.top/20201216170359.png" style="zoom:50%;" />

注：本实验所有代码均已上传至[github](https://github.com/nc-77/cmu_csapp_lab/tree/master/ProxyLab/proxylab-handout)上，需要自取

## 前置知识

### 客户端—服务器编程模型

![](https://img.nc-77.top/20201216171148.png)

需要注意的是，客户端和服务器是进程，而不是主机。一台主机上可以同时运行许多不同的客户端和服务器。

### 因特网连接

因特网客户端和服务器通过在连接上发送和接收字节流来通信，并且这种连接是点对点的，具体来说，是通过socket来实现的。

socket地址是由一个因特网地址和一个16位的整数端口组成的，形如”地址：端口“，一个socket就是连接的一个端点。

所以一个连接是由它两端的socket地址唯一确定的。形如（clientaddr：cliport，servaddr：servport）,如下图。

![](https://img.nc-77.top/20201216172115.png)

### 具体连接过程

先上书上的一张向导图。

<img src="https://img.nc-77.top/20201216173317.png" style="zoom: 67%;" />

从客户端开始说起

1. 将服务器的ip+端口通过getaddrinfo转换成socket地址结构

2. 利用socket函数来创建一个socket descriptor——clientfd，客户端将通过读写clientfd来与服务器进行通信，当然，此时的clientfd仅仅是打开的，还不能用于读写。

3. 客户端通过connet函数与目标服务器建立一个因特网连接，connet会阻塞，一直到连接成功建立或是发生错误，如果成功，就可以通过读写clientfd来与服务器进行通信了。

再来说服务器这边

1. 将服务器的端口通过getaddrinfo转换成socket地址结构
2. 同客户端一样创建一个socket descriptor——listenfd，但这个listenfd并不直接与客户端进行实质内容上的通信，listenfd仅作为客户端连接请求的一个端点，并存在与服务器的整个周期。
3. bind将刚刚生成的listenfd与客户端地址绑定起来。
4. listen告诉内核将listenfd从一个主动套接字转化为一个监听套接字，该套接字可以接受来自客户端的请求。
5. 至此，服务器的准备工作都做完了，接下来就是accept客户端的connect了，服务器会等待来自客户端的连接请求到达listenfd并生成一个已连接描述符（connected descriptor),服务器通过读写connfd来与客户端进行通信。

这里有一个问题，服务器的listenfd与connfd有什么区别，为什么需要两个，这是因为它使得我们可以实现并发服务器，每次一个连接请求到listenfd，我们可以fork一个新的进程，通过新的connfd与客户端进行通信，从而能够同时处理许多客户端的连接。

值得一提的是，从linux的角度来看，socket就是一个有相应描述符的打开文件。

## PART1

前面说了这么多，让我们直接进入实战吧。

当然，在做这个lab之前，推荐先去把书上实现的一个tiny server复现一下。

part1部分其实要做的工作很少，我们只需要在书上实现的tiny server框架下改写实现一个proxy能转发客户端的请求到服务器，同时服务器的回复也通过proxy转发到客户端。

我的思路是这样的，proxy相对于客户端而言是作为服务器的，相对于服务器而言又作为客户端，相当于proxy充当了客户端和服务器的双重身份。

因此proxy总体的框架如下所示（是相当于客户端的服务器）

```c
#include "csapp.h"
#include "doit.h"

int main(int argc,char **argv)
{
    int listenfd,connfd;
    socklen_t clientlen;
    char *port;
    char hostname[MAXLINE];
    struct sockaddr_storage clientaddr;
   
    if (argc != 2) {
	fprintf(stderr, "usage: %s <port>\n", argv[0]);
	exit(1);
    }
    port=argv[1];
    listenfd=Open_listenfd(port);

    while (1) {
        clientlen = sizeof(clientaddr);
        connfd = Accept(listenfd, (SA *)&clientaddr, &clientlen); 
            Getnameinfo((SA *) &clientaddr, clientlen, hostname, MAXLINE, 
                        port, MAXLINE, 0);
            printf("Accepted connection from (%s, %s)\n", hostname, port);
       
        doit(connfd);
        Close(connfd);
    }
    return 0;
}


```

其中doit部分要处理的就是接受客户端Req并发送至服务器，最后再向客户端发送服务器的response。

```c
void doit(int listenfd)
{
    /* 
        通过读写listenfd 与客户端通信
        通过读写clientfd 与服务端通信
     */
    rio_t rio_listen,rio_client;
    int n;
    char buf[MAXLINE],method[MAXLINE],uri[MAXLINE],version[MAXLINE];
    char newreq[MAXLINE+20];
    Request req;
    int clientfd;

    /* proxy 接收客户端 Req */
    Rio_readinitb(&rio_listen,listenfd);
    Rio_readlineb(&rio_listen,buf,MAXLINE);
    printf("Request line:\n");
    printf("%s\n",buf);
    sscanf(buf,"%s %s %s",method,uri,version);
    
    if(strcasecmp(method,"GET")){
        /* 忽略除get外所有请求 */
        clienterror(listenfd,method,"501","Not implemented","Tiny does not implement this method");
        return;
    }
    /* 忽略所有请求报头 */
    read_requestHeaders(&rio_listen);
  
    /* 解析uri */
    parse_uri(&req,uri);
    
    /* 浏览器使用 */
    sprintf(newreq, "GET http://%s:%s%s HTTP/1.0\r\n\r\n", req.host,req.port,req.path); 
    /* drive.sh 测试用 */
    //sprintf(newreq, "GET %s HTTP/1.0\r\n\r\n", req.path);  
    /* proxy 向服务端发送 request */
    clientfd=Open_clientfd(req.host,req.port);
    Rio_writen(clientfd,newreq,MAXLINE);

    printf("proxy send request successfully\n");

    /* proxy 向客户端发送 response */
    Rio_readinitb(&rio_client,clientfd);

    while ((n = Rio_readlineb(&rio_client, buf, MAXLINE))) {//real server response to buf
        printf("proxy received %d bytes,then send\n",n);
        Rio_writen(listenfd, buf, n);  //real server response to real client
    }

    Close(clientfd);
}
```

### 辅助函数

#### read_requesetHeaders

读取所有请求头并忽略

```c
void read_requestHeaders(rio_t *rp){
    char buf[MAXLINE];
    memset(buf,0,sizeof(buf));
    printf("request headers:\n");
    while(strcmp(buf,"\r\n")){
        Rio_readlineb(rp,buf,MAXLINE);
        printf("%s",buf);
    }
}
```

#### parse_uri

分割Req中的host，port，path，没有指明的端口默认80

```c
void parse_uri(Request *req,char *uri){
    //printf("parse_uri begin\n");
    
    if (strstr(uri, "http://") != uri) {
        fprintf(stderr, "Error: invalid uri!\n");
        exit(0);
    }
    uri += strlen("http://");
    char *c = strstr(uri, ":");
    if(!c){
        /* 无显示端口默认80 */
        strcpy(req->port,"80");
        c = strstr(uri, "/");
        *c = '\0';
        strcpy(req->host, uri);
        *c = '/';
        strcpy(req->path, c);
    }
    else{
        *c = '\0';
        strcpy(req->host, uri);
        uri = c + 1;
        c = strstr(uri, "/");
        *c = '\0';
        strcpy(req->port, uri);
        *c = '/';
        strcpy(req->path, c);
    }
    
    //printf("parse_uri end\n");
    //printf("host=%s,port=%s,path=%s\n",req->host,req->port,req->path);
}
```

用提供的./drive.sh 测试一下

<img src="https://img.nc-77.top/20201217102137.png" style="zoom: 80%;" />

基础的部分都拿到了，代表我们已经实现了一个单线程无缓存的超级简陋的proxy。当然，即使这个只有最基本的功能，你也可以像文章开头一样在浏览器中使用这个proxy。

## PART2

实现多线程

咕~

## PART3

实现缓存

一意咕行~

## 总结

这是csapp中的最后一个lab了，写完就相当于结束了csapp系列。写下csapp的阅读感受吧。

从这学期初开始阅读csapp，花了差不多半个学期多的时间，跟着b站上的视频囫囵吞枣过了一遍，毕竟还没正式接触过计网，计组和操作系统。虽然感觉老师上课一些的概念已经讲的简单化了，但是很是有许多一知半解的东西。其间还跳过了一些偏硬件的章节。

当然，给我帮助最大的还是csapp配套的lab，代码注入攻击，实现内存管理器，实现一个shell，实现一个proxy......这些在以前看来是想都不敢想的事情，但是跟着这些lab，我也都自己做出来了（虽然不是很完善，而且借鉴了网上其他人的做法）。

总而言之，阅读完csapp给我最大的感受是开阔了眼界吧，估计以后系统学完计算机知识后应该会再来重温一遍这本经典吧，毕竟还留了几个坑在这儿。

临近考试周，这应该是本学期最后一篇博客了吧，下次应该是2020年总结（先立个flag,逃）

