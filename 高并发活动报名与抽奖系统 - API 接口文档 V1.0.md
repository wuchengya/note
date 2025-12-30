## **高并发活动报名与抽奖系统 - API 接口文档 V1.0**

### 1. 项目概述

本系统旨在提供一个支持高并发秒杀报名场景的在线活动平台。用户可以浏览活动、在指定时间抢注名额，并在活动结束后参与抽奖。系统后端将利用 Redis 处理瞬时高并发请求，并使用消息队列（MQ）对报名和抽奖等耗时操作进行异步解耦处理。

### 2. 通用约定

- **根路径 (Base URL):** http://your-domain.com/api/v1

- **认证方式:** 除特别说明外（如登录、注册），所有接口都需要在请求头中携带认证信息。

  - Authorization: Bearer <JWT_TOKEN>

- **请求/响应格式:** 所有请求和响应体均为 application/json 格式。

- **日期格式:** 所有日期时间格式均为 YYYY-MM-DD HH:mm:ss。

- **统一响应体结构:** 所有接口返回的数据都将包裹在以下结构中。

  codeJSON

  ```
  {
    "code": 200, // 业务状态码, 200 为成功
    "message": "Success", // 提示信息
    "data": {} // 实际返回的数据, 可能为对象或数组
  }
  ```

### 3. 数据模型 (Data Transfer Objects - DTOs)

- **User (用户)**

  codeJSON

  ```
  {
    "id": 1,
    "username": "zhangsan",
    "studentId": "20210001"
  }
  ```

- **Activity (活动)**

  codeJSON

  ```
  {
    "id": 101,
    "title": "计算机学院毕业晚会",
    "description": "一场精彩绝伦的视听盛宴",
    "totalSlots": 200, // 总名额
    "availableSlots": 50, // 剩余名额 (可能实时变化)
    "registrationStartTime": "2023-12-01 12:00:00", // 报名开始时间
    "registrationEndTime": "2023-12-01 12:05:00", // 报名结束时间
    "status": "IN_PROGRESS" // 活动状态: NOT_STARTED, IN_PROGRESS, FINISHED, DRAWN
  }
  ```

- **Registration (报名记录)**

  codeJSON

  ```
  {
    "id": "REG-20231201-...", // 报名订单号/ID
    "userId": 1,
    "activityId": 101,
    "status": "SUCCESS", // 报名状态: PENDING(排队中), SUCCESS(成功), FAILED(失败)
    "registrationTime": "2023-12-01 12:00:01"
  }
  ```

- **LotteryResult (中奖结果)**

  codeJSON

  ```
  {
    "activityId": 101,
    "userId": 1,
    "prizeName": "一等奖：蓝牙耳机",
    "isWinner": true
  }
  ```

------



### 4. 接口定义

#### 4.1 用户模块 (User Module)

1. **用户注册**

   - **Endpoint:** POST /users/register

   - **描述:** 创建一个新用户账户。

   - **认证:** 无需

   - **请求体:**

     codeJSON

     ```
     {
       "username": "zhangsan",
       "password": "password123",
       "studentId": "20210001"
     }
     ```

   - **成功响应 (201 Created):**

     codeJSON

     ```
     {
       "code": 201,
       "message": "User registered successfully",
       "data": {
         "id": 1,
         "username": "zhangsan",
         "studentId": "20210001"
       }
     }
     ```

2. **用户登录**

   - **Endpoint:** POST /users/login

   - **描述:** 用户登录以获取 JWT Token。

   - **认证:** 无需

   - **请求体:**

     codeJSON

     ```
     {
       "studentId": "20210001",
       "password": "password123"
     }
     ```

   - **成功响应 (200 OK):**

     codeJSON

     ```
     {
       "code": 200,
       "message": "Login successful",
       "data": {
         "token": "ey..."
       }
     }
     ```

3. **获取当前用户信息**

   - **Endpoint:** GET /users/me

   - **描述:** 获取当前登录用户的详细信息。

   - **认证:** 需要

   - **成功响应 (200 OK):**

     codeJSON

     ```
     {
       "code": 200,
       "message": "Success",
       "data": {
         "id": 1,
         "username": "zhangsan",
         "studentId": "20210001"
       }
     }
     ```

#### 4.2 活动模块 (Activity Module)

1. **获取活动列表**

   - **Endpoint:** GET /activities

   - **描述:** 获取所有可用的活动列表，支持分页。

   - **认证:** 无需

   - **查询参数:**

     - page (int, optional, default: 1): 页码
     - size (int, optional, default: 10): 每页数量

   - **成功响应 (200 OK):**

     codeJSON

     ```
     {
       "code": 200,
       "message": "Success",
       "data": [
         // Activity 对象的数组
       ]
     }
     ```

2. **获取指定活动详情**

   - **Endpoint:** GET /activities/{id}

   - **描述:** 根据活动 ID 获取其详细信息。

   - **认证:** 无需

   - **路径参数:**

     - id (int, required): 活动 ID

   - **成功响应 (200 OK):**

     codeJSON

     ```
     {
       "code": 200,
       "message": "Success",
       "data": {
         // 单个 Activity 对象
       }
     }
     ```

