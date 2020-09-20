# 1、Nginx基础

## 1-1 概念

Nginx 是一个高性能的 HTTP 和反向代理 web 服务器。

## 1-2 高性能的 WEB 服务器（核心1）

- 高性能的静态 WEB 服务器（不能处理动态资源）

## 1-3 反向代理（核心2）

- 正向代理：隐藏了真实的请求客户端，服务端不知道真实的客户端是谁，客户端请求的服务都被代理服务器代替来请求。
- 反向代理：客户不知道真正提供服务人的是谁，反向代理隐藏了真实的服务端。

客户端发送 http 请求，nginx 接收到请求之后把静态资源返回给客户端，可以把动态的资源继续转发给应用程序服务器；应用程序服务器返回响应给 nginx，然后动态资源和静态资源就都返回给客户端了。

![](./media/1.png)

负载均衡就是通过反向代理实现。

## 1-4 Nginx 相较于 Apache 的优势

| Apache               | Nginx                |
| -------------------- | -------------------- |
| 一个进程处理一个请求 | 一个进程处理多个请求 |
| 阻塞式的             | 非阻塞式的           |

## 1-5 Nginx 的主要的应用场景

- 静态资源服务
  - 通过本地文件系统提供服务
- 反向代理服务
  - Nginx 的强大性能
  - 缓存 
  - 负载均衡
- API 服务
  - OpenResty

<img src="./media/2.png" style="zoom:90%;" />



## 1-6 Nginx 的优势

- 高并发、高性能（一个进程处理多个请求）
- 扩展性好（自身使用模块化设计）
- 异步非阻塞的事件驱动模型
- 高可靠性
- BST 许可（允许二次开发）

## 1-7 安装第一个 rpm 的 Nginx

rpm 在生产环境可能用的比较少，因为它是 yum 源提供，说明是个比较通用的版本，如果想要定制我们的需求，那么我们就可能需要编译安装我们的 nginx。

- **rpm 命令**：rpm命令是RPM软件包的管理工具。rpm原本是Red Hat Linux发行版专门用来管理Linux各项套件的程序，由于它遵循GPL规则且功能强大方便，因而广受欢迎。逐渐受到其他发行版的采用。RPM套件管理方式的出现，让Linux易于安装，升级，间接提升了Linux的适用度。
- **epel 源**：EPEL (Extra Packages for Enterprise Linux)是基于Fedora的一个项目，为“红帽系”的操作系统提供额外的软件包，适用于RHEL、CentOS和Scientific Linux.

### 安装 nginx

```shell
# 首先安装一波 epel 源
yum install epel-release

# 可以看看软件是不是包含了 nginx 
yum list all | grep nginx

# 然后用 yum 源下载 nginx
yum install nginx

# 然后就可以看看系统中跟 nginx 有关的文件
rpm -ql nginx

# 可以看到 /var/log/nginx 其实放的就是我们的 nginx，但其实它下面是空的，程序起来了之后它会创建两个文件，一个是访问日志，一个是错误日志
# /etc/nginx/nginx.conf
# /usr/sbin/nginx -h 可以知道 nginx 用法
```

### 运行

```shell
# 可以这样运行，-c 参数是指定配置文件
/usr/sbin/nginx -c /etc/nginx/nginx.conf
```

