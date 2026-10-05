---
name: sdd4j-hexagonal
description: SDD4J architecture adapter for hexagonal (ports and adapters) Java projects. Use when a Java project organizes capabilities as hexagonal modules with domain, application ports and use cases, and driving/driven adapters, or when mapping package-info.java specs to ports-and-adapters code. Trigger on sdd4j-hexagonal, hexagonal architecture, ports and adapters, clean architecture adapter, onion architecture, or SDD4J with hexagonal.
metadata:
  type: architecture-adapter
---

# Skill: Hexagonal Architecture Adapter

## Objective

Map a Java capability spec to a hexagonal (ports and adapters) module: domain core, application ports and use cases, and driving/driven adapters. This skill is the SDD4J-facing architecture adapter for hexagonal projects. It owns the concrete mapping SDD4J needs for package docs, operations, entities, and drift — including the dependency rule that makes the layout meaningful.

Compose it with `sdd4j` for spec workflow and with a Java stack skill for implementation idioms and verification.

This adapter is opinionated: it mandates one canonical package pattern so the layout is predictable for both agents and humans. Projects whose hexagonal naming deviates from the canonical shape declare the mapping in `AGENTS.md` rather than expecting this adapter to recognize every convention.

## Architecture Invariants

- The project or module has `sdd4j-hexagonal` as its primary SDD4J architecture adapter.
- One SDD4J capability maps to one hexagonal module package.
- The capability name is the module package name unless `AGENTS.md` declares a concrete naming rule.
- The module package is the capability ownership boundary.
- The capability spec lives in the module package's `package-info.java`.
- `## Boundary` maps only to inbound port operations (`application.port.in` use case interfaces).
- `## Requirements` maps to behavior and traceable tests.
- `## Entities` maps to contract-relevant types in `domain`.
- `application.service` (use case implementations), `adapter`, and `application.port.out` are implementation detail and never get a spec section.
- Dependencies point inward: adapters may depend on ports and domain; domain depends on no adapter, port, or framework package.
- Do not mix this adapter with feature-package, BCE, or generic layered mapping inside the same capability.

## Hexagonal Parity

When composed with SDD4J, this adapter must preserve hexagonal behavior:

| Hexagonal rule | SDD4J+hexagonal realization |
| --- | --- |
| Spec is the inbound contract | `package-info.java` is the module contract |
| One capability equals one module | Capability name resolves to one module package |
| Inbound ports are the public contract | `## Boundary` maps to `port.in` use case operations |
| Requirements are mandatory behavior | `## Requirements` maps to implementation and tests |
| Domain is contract-relevant state | `## Entities` maps to `domain` types |
| Services, outbound ports, adapters are implementation | No spec section for `application.service`, `port.out`, or `adapter` |
| Dependencies point inward | Domain has no dependency on ports, adapters, or frameworks |

## Default Layout

Use this shape unless the project's `AGENTS.md` declares a different mapping:

```text
src/main/java/<base>/<capability>/
  package-info.java
  domain/
    <DomainType>.java
  application/
    port/
      in/
        <Operation>UseCase.java        # inbound port interfaces
      out/
        Load<DomainType>Port.java      # outbound port interfaces
    service/
      <Capability>Service.java         # use case implementations
  adapter/
    in/
      web/                             # driving adapters (REST, controllers)
      messaging/                       # driving adapters (consumers, listeners)
    out/
      persistence/                     # driven adapters (repositories)
      client/                          # driven adapters (external clients)

src/test/java/<base>/<capability>/
```

The module package is the capability. The final package segment is the capability name unless `AGENTS.md` declares another mapping. A project without `port` and `adapter` separation is not hexagonal for SDD4J purposes — use `sdd4j-package-by-layer` or `sdd4j-package-by-feature` instead.

## SDD4J Mapping

When composed with SDD4J:

- An SDD4J capability maps to one hexagonal module package.
- The capability name is the module package name.
- The spec lives in the module package's `package-info.java`.
- `## Boundary` operations map to methods on `application.port.in` use case interfaces. Driving adapters expose or invoke them but carry no contract themselves.
- `## Requirements` maps to behavior and traceable tests.
- `## Entities` maps to domain types in `domain`.
- `application.service` use case implementations, `application.port.out` interfaces, and everything under `adapter` are implementation detail and have no spec section.
- Outbound ports describe what the capability *needs*, not what it promises; they become contract-relevant only when a system doc or `AGENTS.md` declares them so (e.g. an integration event schema another capability consumes).
- Requirement ids must resolve to their exact runner-visible forms in executable tests or cases according to the stack's trace convention. Traces may use literal ids or resolvable symbols; JavaDoc and comments alone do not count.

## Layer Responsibilities

- `domain` owns domain state, invariants, value types, and domain behavior. No framework or adapter imports.
- `application.port.in` declares the use case interfaces driving adapters call — the capability's public contract.
- `application.port.out` declares the interfaces the capability needs from driven infrastructure.
- `application.service` implements the use cases and orchestrates domain and outbound ports.
- `adapter.in` adapts transports and triggers (REST, messaging, CLI, schedulers) into inbound port calls.
- `adapter.out` implements outbound ports against persistence, clients, or brokers.

