# Nginx

​					

### 什么是Nginx

***

Nginx (engine x) 是一个高性能的HTTP和反向代理web服务器，特点是占用内存少，并发能力强。

**中文文档**：https://www.nginx.cn/doc/

***

​					

​				

​				

### 几个常见的概念	

***

#### 一、正向代理

如果把局域网外的Internet想象成一个巨大的资源库，则局域网中的客户端要访问Internet需要通过代理服务器来访问，这种代理服务就称为正向代理。正向代理浏览器要主动配置代理服务器，而反向代理不需要。正向代理是客户端可知的，在客户端上配置。反向代理是客户端不可知的。中间商赚差价。

![](https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201213165205.png)

​				

#### 二、反向代理

客户端对代理是无感知的，因为客户端不需要任何配置就可以访问，我们只需要将请求发送到反向代理服务器，有反向代理服务器去选择目标服务器获取数据后，再返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真实服务器的IP地址

<img src="https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201213165336.png" style="zoom:60%;" />

​			

#### 三、负载均衡

单个服务器解决不了，通过增加服务器的数量，将请求分发到各个服务器上，将原先请求几种到单个服务器上的情况改为请求分发到多个服务器上，将负载分发到不同的服务器，也就是负载均衡 

<img src="https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201213165429.png" style="zoom: 80%;" />

​			

#### 四、动静分离

为了加快网页的解析速度，可以把动态页面和静态页面由不同的服务器来解析，加快解析速度，降低原来单个服务器的压力 

<img src="https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201213165719.png" style="zoom:67%;" />

<img src="https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201213165748.png" style="zoom:80%;" />

***

​			

​			

### Nginx配置文件

****

#### nginx配置文件主要由三部分组成

##### 一、全局块

> 从配置文件开始到events块之间的内容就是全局块，全局块作用与全局，全局都能生效。主要会设置一些影响nginx服务器整体运行的配置指令，包括排至运行nginx服务器的用户组、一共多少个工作进程（work process）、进行PID存放路径、日志存放路径、配置文件的引入等。

##### 二、events块

> events块涉及的指令主要影响nginx服务器与用户的网络连接，每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。
>
> 例如：worker_connections 1024；这个配置就是每个work process工作进程的最大连接数

##### 三、http块

> nginx服务器配置中最频繁的部分
>
> http块包含：http全局块，server块，server块中还有location块。负载均衡还有upstream块
>
> * **http全局块**
>
>   其实就是http开始到server块之间的内容。包括配置代理、缓存、文件引入、MIME-TYPE定义、日志自定义、连接超时时间、单链接请求数上线等。可以嵌套多个server。
>
>   <img src="https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201213172253.png" style="zoom:50%;" />
>
>   ​			
>
> * **server块**
>
>   这块和虚拟主机有密切关系，配置虚拟机主机的相关参数。可以把它理解为tomcat运行的多个web项目吧。一个http块中可以有多个server
>
>   <img src="https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201213173105.png" style="zoom:90%;" />
>
>   ​	
>
> * **location块**
>
>   配置请求的路由，以及各种页面的处理情况。

​					

##### 配置文件结构总览

> <img src="https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201213213613.png" style="zoom:50%;" />

​					

##### 块之间的包含关系

> * main就是全局块
> * server继承main，location继承server，upstream即不会继承其他设置也不会被继承。
>
> <img src="https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201213214348.png" style="zoom: 67%;" />

***

​			

​							

### Nginx操作的常用命令

***

**nginx -v**：查看nginx的版本号

**nginx -t**：检测配置文件是否正常

**nginx**：启动nginx

**nginx -s quit**：优雅关闭nginx，不再接收新的请求，把已接收的请求处理完毕后关闭。

**nginx -s stop**：关闭nginx，直接关闭。

**nginx -s reload**：重新加载nginx。修改配置文件的时候需要重新加载，并不会重新启动nginx而是加载配置文件，类似source /etc/profile

****

​				

​				

### Nginx配置实例

***

#### 部署静态web服务器

