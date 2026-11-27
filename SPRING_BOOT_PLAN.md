# SailChat Spring Boot 服务端开发计划书

## 一、项目目标

构建一个安全、稳定、可扩展、支持多设备及离线同步的即时通讯服务端。

服务端负责 SailChat 的互联网通信、账号认证、消息转发、离线消息存储、用户关系、群组管理、文件服务、消息同步及音视频信令。

服务端不得成为客户端 BLE、局域网及纯本地通信的必要依赖。

服务端所有开发行为必须遵守根目录 AGENTS.md。

## 二、技术选型

| 技术 | 用途 |
|---|---|
| Java 21 LTS | 开发语言 |
| Spring Boot | 核心框架 |
| Spring Security | 认证与授权 |
| Spring MVC | REST API |
| Spring WebSocket | 实时通信 |
| MyBatis-Plus | 数据访问 |
| PostgreSQL | 关系型数据库 |
| Redis | 缓存、限流及在线状态 |
| RabbitMQ | 后续异步消息分发 |
| Protobuf | 二进制通信协议 |
| JWT | Access Token |
| Flyway | 数据库版本迁移 |
| MinIO / S3 | 对象存储 |
| SpringDoc OpenAPI | 接口文档 |
| Maven | 项目构建 |
| Nginx | 反向代理及负载均衡 |
| Docker Compose | 初期部署 |
| Coturn | STUN/TURN |
| Prometheus | 服务监控 |
| Grafana | 监控可视化 |
| JUnit 5 | 单元测试 |
| Testcontainers | 集成测试 |

使用受维护且与 Java 21、各项依赖兼容的 Spring Boot 稳定版本。

首期采用模块化单体架构。

禁止在项目初期无必要地引入大规模微服务、Kubernetes、分布式事务及多个功能重复的基础设施。

## 三、项目结构

```text
sail-chat-server/
├── sail-bootstrap/
├── sail-common/
├── sail-protocol/
├── sail-security/
├── sail-auth/
├── sail-user/
├── sail-identity/
├── sail-device/
├── sail-contact/
├── sail-conversation/
├── sail-message/
├── sail-group/
├── sail-websocket/
├── sail-sync/
├── sail-file/
├── sail-notification/
├── sail-call/
├── sail-admin/
├── sail-infrastructure/
├── sail-federation/
├── docker/
├── scripts/
├── docs/
├── pom.xml
├── compose.yaml
├── .env.example
├── README.md
└── AGENTS.md
```

Java 根包名暂定：

com.sailchat

各业务模块内部目录：

```text
module/
├── controller/
├── application/
├── domain/
├── service/
├── repository/
├── entity/
├── mapper/
├── dto/
├── vo/
├── event/
├── enums/
├── config/
└── exception/
```

核心业务模块不得循环依赖。

业务模块不得直接修改其他模块管理的数据表。

跨模块操作通过应用服务、明确的接口及领域事件完成。

## 四、服务端架构

### 接入层

- REST API。
- WebSocket Gateway。
- 请求认证。
- 参数校验。
- 限流。
- 请求追踪。

### 业务层

- 认证。
- 用户。
- 设备。
- 联系人。
- 会话。
- 消息。
- 群组。
- 同步。
- 文件。
- 通知。
- 通话。

### 基础设施层

- PostgreSQL。
- Redis。
- Object Storage。
- Outbox。
- 消息队列。
- 定时任务。
- 外部推送服务。

### 运维层

- 日志。
- 指标。
- 服务健康检查。
- 告警。
- 审计。
- 备份。
- 恢复。
- 部署。

## 五、账号认证模块

模块：sail-auth

### 功能

- 用户注册。
- 用户名登录。
- 邮箱登录。
- 手机号码登录。
- 密码登录。
- 邮箱验证码登录。
- 手机验证码登录。
- 邮箱验证。
- 手机号码验证。
- 密码重置。
- 密码修改。
- Access Token。
- Refresh Token。
- Token 轮换。
- Token 撤销。
- 二维码登录。
- 多设备登录。
- 退出指定设备。
- 退出全部设备。
- 账号冻结。
- 账号注销。
- 登录异常检测。
- 登录历史。

### 认证规范

密码采用 Argon2id 或经过合理成本配置的 BCrypt 存储。

禁止明文存储密码。

Access Token 采用短期有效期。

Refresh Token 采用安全随机令牌，数据库仅保存必要的令牌哈希及状态。

Refresh Token 使用轮换机制，并检测已撤销令牌的重复使用。

Token 必须与用户、设备、会话及服务端签发者关联。

账号或设备被吊销后应能够使对应连接及认证凭证失效。

验证码必须具有：

