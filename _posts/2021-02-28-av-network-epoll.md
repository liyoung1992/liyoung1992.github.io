---
layout: post
title:  "音视频-03-网络编程-基于epoll服务器/客户端"
categories: webrtc
tags:  webrtc C++ 网络编程
---


* content
{:toc}

网络编程基础

<!--excerpt-->

## 概述
网络编程基础（TCP server /client），基于epoll.

## 服务端

```cpp
#include <iostream>
#include <sys/socket.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <unistd.h>
#include <string.h>
#include <fcntl.h>
#include <sys/epoll.h>
#include <sys/wait.h>
#define MAX_EVENTS 20
#define FD_SIZE 1024
#define TIMEOUT 500
#define MAX_PROCESS 4
class TCPServer
{
public:
    TCPServer():socket_opt_on(1){

    };
    ~TCPServer(){};
    /*set port*/
    void set_port(const int& p) {
        port = p;
    }
    int get_port() const {
        return port;
    }
    bool run(){
    /*
    AF_INET             IPv4 Internet protocols 

    SOCK_STREAM   Provides sequenced, reliable, two-way, connection-
                       based byte streams.  An out-of-band data transmis‐
                       sion mechanism may be supported. 
    */
        // int events = 0;
        // int max_fd = -1;
        socket_fd = socket(PF_INET,SOCK_STREAM,0);
        if (socket_fd == -1) {
            std::cout << "failed to create socket" << std::endl;
            return false;
        }
        auto flags = fcntl(socket_fd,F_GETFL,0);
        fcntl(socket_fd,F_SETFL,flags | O_NONBLOCK);

        //opt 
        auto ret = setsockopt(socket_fd,
                    SOL_SOCKET,SO_REUSEADDR,
                    &socket_opt_on,sizeof(socket_opt_on));
        if (ret == -1) {
            std::cout << "setsockopt error" <<std::endl;
            return false;
        }
        //bind
        struct sockaddr_in localaddr,remoteaddr;
        localaddr.sin_family = AF_INET;
        localaddr.sin_port = port;
        localaddr.sin_addr.s_addr = INADDR_ANY;
        bzero(&(localaddr.sin_zero),8);
        ret = bind(socket_fd,(struct  sockaddr *)&localaddr,
                sizeof(struct sockaddr));

       if (ret == -1) {
           std::cout << "bind error" <<std::endl;
           return false;
       }
       //listen
       ret = listen(socket_fd,backlog);
        if (ret == -1) {
            std::cout << "listen error" <<std::endl;
            return false;
        }
        int epoll_fd;

        struct epoll_event ev, events[MAX_EVENTS];
        int event_number;
        pid_t pid = -1;
        for (int i = 0; i < MAX_PROCESS;i++) {
            if (pid != 0) {
                pid = fork();
            }
        }
        if (pid == 0)
        {
            epoll_fd = epoll_create(256);
            ev.events = EPOLLIN;
            ev.data.fd = socket_fd;
            epoll_ctl(epoll_fd,EPOLL_CTL_ADD,socket_fd,&ev);

        while (1)
        {
            event_number = epoll_wait(epoll_fd,events,
                                MAX_EVENTS,TIMEOUT);
            for (size_t i = 0; i < event_number; i++)
            {
                if (events[i].data.fd == socket_fd) {
                    socklen_t addr_len = sizeof(struct socketaddr*);
                    accept_fd = accept(socket_fd,
                     (struct sockaddr*) &remoteaddr,&addr_len);

                    //set socket sysn
                    auto flags = fcntl(accept_fd,F_GETFL,0);
                    fcntl(accept_fd,F_SETFL,flags | O_NONBLOCK);

                    ev.events = EPOLLIN | EPOLLET;
                    ev.data.fd = accept_fd;
                    epoll_ctl(epoll_fd,EPOLL_CTL_ADD,accept_fd,&ev);
                    
                }else if (events[i].events && EPOLLIN) {
                    do
                    {
                        memset(in_buff, 0, message_len);
                        ret = recv(events[i].data.fd, (void *)in_buff, message_len, 0);
                        std::cout << "ret:" << ret << "  in_buff:" << std::endl;
                        if (ret == 0)
                        {
                            close(events[i].data.fd);
                        }
                        if (ret == message_len) {
                            std::cout << "data full" << std::endl;
                        }
                        // std::cout << "recv :" << in_buff << std::endl;
                        // send(events[i].data.fd,(void*)in_buff,message_len,0);
          
                    } while (ret < -1&& errno == EINTR);
                    if (ret < 0) {
                        switch (errno)
                        {
                        case EAGAIN:
                            break;
                        default:
                            break;
                        }
                    }
                    if (ret > 0) {
                        std::cout << "receive message:" << in_buff << std::endl;
                        send(events[i].data.fd,(void*)in_buff,message_len,0);
                    }
                }
            }
        }
       
        close(socket_fd);
        }
        else {//pid != 0
            do {
                pid = waitpid(-1,NULL,0);
            } while (pid != -1);
         }
    }
private:
    int socket_fd;
    int accept_fd;
    int socket_opt_on;
    int port = 8111;
    int backlog = 10;
    int message_len = 1024;
    char in_buff[1024] = {0,};
};


int main() {
    TCPServer server;
    
    server.run();
    // if (ret) {
    //     std::cout << "server running " << std::endl;
    // }else {
    //     std::cout << "server running error" << std::endl;
    // }
}
```

## 客户端

```cpp
#include <iostream>
#include <sys/socket.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>
#include <stdio.h>
#define PORT 8111
#define MESSAGE_LEN 1024
int main() {
    int  socket_fd;
    socket_fd = socket(AF_INET,SOCK_STREAM,0);
    if (socket_fd < 0) {
        std::cout << "failed to create socket" << std::endl;
        exit (-1);
    }
    struct sockaddr_in serveraddr;
    serveraddr.sin_family = AF_INET;
    serveraddr.sin_port = 8111;
    serveraddr.sin_addr.s_addr = inet_addr("127.0.0.1");
    auto ret = connect(socket_fd,(struct sockaddr *)&serveraddr,sizeof(struct sockaddr));
    if (ret < 0) {
        std::cout << "failed to connect server!" << std::endl;
        exit (-1);
    }
    char sendbuff[MESSAGE_LEN] = {0,};
    char recvbuff[MESSAGE_LEN] = {0,};

    while (1) {
        memset(sendbuff,0,MESSAGE_LEN);
       // gets(sendbuff);
        scanf("%s", sendbuff);
        std::cout << "client send --" << sendbuff << std::endl;
        ret = send(socket_fd,sendbuff,strlen(sendbuff),0);
        if (ret <= 0) {
            std::cout << "failed to send data" << std::endl;
            break;
        }

        if (strcmp(sendbuff,"quit") == 0) {
            break;
        }
        ret = recv(socket_fd,recvbuff,MESSAGE_LEN,0);
        recvbuff[ret] = '\0';
        std::cout << "recv:" << recvbuff << std::endl;
    }
}

```

## 使用方法

### 编译：
- `g++ tcp_server.cpp -g -o tcp_server -std=c++17`
- `g++ tcp_client.cpp -o tcp_client -std=c++17`

### 运行
```cpp
./tcp_server 
----------------
ret:6  in_buff:
recv :12sasa
----------------
ret:5  in_buff:
recv :clinta

./tcp_client 
12sasa
client send --12sasa
recv:12sasa
clint send 
client send --clint
```