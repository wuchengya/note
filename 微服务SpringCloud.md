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

### Ribbon（瑞本）

#### Ribbon是什么？

在微服务架构中，一个服务（比如订单服务）常常需要调用另一个服务（比如用户服务）。但用户服务为了保证高可用，通常会部署多个实例（比如在 8001, 8002, 8003 端口上运行了三个相同的用户服务实例）。

这时，订单服务在调用时就面临一个问题：**我应该调用哪一个用户服务实例呢？**

**Ribbon** 就是来解决这个问题的。它是一个**客户端负载均衡器**。

#### Ribbon负载均衡原理

![image-20250820160414638](E:\Typora picture\image-20250820160414638.png)

由上图可以看出我们对于选择哪一台服务器去发送请求是按照负载均衡的选择规则的，那么这个规则是什么呢？又该如何改变呢？

#### Ribbon负载均衡策略

##### IRule 接口

IRule 就是 Ribbon 负载均衡策略的核心接口。它定义了“如何从一堆服务实例中选择一个”的算法。可以把它想象成一个“决策者”，每次调用前，Ribbon 都会问它：“嘿，IRule，这次该调用哪个服务器？”

##### IRule 的几种内置实现（负载均衡策略）

Ribbon 已经为我们提供了多种开箱即用的负载均衡策略，它们都是 IRule 接口的实现类。

| 类名                          | 策略描述                                                     |
| ----------------------------- | ------------------------------------------------------------ |
| **RoundRobinRule**            | **轮询（Round Robin）**。按顺序循环选择服务实例。比如有3个实例，第一次选第1个，第二次选第2个，第三次选第3个，第四次再回到第1个。这是最经典的策略。 |
| **RandomRule**                | **随机（Random）**。从服务列表中随机选择一个实例。           |
| **AvailabilityFilteringRule** | **可用性过滤**。会先过滤掉那些多次连接失败、处于“断路器跳闸”状态的服务，然后再对剩下的服务进行轮询。 |
| **WeightedResponseTimeRule**  | **加权响应时间**。根据每个服务实例的平均响应时间来计算权重。响应时间越短，权重越大，被选中的概率就越高。它在启动时会先用轮询，等统计信息足够后再切换。 |
| **RetryRule**                 | **重试**。在默认的轮询策略基础上，增加了重试机制。如果在指定时间内选择的实例无法连接，它会自动重试，选择下一个实例。 |
| **BestAvailableRule**         | **最佳可用**。选择并发请求量最小的服务器。它会遍历所有实例，选出那个最“闲”的。 |
| **ZoneAvoidanceRule**         | **区域回避**。这是 **Spring Cloud Ribbon 的默认规则**。它综合了服务所在区域（Zone）和可用性来选择服务器。在没有多区域（Zone）部署的简单场景下，它的行为基本等同于 RoundRobinRule（轮询/默认）。 |

##### 如何更换默认的负载均衡策略？

###### 方法一：通过配置类（推荐，最灵活）

**核心思想**：创建一个独立的配置类，在这个类中创建你想要的 IRule Bean。然后通过注解告诉 Ribbon，在调用指定服务时使用这个配置类。

第一步：新建一个包，这个包不能是启动类的所在包及子包，然后在新建包中新建一个配置类，然后注册一个bean就是代表你要是用的策略。

```java
// 正确示范
package com.example.myrules; // 独立的包

@Configuration
public class MyRuleConfig {
    @Bean
    public IRule myRule() {
        return new RandomRule();
    }
}
```

第二步：在启动类或者子类（能够被扫描到的）中添加注解@RibbonClient

```java
// 正确示范
package com.example.myapp; // 主程序的包

@SpringBootApplication
// 这个注解告诉 Spring: "去为 USERSERVICE 创建一个子上下文，并使用 com.example.myrules.MyRuleConfig 来加载配置"
@RibbonClient(name = "USERSERVICE", configuration = com.example.myrules.MyRuleConfig.class)
public class OrderServiceApplication { ... }
```

