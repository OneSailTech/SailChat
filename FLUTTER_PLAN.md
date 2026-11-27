# SailChat AGENTS.md

## 1. 项目基本信息

- 项目名称：SailChat
- 项目类型：混合式、分布式、离线优先的即时通讯软件
- 客户端框架：Flutter
- 客户端语言：Dart
- 状态管理：Provider + ChangeNotifier
- 客户端架构：模块化 MVVM + Repository + Service
- 路由管理：go_router
- 服务端框架：Spring Boot
- 服务端语言：Java 21 LTS
- 服务端架构：模块化单体架构
- 通信协议：Protocol Buffers
- 本地数据库：Drift + SQLite
- 服务端数据库：PostgreSQL
- 缓存服务：Redis
- 对象存储：MinIO / S3
- 项目语言：代码使用英文命名，开发文档及必要注释使用中文。

本文件为 SailChat 项目的全局开发规范，所有 AI 编程代理及开发人员必须遵守。

## 2. 项目愿景

SailChat 是一个融合中心化与去中心化通信能力的跨平台即时通讯系统。

传统即时通讯软件依赖互联网及中心服务器，SailChat 必须打破这一限制，使用户在没有互联网、服务器不可用、没有 Wi-Fi 路由器等情况下，仍然能够利用设备自身的通信能力，与附近用户建立连接并交换消息。

在具备互联网连接时，SailChat 应具备完整的现代即时通讯软件功能，包括账号注册、登录、好友、群组、消息同步、文件传输、音视频通话及多设备管理。

系统必须实现：

- 统一用户身份。
- 统一设备身份。
- 统一联系人。
- 统一会话。
- 统一消息模型。
- 统一通信协议。
- 统一消息状态。
- 多通信链路自动调度。
- 本地与云端数据同步。
- 安全可靠的消息传输。

SailChat 不应仅仅是一个普通的 WebSocket 聊天软件，而应具备独立于中心服务器运行的通信能力。

## 3. 核心开发原则

1. **Offline First**：离线优先，基础功能不得强制依赖互联网。
2. **Local First**：本地数据优先，聊天 UI 以本地数据库作为可信数据来源。
3. **Peer-to-Peer**：支持设备之间直接通信。
4. **Multi-Transport**：支持多种通信链路。
5. **Server Optional**：服务器不是附近通信的必要条件。
6. **End-to-End Encryption**：生产版本使用成熟且经过审查的端到端加密协议。
7. **Privacy First**：隐私优先，不强制上传本地聊天数据。
8. **Cross Platform**：跨平台通信能力统一，平台差异通过适配层解决。
9. **Modular Architecture**：模块独立、低耦合、高内聚、可扩展。
10. **Reliable Delivery**：通过持久化、确认、重试及幂等机制保障消息可靠性。
11. **Capability Driven**：根据设备真实能力决定功能是否可用。
12. **Backward Compatibility**：通信协议及数据库结构支持版本升级。
13. **Single Source of Truth**：避免多个模块分别维护不一致的业务状态。
14. **Security by Design**：安全能力必须从架构设计阶段开始建设。
15. **Testability**：核心业务及通信模块必须具备独立测试能力。

禁止为了快速完成互联网聊天而删除、弱化或替换 BLE 和局域网通信能力。

## 4. 项目组成

```text
SailChat/
├── AGENTS.md
├── README.md
├── FLUTTER_PLAN.md
├── SPRING_BOOT_PLAN.md
├── DEVELOPMENT_PROGRESS.md
├── TEST_REPORT.md
├── CHANGELOG.md
├── sail-chat-app/
├── sail-chat-server/
├── sail-chat-protocol/
├── sail-chat-admin/
└── docs/
```

### 项目职责

- `sail-chat-app/`：Flutter 跨平台客户端。
- `sail-chat-server/`：Spring Boot 即时通讯服务端。
- `sail-chat-protocol/`：Flutter 与 Java 共享的 Protobuf 通信协议。
- `sail-chat-admin/`：后续开发的管理后台。
- `docs/`：架构、接口、协议、安全、部署及开发文档。

协议定义必须拥有唯一来源。

禁止 Flutter 与 Java 分别维护互不兼容的消息模型。

客户端及服务端的详细开发要求分别以 FLUTTER_PLAN.md 和 SPRING_BOOT_PLAN.md 为准。

当详细计划与本文件存在冲突时，应遵循本文件中的全局架构及安全约束，并记录冲突原因。

## 5. 系统总体架构

系统按照以下层级设计。

### 5.1 表现层（Presentation Layer）

负责：

- 页面渲染。
- 用户交互。
- 状态展示。
- 页面导航。
- 用户操作反馈。

不得直接实现通信协议、数据库操作及复杂业务逻辑。

### 5.2 应用层（Application Layer）

负责：

- 用户账号。
- 聊天业务。
- 联系人业务。
- 会话业务。
- 群组业务。
- 文件传输。
- 消息同步。
- 音视频通话。
- 业务流程调度。

### 5.3 领域层（Domain Layer）

负责定义：

- 用户模型。
- 设备模型。
- 身份模型。
- 消息模型。
- 会话模型。
- 群组模型。
- 通信链路模型。
- 业务规则。
- Repository 抽象接口。
- UseCase。

领域层不得直接依赖具体 UI 框架、数据库驱动或通信插件。

### 5.4 数据层（Data Layer）

负责：

- 本地数据库。
- HTTP API。
- 数据仓库。
- 数据缓存。
- 数据映射。
- 增量同步。
- 数据持久化。

### 5.5 通信层（Transport Layer）

负责：

- 设备发现。
- 连接建立。
- 数据传输。
- 通信协议解析。
- 路由选择。
- 链路切换。
- 消息确认。
- 消息重试。
- 分包与组包。
- 连接生命周期管理。

### 5.6 安全层（Security Layer）

负责：

- 身份认证。
- 设备验证。
- 密钥管理。
- 加密与解密。
- 消息完整性认证。
- 安全会话。
- 设备信任。
- 权限控制。

### 5.7 平台适配层（Platform Layer）

负责封装以下平台的原生能力：

- Android
- iOS
- Windows
- macOS
- Linux
- HarmonyOS

不同平台的能力差异必须通过统一接口及能力检测机制处理。

## 6. Flutter 客户端架构规范

### 6.1 技术架构

Flutter 客户端统一使用：

- Provider。
- ChangeNotifier。
- 模块化 MVVM。
- Repository Pattern。
- Service Layer。
- Constructor Dependency Injection。
- go_router。

禁止引入 GetX、Riverpod、Bloc 等其他状态管理框架替代 Provider。

通信模块、数据库模块、加密模块及核心业务模块必须独立于 UI 状态管理框架。

Provider 负责状态管理及依赖提供，不负责直接实现底层通信协议。

### 6.2 核心目录结构

```text
lib/
├── app/
│   ├── bootstrap/
│   ├── providers/
│   ├── modules/
│   ├── routes/
│   ├── theme/
│   ├── widgets/
│   └── app.dart
├── core/
│   ├── transport/
│   ├── discovery/
│   ├── connection/
│   ├── routing/
│   ├── protocol/
│   ├── identity/
│   ├── encryption/
│   ├── handshake/
│   ├── message/
│   ├── queue/
│   ├── sync/
│   ├── file/
│   └── lifecycle/
├── domain/
│   ├── entities/
│   ├── repositories/
│   └── usecases/
├── db/
│   ├── tables/
│   ├── dao/
│   └── migrations/
├── net/
│   ├── api/
│   ├── interceptor/
│   ├── request/
│   └── response/
├── data/
│   ├── local/
│   └── remote/
├── repository/
├── services/
├── event/
├── models/
├── common/
├── config/
├── utils/
├── generated/
└── main.dart
```