- 有效期。
- 使用次数限制。
- 发送频率限制。
- 验证次数限制。
- 绑定业务用途。
- 防止重复使用。
- 防止账号枚举。

不得在接口响应中直接返回验证码。

开发环境验证码模拟器只能在明确的非生产环境使用。

## 六、身份与设备模块

模块：

- sail-user
- sail-identity
- sail-device

### 用户资料

- 用户 ID。
- 用户名。
- 昵称。
- 头像。
- 邮箱。
- 手机号码。
- 个性签名。
- 账号状态。
- 隐私设置。
- 创建时间。
- 更新时间。

邮箱及手机号等敏感信息不得默认公开。

### 本地身份绑定

支持客户端无网络创建的 identityId 与在线账号进行绑定。

绑定流程：

1. 客户端发起绑定请求。
2. 服务端检查登录状态及身份绑定关系。
3. 服务端生成一次性随机挑战。
4. 客户端使用身份私钥完成签名。
5. 服务端验证签名、挑战用途及有效期。
6. 验证 identityId 与公钥的一致性。
7. 建立账号和本地身份映射。
8. 返回绑定结果及必要的身份凭证。

不得通过提交 identityId 和 userId 直接完成绑定。

身份已经绑定其他账号时不得静默覆盖。

### 设备管理

每台设备维护：

- deviceId。
- userId。
- identityId。
- deviceName。
- platform。
- appVersion。
- protocolVersion。
- devicePublicKey。
- deviceTrustState。
- lastActiveAt。
- revokedAt。

支持设备注册、公钥更新、设备授权、设备撤销及登录设备管理。

设备公钥目录必须防止服务器或其他用户未经授权替换受信任身份。

生产版本应建立明确的密钥变更告警及设备验证机制。

## 七、WebSocket 网关

模块：sail-websocket

连接地址：

/ws/v1

采用原生 WebSocket 二进制数据帧。

业务协议使用共享 Protobuf，不依赖 STOMP 作为唯一消息格式。

### 连接流程

1. 客户端通过 WSS 建立连接。
2. 校验 Access Token 或一次性 WebSocket 连接票据。
3. 校验用户及设备状态。
4. 协商协议版本。
5. 创建连接上下文。
6. 注册在线连接。
7. 返回连接确认。
8. 恢复必要的同步状态。
9. 开始处理实时消息。

禁止将长期 Token 放入可能被访问日志记录的 URL 查询参数。

### 连接管理

实现：

- UserConnectionRegistry。
- DeviceConnectionRegistry。
- WebSocketSessionManager。
- WebSocketMessageDispatcher。
- HeartbeatManager。
- ConnectionRateLimiter。
- ProtocolValidator。
- ConnectionCleanupService。

单实例使用本地连接注册表管理真实 Socket。

Redis 仅保存具有 TTL 的分布式连接位置及在线状态等元数据，不直接存储 WebSocket 对象。

### 数据包类型

- AUTH
- AUTH_RESULT
- PING
- PONG
- MESSAGE_SEND
- MESSAGE_RECEIVE
- MESSAGE_ACK
- MESSAGE_RECEIPT
- SYNC_REQUEST
- SYNC_RESPONSE
- CONTACT_EVENT
- GROUP_EVENT
- PRESENCE_EVENT
- CALL_SIGNAL
- ERROR
- SERVER_NOTICE

### 网关要求

- 最大消息帧限制。
- 连接速率限制。
- 单设备连接策略。
- 心跳超时。
- 写入超时。
- 出站有界队列。
- 慢客户端处理。
- 连接清理。
- 协议异常处理。
- 请求关联 ID。
- 二进制协议版本管理。
- 优雅关闭。
- 多实例扩展能力。

所有数据库持久化、文件处理及耗时业务不得长时间阻塞 WebSocket I/O 线程。

## 八、消息服务

模块：sail-message

### 支持消息类型

- TEXT
- IMAGE
- VIDEO
- AUDIO
- FILE
- LOCATION
- CONTACT
- STICKER
- SYSTEM
- CALL_RECORD
- REPLY
- FORWARD
- REACTION

### 支持消息事件

- MESSAGE_CREATED
- MESSAGE_EDITED
- MESSAGE_RECALLED
- MESSAGE_DELETED
- MESSAGE_DELIVERED
- MESSAGE_READ
- MESSAGE_EXPIRED
- MESSAGE_REACTION_ADDED
- MESSAGE_REACTION_REMOVED

### 消息处理流程

