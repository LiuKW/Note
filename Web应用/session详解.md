## Session详解

### 什么是session

* 说完了cookie，下面就来说一说session。session是保存在服务端的，而cookie是保存在客户端的。

* session就是当前会话对象，什么是会话对象？一个客户端（浏览器）与服务端建立了连接，服务端就会创建一个会话对象，用于**客户端与服务端之间进行数据共享**。一个浏览器对应唯一的一个的会话对象。也就是说服务器可能存在多个session对象分别对应不同的浏览器。以下是示意图。

  ​		

  ![](../img/session/1.png)

  ​		

* 上面说到一个浏览器对应一个session。那么session是如何来区分不同的客户端呢？这里就需要借助cookie的机制。还记得在讲cookie的时候，我们在chrome调试工具中看到的**name=JSESSIONID**的cookie吗。现在就来解释它。

* 当客户端第一次与服务器创建连接的时候，服务器会创建一个session与之对应。session的数据结构和cookie类似，也是键值对（key-value）的形式。不过session的value字段可以存的类型更多，例如：String，Integer，List，Map等数据类型。而cookie只能存字符串类型。

* 新创建的session的key=sessionId，value就为一个字符串，这个字符串是无规律、唯一不重复的、且每次都不一样的。同时服务器会向浏览器发送一个key=jsessionid，value就是sessionid对应的字符串的cookie保存在客户端。此后客户端每次发送请求，都把这个cookie带上。服务器通过jsessionid和sessionid匹配，就能知道是否是同一个客户端在操作了。

  ​		

  ![](../img/session/2.png)

  ​		

  分析上面这张图。这是客户端第一次与服务器建立连接，可以看到服务器向浏览器写回了一个cookie，Name=JSESSIONID，Value=不重复的字符串。它的路径为 / ，前面cookie那篇文档也提到过，路径为 / 的时候，就代表无论访问服务器的什么资源，都会把这个cookie带回去。这样服务器每次请求解析这个cookie就能得到对应的session。
  
  ​	
  
  ​	

### 验证JSESSIONID和SESSIONID

通过上面了解到，客户都每次访问服务器都会创建一个cookie与session对应，通过这两种机制作唯一标识客户端。

```java
@GetMapping("/test")
public void JidAndSid(HttpServletRequest request)
{
 	String sessionid = request.getSession().getId(); //获取request对象获取session再获取id
    String jsessionid = null;
    Cookie[] cookies = request.getCookies();
    for (Cookie cookie : cookies) {
        if (cookie.getName().equals("JSESSIONID")) {
            jsessionid = cookie.getValue();   
            //读取传回来的cookie值，获取名字为JSESSION的cookie的值
        }
    }

    System.out.println("SESSIONID: " + sessionid);
    System.out.println("JSESSIONID: " + jsessionid);   
}
```

以下是两次访问的截图，可以看到在控制台每次打印的JSESSIONID那个cookie的值和SESSIONID的值是一样的

![](../img/session/3.png)

**小结：**服务器就是根据sessionid和JSESSION来匹配的，浏览器把带有JSESSIONID的cookie带回给服务器，服务器解析这个cookie，与sessionid匹配。

​		

​		

### session的生命周期

* session是存活在服务器，服务器每天甚至每秒要应对特别多的客户端，每个客户端都对应一个session，那么久而久之，服务器就会撑不住了。所以session肯定不会一直存在，服务器肯定会定期清理session。session从出生到销毁就称为它的生命周期。

* session从客户端第一次访问服务器的时候被创建，从此就生存在服务器中。如果客户端长时间且超过一定时长没有访问，该session就会被销毁，这个时长默认为30分钟。所以session的默认存活时间就是30分钟，可以通过修改servlet容器配置更改默认存活时间。这个就不演示了，应该比较少用。用的时候再查吧。

* 我们也可以手动配置session的存活时间，通过**session.setMaxInactiveInterval(int i)**方法设置session的存活时间，单位为秒，传值为-1表示不销毁session。注意，这里更准确的说法应该是session的活跃时间，也就是说session超过30分钟不活跃（就是没有人用它），它就会被销毁。当时如果有人用它，它的存活时间又会被重新更新。
  
* 可能说的不太清楚，举个例子：浏览器第一次访问服务器之后，服务器创建了一个session，session默认的存活时间为30分钟。然后浏览器很长一段都没有去访问过服务器了，此时过了29分钟，session都快被销毁了。就在这个时候，浏览器在29分钟的时候访问了服务器，这个session又重新被用上了，所以session的存活时间更新了30分钟。
  
* 除了等它自己消失，我们也可以手动直接销毁它，session.invalidate()；这个方法就可以让session失效。一般在用户注销退出的时候使用。

  ​	

  ​		

### session的活化和钝化

* 前面我们提到过，每个浏览器都会对应一个session，服务器中可能存在许多session，如果太多的话，内存就会爆满，然后服务器就宕机了。所以服务器肯定不会那么傻，任由session存在内存中。当服务器中存在特别多session的时候，服务器会把一部分session先放到硬盘上，把session放到硬盘的现象就称为钝化，同样的，把session从硬盘中取出来的现象就称为活化
* 什么时候钝化session？
  * 当session闲置了一段时间后，就会被钝化到硬盘。
  
  * 当session大量存在时，会把一些不常用的session钝化到硬盘。
  
  * 当服务器关闭，但是客户端还连接着服务器式，会把该客户端对应的session钝化到硬盘上。重启服务器的时候再活化回来。
  
    ​		
  
    ​	

### session存储数据

​	**Java域对象简介**

* 前面提到过，session是用于和客户端之间共享数据的。其实session是Java的四大域对象之一，域对象是用于项目内共享数据，分别是：pageContext page，HttpServletRequest request，HttpSession session，ServletContext application的实例对象，这四个都是接口，servlet容器会帮我们实例化。pageContext好像已经废弃了，我反正没用过。除此之外，另外三个域对象的作用范围：request < session < application
* request是在一次请求有效，也就是说，浏览器发送一次请求，在本次请求内，一次请求包括请求和响应。浏览器可以拿到存在request中的内容。但是如果被重定向了，则request中的数据就没了。请求转发依然有效。
* session是在一个会话期间有效，也就是说，只要不关闭浏览器，可以在仍何页面获取session中的数据。
* application就厉害了，只要服务器不关闭，无论什么时候，什么页面，都可以获取存储在内的数据。



通过setAttribute(name, value)向域对象存储数据

通过getAttribute(name)获取域对象中的数据

通过removeAttribute(name)删除域对象中的数据

以上三个方法是域对象共有的方法，方法名字和参数都一模一样

​		

**往session域中存储数据，通过页面取数据**

```java
@GetMapping("/getSessionData")
public String getSessionData(HttpSession session)
{
    session.setAttribute("username", "张三");	//往session域中写数据
    return "welcome";
}
```

前端页面：welcome.html  

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>				<!--下面这个就是取session的数据-->	
    <h1 th:text="你好，+${session.username}"></h1>
</body>
</html>
```



**浏览器显示的内容**

![](../img/session/4.png)

​	

​	

**总结：**

session存取数据就是这么简单，当然request和application也可以存取数据，都差不多。但是我们存取数据应该秉承一个原则就是，尽量往作用域小的地方存储。意思就是，能放request的，尽量不放session；能放session的尽量不妨applicaiton；因为存放的域对象越大，性能开销越大