**总结一下：**

**配置类（MyRuleConfig）和应用该配置的注解（@RibbonClient）必须是分离的。**

- @RibbonClient 注解需要放在一个能被 SpringBoot 主程序扫描到的地方。
- configuration 属性所指向的配置类，则必须放在一个主程序扫描不到的地方，以保证其隔离性，防止配置全局生效。

###### 方法二：通过配置文件设置规则

核心思想

Ribbon 允许你通过在 application.yml (或 application.properties) 文件中指定一个属性，来告诉它应该使用哪个 IRule 实现类。Ribbon 启动时会读取这个配置，并通过反射机制创建对应规则类的实例。

```yml
# 全局设置为随机
ribbon:
  NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule

# 但对 USERSERVICE 单独设置为轮询
USERSERVICE:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RoundRobinRule
```

- **轮询**: com.netflix.loadbalancer.RoundRobinRule
- **随机**: com.netflix.loadbalancer.RandomRule
- **重试**: com.netflix.loadbalancer.RetryRule
- **加权响应时间**: com.netflix.loadbalancer.WeightedResponseTimeRule
- **最佳可用**: com.netflix.loadbalancer.BestAvailableRule
- **可用性过滤**: com.netflix.loadbalancer.AvailabilityFilteringRule
- **区域回避 (默认)**: com.netflix.loadbalancer.ZoneAvoidanceRule

------

##### 懒加载与饥饿加载

###### Ribbon 的懒加载机制是什么？

懒加载意味着，Ribbon 不会在你的服务（比如订单服务）一启动时，就立刻为它需要调用的所有其他服务（比如用户服务、商品服务）创建好对应的负载均衡客户端（LoadBalancerClient）。

相反，它会等到**第一次**实际发生对某个特定服务的调用时，才会去初始化和创建与该服务相关的组件，包括：

- ILoadBalancer：负载均衡器实例。
- IRule：负载均衡策略实例。
- IPing：服务实例健康检查策略实例。
- 从服务注册中心（如 Eureka）拉取该服务的实例列表。

###### 懒加载的优缺点

优点：

1. **加快启动速度**：如果你的服务需要调用很多个其他的微服务，懒加载可以避免在启动时为所有这些服务都创建客户端，从而显著减少应用的启动时间。这在大型微服务项目中尤其重要。
2. **节省资源**：只为实际被调用的服务创建和维护客户端实例，可以节省内存和 CPU 资源。

缺点：

1. **首次调用延迟（First Call Latency）**：由于第一次请求需要触发一系列的初始化工作（创建 Bean、拉取服务列表等），所以**首次调用**的耗时会明显高于后续的调用。这可能会导致第一次请求出现超时失败的情况，尤其是在网络环境较差或服务实例较多时。

###### 如何解决首次调用延迟问题？

如果首次调用的延迟对你的业务场景影响很大，你可以选择关闭懒加载，让 Ribbon 在应用启动时就完成初始化。这种方式被称为**饥饿加载（Eager Loading）**。

###### 如何开启饥饿加载？

```yml
# application.yml

ribbon:
  # 开启 Ribbon 的饥饿加载模式
  eager-load:
    enabled: true




# application.yml

ribbon:
  eager-load:
    enabled: true
    # 指定需要饥饿加载的服务名列表（区分大小写）
    clients:
      - USERSERVICE
      - PRODUCTSERVICE
```

### Nacos(内克思)

#### 如何启动？

到bin目录下，然后运行命令：

```
startup.cmd -m standalone
```

以单机模式启动

```
startup.cmd
```

以集群模式启动

#### 服务注册到nacos步骤

+ 在父工程中添加 spring - cloud - alibaba 的管理依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring - cloud - alibaba - dependencies</artifactId>
    <version>2.2.5.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

+ 添加 nacos 的客户端依赖(对于每一个微服务都要添加)

