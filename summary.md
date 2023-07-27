# 第一、二章

### 服务器套接字模板

```python
1. socket() 创建套接字
2. bind() 绑定IP以及port号
3. listen() 开始监听
4. accept() 有客户端请求的时候控制接受
5. close() 关闭套接字
```

### 客户端套接字模板

```python
1. socket() 创建套接字
2. connect() 与服务器请求连接
3. close() 关闭套接字
```

----------------------

```python
gcc c文件 -o 执行文件
# 例如： gcc op_serv.c -o opserv
可以自己写一下makefile

文件描述符
0 标准输入
1 标准输出
2 标准错误
# 默认的，所以自己新建文件描述的时候只能从3开始表示

# 以_t为后缀的数据类型都是属于元数据类型

# Linux文件操作
open(fd, "打开方式")
read(fd, buf, sizeof(buf))
write(fd, buf, sizeof(buf))
close(fd)

socket在linux中操作的时候与文件操作一样，而Windows系统中并不是
```

# 第三、四章

```python
# 服务器端使用描述符的状态
1. write(clnt_sock, buf, str_len)
2. read(clnt_sock, buf, sizeof(buf))
3. bind(serv_sock, serv_addr, ...)
4. accept(serv_sock, clnt_sock, ...)

# 客户端使用描述符的状态
1. write(sock, buf, str_len)
2. read(sock, buf, sizeof(buf))
4. connect(clnt_sock, serv_addr, ...)

# TCP客户端套接字在调用connect的时候会自动分配IP地址和端口号
```

```python
# 下面两个函数作用一样都是将字符串形式的地址转换成32位整数型的类型
inet_aton()
inet_addr()
# 区别在于：前者得到的数据会直接存进in_addr结构体的s_addr中，后者则需要自己赋值给结构体
inet_ntoa() 作用与 inet_aton恰好相反

# INADDRANY的作用就是自动获取服务器的IP地址，不必亲自输入

# 链路层的作用： 负责物理连接，定义LAN，WAN，MAN等网络标准
# IP层作用: IP本身就是面向消息的不可靠协议，每次传输数据时会帮助选择路径，但是每次选择的路径可能不太一样
# 传输层： 按照IP的选择的路传输即可

```

```python
# TCP表示 SOCK_STREAM
# UDP表示 SOCK_DGRAM

# 面向连接的套接字  --- SOCK_STREAM
1. 传输不存在数据边界： 即 write 调用的次数和 read 调用的次数可以不一样
（一次发送的消息可以分成多次接收）
2. 保证可靠的传输

# 面向消息的套接字  --- SOCK_DGRAM
1. 无连接
2. 不保证可靠传输
3. 存在数据边界

```

```python
# 回声服务器客户端中客户端会出现的问题

write()
read()

1. 两个函数调用太紧凑，万一写过来的数据包分为多次传输，则调用read函数不能一次性得到所有传输过来的数据

# 解决办法

str_len = write(sock, message, strlen(message));

recv_len = 0;
while(recv_len < str_len)
{
    recv_cnt = read(sock, &message[recv_len], BUF_SIZE-1);
    recv_len += recv_cnt;   
}
message[recv_len] = 0;
# 至于read到的数据不够传输的长度，则会一直调用read（适用于TCP----没有传输边界）
```

###### 每个套接字有属于自己的I/O缓冲区，包含（输出流，输入流）

```python
# I/O的特性
1. 关闭套接字也会继续传递输出缓冲中余留的数据
2. 关闭套接字将会丢失输入缓冲中的数据
```

###### TCP连接三次握手

###### TCP断开四次握手

-----------------------------------

# 第五、六章

```python
# 基于UDP的回声服务器

1. 使用sendto / recvfrom 函数
2. 不需要listen()以及accept()

serv_sock = socket()
bind()

recvfrom() / sendto()
close()

# 基于UDP的回声客户端

sock = socket()
sendto(sock, message, strlen(message), 0, &serv_addr, sizeof(serv_addr))
str_len = recvfrom(sock, message, BUF_SIZE, 0, &from_addr, sizeof(from_addr))

# 调用sendto函数时发现没有分配IP地址，会在首次调用sendto函数是给相应套接字自动分配IP地址以及端口号

# sendto函数
1. 第一阶段：向UDP套接字注册目标IP和端口号
2. 第二阶段：传输数据
3. 第三阶段：删除UDP套接字中注册的目标地址信息
这种没有注册目标地址信息的套接字称为未连接的套接字
# 创建连接的UDP套接字
只需要加一步 connect(sock, &addr, sizeof(addr))

# 调用几次 sendto() 函数就需要几次 recvfrom() 函数接收

```

