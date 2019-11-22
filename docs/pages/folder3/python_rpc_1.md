# RPC 交互过程
### 什么是RPC

RPC (Remote Procedure Call)即远程过程调用，是分布式系统常见的一种通信方法，已经有 40 多年历史。当两个物理分离的子系统需要建立逻辑上的关联时，RPC 是牵线搭桥的常见技术手段之一。除 RPC 之外，常见的多系统数据交互方案还有分布式消息队列、HTTP 请求调用、数据库和分布式缓存等。

### RPC交互过程

RPC 是两个子系统之间进行的直接消息交互，它使用操作系统提供的套接字来作为消息的载体，以特定的消息格式来定义消息内容和边界。

RPC 的客户端通过文件描述符的读写 API (read & write) 来访问操作系统内核中的网络模块为当前套接字分配的发送 (send buffer) 和接收 (recv buffer) 缓存。
！[交互1](../../iamges/rpc-1-1.png)

如上图所示，左边的客户端进程写 RPC 指令消息到内核的发送缓存中，内核将发送缓存中的数据传送到物理硬件 NIC，也就是网络接口芯片 (Network Interface Circuit)。NIC 负责将翻译出来的模拟信号通过网络硬件传递到服务器硬件的 NIC。服务器的 NIC 再将模拟信号转成字节数据存放到内核为套接字分配的接收缓存中，最终服务器进程从接收缓存中读取数据即为源客户端进程传递过来的 RPC 指令消息。

消息从用户进程流向物理硬件，又从物理硬件流向用户进程，中间还经过了一系列的路由网关节点。

上图呈现的只是 RPC 一次消息交互的上半场，下半场是一个逆向的过程，从服务器进程向客户端进程返回响应数据。完整的一次 RPC 过程如下图所示：

！[交互2](../../iamges/rpc-1-2.png)

下面用 Python 代码来描述上述过程。

- Server 端死循环监听本地 8080 端口，等待客户端的连接。
- 客户端启动时连接本地 8080 端口，紧接着发送词一个字符串hello，然后等待服务器响应。
- 服务器接收到客户端连接后立即收取客户端发送过来的字符串，也就是hello，打印出来。
- 客户端接收到服务器发送过来的 world，马上打印出来。
- 关闭连接，结束。

```python
# coding: utf-8
# server

import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.bind(("localhost", 8081))
sock.listen(1)  # 监听客户端连接
while True:
    conn, addr = sock.accept()  # 接收一个客户端连接
    print(conn.recv(1024))  # 从接收缓冲读消息 recv buffer
    conn.sendall(b"world")  # 将响应发送到发送缓冲 send buffer
    conn.close() # 关闭连接...
    break

# coding: utf-8
# client

import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(("localhost", 8081))  # 连接服务器
sock.sendall(b"hello")  # 将消息输出到发送缓冲 send buffer
print(sock.recv(1024)) # 从接收缓冲 recv buffer 中读响应
sock.close() # 关闭套接字...
```
### 一个练习
客户端疯狂发送请求，但是服务器不读不处理，会发生什么？

答案是：由于接收滑动窗口消耗殆尽，不接收发送端消息，发送端将不断(时间每次都会递增)尝试接收端窗口信息。