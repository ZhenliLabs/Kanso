<div align="center">

# Kanso

**一款用于构建可靠业务应用的模块化、类型安全的 Rust-native 后端应用框架。**

*少一些框架仪式，不牺牲 Rust 的清晰与可靠。*

[English](./README.md)

</div>

> [!IMPORTANT]
> **Kanso 当前处于 Pre-Alpha / Phase 0：架构设计与可执行技术验证阶段。**
>
> 目前没有可用版本、稳定 API、可安装 CLI 或已发布 crate。下文中的命令和 API 都是设计目标，可能根据原型结果调整。

## Kanso 是什么？

Kanso 正在为希望使用 Rust 构建业务应用的开发者与团队而建设：从专注的 Web API 和模块化单体，到可以逐步演进为 SaaS 或分布式部署的系统。

它的目标不是再包装一层 HTTP，而是提供一套一致、可渐进采用的应用模型。Kanso 在保留 Rust 所有权模型、类型系统与 Cargo-native 开发体验的同时，将常见后端能力组织到同一套工作流中。

规划中的框架能力包括：

- 强类型数据访问与事务；
- Migration；
- Validation；
- OpenAPI Metadata；
- Web Endpoint 与可选 CRUD 工作流；
- 应用生命周期与受管理任务；
- 确定、可审计的生成产物。

Kanso 以强类型 Resource、显式 Operation 与 Capability、静态组合的 Module 和 Cargo-native 工作流为核心。在整个应用中，所有权、依赖、错误和底层逃生口都应保持显式可见。

第一阶段的交付目标有意比这个长期定位更收敛：

> 定义一个 Rust Resource，在大约十分钟内得到可运行、可测试、可扩展的 PostgreSQL CRUD API，并且默认不引入 Controller / Service / Repository 仪式。

Phase 1 从 Axum 与 PostgreSQL 起步，用一条完整路径验证应用模型。它们是基础 Adapter，而不是 Kanso 的定义或永久边界。这是 Phase 1 的成功标准，不是对当前仓库能力的声明。

## 为什么要做 Kanso？

Rust 已经拥有构建可靠软件的优秀基础。开发一个完整应用仍然需要反复连接路由、状态、验证、持久化、事务、Migration、错误映射、OpenAPI、生命周期、异步任务和测试。

Kanso 希望减少这些应用层胶水，同时不使用一套外来的框架模型把 Rust 隐藏起来。

它围绕以下原则设计：

- 用有业务含义的类型和显式状态转换表达约束；
- 让所有权和事务边界可见；
- 使用显式 Capability，而不是 Runtime DI Container；
- 使用静态组合，而不是 Runtime Scanning；
- 局部 Derive 与版本化跨 Resource Metadata 分工；
- 生成代码可读，并具有明确的文件所有权；
- 管理取消、背压和优雅关闭；
- 始终允许直接使用底层 Rust 生态。

核心目标是：

> **Rust 不应只让服务器更快，也应让应用中的非法状态更难被表达。**

## 核心模型

### 应用模型

| 概念 | 职责 |
|---|---|
| **Resource** | 由应用定义、可被框架描述的业务数据；它可以被持久化或暴露，但不自动等同于 Domain Aggregate 或 Wire Contract |
| **Value** | 携带业务不变量的 Newtype、Enum 或 Value Object |
| **Operation** | 具有显式输入、输出、错误和副作用边界的业务动作 |
| **Capability** | Operation 明确要求的外部能力 |
| **Endpoint** | HTTP，以及未来 gRPC、Job 或 CLI 的 Transport Adapter |
| **Module** | Resource、Operation、Endpoint 与 Capability Requirement 的显式组合单元 |

### 基础设施原语

| 概念 | 职责 |
|---|---|
| **Store** | 面向 Resource 的强类型持久化操作，不等同于必须存在的 Repository Interface |
| **TaskScope** | 管理取消、Deadline、Join 与 Shutdown Policy 的并发作用域 |

简单 CRUD 应该保持简单；只有领域确实需要时才增加新的抽象边界。

## 设计草图

> [!NOTE]
> 下面的语法仅用于表达设计，目前不能编译。

```rust
#[derive(Resource)]
#[resource(table = "users")]
pub struct User {
    #[id]
    pub id: UserId,

    #[unique]
    pub email: Email,

    #[secret]
    pub password_hash: Secret<String>,

    #[default(UserStatus::Active)]
    pub status: UserStatus,

    #[created_at]
    pub created_at: DateTime<Utc>,
}
```