1. 获取认证用户及设备身份。
2. 验证消息发送方。
3. 校验协议版本和消息大小。
4. 验证会话权限及成员状态。
5. 校验目标设备和加密信封。
6. 检查会话隐私类型。
7. 根据逻辑消息 ID 进行幂等检查。
8. 在数据库事务中保存消息及设备投递记录。
9. 保存对应 Outbox 事件。
10. 提交事务。
11. 返回 SERVER_ACCEPTED。
12. 异步查找目标设备连接。
13. 执行在线消息推送。
14. 保存离线设备待同步记录。
15. 接收并处理设备回执。
16. 更新用户可见投递状态。

服务端必须验证消息来源，禁止信任客户端自行声明的 senderId。

同一个 messageId 如果重复提交相同数据，应返回已有处理结果。

同一个 messageId 如果提交冲突的不可变消息内容，应返回冲突错误。

不得重复创建消息、会话事件或通知任务。

### 端到端加密原则

服务端主要保存：

- 消息 ID。
- 会话 ID。
- 发送设备。
- 路由信息。
- 接收设备。
- 必要的事件类型。
- 加密消息内容。
- 服务器接收时间。
- 同步序号。
- 投递状态。

服务端不保存客户端身份私钥及端到端加密会话私钥。

服务端不得提供任意解密用户聊天记录的管理接口。

服务端仍可能接触消息路由、成员关系、时间及大小等元数据，必须明确保护范围。

## 九、会话服务

模块：sail-conversation

功能：

- 创建单聊会话。
- 创建群聊会话。
- 查询会话列表。
- 会话成员维护。
- 会话未读状态。
- 最后消息信息。
- 会话置顶。
- 会话静音。
- 会话归档。
- 会话删除状态。
- 会话成员权限。
- 会话增量同步。

纯本地会话不得默认创建服务端记录。

云同步会话建立时需要确定稳定的会话 ID、成员关系及权限。

离线会话转换为云同步会话必须经过身份验证及权限确认。

会话删除应区分：

- 当前设备隐藏。
- 当前用户删除历史。
- 服务端保留期限。
- 对所有用户发起的合法撤回事件。

不得混淆以上操作。

## 十、好友服务

模块：sail-contact

功能：

- 用户搜索。
- 好友申请。
- 申请接受。
- 申请拒绝。
- 申请取消。
- 好友删除。
- 好友备注。
- 好友分组。
- 星标好友。
- 黑名单。
- 好友在线状态订阅。
- 好友关系增量同步。
- 离线联系人绑定。

好友状态至少包括：

- PENDING
- ACCEPTED
- REJECTED
- CANCELLED
- BLOCKED
- REMOVED

好友关系变更必须在事务中处理。

必须防止重复好友关系、重复请求及未授权关系变更。

在线状态仅向具有查看权限的用户提供。

黑名单及隐私规则必须在好友请求、消息发送、通话及用户查询等入口统一执行。

## 十一、群组服务

模块：sail-group

功能：

- 群组创建。
- 群组解散。
- 群组资料修改。
- 成员邀请。
- 成员加入。
- 成员退出。
- 成员移除。
- 管理员设置。
- 群主转移。
- 群公告。
- 群昵称。
- 群组禁言。
- 群组权限。
- 群消息分发。
- 群组事件同步。
- 加密群组状态更新。

群组角色：

- OWNER
- ADMIN
- MEMBER

群组加入方式：

- INVITATION_ONLY
- APPROVAL_REQUIRED
- OPEN

群组成员变更必须维护 membershipVersion。

群组权限变更需要生成独立的 GroupEvent。

客户端提交过期成员版本时，服务端必须拒绝不合法操作或要求同步最新状态。

成员被移除后，服务端必须停止向该成员投递后续群组消息。

客户端负责正确更新群组加密状态。

群组加密密钥分发完成之前不得将新消息错误发送给已撤销成员。

## 十二、同步服务

模块：sail-sync

### 同步目标

实现本地消息、远程消息、联系人、群组、会话及消息事件的可靠增量同步。

### 同步数据类型

- USER_PROFILE
- DEVICE
- CONTACT
- CONVERSATION
- MESSAGE
- MESSAGE_EVENT
- RECEIPT
- GROUP
- GROUP_MEMBER
- GROUP_EVENT
- ATTACHMENT

### 同步机制

服务端为每个用户的同步事件流维护有明确提交顺序的单调递增序号。

不得直接假设普通数据库自增 ID 可以代表事务提交顺序。

客户端使用持久化 cursor 获取增量数据。

每次同步返回：

- 当前事件集合。
- nextCursor。
- highWatermark。
- hasMore。
- serverTime。
- syncVersion。

客户端只有在本地数据库事务成功提交后才能确认同步游标。

同步游标应绑定用户、设备及对应事件流，防止越权读取其他用户数据。

### 离线消息补传

客户端通过 BLE 或 LAN 发送的消息，如果属于 CLOUD_SYNC 会话，可以在恢复网络后提交加密信封及相关事件。

