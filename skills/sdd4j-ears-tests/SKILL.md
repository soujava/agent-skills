---
name: sdd4j-ears-tests
description: >
  Generate parameterized (table-driven) tests from EARS requirement statements — the deterministic transform that turns an SDD4J capability spec's `## Requirements` into one parameterized test per requirement group and one labeled row per statement id. Stack-neutral; owns only the EARS→table mapping and the spec↔test trace, and delegates the test syntax to the composed stack skill (JUnit 5). Use whenever turning EARS requirements, acceptance criteria, or an SDD4J/`/sdd4j` spec into tests; whenever you see "When/While/If…then, the … shall …" statements that need covering; or when asked for table-driven, data-driven, or parameterized tests from a requirement group. Trigger on "SDD4J tests", "EARS", "parameterized tests", "table-driven tests", "data-driven tests", "tests from requirements", "tests from the spec", "cover requirement Rn", "generate tests for this capability", "acceptance criteria to tests".
metadata:
  type: test-generation
---

# Skill: SDD4J EARS Tests

## Objective

Turn SDD4J EARS requirement statements into traceable parameterized Java tests. The leverage is structural: every EARS pattern is a (condition → response) tuple, and an SDD4J requirement group already collects statements that share one boundary operation — same arrange/act skeleton, only the data differs. That is the textbook precondition for parameterization, so the mapping is mechanical. 
This skill owns only the EARS-to-test transform and the requirement trace. The architecture adapter owns where capability code and tests belong. The stack skill owns concrete test syntax, runner, fixtures, and verification.

## Input Contract

Consume a SDD4J capability spec, usually from `package-info.java`, with this shape:

```md
## Boundary

- `place-order` - Places an order for a valid cart.

## Requirements

### R1 Place order

- R1.1 - When a cart with at least one item is submitted, the capability shall create and confirm an order.
  
- R1.2 - If the cart is empty, then the capability shall reject the request.

```

Do not edit the spec and do not coin requirement ids. Missing, duplicate, or malformed ids are spec issues to report back to SDD4J.

## Core Mapping

- One requirement **group `Rn` maps to one parameterized, table-driven, or grouped test skeleton when the group has multiple statements**.
- One **statement `Rn.m` → one row** in the parameter source.
- The **statement id `Rn.m` resolves to the row's runner-visible display name**, so the spec↔test binding stays per-statement.
- The trace may be a literal id or a symbol such as `R1_2` whose display form is the exact `R1.2` id.
- An id that appears only in JavaDoc or a comment is not a trace.
- The EARS pattern fixes the row's **role and shape** — you do not invent the columns, you read them off the pattern.

The shared test body should exercise the boundary operation or application operation associated with the requirement group, as defined by SDD4J and the architecture adapter.

## EARS Pattern To Case Shape

Each EARS statement decomposes into arrange, act, assert, and role fields. Do not invent concrete values from prose; use clear TODO/failing stubs when values or assertions are not derivable from existing code or user input.

| Pattern | Statement shape | Arrange | Act | Assert | Role |
| --- | --- | --- | --- | --- | --- |
| Event-driven | `When <trigger>, the capability shall <response>` | default state | trigger | response | happy |
| Unwanted behavior | `If <trigger>, then the capability shall <response>` | default state | invalid trigger | rejection/error | unhappy |
| State-driven | `While <precondition>, the capability shall <response>` | precondition | implicit | response | stateful |
| Complex | `While <precondition>, when <trigger>, the capability shall <response>` | precondition | trigger | response | full tuple |
| Optional-feature | `Where <feature is included>, the capability shall <response>` | feature enabled | trigger or implicit | response | gated |
| Ubiquitous | `The capability shall <response>` | default or global state | implicit | invariant | invariant |

Mixing happy and unhappy rows in one group is normal when they share the same operation skeleton.

## Deterministic vs Authored

Generate without judgment:

- Group-to-test skeleton.
- Statement-to-row mapping.
- Requirement id labels.
- EARS role classification.
- Bidirectional trace coverage checks between spec ids and test ids.
- The shared operation target when SDD4J and the architecture adapter already identify it.

Leave authored or explicit TODOs for:

- Concrete fixture values.
- Domain object construction when not already obvious.
- Exact assertion expression for prose responses.
- Error type, status code, or message when the spec does not define it.

Prefer a failing TODO, disabled placeholder with a visible reason, or stack-idiomatic red test over a fabricated green assertion.

## Composition

- SDD4J owns the spec, stable ids, gap workflow, and drift policy.
- The architecture adapter owns whether the operation is a controller method, application service method, BCE boundary method, listener, command, or another entrypoint.
- The stack skill owns test syntax and the verification command.
- This skill consumes `## Requirements` and emits or updates tests with exact runner-visible ids, represented by literals or resolvable symbols.

During `/sdd4j apply`, use this skill when an `Rn.m` statement lacks a test trace or when a requirement group should be represented as table-driven tests.

## Realization Rules

Read `references/realizations.md` when concrete examples are useful. Use the project's existing test idiom; never introduce a second framework or style just because an example uses it. 

The parameter-source mechanism and display-name form belong to the composed stack skill — read `references/realizations.md` for the per-stack shape (JUnit 5
`@ParameterizedTest` + `@MethodSource`). Java stacks materialize the `Rn.m` ids as a generated per-BC `{BC name in CamelCase}Requirement` annotation with a nested `Rn` enum — the label contract is unchanged because the enum's display form is the literal id; web stacks and the black-box `-st` module keep literal strings. Resolve the mechanism in this order: the `trace convention` field in `AGENTS.md ## SDD4J` when declared; otherwise the idiom the project's tests already use; if neither exists, propose one and ask. Never introduce a second test idiom.

## Trace Coverage Check

Before and after editing tests, check:

- Every `Rn.m` in the spec resolves to at least one runner-visible test trace.
- Every literal or symbolic test trace resolves to an existing statement in the corresponding capability spec.
- For symbolic traces such as `R1_2`, resolve the symbol's configured display form before comparing it with the spec id.
- One statement can have multiple tests only when each test covers a distinct level or scenario and the duplication is intentional.
- A test must not claim an id from another capability unless the project explicitly has cross-capability system tests.

Report orphan test ids as drift. Do not edit the spec to absorb orphan ids.
