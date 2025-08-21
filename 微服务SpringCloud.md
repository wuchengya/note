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