Keep framework and infrastructure details outside the spec unless they are part of the user-visible capability contract.

## Operation Mapping

Map each SDD4J `## Boundary` operation to one stable inbound port operation.

Examples:

```text
`place-order` -> port.in.PlaceOrderUseCase.placeOrder(...)
`cancel-order` -> port.in.CancelOrderUseCase.cancelOrder(...)
`list-orders` -> port.in.ListOrdersQuery.listOrders(...)
```

The exact interface shape belongs to the stack and project conventions. The adapter only requires the operation to be discoverable on an `application.port.in` interface.

## Entity Mapping

Map each `## Entities` item to one or more contract-relevant types under `domain`.

Do not treat persistence records, DTOs, generated classes, client models, or mappers in `adapter` packages as SDD4J entities unless the project mapping explicitly says they are part of the capability contract.

## Drift Detection

Detect spec-to-code gaps:

- A `## Boundary` operation has no discoverable inbound port operation.
- A `## Entities` item has no corresponding type in `domain` when it should be materialized.
- A requirement id has no executable test trace.

Detect code-to-spec drift:

- A public inbound port operation has no matching `## Boundary` operation.
- A contract-relevant domain type is absent from `## Entities`.
- A test references a requirement id not present in the capability spec.
- Contract-relevant capability behavior exists outside `domain`, `application`, and `adapter` without an explicit project exception.

Detect structural drift against the dependency rule:

- A type in `domain` imports an `adapter`, `application`, or framework package.
- A type in `application` imports an `adapter` package.
- A driving adapter bypasses `port.in` to call `application.service` or `domain` internals directly.
- An adapter in `adapter.in` calls an adapter in `adapter.out` directly instead of going through the core.

Surface inverse drift to the user. Do not silently expand the spec to match code.

## Setup Bias

Prefer explicit `AGENTS.md` mapping when the project deviates from the canonical layout. Naming variants such as `inbound`/`outbound`, `ports`/`adapters`, or `primary`/`secondary` are parameters to declare, not new shapes.

During `/sdd4j setup`, ask for or confirm:

- Module package pattern.
- Domain package name.
- Inbound and outbound port package names.
- Use case implementation package name.
- Driving and driven adapter package names.
- Whether outbound ports or adapter types are ever contract-relevant.
- Test trace convention — recorded in the `trace convention` field (e.g. the `{Capability}Requirement` annotation with nested `Rn` enum, or literal display-name ids).

When the scan finds layer roots (`application`, `domain`, `infrastructure`) *and* `port`/`adapter` packages, the layout reads as both layered and hexagonal — present the evidence and ask which semantics to enforce rather than guessing. Absence of `port`/`adapter` packages rules this adapter out.

## Fit And Rejection Rules

This adapter fits Java projects that intentionally organize capabilities as hexagonal modules with explicit ports and adapters.

Reject or ask for a different mapping when:

- A capability cannot be mapped to one module package.
- The project has no `port`/`adapter` separation; use `sdd4j-package-by-layer` or `sdd4j-package-by-feature`.
- Operations are primarily owned by global controller/service/repository layers; use `sdd4j-package-by-layer`.
- Capabilities are primarily self-contained feature packages without port/adapter semantics; use `sdd4j-package-by-feature`.
- The project requires BCE semantics with `boundary`, `control`, and `entity` layers per business component; use `sdd4j-bce`.
- Multiple modules use different architectures without explicit routing in `AGENTS.md`.

## Cross-Module Rules

SDD4J specs describe one capability at a time. Cross-module calls through another module's inbound port, integration events, shared nouns, and system invariants belong in a system-level package doc or project-level architecture documentation. Do not duplicate another capability's requirements in this capability's spec.

## AGENTS.md Mapping Template

```md
## SDD4J

Architecture layout:
- skill: `sdd4j-hexagonal`
- scope: primary project architecture
- source root: `src/main/java`
- base package: `com.acme`
- module package pattern: `com.acme.<capability>`
- domain package: `domain`
- inbound port package: `application.port.in`
- outbound port package: `application.port.out`
- use case package: `application.service`
- driving adapter package: `adapter.in`
- driven adapter package: `adapter.out`

Traceability:
- trace convention: per-capability `{Capability}Requirement` annotation with a nested `Rn` enum; enum constants (e.g. `R1_2`) resolve to the literal `Rn.m` id
- requirement id must resolve to its exact runner-visible `Rn.m` form through a display name, case label, symbol, or annotation consumed by the test/reporting infrastructure
- a normalized Java identifier such as `R1_2` is valid only when the configured infrastructure resolves and displays it as `R1.2`
- JavaDoc and comments alone do not count
```

If the repository already has a separate hexagonal convention document, follow it for detailed coding rules and use this skill only for SDD4J mapping decisions.
