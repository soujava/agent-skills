---
name: sdd4j
description: Spec-Driven Development for Java workflow using package-info.java as the co-located capability contract. Use when the user asks for SDD4J, spec-driven Java development, package-info.java specs, EARS requirements, traceable requirement tests, or wants to set up, create, apply, verify, or converge a capability spec across architectures such as sdd4j-package-by-feature, sdd4j-package-by-layer, or sdd4j-bce.
metadata:
  type: workflow
---

# Skill: SDD4J

## Objective

Drive Spec-Driven Development for Java without owning the project's architecture. SDD4J owns the workflow, the spec format, and traceability. The composed architecture skill owns how a capability maps to code. The composed stack skill owns Java idioms, build, run, and test verification.

Use SDD4J as:

```text
/sdd4j setup
/sdd4j new <capability-or-feature>
/sdd4j apply <capability>
/sdd4j verify <capability>
```

If the user clearly asks for this workflow without slash syntax, infer the matching mode.

## Core Invariants

- The spec is the capability contract: what the Java package promises, not how it is implemented.
- The spec lives in `package-info.java` using Markdown doc comments ([JEP 467](https://openjdk.org/jeps/467)) - each line prefixed `///`, ending with the `package …;` declaration; if the project does not support it, then use conventional Javadoc, ending with the `package …` declaration;
- One capability spec maps to one architecture-defined component, feature, package, module, or business component.
- One spec per capability is the single source of truth. Never create parallel specs for one capability.
- A project should have one primary SDD4J architecture adapter. Multiple adapters in one repository are exceptional and must be declared explicitly per module, package root, or capability set in `AGENTS.md`.
- The task list is the current gap between spec, code, and tests. Read it on demand; do not maintain a separate task file.
- `## Requirements` uses EARS statements with stable ids such as `R1.1`.
- Every requirement id must be represented by at least one executable test or case whose runner-visible identity resolves to the exact `Rn.m` id according to the stack or project trace convention. The trace may use the literal id or a symbol whose display form is the literal id. JavaDoc or comments alone do not count as traceability.
- Done means the stack verification is green, no structural gap or drift remains, and traced tests and implementation semantically conform to every requirement.
- SDD4J does not prescribe `boundary/control/entity`, `controller/service/repository`, or any other layout.
- SDD4J must obey stronger invariants imposed by the selected architecture adapter. Do not weaken an adapter's contract to make an existing project fit silently.

## Composition Model

Resolve three roles before writing or applying code:

```text
SDD4J workflow
  spec format, setup/new/apply/verify, EARS, traceability, gap loop

Architecture adapter
  package layout, capability location, operation mapping, entity/model mapping, drift rules

Stack skill
  Java framework conventions, build tool, test command, runtime verification
```

Examples:

```text
sdd4j + sdd4j-package-by-feature + spring-boot-server
sdd4j + sdd4j-package-by-layer + spring-boot-server
sdd4j + sdd4j-package-by-feature + microprofile-server
sdd4j + sdd4j-bce + java-cli-app
```

## Architecture Adapter Contract

The architecture skill or the project's `AGENTS.md` must answer these questions:

- What is the project's primary SDD4J architecture adapter?
- Are there any explicitly declared exceptions by module, package root, or capability set?
- Where is the source root?
- Where does a capability's `package-info.java` live?
- How does a capability name map to packages and classes?
- How do `## Boundary` operations map to entrypoints or application operations?
- How do `## Entities` entries map to domain, model, entity, aggregate, DTO, or persistence classes?
- Which implementation areas are intentionally not part of the spec?
- How is structural drift detected in both directions?
- How do tests expose requirement ids?

If the adapter cannot answer confidently, run `setup` or ask one specific question. Do not infer a complex architecture silently.

Treat architecture as a project-level decision, not a per-capability preference. Use one primary adapter for the project or module. If a repository genuinely mixes architectures, `AGENTS.md` must declare the routing rule before SDD4J writes, applies, or verifies affected capabilities. Never mix adapters inside one capability.

## Project Configuration

Prefer a project-local `## SDD4J` section in `AGENTS.md` when the project does not follow an adapter's defaults exactly.

Use this shape and keep it concise:

```md
## SDD4J

Spec source:
- format: `package-info.java`
- source root: `src/main/java`
- requirements style: EARS
- trace ids: `R<n>.<m>`

Spec language:
- default: `en`
- requirements: localized EARS

Architecture layout:
- skill: `sdd4j-package-by-feature`
- scope: primary project architecture
- capability package pattern: `com.acme.<capability>`
- test package mirrors main package: true

Stack:
- skill: `spring-boot-server`
- build tool: Maven
- verification command: `./mvnw test`

Traceability:
- requirement id must resolve to its exact runner-visible `Rn.m` form through a display name, case label, symbol, or annotation consumed by the test/reporting infrastructure
- a normalized Java identifier such as `R1_2` is valid only when the configured infrastructure resolves and displays it as `R1.2`
- JavaDoc and comments alone do not count
```

For sdd4j-package-by-layer projects, include explicit layer package roots and the capability mapping convention. For mixed or transitional repositories, include a short `architecture routing` rule that maps package roots or modules to adapters.

`Spec language` is optional. When absent, write specs and EARS requirements in English. When present, write the capability title, responsibility, boundary descriptions, requirement prose, entity descriptions, out-of-scope items, system docs, and README generated projections in the configured language. Keep structural section names, requirement ids, package names, class names, method names, boundary operation ids, and trace ids stable unless the project explicitly declares otherwise.

Before authoring, extending, applying, or verifying a capability spec, read the project-local `AGENTS.md ## SDD4J` section and resolve `Spec language`. Apply the resolved language to all generated or modified spec prose and localized EARS statements. If `Spec language` is absent, use English.

## System Doc

An optional system doc can live one package above the capability packages, usually at
`src/main/java/<base>/package-info.java`. Use it only for concerns that span capabilities and have no better home. A one-capability system does not need it. Author from `references/system-doc-template.md`.

- **Charter** — one sentence for the whole assembly.
- **Vision** *(optional)* — one aspirational sentence: the outcome the assembly chases. Rationale, not contract — no `Sn`, no test; the single non-verifiable line in the doc, and the deliberate exception to the traceability invariant. May be proposed by `/sdd4j new` distilling a README seed (human accepts/edits).
- **Components** — this system's concrete wiring: which SDD4J capability may call which, which integration events cross boundaries (`/bce` owns the generic layering; this owns the concrete dependencies).
- **System invariants** — cross-cutting EARS `shall` statements (id `Sn`) no single SDD4J capability owns.
- **Ubiquitous language** — shared domain nouns defined once, so each SDD4J capability's `## Entities` stays terse.
- **Decisions** *(optional)* — append-only log of confirmed choices and their rejected alternatives (a stack pick, a carving, an integration style): `Dn — <choice>. _(why: …; rejected: …)_`. Rationale, not contract — like Vision and a statement's `why`: no test, not a trace target. Ids stable; a reversed decision gets a new entry, the old one marked `superseded by Dm` — never edited or deleted. Litmus: testable behaviour → an EARS statement (tested); a standing project rule → README `## Conventions`; a point-in-time choice with rejected alternatives → `Dn`. A decision owned by a single SDD4J capability may live in that SDD4J capability's package doc under the same rules.
- **Stack** — the composed stack skill + package base, so `apply` reads it instead of re-inferring.

Never duplicate a SDD4J capability's one-liner — a hand-typed SDD4J capability index drifts; the gap is read, not stored. For a SDD4J capability map, mark it **generated** and regenerate it from the per-SDD4J capability docs.

## README Projection

An **optional** repo-root `README.md` — a human on-ramp that is a **projection of the specs, not a
source of truth**. Author from `references/readme-template.md`. Two slices, handled oppositely:

- **Generated** (never hand-edited) — the system doc's Charter + Vision, a SDD4J capability map (each SDD4J capability name, its `>` one-liner, a link to its `package-info`), and a **Mermaid diagram of the declared `## Components` wiring**, fenced by `<!-- sdd4j:generated:start -->` / `<!-- sdd4j:generated:end -->`.
- **Hand-maintained** (outside the markers, since no spec covers it — so it can't drift): `## Conventions`, build/run/test delegated to the stack skill, plus free-form meta (license, links, motivation).

- **Doubles as the inception seed.** The hand-written prose outside the markers is what `/sdd4j new` (no argument) reads to bootstrap vision + specs (see `new`); SDD4J reads it, never rewrites it.
- **Components diagram — projection, never inference.** Render only the *declared* wiring in the system doc's `## Components` (allowed calls + integration events) as a Mermaid graph: nodes are SDD4J capabilities, edges the declared directed relationships. **Never infer edges by scanning code** — that is discovery, not projection, and drift-prone. No `## Components` (a one-SDD4J capability system) → nodes only, or omit. Basic Mermaid `flowchart`/`graph` syntax (version-stable, corpus-dense); delegate diagram style to `/mermaid` or `/bce-diagrams`.
- `## Conventions` is the home for **project-specific, non-behavioral standards** (coverage target, "money is always cents", review policy): **declared, not verified** — no `Sn`, no test — and distinct from a `System invariant`, which must be behavioral *and* tested, and from a `Dn` decision, which records a point-in-time choice with its rejected alternatives.
- Optional: a one-SDD4J capability project needs none. No markers → `apply` leaves the README untouched.

## Determinism Boundary

| Concern | Owner | Deterministic? |
| --- | --- | --- |
| Run verification | Stack skill or configured project command | Yes |
| Locate code and map architecture | Architecture adapter plus `AGENTS.md` | Mostly deterministic |
| Decompose natural-language feature into capabilities | SDD4J judgment, user-confirmed | No |
| Record a confirmed choice as a `Dn` decision | this skill offers, **user-confirmed** — never recorded silently | no — semantic |
| Author boundary operations and EARS statements | SDD4J judgment, user-confirmed when ambiguous | No |
| Place packages and classes | Architecture adapter plus stack skill | Yes when configured |
| Structural sync both directions | SDD4J using adapter rules and resolvable test traces | Mostly deterministic |
| Decide if code satisfies a requirement | SDD4J judgment grounded by passing tests | No |
| Regenerate README generated block | SDD4J from package docs | Yes |

- Ask the stack skill "are you green?" — never name a runner or test kind, and never self-certify convergence.

## Invocation modes: setup · new · apply · verify 

Mode ownership:

- `setup` writes project configuration only.
- `new` writes capability and system contracts only.
- `apply` writes implementation, tests, and generated projections.
- `verify` performs read-only conformance analysis.

### setup

Use `setup` to configure SDD4J for an existing Java project before creating specs.

Workflow:

1. Inspect project files enough to identify Java source roots, build tool, likely stack, and likely layout.
2. Detect or ask for the primary architecture layout: `sdd4j-package-by-feature`, `sdd4j-package-by-layer`, `sdd4j-bce`, or a project-specific mapping.
3. Detect or ask for the stack skill: Spring Boot, MicroProfile, Java CLI, or another Java stack.
4. If multiple layouts appear, treat that as exceptional. Ask whether the project is transitional or multi-module, then propose explicit architecture routing by module, package root, or capability set.
5. Propose the `## SDD4J` section for `AGENTS.md`.
6. Ask for confirmation before writing or replacing that section.
7. Write only the project mapping. Do not create capability specs or domain code during setup.

When confidence is low, ask one concrete question at a time. Prefer enumerable choices with room for a custom answer.

### new - declare

Use `new` to declare a capability spec from a precise capability name, a natural-language feature description, or a README seed when no argument is provided. Coining a capability and extending an existing one are both valid `new` work because the novelty is the intended behavior.

Clarify before authoring:

- Loop until every contract ambiguity is resolved. Each answer can expose a new gap.
- Never assume silently. If leaning on a default, name it and ask for confirmation.
- Ask one concrete ambiguity per question. Prefer specific options with room for a custom answer.
- Interrogate each boundary operation for trigger, response, invalid triggers, state constraints, entities, identity, validation, lifecycle scope, and what done means.
- Stop only when another engineer could author the same spec from the user's words. If not, ask one more focused question.

For a capability name:

1. Resolve `Spec language` from `AGENTS.md ## SDD4J`; if absent, use English.
2. Validate the name using the architecture adapter's naming rules.
3. Locate the target package from `AGENTS.md` or the architecture adapter.
4. If `package-info.java` exists, do not overwrite it without explicit user confirmation.
5. Clarify boundary operations, entities, happy paths, edge cases, and out-of-scope behavior until the contract is answerable from user intent.
6. Read `references/capability-spec-template.md` and author the package spec from it in the resolved spec language.
7. Use the architecture adapter only to locate the capability package and validate its naming. Do not create implementation classes, entities, test classes, or stack-specific scaffolding during `new`; leave all code and test convergence to `/sdd4j apply <capability>`.
8. Report the open gap: operation count, requirement count, entity count, and suggested `/sdd4j apply <capability>`.

For a feature description:

1. Scan existing package specs and architecture mapping.
2. Propose one or more capabilities to create or extend. Tag each as `new` or `extend-existing` and include the one-line responsibility.
3. Ask the user to confirm the decomposition before writing.
4. Apply the capability-name workflow to each approved capability. Extend the single existing package spec for existing capabilities; never create a second spec.
5. If decomposition introduces cross-capability wiring, shared language, or system invariants, propose a system doc update and ask before writing it.

For a README seed:

1. Read only hand-written prose outside `sdd4j:generated` markers.
2. Treat it as a feature description and run the feature-description workflow.
3. Propose a `## Vision` line distilled from the seed for user acceptance or editing.
4. Leave the seed prose human-owned. Do not rewrite it except for the generated block when explicitly requested.

### apply - converge

Use `apply` to converge code and tests to an existing capability spec.

Workflow:

1. Locate the capability's `package-info.java`; if missing, stop and suggest `/sdd4j new <capability>`.
2. Resolve `Spec language` from `AGENTS.md ## SDD4J`; if absent, use English.
3. Resolve the applicable architecture and stack from `AGENTS.md`, system package docs, repository conventions, or one focused question. If multiple adapters could apply, stop until the routing rule is explicit. Read the system doc's `## Decisions` if present — never close a gap with an approach a `Dn` rejected.
4. Run the stack verification loop before editing when feasible. Treat a green result as necessary evidence, not proof of convergence.
5. Read the structural gap both ways.
6. Audit semantic conformance for every requirement: compare its trigger, preconditions, observable response, rejection behavior, constraints, and exact contract values with the traced tests and mapped implementation. A resolvable trace id and a green test do not prove that the test asserts the requirement's semantics.
7. If verification is green and no structural or semantic gap or drift exists, stop and report already converged.
8. Close spec-to-code gaps: each missing operation becomes the adapter-defined operation; each untested `Rn.m` gets a traceable test; each declared entity gets the adapter-defined representation when needed.
9. Delegate EARS-to-test mapping to `sdd4j-ears-tests` when available: one parameterized or table-driven test per `### Rn` group and one labeled row or case per statement id `Rn.m`, adapted to the stack's test framework.
10. Write the correct implementation to pass the new tests.
11. Surface code-to-spec drift instead of silently editing the spec to match code. The user decides whether to declare it or delete the orphan.
12. Re-run verification and repeat for at most three passes. Then surface remaining failures, gaps, or drift.

Stop when verification is green and no structural or semantic gap or drift remains.

### verify

Use `verify` to check conformance without intentionally implementing missing behavior.

Report:

- Spec location.
- Resolved spec language.
- Resolved architecture adapter, routing rule, and stack.
- Requirement ids with and without tests.
- Semantic mismatches between requirements, their traced tests, and mapped implementation.
- Boundary operations with and without mapped code.
- Entities/models with and without mapped code.
- Inverse drift found in code or tests.
- Verification command and result.

#### Semantic conformance audit procedure

A green verification and complete traceability are necessary but not sufficient. During `verify`, audit every `Rn.m` for semantic drift:

1. Extract the concrete contract value the statement fixes: a number, a string, an HTTP status, a timeout, a cache key prefix, a header name, an endpoint, a default fallback, or any literal value.
2. Locate the mapped implementation that realizes the statement and the traced test that exercises it.
3. Verify that the implementation and the test honor the same contract value, not only the same code path or exception type.
4. Pay explicit attention to hardcoded defaults, constants, and fallback values; compare them against the spec.
5. If a default, constant, or fallback value differs from the spec, report it as semantic drift and do not treat the build as proof of conformance.

## Spec Format

Author capability specs from `references/capability-spec-template.md`.

Sections must appear in this order: capability title and responsibility, `## Boundary`, `## Requirements`, optional `## Entities`, optional `## Decisions`, and `## Out of scope`. Keep `## Out of scope` even when empty.

Boundary operations must be verb-noun and transport-neutral, such as `place-order`, not `POST /orders` or `click-submit`.

## EARS Rules

- Use mandatory requirement wording for every requirement. In English specs, use `shall`. In localized specs, use a consistent configured-language equivalent.
- Use stable ids and never reuse retired ids.
- Keep statements behavioral and verifiable.
- Keep framework details, URLs, HTTP verbs, database tables, and implementation choices out of requirements unless the stack contract itself is the capability.
- Use EARS as semantic patterns, not English-only keywords. In English specs, use `When`, `If`, `While`, `Where`, or ubiquitous EARS statements. In localized specs, express the same event-driven, unwanted-behavior, state-driven, optional-feature, ubiquitous, or complex pattern in the configured language.
- Every boundary op traces to a group `Rn`; 
- Every statement `Rn.m` traces to at least one executable test or case whose runner-visible identity resolves to its exact id.
- Traceability must be complete in both directions: every statement has a test trace, and every literal or symbolic test trace resolves to an existing statement in the corresponding capability spec.
- One statement may have multiple tests when they cover distinct scenarios or verification levels and the duplication is intentional.
- A new `Rn.m` with no test is a gap, and a method, trace id, or `entity` type with no spec counterpart is inverse drift (surfaced, never absorbed into the spec).
- The trace may be a literal display name or case label, or a symbol or annotation consumed by the test/reporting infrastructure whose display form is the exact id. A normalized Java identifier such as `R1_2` is valid only when it resolves to `R1.2`; JavaDoc and comments alone do not count.
- Optional behavior is expressed with the optional-feature EARS pattern. In English specs, use `Where <feature is included>, the capability shall <response>.` Do not use `should` or `may` for contract behavior; use equivalent non-optional wording in localized specs.

EARS templates:

| Pattern | Template |
| --- | --- |
| Ubiquitous | `The capability shall <response>.` |
| State-driven | `While <precondition>, the capability shall <response>.` |
| Event-driven | `When <trigger>, the capability shall <response>.` |
| Optional-feature | `Where <feature is included>, the capability shall <response>.` |
| Unwanted behavior | `If <trigger>, then the capability shall <response>.` |
| Complex | `While <precondition>, when <trigger>, the capability shall <response>.` |

## Optional Why

Any boundary operation or `Rn.m` statement may carry a trailing `_(why: ...)_` note.

Rules:

- The `why` explains origin or intent, not current implementation.
- The `why` is rationale, not contract, and is not tested.
- The id must remain first so spec parsing and trace resolution remain deterministic.
- A `why` retires with its operation or statement and must not become an orphan.

## Optional Decisions

- A confirmed capability-local SDD4J decision may be logged before `## Out of scope`, under the same rules as the system doc's `## Decisions`: stable `Dn`, rejected alternatives in a trailing `_(why: …; rejected: …)_`, append-only (supersede, never edit), rationale not contract — no test, not a trace target.

## Drift Policy

The spec is source of truth for intended behavior. SDD4J can implement missing declared behavior, but it must not auto-expand the spec to bless existing code. When code exists without a spec counterpart, report it as drift and ask whether to declare it or remove it.

Spec-to-code gaps are work SDD4J may close:

- Missing mapped operation for a declared boundary operation.
- Missing mapped representation for a declared entity when the adapter says it should exist.
- Missing test trace for a requirement id.
- Failing behavior for a declared requirement.

Code-to-spec drift is user-decision territory:

- Mapped entrypoint or application operation absent from `## Boundary`.
- Contract-relevant domain/model/entity artifact absent from `## Entities`.
- Test tracing an id no statement carries.
- Behavior that expands the capability beyond the spec.

Never edit the spec merely to absorb drift. Ask whether to declare the behavior or remove the orphan.
