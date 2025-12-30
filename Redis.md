### Redis x Spring Boot 笔记目录

- **一、Redis 基础**
  - 1.1 Redis 简介（和数据库、MQ 的区别）
  - 1.2 常用数据类型 + 典型业务场景
  - 1.3 常用基础命令 & 可视化工具
- **二、在 Spring Boot 中集成 Redis**
- **三、Redis 作为缓存**
- **四、常见缓存问题及解决方案**
- **五、Redis 分布式锁**
- **六、Redis 常见业务功能示例**
- **七、Redis 发布订阅 & Spring 集成**
- **八、Redis 进阶能力：持久化、主从、哨兵、集群**
- **九、综合小案例：在 Spring Boot 项目中使用 Redis 优化性能**

# 一、Redis 基础

## 1.1 Redis 简介

- 内存型 key-value 数据库，读写速度快。
- 典型用途：缓存、计数器、排行榜、Session 存储等。

### 1.1.1 和 MySQL、RabbitMQ 的定位区别

- **和 MySQL**：
  - MySQL：长期存储、强一致、复杂查询（SQL、多表关联）。
  - Redis：高并发、简单查询（按 key 直接取值，多种数据结构）。
  - 常见组合：MySQL 负责持久化，Redis 负责加速读取（缓存）。

- **和 MQ（RabbitMQ）**：
  - MQ：消息中间件，强调异步解耦、削峰填谷、消息确认。
  - Redis：数据存储为主，Pub/Sub 功能不保证消息可靠投递。

## 1.2 数据类型 + 场景

### 1.2.1 String

- **特点**：一个 key 对应一个字符串（可以是普通文本、JSON、数字等）。
- **典型场景**：
  - 缓存：`user:1` -> 用户信息 JSON。
  - 计数器：`pv:homepage` -> 浏览量，用 `INCR` 自增。

### 1.2.2 Hash（对象 / Map）

- **特点**：一个 key 对应一个小 Map：`user:1` 下面可以有很多字段。
- **典型场景**：
  - 缓存“对象”的部分字段：
    - `user:1 -> { name: 张三, age: 20, balance: 100.0 }`
  - 适合字段经常单独读写的场景（只改余额，不动其它字段）。

### 1.2.3 List（列表）

- **特点**：类似链表，可以从左/右两端 push/pop。
- **典型场景**：
  - 简单的“消息队列”（不过真正 MQ 还是用 RabbitMQ）。
  - 最近浏览记录、评论列表等。

### 1.2.4 Set（无序去重集合）

- **特点**：不允许重复元素，可以做交集、并集、差集运算。
- **典型场景**：
  - 统计“某日独立访问用户数”（去重）。
  - 做黑名单、关注列表等。

### 1.2.5 ZSet（有序集合）

- **特点**：每个元素有一个 score，Redis 会按 score 排序。
- **典型场景**：
  - 排行榜：积分榜、热度榜。
  - 延迟任务的简单实现（score 当作时间戳）。

### 1.2.6 Bitmap（位图）

- 基于 String 的位操作，每一位只能是 0/1。
- 场景：签到、是否活跃标记、是否完成某个任务等布尔状态统计。

### 1.2.7 HyperLogLog

- 用于**近似**统计去重数量，占用内存极小。
- 场景：UV 统计（独立访客数），对精度要求不那么死的场景。

### 1.2.8 Geo（地理位置）

- 存经纬度，支持附近搜索、距离计算。
- 场景：附近的店铺/骑手/司机等。

### 1.2.9 Stream（流）

- Redis 5.0 引入，用于消息流，支持消费组、持久化等。
- 场景：日志/消息流、简化版消息队列。

## 1.3 常用基础命令 & 可视化工具

### 1.3.1 常用命令

#### 通用 key 操作

```text
KEYS pattern          # 列出所有匹配的 key（线上慎用，大量 key 会很慢）
DEL key [key ...]     # 删除 key
EXPIRE key seconds    # 设置过期时间（秒）
TTL key               # 查看剩余过期时间（-1 表示永不过期）
TYPE key              # 查看 key 的数据类型
EXISTS key            # 判断 key 是否存在
```

#### String

```text
SET key value               # 设置值
SETEX key seconds value     # 设置值并指定过期时间
GET key                     # 获取值
INCR key                    # 自增 1（初始不存在则从 0 开始）
DECR key                    # 自减 1
INCRBY key increment        # 按指定步长自增
APPEND key value            # 追加字符串
```

#### Hash

```text
HSET key field value        # 设置字段
HGET key field              # 获取字段
HGETALL key                 # 获取所有字段和值
HDEL key field [field ...]  # 删除字段
HEXISTS key field           # 判断字段是否存在
HINCRBY key field increment # 对字段做数值自增
```

#### List

```text
LPUSH key value [value ...] # 从左侧推入
RPUSH key value [value ...] # 从右侧推入
LPOP key                    # 从左侧弹出
RPOP key                    # 从右侧弹出
LRANGE key start stop       # 按范围取列表元素
LLEN key                    # 列表长度
```

#### Set

```text
SADD key member [member ...]    # 添加成员
SREM key member [member ...]    # 删除成员
SMEMBERS key                    # 获取所有成员
SISMEMBER key member            # 判断是否是成员
SCARD key                       # 成员数量
SINTER key [key ...]            # 交集
SUNION key [key ...]            # 并集
SDIFF key [key ...]             # 差集
```

