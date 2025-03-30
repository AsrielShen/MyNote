# Linux文件系统

## 系统调用

read(),write(),open(),close()等

## 文件IO 不带缓存

- 包含在#include<unistd.h> 和 <fcntl.h> 如open read write lseek close

- 对于内核而言，所有打开的文件都通过文件描述符（一个非负整数）引用。当打开或创建文件时，内核向进程返回一个文件描述符

- STDIN_FILENO、STDOUT_FILENO、STDERR_FILENO分别为0、1、2三个文件描述符，并与标准输入输出错误相互关联
- 每个进程包含一个**打开文件描述符表**，每一个文件描述符（表的索引）和文件描述符标志、文件指针关联
- 文件指针指向文件表项，每个进程中每一个打开的文件都有一个文件表项，但每个打开的文件只有一个v节点
- 多个进程打开相同文件时，会产生多个文件表项（独立），指向同一个v节点

### `open()`

- `int open(const char* path, int oflag, .../*mode_t mode */);`
- `int openat(int dirfd, const char* path, int oflag, .../*mode_t mode */);`
- 打开文件并返回文件描述符fd，mode是创建文件时使用的参数，用mode指定文件的访问权限
- open和openat返回的文件描述符一定是最小的未用的文件描述符
- open和openat区别
  - path均为绝对路径时，没区别，自动忽略fd
  - open只能使用当前目录作为相对路径
  - openat中的dirfd指明了相对路径的开始地址，dirfd=AT_FDCWD时开始地址为工作目录
- O_APPEND标志下，每次写入的时候后会动态调整偏移量到最后，即使用lseek改变了偏移量的值，这样使得write成为原子操作

### `creat()`

- `int creat(const char* path, mode_t mode)`
- 等价为 `open(path, O_WRONLY|O_CREAT|O_TRUNC, mode);`
- creat只能只写创建，不能像open创建那么灵活
- 会清空已存在的文件，O_TRUNC的作用就是截断文件为0

### `close()`

- `int close(int fd);`
- 关闭文件时还会释放该进程在该文件上的所有记录锁
- 进程结束时，会自动释放打开的所有文件

### `lseek()`

- `off_t lseek(int fd, off_t offset, int whence);`
- 作用：设置文件偏移量
- 每个文件打开时都会有一个相关的**当前文件偏移量**，通常是一个非负整数，表示从文件开始到现在位置的字节数
- open时指定O_APPEND选项，会到文件末尾，否则偏移量设置为0
- lseek成功返回新的文件偏移量
- offset含义和whence有关
  - whence == SEEK_SET，将文件偏移量设置为offset
  - whence == SEEK_CUT，将文件偏移量设置为当前值+offset（offset可正可负）
  - whence == SEEK_END，将文件偏移量设置为文件长度+offset（offset可正可负）
  - 分别对应0 1 2

- 若文件是管道、FIFO、网络套接字，则会出错，返回-1
- 文件偏移量允许超过当前文件长度，并在这种情况下进行写操作，则会在文件中构成一个**空洞**
- 空洞不占用磁盘空间，但是算作文件长度的一部分
- 只操作了文件表项，没有操作到v节点i节点，没有进行任何io操作

### `read()`

### `write()`

### `dup() dup2()`

### `sync() fsync() fdatasync()`

文件写入的同步函数，文件写入时通常先写入缓存区，**延迟写**入磁盘

- `void sync();` 只将所有修改过的块缓冲区排入写队列，并不等待实际写操作结束。每隔一段时间操作系统会自动调用
- `int fsync(int fd);` 只对fd指定的文件起作用，并等待写操作结束才返回
- `int fdatasync(int fd);` 只返回数据，fsync还会更新文件属性 
- 上面的read write默认使用内核缓冲区，写入也就是写入内核缓冲区，然后操作系统来完成剩余操作

### `fcntl()`

- `int fcntl(int fd, int cmd, .../*int arg*/);`
- 改变已经打开的文件的属性
- 复制文件描述符
- 获取/设置文件描述符标志
- 获取/设置文件状态标志
- 获取/设置异步IO所有权
- 获取/设置记录锁

### `ioctl()`

没懂

### /dev/fd

没懂

## 文件和目录

### 文件类型

1. 普通文件
2. 目录文件
   - 所有有读权限的进程都可以读目录
   - 但只有内核可以写文件——使用系统调用处理目录
3. 块特殊文件
4. 字符特殊文件
5. FIFO（命名管道）： 用于进程通信
6. 嵌套字：用于进程的网络通信
7. 符号链接：指向另一个文件

文件类型保存在stat的**st_mode**成员（含有文件类型和访问权限位）中，可用 **宏**来确定文件类型

### 文件的权限

- 具体详细看书
- 目录的读权限和执行权限不同，读权限允许读目录获得目录项，但执行权限允许通过这个目录（运行打开这个目录下的文件）

### 文件系统





### `stat() fstat() lstat() fstatat()`