服务端必须验证：

- 发送者身份。
- 设备身份。
- 会话权限。
- 消息归属。
- 会话同步策略。
- 消息唯一性。
- 群组成员版本。
- 加密信封合法性。

纯本地消息不得被正常同步流程接收。

补传已送达消息时应保持原 messageId，并区分历史同步与实时通知，避免重复推送通知。

### 冲突处理

- 消息：不可变消息 ID 去重。
- 消息修改：独立事件及版本规则。
- 用户资料：服务端版本控制。
- 联系人：关系事件及权限校验。
- 群组：成员版本及管理员权限。
- 删除：使用保留期限明确的删除标记。
- 已读状态：按照会话中可确认的消息位置进行更新。

不得简单使用最后上传时间覆盖所有数据。

## 十三、文件服务

模块：sail-file

采用 MinIO 或其他兼容 S3 的对象存储。

### 功能

- 初始化上传任务。
- 单文件上传。
- 分片上传。
- 断点续传。
- 上传完成。
- 上传取消。
- 文件下载。
- 临时下载凭证。
- 文件元数据。
- 文件大小限制。
- 文件存储配额。
- 文件过期清理。
- 无主文件回收。
- 文件访问授权。
- 上传任务状态查询。

### 文件上传流程

1. 客户端生成附件 ID。
2. 客户端根据会话安全协议加密文件。
3. 计算上传密文的完整性哈希。
4. 请求创建上传任务。
5. 服务端验证用户权限和配额。
6. 返回上传凭证及分片策略。
7. 客户端上传密文。
8. 客户端提交上传完成请求。
9. 服务端校验上传结果。
10. 创建文件元数据记录。
11. 客户端发送包含附件引用的加密消息。
12. 接收方通过授权下载并在客户端解密。

文件密钥不得以明文形式保存在服务端。

文件对象默认不公开。

不得使用永久公开下载地址。

需要支持访问控制及短期签名链接。

端到端加密文件无法由服务器直接检查明文内容，应通过大小、配额、内容类型声明校验及客户端安全措施控制风险。

## 十四、通知服务

模块：sail-notification

支持：

- APNs。
- FCM。
- 后续国内 Android 厂商推送。
- 远程消息通知。
- 好友申请通知。
- 群组邀请通知。
- 来电通知。
- 安全事件通知。
- 新设备登录通知。

管理：

- Push Token。
- 用户设备绑定。
- Token 更新及失效清理。
- 通知频率。
- 免打扰状态。
- 通知去重。
- 离线设备通知策略。
- 推送失败重试。

端到端加密会话默认仅发送通用消息提醒及必要的事件引用，不在服务器推送载荷中包含聊天明文。

## 十五、音视频信令服务

模块：sail-call

采用 WebRTC 体系。

功能：

- 通话发起。
- 通话邀请。
- 接听。
- 拒绝。
- 取消。
- 挂断。
- 超时。
- 通话状态管理。
- 多设备来电竞争处理。
- SDP Offer。
- SDP Answer。
- ICE Candidate。
- TURN 临时凭证。
- 通话记录。
- 多人通话房间。

通话状态：

- CREATED
- RINGING
- ACCEPTED
- CONNECTING
- CONNECTED
- REJECTED
- CANCELLED
- ENDED
- TIMEOUT
- FAILED

Spring Boot 只负责业务及信令。

STUN/TURN 由 Coturn 等独立服务提供。

多人媒体流通过后续独立 SFU 服务实现。

禁止使用普通 Spring Boot Controller 或 WebSocket 网关直接转发大量实时音视频流。

## 十六、管理员模块

模块：sail-admin

功能：

- 管理员登录。
- 角色管理。
- 权限管理。
- 用户管理。
- 账号冻结。
- 用户设备管理。
- 群组管理。
- 举报管理。
- 文件配额管理。
- 服务端配置。
- 系统公告。
- 在线用户统计。
- 消息吞吐量统计。
- WebSocket 连接统计。
- 消息积压统计。
- 服务状态。
- 操作审计。
- 系统日志。

管理员系统使用独立授权体系及更严格的登录安全策略。

禁止管理员默认拥有读取用户私密聊天明文的能力。

举报系统可以接收用户主动提交的必要证据，并限制证据的访问范围及保留期限。

## 十七、REST API 设计

统一前缀：

/api/v1

统一响应结构：

```json
{
  "code": "OK",
  "message": "success",
  "data": {},
  "requestId": "request-id",
  "timestamp": 0
}
```

使用正确的 HTTP 状态码。

不得所有业务错误都返回 HTTP 200。

