# Shiro

​			

### Shiro简介

***

> Apache Shiro是一个强大且易用的Java安全框架,执行身份验证、授权、密码和会话管理。使用Shiro的易于理解的API,您可以快速、轻松地获得任何应用程序,从最小的移动应用程序到最大的网络和企业应用程序。
>
> **官方文档**：https://shiro.apache.org/reference.html
>
> https://www.w3cschool.cn/shiro/oibf1ifh.html

​			

​			

​			

### SpringBoot中整合Shiro

***

#### 引入pom文件

```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.4.0</version>
</dependency>
```

#### 编写配置

Springboot原生没有提供shiro的支持，没有默认的自动配置类，所以需要我们手动配置

```java
/**
 * 配置shiro分三步：
 *  1、配置SecurityManager，我们使用默认的子类DefaultWebSecurityManager即可
 *  2、配置Realm，新建一个Realm继承Realm的子类（有验证和鉴权的子类，看情况继承），这个Realm用于处理身份检验和鉴权逻辑
 *  3、配置shiro过滤器ShiroFilterFactoryBean，配置拦截和鉴权规则
 *
 *  之后就是按需配置了，配置开启AOP支持，与模板引擎的整合等；
 */
@Configuration
public class ShiroConfig {

    @Bean
    public SecurityManager securityManager(Realm myRealm) {
        DefaultWebSecurityManager manager = new DefaultWebSecurityManager();
        manager.setRealm(myRealm);
        return manager;
    }

    @Bean
    public MyRealm myRealm() {
        return new MyRealm();
    }

    // 配置Shiro过滤器Bean，这个Bean将配置Shiro相关的规则拦截
    @Bean
    public ShiroFilterFactoryBean shiroFilter(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        // 设置登录等页面
        shiroFilterFactoryBean.setLoginUrl("/");   // 如果需要登录，就会转到这个请求地址
        shiroFilterFactoryBean.setSuccessUrl("/success"); // 配置登录成功以后转向的请求地址
        shiroFilterFactoryBean.setUnauthorizedUrl("/noPermission"); // 配置没有权限时转向的请求地址

        // 配置权限拦截
        Map<String, String> filterChainMap = new LinkedHashMap<>();
        filterChainMap.put("/login", "anon");       // 配置登录请求不需要认证，anon表示某个请求不需要认证
        filterChainMap.put("/logot", "logout");     // 配置登出请求，登出会自动清除相应的session
        // 配置 admin 开头的所有请求需要登录，authc表示需要登录认证
        // roles[admin] /admin 开头的所有请求，需要有admin角色才可以使用
        filterChainMap.put("/admin/**", "authc, roles[admin]");
        // 配置 admin 开头的所有请求需要登录，authc表示需要登录认证
        filterChainMap.put("/user/**", "authc, roles[user]");    
        filterChainMap.put("/**", "authc");         // 剩余的所有请求需要登录认证

        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainMap);
        return shiroFilterFactoryBean;
    }

    /**
     * 开启shiro的注解支持（例如:@RequiresRoles()、@RequiresPermissions())
     * shiro的注解需要借助Spring的AOP实现
     * @return
     */
    @Bean
    public DefaultAdvisorAutoProxyCreator advisorAutoProxyCreator() {
        DefaultAdvisorAutoProxyCreator autoProxyCreator = new DefaultAdvisorAutoProxyCreator();
        autoProxyCreator.setProxyTargetClass(true);
        return autoProxyCreator;
    }

    /**
     * 开启AOP的注解支持
     * @param securityManager
     * @return
     */
    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor
        (SecurityManager securityManager) {
        AuthorizationAttributeSourceAdvisor advisor = new AuthorizationAttributeSourceAdvisor();
        advisor.setSecurityManager(securityManager);
        return advisor;
    }


    /**
     * 配置shiro标签与Themeleaf的集成
     * @return
     */
    @Bean
    public ShiroDialect shiroDialect() {
        return new ShiroDialect();
    }

    // 引入命名空间
    // xmlns:shiro="http://www.pollix.at/thymeleaf/shiro"
}
```

#### 自定义Realm

