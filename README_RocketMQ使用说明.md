# RocketMQ消息队列配置和使用说明

## 一、RocketMQ配置详解

### 1. 基础配置（在application.yml中）

```yaml
rocketmq:
  name-server: 10.1.230.27:9876        # RocketMQ NameServer地址
  isVIPChannel: false                  # 是否使用VIP通道（通常设为false）
  producer:
    group: lili_group                  # 生产者组名
    send-message-timeout: 30000        # 发送消息超时时间（毫秒），30秒
```

**配置说明：**
- **name-server**: RocketMQ的NameServer地址，用于服务发现和路由
  - 格式：`IP:端口`
  - 默认端口：9876
  - 这是RocketMQ的核心组件，管理所有Broker的路由信息

- **isVIPChannel**: VIP通道开关
  - `false`: 使用普通通道（推荐）
  - `true`: 使用VIP通道（需要Broker支持）

- **producer.group**: 生产者组名
  - 用于标识一组生产者
  - 同一个组内的生产者可以共享一些配置
  - 用于事务消息和消息回查

- **send-message-timeout**: 发送消息超时时间
  - 单位：毫秒
  - 30000 = 30秒
  - 如果30秒内消息未发送成功，会抛出超时异常

### 2. 业务Topic和Group配置（在application.yml中）

```yaml
lili:
  data:
    rocketmq:
      # 促销相关
      promotion-topic: shop_lili_promotion_topic      # 促销主题
      promotion-group: shop_lili_promotion_group       # 促销消费者组
      
      # 订单相关
      order-topic: shop_lili_order_topic              # 订单主题
      order-group: shop_lili_order_group               # 订单消费者组
      
      # 商品相关
      goods-topic: shop_lili_goods_topic               # 商品主题
      goods-group: shop_lili_goods_group              # 商品消费者组
      
      # 会员相关
      member-topic: shop_lili_member_topic              # 会员主题
      member-group: shop_lili_member_group             # 会员消费者组
      
      # 店铺相关
      store-topic: lili_store_topic                    # 店铺主题
      store-group: lili_store_group                     # 店铺消费者组
      
      # 消息扩展
      msg-ext-topic: shop_lili_msg_topic                # 消息扩展主题
      msg-ext-group: shop_lili_msg_group               # 消息扩展消费者组
      
      # 其他
      other-topic: shop_lili_other_topic               # 其他主题
      other-group: shop_lili_other_group                # 其他消费者组
      
      # 通知相关
      notice-topic: shop_lili_notice_topic              # 通知主题
      notice-group: shop_lili_notice_group              # 通知消费者组
      notice-send-topic: shop_lili_send_notice_topic   # 通知发送主题
      notice-send-group: shop_lili_send_notice_group   # 通知发送消费者组
      
      # 售后相关
      after-sale-topic: shop_lili_after_sale_topic     # 售后主题
      after-sale-group: shop_lili_after_sale_group     # 售后消费者组
```

**配置说明：**

#### Topic（主题）
- **作用**：消息的分类，类似于数据库的表
- **命名规则**：`业务模块_topic`
- **示例**：`shop_lili_order_topic` 表示订单相关的所有消息

#### Group（消费者组）
- **作用**：标识一组消费者，用于负载均衡和消息分发
- **命名规则**：通常与Topic对应，`业务模块_group`
- **示例**：`shop_lili_order_group` 表示处理订单消息的消费者组

**Topic和Group的关系：**
- 一个Topic可以有多个消费者组（Group）
- 每个消费者组都会收到Topic的所有消息（广播模式）
- 同一个消费者组内的多个消费者会负载均衡消费消息（集群模式）

## 二、配置类详解

### RocketmqCustomProperties.java

这个类用于读取配置文件中的Topic和Group配置：

```java
@Component
@ConfigurationProperties(prefix = "lili.data.rocketmq")
public class RocketmqCustomProperties {
    private String promotionTopic;    // 促销主题
    private String promotionGroup;    // 促销消费者组
    private String orderTopic;        // 订单主题
    private String orderGroup;        // 订单消费者组
    // ... 其他配置
}
```

**作用：**
- 通过`@ConfigurationProperties`注解自动绑定配置文件中的值
- 在代码中可以通过`rocketmqCustomProperties.getOrderTopic()`获取配置的Topic名称
- 统一管理所有Topic和Group，避免硬编码

