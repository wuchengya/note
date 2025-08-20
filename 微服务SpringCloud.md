# 微服务SpringCloud

## Day1

### 什么是微服务？

在大型项目中，为了降低代码之间的耦合，我们分模块开发，每个模块只负责一个部分的功能，那么如果其他的模块要使用到另一个模块的数据呢?每个模块都会向外提供一个接口，通过这个接口就可以调用到另一个模块的信息，实则是在当今的模块中发送http请求访问服务器，然后拿到的数据。

如何发送http请求？

我们可以使用RestTemplate，它是java中专门来发送http请求的类。

```java
//注册restTemplate，需要在配置类中注册，或者启动类，总之需要有@Configuration
@Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
//使用直接@Autowired注入
//调用：
String url = "http://localhost:8080/api/data";
SomeData = restTemplate.getForEntity(url,SomeData.class);我们的返回值是json数据，但是restTemplate特别智能，可以将json数据自动转成你想要的类型，这里使用SomeData代替，第二个参数就填写你想要的数据类型.class就行了。
```

**问题：我们讲url地址硬编码到了代码中，可是我们服务器地址是会变动的，并非一成不变的，那么该怎么办呢？**

### Eureka

#### Eureka 的作用

1. **服务发现机制**

- 服务提供者启动时向 Eureka 注册自己的信息
- Eureka 服务器保存这些注册信息
- 服务消费者根据服务名称向 Eureka 拉取提供者信息

2. **负载均衡处理**

- 当存在多个服务提供者时，服务消费者利用负载均衡算法，从服务列表中挑选一个合适的提供者

3. **健康状态监控**

- **心跳机制**：服务提供者每隔30秒向 Eureka Server 发送心跳请求，报告健康状态
- **状态更新**：Eureka 会实时更新记录服务列表信息，心跳不正常的服务会被自动删除
- **信息同步**：消费者可以拉取到最新的服务状态信息，确保请求发送到健康的服务实例

#### 注册eureka

第一步：在项目中新建一个微服务，在pom文件中引入

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

第二步：创建一个启动类，添加@EnableEurekaServer注解

```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }
}
```

第三步：配置application.yml文件

```yml
server:
  port: 10086
spring:
  application:
    name: eurekaserver
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka/
```

eureka自己也是一个微服务，因此它也需要一个名字name和访问url。

#### 将其他微服务注册到euraka中

第一步，引入euraka的客户端依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

第二部，配置yml文件

```yml
spring:
  application:
    name: userservice
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka/
```

#### 拒绝硬编码，使用euraka发送http请求（调用接口）

在之前我们讲url地址写死了，现在来使用eruka优化。

```java
之前的
String url = "http://localhost:8080/api/data";
SomeData = restTemplate.getForEntity(url,SomeData.class);
现在的
String url = "http://在euraka中注册的名字/相关方面的接口/"+data;
```

如果一个微服务被运行在多台服务器上该怎么办？这就需要我们选择了，如何选择？只需要在注册RestTemplate的时候添加上@LoadBalanced这个注解就行了。

### Ribbon负载均衡

![image-20250820160414638](E:\Typora picture\image-20250820160414638.png)