```java
/**
 * 自定义Realm，继承shiro的Realm；
 * 用于认证或授权
 * AuthenticatingRealm是AuthorizingRealm的父类
 * AuthenticatingRealm只负责认证（登录）
 * AuthorizingRealm用于授权（既可以认证也可以授权）
 */
public class MyRealm extends AuthorizingRealm {
    /**
     * 用户认证的方法，这个方法不能手动调用，shiro会自动调用
     * @param authenticationToken 用户身份，该参数存放着用户的账号和密码，这个参数就是 subject.login() 传进来的参数
     * @return 用户登录成功后的身份证明
     * @throws AuthenticationException 如果认证失败，shiro会抛出各种异常，需要一个个去捕获不同的异常作不同的处理
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo
        (AuthenticationToken authenticationToken) throws AuthenticationException {
        UsernamePasswordToken usernamePasswordToken = (UsernamePasswordToken)authenticationToken;
        String username = usernamePasswordToken.getUsername();
        String password = new String(usernamePasswordToken.getPassword());

        /**
         * 认证账号，从数据库中查询数据。
         * 注意：
         *  认证之前首先要判断账号是否可用，IP是否允许等之后，再进行身份认证
         */
        if(!"admin".equals(username) && !"user".equals(username))
            throw new UnknownAccountException();
        if("zhangsan".equals(username))
            throw new LockedAccountException();

        /**
         * 数据密码加密主要是防止数据在浏览器传输到后台的过程中防止被篡改或截获，因此应该在前台到后台的过程中进行加密。
         * 而我们这里的加密是将浏览器传输过来的明文加密和对数据库中的数据进行加密。这就丢失了数据加密的意义。因此页面也要加密
         */
        // 设置让当前登录用户中的密码数据进行加密
        HashedCredentialsMatcher credentialsMatcher = new HashedCredentialsMatcher();
        credentialsMatcher.setHashAlgorithmName("MD5");     // 加密算法
        credentialsMatcher.setHashIterations(1);            // 加密次数
        this.setCredentialsMatcher(credentialsMatcher);

        /**
         * 使用shiro提供的工具对数据进行加密
         * 参数 1 ：加密算法
         * 参数 2 ：加密的数据（密码）
         * 参数 3 ：盐
         * 参数 4 ：加密次数
         */
        // Object obj = new SimpleHash("MD5", "123456", "", 2);


        /**
         * 创建密码认证对象，由shiro自动认证密码
         * 参数 1 ：数据库中的账号，或者前端用户输入的账号
         * 参数 2 ：数据库中读取出来的密码
         * 参数 3 ：当前Realm名字
         * 如果密码认证成功则返回一个用户身份对象，如果密码认证失败shiro会抛出 IncorrectCredentialsException 异常（密码错误）
         */
        return new SimpleAuthenticationInfo(username, "e10adc3949ba59abbe56e057f20f883e", getName());
    }


    /**
     * 用户授权的方法，当用户认证通过以后，每次访问需要授权的请求时，都要执行这段代码以完成授权操作
     * 这里应该查询数据库来获取当前用户的所有角色和权限，并设置到shiro中
     * @param principalCollection
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        // 查询数据库
        // 获取用户账号，根据账号从数据库中获取数据，这个其实是前端传过来的登录用户名，好像强转成字符串也ok
        Object obj = principalCollection.getPrimaryPrincipal();     

        // 定义用户角色的set集合，这个集合应该来自数据库
        // 注意，由于每次点击需要授权的请求时，shiro都会执行这个方法，所以一定要控制好数据库的请求次数，比如用缓存等
        Set<String> roles = new HashSet<>();

        // 设置角色，这里的操作应该时读取数据库中的数据
        if("admin".equals(obj)) {
            roles.add("admin");
            roles.add("user");
        }
        if("user".equals(obj)) {
            roles.add("user");
        }

        // 设置权限，这个操作应该是从数据库中读取
        Set<String> permissions = new HashSet<>();
        if("admin".equals(obj)) {
            // 添加一个权限，admin:add只是一种命名风格，表示admin下的add功能
            permissions.add("admin:add");
            permissions.add("user:add");
        }
        if("user".equals(obj)) {
            // 添加一个权限
            permissions.add("user:add");
        }


        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        info.setRoles(roles);                       // 设置角色信息
        info.setStringPermissions(permissions);     // 设置用户的权限信息
        return info;
    }
}
```

