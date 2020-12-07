## 构建SpringCloud项目

### 创建一个空 maven 项目，将此项目作为父项目。

* **首先将打包方式设为pom**

  配置<packaging>pom</packaging>的意思是使用maven分模块管理，都会有一个父级项目，pom文件一个重要的属性就是packaging（打包类型），**一般来说所有的父级项目的packaging都为pom，packaging默认类型jar类型，如果不做配置，maven会将该项目打成jar包**。

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>
  
      <groupId>com.kingwait</groupId>
      <artifactId>testspringcloud</artifactId>
      <version>1.0-SNAPSHOT</version>
      <modules>
          <module>consumer</module>
      </modules>
      <packaging>pom</packaging>
  </project>
  ```

  ​				

*  **统一jar包管理**

  定义一些共用依赖的版本号，例如：定义mysql驱动版本号等

  ```xml
  <!--统一管理jar包版本-->
  <properties>
      <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
      <maven.compiler.source>1.8</maven.compiler.source>
      <maven.compiler.target>1.8</maven.compiler.target>
  </properties>
  ```

  ​				

* **引入SpringCloud必须的依赖。**

  1、Springboot；2、SpringCloud；3、SpringCloud Alibaba；

  其他必要的依赖也可以写在父工程，由父工程统一作版本控制管理。也可以另外创建一个 common 工程统一管理

  ```xml
  <dependencyManagement>
      <dependencies>
          <!--Springboot-->
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-dependencies</artifactId>
              <version>2.2.2.RELEASE</version>
              <type>pom</type>
              <scope>import</scope>
          </dependency>
          <!--Springcloud-->
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-dependencies</artifactId>
              <version>Hoxton.SR1</version>
              <type>pom</type>
              <scope>import</scope>
          </dependency>
          <!--Springcloud Alibaba-->
          <dependency>
              <groupId>com.alibaba.cloud</groupId>
              <artifactId>spring-cloud-alibaba-dependencies</artifactId>
              <version>2.1.0.RELEASE</version>
              <type>pom</type>
              <scope>import</scope>
          </dependency>
      </dependencies>
  </dependencyManagement>
  ```
  
​			
  
* **建立其他子模块**

  示例

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <parent>
          <artifactId>testspringcloud</artifactId>
          <groupId>com.kingwait</groupId>
          <version>1.0-SNAPSHOT</version>
      </parent>
      <modelVersion>4.0.0</modelVersion>
      <artifactId>consumer</artifactId>
  
      <dependencies>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-web</artifactId>
          </dependency>
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-starter-openfeign</artifactId>
          </dependency>
          <dependency>
              <groupId>com.alibaba.cloud</groupId>
              <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
          </dependency>
      </dependencies>
  </project>
  ```

  然后就可以开心地搬砖了

  ​				

  ​				

### maven中一些标签的解释

* **optional标签**

  > - 官方文档：https://maven.apache.org/guides/introduction/introduction-to-optional-and-excludes-dependencies.html
  > - 使用optional依赖可以节省空间和内存
  > - 当编写一个类库时，比如projectX，可能会包含很多功能，但是依赖projectX的项目projectA只会使用其中一部分功能，对于没有用到的功能，projectA不希望依赖无用功能的jar包。这时就需要projectX使用optional依赖。
  > - 如果projectA没有使用projectY相关的类，则projectY不会被打包到war包或fatjar。如果projectA要使用所有的功能，则需要在自己的项目中显式的引入相关依赖。
  > - 例如：<optional>true</optional>设置为true时候，可以只引入当前项目依赖的内容，避免了其他冗余繁杂的依赖，也降低了jar包冲突的风险

  ​				

  ​				