```xml
<!-- nacos客户端依赖 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring - cloud - starter - alibaba - nacos - discovery</artifactId>
</dependency>
```

+ 修改微服务中的所有application.yml文件

```
spring:
	cloud:
		nacos:
			sever-addr: localhost:8848
```

#### nacoa服务分级存储模型

配置集群

```
spring:
	cloud:
		nacos:
			sever-addr: localhost:8848
			discovery:
				cluster-name: HZ（自定义）
```

上述操作讲这个实例位置订到了杭州

一个微服务要调用另一个微服务，那我们肯定是越近调用越好，因此，如果一个微服务a在杭州，他要调用微服务b，那么肯定是调用本地的好，因为延迟低，在上面我们已经给实例配置了地址，如何在调用服务的时候使用与调用的服务相同的集群呢？

```xml
userservice(b服务):
  ribbon:
    NFLoadBalancerRuleClassName: com.alibaba.cloud.nacos.ribbon.NacosRule # 负载均衡规则 
```

这个会随机调用当地的实例。

#### 根据权重负载均衡

好的机器要承担更多的访问量，毕竟能者多劳。如何修改权重配置呢？

在nacos控制台修改即可，0-1，越大越能被访问。

#### nacos环境隔离

在nacos台中新建命名空间(会自动生成id)，然后在微服务的yml文件中配置namespace的id

```yml
spring:
	cloud:
		nacos:
			sever-addr: localhost:8848
			discovery:
				cluster-name: HZ（自定义）
				namcespace: id
```

这样的话此服务就会被分配到另一个空间中，不同的空间不能够互相访问

服务注册到 Nacos 时，可配置注册为临时或非临时实例，配置方式如下:

```yml
spring:
  cloud:
    nacos:
      discovery:
        ephemeral: false # 设置为非临时实例 
```

1. Nacos 与 Eureka 的共同点

- 都支持服务注册和服务拉取
- 都支持服务提供者心跳方式做健康检测

2. Nacos 与 Eureka 的区别

- Nacos 支持服务端主动检测提供者状态：临时实例采用心跳模式，非临时实例采用主动检测模式
- 临时实例心跳不正常会被剔除，非临时实例则不会被剔除
- Nacos 支持服务列表变更的消息推送模式，服务列表更新更及时
- Nacos 集群默认采用 AP 方式，当集群中存在非临时实例时，采用 CP 模式；Eureka 采用 AP 方式

## Day2

#### Nacos配置管理

1. 依赖引入

- 需在项目中添加 **Nacos 配置管理客户端依赖**，代码片段：

```xml
<!--nacos配置管理依赖-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

作用是让应用能**连接 Nacos 配置中心**，拉取配置。

2. 配置引导文件

- 在`userservice`的`resource`目录新增 **`bootstrap.yml`文件**（优先级高于`application.yml` ），配置内容：

```yaml
spring:
  application:
    name: userservice # 服务名称
  profiles:
    active: dev # 开发环境，这里是dev
  cloud:
    nacos:
      server-addr: localhost:8848 # Nacos地址
      config:
        file-extension: yaml # 文件后缀名
```

**目的是指定服务名、环境、Nacos 地址及配置文件格式**，让应用启动时从 Nacos 拉取对应配置 。

### Feign(发送http请求，用来代替RestTemplate)

#### 如何使用？

1：

```xml
<dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

在微服务中引入依赖

2：

添加 @EnableFeignClients 注解

这个注解的作用是**开启 Spring Cloud 对 Feign 的支持**。Spring Boot 在启动时会扫描被 @FeignClient 注解的接口，并为它们创建代理实现类。

将这个注解添加到你的**启动类**上。

3：

```java
@FeignClient("userservice")//服务名称，表示为那个微服务发送请求
public interface UserClient {
    @GetMapping("/user/{id}")
    User findById(@PathVariable("id") Long id);
}
```

如何使用？注入然后调用方法就行了

feign内置ribbon,会自动实现负载均衡。