#### 来看一个登录请求

```java
@PostMapping("/login")
public String login(String username, String password, Model model) {
    Subject subject = SecurityUtils.getSubject();	// 获取当前登录的用户信息
    // 如果用户未登录过
    if (!subject.isAuthenticated()) {
        // 将登录账号和密码传入
        UsernamePasswordToken token = new UsernamePasswordToken(username, password);	
        try {
            // 登录
            subject.login(token);
        } catch (UnknownAccountException e) {
            model.addAttribute("errMsg", e.getMessage());
            return "login";
        } catch (LockedAccountException e) {
            model.addAttribute("errMsg", e.getMessage());
            return "login";
        } catch (AuthenticationException e) {
            model.addAttribute("errMsg", e.getMessage());
            return "login";
        }
		// 通过捕获不同的异常判断登录阶段返回什么错误，以提示相应详细信息
    }
    return "redirect:/success";
}
```

​		

​		

​		

### 权限和角色校验

***

**在Shiro中有三种方式判断用户是否符合校验规则**

* 基于配置的方式

  ```java
  // 就是写在ShiroConfig的配置中，在ShiroFilterFactoryBean内配置
  public ShiroFilterFactoryBean shiroFilterFactoryBean(DefaultWebSecurityManager webSecurityManager) {
      ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
      shiroFilterFactoryBean.setSecurityManager(webSecurityManager);
      shiroFilterFactoryBean.setLoginUrl("/");
      shiroFilterFactoryBean.setSuccessUrl("/success");
      shiroFilterFactoryBean.setUnauthorizedUrl("/no_permission");
  
      // 配置拦截规则，k是拦截路径，v是拦截器；更多的拦截器可以参考DefaultFilter类
      Map<String, String> filterChain = new HashMap<>();
      filterChain.put("/login", "anon");
      filterChain.put("/logout", "logout");
      filterChain.put("/admin/**", "authc, roles[admin, user]");
      filterChain.put("/user/**", "authc, roles[user]");
      filterChain.put("/**", "authc");
  
      shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChain);
  
      return shiroFilterFactoryBean;
  }
  ```

* 基于注解的方式配置

  使用注解需要在配置中声明开启AOP注解，shiro的注解是基于Spring的AOP实现的

  ```java
  /**
       * 开启shiro的注解支持（例如:@RequiresRoles()、@RequiresPermissions())
       * shiro的注解需要借助Spring的AOP实现
       * @return
       */
  @Bean
  public DefaultAdvisorAutoProxyCreator advisorAutoProxyCreator() {
      DefaultAdvisorAutoProxyCreator autoProxyCreator = new DefaultAdvisorAutoProxyCreator();
      autoProxyCreator.setProxyTargetClass(true);
      return autoProxyCreator;
  }
  
  /**
       * 开启AOP的注解支持
       * @param securityManager
       * @return
       */
  @Bean
  public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor
      (SecurityManager securityManager) {
      AuthorizationAttributeSourceAdvisor advisor = new AuthorizationAttributeSourceAdvisor();
      advisor.setSecurityManager(securityManager);
      return advisor;
  }
  ```

* 基于方法调用的方式配置

  ```java
  @RequestMapping("/admin/add")
  @ResponseBody
  public String adminAdd() {
      // 通过SecurityUtils 获取subject，进行一系列鉴权操作
      Subject subject = SecurityUtils.getSubject();
      String[] roles = {"admin"};
      String[] permissions = {"admin:add"};
      subject.checkRoles(roles);
      subject.checkPermissions(permissions);
      return "/admin/add请求";
  }
  // 这种基于方法调用的方式与注解是等价的，上下两种编码是等价的
  @RequiresRoles({"admin"})
  @RequiresPermissions(value = "admin:add")
  @RequestMapping("/admin/add")
  @ResponseBody
  public String adminAdd() {
      return "/admin/add请求";
  }
  ```

  