# 第七章

```python
# 优雅断开

shutdown(int sock, int howto)

1. SHUT_RD ---- 关闭输入流
2. SHUT_WR ---- 关闭输出流
3. SHUT_RDWR ---- 同时断开I/O流
```

# 第十章

```python
# 僵尸进程

# 僵尸进程产生的原理 “子进程结束时会把返回值传给操作系统，但是操作系统不会销毁子进程，直到值传递给产生子进程的父进程” 而在这个过程中进程就是僵尸进程的状态

# 解决办法： 产生子进程的父进程主动向操作系统要子进程结束的返回值

1. wait()函数
    ----- 搭配WIFEXITED, WEXITSTATUS
    ----- wait函数会阻塞程序的运行，慎用
2. waitpid()函数
    ----- 使用此函数，程序不会阻塞
    ----- 

# 信号处理解决僵尸进程
 
1. signal()函数
       ---- SIGALRM，配合alarm函数使用
       ---- SIGINT，输入CTRL+C的时候，操作系统给出的反应
       ---- SIGCHLD，子进程终止的时候触发
     --- signal(SIGINT, "会执行的响应函数")

2. sigaction()函数
       ---- 对应一个叫sigaction的结构体
   ## struct sigaction act;
   ## act.sa_handler = "会执行的响应函数"
   ## sigemptyset(&act.sa_mask)
   ## act.sa_flags = 0
   ## sigaction(SIGALRM, &act, 0)
       ---- 使用此函数相应流程多一些
    
# 套接字属于操作系统，进程只是拥有代表相应套接字的文件描述符 #

# 并发进程服务器
    ----- 使用fork时，子进程会复制父进程创建的文件描述符
    ----- 所以需要关闭父进程的客户端套接字，关闭子进程的服务器套接字

```

# 第十一章

```python
# 进程间的通信

1. 通过操作系统作为媒介
pipe() --- 进程间通信的管道  ---- IPC机制

2. int fds[2];
   pipe(fds);
   # 其中fds[0]是指出口描述符
   # fds[1]是指入口描述符
    
3. 管道还可以双向通信

进程之间交换数据必须依赖操作系统的pipe
```

###### 多进程程序需要开销大量的存储空间

# 第十二章

```python
# I / O 复用

select函数检测、控制多个套接字描述符
# 无论连接的客户端有多少个，始终只有一个进程提供服务
1. 是否有套接字接收数据？
2. 是否有阻塞的套接字？
3. 是否有套接字发生异常？

# 在select监视范围中的套接字会让程序对此套接字做出响应
1. FD_ZERO(&reads)
2. FD_SET(serv_sock, &reads)  // 把要监视的sock在reads数组中置1

# select调用一次监视多个文件描述符，实现高并发的处理能力

```

# 第十四章

```python

# 多播与广播

# 多播: 可以在不同的网络进行“广播”
   ---- 主要依赖于路由器复制包的功能
   ---- 基于UDP完成
   ---- 加入特定组就可以接收多播组的消息
    
    1. 接收者套接字需要设置加入多播组
      struct ip_mreq join_adr
    2. 修改 setsockopt() 里面的选项
   
# 广播：只在同一网段的所有主机进行数据包传输
  
```

# 十七章

###### 条件触发与边缘触发的区别

```python
# 条件触发

1. 数据区有数据的时候以及数据还没有取完的时候，不停的向epoll作报告

# 边缘触发

1. 数据区有数据的时候报告一次，后面不在报告
```



# 第十八章

```python
# 线程的实现 ##########################################################

1. 线程会共享保存全局变量的“数据区”，以及负责动态分配的堆区
      故线程只会独自享有栈区域
    
2. 上下文切换的时候不需要切换数据区以及堆区

3. 可以利用数据区和堆交换数据


# 重要的函数 ##########################################################

1. pthread_create(线程的id,NULL,"线程main函数执行函数",传过去的参数)
2. pthread_join(线程的id,"保存线程main函数的返回值")



# 线程存在的问题 #######################################################

1. 临界区
 
    多个线程需要同时访问的代码区域

2. 线程同步问题

    创建锁、销毁锁

    # 互斥量
    lock()
    unlock()
    
    # 信号量
    sem_wait()
    sem_post()
    
3. 销毁线程的函数

   pthread_detach(线程的id)
   pthread_join()
   前者不会阻塞调用此函数的程序，后者会阻塞调用的程序
```