#### 4.3 报名（秒杀）模块 (Registration Module)

1. **【核心】提交报名请求（秒杀接口）**

   - **Endpoint:** POST /activities/{activityId}/register

   - **描述:** 这是系统的核心高并发接口。用户调用此接口进行抢票报名。后端会进行快速校验（如是否重复报名、活动是否开始），然后通过 Redis 原子操作预减库存。若预减成功，则将报名信息（userId, activityId）发送到消息队列（MQ），并立即向用户返回“处理中”状态。

   - **认证:** 需要

   - **路径参数:**

     - activityId (int, required): 活动 ID

   - **成功响应 (202 Accepted):** **注意：**这里返回 202 而不是 200，表示服务器已接受请求，但尚未完成处理。

     codeJSON

     ```
     {
       "code": 202,
       "message": "Your registration is being processed. Please check the result later.",
       "data": {
         "registrationId": "REG-20231201-..." // 返回一个唯一的报名ID，用于后续查询结果
       }
     }
     ```

   - **失败响应 (4xx/5xx):**

     - 400 Bad Request: 活动未开始或已结束。
     - 409 Conflict: 您已报名，请勿重复提交。
     - 503 Service Unavailable: 名额已满或系统繁忙，请稍后再试。

2. **查询报名结果**

   - **Endpoint:** GET /registrations/{registrationId}

   - **描述:** 用户通过上一步获取的 registrationId 来轮询查询自己的报名最终结果。

   - **认证:** 需要

   - **路径参数:**

     - registrationId (string, required): 报名 ID

   - **成功响应 (200 OK):**

     codeJSON

     ```
     {
       "code": 200,
       "message": "Success",
       "data": {
         // 单个 Registration 对象
         "id": "REG-20231201-...",
         "status": "SUCCESS" // 可能是 PENDING, SUCCESS 或 FAILED
       }
     }
     ```

3. **获取我的报名记录**

   - **Endpoint:** GET /users/me/registrations

   - **描述:** 获取当前用户所有的报名记录。

   - **认证:** 需要

   - **成功响应 (200 OK):**

     codeJSON

     ```
     {
       "code": 200,
       "message": "Success",
       "data": [
         // Registration 对象的数组
       ]
     }
     ```

#### 4.4 抽奖与结果模块 (Lottery & Result Module)

1. **【管理员】触发抽奖**

   - **Endpoint:** POST /admin/activities/{activityId}/draw

   - **描述:** (此接口仅限管理员调用) 对一个已结束且报名成功的活动进行抽奖。该操作同样是异步的，它会向 MQ 发送一个抽奖任务消息，由后台的抽奖服务消费并执行抽奖逻辑，最终将结果写入数据库。

   - **认证:** 需要（管理员权限）

   - **路径参数:**

     - activityId (int, required): 活动 ID

   - **成功响应 (202 Accepted):**

     codeJSON

     ```
     {
       "code": 202,
       "message": "Lottery process has been started.",
       "data": null
     }
     ```

2. **查询活动中奖结果**

   - **Endpoint:** GET /activities/{activityId}/lottery-results

   - **描述:** 公开查询某个活动的中奖名单。

   - **认证:** 无需

   - **路径参数:**

     - activityId (int, required): 活动 ID

   - **成功响应 (200 OK):**

     codeJSON

     ```
     {
       "code": 200,
       "message": "Success",
       "data": [
         // LotteryResult 对象的数组，只包含中奖者信息
       ]
     }
     ```

3. **查询我的中奖信息**

   - **Endpoint:** GET /activities/{activityId}/my-lottery-result

   - **描述:** 查询当前登录用户在某个特定活动中的中奖情况。

   - **认证:** 需要

   - **路径参数:**

     - activityId (int, required): 活动 ID

   - **成功响应 (200 OK):**

     codeJSON

     ```
     {
       "code": 200,
       "message": "Success",
       "data": {
         "activityId": 101,
         "userId": 1,
         "prizeName": "三等奖：纪念T恤",
         "isWinner": true // true为中奖，false为未中奖
       }
     }
     ```

------



### 5. 实施建议

1. **秒杀接口 (POST /activities/{activityId}/register) 的实现要点:**
   - **Redis 预热:** 活动开始前，将活动库存加载到 Redis 的 string 或 hash 中。
   - **Redis 原子扣减:** 使用 DECR 命令来扣减库存，保证原子性。
   - **Redis 防重:** 使用 SETNX 或 Set 数据结构来判断用户是否已经提交过请求，防止一人多次报名。
   - **发送 MQ 消息:** Redis 操作成功后，立即将 {userId, activityId} 封装成消息发送到 MQ，方法即可返回。
2. **MQ 消费者:**
   - **报名消费者:** 独立的服务或线程，监听报名队列。消费消息，执行数据库 INSERT 操作，创建报名记录。需要处理消息的幂等性（防止重复消费）。
   - **抽奖消费者:** 监听抽奖任务队列。消费消息，从数据库拉取该活动所有报名成功的用户，执行抽奖算法，然后批量更新中奖结果到数据库。

这个项目挑战与实践价值并存，祝你编码愉快！如果在实现过程中遇到具体问题，随时可以再来问我。	







































