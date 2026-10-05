---
name: diagrams
description: Create high-level overview diagrams showing interactions between components, subsystems, layers, feature packages, services, or systems, rendered via the mermaid or drawio skill. Use when asked to create architecture overviews, component interaction diagrams, subsystem diagrams, layer diagrams, feature maps, service interaction diagrams, system landscapes, or integration maps. Triggers on "diagram", "overview diagram", "BC interaction", "component interaction", "architecture overview", "layer diagram", "feature map", "system diagram", "integration diagram", or requests to visualize how components, layers, or services communicate. Not for detailed class diagrams or sequence diagrams.
argument-hint: "[services or components to visualize]"
---

# Architecture Overview Diagrams

Create diagrams that show interactions and dependencies between components, groupings, and external services at a high level of abstraction. This skill routes the request: it detects the architecture, scope, and format, then delegates rendering to the `mermaid` or `drawio` skill, which owns the shape mapping and palette.

## Workflow

1. **Detect the architecture** — resolve which shape mapping applies:
   - the user names it explicitly ("BCE diagram", "layered view", "feature map")
   - `AGENTS.md` `## SDD4J` declares an `sdd4j-*` architecture adapter
   - codebase scan (see Information Gathering)
   - still ambiguous → include it as an extra question in step 3
2. **Gather information** — identify components, groupings, external services, and interactions to visualize, per the detected architecture.
3. **Choose scope and format** — ask the user using a single AskUserQuestion call with two questions:
   - **Scope**: "System view" (each component is a single opaque node — default) or "Inside a component" (expand one component's internal structure)
   - **Format**: "Mermaid (Recommended)" (text-based, version-control friendly, renders natively in GitHub/GitLab — default for README.md in GitHub projects) or "draw.io" (visual editor, richer styling, exportable to PNG/SVG)
4. **Invoke the chosen skill** — use the Skill tool to invoke `mermaid` or `drawio` with the detected architecture, scope, identified elements, and interactions. When the diagram is destined for a README.md in a GitHub project, default to Mermaid without asking.

## Scope Guidelines

### System view (default)

- Show components, groupings, and external services — not classes, methods, or fields
- Represent each component as a single node labeled by its responsibility: `Orders`, `Payments`, `Users`
- Group components into containers when they share a domain, module, or layer
- Focus on interactions: which component calls which, what data flows where
- Label arrows with the interaction type when not obvious: `REST`, `async`, `JPA`
- Show external dependencies (third-party APIs, databases, message brokers) as separate nodes

### Inside a component

- Expand one component into its internal structure, per the architecture:
  - BCE → boundary, control, and entity layers
  - Package By Layer → the capability's classes inside layer containers
  - Package By Feature → the feature package's internals, using layer element styles
  - Hexagonal → ports, use case, and domain inside the hexagon; driving adapters on the left, driven adapters and externals on the right
- Show internal flow (e.g. boundary → control → entity, entrypoint → application → domain)
- Cross-component edges connect at the boundary or entrypoint that owns the interaction
- External dependencies connect to the internal element that uses them
- Expanding several components at once produces noisy diagrams — expand a single component unless the user asks otherwise

## Architecture Profiles

### BCE

- Vocabulary: business component (BC), subsystem, boundary, control, entity, external
- Detection: packages shaped `[org].[project].[bc].[boundary|control|entity]`; `sdd4j-bce` declared in `AGENTS.md`
- Interactions: which boundary calls which boundary; externals connect to the layer that uses them
- Based on BCE architecture (see [bce.design](https://bce.design))

### Package By Layer

- Vocabulary: capability, layer container (entrypoints, application, domain, infrastructure), entrypoint/application/domain/infrastructure class, external
- Detection: global `controller`, `service`, `repository`, `model` package roots; `sdd4j-package-by-layer` declared in `AGENTS.md`
- Interactions: entrypoint → application → domain/infrastructure flow

### Package By Feature

- Vocabulary: feature package (capability), module, shared package, external
- Detection: self-contained capability packages under a base package; `sdd4j-package-by-feature` declared in `AGENTS.md`
- Interactions: declared calls and events between feature packages; shared packages are exceptional nodes

### Hexagonal (Ports & Adapters)

- Vocabulary: capability, driving adapter (inbound), inbound port, use case, outbound port, driven adapter (outbound), domain, external
- Detection: `port`/`adapter` package roots or `*.port.*`/`*.adapter.*` segments (canonical: `<capability>.application.port.in|out`, `<capability>.adapter.in|out`); `sdd4j-hexagonal` declared in `AGENTS.md`
- Ambiguity: `application`/`domain`/`infrastructure` layer names overlap with Package By Layer — the distinguishing signal is `port`/`adapter` packages; layer roots alone do not imply hexagonal; when both readings fit, ask rather than guess
- Interactions: driving adapter → inbound port → use case → domain; use case → outbound port → driven adapter → external. Draw driving side left, driven side right.

### Generic

- No declared or detectable architecture → component, grouping, and external vocabulary only (the renderers' Default mapping)

## Information Gathering

When the user does not specify components, analyze the codebase per the detected architecture:

- **BCE**: scan for BC packages (children of the top-level application package containing `boundary`/`control`/`entity` sub-packages); find cross-BC interactions via injected references, REST client calls, or messaging
- **Package By Layer**: scan layer roots and group classes by the project's capability mapping (class-name prefix, declared `AGENTS.md` mapping)
- **Package By Feature**: scan feature packages under the base package; find edges via imports or calls crossing package boundaries
- **Hexagonal**: scan capability packages for `adapter.in.*`/`adapter.out.*` and `application.port.*`; edges are driving-adapter calls into inbound ports and outbound-port usages resolved to driven adapters
- **All**: determine external dependencies (third-party APIs, cloud services, databases, message brokers)
