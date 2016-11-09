# 构建RESTful的Web服务

这个指南带领你经历使用Spring创建一个"hello world"的RESTful web服务的过程。

## 你将要构建什么
你将要构建一个服务，这个服务将通过下面的地址接受一个HTTP GET请求：
```
http://localhost:8080/greeting
```

并且将响应一个JSON表示的问候：
```
{"id":1,"content":"Hello, World!"}
```

你可以使用查询字符串中可选的name参数自定义问候。
```
http://localhost:8080/greeting?name=User
```

name参数的值覆盖了默认值"World"并且反应到响应中：
```
{"id":1,"content":"Hello, User!"}
```

## 你需要什么
+ 大约15分钟
+ 一个你喜爱的文本编辑器或者IDE
+ JDK 1.8或者是更新版本
+ Gradle 2.3+或者Maven 3.0+
+ 你也可以从这个指南导入代入代码，同时查看问有直接进入Spring工具套件（STS）。并且通过它离开这里进行你自己的工作。

## 如何完成这个指南
和大多数Spring入门指南一样，你可以随意开始并完成每一步，或者你可以跳过对你来说已经熟悉的基础设置。无论哪种方式，你都是以写代码结束的。

随意的开始，跳转到使用GRadle构建
跳过基础，执行下列操作：
+ 从这个指南下载并且解压源仓库，或者使用Git克隆它：
`git clone https://github.com/spring-guides/gs-rest-service.git`
+ 转移到 `gs-rest-service/initial`
+ 跳转到前面创建一个资源表示类

当你完成以上操作，你可以在`gs-rest-service/complete`再检查一遍你的代码。

## 使用Maven构建
首先，你要设置一个基本的构建脚本。当你使用Spring构建一个apps时，你可以使用任何你喜欢的构建系统，但是使用Maven构建你需要的代码就在这里。如果你不熟悉Maven，参考使用Maven构建Java项目。

### 创建目录结构
在你选的项目目录下创建下面的子目录结构；举个例子，在*nix系统下使用`mkdir -p src/main/java/hello`创建：
```
└── src
    └── main
        └── java
            └── hello
```

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-rest-service</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.4.1.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.jayway.jsonpath</groupId>
            <artifactId>json-path</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <properties>
        <java.version>1.8</java.version>
    </properties>


    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

    <repositories>
        <repository>
            <id>spring-releases</id>
            <url>https://repo.spring.io/libs-release</url>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>spring-releases</id>
            <url>https://repo.spring.io/libs-release</url>
        </pluginRepository>
    </pluginRepositories>
</project>
```

Spring Boot Maven插件提供了许多方便的特征：
+ 它收集classpath下所有的jars并且构建成单一，可运行的"über-jar"，这使得执行和迁移服务都更方便了。
+ 它查找`public static void main()`方法作为可执行类的标签。
+ 它提供一个内置的依耐解决方案，用于设置版本号以匹配Spring Boot依赖项。你可以用你喜欢的任意版本号重写，但是这将是默认Boot选择的版本集。

## 使用你的IDE构建
+ 阅读如何直接导入这个指南到Spring Tool Suite。
+ 阅读如何使用IntelliJ IDEA实现这个指南。

### 创建一个资源表示类
现在你设置好了项目和构建系统，可以创建自己的web服务了。

通过考虑服务的交互开始这个过程。

这个服务将处理对/greeting的GET请求，在查询字符串中有一个可选的name参数。这个GET请求将返回一个200 OK响应，在正文中的JSON表示一个问候。它看起来是和下面这样的：
```
{
    "id": 1,
    "content": "Hello, World!"
}
```

这个id是问候的唯一标示，并且content是一个文本表示的问候。

为了模拟资源表示，创建一个资源表示类。提供一个含有数据域，构造函数和id及conent数据访问方法的普通java对象：
`src/main/java/hello/Greeting.java`

```java
package hello;

public class Greeting {

    private final long id;
    private final String content;

    public Greeting(long id, String content) {
        this.id = id;
        this.content = content;
    }

    public long getId() {
        return id;
    }

    public String getContent() {
        return content;
    }
}
```

> 如你在下面步骤看到的，Spring使用Jackson JSON库自动整理Greeting实例成为JSON。

下面你创建的资源控制器将服务于这个问候。

### 创建一个资源控制器
使用Spring的方法创建一个RESTful的web服务，HTTP请求被控制器处理。这个主键很容易通过`@RestController`注解实现，并且GreetingController通过返回一个新的Greeting实例处理对/greeting的请求。
`src/main/java/hello/GreetingController.java`

```java
package hello;

