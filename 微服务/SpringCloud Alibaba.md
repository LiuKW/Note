## SpringCloud Alibaba

**SpringCloud Alibaba的优势**

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

### Nacos

> **官方文档**：https://nacos.io/
>
> ​				
>
> **Nacos**
>
> * 前四个字母分别为 Naming 和 Configuration 的前两个字母，最后的s为service
>
> * Nacos是一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台
>
> * Nacos是 **注册中心 + 配置中心** 的组合，可以理解为Nacos = Eureka + Config + Bus
>
> * Nacos可以替代Eureka作服务注册中心，也可以替代Config作服务配置中心
>
>   ​		
>
> 