#### feigin配置日志级别

**日志级别都有什么？**

共有四种级别，从简单到详细：

1. **NONE**: 不记录任何日志 (默认值)。
2. **BASIC**: 仅记录请求方法、URL、响应状态码和执行时间。
3. **HEADERS**: 在 BASIC 基础上，额外记录请求和响应的头信息。
4. **FULL**: 记录所有请求和响应的明细，包括头信息、正文和元数据。

两种实现方式

1通过配置文件

```yml
feign:
  client:
    config:
      default: # default表示全局配置
        loggerLevel: full # 对所有Feign客户端生效



# 局部配置
feign:
  client:
    config:
      局部微服务: # default表示全局配置
        loggerLevel: full # 对所有Feign客户端生效
```

2通过代码

**步骤：**

1. **第一步：创建配置类**

   首先，创建一个独立的 Java 配置类，用于定义 Feign 的日志级别 Bean。

   ```
   public class FeignClientConfiguration {
   
       @Bean
       public feign.Logger.Level feignLogLevel() {
           // 你可以返回任何一个日志级别：NONE, BASIC, HEADERS, FULL
           return feign.Logger.Level.BASIC;
       }java
   }
   ```

   **注意**：这个配置类不要加上 @Configuration 注解，并且不要放在主启动类能够扫描到的包下，以避免它被意外应用为全局配置。

   ------

2. **第二步：应用配置（二选一）**

   选择将这个配置类应用为全局或局部配置。

   将配置类应用到启动类的 @EnableFeignClients 注解上，通过 defaultConfiguration 属性指定。

   ```java
   // 在你的 Spring Boot 启动类上
   @EnableFeignClients(defaultConfiguration = FeignClientConfiguration.class)
   @SpringBootApplication
   public class YourApplication {
       // ...
   }
   ```

   将配置类应用到具体的 @FeignClient 注解上，通过 configuration 属性指定。

   codeJava

   ```java
   // 在你的 Feign 接口上
   @FeignClient(value = "userservice", configuration = FeignClientConfiguration.class)
   public interface UserClient {
       // ...
   }
   ```

#### Feign 的性能优化 - 连接池配置

默认情况下，Feign 使用 Java 的 HttpURLConnection 进行网络请求，它不支持连接池。在高并发场景下，频繁地创建和销毁 TCP 连接会造成巨大的性能开销。

通过引入 Apache HttpClient，我们可以利用其连接池技术来复用连接，从而显著提升 Feign 的性能。

##### 步骤一：引入 HttpClient 依赖

在 pom.xml 文件中添加 feign-httpclient 的依赖，以替换掉 Feign 默认的 HTTP 客户端。

codeXml

```xml
<!-- 引入 Apache HttpClient 依赖，为 Feign 开启连接池支持 -->
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-httpclient</artifactId>
</dependency>
```

##### 步骤二：配置连接池

在你的 application.yml 配置文件中添加以下配置来启用 HttpClient 并设置连接池参数。

codeYaml

```yml
feign:
  client:
    config:
      default: # "default" 代表这是全局配置，对所有 FeignClient 生效
        loggerLevel: BASIC # (可选) 日志级别
  # HttpClient 的专属配置
  httpclient:
    enabled: true # 开启 Feign 对 HttpClient 的支持
    max-connections: 200 # 连接池的最大总连接数
    max-connections-per-route: 50 # 每个主机(路由)的最大连接数
```

**配置项说明：**

- feign.httpclient.enabled: true
  - **作用**：这个开关是关键，设置为 true 后，Feign 才会使用 Apache HttpClient 代替默认的客户端。
- feign.httpclient.max-connections
  - **作用**：整个连接池允许的最大连接总数。
- feign.httpclient.max-connections-per-route
  - **作用**：分配给单个目标主机（例如 "user-service"）的最大连接数。可以防止某个服务的请求过多而占满整个连接池。

### 统一网关gateway

网关也属于一个微服务，因此我们要自己搭建。

