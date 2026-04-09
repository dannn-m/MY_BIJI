---
date: 2026-03-28T08:17:00
aliases:
  - 网络编程库
tags:
  - 网络，py，编程
---
## 一、常见用法

### （一）创建 Socket 对象

使用 `socket.socket()` 方法创建一个 socket 对象，它需要两个参数：

- **`family`**：指定协议族，常用的有 `AF_INET` 代表 IPv4 协议族，`AF_INET6` 代表 IPv6 协议族。
- **`type`**：指定套接字类型，`SOCK_STREAM` 表示基于 TCP 协议（面向流，提供可靠的、有序的字节流传输），`SOCK_DGRAM` 表示基于 UDP 协议（面向数据报，不保证数据的顺序和完整性）。
    
    示例：

python

```
# 创建一个基于IPv4和TCP协议的socket对象
tcp_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# 创建一个基于IPv4和UDP协议的socket对象
udp_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
```

### （二）绑定（Bind）

对于服务器端 socket，需要使用 `bind()` 方法将其绑定到一个特定的 IP 地址和端口号，这样客户端才能连接到该地址。

- **参数**：接受一个元组，格式为 `(ip_address, port)` 。`ip_address` 可以是具体的 IP 地址，也可以是 `'localhost'` 表示本地主机；`port` 是一个 1 - 65535 之间的整数（一般 1024 以下的端口为系统保留端口，建议使用大于 1024 的端口）。
    
    示例：

python

```
server_address = ('localhost', 8888)
tcp_socket.bind(server_address)
```

### （三）监听（Listen）

仅适用于服务器端的 TCP socket。`listen()` 方法使 socket 进入监听状态，等待客户端的连接请求。

- **参数**：`backlog` 表示等待连接的最大数量，它是一个整数。
    
    示例：

python

```
tcp_socket.listen(5)
```

### （四）接受连接（Accept）

同样仅适用于服务器端的 TCP socket。`accept()` 方法用于接受客户端的连接请求，它是一个阻塞式方法，即程序执行到这里会暂停，直到有客户端连接进来。

- **返回值**：返回一个元组，包含两个元素，第一个是新的 socket 对象（用于与该客户端进行通信），第二个是客户端的地址（元组形式，包含 IP 地址和端口号）。
    
    示例：

python

```
client_socket, client_address = tcp_socket.accept()
print(f'连接来自: {client_address}')
```

### （五）连接（Connect）

用于客户端的 TCP socket，`connect()` 方法使客户端连接到服务器指定的 IP 地址和端口号。

- **参数**：与 `bind()` 方法中的地址参数格式相同，是一个元组 `(ip_address, port)` 。
    
    示例：

python

```
server_address = ('localhost', 8888)
tcp_socket.connect(server_address)
```

### （六）发送数据（Send）

无论是服务器端还是客户端，使用 `send()` 方法来发送数据。

- **参数**：要发送的数据必须是字节串（在 Python 3 中，字符串需要先编码为字节串）。
    
    示例：

python

```
message = '这是要发送的消息'
tcp_socket.send(message.encode())
```

### （七）接收数据（Recv）

使用 `recv()` 方法接收数据。

- **参数**：`bufsize` 指定接收缓冲区的大小（以字节为单位），它决定了一次最多能接收多少数据。
    
    示例：

python

```
data = tcp_socket.recv(1024)
print(f'收到数据: {data.decode()}')
```

### （八）关闭连接（Close）

使用 `close()` 方法关闭 socket 连接，释放相关资源。

示例：

python

```
tcp_socket.close()
```

## 二、实战案例

### （一）TCP 服务器 - 客户端示例

#### 1. 服务器端代码

python

```
import socket

# 创建一个TCP socket
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# 绑定IP地址和端口号
server_address = ('localhost', 8888)
server_socket.bind(server_address)

# 监听连接
server_socket.listen(5)

while True:
    # 接受客户端连接
    client_socket, client_address = server_socket.accept()
    print(f'连接来自: {client_address}')

    # 接收数据
    data = client_socket.recv(1024)
    print(f'收到数据: {data.decode()}')

    # 发送响应
    response = '你好，客户端！'
    client_socket.send(response.encode())

    # 关闭连接
    client_socket.close()
```

