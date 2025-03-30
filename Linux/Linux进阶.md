# Linux进阶内容

## 高级IO

### `readv()/writev()`

- `readv()` 是 Linux 系统中的一个系统调用，用于从文件描述符中读取多个非连续的内存缓冲区（i.e. 一组 `iovec` 结构所指向的缓冲区）到用户空间。这种方法比 `read()` 或 `recv()` 更高效，尤其是在需要处理多个缓冲区的场景下，避免了每次读操作时都需要进行系统调用

```cpp
#include <iostream>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/uio.h>

int main() {
    // 打开文件
    int fd = open("example.txt", O_RDONLY);
    if (fd == -1) {
        perror("open");
        return 1;
    }

    // 定义两个缓冲区
    char buf1[10];
    char buf2[20];
    
    // 设置 iovec 数组
    struct iovec iov[2];
    iov[0].iov_base = buf1;
    iov[0].iov_len = sizeof(buf1);
    iov[1].iov_base = buf2;
    iov[1].iov_len = sizeof(buf2);

    // 使用 readv 读取数据
    ssize_t bytes_read = readv(fd, iov, 2);
    if (bytes_read == -1) {
        perror("readv");
        close(fd);
        return 1;
    }

    // 输出读取的数据
    std::cout << "Read " << bytes_read << " bytes\n";
    std::cout << "Buffer 1: " << std::string(buf1, bytes_read > 10 ? 10 : bytes_read) << "\n";
    std::cout << "Buffer 2: " << std::string(buf2, bytes_read > 10 ? bytes_read - 10 : 0) << "\n";

    // 关闭文件
    close(fd);
    return 0;
}
```

---

### 复用IO - epoll



#### 基本函数

```cpp
#include <sys/epoll.h>
#include <unistd.h>

//创建函数
//size 参数是一个建议的 epoll 实例容量，通常这个参数在现代 Linux 版本中可以设置为 1（通过 epoll_create1）。
//该函数返回一个 epoll 文件描述符。
int epoll_create(int size);
//flags 一般设置为 EPOLL_CLOEXEC，即在 exec 调用时关闭文件描述符。
int epoll_create1(int flags);

//注册事件
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
epoll_event event;
event.events = EPOLLIN | EPOLLONESHOT;
event.data.fd = fd;
epoll_ctl(epollFd, EPOLL_CTL_ADD, fd, &event);
/*
    epfd 是通过 epoll_create 或 epoll_create1 创建的 epoll 实例文件描述符。
    op 是操作类型，通常有：
        EPOLL_CTL_ADD: 将新的文件描述符添加到 epoll 监控列表。
        EPOLL_CTL_MOD: 修改已有的文件描述符的监控事件。
        EPOLL_CTL_DEL: 从 epoll 中删除文件描述符。
    fd 是要监控的文件描述符。
    event 是一个 epoll_event 结构体，指定你希望监控的事件。
*/

//等待事件
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
/*
	return：events中包含的时间个数
    epfd 是通过 epoll_create 或 epoll_create1 创建的 epoll 实例文件描述符。
    events 是一个指向 epoll_event 数组的指针，存储返回的事件。
    maxevents 是 events 数组的大小，即最大可以返回多少个事件。
    timeout 是等待的时间（毫秒），-1 表示无限等待，0 表示非阻塞模式
*/

struct epoll_event {
    uint32_t events;    // 事件类型，可读可写等
    epoll_data_t data;  // 可以保存一个文件描述符或自定义数据
};
/*
    EPOLLIN: 有数据可读。
    EPOLLOUT: 可以写数据。
    EPOLLERR: 出现错误。
    EPOLLHUP: 连接关闭。
    EPOLLET: 边缘触发模式（Edge-Triggered）。
    EPOLLONESHOT: 表示事件触发后，只会触发一次，之后该文件描述符会自动从 epoll 实例中移除，
    			  直到你再次显式地将其添加到 epoll 中。
    			  避免重复处理：如果你只关心某个文件描述符上的某个事件一次，使用 EPOLLONESHOT 可以防止重复触发处理。
	EPOLLRDHUP: 对面关闭了写端
*/


```

















---

## 中间件

### 线程池

- 管理和重用多个线程，中间件的一种
- 使用线程池可以减少线程的销毁，而且如果不使用线程池的话，来一个客户端就创建一个线程。比如有1000，这样线程的创建、线程之间的调度也会耗费很多的系统资源，所以采用线程池使程序的效率更高。 线程池就是项目启动的时候，就先把线程池准备好。
- **一般线程池的实现是通过生产者消费者模型来的**







### 连接池