目录职责：

- `app/`：应用 UI、页面模块、路由、主题及依赖注入。
- `core/`：核心通信能力及基础业务引擎。
- `domain/`：领域模型、抽象接口及业务用例。
- `db/`：本地数据库及迁移。
- `net/`：HTTP 网络请求。
- `data/`：本地及远程数据来源。
- `repository/`：统一数据仓库实现。
- `services/`：应用级业务服务。
- `event/`：必要的跨模块事件。
- `models/`：公共数据模型。
- `common/`：与具体业务及 UI 无关的公共能力。
- `config/`：环境配置及系统配置。
- `utils/`：通用工具类。
- `generated/`：自动生成代码。

禁止手动修改自动生成的代码。

### 6.3 页面模块规范

所有业务页面统一放置在 `lib/app/modules/` 目录下。

每个功能模块根据实际需求划分为：

```text
chat/
├── views/
│   └── chat_page.dart
├── view_models/
│   └── chat_view_model.dart
├── providers/
│   └── chat_providers.dart
├── models/
│   └── chat_ui_state.dart
└── widgets/
    ├── message_bubble.dart
    ├── message_list.dart
    └── message_input_bar.dart
```

目录职责：

- `views/`：界面渲染及用户交互。
- `view_models/`：页面状态及业务操作。
- `providers/`：模块依赖注入及生命周期管理。
- `models/`：模块内部 UI 状态及数据模型。
- `widgets/`：模块内部可复用组件。

简单模块允许省略不必要的目录。

禁止为了形式统一创建大量没有实际用途的空文件。

跨模块复用组件统一放置于 `lib/app/widgets/`。

### 6.4 View 规范

View 负责：

- 界面渲染。
- 状态展示。
- 用户交互。
- 页面导航。
- Dialog、SnackBar 等界面反馈。
- 调用 ViewModel 执行业务操作。

禁止在 View 中：

- 直接操作数据库。
- 直接发送 HTTP 请求。
- 直接操作 BLE。
- 直接操作 TCP、UDP、Socket。
- 直接操作 WebSocket。
- 实现复杂业务逻辑。
- 管理底层消息发送队列。
- 创建长期运行的全局通信服务。

### 6.5 ViewModel 规范

ViewModel 默认继承 ChangeNotifier。

负责：

- 管理页面状态。
- 调用 UseCase。
- 调用 Repository。
- 调用 Service。
- 处理业务执行结果。
- 管理加载状态。
- 管理页面级异常。
- 通知 UI 更新。

ViewModel 必须通过构造函数注入依赖。

禁止在 ViewModel 中长期持有 BuildContext。

禁止直接调用与 Widget 强耦合的导航、弹窗及提示逻辑。

复杂页面优先使用不可变 UI State，避免大量分散且难以维护的可变状态。

ViewModel 销毁时必须释放其拥有的：

- StreamSubscription。
- Timer。
- 监听器。
- 临时资源。

异步操作结束时必须避免更新已经销毁的 ViewModel。

### 6.6 Provider 使用规范

统一使用：

- `Provider`：普通依赖及服务提供。
- `MultiProvider`：集中注册多个依赖。
- `ProxyProvider`：依赖其他 Provider 的对象。
- `ChangeNotifierProvider`：页面及响应式业务状态。
- `Consumer`：局部组件刷新。
- `Selector`：精确状态监听。

使用规则：

- `context.read<T>()`：读取依赖及执行业务操作，不订阅状态变化。
- `context.watch<T>()`：监听对象状态变化。
- `context.select<T, R>()`：精确监听指定字段。

禁止在页面根节点无差别监听所有全局状态。

禁止频繁调用 notifyListeners 导致大范围 UI 重建。

禁止在 build 方法中直接触发网络请求、数据库写入及通信扫描。

禁止在 build 方法中重复创建长期运行的 Service。

新建对象使用 Provider 的 create 方式管理生命周期。

已有实例使用 `.value` 提供时，必须明确实例所有权及销毁责任。

ProxyProvider 更新依赖时，不得无意义地重复创建已经运行的数据库、连接服务及 ViewModel。

高频通信数据应使用独立 Stream、细粒度状态及合理节流策略。

### 6.7 依赖作用域

Provider 依赖分为三个层级。

#### 应用级作用域

整个应用运行期间保持稳定：

- AppConfig
- AppDatabaseManager
- AppLifecycleService
- PlatformCapabilityService
- NetworkMonitor
- ThemeProvider
- LocaleProvider

#### 用户会话级作用域

与当前本地身份及账号会话关联：

- AccountRepository
- IdentityService
- TrustService
- CryptoService
- TransportManager
- DiscoveryManager
- ConnectionManager
- RoutingService
- MessageService
- OutboxService
- InboxService
- SyncService
- FileTransferService
- MessageRepository
- ContactRepository
- ConversationRepository

切换账号时必须正确释放旧会话资源，并确保账号数据及通信连接隔离。

退出在线账号但继续使用原本地身份时，不得错误关闭本地通信能力。

#### 页面级作用域

根据页面生命周期创建及销毁：

- LoginViewModel
- RegisterViewModel
- ConversationViewModel
- ChatViewModel
- ContactViewModel
- NearbyViewModel
- GroupViewModel
- SettingsViewModel
- ProfileViewModel
- FileTransferViewModel
- CallViewModel

禁止使用同一个全局 ChatViewModel 管理所有聊天页面。

关闭聊天页面不得关闭整个应用的 BLE、LAN、WebSocket 连接及消息队列。

### 6.8 Repository 规范

Repository 负责统一数据访问。

主要职责：

- 屏蔽本地及远程数据来源差异。
- 返回领域模型。
- 管理数据缓存。
- 管理增量更新。
- 提供响应式数据流。
- 协调本地数据库与远程接口。

本地数据库作为聊天 UI 的唯一可信数据来源。

禁止 UI 分别维护互相冲突的本地消息列表及远程消息列表。

### 6.9 Service 规范

Service 负责长期运行的业务及基础服务。

包括：

- 通信连接。
- 设备发现。
- 消息收发。
- 消息队列。
- 消息同步。
- 身份认证。
- 密钥管理。
- 文件传输。
- 音视频通话。

所有核心 Service 必须通过构造函数注入依赖。

禁止底层 Service 直接依赖：

- BuildContext。
- Widget。
- 页面 ViewModel。
- Provider 查找机制。
- 页面生命周期。

核心通信能力必须能够脱离 Flutter UI 独立运行及测试。

### 6.10 路由管理

统一使用 go_router。

支持：

- 命名路由。
- 参数传递。
- 嵌套路由。
- ShellRoute。
- 页面重定向。
- 路由守卫。
- 深度链接。
- 页面级 Provider 注入。
- 移动端及桌面端布局。

路由守卫必须区分：

- 尚未创建本地身份。
- 已创建本地身份但未登录在线账号。
- 已登录在线账号。
- 在线凭证过期但本地身份仍有效。
- 当前处于纯离线模式。

禁止因为用户未登录在线账号而阻止其进入附近通信及本地聊天页面。

### 6.11 数据流规范

统一消息接收数据流：