分页接口使用 cursor 或明确的 page、pageSize 参数，并限制最大分页大小。

### 17.1 认证接口

| 方法 | 路径 |
|---|---|
| POST | /auth/register |
| POST | /auth/login |
| POST | /auth/logout |
| POST | /auth/logout-all |
| POST | /auth/refresh |
| POST | /auth/verification-codes |
| POST | /auth/verify-code |
| POST | /auth/password/forgot |
| POST | /auth/password/reset |
| PUT | /auth/password |
| POST | /auth/ws-ticket |
| POST | /auth/qr/sessions |
| POST | /auth/qr/sessions/{id}/approve |
| GET | /auth/qr/sessions/{id} |

### 17.2 用户及身份接口

| 方法 | 路径 |
|---|---|
| GET | /users/me |
| PATCH | /users/me |
| GET | /users/search |
| GET | /users/{id} |
| GET | /users/me/privacy |
| PUT | /users/me/privacy |
| POST | /identities/bind/challenge |
| POST | /identities/bind/confirm |
| GET | /identities/me |
| DELETE | /users/me |

### 17.3 设备及密钥接口

| 方法 | 路径 |
|---|---|
| GET | /devices |
| POST | /devices/register |
| POST | /devices/{id}/authorize |
| DELETE | /devices/{id} |
| PATCH | /devices/{id} |
| GET | /users/{id}/devices |
| POST | /keys/prekeys |
| GET | /keys/bundles/{userId} |

密钥接口具体结构必须根据最终选择的成熟加密协议定义。

### 17.4 联系人接口

| 方法 | 路径 |
|---|---|
| GET | /contacts |
| GET | /contacts/requests |
| POST | /contacts/requests |
| POST | /contacts/requests/{id}/accept |
| POST | /contacts/requests/{id}/reject |
| POST | /contacts/requests/{id}/cancel |
| PATCH | /contacts/{id} |
| DELETE | /contacts/{id} |
| GET | /contacts/blocked |
| POST | /contacts/blocked |
| DELETE | /contacts/blocked/{id} |

### 17.5 会话及消息接口

| 方法 | 路径 |
|---|---|
| GET | /conversations |
| POST | /conversations |
| GET | /conversations/{id} |
| PATCH | /conversations/{id} |
| DELETE | /conversations/{id} |
| GET | /conversations/{id}/messages |
| POST | /messages |
| POST | /messages/{id}/events |
| POST | /messages/receipts |
| GET | /messages/{id}/status |

WebSocket 和 REST 消息提交必须复用同一个 MessageApplicationService。

禁止两个入口分别实现不一致的消息验证、去重及持久化逻辑。

### 17.6 群组接口

| 方法 | 路径 |
|---|---|
| POST | /groups |
| GET | /groups |
| GET | /groups/{id} |
| PATCH | /groups/{id} |
| DELETE | /groups/{id} |
| GET | /groups/{id}/members |
| POST | /groups/{id}/invitations |
| POST | /groups/{id}/join |
| POST | /groups/{id}/leave |
| DELETE | /groups/{id}/members/{userId} |
| PUT | /groups/{id}/members/{userId}/role |
| PUT | /groups/{id}/owner |
| GET | /groups/{id}/events |

### 17.7 同步接口

| 方法 | 路径 |
|---|---|
| GET | /sync/bootstrap |
| GET | /sync/pull |
| POST | /sync/push |
| POST | /sync/ack |
| GET | /sync/status |

### 17.8 文件接口

| 方法 | 路径 |
|---|---|
| POST | /files/uploads |
| GET | /files/uploads/{id} |
| POST | /files/uploads/{id}/parts |
| POST | /files/uploads/{id}/complete |
| DELETE | /files/uploads/{id} |
| GET | /files/{id} |
| GET | /files/{id}/download |
| DELETE | /files/{id} |

### 17.9 通知及通话接口

| 方法 | 路径 |
|---|---|
| POST | /notifications/devices |
| DELETE | /notifications/devices/{id} |
| GET | /notifications/settings |
| PUT | /notifications/settings |
| GET | /calls/history |
| POST | /calls |
| POST | /calls/{id}/accept |
| POST | /calls/{id}/reject |
| POST | /calls/{id}/end |
| GET | /calls/turn-credentials |

所有接口必须维护 OpenAPI 文档、认证要求、请求参数、响应字段、错误码及权限说明。

## 十八、数据库设计

采用 PostgreSQL。

数据库表使用 snake_case 命名。

统一时间类型及 UTC 存储规范。

所有数据库迁移使用 Flyway。

### 18.1 用户及安全表