> * 访问192.168.203.137:9000地址跳转到自己内置的静态页面
>
>   ![](https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201214212139.png)
>
> **注意**：
>
> * 上结合root和index的意思就是，去/usr/local/nginx/proj目录下找index.html文件；
>
> * 以上结合root和index的意思就是，去/usr/local/nginx/proj目录下找index.html文件；
>
> * 静态路径寻找：/{root}/{location拦截的路径}

​							

	#### 负载均衡

> 启动两个web服务器，分别运行在10000和20000端口
>
> * **轮询**
>
> <img src="https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201214230538.png" style="zoom:80%;" />
>
> * **设置权重**：权重越高，则会把更多的请求转发到响应的服务器
>
>   <img src="https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201214230929.png" style="zoom:90%;" />
>
>   ​			
>
> * **最小连接数**
>
>   ![](https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201214234429.png)
>
> * **IP哈希**：
>
>   根据用户的ip地址使用哈希算法模服务器数量的值转发，也就是说如果用户的ip地址不变，则请求永远转发到同一台机器上。
>
>   以上三种方案都会产生session问题，只有IP哈希能够解决session问题。但是如果用户地址变更了，也会产生session问题
>
>   ![](https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201214234039.png)
>
> * **upstream的其他配置**
>
>   ![](https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201215000319.png)
>
>   **backup**：备用服务，当其他所有正常配置的服务实例都down掉后，才会转发给备用服务器。如果其他服务实例正常，则备用服务器不会被使用到。
>
>   **down**：不管服务是否可用，nginx都不会转发给它

​					

#### 动静分离

> **架构图**
>
> 整个架构中，一个nginx负责负载均衡，两个nginx负责静态资源代理，另外两个tomcat则处理动态请求。
>
> <img src="https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201215002714.png" style="zoom:67%;" />
>
> 
>
> **配置文件**
>
> 拦截请求路径带有.css、.js、.jpg等的uri，路径匹配成功到root指定的目录位置寻找资源，再拼接上uri中的资源目录找到文件。 
>
> <img src="https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201215155954.png" style="zoom: 80%;" />
>
> **例如**：
>
> 当我们访问：http://192.168.203/img/1.jpg 的地址时，就会去 /usr/local/source + /img 的目录位置寻找1.jpg文件
>
> ![](https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201215160018.png)

   #### 反向代理1

> * 访问192.168.203.137:81地址后跳转到192.168.203.137:8080
>
> ![](https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201213205935.png)

​							

#### 反向代理2

> 监听9001端口
>
> * 访问127.0.0.1:9001/bd时，跳转到www.baidu.com;
> * 访问127.0.0.1:9001/qq时，跳转到www.qq.com;
>
> ![](https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201213211956.png)

***

​			

​								

### 虚拟主机

***

虚拟主机，就是把一台物理服务器划分成多个“虚拟”的服务器，这样我们的一台物理服务器就可以当作多个服务器来使用，从而可以配置多个网站。

Nginx提供虚拟主机的功能，就是为了让我们不需要安装多个Nginx，就可以运行多个域名不同的网站。

Nginx下，一个server块就是一个虚拟主机。nginx的虚拟主机是通过nginx配置文件中的server节点指定的，想要设置多个虚拟主机，配置多个server节点即可。

例如：www.58.com 。切换城市，可以看到不同的城市地址的二级域名。

如一个公司有多个二级域名，没必要为每个二级域名都提供一台服务器，使用nginx的虚拟主机技术就可以转发到不同二级域名的服务上了。只要备案一下二级域名就好了，例如：58.com网站的二级域名：m.58.com、bj.58.com

****

​					

​						

### 指令说明

***

#### location指令说明

location指令用于匹配URI

```java
location [ = | ~ | ~* | ^~ ] uri {
    
}
```

* =：  用于不含正则表达式的uri前，要求请求字符串与uri严格匹配，如果匹配成功，就停止继续向下搜索并立即处理该请求；

* ~：  用于包含正则表达式的uri前，并且区分大小写；

* ~*：用于包含正则表达式的uri前，不区分大小写；

* ^~：用于不包含正则表达式的uri前，要求nginx服务器找到标识uri和请求字符串匹配度最高的location后，立即使用此location处理请求，而不再使用location块中的正则uri和请求字符串匹配。

注意：如果uri中包含正则表达式，则必须要有 ~ 或者 ~* 标识。

***