## 三、如何发送消息

### 方式1：直接发送（最常用）

```java
@Service
public class YourService {
    
    @Autowired
    private RocketMQTemplate rocketMQTemplate;  // RocketMQ模板
    
    @Autowired
    private RocketmqCustomProperties rocketmqCustomProperties;  // 配置类
    
    public void sendMessage() {
        // 1. 构建destination（目标地址）
        // 格式：Topic:Tag
        String destination = rocketmqCustomProperties.getOrderTopic() 
                          + ":" 
                          + OrderTagsEnum.ORDER_CREATE.name();
        
        // 2. 准备消息内容（可以是任何对象）
        Order order = new Order();
        order.setOrderId("123456");
        order.setAmount(100.00);
        
        // 3. 异步发送消息
        rocketMQTemplate.asyncSend(
            destination,                                    // 目标地址
            order,                                         // 消息内容
            RocketmqSendCallbackBuilder.commonCallback()   // 回调函数
        );
    }
}
```

**关键点：**
- **destination格式**：`Topic:Tag`
  - Topic：消息主题（如：`shop_lili_order_topic`）
  - Tag：消息标签，用于消息过滤（如：`ORDER_CREATE`）
  - 用冒号`:`分隔

- **asyncSend方法**：异步发送，不阻塞当前线程
  - 参数1：destination（目标地址）
  - 参数2：message（消息内容，可以是对象、字符串等）
  - 参数3：callback（回调函数，用于处理发送结果）

### 方式2：事务提交后发送（保证数据一致性）

```java
@Service
public class MemberService {
    
    @Autowired
    private ApplicationEventPublisher applicationEventPublisher;
    
    @Autowired
    private RocketmqCustomProperties rocketmqCustomProperties;
    
    @Transactional
    public void registerMember(Member member) {
        // 1. 保存会员到数据库
        this.save(member);
        
        // 2. 发布事务提交事件（事务提交后才会发送MQ）
        applicationEventPublisher.publishEvent(
            new TransactionCommitSendMQEvent(
                "new member register",                    // 描述
                rocketmqCustomProperties.getMemberTopic(), // Topic
                MemberTagsEnum.MEMBER_REGISTER.name(),    // Tag
                member                                    // 消息内容
            )
        );
        // 注意：消息会在事务提交成功后自动发送
    }
}
```

**工作原理：**
1. 使用Spring的`@Transactional`注解开启事务
2. 发布`TransactionCommitSendMQEvent`事件
3. `TransactionCommitSendMQListener`监听器会在事务提交后自动发送消息
4. 如果事务回滚，消息不会发送（保证数据一致性）

## 四、实际项目中的使用示例

### 示例1：店铺设置修改后发送消息

```java
// StoreDetailServiceImpl.java
@Override
public boolean editStoreSetting(StoreSettingDTO storeSettingDTO) {
    // 1. 更新店铺设置
    Store store = this.getById(storeSettingDTO.getStoreId());
    BeanUtil.copyProperties(storeSettingDTO, store);
    boolean result = this.updateById(store);
    
    if (result) {
        // 2. 清除缓存
        this.removeCache(store.getId());
        
        // 3. 发送MQ消息，通知其他系统店铺信息已更新
        String destination = rocketmqCustomProperties.getStoreTopic() 
                          + ":" 
                          + StoreTagsEnum.EDIT_STORE_SETTING.name();
        
        rocketMQTemplate.asyncSend(
            destination, 
            store, 
            RocketmqSendCallbackBuilder.commonCallback()
        );
    }
    
    return result;
}
```

**说明：**
- 当店铺设置修改后，发送消息到`lili_store_topic`主题
- Tag为`EDIT_STORE_SETTING`（修改店铺设置）
- 其他系统（如商品系统）可以监听这个消息，同步更新相关数据

### 示例2：会员注册后发送消息

```java
// MemberServiceImpl.java
@Override
public void registerMember(Member member) {
    // 保存会员
    this.save(member);
    
    // 发送会员注册消息（事务提交后发送）
    applicationEventPublisher.publishEvent(
        new TransactionCommitSendMQEvent(
            "new member register",
            rocketmqCustomProperties.getMemberTopic(),
            MemberTagsEnum.MEMBER_REGISTER.name(),
            member
        )
    );
}
```

