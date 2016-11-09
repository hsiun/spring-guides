# 第一节 消费RESTful的Web服务

这个指南带领你历经创建消费RESTful web服务的过程。

## 你将要构建什么
你将构建一个应用，这个应用使用Spring的`RestTemplate` 从`http://gturnquist-quoters.cfapps.io/api/random`取回一个随机的Spring Boot引用。

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
`git clone https://github.com/spring-guides/gs-consuming-rest.git`
+ 转移到 `gs-consuming-rest/initial`
+ 跳转到前面获取一个资源表示类

当你完成以上操作，你可以在`gs-consuming-rest/complete`在检查一遍你的代码。

# 使用Maven创建
首先，你要设置一个基本的构建脚本。当你使用Spring构建一个apps时，你可以使用任何你喜欢的构建系统，但是使用Maven构建你需要的代码就在这里。如果你不熟悉Maven，参考使用Maven构建Java项目。

### 创建目录结构
在你选的项目目录下创建下面的子目录结构；举个例子，在*nix系统下使用`mkdir -p src/main/java/hello`创建：

```
└── src
    └── main
        └── java
            └── hello
```

`pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-consuming-rest</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.4.1.RELEASE</version>
    </parent>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>
    </dependencies>


    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```


Spring Boot Maven插件提供了许多方便的特征：
+ 它收集classpath下所有的jars并且构建成单一，可运行的"über-jar"，这使得执行和迁移服务都更方便了。
+ 它查找`public static void main()`方法作为可执行类的标签。
+ 它提供一个内置的依耐解决方案，用于设置版本号以匹配Spring Boot依赖项。你可以用你喜欢的任意版本号重写，但是这将是默认Boot选择的版本集。

## 使用你的IDE构建
+ 阅读如何直接导入这个指南到Spring Tool Suite。
+ 阅读如何使用IntelliJ IDEA实现这个指南。

## 获取REST资源
随着项目设置完成，你可以完成简单的消费RESTful服务的应用了。

RESTful的服务已经启动在`http://gturnquist-quoters.cfapps.io/api/random` 。它随机的获取一个Spring Boot的引用，并且以JSON文本的对象返回它，这个对象看起来是下面这样的：
```
{
   type: "success",
   value: {
      id: 10,
      quote: "Really loving Spring Boot, makes stand alone Spring apps easy."
   }
}
```

非常简单，但当浏览区或者通过curl获取他并不是毫无用处。

一个更有用的方式消费REST web服务是通过编程。为了帮助你完成这个任务，Spring提供了非常方便被称作`RestTemplate`的模板类。`RestTemplate`使得和大多数RESTful服务交流只需一行代码。并且它可以绑定数据到用户域类型。

首先，创建一个包含你需要的域的类去保存数据。如果所有你所需要知道的是Pivotal的名字，电话号码，网站URL，和pivotal软件的页面是关于什么的，然后下面的域类应该可以很好的完成这些：
`src/main/java/hello/Quote.java`

```java
package hello;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties(ignoreUnknown = true)
public class Quote {

    private String type;
    private Value value;

    public Quote() {
    }

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    public Value getValue() {
        return value;
    }

    public void setValue(Value value) {
        this.value = value;
    }

    @Override
    public String toString() {
        return "Quote{" +
                "type='" + type + '\'' +
                ", value=" + value +
                '}';
    }
}
```
正如你所看到的，这个简单Java类处理属性并且有想匹配的getter方法。`@JsonIgnoreProperties`注解来自Jackson JSON处理库，它表示任何无法绑定到这种类型的属性都应该被忽略。

添加一个被嵌入到Quote内部的内。
`src/main/java/hello/Value.java`

```java
package hello;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties(ignoreUnknown = true)
public class Value {

    private Long id;
    private String quote;

    public Value() {
    }

    public Long getId() {
        return this.id;
    }

    public String getQuote() {
        return this.quote;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public void setQuote(String quote) {
        this.quote = quote;
    }

    @Override
    public String toString() {
        return "Value{" +
                "id=" + id +
                ", quote='" + quote + '\'' +
                '}';
    }
}
```
这使用了相同的注解，只是简单映射到其他数据字段了。