- **重要代码讲解**：
    
    - `server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)`：创建一个基于 IPv4 和 TCP 协议的 socket 对象，用于服务器端监听客户端连接。
    - `server_socket.bind(server_address)`：将服务器 socket 绑定到本地地址 `localhost` 和端口号 `8888`，这样客户端就可以通过这个地址和端口连接到服务器。
    - `server_socket.listen(5)`：使服务器 socket 进入监听状态，最多允许 5 个客户端连接在等待队列中。
    - `client_socket, client_address = server_socket.accept()`：这是一个阻塞式调用，等待客户端连接。一旦有客户端连接，返回一个新的 socket 对象 `client_socket` 用于与该客户端通信，同时获取客户端的地址 `client_address`。
    - `data = client_socket.recv(1024)`：从与客户端连接的 socket 接收数据，最多接收 1024 字节。
    - `client_socket.send(response.encode())`：向客户端发送响应数据，这里先将字符串编码为字节串再发送。
    - `client_socket.close()`：关闭与客户端的连接，释放资源。
    

#### 2. 客户端代码

python

```
import socket

# 创建一个TCP socket
client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# 连接到服务器
server_address = ('localhost', 8888)
client_socket.connect(server_address)

# 发送数据
message = '这是来自客户端的消息'
client_socket.send(message.encode())

# 接收响应
response = client_socket.recv(1024)
print(f'收到响应: {response.decode()}')

# 关闭连接
client_socket.close()
```

- **重要代码讲解**：
    - `client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)`：创建一个基于 IPv4 和 TCP 协议的 socket 对象，用于客户端连接服务器。
    - `client_socket.connect(server_address)`：客户端尝试连接到服务器指定的地址 `localhost` 和端口号 `8888`。
    - `client_socket.send(message.encode())`：将本地的消息编码后发送给服务器。
    - `response = client_socket.recv(1024)`：从服务器接收响应数据，最多接收 1024 字节。
    - `client_socket.close()`：关闭客户端 socket 连接，释放资源。
## 三、聊天系统
[[聊天系统]]
## 四、进阶使用
#### 1.多路复用
 
##### （1）概述
 - 传统的socket编程中一个进程只能处理一个socket链接，如果多个客户端同时链接服务器，服务器要么使用多线程要么使用多进程处理，这样会消耗大量的系统资源。

- 多路复用技术允许一个进程可以监控多个socket，当其中任何一个socket有数据可读，可写或者出错时，进程就能及时进行通知并得到相应处理

##### （2）实现方式
- select：这是最早的多路复用实现方式。`select`函数通过监听一组文件描述符（socket 在系统中以文件描述符形式存在），可以同时监控多个 socket 的读、写和异常事件。它的优点是跨平台性好，几乎所有操作系统都支持；缺点是它能监控的文件描述符数量有限（通常在 1024 个以内），并且随着监控的 socket 数量增加，性能会逐渐下降，因为它采用轮询的方式检查每个 socket 是否有事件发生。

- **poll**：`poll`函数和`select`类似，但它没有文件描述符数量的限制，并且采用链表的方式存储被监控的文件描述符，在一定程度上提高了效率。不过，它仍然是基于轮询机制，当 socket 数量非常大时，性能问题依然存在。

- **epoll**：这是 Linux 特有的高性能多路复用机制。`epoll`采用事件驱动的方式，当某个 socket 有事件发生时，内核会将该事件通知给应用程序，而不需要应用程序去轮询所有的 socket。`epoll`有两种工作模式：水平触发（LT）和边缘触发（ET）。水平触发模式下，只要 socket 对应的缓冲区有数据（可读或可写），就会一直通知应用程序；边缘触发模式则只有在 socket 状态发生变化（如从无数据到有数据）时才通知应用程序，这种模式效率更高，但编程难度也相对较大，因为需要确保在一次通知中尽可能多地处理数据，否则可能会遗漏事件。
##### （3）实列
~~~
import socket
import selectors