- `int stat(const char* path, struct stat* buf);`
- stat返回路径文件对应有关的信息结构
- fstat返回fd对应的
- lstat返回符号链接文件的信息，而非符号链接引用的文件信息，其他和stat相同
- fstatat可以控制相对路径的原始路径
- 具体使用查字典

**stat结构体内容包含了**

- 文件类型
- 大小 size
- uid gid
- 时间信息
- IO阻塞信息
- 等等

### `access() faccessat()`

- 使用实际id实际组id测试访问权限
- faccesssat还通过flag参数选择实际id还是有效id

### `umask()`

- 在进程中设置文件模式创建屏蔽字，来屏蔽进程后续创建的文件的权限
- 也有umask的shell命令，可以修改shell的文件模式创建屏蔽字
- 可以用于保证其他人不会写你的文件（创建文件时自动帮我们把其他人写的权限屏蔽掉）

### `chmod()`

- 修改文件的访问权限位
- 有效id等于所有者id 或者 具有超级用户权限

### `chown()`

- 修改文件用户id组id

### `truncate()`

- 截断文件





## 标准IO库

以下均是用流实现，c++有封装好的文件流类

### `fopen()`

访问模式

- r：只读模式，如果没有文件则会报错，指针会指向**文件开头**
- w：只写模式，如果文件存在则会清空，如果不存在则创建新文件
- a：只追加写模式，如果文件存在则末尾追加写，如果不存在则创建新文件，指针指向**末尾**
-  r+：读写模式，逻辑和r一样，文件必须存在，写入是从头一个一个覆盖
- w+：读写模式，逻辑和w一样，如果文件存在则清空文件，如果不存在则创建新文件
- a+：读追加写模式，逻辑和a一样

返回内容

- 返回`FILE*`结构体指针，表示一个文件
- 报错则返回NULL

```c++
#include <stdio.h>
//filename是一个字符串地址，可以是相对也可以是绝对
//mode是访问模式，也是字符串 const char*
FILE* fp = fopen(filename, mode);
if(fd == NULL)
    printf("Error!\n");
else printf("Success!\n");
```

### `fclose()`

打开的文件一定要关闭文件，容易报错

返回内容

- 成功返回 0
- 失败返回EOF（-1），通常关闭文件失败会直接程序报错

```c++
#include<stdio.h>

//int fclose(FILE*);

int result = fclose(FILE*);
if(result == EOF)
    printf("Error!\n");
else if(result == 0)
    printf("Success!\n");
```

### `fputc()`

所有的写入和读出操作，均需要按照对应模式先打开文件

读写权限记录在fopen()的参数中

返回内容

- 成功返回对应的char
- 失败返回EOF

```c++
#include <stdio.h>
//写入一个字符
//int fputc(int , FILE); //int表示对应的ASCII的char
int result = fputc(char,FILE*)
if(result == EOF)
    printf("Error!\n");
else printf("Success!\n");
```

### `fputs()`

写入一个字符串

返回内容

- 成功返回非负整数（0 ，1）
- 失败返回EOF

```cpp
#include <stdio.h>
//写入一个字符串
//int fputs(const char*, FILE); //int表示对应的ASCII的char
int result = fputs(string, FILE*)
if(result == EOF)
    printf("Error!\n");
else printf("Success!\n");
```

### `fprintf()`

返回内容

- 成功返回写入字符的个数
- 失败返回EOF

```cpp
#include <stdio.h>
//写入一个格式化字符串 和 printf() 类似
int result = fprintf(FILE*, "%s\n",string);
if(result == EOF)
    printf("Error!\n");
else printf("Success!\n"); 
```

### `fgetc()`

- 成功返回对应的char
- 失败返回EOF

```c++
#include <stdio.h>
//读出一个字符，一个字节内容
//char fgetc(FILE);
char ch = fgetc(FILE*)
printf("%c\n",ch);

//但注意中文不是只占一个字节，汉字所占内存是不确定的
char ch = fgetc(FILE)
while(ch != EOF)
    printf("%c",ch);
printf("\n");
```

### `fgets()`

- 成功字符串
- 失败返回NULL

```c++
#include <stdio.h>
//从FILE中读出长度为n的字符串，保存到buff中
//char* fgetc(char* buff, int n, FILE*);
char* buff = malloc(sizeof(char)*10);
char* result = fgets(buff, 5, FILE*);
if(result == NULL) printf("Error\n");
else printf("%s\n",buff);
```

### `fscanf()`

读出格式化字符串

- 成功返回匹配到的参数个数
- 匹配失败返回 0
- 报错返回EOF

```c++
#include <stdio.h>
//和scanf()类似
//int fscanf(FILE,string, ...); //int表示对应的ASCII的char
int a;
int result = fscanf(FILE*, "%s %d %s", string, &a, string);
if(result == EOF) printf("Error\n");

```

### `stdin stdout stderr` 

可以当成一个文件指针

```
#include <stdio.h>
```















































