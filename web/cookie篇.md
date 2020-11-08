## Cookie

### Cookie介绍

* cookie是一种用于web应用会话跟踪的技术。

* 因为http协议是一种无状态的协议。无状态就是http的每一次访问之间都是独立的，http的上一次请求和下一次请求之间是没有任何关联的。而在实际web应用场景中，很多时候我们都需要判断上一次操作和下一次操作是否是同一个用户执行的。但是http是无状态协议，无法判断。因此cookie就诞生了。

* cookie是由服务器生成保存在客户端的一种信息载体。服务器生成数据保存在cookie中，通过http协议传回给客户端保存在本地磁盘中。客户端在下一次访问服务器的时候，再把cookie里面的数据通过http协议带回给服务器，这样就能够判断是否是同一个用户在操作了。对，cookie就是这样简单的概念。

* cookie的数据结构由若干个键值对构成，类似Java中的Map的key-value结构。cookie的键一般称为name，值称为value，**键和值的数据类型只能为字符串**。且cookie中的name值是不能由重复的，**如果name值重复，新的就会把旧的覆盖掉**

  ​	

  ​	

### Cookie用途

* 服务器结合Cookie和Session跟踪客户端状态

* 保存购物车信息

* 显示登录用户名，等等...

  **下面我们通过Cookie的实例，了解在开发中如何使用cookie**
  
  ​	
  
  ​	

### 实例1：用Java写一个cookie，并传回给浏览器

**创建springboot工程**

```java
@ResponseBody
@GetMapping("/Cookie")
public String GetCookie(HttpServletResponse response)
{
    Cookie cookie = new Cookie("UserName","Mike");  //新建一个cookie
    cookie.setMaxAge(60*6C0);	//存活时间一个小时
    //cookie.setPath("/cookie")
    response.addCookie(cookie);
    return "Cookie已经返回给客户端，请在浏览器调试工具中查看";
}
```

- 通过Cookie这个类new出一个cookie，构造方法传递的参数就是cookie的键值对

* 通过**setMaxAge(int expiry)**方法设置cookie的存活时间，存活时间指cookie在浏览器中保存的时间。如果不设置保存时间，默认的存活时间是一个会话期间。浏览器从开启直至关闭则称为一个会话期间。这个方法的单位是秒，程序中设置成60*60就是一个小时。
  * expiry > 0：浏览器会把cookie保存在硬盘上，时间为expiry的值，单位秒。
  * expiry < 0：浏览器会把cookie保存在内存中，浏览器进程关闭，内存就被释放，cookie就没了。
  * expiry = 0：浏览器会马上删除这个cookie。
* 通过**setPath**方法设置cookie在访问哪个url的时候会被带上 。如果不写的话，默认访问项目下的所有资源都会被带上这个cookie。上面的例子，在我们访问localhost:8080/cookie的时候，http协议就会把这个cookie封装到报文里面，带回给服务器。
* response.addCookie()就可以向浏览器写回一个cookie了。用的话就是这么简单。

  



**打开Chrome的调试工具，访问localhost:8080/cookie，在调试工具中查看http携带的报文数据**

![](../img/cookie/1.png)

* 在**Response Cookies(响应报文的cookies字段)**可以看到，客户端返回了一个cookies。

  - Name值为UserName，Value值为Mike，Path是源地址，来源于/cookie，Expires为存活时间，值是1.0hrs。和我们在程序中设置的一模一样。

    

* 再观察**Request Cookies(请求报文的cookies字段)**，有两个cookie，其中一个是JSESSIONID，这个在session中会讲到。

  - **Path**字段就是访问该路径的资源文件时，会把这个cookie带回给服务器。'/' 就是指访问该项目下的所有资源文件都会把这个cookie带回给服务器。我们在代码中没有设置cookie的访问路径，所以默认为'/'（那行setPath代码被注释了）。

  - **Expires/Max-Age**字段就是cookie的生存时长

    

**小结：**

在Java中使用cookie就是这么简单，通过Cookie类new出一个cookie，在由HttpServletResponse的实例对象传回给浏览器，这个实例对象SpringMVC已经帮你做好了封装，我们直接用就可以了。设置cookie的存活时间。浏览器就会自动帮我们保存cookie数据了。

​	

​		

### 实例2：在Java中读取cookie的值

在Java程序中我们通过**HttpServletRequest**的实例对象获取cookie值