#### 搭建网关服务

##### 搭建网关服务的步骤

1. **创建新模块并引入核心依赖**

在项目中创建一个新的 Maven模块，用于承载网关服务。然后在 pom.xml 文件中引入以下两个核心依赖：

**a. Spring Cloud Gateway 依赖**

这是实现网关功能的基础。

```xml
<!-- 网关依赖 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

**b. Nacos 服务发现依赖**

网关需要通过服务发现来找到后端的微服务，这里使用 Nacos 作为注册中心。

```xml
<!-- nacos服务发现依赖 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

**2. 编写路由配置及 Nacos 地址**

在网关服务的 application.yml 文件中，需要进行以下核心配置：

```yml
server:
  port: 10010 # 1. 设置网关自身的端口号

spring:
  application:
    name: gateway # 2. 设置网关的服务名称，用于注册到Nacos
  cloud:
    # 3. 配置Nacos服务发现
    nacos:
      server-addr: localhost:8848 # Nacos服务器地址

    # 4. 配置网关路由规则
    gateway:
      routes:
        # --- 第一个路由规则 ---
        - id: user-service          # 路由的唯一ID，自定义即可
          uri: lb://userservice     # 路由的目标地址，lb表示负载均衡，后面跟服务名
          predicates:               # 路由断言，即匹配条件
            - Path=/user/**         # 路径匹配规则，所有/user/开头的请求都会匹配
```

##### 过滤器工厂

什么是路由过滤器工厂 (GatewayFilter Factory)？

简单来说，过滤器工厂的作用就是在请求被路由到目标服务**之前**，或者在收到目标服务的响应**之后**，对请求或响应进行**修改和处理**。

局部过滤器

```yml
spring:
  cloud:
    gateway:
      routes: # 网关路由配置
        - id: user-service
          uri: lb://userservice
          predicates:
            - Path=/user/**
          filters: # 过滤器
            - AddRequestHeader=Truth, Itcast is freaking awesome! # 添加请求头
```

默认过滤器

```yml
spring:
  cloud:
    gateway:
      # 1. 路由规则列表
      routes:
        - id: user-service
          uri: lb://userservice
          predicates:
            - Path=/user/**
        - id: order-service
          uri: lb://orderservice
          predicates:
            - Path=/order/**

      # 2. 默认过滤器列表
      # 这里的过滤器会对上面所有的路由（user-service, order-service等）都生效
      default-filters:
        - AddRequestHeader=Truth, Itcast is freaking awesome! # 为所有请求添加一个固定的请求头
```

##### 全局过滤器

需求：定义全局过滤器，拦截请求，判断请求参数是否满足：

- 含`authorization`参数
- `authorization`参数值为`admin`
  满足则放行，否则拦截 。

第一步：在gateway服务中新建一个类

第二步：实现GlobalFliter接口

```java
@Order(-1)//越小优先值越高
@Component
public class AuthorizeFilter implements GlobalFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 1.获取请求参数
        MultiValueMap<String, String> params = exchange.getRequest().getQueryParams();
        // 2.获取authorization参数
        String auth = params.getFirst("authorization");
        // 3.校验
        if ("admin".equals(auth)) {
            // 放行
            return chain.filter(exchange);
        }
        // 4.拦截
        // 4.1.禁止访问
        exchange.getResponse().setStatusCode(HttpStatus.FORBIDDEN);
        // 4.2.结束处理
        return exchange.getResponse().setComplete();
    }
}
```

- **全局过滤器作用**：对所有路由生效，可自定义处理逻辑