#### ZSet

```text
ZADD key score member [score member ...]   # 添加成员及分数
ZREM key member [member ...]              # 删除成员
ZRANGE key start stop WITHSCORES          # 正序按下标范围取（含分数）
ZREVRANGE key start stop WITHSCORES       # 倒序按下标范围取（含分数）
ZSCORE key member                         # 查看成员的 score
ZCARD key                                 # 成员数量
ZINCRBY key increment member              # 给成员的 score 增加指定值
```

#### Bitmap / HyperLogLog / Geo / Stream（只列常用）

```text
# Bitmap（基于 String 的位操作）
SETBIT key offset value     # 设置某一位（0 或 1）
GETBIT key offset           # 读取某一位
BITCOUNT key                # 统计值为 1 的位数量

# HyperLogLog
PFADD key element [element ...]  # 添加元素
PFCOUNT key [key ...]            # 估算基数（去重后的数量）

# Geo
GEOADD key lon lat member   # 添加经纬度
GEOPOS key member           # 查询经纬度
GEODIST key m1 m2 [unit]    # 计算两点距离
GEORADIUS key lon lat radius m WITHDIST WITHCOORD  # 半径范围查询

# Stream
XADD key * field value [field value ...]     # 追加一条消息（* 让 Redis 自动生成 ID）
XRANGE key - + [COUNT n]                     # 正序读取消息
XREVRANGE key + - [COUNT n]                  # 逆序读取消息
XREAD COUNT n STREAMS key id                 # 从指定 ID 后读取消息
```

### 1.3.2 如何连上 Redis？

- **方式一：命令行工具 `redis-cli`**
  - 本地安装 Redis 后，可以直接：
    - `redis-cli -h 127.0.0.1 -p 6379 -a 密码`
  - 连接成功后就可以输入上面的各种命令。

- **方式二：图形化客户端（推荐给入门时观察数据用）**
  - 推荐类型（任选一个你顺手的即可）：
    - RedisInsight
    - AnotherRedisDesktopManager
    - Medis / RDM 等

---

# 二、在 Spring Boot 中集成 Redis

## 2.1 引入依赖

Spring Boot 推荐使用 Spring Data Redis + Lettuce。

```xml
<!-- Spring Data Redis（包含自动配置） -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!-- 可选：如果项目没有 Web 依赖，又需要 JSON，可引入 Jackson -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
```

## 2.2 基础配置：application.yml

```yaml
spring:
  redis:
    host: 127.0.0.1       # Redis 服务器地址
    port: 6379            # 端口
    password:             # 没有密码可留空
    database: 0           # 使用的库，默认 0，范围 0~15
    timeout: 3000ms       # 连接超时时间
    lettuce:
      pool:
        max-active: 8     # 最大连接数
        max-idle: 8       # 最大空闲连接
        min-idle: 0       # 最小空闲连接
        max-wait: 1000ms  # 连接耗尽时的最大等待时间
```

## 2.3 使用 StringRedisTemplate

`StringRedisTemplate` 专门处理 String 类型的 key 和 value，最常用。

### 2.3.1 注入与简单读写

```java
@Service
public class DemoService {

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    // 写入一个字符串，设置 10 分钟过期
    public void setExample() {
        stringRedisTemplate.opsForValue()
                .set("demo:string:key", "hello redis", Duration.ofMinutes(10));
    }

    // 读取字符串
    public String getExample() {
        return stringRedisTemplate.opsForValue().get("demo:string:key");
    }
}
```

### 2.3.2 自增计数器示例

```java
// 统计某接口的访问次数
public long increaseApiCounter(String apiName) {
    String key = "counter:api:" + apiName;
    return stringRedisTemplate.opsForValue().increment(key);
}
```

### 2.3.3 存储 JSON（手动序列化）

```java
@Data
public class UserDTO {
    private Long id;
    private String name;
}

@Service
public class UserCacheService {

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    @Resource
    private ObjectMapper objectMapper;

    private static final String USER_CACHE_KEY_PREFIX = "user:cache:";

    public void saveUser(UserDTO user) throws JsonProcessingException {
        String key = USER_CACHE_KEY_PREFIX + user.getId();
        String json = objectMapper.writeValueAsString(user);
        stringRedisTemplate.opsForValue().set(key, json, Duration.ofMinutes(30));
    }

    public UserDTO getUser(Long userId) throws JsonProcessingException {
        String key = USER_CACHE_KEY_PREFIX + userId;
        String json = stringRedisTemplate.opsForValue().get(key);
        if (json == null) {
            return null;
        }
        return objectMapper.readValue(json, UserDTO.class);
    }
}
```

## 2.4 使用 RedisTemplate（存对象、Hash 等）

`RedisTemplate<Object, Object>` 是更通用的模板，默认使用 JDK 序列化（不推荐直接用默认配置）。

### 2.4.1 配置 RedisTemplate 序列化