| 表名 | 主要字段 |
|---|---|
| users | id、username、email、phone、password_hash、status |
| user_profiles | user_id、nickname、avatar、signature、version |
| user_identities | identity_id、user_id、public_key、status |
| user_devices | device_id、user_id、identity_id、platform、trust_state |
| device_public_keys | key_id、device_id、public_key、version |
| device_prekeys | key_id、device_id、public_key、status |
| auth_sessions | session_id、user_id、device_id、status |
| refresh_tokens | token_id、session_id、token_hash、expires_at |
| verification_codes | id、target、code_hash、purpose、expires_at |
| identity_challenges | id、user_id、challenge、expires_at |
| login_logs | id、user_id、device_id、result、created_at |

### 18.2 联系人及会话表

| 表名 | 主要字段 |
|---|---|
| friend_relations | user_id、friend_id、status、version |
| friend_requests | id、sender_id、receiver_id、status |
| contact_groups | id、user_id、name |
| user_blacklists | user_id、blocked_user_id |
| conversations | id、type、scope、version |
| conversation_members | conversation_id、user_id、role、status |
| conversation_preferences | conversation_id、user_id、muted、pinned |

### 18.3 消息表

| 表名 | 主要字段 |
|---|---|
| messages | message_id、conversation_id、sender_id、sender_device_id、created_at |
| message_envelopes | envelope_id、message_id、target_device_id、ciphertext |
| message_events | event_id、message_id、event_type、version、payload |
| message_receipts | message_id、device_id、receipt_type、received_at |
| message_deliveries | delivery_id、message_id、target_device_id、status |
| conversation_sequences | conversation_id、next_sequence |

### 18.4 群组表

| 表名 | 主要字段 |
|---|---|
| groups | id、conversation_id、owner_id、name、membership_version |
| group_members | group_id、user_id、role、status |
| group_events | event_id、group_id、version、payload |
| group_invitations | id、group_id、inviter_id、invitee_id、status |

### 18.5 同步及异步任务表

| 表名 | 主要字段 |
|---|---|
| sync_events | user_id、sequence、event_id、type、payload |
| sync_streams | user_id、last_committed_sequence |
| sync_cursors | user_id、device_id、stream_id、last_sequence |
| event_outbox | id、event_type、payload、status、retry_count |
| processed_events | consumer_name、event_id、processed_at |

### 18.6 文件及通知表

| 表名 | 主要字段 |
|---|---|
| files | file_id、owner_id、object_key、size、ciphertext_hash |
| file_permissions | file_id、conversation_id、permission_type |
| upload_tasks | upload_id、user_id、status、expires_at |
| push_tokens | device_id、provider、token、status |
| notification_tasks | id、user_id、event_id、status |
| call_records | call_id、caller_id、status、started_at、ended_at |
| user_reports | report_id、reporter_id、target_id、status |
| audit_logs | id、operator_id、action、target_id、created_at |

### 18.7 索引及约束

必须建立：

- 用户名唯一索引。
- 标准化邮箱唯一索引。
- 手机号码唯一索引。
- 身份 ID 唯一索引。
- 设备 ID 唯一索引。
- 消息逻辑 ID 唯一索引。
- 消息与目标设备投递唯一索引。
- 会话消息分页索引。
- 群成员组合唯一索引。
- 用户同步序号唯一索引。
- 事件幂等唯一索引。
- Outbox 待处理状态索引。
- 上传任务过期时间索引。

数据库层必须通过唯一约束防止并发重复写入，而不是只依赖 Java 代码提前查询。

不得为了方便开发而省略核心数据完整性约束。

## 十九、Redis 使用规范

Redis 用于：

- 在线状态。
- WebSocket 节点路由。
- 验证码辅助状态。
- 请求限流。
- 短期连接票据。
- 幂等辅助缓存。
- 热点数据缓存。
- 分布式协调。

Key 命名建议：

```text
sailchat:presence:{userId}
sailchat:device:online:{deviceId}
sailchat:ws:route:{deviceId}
sailchat:auth:ticket:{ticketId}
sailchat:rate:{scope}:{subject}
sailchat:cache:user:{userId}
```

所有临时 Key 必须设置合理的 TTL。

Redis 数据不得作为聊天消息、好友关系及群组权限的唯一永久存储。

Redis 故障后应有明确的降级策略。

对于安全认证及限流相关能力，故障时不得默认放行危险请求。

## 二十、可靠消息分发

首期采用数据库事务 Outbox。

流程：

1. 在同一事务中写入业务数据和 Outbox。
2. 后台任务读取待处理事件。
3. 执行事件分发。
4. 成功后更新处理状态。
5. 失败后根据策略重试。
6. 达到限制后记录失败事件及告警。

多实例阶段可使用 RabbitMQ。

消息消费必须幂等。

RabbitMQ 可用于：

