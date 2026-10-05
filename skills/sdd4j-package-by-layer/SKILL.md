---
name: sdd4j-package-by-layer
description: SDD4J architecture adapter for Java projects organized by technical layers such as controller, service, repository, model, domain, or dto. Use with SDD4J and a Java stack skill when specs need to map capabilities to classes spread across layer packages. Trigger on sdd4j-package-by-layer, package-by-layer, layered architecture, controller-service-repository layout, Spring-style layers, or SDD4J setup for existing layered Java projects.
metadata:
  type: architecture-adapter
---

# Skill: Package By Layer

## Objective

Map a Java capability spec to classes spread across technical layer packages. This adapter is intentionally more configuration-driven than sdd4j-package-by-feature because layered projects often need project-specific naming conventions.

Compose it with `sdd4j` for spec workflow and with a stack skill for framework idioms and verification.

Use this adapter only when the project accepts strict declared mapping: one SDD4J capability maps to one spec package plus the layer classes selected by the configured capability-to-class rule. Guessing by class names is a fallback for setup discovery, not a stable operating mode.

## Architecture Invariants

- The project or module has `sdd4j-package-by-layer` as its primary SDD4J architecture adapter.
- One SDD4J capability maps to one spec package and one declared set of layer classes.
- The spec package is the capability identity even when implementation code lives elsewhere.
- The capability spec lives in the spec package's `package-info.java`.
- Controller, service, repository, model, domain, DTO, and infrastructure packages are interpreted only through the project mapping.
- A class belongs to a capability only when the configured mapping says it does.
- Shared models and infrastructure must be declared as shared, excluded from capability drift checks, or mapped explicitly.
- Do not mix this adapter with feature-package or BCE mapping inside the same capability.

## Default Layout

Use this shape only when the project does not declare a different mapping:

```text
src/main/java/<base>/capabilities/<capability>/
  package-info.java

src/main/java/<base>/controller/
  <Capability>Controller.java

src/main/java/<base>/service/
  <Capability>Service.java

src/main/java/<base>/repository/
  <Capability>Repository.java

src/main/java/<base>/model/
  <DomainType>.java
```

The spec package is a contract home for the capability. The implementation may live in multiple layer packages, but only through a stable declared mapping.

## SDD4J Mapping

When composed with SDD4J:

- An SDD4J capability maps to one spec package plus zero or more layer classes.
- The spec lives at `<base>.capabilities.<capability>.package-info` by default.
- `## Boundary` operations map to controller, handler, listener, command, facade, or service methods according to project configuration.
- `## Entities` entries map to model, domain, DTO, entity, aggregate, or persistence classes according to project configuration.
- Repositories, clients, mappers, configuration, and persistence adapters are implementation unless the project mapping marks them contract-relevant.
- Requirement ids must resolve to their exact runner-visible forms in executable tests or cases that exercise the capability, even if those tests live outside the spec package. Traces may use literal ids or resolvable symbols; JavaDoc and comments alone do not count.

## Setup Bias

Prefer explicit `AGENTS.md` mapping for package-by-layer projects. Do not rely on class-name guessing when several capabilities share layer classes or when model names do not include the capability prefix.

During `/sdd4j setup`, ask for or confirm:

- Spec package root.
- Controller or entrypoint package roots.
- Service or application package roots.
- Model/domain/entity package roots.
- Repository or infrastructure package roots, if they matter for drift checks.
- Capability-to-class convention: prefix, suffix, annotation, package list, or explicit class list.
- Test trace convention — recorded in the `trace convention` field (e.g. the `{Capability}Requirement` annotation with nested `Rn` enum, or literal display-name ids).

## Capability Mapping Strategies

Use the first applicable strategy from the project mapping:

1. Explicit class list per capability.
2. Annotation or marker interface that names the capability.
3. Package pattern with a capability segment.
4. Class name prefix or suffix, such as `CheckoutController` and `CheckoutService`.
5. Manual confirmation for ambiguous classes.

Avoid assuming that every `Order` or `User` model belongs to a single capability. Shared domain objects should be declared in a system/shared doc or excluded from capability drift checks.

## Drift Detection

Detect spec-to-code gaps:

- A `## Boundary` operation has no mapped controller, handler, facade, or service operation.
- A `## Entities` item has no mapped model/domain/entity class when the mapping says it should be materialized.
- A requirement id has no executable test trace.

Detect code-to-spec drift:

- A mapped public entrypoint or service operation has no `## Boundary` operation.
- A mapped domain/model/entity class is contract-relevant but absent from `## Entities`.
- A test references a requirement id not present in the capability spec.
- A class appears to implement the capability but is not covered by any declared mapping rule.

When drift is ambiguous because the layer mapping is weak, report uncertainty and ask for a project mapping update instead of editing code or the spec speculatively.

## Fit And Rejection Rules

This adapter fits existing layered Java applications where capability code is intentionally spread across technical packages and the project can state a stable capability-to-class mapping.

Reject or ask for a different mapping when:

- The project cannot express how classes map to capabilities.
- A capability is better represented by a self-contained package; use `sdd4j-package-by-feature` for new code.
- The project requires BCE semantics with `boundary`, `control`, and `entity` layers per business component; use `sdd4j-bce`.
- Different modules use different architectures without explicit routing in `AGENTS.md`.

## AGENTS.md Mapping Template

```md
## SDD4J

Architecture layout:
- skill: `sdd4j-package-by-layer`
- scope: primary project architecture
- source root: `src/main/java`
- base package: `com.acme`
- spec package pattern: `com.acme.capabilities.<capability>`

Layer packages:
- entrypoints: `com.acme.controller`, `com.acme.messaging`
- application: `com.acme.service`
- entities: `com.acme.model`, `com.acme.domain`
- infrastructure: `com.acme.repository`, `com.acme.client`

Capability mapping:
- strategy: class name prefix
- example: `CheckoutController`, `CheckoutService`, `CheckoutRepository` map to `checkout`
- shared models: `User`, `Money` are excluded from capability entity drift unless declared

Traceability:
- trace convention: per-capability `{Capability}Requirement` annotation with a nested `Rn` enum; enum constants (e.g. `R1_2`) resolve to the literal `Rn.m` id
- requirement id must resolve to its exact runner-visible `Rn.m` form through a display name, case label, symbol, or annotation consumed by the test/reporting infrastructure
- a normalized Java identifier such as `R1_2` is valid only when the configured infrastructure resolves and displays it as `R1.2`
- JavaDoc and comments alone do not count
```

If the project cannot express a stable mapping, recommend sdd4j-package-by-feature for new capabilities while preserving sdd4j-package-by-layer for legacy code.