```java
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(factory);

        // Key 使用 String 序列化
        StringRedisSerializer stringSerializer = new StringRedisSerializer();
        // Value 使用 JSON 序列化
        Jackson2JsonRedisSerializer<Object> jsonSerializer =
                new Jackson2JsonRedisSerializer<>(Object.class);

        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectMapper.activateDefaultTyping(
                objectMapper.getPolymorphicTypeValidator(),
                ObjectMapper.DefaultTyping.NON_FINAL);
        jsonSerializer.setObjectMapper(objectMapper);

        template.setKeySerializer(stringSerializer);
        template.setHashKeySerializer(stringSerializer);
        template.setValueSerializer(jsonSerializer);
        template.setHashValueSerializer(jsonSerializer);

        template.afterPropertiesSet();
        return template;
    }
}
```

### 2.4.2 使用 RedisTemplate 操作 Hash

```java
@Service
public class ProductCacheService {

    @Resource
    private RedisTemplate<String, Object> redisTemplate;

    private static final String PRODUCT_HASH_KEY = "product:detail";

    // 将商品信息缓存在一个 Hash 里，field 使用商品 ID
    public void saveProduct(ProductDTO product) {
        redisTemplate.opsForHash().put(PRODUCT_HASH_KEY, product.getId().toString(), product);
    }

    public ProductDTO getProduct(Long productId) {
        Object value = redisTemplate.opsForHash().get(PRODUCT_HASH_KEY, productId.toString());
        return value == null ? null : (ProductDTO) value;
    }
}
```

## 2.5 小结：什么时候用哪个模板

- **StringRedisTemplate**：
  - key 和 value 都是字符串；
  - 典型场景：简单缓存、计数器、分布式锁 key、存 JSON 字符串等。
- **RedisTemplate\<String, Object\>（配好 JSON 序列化）**：
  - 需要直接存对象、用 Hash/List/Set/ZSet 等复杂结构时使用。

---

# 三、Redis 作为缓存

## 3.1 典型读流程：先查缓存，再查数据库

目标：减少对数据库的直接访问，把“读多写少”的数据缓存在 Redis 中。

### 3.1.1 伪代码示例（用户信息查询）

```java
@Service
public class UserService {

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    @Resource
    private UserMapper userMapper; // MyBatis / JPA 均可

    private static final String USER_CACHE_KEY_PREFIX = "user:cache:";

    public UserDTO getUserById(Long userId) {
        String key = USER_CACHE_KEY_PREFIX + userId;

        // 1. 先查缓存
        String json = stringRedisTemplate.opsForValue().get(key);
        if (json != null) {
            return JsonUtils.toBean(json, UserDTO.class);
        }

        // 2. 缓存没有，查数据库
        UserDTO user = userMapper.selectById(userId);
        if (user == null) {
            return null;
        }

        // 3. 回填缓存，设置过期时间
        stringRedisTemplate.opsForValue()
                .set(key, JsonUtils.toJson(user), Duration.ofMinutes(30));

        return user;
    }
}
```

## 3.2 写流程：先写数据库，再删/更新缓存

更新数据时，优先保证数据库正确，然后再处理缓存。

### 3.2.1 简单做法：更新后删除缓存

```java
public void updateUser(UserDTO user) {
    // 1. 更新数据库
    userMapper.updateById(user);

    // 2. 删除缓存，下次查询会自动回填
    String key = "user:cache:" + user.getId();
    stringRedisTemplate.delete(key);
}
```

### 3.2.2 或者：更新后直接覆盖缓存

```java
public void updateUserAndCache(UserDTO user) {
    userMapper.updateById(user);

    String key = "user:cache:" + user.getId();
    stringRedisTemplate.opsForValue()
            .set(key, JsonUtils.toJson(user), Duration.ofMinutes(30));
}
```

## 3.3 使用 Spring Cache 注解

Spring 提供了基于注解的缓存抽象，可以将 Redis 作为底层存储。

### 3.3.1 开启缓存功能

```java
@SpringBootApplication
@EnableCaching   // 开启 Spring Cache
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

### 3.3.2 配置 Redis 作为缓存实现

在使用 `spring-boot-starter-data-redis` 时，Spring Boot 会自动配置 `RedisCacheManager`，
只需要在 `application.yml` 中指定一些默认策略。

```yaml
spring:
  cache:
    type: redis
    redis:
      time-to-live: 1800000   # 默认缓存 30 分钟（毫秒）
      cache-null-values: false # 是否缓存 null 值
      use-key-prefix: true    # 是否使用缓存名前缀
```

### 3.3.3 @Cacheable 查询缓存

```java
@Service
@CacheConfig(cacheNames = "user") // 公共 cacheName 前缀
public class UserCacheWithSpringService {

    @Resource
    private UserMapper userMapper;