## 让应用可执行
尽管为了部署到外部的应用服务器，可以将服务打包为传统的war文件，下面展示了更简单的方法创建一个独立的应用程序。将所有的东西都打包为一个单一可执行的JAR文件，通过传统的main()方法来运行。一直以来，你使用的Spring支持的内置的tomcat servlet容器作为HTTP运行时环境，代替部署到外部的实例。

现在您可以使用`RestTemplate`编写`Application`去从我们的Spring Boot quotation服务获取数据。
`src/main/java/hello/Application.java`

```java
public class Application {

    private static final Logger log = LoggerFactory.getLogger(Application.class);

    public static void main(String args[]) {
        RestTemplate restTemplate = new RestTemplate();
        Quote quote = restTemplate.getForObject("http://gturnquist-quoters.cfapps.io/api/random", Quote.class);
        log.info(quote.toString());
    }

}
```

因为Jackson JSON处理库是在classpath中的，`RestTemplate`将使用它转换输入的JSON数据为Quote对象。在这里，`Quote`对象的内容将被记录到控制台。

这你仅仅使用`RestTemplate`去处理HTTP `GET`请求。但是`RestTemplate`也可以支持其他的HTTP动词，例如`POST`,`PUT`,和`DELETE`。

### 使用Spring Boot管理应用的生命周期

到目前为止，我们还没有使用Spring Boot在我们的应用中，但是有一些这样做的好处，并且这样做也不难。优点之一是我们可以让Spring Boot管理`RestTemplate`信息的转换，因此这样自定义很容易被添加到声明。为了这样做，我们使用在主类中使用`@SpringBootApplication`并且转换主方法去启动它，像在任何Spring Boot应用中。最后，我们使用`CommandLineRunner`回调代替`RestTemplate`，因此他可以在开始时通过Spring Boot执行：
`src/main/java/hello/Application.java`

```java
package hello;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
public class Application {

	private static final Logger log = LoggerFactory.getLogger(Application.class);

	public static void main(String args[]) {
		SpringApplication.run(Application.class);
	}

	@Bean
	public RestTemplate restTemplate(RestTemplateBuilder builder) {
		return builder.build();
	}

	@Bean
	public CommandLineRunner run(RestTemplate restTemplate) throws Exception {
		return args -> {
			Quote quote = restTemplate.getForObject(
					"http://gturnquist-quoters.cfapps.io/api/random", Quote.class);
			log.info(quote.toString());
		};
	}
}
```

`RestTemplateBuilder`是被Spring注入的，并且如果你使用它创建`RestTemplate`然后The RestTemplateBuilder is injected by Spring, and if you use it to create a RestTemplate then you will benefit from all the autoconfiguration that happens in Spring Boot with message converters and request factories. We also extract the RestTemplate into a @Bean to make it easier to test (it can be mocked more easily that way).


如果你使用Gradle，你可以使用`./gradlew bootRun`运行应用。或者你可以使用`./gradlew build`构建JAR文件。然后你就可以运行JAR文件了：
`java -jar build/libs/gs-consuming-rest-0.1.0.jar`

如果你使用Maven，你可以通过`./mvnw spring-boot:run`运行应用。或者你可以构建JAR文件使用`./mvnw clean package`。然后你可以运行JAR文件。
`java -jar target/gs-consuming-rest-0.1.0.jar`

> 上面过程将创建一个可执行的JAR。你也可以选择建立一个经典的WAR文件来代替。

你应该可以看到下面的输出：
```
2015-09-23 14:22:26.415  INFO 23613 --- [main] hello.Application  : Quote{type='success', value=Value{id=12, quote='@springboot with @springframework is pure productivity! Who said in #java one has to write double the code than in other langs? #newFavLib'}}

```

> 如果你看到一个错误`Could not extract response: no suitable HttpMessageConverter found for response type [class hello.Quote]` 这可能是因为你所在的环境不能连接到后端的服务（如果你能到达它，它将发送JSON）。也许你是在公司的代理之后？尝试设置标准系统参数`http.proxyHost`和`http.proxyPort`为适合你环境的值。

## 摘要
恭喜！你已经使用Spring开发了一个简单的REST客户端。

 想要写一个新的指南或者贡献一个已存在的？点击我们的贡献指南。

 > 所有的指南都发布了针对代码ASLv2许可证，以及归属，非衍生创作共用许可的写作。