# 管理员端消息机制详解

## 一、核心问题：管理员如何知道有商家申请需要审核？

**答案：通过轮询查询数据库，不是实时推送！**

## 二、完整流程

### 1. 用户申请成为商家

```java
// 位置：StoreServiceImpl.java - applyThirdStep方法
@Override
public boolean applyThirdStep(StoreOtherInfoDTO storeOtherInfoDTO) {
    // ... 填写店铺信息

    // 最后一步：设置店铺状态为"申请中"
    store.setStoreDisable(StoreStatusEnum.APPLYING.name());
    return this.updateById(store);
}
```

**关键点：**

- 店铺状态设置为 `APPLYING`（申请中）
- 数据保存到数据库的 `Store` 表

### 2. 管理员端获取待审核数量

#### 接口位置

```
GET /manager/statistics/index/notice
```

#### 控制器代码

```java
// 位置：IndexStatisticsManagerController.java
@ApiOperation(value = "通知提示信息")
@GetMapping("/notice")
public ResultMessage<IndexNoticeVO> notice() {
    return ResultUtil.data(indexStatisticsService.indexNotice());
}
```

#### 返回数据结构

```java
// IndexNoticeVO.java
{
    "goods": 5,              // 待处理商品审核数量
    "store": 3,              // 待处理店铺入驻审核数量 ⭐
    "refund": 10,            // 待处理售后申请数量
    "complain": 2,           // 待处理投诉审核数量
    "distributionCash": 1,   // 待处理分销员提现申请数量
    "waitPayBill": 8         // 待处理商家结算数量
}
```

#### 统计逻辑

```java
// 位置：IndexStatisticsServiceImpl.java
@Override
public IndexNoticeVO indexNotice() {
    IndexNoticeVO indexNoticeVO = new IndexNoticeVO();

    // 商品审核数量
    indexNoticeVO.setGoods(
        goodsStatisticsService.goodsNum(null, GoodsAuthEnum.TOBEAUDITED)
    );

    // ⭐ 店铺入驻审核数量（关键）
    indexNoticeVO.setStore(
        storeStatisticsService.auditNum()
    );

    // 其他统计...
    return indexNoticeVO;
}
```

#### 店铺审核数量统计

```java
// 位置：StoreStatisticsServiceImpl.java
@Override
public long auditNum() {
    LambdaQueryWrapper<Store> queryWrapper = Wrappers.lambdaQuery();
    // 查询状态为"申请中"的店铺数量
    queryWrapper.eq(Store::getStoreDisable, StoreStatusEnum.APPLYING.name());
    return this.count(queryWrapper);
}
```

**关键点：**

- 直接查询数据库，统计状态为 `APPLYING` 的店铺数量
- **这是轮询方式，不是实时推送**

### 3. 前端如何获取消息

前端（管理后台页面）会定时调用 `/notice` 接口：

```javascript
// 前端伪代码示例
setInterval(() => {
    // 每30秒查询一次待处理消息数量
    fetch('/manager/statistics/index/notice')
        .then(res => res.json())
        .then(data => {
            // 更新页面上的消息提示
            updateNoticeBadge(data.store);  // 显示待审核店铺数量
        });
}, 30000);  // 30秒轮询一次
```

**技术特点：**

- ✅ **轮询（Polling）**：前端定时请求接口
- ❌ **不是WebSocket**：没有实时推送
- ❌ **不是SSE**：没有服务器推送事件

## 三、技术实现总结

### 使用的技术栈

1. **后端统计**：MyBatis-Plus 查询数据库
2. **前端获取**：HTTP REST API（轮询）
3. **数据存储**：MySQL数据库

### 消息获取方式对比

| 方式                          | 本项目使用 | 说明             |
| --------------------------- | ----- | -------------- |
| **轮询（Polling）**             | ✅ 是   | 前端定时请求接口       |
| **WebSocket**               | ❌ 否   | 用于IM聊天，不用于审核消息 |
| **SSE（Server-Sent Events）** | ❌ 否   | 未使用            |
| **RocketMQ推送**              | ❌ 否   | 用于业务消息，不用于前端通知 |

### 为什么使用轮询而不是实时推送？

1. **简单可靠**：不需要维护WebSocket连接
2. **实现简单**：只需要一个查询接口
3. **兼容性好**：所有浏览器都支持HTTP请求
4. **适合场景**：审核消息不需要毫秒级实时性

## 四、相关代码位置

### 1. 商家申请

- **文件**：`framework/src/main/java/cn/lili/modules/store/serviceimpl/StoreServiceImpl.java`
- **方法**：`applyThirdStep()` - 提交商家申请

### 2. 统计服务

- **文件**：    `framework/src/main/java/cn/lili/modules/statistics/serviceimpl/IndexStatisticsServiceImpl.java`
- **方法**：`indexNotice()` - 获取待处理消息数量

### 3. 店铺统计

- **文件**：`framework/src/main/java/cn/lili/modules/statistics/serviceimpl/StoreStatisticsServiceImpl.java`
- **方法**：`auditNum()` - 统计待审核店铺数量

### 4. 接口控制器

- **文件**：`manager-api/src/main/java/cn/lili/controller/statistics/IndexStatisticsManagerController.java`
- **接口**：`GET /manager/statistics/index/notice`

### 5. 数据模型

- **文件**：`framework/src/main/java/cn/lili/modules/statistics/entity/vo/IndexNoticeVO.java`
- **说明**：返回的消息数量对象

## 五、完整流程图

```
用户提交商家申请
    ↓
店铺状态设置为 APPLYING
    ↓
数据保存到数据库 Store 表
    ↓
前端定时轮询（每30秒）
    ↓
调用 GET /manager/statistics/index/notice
    ↓
后端查询数据库统计待审核数量
    ↓
返回 JSON: { "store": 3 }
    ↓
前端更新页面消息提示
```

## 六、其他类型的消息

除了店铺审核，还有其他类型的待处理消息：

1. **商品审核**：`goodsStatisticsService.goodsNum(null, GoodsAuthEnum.TOBEAUDITED)`
2. **售后申请**：`afterSaleStatisticsService.applyNum(null)`
3. **投诉审核**：`orderComplaintStatisticsService.waitComplainNum()`
4. **分销员提现**：`distributionCashStatisticsService.newDistributionCash()`
5. **商家结算**：`billStatisticsService.billNum(BillStatusEnum.CHECK)`

**所有这些都是通过轮询数据库统计实现的！**

## 七、注意事项

1. **轮询频率**：前端需要合理设置轮询间隔，太频繁会增加服务器压力
2. **数据一致性**：由于是轮询，可能存在短暂的数据延迟
3. **性能优化**：如果待处理数量很大，可以考虑添加缓存

## 八、总结

**管理员端接收消息的技术：**

- ✅ **轮询查询**：前端定时调用接口
- ✅ **数据库统计**：后端查询数据库统计数量
- ❌ **不是实时推送**：没有使用WebSocket或SSE
- ❌ **不是消息队列推送**：RocketMQ用于业务消息，不用于前端通知

**简单来说：**
管理员端通过**定时查询数据库**来获取待处理消息数量，而不是通过实时推送技术。
