# 第一节 构建RESTful的Web服务

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
