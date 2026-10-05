---
name: sdd4j-bce
description: SDD4J architecture adapter for Boundary-Control-Entity Java components. Use when a Java project organizes capabilities as BCE business components with boundary, control, and entity layers, or when mapping package-info.java specs to BCE code. Trigger on sdd4j-bce, BCE architecture adapter, boundary-control-entity, business component mapping, or SDD4J with BCE.
metadata:
  type: architecture-adapter
---

# Skill: BCE Architecture Adapter

## Objective

Map a Java capability spec to a Boundary-Control-Entity business component. This skill is the SDD4J-facing architecture adapter for BCE. It does not replace a broader BCE design skill; it states the concrete mapping SDD4J needs for package docs, operations, entities, and drift.

Compose it with `sdd4j` for spec workflow and with a Java stack skill for implementation idioms and verification.

Use this adapter only when the project accepts strict BCE mapping. For Java BCE projects, `sdd4j + sdd4j-bce + <stack>` should be behaviorally equivalent to an integrated spec-driven BCE workflow while preserving SDD4J's pluggable architecture model.

## Architecture Invariants

- The project or module has `sdd4j-bce` as its primary SDD4J architecture adapter.
- One SDD4J capability maps to one BCE business component.
- The capability name is the BC name unless `AGENTS.md` declares a concrete naming rule.
- The component package is the BC ownership boundary.
- The capability spec lives in the component package's `package-info.java`.
- `## Boundary` maps only to the component's boundary layer.
- `## Requirements` maps to behavior and traceable tests.
- `## Entities` maps to contract-relevant domain/entity types in the entity layer.
- `control` is implementation detail and never gets a spec section.
- Do not mix this adapter with feature-package or generic layered mapping inside the same capability.

## BCE Parity

When composed with SDD4J, this adapter must preserve BCE behavior:

| BCE rule | SDD4J+BCE realization |
| --- | --- |
| Spec is the boundary contract | `package-info.java` is the component contract |
| One capability equals one BC | Capability name resolves to one component package |
| Boundary operations are public contract | `## Boundary` maps to boundary-layer operations |
| Requirements are mandatory behavior | `## Requirements` maps to implementation and tests |
| Entities are contract-relevant domain concepts | `## Entities` maps to entity-layer types |
| Control is implementation | No `## Control` section is allowed |

## Default Layout

```text
src/main/java/<base>/<component>/
  package-info.java
  boundary/
  control/
  entity/

src/test/java/<base>/<component>/
```

The component package is the business component. The final package segment is the component name unless `AGENTS.md` declares another mapping. A project that cannot identify one component package per capability is not using strict BCE for SDD4J purposes.

## SDD4J Mapping

When composed with SDD4J:

- An SDD4J capability maps to one BCE business component.
- The capability name is the BCE component package name.
- The spec lives in the component package's `package-info.java`.
- `## Boundary` operations map to public methods, commands, resources, handlers, or ports in the `boundary` layer.
- `## Requirements` maps to behavior and traceable tests.
- `## Entities` maps to domain/entity types in the `entity` layer.
- The `control` layer is implementation detail and has no spec section.
- Requirement ids must resolve to their exact runner-visible forms in executable tests or cases according to the stack's trace convention. Traces may use literal ids or resolvable symbols; JavaDoc and comments alone do not count.

## Layer Responsibilities

- `boundary` exposes the component contract to callers, transports, schedulers, message brokers, or other components.
- `control` coordinates use cases and policies needed to satisfy the boundary contract.
- `entity` owns domain state, invariants, value types, and domain behavior relevant to the component.

Keep framework and infrastructure details outside the spec unless they are part of the user-visible capability contract.

## Operation Mapping

Map each SDD4J `## Boundary` operation to one stable boundary-layer operation.

Examples:

```text
`place-order` -> boundary.PlaceOrder.placeOrder(...)
`cancel-order` -> boundary.OrderResource.cancelOrder(...)
`expire-reservations` -> boundary.ReservationExpiryJob.expireReservations(...)
```

The exact class shape belongs to the stack and BCE coding conventions. The adapter only requires the operation to be discoverable in the boundary layer.

## Entity Mapping

Map each `## Entities` item to one or more contract-relevant types under `entity`.

Do not treat DTOs, persistence records, generated classes, clients, or mappers as SDD4J entities unless the project mapping explicitly says they are part of the capability contract.

## Drift Detection

Detect spec-to-code gaps:

- A `## Boundary` operation has no discoverable boundary-layer operation.
- A `## Entities` item has no corresponding type in `entity` when it should be materialized.
- A requirement id has no executable test trace.

Detect code-to-spec drift:

- A public boundary-layer operation has no matching `## Boundary` operation.
- A contract-relevant entity-layer type is absent from `## Entities`.
- A test references a requirement id not present in the component spec.
- Contract-relevant component behavior exists outside `boundary`, `control`, or `entity` without an explicit project exception.

Surface inverse drift to the user. Do not silently expand the spec to match code.

## Fit And Rejection Rules

This adapter fits Java projects that intentionally organize capabilities as BCE business components with component-local `boundary`, `control`, and `entity` layers.

Reject or ask for a different mapping when:

- A capability cannot be mapped to one business component.
- Component packages do not own their boundary/control/entity structure.
- Operations are primarily owned by global controller/service/repository layers; use `sdd4j-package-by-layer`.
- Capabilities are primarily self-contained feature packages without BCE layer semantics; use `sdd4j-package-by-feature`.
- Multiple modules use different architectures without explicit routing in `AGENTS.md`.

## Cross-Component Rules

SDD4J specs describe one component at a time. Cross-component calls, events, shared nouns, and system invariants belong in a system-level package doc or project-level architecture documentation. Do not duplicate another component's requirements in this component's spec.

## AGENTS.md Mapping Template

```md
## SDD4J

Architecture layout:
- skill: `sdd4j-bce`
- scope: primary project architecture
- source root: `src/main/java`
- base package: `com.acme`
- component package pattern: `com.acme.<component>`
- boundary package: `boundary`
- control package: `control`
- entity package: `entity`

Traceability:
- trace convention: per-component `{Component}Requirement` annotation with a nested `Rn` enum; enum constants (e.g. `R1_2`) resolve to the literal `Rn.m` id
- requirement id must resolve to its exact runner-visible `Rn.m` form through a display name, case label, symbol, or annotation consumed by the test/reporting infrastructure
- a normalized Java identifier such as `R1_2` is valid only when the configured infrastructure resolves and displays it as `R1.2`
- JavaDoc and comments alone do not count
```

If the repository already has a separate BCE skill or convention document, follow it for detailed BCE coding rules and use this skill only for SDD4J mapping decisions.