sel = selectors.DefaultSelector()


def accept(sock, mask):
    conn, addr = sock.accept()
    print('accepted', conn, 'from', addr)
    conn.setblocking(False)
    sel.register(conn, selectors.EVENT_READ, read)


def read(conn, mask):
    data = conn.recv(1024)
    if data:
        print('echoing', repr(data), 'to', conn)
        conn.send(data)
    else:
        print('closing', conn)
        sel.unregister(conn)
        conn.close()


sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.bind(('localhost', 8888))
sock.listen(100)
sock.setblocking(False)
sel.register(sock, selectors.EVENT_READ, accept)

while True:
    events = sel.select()
    for key, mask in events:
        callback = key.data
        callback(key.fileobj, mask)
~~~
### 2. 异步 I/O

##### （1)**概念**：
- 在传统的 socket 编程中，无论是发送还是接收数据，I/O 操作通常是阻塞的。这意味着当执行`recv`或`send`方法时，程序会暂停执行，直到数据接收完成或者发送完毕。而异步 I/O 允许程序在发起 I/O 操作后，不需要等待操作完成就可以继续执行其他任务，当 I/O 操作完成后，系统会通过回调函数或者事件通知程序。这样可以显著提高程序的整体性能和响应速度，特别是在处理大量 I/O 操作的网络应用中。
- **Python 中的异步 I/O**：Python 的`asyncio`库为异步编程提供了强大的支持。通过`asyncio`，你可以定义异步函数（使用`async def`语法），这些函数可以暂停和恢复执行，允许其他异步任务在等待 I/O 操作时运行。`await`关键字用于暂停异步函数的执行，直到等待的 I/O 操作完成。例如，在使用`asyncio`进行 socket 编程时，你可以使用`asyncio.open_connection`来创建一个异步的 TCP 连接，使用`reader.read`和`writer.write`进行异步的数据读取和写入。

 #####  (2)示例代码:
 
（使用 asyncio 实现简单的异步客户端）**：

python

```
import asyncio


async def tcp_echo_client(message):
    reader, writer = await asyncio.open_connection('localhost', 8888)
    print(f'Send: {message!r}')
    writer.write(message.encode())
    await writer.drain()

    data = await reader.read(100)
    print(f'Received: {data.decode()!r}')

    print('Close the connection')
    writer.close()
    await writer.wait_closed()


asyncio.run(tcp_echo_client('Hello, World!'))
```

### 3. 安全 Socket 通信

##### (1)概念：
- 在网络通信中，数据在传输过程中可能会被窃取、篡改或伪造。安全 Socket 通信通过使用加密和认证技术，确保数据的保密性、完整性和真实性。SSL（Secure Sockets Layer）及其继任者 TLS（Transport Layer Security）是常用的实现安全 Socket 通信的协议。
- **Python 中的实现**：Python 的`ssl`模块提供了对 SSL/TLS 协议的支持。通过`ssl.wrap_socket`方法，你可以将普通的 socket 包装成安全的 socket。在服务器端，你需要提供服务器证书（通常是.pem 格式）和私钥，用于验证服务器身份并加密通信数据；在客户端，你可以选择验证服务器证书（通过提供 CA 证书），以确保连接到的是可信的服务器。

##### (2)示例代码:
（使用 ssl 模块实现简单的安全服务器）

python

```
import socket
import ssl

context = ssl.SSLContext(ssl.PROTOCOL_TLSv1_2)
context.load_cert_chain(certfile='server.crt', keyfile='server.key')

bindsocket = socket.socket()
bindsocket.bind(('localhost', 8888))
bindsocket.listen(5)

while True:
    newsocket, fromaddr = bindsocket.accept()
    conn = context.wrap_socket(newsocket, server_side=True)
    try:
        data = conn.recv(1024)
        print(f'Received: {data.decode()}')
        conn.sendall(b'Hello, Client!')
    finally:
        conn.shutdown(socket.SHUT_RDWR)
        conn.close()
```