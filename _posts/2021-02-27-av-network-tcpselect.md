---
layout: post
title:  "音视频-01-网络编程-使用select实现高并发"
categories: webrtc
tags:  webrtc C++
---


* content
{:toc}

网络编程基础

<!--excerpt-->

## 概述
网络编程基础，使用select实现高并发

## 服务端

```cpp
#include <iostream>
#include <sys/socket.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <unistd.h>
#include <string.h>
#include <fcntl.h>
#define FD_SIZE 1024
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
        int events = 0;
        int max_fd = -1;
        socket_fd = socket(PF_INET,SOCK_STREAM,0);
        if (socket_fd == -1) {
            std::cout << "failed to create socket" << std::endl;
            return false;
        }
        //set socket sysn
        auto flags = fcntl(socket_fd,F_GETFL,0);
        fcntl(socket_fd,F_SETFL,flags| O_NONBLOCK);

          max_fd = socket_fd;
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
        fd_set fd_set;
        int accept_fds[FD_SIZE] = {0,};
        int  curpos;
        while(1) {
            FD_ZERO(&fd_set);
            FD_SET(socket_fd,&fd_set);
            for (int i = 0; i < FD_SIZE;i++) {
                if (accept_fds[i] != -1) {
                    if (accept_fds[i] > max_fd) {
                        max_fd = accept_fds[i];
                    }
                    FD_SET(accept_fds[i],&fd_set);
                }
                
            }
            events = select(max_fd+1,&fd_set,NULL,NULL,NULL);
            if (events < 0) {
                std::cout << "Failed to use select !" << std::endl;
                break;
            }else if (events == 0) {
                std::cout << "timeout ...." << std::endl;
            }else if (events) {
                if (FD_ISSET(socket_fd,&fd_set)) {
                    for (size_t i = 0; i < FD_SIZE; i++)
                    {
                        if (accept_fds[i] == -1) {
                            curpos = i;
                            break;
                        }
                    }
                socklen_t addr_len = sizeof(struct socketaddr*);
                accept_fd = accept(socket_fd,
                    (struct sockaddr*) &remoteaddr,&addr_len);

                        //set socket sysn
                auto flags = fcntl(accept_fd,F_GETFL,0);
                fcntl(accept_fd,F_SETFL,flags| O_NONBLOCK);
                accept_fds[curpos] = accept_fd;
                }
        for (size_t i = 0; i < FD_SIZE; i++)
            {
                if (accept_fds[i] != -1 && FD_ISSET(accept_fds[i],&fd_set)) {
                    memset(in_buff, 0, message_len);
                    ret = recv(accept_fds[i], (void *)in_buff, message_len, 0);
                    std::cout << "ret:" << ret << "  in_buff:" << std::endl;
                    if (ret == 0)
                    {
                        close(accept_fds[i]);
                        break;
                    }
                    std::cout << "recv :" << in_buff << std::endl;
                    send(accept_fds[i],(void*)in_buff,message_len,0);
                }
            }
            }
        }
        close(socket_fd);
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