Phase 0 设计验证通过后，经过编译器检查的 Resource Metadata 可能驱动：

```text
Resource Companion Types
New / Patch / View Types
Typed Field Markers
Typed Store Operations
Initial Migration
Validation Metadata
Optional CRUD Endpoints
OpenAPI Metadata
Tests and Snapshots
```

计划中的应用调用路径是：

```text
Transport Input
     ↓ TryFrom / Validate
Operation Input
     ↓
Operation + Exact Capabilities
     ↓
Typed Store / External Capability
     ↓
Typed Output / Typed Error
     ↓ Transport Mapping
Transport Output
```

## Kanso 不是什么？

Kanso 不计划成为：

- Tokio、Axum、Tower、SQLx、Serde 或 tracing 的替代品；
- 新的 Async Runtime、HTTP/TLS Stack 或 Database Driver；
- Runtime DI Container 或 Service Locator；
- Annotation Scanner 或隐式全局 Registry；
- 把 Java 式 `Controller → Service → Repository` 换成 Rust 语法；
- 把 I/O 隐藏到 Domain Instance 中的 Active Record；
- 覆盖用户业务代码的黑盒 Codegen；
- 强迫所有项目使用微服务的架构；
- Full-stack Frontend Framework；
- 当前即可用于生产环境的产品。

## 底层逃生口

Kanso 把逃生口视为正式产品能力。应用必须能够直接使用：

- Axum Router 与 Extractor；
- Tower Service 与 Layer；
- SQLx Query、Pool、Connection 与 Transaction；
- 在受管理抽象不适用时使用 Tokio 原生能力。

框架缺少某个功能，不应该意味着项目无法继续开发。

## Cargo-native 工作流

Kanso 会补充而不是替代 Cargo：

```text
cargo build
cargo check
cargo test
cargo clippy
cargo doc
```

框架专用操作计划通过 `cargo-kanso` 二进制提供：

```text
cargo kanso new
cargo kanso dev
cargo kanso generate
cargo kanso schema check
cargo kanso codegen check
cargo kanso migrate
cargo kanso routes
cargo kanso doctor
```

这些命令目前还不能使用。

## Roadmap

| Phase | 目标 | 状态 |
|---|---|---|
| **0** | 用可执行 Spike 验证 Metadata、Codegen、Store、Endpoint、Capability 与 TaskScope 架构 | **当前阶段** |
| **1** | 交付第一版强类型 PostgreSQL Web API Productivity Kernel | Planned |
| **2** | 增加企业级模块化应用能力 | Deferred |
| **3** | 研究类型化 SaaS Module | Deferred |
| **4** | 研究 Cloud-native 与分布式系统 Adapter | Deferred |
| **5** | 研究生态与平台工具 | Deferred |

阶段范围和退出条件以 [Roadmap](./ROADMAP.md) 为准。

## 文档

- [产品愿景](./docs/vision.md)
- [技术架构](./docs/architecture.md)
- [工程策略](./docs/engineering-policy.md)
- [参考资料](./docs/references.md)
- [Phase 0 / Phase 1 Backlog](./docs/backlog.md)
- [Architecture Decision Records](./docs/adr/README.md)
- [文档索引](./docs/README.md)

原 v0.5 合并设计稿继续作为历史资料保留，但不再是规范事实来源。

## Phase 0 如何参与？

现在最有价值的是有证据支持的架构反馈：

- 使用具体 Rust 场景挑战某个假设；
- 实现小型竞争性原型；
- 测试 compiler diagnostics 与 rust-analyzer 体验；
- 测量 clean/incremental compile time 或 binary size；
- 找出 ownership、cancellation、transaction 或 generated-code failure mode。

提交大型实现 PR 前，请先创建 Issue，说明问题、约束和准备验证的方案。

- [Issues](https://github.com/ZhenliLabs/Kanso/issues)
- [Zhenli Labs](https://github.com/ZhenliLabs)

## 稳定性

Phase 0 期间所有内容都可能变化，包括 Crate Layout、Resource Syntax、Generated Source、Query API、CLI Command 以及 Module/Capability Contract。

只有当相关契约被可运行示例和外部用户验证后，Kanso 才会承诺 API 稳定性。

## License

Kanso 计划以 **MIT OR Apache-2.0** 开源发布。在加入许可证文件之前，项目尚未授予开源许可证。

## 维护者

Kanso 由 [Zhenli Labs](https://github.com/ZhenliLabs) 发起并维护。

---

<div align="center">

**Kanso 还没有准备好，但它的设计已经开放评审。**

</div>