- 消息投递。
- 跨节点分发。
- 推送通知。
- 文件异步处理。
- 业务事件。
- 延迟任务。

RabbitMQ 不替代 PostgreSQL 消息持久化。

不得宣称消息队列天然实现严格 Exactly Once。

## 二十一、错误码规范

统一业务错误码：

- AUTH_REQUIRED
- AUTH_INVALID
- TOKEN_EXPIRED
- DEVICE_REVOKED
- ACCOUNT_DISABLED
- INVALID_CREDENTIALS
- VERIFICATION_CODE_INVALID
- RATE_LIMITED
- PERMISSION_DENIED
- RESOURCE_NOT_FOUND
- VALIDATION_FAILED
- IDENTITY_CONFLICT
- IDENTITY_VERIFICATION_FAILED
- PROTOCOL_UNSUPPORTED
- MESSAGE_TOO_LARGE
- MESSAGE_CONFLICT
- CONVERSATION_NOT_FOUND
- CONVERSATION_FORBIDDEN
- LOCAL_ONLY_NOT_ALLOWED
- GROUP_VERSION_CONFLICT
- FILE_TOO_LARGE
- STORAGE_QUOTA_EXCEEDED
- SYNC_CURSOR_EXPIRED
- SERVICE_UNAVAILABLE
- INTERNAL_ERROR

每个错误码应定义：

- HTTP 状态码。
- 是否可重试。
- 用户可见提示。
- 开发者诊断信息。
- 是否需要重新认证。
- 是否需要重新同步。

服务端不得直接向客户端暴露 Java 堆栈、SQL 语句及内部服务器路径。

## 二十二、安全规范

必须实现：

- HTTPS。
- WSS。
- Spring Security。
- 密码安全哈希。
- 安全 Token 管理。
- 接口级权限验证。
- 资源所有权验证。
- 对象级访问控制。
- 群组成员权限。
- WebSocket 消息级认证授权。
- 防 SQL 注入。
- 请求体大小限制。
- 文件路径校验。
- 验证码防刷。
- 登录限流。
- 用户搜索限流。
- 消息发送限流。
- 防止消息重放。
- 消息幂等。
- 防止账号枚举。
- 防止越权访问。
- 管理员审计。
- 敏感日志脱敏。
- 安全配置管理。
- 依赖漏洞检查。

不能因为用户已经登录，就允许其通过修改 URL 中的 userId、groupId、messageId 或 fileId 获取他人数据。

所有资源访问都必须进行归属及权限校验。

禁止写入生产环境默认密码。

生产环境密钥及数据库凭证通过安全环境变量或密钥管理服务注入。

## 二十三、环境配置

支持：

- local
- dev
- test
- staging
- prod

配置包括：

- Server Port。
- PostgreSQL。
- Redis。
- JWT 签发配置。
- 对象存储。
- 邮件服务。
- 短信服务。
- 推送服务。
- TURN 服务。
- 日志级别。
- CORS。
- 文件大小限制。
- 消息大小限制。
- 认证策略。
- 数据保留期限。

所有真实凭证禁止提交 Git。

提供 .env.example 及清晰的配置说明。

## 二十四、部署方案

首期 Docker Compose 包含：

- sail-chat-server。
- PostgreSQL。
- Redis。
- MinIO 或兼容对象存储。
- Nginx。

音视频阶段增加 Coturn。

高并发阶段增加 RabbitMQ、Prometheus、Grafana 及多实例 WebSocket 节点。

提供：

- Dockerfile。
- compose.yaml。
- 环境变量模板。
- 数据库迁移脚本。
- Nginx 配置。
- 健康检查。
- 服务启动脚本。
- 备份脚本。
- 恢复说明。
- HTTPS 证书配置说明。

生产环境不得将 PostgreSQL、Redis、对象存储管理端口直接无保护暴露到公网。

必须验证数据库和对象存储备份可以实际恢复。

## 二十五、监控及日志

记录必要的结构化日志。

统一 requestId 或 traceId。

监控指标：

- HTTP 请求量。
- HTTP 错误率。
- HTTP 延迟。
- WebSocket 在线连接。
- WebSocket 重连次数。
- 消息发送速率。
- 消息投递延迟。
- 消息投递失败率。
- ACK 延迟。
- Outbox 积压数量。
- 数据库连接池状态。
- Redis 错误率。
- 文件上传成功率。
- 文件存储使用量。
- JVM 堆内存。
- GC 状态。
- 线程池状态。
- CPU 使用率。

日志不得包含聊天明文、密码、完整 Token、私钥等敏感数据。

监控系统必须区分真实生产指标与开发测试数据。

## 二十六、服务端开发阶段

### S0：工程基础