- 实现全局过滤器步骤

  ：

  1. 实现`GlobalFilter`接口
  2. 添加`@Order`注解或实现`Ordered`接口
  3. 编写处理逻辑

  #### 全局过滤器 vs 默认过滤器

  | 特性         | **全局过滤器 (GlobalFilter)**              | **默认过滤器 (default-filters)**                     |
  | ------------ | ------------------------------------------ | ---------------------------------------------------- |
  | **实现方式** | **Java 代码**，实现 GlobalFilter 接口      | **YAML 配置文件**                                    |
  | **灵活性**   | **极高**，可以编写任意复杂的业务逻辑       | **有限**，只能使用 Gateway 内置的过滤器工厂          |
  | **适用场景** | 统一鉴权、日志监控、安全校验等**复杂业务** | 添加统一请求头、剥离路径前缀等**简单、标准化的操作** |

##### 过滤器执行顺序规则

1. 过滤器需指定`int`类型`order`值，**值越小优先级越高、执行越靠前**
2. `GlobalFilter`：通过实现`Ordered`接口或加`@Order`注解，自主指定`order`值
3. 路由过滤器、`defaultFilter`：`order`由 Spring 指定，默认按声明顺序从`1`递增
4. `order`值相同时，执行顺序：**defaultFilter > 路由过滤器 > GlobalFilter**

### 解决跨域问题

```yml
spring:
  cloud:
    gateway:
      # 。。。
      globalcors: # 全局的跨域处理
        add-to-simple-url-handler-mapping: true # 解决options请求被拦截问题
        corsConfigurations:
          '[/**]':
            allowedOrigins: # 允许哪些网站的跨域请求
              - "http://localhost:8090"
              - "http://www.leyou.com"
            allowedMethods: # 允许的跨域ajax的请求方式
              - "GET"
              - "POST"
              - "DELETE"
              - "PUT"
              - "OPTIONS"
            allowedHeaders: "*" # 允许在请求中携带的头信息
            allowCredentials: true # 是否允许携带cookie
            maxAge: 360000 # 这次跨域检测的有效期
```

## Day3

### RabbitMq

#### SpringAMQP

使用：

在父工程中导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

##### 利用SpringAMQP去发送消息

1配置 RabbitMQ 连接（`application.yml` 配置）

在 `publisher` 服务的 `application.yml` 中，添加 Spring 对 RabbitMQ 的连接配置：

```yaml
spring:
  rabbitmq:
    host: 192.168.150.101 # 主机名，需根据实际 RabbitMQ 部署地址调整
    port: 5672 # RabbitMQ 的 AMQP 协议默认端口 
    virtual-host: / # 虚拟主机，RabbitMQ 中用于逻辑隔离的概念
    username: itcast # 登录用户名
    password: 123321 # 登录密码 
```

作用：让 Spring Boot 项目能通过这些配置参数，与指定的 RabbitMQ 服务器建立连接 。

2测试

```java
@RunWith(SpringRunner.class) 
@SpringBootTest 
public class SpringAmqpTest {

    @Autowired 
    private RabbitTemplate rabbitTemplate; 

    @Test 
    public void testSimpleQueue() {
        String queueName = "simple.queue"; 
        String message = "hello, spring amqp!"; 
        rabbitTemplate.convertAndSend(queueName, message); 
    }
}
```

#### 利用SpringAMQP去接受消息

1配置 RabbitMQ 连接（`application.yml` 配置）

在 `publisher` 服务的 `application.yml` 中，添加 Spring 对 RabbitMQ 的连接配置：

```yaml
spring:
  rabbitmq:
    host: 192.168.150.101 # 主机名，需根据实际 RabbitMQ 部署地址调整
    port: 5672 # RabbitMQ 的 AMQP 协议默认端口 
    virtual-host: / # 虚拟主机，RabbitMQ 中用于逻辑隔离的概念
    username: itcast # 登录用户名
    password: 123321 # 登录密码 
```

作用：让 Spring Boot 项目能通过这些配置参数，与指定的 RabbitMQ 服务器建立连接 。

2

```java
@Component
public class SpringRabbitListener {

    @RabbitListener(queues = "simple.queue")
    public void listenSimpleQueueMessage(String msg) throws InterruptedException {
    System.out.println("spring 消费者接收到消息 : 【" + msg + "】");
    }
}
```