```text
Transport
    ↓
InboxService
    ↓
MessageService
    ↓
Repository
    ↓
Drift / SQLite
    ↓
Stream
    ↓
ViewModel
    ↓
Provider
    ↓
UI
```

统一用户操作数据流：

```text
View
    ↓
ViewModel
    ↓
UseCase / Service
    ↓
Repository / TransportManager
    ↓
Database
    ↓
ViewModel 状态更新
    ↓
UI
```

禁止底层通信服务直接修改 Widget 状态。

禁止使用 EventBus 替代数据库及核心业务状态管理。

## 7. Spring Boot 服务端架构规范

### 7.1 技术架构

服务端统一使用：

- Java 21 LTS。
- Spring Boot。
- Spring Security。
- Spring MVC。
- Spring WebSocket。
- MyBatis-Plus。
- PostgreSQL。
- Redis。
- Flyway。
- Maven。
- Protobuf。
- MinIO / S3。
- Docker Compose。

后续根据实际业务需求引入 RabbitMQ、独立 WebSocket 网关及其他分布式基础设施。

首期采用模块化单体架构。

禁止在业务规模尚未明确时盲目引入复杂微服务体系。

### 7.2 模块划分

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
└── pom.xml
```

Java 根包名暂定为 `com.sailchat`。

### 7.3 模块内部结构

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

模块根据实际需求选择对应目录。

### 7.4 服务端开发要求

- Controller 负责接口接入、参数校验及响应处理。
- Application Service 负责业务用例及跨模块流程编排。
- Domain 负责核心领域模型及业务规则。
- Service 负责具体业务能力。
- Repository 负责数据访问抽象。
- Mapper 负责数据库操作。
- DTO 负责数据传输。
- VO 负责接口响应模型。
- Entity 负责持久化实体。

禁止 Controller 直接操作数据库。

禁止将数据库实体直接作为所有外部接口响应。

禁止核心业务模块循环依赖。

禁止业务模块未经授权直接修改其他模块管理的数据。

跨模块操作通过明确接口及领域事件完成。

WebSocket 与 REST 消息提交必须复用同一套消息业务服务。

服务端不得成为客户端 BLE、LAN 及其他纯本地通信的必要依赖。

## 8. 支持的通信方式

| 通信方式 | 用途 | 优先级 |
|---|---|---|
| BLE GATT | 无网络情况下的附近消息通信 | P0 |
| TCP Socket | 局域网可靠消息及文件传输 | P0 |
| UDP | 局域网辅助设备发现 | P0 |
| mDNS / DNS-SD | 局域网服务发现 | P0 |
| WebSocket / WSS | 互联网实时消息 | P0 |
| HTTPS | 账号、同步、文件及消息补传 | P0 |
| 手机热点 | 无路由器的局域网环境 | P1 |
| Wi-Fi Direct | 支持该能力的平台进行无线直连 | P1 |
| WebRTC | 音视频及点对点文件传输 | P1 |
| Application Mesh | 应用层多跳中继通信 | P2 |
| Federation | 跨服务器互联 | P2 |

优先级说明：

- P0：核心版本必须实现。
- P1：完整功能版本实现。
- P2：高级扩展版本实现。

所有通信方式必须接入统一通信架构。

禁止不同通信方式分别维护独立且互不兼容的聊天业务。

## 9. 统一通信架构

### 9.1 Transport 抽象

所有通信方式必须通过统一 Transport 接口接入系统。

```dart
abstract interface class Transport {
  TransportType get type;

  TransportCapabilities get capabilities;

  Stream<TransportState> get states;

  Stream<InboundPacket> get incomingPackets;

  Future<void> initialize();

  Future<void> connect(PeerEndpoint endpoint);

  Future<void> disconnect(String connectionId);

  Future<void> send(
    String connectionId,
    Uint8List packet,
  );

