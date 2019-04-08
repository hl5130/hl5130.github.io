---
title: OkHttpClient的Timeout解析
date: 2019-04-08 16:33:50
tags: okHttp3
categories: Android
---
# 简介
okHttp3要设置OkHttpClient的TimeOut，需要通过``OkHttpClient.Builder``来设置。如下代码：
```
OkHttpClient client = new OkHttpClient.Builder()
                .connectTimeout(15,TimeUnit.SECONDS)
                .readTimeout(20, TimeUnit.SECONDS)
                .writeTimeout(20, TimeUnit.SECONDS)
                .cache(new Cache(file,10*1024*1024))
                .build();
```

有时候我很疑惑，``connectTimeout``、``readTimeout``、``writeTimeout``，到底有什么用？
下面来讲解一下：
# 解析
首先简单说一下``socket``，因为``okHttp``是基于``socket``实现的。
## socket
socket是对网络传输层TCP/IP协议的封装，便于程序员进行使用。它定义了一些基本的函数：create、listen、connect、accept、send、read和write等等。其中``connect``对应于TCP协议中的三次握手建立连接，``read``对应于建立连接之后客户端读取服务器数据的操作，``write``对应于建立连接之后客户端向服务器写入数据的操作。只需要了解这些就可以了，如果要深入了解，请查看《[TCP/IP、Http、Socket区别]()》这一篇文章。
## connectTimeout
connectTimeout 既连接超时。精确来讲就是客户端和服务器要在设置的时间内进行三次握手并建立连接，如果在此时间内没有建立连接，那就是连接超时。
作用：限制客户端和服务器建立连接的时间，减少服务器的压力，释放更多资源供其他连接使用。
## readTimeout
readTimeout 既读取超时。精确来讲就是客户端要在规定的时间内从服务器读取到数据内容，注意不是读取所有数据花费的时间。例如服务器有一个大小为100字节的数据，只要客户端在 readTimeout时间内能够从服务器读取到1字节的数据，就不会超时。举一个更生动的例子：用桶接水，打开水龙头(三次握手成功建立了连接)，有水流出来，无论大小(获取到的数据流大小)，但都是有水的；此时停水了(相当于出现了网络状况，导致读取不到数据了)，从现在开始计算到readTimeout时间，还没有水出来的话(说明读取超时了)，就关闭水龙头(关闭连接)。
## writeTimeout
writeTimeout 既写入超时。精确来讲就是客户端向服务器上传文件时，要在规定的时间向服务器发送数据。比如我要发10M的数据，结果只发了1M出去，网络就一直丢包，从这个时间起就开始算超时，但是只要有新的数据发出去了那么超时就重新计算。说白了超时就是每次最多干等的时间。
