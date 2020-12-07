# SpringCloud Alibaba

​				

## SpringCloud Alibaba的优势

* **服务限流降级**：默认支持Servlet、Feign、RestTemplate、Dubbo和RocketMQ限流降级功能的接入，可以在运行时通过控制多台实时修改限流降级规则，还支持查看限流降级 Metrics 监控。

* **服务注册与发现**：适配Spring Cloud服务注册与发现标准，默认集成了Ribbon的支持。

* **分布式配置管理**：支持分布式系统中的外部化配置，配置更改时自动刷新。

* **消息驱动能力**：基于Spring Cloud Stream 为微服务应用构建消息驱动能力。

* **阿里云对象存储**：阿里云提供的海量、安全、低成本、高可靠的云存储服务。支持在任何应用、任何时间、任何地点存储和访问任意类型的数据。

* **分布式任务调度**：提供秒级、精准、高可靠、高可用的定时（基于Cron表达式）任务调度服务。同时提供分布式的任务执行模型，如网格任务。网格任务支持海量子任务均匀分配到所有Worker（schedulerx-client）上执行。

* **分布式事务**：支持高性能且易于使用的分布式事务解决方案。

  ​			

**使用方式**

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2.2.3.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

​					

​					

​						

## Nacos

### 官方文档：

* https://nacos.io/

  ​		

### Nacos简介

> * 前四个字母分别为 Naming 和 Configuration 的前两个字母，最后的s为service
> * Nacos是一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台
> * Nacos是 **注册中心 + 配置中心** 的组合，可以理解为Nacos = Eureka + Config + Bus
> * Nacos可以替代Eureka作服务注册中心，也可以替代Config作服务配置中心

​					

### 使用方法

> #### Nacos作为注册中心使用的方法
>
> 下载nacos，后台启动 Nacos 服务端实例，ubuntu下启动命令：**bash startup.sh -m standalone**
>
> * **导入依赖**
>
>   ```xml
>   <!--Nacos注册中心-->
>   <dependency>
>    <groupId>com.alibaba.cloud</groupId>
>    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
>   </dependency>
>   ```
>
> * **编写配置文件**
>
>   ```yaml
>   spring:
>   cloud:
>    nacos:
>      server-addr: f.com:8848
>   application:
>    name: gulimall-coupon
>    # 注意这里需要配置应用名称才可以
>   ```
>
>   ​							
>
> #### Nacos作为配置中心使用方法
>
> * **引入依赖**
>
>   ```xml
>   <dependency>
>       <groupId>com.alibaba.cloud</groupId>
>       <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
>   </dependency>
>   ```
>
> * **建立配置文件bootstrap.yml，**指定配置中心地址
>
>   ```yaml
>   spring:
>     application:
>       name: gulimall-coupon
>     cloud:
>       nacos:
>         discovery:
>           server-addr: f.com:8848
>         config:
>           server-addr: f.com:8848
>           file-extension: yaml # 指定读取远端配置文件的格式为yaml，记得要指定文件后缀
>   # 这个配置文件是写在bootstrap.yml中的
>   ```
>
> * 配合注解@RefreshScope，就可以实现动态刷新了。在使用到配置文件配置的类头上标注
>
>   ![](https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201207212025.png)
>
>   ​				
>
>   ​	
>
> ##### 修改远端配置文件
>
> * **进入nacos可视化界面，点击配置列表，点击右边的加号新增配置**
>
>   <img src="https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201207214744.png" style="zoom:67%;" />
>
>   ​								
>
> * **DataId规则，官网说明如下**
>
>   <img src="https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201207213659.png" style="zoom:67%;" />
>
>   ​					
>
> #### Nacos多环境多项目配置管理
>
> * Nacos命名空间、分组、DataId三者的关系。
>
>   可以理解为（中国-成都-郫都区，只有属于该类目下面的项目才能用相应的配置文件）
>
>   <img src="https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201207215700.png" style="zoom:67%;" />
>
> * Namespace主要是用来隔离的，默认有一个命名空间为public；
>
>   例如：我们现在有三个环境：开发、测试、生产。那么我们就可以创建三个Namespace，不同的Namespace之间是隔离的
>
> * Group可以把不同的微服务划分到同一个分组里面，默认有一个分组为DEFAULT_GROUP
>
>   例如：一些微服务部署的机器硬件比较差，可以设置负载均衡的权重较小；一些部署机器硬件较好，就可以把权重设置的更高
>
> * Service就是微服务，一个微服务可以包含多个Cluster（集群），Nacos默认Cluster是DEFAULT，集群是对微服务的一个虚拟划分。例如容灾，将微服务分别部署在杭州机房和广州机房。这时候就可以给杭州机房的微服务起一个集群名字（HZ）。给广州机房的微服务起一个集群名字（GZ），还可以让同一个机房的微服务互相调用，以提升性能。
>
> * DataId如上
>
> 总之命名空间、分组、DataId就是为了区分不同条件下使用哪种配置。每个微服务配置不同的选择条件，就能读取不同的配置。

​					

​								

### Nacos集群和持久化配置

> * Nacos默认自带了一个嵌入式数据库derby，这是个嵌入式内存数据库，会将我们微服务注册的信息保存到数据库中，但是断电就会失去数据。
>
> * 因为Nacos自身自带了一个数据库，那么在当我们使用Nacos集群的时候，相当于集群中 每一台主机都带了一个数据库。这样就会导致数据不一致。
>
> * 所以Nacos提供了集中式的存储方式来支持集群化部署，目前存储方式只支持MySQL。
>
> * Nacos切换统一数据源，搭建Nacos集群必须要切换统一数据源（MySQL）。在nacos的conf目录下找到sql脚本，执行脚本nacos-mysql.sql，创建数据库和数据表（别乱改）。主页用户名密码啥的也都是存储在数据库中的。
>
>   ​		
>
> #### 官网架构图
>
> <img src="https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201207230056.png" style="zoom: 67%;" />
>
> ​			



