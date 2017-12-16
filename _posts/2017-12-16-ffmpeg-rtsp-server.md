---
layout: post
title:  "ffmpeg学习-搭建rtsp服务器"
categories: go
tags:  go redis
---


* content
{:toc}

利用VLC播放器搭建RTSP服务器

<!--excerpt-->

## 概述

最近在研究ffmpeg，网上的教程虽然多，但是大部分都是没有经过试验，在此把我自己的学习过程分享出来，给大家一个参考。这篇文章主要是搭建一个rtsp服务器，后面的很多过程都是基于rtsp来推送流！

## 具体步骤

1.下载VLC，直接百度就好！

下面看具体流程：

![](https://raw.githubusercontent.com/liyoung1992/liyoung1992.github.io/master/image/rtsp-1.png)

![](https://raw.githubusercontent.com/liyoung1992/liyoung1992.github.io/master/image/rtsp-2.png)

![](https://raw.githubusercontent.com/liyoung1992/liyoung1992.github.io/master/image/rtsp-3.png)

![](https://raw.githubusercontent.com/liyoung1992/liyoung1992.github.io/master/image/rtsp-4.png)

![](https://raw.githubusercontent.com/liyoung1992/liyoung1992.github.io/master/image/rtsp-5.png)

![](https://raw.githubusercontent.com/liyoung1992/liyoung1992.github.io/master/image/rtsp-6.png)

![](https://raw.githubusercontent.com/liyoung1992/liyoung1992.github.io/master/image/rtsp-7.png)

![](https://raw.githubusercontent.com/liyoung1992/liyoung1992.github.io/master/image/rtsp-8.png)


最后在rtsp的端口前面加上自己电脑的ip即可播放。播放直接打开VLC媒体，播放流。输入rtsp地址：rtsp://192.168.1.188:5544/1