```java
@ResponseBody
@GetMapping("/ReadCookie")
public void ReadCookie(HttpServletRequest request, HttpServletResponse response) throws IOException
{
    Cookie[] cookies = request.getCookies();  //获取cookies
    for (Cookie cookie : cookies) {
        if(cookie.getName().equals("UserName"))     //判断是否是想要的cookie
        {
            response.getWriter().println( //输出到浏览器
                "Name: "+cookie.getName()+" Value: "+ cookie.getValue()
            );
        }
    }
}
```

- HttpServletRequest只提供了返回cookie数组的方法。在实际应用中，cookie可能是有许多个的，我们通过HttpServletRequest提供的getCookies返回cookie数组。然后遍历取出想要的值。取出cookie的Name字段，通过equals方法判断是否是想要的那个cookie。

- 其实SpringMVC中提供了一个注解**@CookieValue**可以取cookie的值。

  ​	

  ​		

### 实例3：在Java中修改cookie的值

我们前面提到过，cookie是map的数据结构，且name值不能重复，name值重复的cookie会被新的覆盖掉，所以当我们想要删除一个cookie的时候，只需要new一个name值相同的cookie把之前的覆盖就可以了即可。

```java
@ResponseBody
@GetMapping("/UpdateCookie")
public String UpdateCookie(HttpServletResponse response)
{
    Cookie cookie = new Cookie("UserName","Jack");
    cookie.setMaxAge(60*60);
    response.addCookie(cookie);
    return "Cookie已修改";
}
```

​		

​		

### 实例4：删除cookie

我们前面提到过，当cookie的存活时间为0的时候，则cookie会被删除，所以我们只需要公共sesMaxAge方法设置存活时长为0即可。我们也可以直接new一个name值重复的把之前的覆盖掉就可以了。

```java
@ResponseBody
@GetMapping("/DeleteCookie")
public String DeleteCookie(HttpServletResponse response)
{
    Cookie cookie = new Cookie("UserName","Jack");
    cookie.setMaxAge(0);  //存活时间设置为0，就是删除
    response.addCookie(cookie);
    return "Cookie已删除";
}
```

​		

​		

### 实例5：通过cookie保存登录信息。

我们创建一个springboot工程，写一个用户名密码登录的简单表单提交程序，使用thymeleaf模板引擎。

用户在login.html输入用户名密码，正确则跳转到success.html页面。

如果登录成功，则我们下次再访问login.html页面时，就可以看到用户名输入框就直接有上次输入过的用户名。具体看下面的代码和实例截图。

```java
@Controller
public class MyController {
    @GetMapping("/index")
    public String Index()
    {
        return "login";	//登录页面
    }
    @PostMapping("/login")
    public String Login(@RequestParam("username") String username, 										@RequestParam("userpw") String userpw, HttpServletResponse response)
    {
        if(!"asd".equals(username) && "123".equals(userpw))
        {//这里就不连接数据库了，直接判断用户名密码是否正确
         
            Cookie cookie = new Cookie("username",username);
            cookie.setMaxAge(60*60);
            response.addCookie(cookie);
            return "success";  //登录成功页面
        }
        else
            return "login";  //用户名密码错误，返回登录页面
    }
}
```

​		

**login.html**

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
</head>
<body>
    <form>
   		<input 
               type="text" name="username" 
               placeholder="username" 
               th:each="cookie:${#request.getCookies()}"
               th:if="${cookie.getName().equals('username')}"
               th:text="${cookie.getValue}"
        /><br/>
        <input type="password" name="password" placeholder="password"/><br/>
        <input type="submit" value="Login in"/>
    </form>
</body>
</html>
这里我没能想到应该怎样回显正确的cookie到表单，因为只能获取cookie的数组，又必须要遍历，遍历就会生成很多没必要的标签。大家可以帮我想一想。
```

​	

**登录成功跳转的页面：success.html**

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
</head>
<body>
    <h1 th:each="cookie:${#request.getCookies()}" th:if="${cookie.getName().equals('username')}" th:value="Welcome，${cookie.getValue()}"></h1>
</body>
</html>
```

取出存在cookie中的用户名字

​		

​		

### Cookie的安全隐患

Cookie是由服务端生成保存在客户端的数据。既然是在客户端保存的，那么就可能会有被恶意修改、破解的风险。所以在实际开发中，不应该把敏感信息的数据写在cookie里面。例如，密码信息就不应该保存在cookie中。在实例5中，只是把用户名通过cookie保存在客户端，而并没有把密码也保存进去。密码就属于敏感信息的一项。在开发中要留意这一点，以免造成损失。











