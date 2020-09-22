# 4、HTTP 核心模块-Nginx应用进阶

## 4-1 connection 和 request

### 基本含义

- **connection**：连接，就是常说的 tcp 连接，三次握手，状态机
- **request**：请求，例如 http 请求，无状态的协议
- **request** 是 必须建立在 **connection** 之上

## 4-2 对 connection 做限制的 limig_conn 模块

### limig_conn 模块基本功能

- 用于限制客户端并发连接数
- 默认编译进 nginx，通过 --without-http_limit_conn_module 来禁用。
- 使用共享内存（也就是在这个内存中保存指定客户端的连接数），对所有 worker 子进程生效。

### 常用指令

- limit_conn_zone：定义共享内存
  - 语法：**limit_conn_zone key zone=name:size**   这个 **key** 是用来标识客户端的唯一性的。通常这个 key 我们会使用一个变量叫做 remote_addr，这个变量是 nginx 内置的。也就是说当我们部署好 nginx 之后，客户端发来一个请求，你的 nginx 会将客户端 ip 赋值给  remote_addr，ip 就能当作一个唯一性标识。**zone** 后面跟的 name 是一个名称（随便取啥都可以）。**size** 是定义共享内存的空间。
  - 默认值：无
  - 上下文：http
  - 示例：**limit_conn_zone $binary_remote_addr zone=addr:10m**    binary_remote_addr 和 remote_addr 差不多是一个东西，都保存客户端的 ip。不同之处在于 binary_remote_addr 只使用 4 个字节的空间，而 remote_addr 会使用 7~15 个字节的空间。
- limit_conn_status：定义限制发生的时候返回给客户端的状态
  - 语法：**limit_conn_status code**    
  - 默认值：**limit_conn_status 503**
  - 上下文：http、server、location
- limit_conn_log_level：定义限制发生的时候在日志中记录的日志信息等级
  - 语法：**limit_conn_log_level info | notice | warn | error**
  - 默认值：**limit_conn_log_level error;**
  - 上下文：http、server、location
- limit_conn：定义限制客户端的并发连接数
  - 语法：**limit_conn zone number;**    zone 就是上面定义共享内存的时候定义的 zone 的名字
  - 默认值：无
  - 上下文：http、server、location

`/opt/nginx/conf/nginx.conf`

```shell
http {
	limit_conn_zone $binary_remote_addr zone=limit_addr:10m; # 定义共享内存空间
	server {
		listen 80;
		server_name localhost;
		
		location / {
			root html;
			index index.html index.htm;
			# 对系统的首页进行限制连接数
			limit_conn_status 503;
			limit_conn_log_level warn;
			limit_conn limit_addr 2; # 定义并发连接的限制是 2
			limit_rate 50; # 响应给客户端的时候速率是 50 字节/秒，这样有利于显示
		}
	}
}
```



## 4-3 对 request 处理速率做限制的 limit_req 模块

### limit_req 模块基本功能

- 用于限制客户端处理请求的平均速率
- 默认编译进 nginx，通过 --without-http_limit_req_module 来禁用
- 使用共享内存，对所有 worker 子进程生效
- 使用的**限流算法**：**[leaky_bucket](https://blog.51cto.com/leyew/860302)**   这个算法主要的作用就是把突发的流量变得平稳

### 常用指令

- limit_req_zone：定义共享内存，也可以限定请求

  - 语法：**limit_req_zone key zone=name:size rate=rate;**    rate 就是限定请求的速率
  - 默认值：无
  - 上下文：http
  - 示例：**limit_req_zone $binary_remote_addr zone=one:10m rate=2r/m;**    2r/m 表示每分钟处理两个请求，而且这两个请求也是平均分布的，比如 30 秒之后处理一个请求，再过 30 秒再处理一个请求。

- limit_req_status：限制触发时返回的状态码

  - 语法：**limit_req_status code;**    
  - 默认：**limit_req_status 503;**
  - 上下文：http、server、location;

- limit_req_log_level：限制触发时，写在日志的日志等级

  - 语法：**limit_req_log_level info | notice | warn | error**;
  - 默认：**limit_req_log_level error;**
  - 上下文：http、server、location;

- limit_req：此指令用于设置共享的内存**zone**和最大的突发请求大小，若请求速率超过了**limit_req_zone**中指定的**rate**但小于**limit_req** 中的**burst**，则进行延迟处理，若再超过**burst**，就可以通过设置nodelay对其进行丢弃处理。

  - 语法：**limit_req zone=name [ burst=number ] [ nodelay | delay=number ];**   
  - 默认：无
  - 上下文：http、server、location
  - 示例：
    - limit_req zone=one;
    - limit_conn zone=one burst=5 nodelay;

  ```shell
  http {
  	limit_req_zone $binary_remote_addr zone=limit_req:15m rate=2r/m;
  	server {
  		listen 80;
  		server_name localhost;
  		
  		location / {
  			root html;
  			index index.html index.htm;
  			# 限定请求数
  			limit_req_status 504;
  			limit_req_log_level notice;
  			limit_req zone=limit_Req;
  			# limit_req zone=limit_req burst=7 nodelay; 
  		}
  	}
  }
  ```

  

## 4-4 限制特定 IP 或网段访问的 access 模块



## 4-5 限制特定用户访问的 auth_basic 模块



## 4-6 基于 HTTP 响应状态码做权限控制的 auth_request 模块



## 4-7 rewrite 模块中的 return 指令



## 4-8 rewrite 模块中的 rewrite 指令



## 4-9 return 和 rewrite 指令执行顺序



## 4-10 rewrite 模块中if指令



## 4-11 autoindex 模块用法



## 4-12 Nginx 变量的分类



## 4-13 TCP 连接相关变量



## 4-14 发送 HTTP 请求变量



## 4-15 处理 HTTP 请求变量