    // key = "user::" + userId
    @Cacheable(key = "#userId")
    public UserDTO getUserById(Long userId) {
        // 第一次调用会执行 SQL，并把结果放入 Redis
        // 后续相同参数的调用直接走缓存
        return userMapper.selectById(userId);
    }
}
```

### 3.3.4 @CachePut 更新缓存

```java
@CachePut(key = "#user.id")
public UserDTO updateUser(UserDTO user) {
    userMapper.updateById(user);
    // 返回值会被放入缓存
    return user;
}
```

### 3.3.5 @CacheEvict 删除缓存

```java
@CacheEvict(key = "#userId")
public void deleteUser(Long userId) {
    userMapper.deleteById(userId);
    // 删除成功后，对应缓存也被清理
}
```

### 3.3.6 注解方式与手动 Redis 操作对比

- 注解方式（Spring Cache）：
  - 快速、适合简单方法级缓存；
  - key 规则和缓存时间较为统一，通过配置管理。
- 手动使用 `StringRedisTemplate` / `RedisTemplate`：
  - 适合复杂缓存策略（自定义 key、逻辑过期、互斥锁、防击穿等）；
  - 后续章节会在此基础上扩展缓存穿透 / 击穿 / 雪崩的解决方案。

---

# 四、常见缓存问题及解决方案

## 4.1 缓存穿透（查询的都是不存在的数据）

### 4.1.1 现象

不停查询数据库中本来就不存在的 id，比如被恶意脚本刷一堆 `id = -1, 0, 999999999` 这样的请求：

- Redis 查不到（缓存未命中）；
- 每次都要落到数据库，数据库被打爆，缓存形同虚设。

### 4.1.2 方案一：缓存空对象

核心：数据库查不到结果时，也往 Redis 里放一个标记，短期内再次访问直接命中缓存。

```java
public UserDTO getUserByIdWithNullCache(Long userId) {
    String key = "user:cache:nullSafe:" + userId;

    String json = stringRedisTemplate.opsForValue().get(key);
    if (json != null) {
        // 命中缓存（可能是正常对象，也可能是一个特殊空值标记）
        if ("NULL".equals(json)) {
            return null;
        }
        return JsonUtils.toBean(json, UserDTO.class);
    }

    // Redis 没有，查数据库
    UserDTO user = userMapper.selectById(userId);

    if (user == null) {
        // 写入一个空值标记，过期时间可以短一些
        stringRedisTemplate.opsForValue()
                .set(key, "NULL", Duration.ofMinutes(5));
        return null;
    }

    // 正常回填缓存
    stringRedisTemplate.opsForValue()
            .set(key, JsonUtils.toJson(user), Duration.ofMinutes(30));

    return user;
}
```

### 4.1.3 方案二：布隆过滤器（适合 ID 空间巨大的情况）

思路：将所有**可能存在**的 id 预先放入布隆过滤器；请求先查过滤器，如果过滤器判断“不可能存在”，就直接拦截，不访问 Redis 和 DB。

伪代码（以 Redisson 提供的 BloomFilter 为例）：

```java
@Service
public class UserBloomService {

    @Resource
    private RedissonClient redissonClient;

    private RBloomFilter<Long> userBloomFilter;

    @PostConstruct
    public void init() {
        userBloomFilter = redissonClient.getBloomFilter("bf:user:id");
        // 预估插入数量 + 期望误判率
        userBloomFilter.tryInit(1_000_000L, 0.01);
        // 初始化时从数据库加载所有可能存在的 id（可按自己项目情况定）
    }

    public UserDTO getUserById(Long userId) {
        if (!userBloomFilter.contains(userId)) {
            // 过滤器认为“不可能存在”，直接返回
            return null;
        }
        // 后面照常走 Redis + DB 逻辑
        return null;
    }
}
```

> 在中小型项目中，简单的“缓存空对象”通常已经够用。

## 4.2 缓存雪崩（大量 key 同时失效）

### 4.2.1 现象

给大量缓存设置了相同的过期时间（比如都设为 0 点失效）：

- 某一时刻，全部缓存同时失效；
- 大量请求一起打到数据库，导致数据库压力激增。

### 4.2.2 解决方案

1. 给缓存过期时间增加随机值，避免集中在同一时间点失效。
2. 对极其重要的热点数据，可以不设过期时间，采用“逻辑过期 + 后台异步更新”。

#### 4.2.2.1 过期时间加随机

```java
public void cacheWithRandomExpire(String key, String value) {
    // 基础 30 分钟 + 随机 0~10 分钟
    long baseMinutes = 30;
    long randomMinutes = ThreadLocalRandom.current().nextLong(0, 10);
    Duration ttl = Duration.ofMinutes(baseMinutes + randomMinutes);

    stringRedisTemplate.opsForValue().set(key, value, ttl);
}
```

## 4.3 缓存击穿（单个热点 key 失效）

### 4.3.1 现象

某个单 key 非常热点（比如热门商品详情）：

- 有大量并发请求；
- 刚好这个 key 过期的瞬间，大量请求同时打到数据库，可能把数据库压垮。

### 4.3.2 方案一：互斥锁（Mutex）

思路：只有拿到锁的线程可以去查数据库并回填缓存，其它线程要么等待、要么直接返回旧数据。

示例（使用 Redis 的 `SETNX` 实现简单互斥锁）：

```java
public ProductDTO getProductWithMutex(Long productId) {
    String cacheKey = "product:cache:" + productId;
    String lockKey = "lock:product:" + productId;

    // 1. 先查缓存
    String cacheJson = stringRedisTemplate.opsForValue().get(cacheKey);
    if (cacheJson != null) {
        return JsonUtils.toBean(cacheJson, ProductDTO.class);
    }

    // 2. 缓存没命中，尝试加锁
    boolean locked = tryLock(lockKey, 10);
    if (!locked) {
        // 加锁失败，说明其他线程在重建缓存，
        // 可以等待一小会儿后再查一次缓存，也可以直接返回 null/降级数据
        try {
            Thread.sleep(50);
        } catch (InterruptedException ignored) {}
        cacheJson = stringRedisTemplate.opsForValue().get(cacheKey);
        if (cacheJson != null) {
            return JsonUtils.toBean(cacheJson, ProductDTO.class);
        }
        return null;
    }

    try {
        // 3. 再次检查缓存（双重检查，防止重复重建）
        cacheJson = stringRedisTemplate.opsForValue().get(cacheKey);
        if (cacheJson != null) {
            return JsonUtils.toBean(cacheJson, ProductDTO.class);
        }

        // 4. 查数据库，重建缓存
        ProductDTO product = productMapper.selectById(productId);
        if (product != null) {
            stringRedisTemplate.opsForValue()
                    .set(cacheKey, JsonUtils.toJson(product), Duration.ofMinutes(30));
        } else {
            // 防止穿透，缓存空值
            stringRedisTemplate.opsForValue()
                    .set(cacheKey, "NULL", Duration.ofMinutes(5));
        }
        return product;
    } finally {
        // 5. 释放锁
        unlock(lockKey);
    }
}