* **scope标签**

  > - 官方文档：https://maven.apache.org/pom.html#Dependencies
  >
  > - scope定义了依赖在项目中的使用阶段。项目阶段包括：编译，运行，测试和发布。
  >
  >   ​			
  >
  > **scope标签有一系列的取值**
  >
  > - **compile**：
  >
  > - - 如果没有提供一个范围，那该依赖的范围就是编译范围。编译范围依赖在所有的classpath 中可用，同时它们也会被打包。
  >   - 默认scope为compile，表示为当前依赖参与项目的编译、测试和运行阶段，属于强依赖。打包之时，会达到包里去。
  >
  > - **provided**：
  >
  > - - 依赖只有在当JDK 或者一个容器已提供该依赖之后才使用。例如， 如果你开发了一个web 应用，你可能在编译 classpath 中需要可用的Servlet API 来编译一个servlet，但是你不会想要在打包好的WAR 中包含这个Servlet API；这个Servlet API JAR 由你的应用服务器或者servlet 容器提供。已提供范围的依赖在编译classpath （不是运行时）可用。它们不是传递性的，也不会被打包。
  >   - 该依赖在打包过程中，不需要打进去，这个由运行的环境来提供，比如tomcat或者基础类库等等，事实上，该依赖可以参与编译、测试和运行等周期，与compile等同。区别在于打包阶段进行了exclude操作。
  >
  > - **runtime**：
  >
  > - - 依赖在运行和测试系统的时候需要，但在编译的时候不需要。比如，你可能在编译的时候只需要JDBC API JAR，而只有在运行的时候才需要JDBC
  >   - 依赖仅参与运行周期中的使用。一般这种类库都是接口与实现相分离的类库，比如JDBC类库，在编译之时仅依赖相关的接口，在具体的运行之时，才需要具体的mysql、oracle等等数据的驱动程序。此类的驱动都是为runtime的类库。
  >
  > - **test**：
  >
  > - - 在一般的编译和运行时都不需要，它们只有在测试编译和测试运行阶段可用
  >   - 该依赖仅仅参与测试相关的内容，包括测试用例的编译和执行，比如定性的Junit。
  >
  > - **system**：
  >
  > - - system范围依赖与provided 类似，但是你必须显式的提供一个对于本地系统中JAR 文件的路径。这么做是为了允许基于本地对象编译，而这些对象是系统类库的一部分。这样的构件应该是一直可用的，Maven 也不会在仓库中去寻找它。如果你将一个依赖范围设置成系统范围，你必须同时提供一个 systemPath 元素。注意该范围是不推荐使用的（你应该一直尽量去从公共或定制的 Maven 仓库中引用依赖）。
  >   - 使用上与provided相同，不同之处在于该依赖不从maven仓库中提取，而是从本地文件系统中提取，其会参照systemPath的属性进行提取依赖。
  >
  > - **import**：
  >
  > - - 这个是maven2.0.9版本后出的属性，import只能在dependencyManagement的中使用，能解决maven单继承问题，import依赖关系实际上并不参与限制依赖关系的传递性。

  ​						

  ​						

* **packaging标签**

  > 配置<packaging>pom</packaging>的意思是使用maven分模块管理，都会有一个父级项目，pom文件一个重要的属性就是packaging（打包类型），一般来说所有的父级项目的packaging都为pom，packaging默认类型jar类型，如果不做配置，maven会将该项目打成jar包。

  ​			

  ​			

* **type标签**

  > * type标签代表引入的类型。默认值为jar，其他取值还有war、bar、pom。当type=pom时，必须scope=import。
  >
  > * 当需要引入很多依赖的时候，pom.xml文件会过大，我们可以通过依赖一个父项目来解决这个问题，但也可能导致府项目pom.xml文件过大，所以最终的方法是type改成pom方式，即把很多jar包打包到一个pom中，我们依赖了pom，就可以下载所有的jar包。
  >
  > * pom是Maven对一个单一项目的描述。没有pom的话，Maven是毫无用处的——pom是Maven的核心。是pom实现的并驱动了这种以模型来描述的构建方式。一个pom包含了关于你的项目的所有重要信息。
  > * 每个工程有且只有一个 pom文件，所有的 pom文件需要 project 元素和三个必须的字段：groupId, artifactId,version。
  >   所有的 POM 都继承自一个父 pom（Super POM），Maven 使用 effective pom（Super pom 加上工程自己的配置）来执行相关的目标，它帮助开发者在 pom.xml 中做尽可能少的配置，当然这些配置可以被方便的重写。