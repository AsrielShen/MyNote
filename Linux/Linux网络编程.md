# Socket编程

## 监听流程

### 创建socket

```cpp
int socket(int domain, int type, int protocol);
int sfd = socket(AF_INET, SOCK_STREAM, 0);

参数的含义如下：
//domain (协议族)：
指定套接字的协议族或地址类型。常见的值有：
AF_INET：IPv4 地址族，用于 Internet 协议。
AF_INET6：IPv6 地址族。
AF_UNIX：本地通信，用于 Unix 域套接字。
AF_PACKET：底层网络设备，通常用于 Linux。

//type (套接字类型)：
指定套接字的类型。常见的值有：
SOCK_STREAM：面向连接的字节流（TCP）套接字。
SOCK_DGRAM：数据报套接字（UDP）。
SOCK_RAW：原始套接字，允许应用程序直接访问底层协议（例如 ICMP）。

//protocol (协议)：
指定所使用的协议。通常设置为 0，由操作系统根据 domain 和 type 自动选择合适的协议。例如：
对于 SOCK_STREAM 和 AF_INET，系统会选择 IPPROTO_TCP。
对于 SOCK_DGRAM 和 AF_INET，系统会选择 IPPROTO_UDP。
如果明确指定协议，可以使用具体的协议常量，如 IPPROTO_TCP、IPPROTO_UDP 或 IPPROTO_ICMP。
```

### 绑定socket

`bind()` 函数用于将一个本地协议地址（例如 IP 地址和端口号）绑定到一个套接字上。这是服务器端网络编程中非常重要的一步，因为它告诉操作系统在哪个地址和端口上监听传入的连接请求。

```cpp
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>

int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);

struct sockaddr_in server_addr;
memset(&server_addr, 0, sizeof(server_addr));  // 清空结构体
server_addr.sin_family = AF_INET;               // 使用 IPv4 地址族
server_addr.sin_addr.s_addr = inet_addr("127.0.0.1");  // 设置本地 IP 地址
server_addr.sin_port = htons(8080);

if (bind(sockfd, (struct sockaddr *)&server_addr, sizeof(server_addr)) < 0) {
    perror("Bind failed");
    return 1;
}

//参数解析：
//sockfd：
套接字文件描述符，通常是由 socket() 创建的套接字描述符。该套接字将会与指定的地址（addr）进行绑定。

//addr：
一个指向 struct sockaddr 类型的指针，它包含了要绑定的地址信息。
通常该结构体会被类型转换为特定协议族的结构体类型，例如：
struct sockaddr_in：用于 IPv4 地址。
struct sockaddr_in6：用于 IPv6 地址。
addr 包含了 IP 地址和端口号等信息。

//addrlen：
addr 所指向结构体的长度。对于 struct sockaddr_in 类型，通常是 sizeof(struct sockaddr_in)。
该参数用于告知内核地址结构体的大小，以便它能够正确地读取地址数据。

//作用：
bind() 的作用是将套接字与一个特定的本地地址（通常是 IP 地址和端口号）绑定，通常服务器会使用该套接字来接收客户端的连接请求。
例如，在一个服务器程序中，你需要指定服务器的 IP 地址和端口，以便客户端能够通过该地址和端口与服务器建立连接。
如果服务器没有明确绑定端口，操作系统可能会为其分配一个随机端口。

```

### 监听socket

```cpp
int listen(int sockfd, int backlog);
listen(sockfd, 5);

//参数说明：
sockfd：已绑定的套接字文件描述符（通过 socket() 和 bind() 创建）。该套接字用于监听连接请求。
backlog：连接请求队列的大小，表示在服务器开始接受连接之前，内核可以缓存的最大未处理连接数。

//主要功能：
监听连接请求：listen() 函数使得套接字进入监听状态，准备接受来自客户端的连接。此时，套接字会等待并排队处理连接请求。
定义最大连接队列：backlog 参数定义了内核用于存放已建立连接请求的队列的大小。此队列中存放的是尚未被应用程序 accept() 接受的连接。
```

### 修改socket属性

- `setsockopt()` 是一个用于设置套接字选项的系统调用，允许开发者配置套接字的行为或属性。它在网络编程中非常重要，尤其是在配置套接字的缓冲区大小、超时、连接特性等方面。



```cpp
int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);

//参数说明：
//sockfd：待设置选项的套接字文件描述符。通常由 socket() 或 accept() 返回。

//level：指定要设置的选项的协议层次。常见的协议层包括：
SOL_SOCKET：表示套接字层，适用于所有套接字相关的设置。
IPPROTO_TCP：表示 TCP 层，适用于 TCP 协议相关的设置。
IPPROTO_IP：表示 IP 层，适用于 IP 协议相关的设置。

//optname：指定要设置的具体选项。不同的 level 会有不同的选项。例如：
SO_RCVBUF：接收缓冲区的大小。
SO_RCVBUF：发送缓冲区的大小。
SO_REUSEADDR：允许重用本地地址。
/*
TCP的唯一标识符是一个五元组，一对socket和一个协议

这个选项通常在服务器端使用，特别是在服务器重启后希望立即重新绑定到同一端口时。
在TIME_WAIT时快速绑定
当你在 bind 一个端口时，操作系统会检查该端口是否已经被占用。TIME_WAIT 状态的连接会占用端口，但并不会阻止新程序绑定到同一个端口。

*/  
//强制关闭，还是默认缓冲发送完后关闭
SO_LINGER：控制关闭套接字时的行为。
TCP_NODELAY：禁用 Nagle 算法。
SO_TIMEOUT：设置套接字超时。

//optval：指向选项值的指针。不同的选项需要不同类型的值。例如，设置接收缓冲区大小时，optval 会指向一个整数，设置 SO_RCVBUF 时，通常是一个整型变量，表示缓冲区的字节数。

//optlen：optval 参数的长度，以字节为单位。例如，对于整数类型的选项，通常是 sizeof(int)，对于结构体类型的选项，则是结构体的大小。

```



## 接受流程

### 服务器接受socket连接

服务器端通过 `accept()` 函数接受客户端的连接请求，返回一个新的套接字。

```cpp
int client_sockfd = accept(sockfd, NULL, NULL);  // 接受连接
```



### 客户端创建socket连接

客户端使用 `connect()` 函数连接到服务器。

```cpp
struct sockaddr_in server_addr;
server_addr.sin_family = AF_INET;
server_addr.sin_port = htons(8080);
server_addr.sin_addr.s_addr = inet_addr("127.0.0.1");  // 连接到本地

if (connect(sockfd, (struct sockaddr*)&server_addr, sizeof(server_addr)) == -1) {
    perror("Connect failed");
    close(sockfd);
    exit(1);
}

```





## 读写流程

### 读写socket

一旦连接建立，客户端和服务器可以使用 `send()` 和 `recv()` 函数进行数据交换。

```cpp
#include <sys/socket.h>

ssize_t recv(int sockfd, void *buf, size_t len, int flags);
ssize_t send(int sockfd, const void *buf, size_t len, int flags);
```



```cpp
// 发送数据
const char* message = "Hello, Server!";
if (send(sockfd, message, strlen(message), 0) == -1) {
    perror("Send failed");
}

// 接收数据
char buffer[1024];
int len = recv(sockfd, buffer, sizeof(buffer), 0);
if (len == -1) {
    perror("Recv failed");
}
buffer[len] = '\0';  // 添加字符串结束符
printf("Received message: %s\n", buffer);

```