private boolean tryLock(String key, long expireSeconds) {
    Boolean success = stringRedisTemplate.opsForValue()
            .setIfAbsent(key, "1", Duration.ofSeconds(expireSeconds));
    return Boolean.TRUE.equals(success);
}

private void unlock(String key) {
    stringRedisTemplate.delete(key);
}
```

### 4.3.3 方案二：逻辑过期（热点数据永不过期）

思路：

- Redis 中存放的数据结构中，额外保存一个“逻辑过期时间”；
- 即使物理上 key 不过期，当发现逻辑时间已过期时，先返回旧数据，然后异步触发缓存重建。

简单结构示例：

```json
{
  "data": { ... 商品数据 ... },
  "expireTime": "2025-01-01T12:00:00"
}
```

伪代码：

```java
public ProductDTO getProductWithLogicExpire(Long productId) {
    String cacheKey = "product:cache:logic:" + productId;
    String json = stringRedisTemplate.opsForValue().get(cacheKey);
    if (json == null) {
        return null;
    }

    CacheData cacheData = JsonUtils.toBean(json, CacheData.class);
    ProductDTO data = cacheData.getData();
    LocalDateTime expireTime = cacheData.getExpireTime();

    // 未过期，直接返回
    if (expireTime.isAfter(LocalDateTime.now())) {
        return data;
    }

    // 已过期，尝试异步刷新（加锁防止多个线程同时刷新）
    String lockKey = "lock:product:logic:" + productId;
    if (tryLock(lockKey, 10)) {
        // 提交到线程池异步刷新缓存（伪代码）
        cacheRebuildExecutor.submit(() -> {
            try {
                ProductDTO fresh = productMapper.selectById(productId);
                CacheData newCache = new CacheData();
                newCache.setData(fresh);
                newCache.setExpireTime(LocalDateTime.now().plusMinutes(30));
                stringRedisTemplate.opsForValue()
                        .set(cacheKey, JsonUtils.toJson(newCache));
            } finally {
                unlock(lockKey);
            }
        });
    }

    // 先返回旧数据，不阻塞当前请求
    return data;
}
```

> 对少数超热点 key，可以考虑逻辑过期 + 后台异步刷新；其他普通 key 用常规过期时间 + 随机 TTL 即可。

---

# 五、Redis 分布式锁

## 5.1 为什么需要分布式锁

- 单机应用可以用 `synchronized` / `ReentrantLock` 控制并发。
- 多实例部署（多台机器、多 Pod）时，JVM 内部的锁无法在不同进程之间共享。
- 使用 Redis 提供一个所有实例都能访问的“锁”，保证某段代码在集群中同一时刻只有一个线程在执行。

## 5.2 基础实现：SETNX + EXPIRE

使用 Redis 的 `SETNX`（`SET key value NX`）实现抢锁，并给锁设置过期时间。

### 5.2.1 获取锁

```java
public boolean tryLock(String key, String requestId, long expireSeconds) {
    // value 使用请求唯一标识，用于释放锁时校验
    Boolean success = stringRedisTemplate.opsForValue()
            .setIfAbsent(key, requestId, Duration.ofSeconds(expireSeconds));
    return Boolean.TRUE.equals(success);
}
```

### 5.2.2 释放锁（只删除自己的锁）

```java
public void unlock(String key, String requestId) {
    String value = stringRedisTemplate.opsForValue().get(key);
    if (requestId.equals(value)) {
        stringRedisTemplate.delete(key);
    }
}
```

> “先 GET 再 DEL”在极端情况下不是原子操作，严格场景需要用 Lua 脚本保证原子性。

### 5.2.3 Lua 脚本释放锁（原子）

```java
private static final String UNLOCK_LUA =
        "if redis.call('get', KEYS[1]) == ARGV[1] then " +
        "  return redis.call('del', KEYS[1]) " +
        "else " +
        "  return 0 " +
        "end";

