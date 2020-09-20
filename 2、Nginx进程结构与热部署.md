# 2、Nginx进程结构与热部署

## 2-1 Nginx 的进程结构

- nginx 采用的是多进程的一种进程结构，这是为了**高可用性和高可靠性**。

<img src="./media/3.png"/>

### Master Process

- Master Process 并不真正地处理用户的请求，真正处理请求的是下方的 Worker Process，Master Process 进程只是监控它下面的子进程，看看它们有没有挂掉啥的，如果挂掉了，它会新开一个子进程来维持当前的结构。还有一种情况就是配置文件修改了之后，Master Process 也会去通知它的子进程（其实就是热部署）。
- 我们知道 nginx 是基于模块化的，自己设计模块的时候尽量少动 Master Process，因为如果 Master Process 挂了的话就会影响到很多东西。

### Worker Process

- 是真正处理请求的子进程。
- Worker Process 在实际的生产环境中也不一定是 4 个。

### CM 和 CL

- CM 和 CL 也是两个子进程，一看就是管理缓存的两个子进程。
- nginx 有一个最主要的应用场景：**反向代理**。也就是说，它会把动态请求代理给后端应用程序服务器，应用程序接受到请求了然后处理计算之后，把内容动态地返回给 nginx 的过程中，可以进行缓存；真正提供缓存功能的不是 CM 和 CL，是 Worker Process（也就是说决定缓存哪些内容），Cache Loader 用来加载缓存，Cache Manager 用来管理缓存

**所有的子进程之间的通信都是通过共享内存的方式来进行通信的。**

## 2-2 Linux 的信号量管理机制

- 在 Linux 中，是通过信号量来进行进程管理的。比如我们经常使用的 **kill** 命令，这个命令可以发送很多的信号量。Linux 中的信号量如下所示：

<img src="./media/4.png"/>

- 我们日常中使用 **kill $PID** 去杀死一个进程，实际上就是发送了 **SIGTERM** 这个信号给 Linux ，Linux 收到了这个信号之后就知道要关闭这个进程。
- 常用的信号量：

| 信号量   | 命令格式      | 作用                                                         |
| -------- | ------------- | ------------------------------------------------------------ |
| SIGCHLD  | kill -17 $PID | 在一个进程终止或者停止时，将 SIGCHLD 信号发送给其父进程，按系统默认将忽略此信号，如果[父进程](https://baike.baidu.com/item/父进程/614062)希望被告知其子系统的这种状态，则应捕捉此信号。 |
| SIGQUIT  | kill -3 $PID  | 程序终止信号，在用户键入 QUIT 字符时发出，用于通知前台进程组终止进程。进程在因收到 SIGQUIT 退出时会产生 core 文件, 在这个意义上类似于一个程序错误信号。 |
| SIGTERM  | kill -15 $PID | 程序结束(terminate)信号, 与 SIGKILL 不同的是该信号可以被阻塞和处理。通常用来要求程序自己正常退出，shell命令kill缺省产生这个信号。如果进程终止不了，我们才会尝试 SIGKILL。 |
| SIGKILL  | kill -9 $PID  | SIGKILL是发送给一个[进程](https://baike.baidu.com/item/进程/382503)来导致它立即终止的信号 |
| SIGHUP   | kill -1 $PID  | 它会迫使进程重新读取配置文件                                 |
| SIGUSR1  | kill -10 $PID | Linux 提供的自定义的信号量                                   |
| SIGUSR2  | kill -12 $PID | Linux 提供的自定义的信号量                                   |
| SIGWINCH | kill -28 $PID | Linux 提供的自定义的信号量                                   |

## 2-3 利用信号量管理 Nginx

### 使用信号量来管理 master 和 worker

- 可以直接给 Master 进程发送信号量，来让它进行管理。
  - 命令形式：**kill -s 信号量 进程号**
- 可以给 Worker 进程发送信号量，但是本身并不推荐直接给它发信号量来对它进行管理，因为它的管理是由 Master 来的，通常的做法是给 Master 进程发送信号量，然后让 Master 进程去管理 Worker 进程。
  - 命令形式：**kill -s 信号量 进程号**
- 可以直接通过 nginx 的二进制程序（ /opt/nginx/sbin/nginx ）加些参数来对 master 发送信号量。
  - 命令形式：**/opt/nginx/sbin/nginx -s 参数**

**Master 进程接收的信号量：**

- 监控 worker 进程就收信号
  - CHLD（子进程挂掉会发这个信号量给 Master 进程）
- 管理 worker 进程接收信号
  - TERM，INT
  - QUIT
  - HUP
  - USR1
  - USR2
  - WINCH

**worker 进程接收的信号量（通常不建议发送这样的信号量）：**

- TERM，INT
- QUIT
- USR1
- WINCH

**命令行（ nginx 的二进制程序（ /opt/nginx/sbin/nginx ）加些参数，参数如下。执行这些命令本质上就是给 master 进程发送信号量）**

- reload：HUP
- reopen：USR1
- stop：TERM
- quit：QUIT

### 信号量管理例子

- **kill -s SIGTERM 26734**：杀死 Master 进程（Master 进程 PID 为 26734）
- **kill -s SIGHUP 26734**：让 Master 进程重新读配置文件，会重启 worker 进程

用命令行管理（给 master 直接发送信号量的效果一样）

- **/opt/nginx/sbin/nginx -s stop**：让主进程退出

## 2-4 配置文件重载的原理真相

给 Master Process 发送 HUP 信号（或者二进制文件加 reload）能使配置文件重载，这能给 nginx 进行平滑升级。那么它的背后到底发生了一个什么事情呢？

### reload 重载配置文件的流程

1. 向 master 进程发送 HUP 信号（reload 命令）
2. 