  Future<void> dispose();
}
```

该接口作为初始设计，实际实现允许根据不同传输模型进行合理扩展，但不得破坏统一通信架构。

设备发现通过独立 DiscoveryProvider 接口实现。

DiscoveryProvider 属于通信领域接口，与 Flutter Provider 状态管理无关。

禁止强制所有 Transport 实现不适用的设备发现能力。

### 9.2 核心通信服务

| 服务 | 职责 |
|---|---|
| TransportManager | 管理所有通信方式 |
| DiscoveryManager | 管理设备发现 |
| PeerRegistry | 管理附近设备记录 |
| CapabilityService | 检测设备通信能力 |
| ConnectionManager | 管理连接及生命周期 |
| HandshakeService | 身份认证及协议协商 |
| RoutingService | 选择通信链路 |
| IdentityService | 管理用户及设备身份 |
| TrustService | 管理设备信任关系 |
| CryptoService | 加密及密钥管理 |
| ProtocolService | 协议编码及解析 |
| MessageService | 统一消息业务入口 |
| OutboxService | 持久化发送队列 |
| InboxService | 接收及消息去重 |
| ReceiptService | 消息确认及已读回执 |
| SyncService | 本地及云端同步 |
| FileTransferService | 文件传输 |
| CallService | 音视频通话 |
| MeshService | 多跳消息转发 |

所有服务必须职责明确。

禁止多个 Service 重复实现相同通信逻辑。

### 9.3 BLE 通信规范

BLE 模块必须支持：

- Central。
- Peripheral。
- GATT Client。
- GATT Server。
- Advertising。
- Scanning。
- Characteristic Write。
- Characteristic Notify / Indicate。
- 数据分包。
- 数据组包。
- 消息确认。
- 流量控制。
- 超时重传。
- 连接恢复。

必要时创建独立 Flutter 原生插件 `sail_ble`。

Android 使用 Kotlin，iOS 使用 Swift。

Flutter 与原生层优先使用 Pigeon 进行类型安全通信。

禁止假设设备拥有固定 MTU。

必须根据平台实际支持的有效数据长度进行动态分包。

接收端必须限制最大消息长度、分包数量、同时重组任务数量及内存占用。

BLE 主要用于文字、控制消息、身份交换、回执及小型数据。

禁止将 BLE 作为大型文件或常规实时音视频的默认传输方式。

### 9.4 LAN 通信规范

使用 mDNS / DNS-SD 及 UDP 辅助发现。

使用 TCP Socket 进行可靠数据传输。

服务发现名称：

```text
_sailchat._tcp.local.
```

设备发现数据只允许包含必要信息：

- 临时发现 ID。
- 协议版本。
- 监听端口。
- 通信能力。

不得通过广播公开敏感身份信息。

TCP 必须使用明确的帧边界协议。

必须处理：

- 粘包。
- 拆包。
- 无效长度。
- 超大数据帧。
- 心跳超时。
- 网络中断。
- 自动重连。
- 数据流背压。

支持无互联网的 Wi-Fi 及手机热点通信。

局域网连接必须执行身份认证及安全握手。

禁止因为处于同一局域网而默认信任其他设备。

### 9.5 WebSocket 通信规范

WebSocket 使用 WSS 安全连接。

业务数据采用共享 Protobuf 二进制协议。

必须支持：

- Access Token 或短期连接票据认证。
- 协议版本协商。
- 心跳检测。
- 指数退避重连。
- 重连随机抖动。
- Token 失效处理。
- 消息确认。
- 消息去重。
- 离线消息恢复。
- 多设备同步。
- 连接异常清理。

服务端连接不可用时，不得影响本地通信服务运行。

禁止将长期 Token 放在可能被日志记录的 URL 查询参数中。

### 9.6 HTTP 通信规范

客户端统一使用 Dio。

服务端统一提供 REST API。

接口前缀：

```text
/api/v1
```

HTTP 主要负责：

- 账号认证。
- 用户资料。
- 好友关系。
- 群组管理。
- 设备管理。
- 历史消息同步。
- 离线消息补传。
- 文件上传下载。
- WebSocket 连接票据申请。

Token 刷新必须进行并发控制。

禁止多个请求同时触发大量重复 Token 刷新操作。

### 9.7 Mesh 通信规范

Mesh 作为高级扩展能力。

采用应用层 Store-and-Forward 多跳通信模式。

必须实现：

- 邻居节点发现。
- 中继能力声明。
- TTL。
- 最大跳数。
- 消息去重。
- 中继缓存。
- 路由发现。
- 转发配额。
- 转发频率限制。
- 防止循环转发。
- 消息过期清理。
- 用户授权。

Mesh 中继默认关闭。

中继节点不得读取转发消息明文。

禁止将应用层 Mesh 直接等同于标准 Bluetooth Mesh。

## 10. 身份系统

身份系统不得只依赖服务器账号。

统一定义：

| 标识 | 职责 |
|---|---|
| identityId | 用户密码学身份的稳定唯一标识 |
| userId | 服务器账号标识 |
| deviceId | 设备实例唯一标识 |
| conversationId | 会话唯一标识 |
| messageId | 逻辑消息全局唯一标识 |
| eventId | 业务事件唯一标识 |
| deliveryId | 具体设备投递任务标识 |
| transferId | 文件传输任务标识 |
| connectionId | 通信连接标识 |

identityId 应基于身份公钥及明确的编码规则生成。

userId、deviceId 等普通标识采用 UUID 等可靠的唯一标识方案。

禁止使用以下数据作为用户唯一身份：

- MAC 地址。
- IMEI。
- IP 地址。
- 设备名称。
- 手机型号。
- 用户昵称。

### 10.1 本地身份

用户首次在无网络环境下打开应用时，必须能够：

1. 生成安全身份密钥。
2. 创建本地身份。
3. 创建设备身份。
4. 设置用户资料。
5. 保存本地身份及安全凭证。
6. 初始化本地通信能力。
7. 进入应用。

本地身份不得强制要求登录服务器。

### 10.2 在线账号绑定

本地身份绑定云端账号时，必须通过服务端随机挑战及签名验证证明身份所有权。

禁止仅提交 identityId 和 userId 就完成账号绑定。

绑定账号后不得重新生成已有 identityId。

已有联系人及会话必须保持有效。

身份已绑定其他账号时不得静默覆盖。

### 10.3 多设备身份

每台设备具有独立设备标识及设备密钥。

新设备加入已有账号时，必须通过受信任设备授权或安全恢复流程建立信任关系。

无法获得原身份密钥时，不得假装恢复了原有密码学身份。

设备撤销后应停止后续合法投递，并更新相关加密状态。

离线设备无法及时获得撤销信息时，必须根据明确的信任有效期及安全策略处理。

## 11. 运行模式

### AUTO

自动选择合适的通信链路。

根据目标设备可达性、连接质量、消息大小、通信能力及用户隐私策略进行调度。

### OFFLINE_ONLY

纯离线模式。

禁止所有 SailChat 互联网业务请求、远程消息发送、云同步、遥测及远程资源加载。

### NEARBY_ONLY

仅允许 BLE、LAN 及平台支持的本地点对点通信。

### INTERNET_PREFERRED

优先使用 WebSocket 及 HTTPS，可根据用户设置使用本地通信。

### BLE_ONLY

仅使用 BLE。

### LAN_ONLY

仅使用局域网通信。

### INTERNET_ONLY

仅使用互联网通信。

用户指定的通信方式不可用时，必须提供明确提示。

禁止未经用户授权绕过通信模式及隐私设置。

## 12. 会话分类

### LOCAL_ONLY

纯本地会话。

- 仅通过允许的本地通信链路交换数据。
- 不自动创建服务端会话。
- 不自动上传聊天记录。
- 不进入云端同步队列。

### CLOUD_SYNC

云同步会话。

- 支持本地通信。
- 支持互联网通信。
- 支持加密云端同步。
- 支持多设备同步。
- 支持离线消息补传。

### TEMPORARY

临时会话。

- 支持附近临时聊天。
- 支持过期时间。
- 支持自动清理策略。

临时会话必须同时具有明确的本地或云端同步策略，不得仅因为会话属于临时类型而默认允许上传服务器。

临时会话及阅后即焚不得承诺能够阻止对方截图、复制或保留数据。

纯本地会话转换为云同步会话时，必须经过用户明确授权，并验证相关参与方权限。

历史聊天记录不得因为会话类型变化而默认全部上传。

## 13. 统一通信协议

使用 Protobuf 定义共享二进制协议。

协议目录：

```text
sail-chat-protocol/
├── proto/
│   ├── common.proto
│   ├── identity.proto
│   ├── discovery.proto
│   ├── handshake.proto
│   ├── envelope.proto
│   ├── message.proto
│   ├── receipt.proto
│   ├── conversation.proto
│   ├── group.proto
│   ├── contact.proto
│   ├── sync.proto
│   ├── file.proto
│   ├── call.proto
│   ├── transport.proto
│   └── mesh.proto
├── scripts/
├── test_vectors/
└── README.md
```

### 13.1 统一消息信封

统一消息信封至少包含：

- protocolVersion
- messageId
- conversationId
- senderIdentityId
- senderDeviceId
- recipientIdentityId 或 groupId
- createdAt
- clientSequence
- conversationScope
- encryptionVersion
- encryptedPayload
- keyReference
- expirationTime

多设备加密投递必须使用独立 RecipientEnvelope，包含：

- 目标设备标识。
- 加密会话信息。
- 对应设备的消息密文。

同一个逻辑消息可以拥有不同目标设备的加密信封，但 messageId 必须保持一致。

### 13.2 协议分层

业务消息协议与传输层协议必须分离。

以下字段属于传输层：

- BLE 分包编号。
- BLE 分包数量。
- 连接序号。
- 重试次数。
- 链路类型。
- Mesh 跳数。
- 传输任务标识。

切换通信链路不得改变原始业务消息身份及已认证的消息内容。

### 13.3 协议兼容性

- 所有协议必须具有版本信息。
- 不得重复使用已经删除的字段编号。
- 删除字段时必须使用 reserved 保留对应编号。
- 协议变更必须同步生成 Dart 与 Java 代码。
- 必须建立跨语言协议测试向量。
- 必须处理未知字段及未知消息类型。
- 必须设置数据包大小限制。
- 不兼容协议必须返回明确错误。

禁止将 Protobuf 默认序列化结果直接作为未经规范化的跨语言数字签名格式。

必须采用成熟协议提供的认证数据格式，或经过明确规范及审查的签名编码方式。

## 14. 可靠消息机制

消息采用至少一次传输与幂等处理机制。

不承诺底层网络提供严格的 Exactly Once 投递。

通过消息 ID、事务及去重机制保证同一逻辑消息不会重复展示。

### 14.1 发送流程

1. 生成 messageId。
2. 检查会话权限。
3. 检查通信模式。
4. 加密消息并生成目标设备信封。
5. 在本地事务中保存消息及待发送任务。
6. 根据可用链路进行发送。
7. 等待对应级别的 ACK。
8. 更新消息状态。
9. 超时后进行退避重试。
10. 必要时在允许范围内切换其他链路。

### 14.2 接收流程

1. 接收及解析数据。
2. 校验协议版本及数据大小。
3. 验证发送方身份。
4. 验证会话权限。
5. 验证消息完整性。
6. 检查消息是否已处理。
7. 执行必要的解密操作。
8. 在事务中保存消息及加密会话状态。
9. 返回设备级确认。
10. 通过本地数据库通知业务及 UI 更新。

重复到达的已认证消息不得重复入库，并应在适当情况下重新返回 ACK。

### 14.3 消息确认

必须区分：

- 链路层确认。
- 服务端持久化确认。
- 接收设备持久化确认。
- 用户已读确认。

禁止将 Socket Write 成功、BLE Write 成功或 WebSocket Send 成功直接视为接收方已经收到消息。

消息接收成功必须以接收设备持久化完成为准。

### 14.4 消息重试

重试任务必须持久化。

应用重启后必须恢复未完成任务。

同一逻辑消息重试时不得重新生成 messageId。

相同目标设备的重试应复用已持久化的加密信封，避免重复推进加密会话状态。

临时网络异常允许重试。

身份认证失败、权限不足、非法数据等错误不得无限重试。

所有重试必须具有明确的次数限制、退避策略及过期时间。

## 15. 消息状态

消息状态分为三个独立维度。

### 15.1 传输状态

- CREATED
- QUEUED
- SENDING
- SERVER_ACCEPTED
- DELIVERED
- READ
- FAILED
- CANCELLED

### 15.2 同步状态

- LOCAL_ONLY
- PENDING_SYNC
- SYNCING
- SYNCED
- SYNC_FAILED

### 15.3 内容状态

- NORMAL
- EDITED
- RECALLED
- EXPIRED
- DELETED

禁止将传输状态、同步状态及内容状态混合为同一个枚举。

服务端接收消息不等于接收方已经收到消息。

群组及多设备消息必须维护设备级投递记录。

消息撤回、编辑、删除及表情回应应使用独立业务事件实现。

消息状态更新必须遵循明确的状态转换规则。

较晚到达的低级别 ACK 不得覆盖已确认的高级别状态。

## 16. 智能通信路由

路由系统必须根据目标设备真实可达性进行决策。

禁止仅根据手机是否开启蓝牙或 Wi-Fi 判断消息是否可以送达。

路由决策因素：

- 目标设备可达性。
- 当前通信模式。
- 会话隐私策略。
- 连接认证状态。
- 链路通信能力。
- 消息类型。
- 数据大小。
- 延迟。
- 吞吐量。
- 电量消耗。
- 连接稳定性。
- 历史传输成功率。

默认策略：

- 小型文字消息优先选择稳定且已认证的本地直连链路。
- 无可用本地直连时，允许使用 WebSocket 或 HTTPS。
- 大型文件优先选择 LAN、Wi-Fi P2P 或 WebRTC DataChannel。
- 实时音视频使用支持媒体传输的 IP 网络。
- BLE 主要用于文字及小型数据。
- 无可用链路时，普通消息进入持久化等待队列。
- 实时通话无法建立连接时，明确提示用户。

路由切换必须保持：

- 用户身份一致。
- 联系人一致。
- 会话一致。
- messageId 一致。
- 消息状态一致。
- 消息安全策略一致。

禁止为每种通信方式分别创建不同的聊天会话。

## 17. 数据库管理规范

### 17.1 客户端数据库

统一使用 Drift + SQLite。

必须实现：

- 数据库版本管理。
- 数据库迁移。
- 事务管理。
- 消息去重唯一索引。
- 会话消息组合索引。
- 发送任务索引。
- 同步事件索引。
- 多账号数据隔离。
- 响应式数据查询。

普通 SQLite 不得保存明文私钥及长期登录凭证。

Drift 本身不提供完整的数据库加密保障，必须针对敏感数据单独选择并验证加密存储方案。

消息持久化及加密会话状态更新必须保证崩溃一致性。

### 17.2 服务端数据库

统一使用 PostgreSQL。

所有数据库迁移通过 Flyway 管理。

数据库命名统一使用 snake_case。

时间统一采用明确的 UTC 存储规范。

必须使用合理的：

- 主键。
- 唯一索引。
- 外键或等效业务完整性约束。
- 组合索引。
- 非空约束。
- 数据校验约束。

数据库层必须防止并发重复写入，禁止仅通过应用层提前查询实现唯一性。

### 17.3 Redis

Redis 主要用于：

- 在线状态。
- WebSocket 连接位置。
- 请求限流。
- 短期连接票据。
- 验证码辅助状态。
- 热点数据缓存。
- 分布式协调。

所有临时 Key 必须配置合理 TTL。

Redis 不得作为消息、好友关系及群组权限的唯一永久存储。

Redis 故障时必须具有明确降级策略。

安全相关功能不得因缓存服务不可用而默认放行危险请求。

### 17.4 数据迁移

禁止为了修复数据库结构问题直接删除用户数据库。

所有数据库结构调整必须提供明确迁移方案。

涉及数据删除、覆盖或不可逆变更时，必须提前说明影响并取得用户授权。

## 18. 数据同步规范

同步机制必须支持：

- 增量同步。
- 游标分页。
- 消息去重。
- 多设备同步。
- 会话同步。
- 联系人同步。
- 群组同步。
- 消息回执同步。
- 离线消息补传。
- 同步冲突处理。
- 同步失败恢复。
- 游标过期后的重新同步。

客户端只有在本地数据库事务成功后，才能确认新的同步游标。

服务端必须维护具有明确提交顺序的同步事件流。

禁止直接假设数据库自增 ID 等同于事务提交顺序。

纯本地会话不得进入云端同步队列。

消息事件使用 eventId 去重。

禁止简单使用客户端时间戳覆盖较新数据。

历史消息补传必须与实时消息通知区分，避免重复通知。

所有同步请求必须验证用户、设备、会话及数据访问权限。

## 19. 安全与隐私规范

### 19.1 加密规范

生产环境必须采用成熟、经过审查的加密实现。

单聊加密可以评估 Signal Protocol 相关实现。

群组加密可以评估 MLS 等成熟方案。

实际选型前必须检查：

- Flutter 兼容性。
- Android 兼容性。
- iOS 兼容性。
- 桌面端兼容性。
- 维护状态。
- 安全审查情况。
- 许可证兼容性。
- 多设备支持。
- 离线会话建立能力。

禁止自行实现未经审查的密码协议。

禁止使用：

- 固定密钥。
- 固定随机数。
- Base64 冒充加密。
- 简单 XOR 冒充加密。
- 不安全的自定义加密算法。

未经验证的模拟加密只能用于明确标记的开发测试环境。

不得将模拟加密描述为生产级端到端加密。

### 19.2 身份及设备安全

必须支持：

- 身份认证。
- 设备认证。
- 公钥指纹验证。
- 安全二维码配对。
- 设备信任。
- 密钥变更提醒。
- 密钥轮换。
- 设备吊销。
- 防止身份冒用。
- 防止中间人攻击。
- 防止消息重放。

禁止将未认证设备直接标记为可信设备。

同一用户通过不同通信方式被发现时，必须在身份认证后进行关联。

禁止仅凭昵称、头像、IP 地址或蓝牙名称合并身份。

### 19.3 密钥及凭证管理

客户端使用平台安全存储保存必要密钥及凭证。

禁止：

- 明文保存身份私钥。
- 上传用户身份私钥。
- 将私钥提交至 Git。
- 在日志中记录私钥。
- 在日志中记录完整 Token。
- 在日志中记录用户密码。
- 在日志中记录聊天明文。

安全备份方案必须具有明确的加密及恢复策略。

### 19.4 服务端认证

使用 Spring Security 实现认证及授权。

密码采用 Argon2id 或合理成本配置的 BCrypt。

Access Token 使用短期有效期。

Refresh Token 使用安全随机令牌，并采用轮换及撤销机制。

必须限制：

- 登录失败次数。
- 验证码发送频率。
- 验证码尝试次数。
- 用户搜索频率。
- 消息发送频率。
- WebSocket 连接频率。

禁止明文存储密码。

### 19.5 权限校验

所有服务端资源必须进行对象级权限检查。

包括：

- 用户资料。
- 好友关系。
- 群组。
- 会话。
- 消息。
- 文件。
- 设备。
- 同步记录。

禁止仅因为用户已登录就允许其访问其他用户的数据。

禁止直接信任客户端自行提交的 senderId、userId、groupId、deviceId 等身份及权限信息。

必须通过认证上下文及业务权限进行验证。

### 19.6 本地通信安全

BLE、LAN、P2P 通信同样必须进行安全握手及身份认证。

禁止因为通信不经过服务器而跳过安全校验。

附近设备发现应使用必要且最小化的信息。

优先采用轮换的临时发现标识。

禁止公开广播：

- 手机号码。
- 邮箱。
- 长期设备唯一标识。
- 私密账号信息。
- 用户密钥。

### 19.7 隐私保护

纯离线模式不得自动上传数据。

纯本地会话不得自动同步服务器。

管理员不得绕过端到端加密读取用户聊天明文。

用户主动举报时，可以提交必要证据，但必须限制证据访问权限及保留期限。

服务器仍可能接触消息时间、大小、路由及成员关系等必要元数据，产品文档不得将端到端加密错误描述为完全隐藏所有元数据。

## 20. 文件传输规范

必须支持：

- 文件分片。
- 流式读取。
- 流式写入。
- 文件完整性校验。
- 传输进度。
- 暂停。
- 恢复。
- 取消。
- 断点续传。
- 文件大小限制。
- 文件路径安全校验。

禁止将大型文件一次性加载到内存。

必须防止路径穿越及非授权文件覆盖。

大型文件优先使用 LAN、Wi-Fi P2P 或 WebRTC。

云端文件使用 MinIO 或兼容 S3 的对象存储。

文件对象默认不公开。

下载接口必须校验访问权限。

端到端加密附件必须在客户端加密后上传。

文件密钥不得以明文形式保存在服务端。

## 21. 音视频通信规范

采用 WebRTC 实现音视频能力。

Spring Boot 负责：

- 通话邀请。
- 通话状态。
- SDP 信令。
- ICE Candidate 信令。
- TURN 临时凭证。
- 通话记录。

局域网通话通过本地通信链路交换信令。

互联网通话通过服务器交换信令。

STUN / TURN 由 Coturn 等独立服务提供。

多人会议后续通过独立 SFU 媒体服务实现。

禁止使用普通 Spring Boot Controller 或 WebSocket 网关直接转发大量实时音视频媒体流。

BLE 仅用于辅助发现、信令交换及连接建立。

## 22. UI 与公共组件规范

### 22.1 设计风格

整体 UI 应具有：

- 现代化。
- 简约。
- 科技感。
- 轻量化。
- 统一视觉语言。
- 清晰的信息层级。
- 一致的交互体验。

必须支持：

- 浅色模式。
- 深色模式。
- 跟随系统。
- 多语言。
- 字体缩放。
- 响应式布局。
- 移动端及桌面端适配。

### 22.2 公共组件管理

跨模块复用组件统一放置于：

```text
lib/app/widgets/
```

建议建立：

- AppScaffold
- AppButton
- AppIcon
- AppTextField
- AppDialog
- AppAvatar
- AppLoading
- AppEmpty
- AppError
- MessageBubble
- MessageInputBar
- ConversationTile
- ContactTile
- TransportIndicator
- ConnectionStatusBar
- FileTransferCard

优先复用已有组件。

禁止重复实现功能及样式相同的组件。

### 22.3 颜色管理

颜色必须通过 ColorUtils 或统一 Theme Token 管理。

禁止在业务 UI 中随意硬编码颜色值。

新增颜色前必须优先搜索现有颜色定义。

主题切换统一通过 ThemeProvider 及 Flutter Theme 实现。

禁止在各业务页面中分别维护不一致的深色及浅色主题逻辑。

### 22.4 尺寸管理

尺寸、间距、圆角及字体统一通过 AppSize 或统一设计 Token 管理。

禁止在大量页面中重复定义相同尺寸。

必须考虑不同屏幕大小、系统字体缩放及无障碍设置。

### 22.5 国际化

所有用户可见文案必须纳入国际化资源。

首期支持：

- 简体中文。
- 英文。

预留：

- 繁体中文。
- 日文。
- 韩文。
- 其他语言。

禁止直接在业务 Widget 中大量硬编码用户可见文案。

语言状态统一通过 LocaleProvider 管理。

## 23. 平台限制与生命周期

所有原生通信能力必须先检测设备是否支持，再开放对应功能。

禁止假设所有 Android 设备均支持 BLE Peripheral。

禁止假设 Android Wi-Fi Direct 能够与 iOS 实现同等直接互通。

禁止保证应用被操作系统终止后仍然能够持续：

- BLE 扫描。
- BLE 广播。
- Socket 通信。
- WebSocket 通信。
- Mesh 中继。
- 消息接收。

后台通信必须遵循各操作系统的权限、电源管理及后台执行规范。

权限应在使用相关功能时按需申请。

用户拒绝权限时必须提供明确提示及可用替代方案。

应用恢复前台后，应重新检查：

- 网络状态。
- 蓝牙状态。
- 连接状态。
- 设备发现状态。
- 待发送消息。
- 增量同步状态。

不得通过无限后台定时器规避操作系统限制。

## 24. 性能与资源管理

### Flutter 客户端

必须：

- 聊天消息分页及窗口化加载。
- 局部组件刷新。
- 合理使用 Selector 和 Consumer。
- 避免高频 notifyListeners。
- 避免大型页面重复重建。
- 合理使用图片缓存。
- 文件流式处理。
- 数据库索引优化。
- BLE 流量控制。
- TCP 背压控制。
- 有界数据缓冲区。
- 必要时使用 Isolate 处理 CPU 密集型任务。
- 及时释放 Timer、StreamSubscription、Socket 及文件流。

禁止全局状态对象无限积累历史消息。

### Spring Boot 服务端

必须：

- 合理配置线程池。
- 合理配置数据库连接池。
- 控制 WebSocket 出站队列。
- 处理慢客户端。
- 限制请求体大小。
- 限制消息帧大小。
- 避免阻塞 WebSocket I/O 线程。
- 优化数据库索引及查询。
- 实现异步消息分发。
- 支持服务优雅关闭。
- 支持服务重启后的任务恢复。

所有性能优化必须以实际测试及分析结果为依据。

## 25. 日志与异常处理

建立统一日志及异常处理机制。

日志级别：

- DEBUG
- INFO
- WARN
- ERROR

必须记录必要的：

- 请求标识。
- 消息标识。
- 连接状态。
- 通信错误。
- 重试次数。
- 同步状态。
- 协议版本。
- 服务异常。
- 性能指标。

禁止记录：

- 用户密码。
- 完整 Token。
- 私钥。
- 聊天明文。
- 未脱敏的敏感身份信息。
- 未经授权的文件内容。

禁止直接吞掉异常。

禁止使用空 catch 处理核心业务错误。

用户界面必须展示可理解的错误信息。

服务端不得将内部 SQL、Java 堆栈及服务器路径直接返回客户端。

纯离线模式禁止自动上传诊断日志。

## 26. 代码命名规范

### Dart

- 文件名：snake_case。
- 类名：PascalCase。
- 方法及变量：lowerCamelCase。
- 私有成员：下划线前缀。
- ViewModel：以 ViewModel 结尾。
- Repository：以 Repository 结尾。
- Service：以 Service 结尾。
- 页面：以 Page 或 View 结尾。

示例：

```text
chat_page.dart
chat_view_model.dart
message_repository.dart
transport_manager.dart
identity_service.dart
```

### Java

- 包名：全小写。
- 类名：PascalCase。
- 方法及变量：lowerCamelCase。
- 常量：UPPER_SNAKE_CASE。
- Controller：以 Controller 结尾。
- Service：以 Service 结尾。
- Repository：以 Repository 结尾。
- DTO：以 DTO 或明确的 Request / Response 结尾。

### 通用要求

- 使用具有明确业务含义的命名。
- 禁止使用无意义缩写。
- 禁止使用难以理解的临时变量名称管理复杂业务。
- 禁止重复定义同一业务常量。
- 公共 API 必须具有明确的参数及返回类型。
- 复杂通信逻辑必须包含必要注释。
- 注释应解释设计原因及关键约束，而不是简单重复代码。

## 27. 依赖管理规范

新增任何依赖前必须检查：

- 依赖是否真实存在。
- 当前维护状态。
- 版本兼容性。
- 平台支持情况。
- 许可证。
- 安全漏洞。
- 依赖引入的实际价值。
- 是否与已有依赖重复。

禁止引入未经验证存在的第三方库及 API。

禁止为了简单功能引入过重的依赖。

禁止随意升级与当前任务无关的依赖。

重要依赖升级必须进行兼容性测试。

Flutter 应维护 pubspec.lock。

Spring Boot 应通过 Maven 统一管理依赖版本。

## 28. 测试规范

必须编写：

- 单元测试。
- Provider 状态测试。
- ViewModel 测试。
- Repository 测试。
- Service 测试。
- Widget 测试。
- 路由测试。
- 数据库测试。
- 数据库迁移测试。
- REST API 测试。
- WebSocket 集成测试。
- Protobuf 跨语言兼容测试。
- BLE 真机测试。
- LAN 真机测试。
- 安全测试。
- 压力测试。
- 异常恢复测试。

重点覆盖：

- 无互联网启动。
- 无服务器启动。
- 本地身份创建。
- BLE 双向通信。
- LAN 双向通信。
- 互联网远程通信。
- 通信链路切换。
- ACK 丢失。
- 消息重复到达。
- 消息乱序。
- 消息重试。
- 网络断开恢复。
- 应用重启。
- 服务端重启。
- 数据库故障。
- Token 过期。
- 设备密钥变更。
- 设备撤销。
- 越权访问。
- 非法数据包。
- 超大数据包。
- 多账号数据隔离。
- 本地会话禁止上传。
- 页面销毁后通信服务继续运行。
- Provider 生命周期正确性。

允许使用 FakeTransport、MockRepository 等模拟对象进行自动化测试。

模拟测试不得替代真实 BLE、LAN 及互联网通信验收。

禁止将没有执行的测试标记为通过。

## 29. AI 开发行为规范

AI 在开发过程中必须严格遵守以下规则。

### 29.1 开发前

1. 完整阅读 AGENTS.md。
2. 阅读对应客户端或服务端计划书。
3. 检查 DEVELOPMENT_PROGRESS.md。
4. 检查已有目录及代码。
5. 检查现有协议定义。
6. 检查依赖及配置。
7. 明确当前任务范围。
8. 分析任务依赖关系。
9. 优先复用已有实现。
10. 对复杂任务建立明确的执行步骤及验收标准。

### 29.2 开发过程中

1. 不得未经授权修改整体架构及核心技术栈。
2. 不得删除与当前任务无关的代码。
3. 不得对无关模块进行大范围重构。
4. 不得为了通过编译而移除关键业务逻辑。
5. 不得使用假数据冒充真实业务。
6. 不得使用模拟通信冒充真实 BLE、LAN 或 WebSocket 通信。
7. 不得将空方法、TODO 及占位页面标记为完成。
8. 不得绕过身份认证、权限验证及安全校验。
9. 不得自行发明未经审查的加密协议。
10. 不得重复创建已有公共组件。
11. 不得重复实现已有业务服务。
12. 不得随意改变公共接口。
13. 不得直接修改自动生成代码。
14. 不得引入未经验证存在的依赖。
15. 不得提交真实密码、Token 及密钥。
16. 不得未经授权执行破坏性数据库操作。
17. 不得用临时补丁替代必要的根本问题修复。
18. 不得将重要错误静默忽略。
19. 必须保证客户端、服务端及共享协议一致。
20. 必须维护清晰的模块边界及依赖关系。
21. 必须处理合理的异常情况。
22. 必须为新增核心业务编写测试。
23. 必须确保应用长期运行服务具有正确生命周期。
24. 必须保持通信底层独立于 UI。
25. 必须保留计划书要求的全部核心通信能力。
26. 必须避免无意义的过度设计。
27. 必须采用实际可运行的实现，而不是仅提供示例或伪代码。

### 29.3 开发完成后

必须执行对应检查。

Flutter：

```bash
dart format .
flutter analyze
flutter test
```

根据实际环境执行对应平台构建及集成测试。

Spring Boot：

```bash
./mvnw verify
```

Windows 环境使用对应 Maven Wrapper 命令。

共享协议变更后必须执行代码生成及跨语言兼容测试。

每次完成任务后必须：

1. 检查编译结果。
2. 检查静态分析结果。
3. 执行相关测试。
4. 检查是否破坏已有功能。
5. 更新开发进度。
6. 更新必要文档。
7. 记录已完成及未完成内容。
8. 记录测试结果。
9. 记录存在的风险及阻塞。
10. 明确下一步开发任务。

### 29.4 自主决策范围

AI 可以在不破坏整体架构的情况下自主决定：

- 内部变量命名。
- 私有方法拆分。
- 公共组件合理封装。
- 局部性能优化。
- 非关键依赖实现细节。
- 测试实现方式。
- 合理的异常处理策略。
- 非破坏性代码组织调整。

AI 不得擅自决定：

- 更换状态管理框架。
- 更换数据库。
- 更换通信协议。
- 删除核心功能。
- 更改身份模型。
- 更换核心加密方案。
- 修改重要隐私策略。
- 删除用户数据。
- 重构整个项目架构。
- 将本地通信改为强制依赖服务器。
- 执行不可逆的数据迁移。
- 向未经授权的外部服务上传数据。

遇到重大架构冲突时，应先记录问题及建议，并继续完成不受影响的任务。

## 30. 开发阶段及优先级

### 阶段 0：公共基础设施

- 项目初始化。
- Provider + MVVM 架构。
- Spring Boot 模块化架构。
- 共享 Protobuf 协议。
- 身份模型。
- 数据库设计。
- 环境配置。
- 测试框架。
- 日志框架。
- 安全技术选型。

### 阶段 1：核心通信验证

- 本地身份创建。
- BLE 双角色通信。
- Android 与 iOS BLE 互通。
- LAN 设备发现。
- TCP 双向通信。
- WebSocket 双向通信。
- 平台能力检测。

必须尽早验证 BLE 和 LAN 的真实通信能力。

禁止先完成所有 UI 后再验证核心通信可行性。

### 阶段 2：核心消息系统

- TransportManager。
- ConnectionManager。
- RoutingService。
- MessageService。
- OutboxService。
- InboxService。
- 消息持久化。
- 消息确认。
- 消息去重。
- 消息重试。
- 单人聊天。
- 安全握手及加密会话。

### 阶段 3：完整核心通信

- BLE 真实消息收发。
- LAN 真实消息收发。
- WebSocket 真实消息收发。
- 多通道自动切换。
- 网络异常恢复。
- 纯离线模式。
- 本地与云端会话隔离。
- 统一联系人身份。

### 阶段 4：账号及社交功能

- 用户注册。
- 用户登录。
- 本地身份绑定。
- 好友关系。
- 附近用户。
- 二维码添加。
- 群组。
- 多设备管理。
- 消息同步。

### 阶段 5：丰富消息及文件

- 图片消息。
- 视频消息。
- 语音消息。
- 文件传输。
- 分片上传。
- 断点续传。
- 消息编辑。
- 消息撤回。
- 消息收藏。
- 消息搜索。

### 阶段 6：高级通信能力

- WebRTC。
- 语音通话。
- 视频通话。
- 局域网通话。
- 系统推送。
- 桌面端适配。
- Wi-Fi Direct。
- P2P 通信。
- Mesh 多跳通信。

### 阶段 7：发布及稳定性

- 安全审查。
- 性能优化。
- 耗电测试。
- 内存泄漏测试。
- 多平台兼容性测试。
- 服务端压力测试。
- 数据库迁移测试。
- 故障恢复测试。
- 部署及发布。

具体开发任务允许在满足依赖关系及安全要求的前提下并行执行。

## 31. 开发文档管理

必须维护以下文档：

| 文件 | 用途 |
|---|---|
| README.md | 项目说明及启动方式 |
| AGENTS.md | 全局开发规范 |
| FLUTTER_PLAN.md | Flutter 客户端开发计划 |
| SPRING_BOOT_PLAN.md | Spring Boot 服务端开发计划 |
| ARCHITECTURE.md | 系统架构 |
| PROTOCOL.md | 通信协议 |
| DATABASE.md | 数据库设计 |
| API.md | 服务端接口 |
| SECURITY.md | 安全模型 |
| PLATFORM_SUPPORT.md | 平台能力支持 |
| DEVELOPMENT_PROGRESS.md | 开发进度 |
| TEST_REPORT.md | 测试结果 |
| CHANGELOG.md | 项目变更 |
| docs/adr/ | 架构决策记录 |

开发进度状态统一使用：

- TODO：未开发。
- IN_PROGRESS：开发中。
- IMPLEMENTED：已实现但尚未完成验证。
- VERIFIED：已通过规定测试。
- BLOCKED：存在阻塞。
- DEFERRED：延期至后续版本。

每个开发任务必须记录：

- 任务名称。
- 所属模块。
- 优先级。
- 当前状态。
- 依赖任务。
- 实现内容。
- 测试结果。
- 已知问题。
- 下一步操作。

未经实际测试不得标记为 VERIFIED。

AI 每次结束开发会话前必须更新开发进度，确保其他 AI 或开发人员能够继续当前工作。

## 32. Git 与版本管理

提交代码前必须检查修改范围。

禁止将无关修改混入当前任务。

提交信息应具有明确含义。

建议使用：

- feat：新增功能。
- fix：修复问题。
- refactor：代码重构。
- perf：性能优化。
- test：测试相关。
- docs：文档更新。
- chore：工程维护。
- build：构建相关。
- ci：持续集成相关。

禁止提交：

- 真实生产环境密钥。
- 用户密码。
- Token。
- 本地数据库。
- 用户聊天记录。
- 敏感日志。
- 无必要的构建产物。

禁止未经授权执行强制推送、破坏性重置、历史重写及删除重要分支。

## 33. 最低可用版本验收标准

SailChat 的 MVP 必须同时满足以下条件：

1. 支持在线注册与登录。
2. 支持无服务器情况下创建本地身份。
3. 支持完全离线进入应用。
4. 两台支持相应 BLE 能力的手机可以在没有互联网的情况下发送文字消息。
5. 两台设备可以在没有互联网的局域网内发送消息。
6. 不同网络环境的用户可以通过服务器进行远程聊天。
7. 同一个已验证联系人可以在多种通信链路之间切换。
8. 同一消息通过多种链路传输时不会重复展示。
9. 消息具有持久化、确认、重试及去重能力。
10. 应用重启后可以恢复聊天记录。
11. 应用重启后可以恢复待发送任务。
12. 服务端停止后附近通信仍然可用。
13. 纯本地会话不会自动上传服务器。
14. 恶意或未认证设备无法冒用已信任用户身份。
15. 用户可以明确控制通信模式。
16. 页面销毁后全局通信服务仍然正常运行。
17. 不同账号的数据及通信连接能够正确隔离。
18. Flutter 使用 Provider + MVVM 架构。
19. Spring Boot 服务端能够实际部署及运行。
20. 客户端及服务端使用统一 Protobuf 协议。
21. 不依赖模拟数据通过核心功能验收。
22. 不依赖未经验证的加密实现通过安全验收。

只有同时实现 BLE、LAN 和互联网三类核心通信能力，才能将项目标记为 MVP 完成。

## 34. 最终交付要求

完整项目必须交付：

- Flutter 客户端完整源代码。
- Spring Boot 服务端完整源代码。
- 共享 Protobuf 协议。
- Android 原生通信插件。
- iOS 原生通信插件。
- 局域网通信模块。
- WebSocket 通信模块。
- 统一通信调度系统。
- 本地身份系统。
- 在线账号系统。
- 消息可靠性机制。
- 联系人及群组系统。
- 文件传输系统。
- 本地数据库及迁移。
- 服务端数据库及迁移。
- 自动化测试。
- 真机测试记录。
- 安全设计文档。
- 平台兼容性文档。
- API 接口文档。
- 通信协议文档。
- Docker 部署配置。
- 构建及发布说明。
- 开发进度及测试报告。

禁止仅交付 UI 原型、模拟聊天程序、空接口或普通 WebSocket 即时通讯软件。

## 35. 最终核心约束

SailChat 的核心竞争力在于多通信链路融合，而不是某一种单独的通信技术。

所有通信方式必须服务于统一的身份系统、联系人系统、会话系统及消息系统。

用户不需要关心底层正在使用 BLE、LAN、WebSocket 或其他通信方式。

只要存在符合用户隐私设置、安全要求及目标设备可达性的通信链路，系统就应选择合适的通信方式完成消息交换。

任何开发行为都不得破坏以下核心能力：

- 无服务器通信。
- 无互联网通信。
- 附近设备通信。
- 互联网远程通信。
- 统一用户身份。
- 统一消息及会话。
- 可靠消息投递。
- 安全与隐私。
- 多平台扩展能力。

以上规范适用于 SailChat 项目的全部开发阶段及所有子项目。