public boolean unlockWithLua(String key, String requestId) {
    DefaultRedisScript<Long> script = new DefaultRedisScript<>();
    script.setScriptText(UNLOCK_LUA);
    script.setResultType(Long.class);

    Long result = stringRedisTemplate.execute(
            script,
            Collections.singletonList(key),
            requestId
    );
    return result != null && result > 0;
}
```

## 5.3 封装一个简单的分布式锁接口

```java
public interface DistributedLock {
    boolean tryLock(long timeout, TimeUnit unit) throws InterruptedException;
    void unlock();
}
```

```java
@Component
public class RedisDistributedLock implements DistributedLock {

    private final StringRedisTemplate stringRedisTemplate;

    private final String lockKey;
    private final String lockValue; // 当前线程持有锁的标识
    private final long expireSeconds;

    public RedisDistributedLock(StringRedisTemplate stringRedisTemplate,
                                String lockKey,
                                long expireSeconds) {
        this.stringRedisTemplate = stringRedisTemplate;
        this.lockKey = lockKey;
        this.expireSeconds = expireSeconds;
        this.lockValue = UUID.randomUUID().toString();
    }

    @Override
    public boolean tryLock(long timeout, TimeUnit unit) throws InterruptedException {
        long deadline = System.currentTimeMillis() + unit.toMillis(timeout);
        while (System.currentTimeMillis() < deadline) {
            Boolean success = stringRedisTemplate.opsForValue()
                    .setIfAbsent(lockKey, lockValue, Duration.ofSeconds(expireSeconds));
            if (Boolean.TRUE.equals(success)) {
                return true;
            }
            // 失败稍等一会儿再重试
            Thread.sleep(50);
        }
        return false;
    }

    @Override
    public void unlock() {
        DefaultRedisScript<Long> script = new DefaultRedisScript<>();
        script.setScriptText(
                "if redis.call('get', KEYS[1]) == ARGV[1] then " +
                "  return redis.call('del', KEYS[1]) " +
                "else " +
                "  return 0 " +
                "end"
        );
        script.setResultType(Long.class);
        stringRedisTemplate.execute(
                script,
                Collections.singletonList(lockKey),
                lockValue
        );
    }
}
```

示例使用（控制某段业务在集群中串行执行）：

```java
@Service
public class OrderCreateService {

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    public void createOrderWithLock(Long userId) throws InterruptedException {
        // 实际项目中可以封装工厂方法创建锁实例
        RedisDistributedLock lock = new RedisDistributedLock(
                stringRedisTemplate,
                "lock:order:create:" + userId,
                10
        );

        if (!lock.tryLock(3, TimeUnit.SECONDS)) {
            // 未获取到锁，可以提示“操作太频繁”或直接返回
            return;
        }

        try {
            // 执行业务逻辑，如扣减库存、生成订单等
        } finally {
            lock.unlock();
        }
    }
}
```

## 5.4 使用 Redisson 简化分布式锁

Redisson 对基于 Redis 的分布式锁做了完整封装，包含锁续约、超时重试等功能，适合生产环境使用。

### 5.4.1 引入依赖

```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
    <version>3.27.0</version> <!-- 版本号根据实际情况选择 -->
</dependency>
```

### 5.4.2 简单使用示例

```java
@Service
public class StockService {

    @Resource
    private RedissonClient redissonClient;

    public void deductStock(Long productId) {
        RLock lock = redissonClient.getLock("lock:stock:" + productId);
        try {
            // 尝试获取锁，最多等待 3 秒，锁持有时间 10 秒（有看门狗续期）
            boolean locked = lock.tryLock(3, 10, TimeUnit.SECONDS);
            if (!locked) {
                // 获取锁失败，直接返回或提示稍后重试
                return;
            }

            // 执行业务逻辑，减库存等

        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            if (lock.isHeldByCurrentThread()) {
                lock.unlock();
            }
        }
    }
}
```

## 5.5 小结

- 自己基于 `SETNX + EXPIRE + Lua` 能实现最小可用的分布式锁，适合简单场景或学习原理。
- 生产环境更推荐使用 Redisson 这类成熟组件，减少细节错误（锁续约、异常情况处理等）。

---

# 六、Redis 常见业务功能示例

## 6.1 计数器：接口调用次数、点赞数、浏览量

### 6.1.1 接口调用计数

```java
@Service
public class ApiCounterService {

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    private static final String API_COUNTER_KEY_PREFIX = "counter:api:";

    public long incrementApiCount(String apiName) {
        String key = API_COUNTER_KEY_PREFIX + apiName;
        return stringRedisTemplate.opsForValue().increment(key);
    }

    public long getApiCount(String apiName) {
        String key = API_COUNTER_KEY_PREFIX + apiName;
        String val = stringRedisTemplate.opsForValue().get(key);
        return val == null ? 0L : Long.parseLong(val);
    }
}
```

### 6.1.2 点赞数 / 浏览量

```java
public void likeArticle(Long articleId) {
    String key = "article:like:" + articleId;
    stringRedisTemplate.opsForValue().increment(key);
}

public long getArticleLikeCount(Long articleId) {
    String key = "article:like:" + articleId;
    String val = stringRedisTemplate.opsForValue().get(key);
    return val == null ? 0L : Long.parseLong(val);
}
```

## 6.2 排行榜：积分榜 / 热度榜（ZSet）

### 6.2.1 更新排行榜分数

```java
@Service
public class RankService {

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    private static final String RANK_KEY = "rank:score";