**说明：**
- 使用事务事件，确保数据库保存成功后才发送消息
- 如果保存失败，消息不会发送

### 示例3：发送站内消息

```java
// MessageServiceImpl.java
@Override
@Transactional(rollbackFor = Exception.class)
public Boolean sendMessage(Message message) {
    // 1. 保存站内信到数据库
    this.save(message);
    
    // 2. 发送消息提醒
    String noticeSendDestination = rocketmqCustomProperties.getNoticeSendTopic() 
                                 + ":" 
                                 + OtherTagsEnum.MESSAGE.name();
    
    rocketMQTemplate.asyncSend(
        noticeSendDestination, 
        message, 
        RocketmqSendCallbackBuilder.commonCallback()
    );
    
    return true;
}
```

## 五、消息标签（Tag）枚举

项目中使用枚举来管理Tag，避免硬编码：

### StoreTagsEnum（店铺标签）
```java
public enum StoreTagsEnum {
    EDIT_STORE_SETTING("修改商家设置");
}
```

### MemberTagsEnum（会员标签）
```java
public enum MemberTagsEnum {
    MEMBER_REGISTER("会员注册"),
    MEMBER_LOGIN("会员登录"),
    MEMBER_INFO_EDIT("会员信息更改"),
    MEMBER_POINT_CHANGE("会员积分变动"),
    MEMBER_WITHDRAWAL("会员提现");
}
```

### OrderTagsEnum（订单标签）
```java
// 订单相关的各种操作标签
```

**Tag的作用：**
- 用于消息过滤，消费者可以只订阅特定Tag的消息
- 例如：只处理`MEMBER_REGISTER`标签的消息，忽略其他会员相关消息

## 六、回调函数

### RocketmqSendCallback

```java
public class RocketmqSendCallback implements SendCallback {
    @Override
    public void onSuccess(SendResult sendResult) {
        log.info("消息发送成功：{}", sendResult);
    }
    
    @Override
    public void onException(Throwable throwable) {
        log.error("消息发送失败", throwable);
    }
}
```

**作用：**
- 异步发送消息后，RocketMQ会回调这个接口
- `onSuccess`：消息发送成功时调用
- `onException`：消息发送失败时调用

## 七、消息发送的完整流程

```
1. 业务代码调用
   ↓
2. 构建destination（Topic:Tag）
   ↓
3. 调用rocketMQTemplate.asyncSend()
   ↓
4. RocketMQ客户端发送消息到Broker
   ↓
5. Broker存储消息
   ↓
6. 回调onSuccess或onException
   ↓
7. 消费者从Broker拉取消息并处理
```

## 八、配置总结

### 必须配置项
1. **name-server**：RocketMQ服务器地址
2. **producer.group**：生产者组名
3. **各业务Topic和Group**：根据业务需要配置

### 推荐配置
1. **send-message-timeout**：根据网络情况调整
2. **isVIPChannel**：通常设为false

### 注意事项
1. Topic和Group名称要唯一，避免冲突
2. 使用枚举管理Tag，不要硬编码
3. 重要消息使用事务事件，保证数据一致性
4. 异步发送不会阻塞，适合高并发场景

## 九、常见问题

### Q1: Topic和Group有什么区别？
A: 
- **Topic**：消息的分类，类似于"订单消息"、"商品消息"
- **Group**：消费者的分组，多个消费者可以组成一个Group，实现负载均衡

### Q2: Tag是什么？
A: Tag是消息的标签，用于进一步细分消息类型。例如：
- Topic: `shop_lili_order_topic`
- Tag: `ORDER_CREATE`（创建订单）、`ORDER_PAY`（支付订单）

### Q3: 为什么要用事务事件发送消息？
A: 保证数据一致性。如果数据库保存失败，消息不会发送；只有事务提交成功后才发送消息。

### Q4: asyncSend和syncSend有什么区别？
A:
- **asyncSend**：异步发送，不等待结果，性能好（推荐）
- **syncSend**：同步发送，等待结果，会阻塞线程

### Q5: 消息发送失败怎么办？
A: 在`onException`回调中处理，可以记录日志、重试或告警。

