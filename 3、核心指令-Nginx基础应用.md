# 3、核心指令-Nginx基础应用

## 3-1 配置文件 main 段核心参数用法

### 核心参数

- **user**：指定运行 nginx 的 worker 子进程的**属主**和**属组**，其中属组可以不指定
  - 形式：user USERNAME [GROUP] ——> 中括号括起来说明是可选
  - 示例：user nginx nginx;
- **pid**：指定运行 nginx 的 **master 主进程的 pid** 文件存放路径
  - 形式：pid DIR
  - 示例：pid /opt/nginx/logs/nginx.pid;
- **worker_rlimit_nofile**：指定 worker 子进程可以打开的最大文件句柄数
  - 形式：worker_rlimit_nofile number
  - 示例：worker_rlimit_nofile 20480;
- **worker_rlimit_core**：指定  worker 子进程异常终止后的 core 文件，用于记录分析问题
  - 形式：worker_rlimit_core size
  - 示例：
    - worker_rlimit_core 50M;
    -  working_directory /opt/nginx/tmp;    # core 文件的存放目录得指定（**nginx 用户必须有写权限！**）
- **worker_processes**：指定 nginx 启动的 worker 子进程数量
  - 形式：worker_processes number | auto
  - 示例：
    - worker_processes 4;
    - worker_processes auto;
- **worker_cpu_affinty**：将每个 **worker 子进程**与我们的 **CPU 物理核心**绑定
  - 形式：worker_cpu_affinity cpumask1 cpumask2 ...
  - 示例：
    - worker_cpu_affinity 0001 0010 0100 1000;    # 4 个物理核心，4 个 worker 子进程
    - worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;    # 8 个物理核心，8 个 worker 子进程
    - worker_cpu_affinity 01 10 01 10;    # 2 个物理核心，4 个子进程
  - **注意：**将每个 worker 子进程与特定 CPU 物理核心绑定，优势在于：避免同一个 worker 子进程在不同的 CPU 核心上切换，缓存失效，降低性能；**其并不能真正的避免进程切换**。
- **worker_priority**：指定 worker 子进程的 nice 值，以调整运行 nginx 的优先级，通常设定为负值，以优先调用 nginx。
  - 形式：worker_priority number
  - 示例：worker_priority -10;
  - **注意**：Linux 默认进程的优先级值是 120，**值越小越优先**；nice 设定范围为 -20 到 +19.
- **worker_shutdown_timeout**：指定 worker 子进程优雅退出时的超时时间
  - 形式：worker_shutdown_timeout time
  - 示例：worker_shutdown_timeout 5s; 
- **timer_resolution**：worker 子进程内部使用的计时器精度，调整时间间隔越大，系统调用越少，有利于性能提升；反之，系统调用越多，性能下降。这具体是啥意思呢？看看下面的**客户端请求处理流程**
  - 形式：timer_resolution time
  - 示例：timer_resolution 100ms;

**客户端请求处理流程**

<img src="./media/8.png" style="zoom: 67%;" />



- **daemon**：设定 nginx 的运行方式，前台还是后台，前台用户调试，后台用于生产
  - 形式：daemon on | off
  - 示例：daemon off

`/opt/nginx/conf/nginx.conf`

```shell

```



## 3-2 配置文件 events 段核心参数用法



## 3-3 server_name 指令用法



## 3-4 root 和 alias 的区别



## 3-5 location 的基础用法



## 3-6 location 指令中匹配规则的优先级



## 3-7 location 中 URL 结尾的反斜线



## 3-8 stub_status 模块用法