    // 增加用户积分
    public void addScore(Long userId, double delta) {
        stringRedisTemplate.opsForZSet()
                .incrementScore(RANK_KEY, userId.toString(), delta);
    }

    // 设置用户积分
    public void setScore(Long userId, double score) {
        stringRedisTemplate.opsForZSet()
                .add(RANK_KEY, userId.toString(), score);
    }
}
```

### 6.2.2 查询 Top N 名

```java
public List<String> topN(int n) {
    Set<String> range = stringRedisTemplate.opsForZSet()
            .reverseRange(RANK_KEY, 0, n - 1);
    if (range == null) {
        return Collections.emptyList();
    }
    return new ArrayList<>(range);
}
```

### 6.2.3 查询某个用户的名次和分数

```java
public Long getRank(Long userId) {
    Long rank = stringRedisTemplate.opsForZSet()
            .reverseRank(RANK_KEY, userId.toString());
    return rank == null ? null : rank + 1; // 排名从 1 开始
}

public Double getScore(Long userId) {
    return stringRedisTemplate.opsForZSet()
            .score(RANK_KEY, userId.toString());
}
```

## 6.3 Session / Token 存储

### 6.3.1 简单登录态

```java
@Service
public class SessionService {

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    private static final String TOKEN_KEY_PREFIX = "session:token:";

    // 登录成功后创建 token
    public String createSession(Long userId) {
        String token = UUID.randomUUID().toString();
        String key = TOKEN_KEY_PREFIX + token;
        stringRedisTemplate.opsForValue()
                .set(key, userId.toString(), Duration.ofHours(2));
        return token;
    }

    // 根据 token 获取用户 ID
    public Long getUserIdByToken(String token) {
        String key = TOKEN_KEY_PREFIX + token;
        String val = stringRedisTemplate.opsForValue().get(key);
        if (val == null) {
            return null;
        }
        // 刷新过期时间（滑动过期，可选）
        stringRedisTemplate.expire(key, Duration.ofHours(2));
        return Long.parseLong(val);
    }

    // 登出
    public void logout(String token) {
        String key = TOKEN_KEY_PREFIX + token;
        stringRedisTemplate.delete(key);
    }
}
```

## 6.4 简单限流（固定窗口）

### 6.4.1 固定窗口限流示例（按 IP 或用户）

限制：某个 key 在 1 分钟内最多访问 N 次。

```java
@Service
public class RateLimitService {

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    /**
     * @param keyPrefix  业务前缀，例如 "rate:login:"
     * @param identifier 唯一标识，例如 IP / userId
     * @param windowSec  时间窗口（秒）
     * @param maxCount   窗口内最大访问次数
     */
    public boolean tryAcquire(String keyPrefix, String identifier,
                              int windowSec, int maxCount) {
        String key = keyPrefix + identifier;

        Long current = stringRedisTemplate.opsForValue().increment(key);
        if (current == null) {
            return true;
        }

        if (current == 1L) {
            // 第一次访问时设置过期时间
            stringRedisTemplate.expire(key, Duration.ofSeconds(windowSec));
        }

        return current <= maxCount;
    }
}
```

使用示例：

```java
public boolean login(String ip, String username, String password) {
    boolean allowed = rateLimitService.tryAcquire("rate:login:", ip, 60, 10);
    if (!allowed) {
        // 拒绝请求：该 IP 在 60 秒内登录次数超过 10 次
        return false;
    }
    // 继续执行登录逻辑
    return true;
}
```

---

# 七、Redis 发布订阅 & Spring 集成

## 7.1 Pub/Sub 基础

- Redis 提供发布订阅机制：`PUBLISH channel message`、`SUBSCRIBE channel`。
- 不保证消息持久化，订阅者如果不在线会直接错过消息。

## 7.2 Spring Boot 中使用 Pub/Sub

### 7.2.1 定义消息监听器

```java
@Component
public class RedisMessageSubscriber implements MessageListener {

    @Override
    public void onMessage(Message message, byte[] pattern) {
        String channel = new String(message.getChannel(), StandardCharsets.UTF_8);
        String body = new String(message.getBody(), StandardCharsets.UTF_8);
        // 处理消息
        System.out.println("收到频道 " + channel + " 的消息：" + body);
    }
}
```

### 7.2.2 配置监听容器

```java
@Configuration
public class RedisPubSubConfig {