- 初始化 Maven 多模块工程。
- 配置 Spring Boot。
- 配置 PostgreSQL。
- 配置 Flyway。
- 配置 Redis。
- 建立共享协议。
- 建立统一异常处理。
- 建立 OpenAPI 文档。
- 建立 Docker Compose。
- 建立测试环境。

### S1：身份认证

- 注册。
- 登录。
- Token。
- 设备注册。
- 本地身份绑定。
- 用户资料。
- 基础权限及安全策略。

### S2：实时通信

- WebSocket 网关。
- 二进制协议。
- 连接注册表。
- 心跳。
- 单人会话。
- 消息持久化。
- 消息确认。
- 消息去重。
- 事务 Outbox。

### S3：同步及离线消息

- 增量同步。
- 离线消息。
- 设备投递状态。
- 多设备同步。
- 本地直连消息补传。
- 同步游标。
- 权限冲突处理。

### S4：社交功能

- 好友。
- 黑名单。
- 群组。
- 群组权限。
- 群组事件。
- 用户在线状态。
- 联系人同步。

### S5：文件及通知

- 对象存储。
- 文件上传。
- 文件分片。
- 断点续传。
- 下载权限。
- 推送通知。
- 附件清理。

### S6：音视频及后台

- WebRTC 信令。
- Coturn。
- 通话记录。
- 管理后台接口。
- 用户举报。
- 审计。
- 服务监控。

### S7：性能及高级扩展

- 多实例 WebSocket。
- RabbitMQ。
- 分布式连接路由。
- 数据库性能优化。
- 灾难恢复。
- 私有化部署。
- 联邦通信预留。

## 二十七、测试要求

必须编写：

- 单元测试。
- REST API 集成测试。
- PostgreSQL 集成测试。
- Redis 集成测试。
- WebSocket 集成测试。
- Protobuf 跨语言兼容测试。
- 身份签名测试。
- 权限测试。
- Token 轮换测试。
- 消息去重测试。
- 同步游标测试。
- 群组权限测试。
- 文件访问控制测试。
- Outbox 故障恢复测试。
- 服务器重启测试。
- 并发写入测试。
- WebSocket 压力测试。
- 数据库迁移测试。
- 备份恢复测试。

必须覆盖以下异常场景：

- Token 失效。
- 设备被撤销。
- 重复注册。
- 重复消息。
- 消息 ID 冲突。
- 网络中断。
- ACK 丢失。
- 客户端重复提交。
- 服务端处理后立即崩溃。
- 数据库事务失败。
- Redis 不可用。
- 对象存储不可用。
- 用户访问他人消息。
- 用户访问他人文件。
- 非群成员发送群消息。
- 已移除群成员尝试继续发送。
- 过期同步游标。
- 协议版本不支持。
- 非法及超大数据包。
- 消息分发任务重复执行。

## 二十八、服务端性能目标

初期参考测试目标：

- 普通 REST API P95 延迟低于 200ms。
- 同区域正常网络下的在线消息 P95 端到端延迟低于 500ms。
- 单节点 WebSocket 以 10000 连接进行压力测试。
- 消息吞吐量根据指定部署资源进行阶梯式压测。
- 数据库重复消息记录为 0。
- 支持普通群组 500 人规模测试。
- 文件流式上传及下载。
- 服务重启后持久化任务可以恢复。

所有性能测试必须记录：

- 服务器 CPU。
- 内存。
- 网络。
- 数据库配置。
- 并发连接数。
- 消息大小。
- 消息发送速率。
- 测试持续时间。
- 延迟分布。
- 错误率。

未经实测不得宣称已达到性能目标。

## 二十九、后续扩展

预留：

- 自定义服务器地址。
- 用户自行部署服务器。
- 企业私有化部署。
- 多租户。
- 跨服务器身份。
- 联邦通信协议。
- Bot 机器人。
- Webhook。
- 开放 API。
- 第三方集成。
- SailChat SDK。
- Web 客户端。

不同服务器上的用户身份需要明确的命名空间及认证体系。

联邦通信必须单独设计身份认证、信任关系、权限、数据保留及滥用控制。

首期不要求实现完整联邦网络。

## 三十、最终交付要求

必须交付：

- 完整 Spring Boot 源代码。
- Maven 工程。
- PostgreSQL 数据库迁移。
- Protobuf 协议及生成流程。
- REST API 文档。
- WebSocket 协议文档。
- Docker Compose 部署配置。
- 自动化测试。
- 环境配置模板。
- 服务启动说明。
- 安全设计说明。
- 运维及备份说明。
- 实际测试报告。

服务端必须能够实际启动并与 Flutter 客户端完成真实消息通信，不得仅提供接口定义或空实现。