import java.util.concurrent.atomic.AtomicLong;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class GreetingController {

    private static final String template = "Hello, %s!";
    private final AtomicLong counter = new AtomicLong();

    @RequestMapping("/greeting")
    public Greeting greeting(@RequestParam(value="name", defaultValue="World") String name) {
        return new Greeting(counter.incrementAndGet(),
                            String.format(template, name));
    }
}
```

这个控制器简洁容易，但大量细节被隐藏。让我们一起来一步步了解隐藏的细节。

@RequestMapping注解保证了对/greeting的请求被映射到greeting()方法。

> 上面的示例不限定GET或者PUT，POST和其他方法，因为@RequestMapping默认映射所有的HTTP操作。使用@RequestMapping(method=GET)限定映射。

@RequestParam绑定查询参数name到greeting()方法的那么参数。这个查询字符串明确对的标记为可选(required=true是默认选项)：如果请求中缺少这个参数，defaultValue的值World将会被使用。

方法体的实现通过使用id和content属性基于从counter来的下一个值创建并返回一个新的Greeting对象，并且name的格式通过使用greeting的template给出。

传统的MVC控制器和上面的RESTful web服务的控制器一个关键的不同点在于，HTTP响应体的创建方式。不同于服务端渲染greeting数据到HTML的view技术，RESTful web服务的控制器只做简单的填充并返回Greeting对象。对象数据将以JSON格式直接写入HTTP响应。

这些代码使用Spring4最新的@RestController注解，这个注解表明这个类是一个控制，这个控制器的每个方法返回一个域对象代替视图。它可以简单理解为是@Controller 和 @ResponseBody的混合体。

Greeting对象必须转化为JSON。感谢与Spring对HTTP信息转换的支持，你不需要手动的做这些转换。因为Jackson 2
已经在classpath中了，Spring的MappingJackson2HttpMessageConverter自动选择转化Greting实例为JSON。

### 让应用可执行
尽管为了部署到外部的应用服务器，可以将服务打包为传统的war文件，下面展示了更简单的方法创建一个独立的应用程序。将所有的东西都打包为一个单一可执行的JAR文件，通过传统的main()方法来运行。一直以来，你使用的Spring支持的内置的tomcat servlet容器作为HTTP运行时环境，代替部署到外部的实例。
`src/main/java/hello/Application.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

@SpringBootApplication是一个可以添加下面所有特征的方便的注解。
+ @Configuration 把类标记为一个应用上下文中定义的bean资源。
+ @EnableAutoConfiguration 告诉Spring Boot开始基于calsspath的设置添加bean，和各个属性的设置。
+ 对应Spring MVC app来说，一般你会添加@EnableWebMvc，但是Spring Boot会自动添加它，当它在classpath中看到spring-webmvc。这把应用标记为web应用，并且激活如设置DispatcherServlet等关键行为。
+ @ComponentScan 告诉Spring在heello包中去查找其他的组成部分，配置和和服务，这允许了应用找到HelloContorller。


main()方法使用Spring Boot的SpringApplication.run() 方法启动一个运用。你注意到这里没有一行XML了么？连web.xml文件都没有。这个web应用是100%纯java的，并且也不需要你来处理配置任何管道或者基础设施。


### 构建一个可执行的JAR
你可以通过Graddle或者Maven在命令运行这个运用。或者你可以构建成一个单一的可执行JAR文件，这个文件包含所有必须的依耐，类，资源。这使得他很容易被运输，改变版本和在整个开发周期中部署服务作为一个应用，迁移到不同的环境，等等。

如果你使用Gradle，你可以使用`./gradlew bootRun`运行应用。或者你可以使用`./gradlew build`构建JAR文件。然后你就可以运行JAR文件了：
`java -jar build/libs/gs-rest-service-0.1.0.jar`

如果你使用Maven，你可以通过`./mvnw spring-boot:run`运行应用。或者你可以构建JAR文件使用`./mvnw clean package`。然后你可以运行JAR文件。
`java -jar target/gs-rest-service-0.1.0.jar`

> 上面过程将创建一个可执行的JAR。你也可以选择建立一个经典的WAR文件来代替。

记录的输出展示在下面。这个服务应该很快就可以启动并运行。

### 测试服务
现在服务已经启动，访问`http://localhost:8080/greeting`将看到：
```
{"id":1,"content":"Hello, World!"}
```

 `http://localhost:8080/greeting?name=User` 提供了一个name查下参数。注意content属性的值是如何从"Hello, World!" 变为 "Hello User!"的。
 ```
 {"id":2,"content":"Hello, User!"}
 ```

 这个改变表明，安排在GreetingController中@RequestParam是工作正常的。name参数被设置为默认的"World"，但是总是可以明确的通过查下字符串重写。

 也注意下id属性是如何重1变为2的。这表明你通过同一个GreetingController实例处理了多个请求，并且counter参数在每次调用时如预期的一样增加了。

 ## 摘要

 恭喜！你已经使用Spring成功的开发了一个RESTful的web服务。

 想要写一个新的指南或者贡献一个已存在的？点击我们的贡献指南。

 > 所有的指南都发布了针对代码ASLv2许可证，以及归属，非衍生创作共用许可的写作。