    @Bean
    public RedisMessageListenerContainer redisMessageListenerContainer(
            RedisConnectionFactory connectionFactory,
            RedisMessageSubscriber subscriber) {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);
        // 订阅频道 "topic:demo"
        Topic topic = new ChannelTopic("topic:demo");
        container.addMessageListener(subscriber, topic);
        return container;
    }
}
```

### 7.2.3 发送消息

```java
@Service
public class PubSubService {

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    public void publishDemo(String msg) {
        stringRedisTemplate.convertAndSend("topic:demo", msg);
    }
}
```

## 7.3 与 RabbitMQ 的简单对比

- Redis Pub/Sub：
  - 订阅者必须在线才能收到消息；
  - 不支持可靠投递、消费确认、消息堆积；
  - 适合简单的通知、配置刷新等。
- RabbitMQ：
  - 支持确认、重试、堆积、路由策略等；
  - 适合重要业务消息、跨服务异步解耦。

---

# 八、Redis 进阶能力：持久化、主从、哨兵、集群

## 8.1 持久化：RDB / AOF

### 8.1.1 RDB（快照）

- 定时把内存数据以快照形式写入磁盘（`.rdb` 文件）。
- 优点：性能好，文件小，适合备份。
- 缺点：故障时会丢失最近一段时间的数据（快照间隔内的修改）。

### 8.1.2 AOF（Append Only File）

- 将写命令以追加方式记录到日志文件（`.aof`）。
- 优点：数据丢失窗口更小（根据 `appendfsync` 策略），可读性好。
- 缺点：文件体积较大，需要定期 rewrite。

### 8.1.3 常见组合

- 一般启用 RDB + AOF：
  - RDB 作为完整快照；
  - AOF 记录快照之后的增量写操作。

## 8.2 主从复制 & 哨兵

### 8.2.1 主从复制

- 一个主节点负责写，从节点负责读；
- 从节点通过复制保持数据同步，用于读扩展和故障切换预备。

### 8.2.2 哨兵（Sentinel）

- 监控多个 Redis 实例；
- 主节点故障时自动选举新主并通知客户端；
- 提供高可用能力。

在 Spring Boot 中，可通过配置多个节点地址 + Redisson 或 Lettuce 的集群/哨兵配置来接入。

## 8.3 集群模式

- 将数据按 slot（0~16383）分布在多个节点上，实现水平扩展；
- 客户端按照 key 计算 slot，自动路由到对应节点；
- 一般用于数据量很大或 QPS 极高的场景。

在代码层面，多数情况下只需要在配置中提供集群节点地址，由客户端驱动（Lettuce/Redisson）负责路由。

---

# 九、综合小案例：在 Spring Boot 项目中使用 Redis 优化性能

## 9.1 场景：商品详情 + 下单

- 商品详情接口：读多写少，数据来自 MySQL。
- 下单接口：涉及库存扣减，需要避免超卖和重复下单。

## 9.2 商品详情：缓存 + 防击穿

### 9.2.1 读流程

```java
@Service
public class ProductDetailService {

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    @Resource
    private ProductMapper productMapper;

    public ProductDTO getProductDetail(Long productId) {
        String key = "product:detail:" + productId;

        // 1. 查缓存
        String json = stringRedisTemplate.opsForValue().get(key);
        if (json != null) {
            if ("NULL".equals(json)) {
                return null;
            }
            return JsonUtils.toBean(json, ProductDTO.class);
        }

        // 2. 缓存未命中，查数据库
        ProductDTO product = productMapper.selectById(productId);
        if (product == null) {
            // 防穿透：缓存空值，设置较短过期时间
            stringRedisTemplate.opsForValue()
                    .set(key, "NULL", Duration.ofMinutes(5));
            return null;
        }

        // 3. 回填缓存，设置过期时间 + 随机
        long baseMinutes = 30;
        long randomMinutes = ThreadLocalRandom.current().nextLong(0, 10);
        Duration ttl = Duration.ofMinutes(baseMinutes + randomMinutes);
        stringRedisTemplate.opsForValue()
                .set(key, JsonUtils.toJson(product), ttl);

        return product;
    }
}
```

## 9.3 下单：分布式锁 + 计数器

### 9.3.1 简化版下单逻辑

```java
@Service
public class OrderService {

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    @Resource
    private ProductMapper productMapper;

    @Resource
    private OrderMapper orderMapper;

    public boolean createOrder(Long userId, Long productId) throws InterruptedException {
        // 1. 基于用户 + 商品的锁，防止同一用户重复下单
        String lockKey = "lock:order:" + userId + ":" + productId;
        String lockValue = UUID.randomUUID().toString();
        Boolean success = stringRedisTemplate.opsForValue()
                .setIfAbsent(lockKey, lockValue, Duration.ofSeconds(10));
        if (!Boolean.TRUE.equals(success)) {
            return false;
        }

        try {
            // 2. 查询库存（可结合 Redis 预减库存等方案，这里用 DB 简化）
            ProductDTO product = productMapper.selectById(productId);
            if (product == null || product.getStock() <= 0) {
                return false;
            }

            // 3. 扣减库存（使用乐观锁 WHERE stock > 0 防止并发超卖）
            int rows = productMapper.deductStock(productId);
            if (rows == 0) {
                return false;
            }

            // 4. 创建订单
            Order order = new Order();
            order.setUserId(userId);
            order.setProductId(productId);
            orderMapper.insert(order);

            // 5. 订单数统计（计数器示例）
            String counterKey = "counter:order:product:" + productId;
            stringRedisTemplate.opsForValue().increment(counterKey);

            return true;
        } finally {
            // 6. 释放锁（简单版，严格可用 Lua 校验 value）
            String value = stringRedisTemplate.opsForValue().get(lockKey);
            if (lockValue.equals(value)) {
                stringRedisTemplate.delete(lockKey);
            }
        }
    }
}
```

## 9.4 小结

- 读侧：商品详情通过 Redis 缓存 + 防穿透 + 防雪崩，减轻数据库压力。
- 写侧：下单过程使用 Redis 分布式锁 + 数据库乐观锁，避免重复下单和超卖，并用 Redis 计数器统计业务指标。 
