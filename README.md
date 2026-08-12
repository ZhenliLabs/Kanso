<div align="center">

# Kanso

**A modular, type-safe Rust-native backend application framework for building reliable business applications.**

*Reliable Rust applications, with less framework ceremony.*

[简体中文](./README.zh-CN.md)

</div>

> [!IMPORTANT]
> **Kanso is in Pre-Alpha / Phase 0: architecture design and executable spikes.**
>
> There is no usable release, stable API, installable CLI, or published crate. Commands and APIs below are design targets and may change after prototype results.

## What is Kanso?

Kanso is being built for developers and teams who want to create business applications in Rust—from focused Web APIs and modular monoliths to systems that can evolve toward SaaS and distributed deployments.

Its goal is to provide a coherent, progressively adoptable application model rather than another thin HTTP wrapper. Kanso brings common backend concerns into one workflow while preserving Rust's ownership model, type system, and Cargo-native development experience.

The planned framework capabilities include:

- typed data access and transactions;
- migrations;
- validation;
- OpenAPI metadata;
- Web endpoints and optional CRUD workflows;
- application lifecycle and managed tasks;
- deterministic, auditable generated artifacts.

Kanso is built around typed Resources, explicit Operations and Capabilities, statically composed Modules, and a Cargo-native workflow. Ownership, dependencies, errors, and escape hatches should remain explicit throughout the application.

The first deliverable is deliberately narrower than that long-term position:

> Define a Rust Resource and obtain a runnable, testable, extensible PostgreSQL CRUD API in about ten minutes—without mandatory Controller / Service / Repository ceremony.

Phase 1 starts with Axum and PostgreSQL so the application model can be validated end to end. These are foundation adapters, not the definition or permanent limit of Kanso. This is a success criterion, not a claim about the current repository.

## Why Kanso?

Rust already has excellent foundations for reliable software. Building an application still requires repeatedly connecting routing, state, validation, persistence, transactions, migrations, error mapping, OpenAPI, lifecycle management, tasks, and tests.

Kanso aims to reduce that application-level glue without hiding Rust behind a foreign framework model.

Its design is built around:

- meaningful types and explicit state transitions;
- visible ownership and transaction boundaries;
- explicit capabilities instead of a runtime DI container;
- static composition instead of runtime scanning;
- local derives and versioned cross-resource metadata;
- readable generation with clear file ownership;
- managed cancellation, backpressure, and shutdown;
- native access to the underlying Rust ecosystem.

The central goal is:

> **Rust should make invalid application states harder to represent, not merely make servers faster.**

## Core model

### Application model

| Concept | Responsibility |
|---|---|
| **Resource** | Application-defined business data described by the framework; it may be persisted or exposed, but is not automatically a domain aggregate or wire contract |
| **Value** | A newtype, enum, or value object that carries business invariants |
| **Operation** | A typed business action with explicit input, output, error, and effect boundaries |
| **Capability** | An external ability explicitly required by an Operation |
| **Endpoint** | A transport adapter for HTTP and, in later phases, gRPC, jobs, or CLI commands |
| **Module** | An explicit composition unit for Resources, Operations, Endpoints, and capability requirements |

### Infrastructure primitives

| Concept | Responsibility |
|---|---|
| **Store** | Typed persistence operations for a Resource; not a mandatory Repository interface |
| **TaskScope** | Managed tasks with cancellation, deadlines, joining, and shutdown policy |

Simple CRUD should remain simple. Additional abstraction boundaries should appear only when the domain requires them.

## Design sketch

> [!NOTE]
> The following syntax is conceptual. It does not compile today.

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

After the Phase 0 design is validated, compiler-checked Resource metadata may drive:

```text
Resource companion types
New / Patch / View types
Typed field markers
Typed Store operations
Initial migration
Validation metadata
Optional CRUD endpoints
OpenAPI metadata
Tests and snapshots
```

The intended application path is:

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

## What Kanso is not

Kanso is not intended to be:

- a replacement for Tokio, Axum, Tower, SQLx, Serde, or tracing;
- a new async runtime, HTTP/TLS stack, or database driver;
- a runtime dependency-injection container or service locator;
- an annotation scanner or implicit global registry;
- a Java-style `Controller → Service → Repository` hierarchy in Rust syntax;
- an Active Record API that hides I/O in domain instances;
- opaque code generation that overwrites user-owned code;
- a mandatory microservice architecture;
- a full-stack frontend framework;
- usable in production today.

## Escape hatches

Kanso treats escape hatches as product features. Applications must be able to use native:

- Axum routers and extractors;
- Tower services and layers;
- SQLx queries, pools, connections, and transactions;
- Tokio primitives when a managed abstraction is not appropriate.

A missing framework feature should not make an application impossible to build.

## Cargo-native workflow

Kanso will complement Cargo rather than replace it:

```text
cargo build
cargo check
cargo test
cargo clippy
cargo doc
```

Planned framework-specific operations will be exposed through the `cargo-kanso` binary:

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

These commands are not available yet.

## Roadmap

| Phase | Goal | Status |
|---|---|---|
| **0** | Validate metadata, generation, Store, Endpoint, Capability, and TaskScope architecture with executable spikes | **Current** |
| **1** | Deliver the first typed PostgreSQL Web API productivity kernel | Planned |
| **2** | Add enterprise modular-application capabilities | Deferred |
| **3** | Explore typed SaaS modules | Deferred |
| **4** | Explore cloud-native and distributed-system adapters | Deferred |
| **5** | Explore ecosystem and platform tooling | Deferred |

See the canonical [Roadmap](./ROADMAP.md) for phase scope and exit criteria.

## Documentation

- [Product vision](./docs/vision.md)
- [Architecture](./docs/architecture.md)
- [Engineering policy](./docs/engineering-policy.md)
- [References](./docs/references.md)
- [Phase 0 / Phase 1 backlog](./docs/backlog.md)
- [Architecture Decision Records](./docs/adr/README.md)
- [Documentation index](./docs/README.md)

The original combined v0.5 design document remains available as a historical source, but it is no longer canonical.

## Contributing during Phase 0

The most valuable early contributions are evidence-backed architecture feedback:

- challenge an assumption with a concrete Rust use case;
- implement a small competing prototype;
- test compiler diagnostics and rust-analyzer behavior;
- measure clean and incremental compile time or binary size;
- identify ownership, cancellation, transaction, or generated-code failure modes.

Before opening a large implementation pull request, please open an issue describing the problem, constraints, and proposed experiment.

- [Issues](https://github.com/ZhenliLabs/Kanso/issues)
- [Zhenli Labs](https://github.com/ZhenliLabs)

## Stability

Everything may change during Phase 0, including crate layout, Resource syntax, generated source, query APIs, CLI commands, and Module/Capability contracts.

Kanso will not claim API stability until those contracts have been exercised by working examples and external users.

## License

Kanso is intended to be released as open-source software under **MIT OR Apache-2.0**. No open-source license has been granted until the license files are added.

## Maintainers

Kanso is initiated and maintained by [Zhenli Labs](https://github.com/ZhenliLabs).

---

<div align="center">

**Kanso is not ready yet—but its design is open for review.**

</div>
