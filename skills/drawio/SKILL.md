---
name: drawio
description: Create draw.io overview diagrams with consistent visual style and shape mappings for BCE, package-by-layer, and package-by-feature Java architectures. Use when asked to create, generate, or edit draw.io diagrams, architecture diagrams, component diagrams, or visual overviews of systems and modules. Triggers on "draw.io", "drawio", "create a diagram", "architecture diagram", "component diagram", or requests to visualize high-level system structure. Not for sequence diagrams or class diagrams.
---

# Draw.io Overview Diagrams

Generate valid draw.io XML and save as `<name>.drawio` using the Write tool.

## Mapping Selection

- pick the shape mapping matching the architecture the user named or the composed skill implies:
  - BCE business components, or `sdd4j-bce` in context → BCE mapping
  - technical layer packages, or `sdd4j-package-by-layer` in context → Package By Layer mapping
  - feature/capability packages, or `sdd4j-package-by-feature` in context → Package By Feature mapping
- when invoked via `diagrams`, use the architecture it detected — the router is architecture-neutral
- no clear architecture → use the Default mapping
- never mix mappings inside one diagram

## Lines

- rounded routing: `edgeStyle=orthogonalEdgeStyle;rounded=1;`
- black stroke: `strokeColor=#000000;`
- arrow at target: `endArrow=block;endFill=1;`
- async/optional dependencies: add `dashed=1;`
- label without shadow: `shadow=0;` on the label cell
- when an edge crosses container boundaries (source or target is nested in a container), pin explicit `exitX`/`exitY` on the source and `entryX`/`entryY` on the target so the orthogonal router does not take ugly paths; when multiple edges share a target, give each a distinct entry point to prevent overlap

## Style

- all nodes: `rounded=1;whiteSpace=wrap;html=1;shadow=1;`
- corner radius: `arcSize=10;`
- font: `fontColor=#000000;fontSize=13;fontStyle=0;`
- grouping: container with title bar (`container=1;collapsible=0;verticalAlign=top;`), fill `#f5f5f5`, border `#666666`

## Palette

Colors carry the same meaning across all mappings — and match the `mermaid` skill, so mixed-format diagrams stay visually consistent:

| Color | Fill | Border | Meaning |
|---|---|---|---|
| Blue | `#dae8fc` | `#6c8ebf` | capability / business component |
| Green | `#d5e8d4` | `#82b366` | entrypoint / boundary |
| Purple | `#e1d5e7` | `#9673a6` | application / control |
| Yellow | `#fff2cc` | `#d6b656` | domain / entity |
| Gray | `#f5f5f5` | `#666666` | infrastructure / shared / grouping container |
| Dashed yellow | `#fff2cc` | `#d6b656` dashed | external service |

## Default Mapping

Use when no architecture mapping applies.

| Element | Shape | Fill | Border |
|---|---|---|---|
| Component | rounded rectangle | `#dae8fc` | `#6c8ebf` |
| Grouping | container with title bar | `#f5f5f5` | `#666666` |
| External service | rounded rectangle, `dashed=1` | `#fff2cc` | `#d6b656` |

## BCE Mapping

| Element | Shape | Fill | Border |
|---|---|---|---|
| Business Component (BC) | rounded rectangle | `#dae8fc` | `#6c8ebf` |
| Subsystem container | container with title bar | `#f5f5f5` | `#666666` |
| External service | rounded rectangle, `dashed=1` | `#fff2cc` | `#d6b656` |
| Boundary layer | rounded rectangle | `#d5e8d4` | `#82b366` |
| Control layer | rounded rectangle | `#e1d5e7` | `#9673a6` |
| Entity layer | rounded rectangle | `#fff2cc` | `#d6b656` |

### BCE Robustness Icons (built-in)

Drawio ships with the Jacobson BCE icons. Use these — never reconstruct them from primitives (lines + ellipses + triangles).

- Boundary Object: `shape=umlBoundary;whiteSpace=wrap;html=1;`
- Control Object: `ellipse;shape=umlControl;whiteSpace=wrap;html=1;`
- Entity Object: `ellipse;shape=umlEntity;whiteSpace=wrap;html=1;`

Color them by adding `fillColor=#ffffff;strokeColor=<layer-border>;`. Typical icon size: `width=30;height=32`. These icons are BCE-specific — do not use them in other mappings.

## Package By Layer Mapping

Layers are containers; nodes are the capability's classes inside each layer.

| Element | Shape | Fill | Border |
|---|---|---|---|
| Capability (spec contract) | rounded rectangle | `#dae8fc` | `#6c8ebf` |
| Layer grouping | container with title bar | `#f5f5f5` | `#666666` |
| Entrypoint (controller, resource, handler, listener, CLI) | rounded rectangle | `#d5e8d4` | `#82b366` |
| Application operation (service, use case) | rounded rectangle | `#e1d5e7` | `#9673a6` |
| Domain/model type (entity, model, DTO) | rounded rectangle | `#fff2cc` | `#d6b656` |
| Infrastructure (repository, client, adapter) | rounded rectangle | `#f5f5f5` | `#666666` |
| External service | rounded rectangle, `dashed=1` | `#fff2cc` | `#d6b656` |

## Package By Feature Mapping

Nodes are feature packages; containers group by module or base package. Drawing inside a feature package reuses the element styles from the Package By Layer mapping.

| Element | Shape | Fill | Border |
|---|---|---|---|
| Feature package (capability) | rounded rectangle | `#dae8fc` | `#6c8ebf` |
| Module grouping | container with title bar | `#f5f5f5` | `#666666` |
| Shared package (declared exception) | rounded rectangle, `dashed=1` | `#f5f5f5` | `#666666` |
| External service | rounded rectangle, `dashed=1` | `#fff2cc` | `#d6b656` |

## Content

- visualize only high-level concepts and modules
- show dependencies between components as directed arrows
- omit implementation details, classes, and methods
- group related components visually
- limit diagram to essential relationships

## Layout

- arrange components in logical flow (top-to-bottom or left-to-right)
- consistent spacing: 40px between nodes, 20px padding inside containers
- avoid crossing lines where possible
- use container shapes (`container=1;`) for subsystem, layer, or module grouping
- if a node blends into its container's fill (e.g. gray infrastructure inside a gray layer container), set `fillColor=none` on the container
- default node size: `width=160;height=60;`

## Labels

- use short, descriptive names
- avoid technical jargon unless domain-specific
- label arrows only when relationship type is ambiguous
- black font color for all labels
- do not use italic font
- connection labels must be without shadow
- connection labels must use `fontSize=16;`

## XML Structure

Minimal skeleton — every `.drawio` file must follow this hierarchy:

```xml
<mxfile>
  <diagram name="Overview" id="diag1">
    <mxGraphModel dx="1024" dy="768" grid="1" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="1" pageScale="1" pageWidth="1169" pageHeight="827" math="0" shadow="0">
      <root>
        <mxCell id="0"/>
        <mxCell id="1" parent="0"/>
        <!-- nodes and edges here, parent="1" -->
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

## Example — Minimal 2-Node BCE Diagram

```xml
<mxfile>
  <diagram name="Overview" id="d1">
    <mxGraphModel dx="1024" dy="768" grid="1" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="1" pageScale="1" pageWidth="1169" pageHeight="827" math="0" shadow="0">
      <root>
        <mxCell id="0"/>
        <mxCell id="1" parent="0"/>
        <mxCell id="2" value="Orders" style="rounded=1;whiteSpace=wrap;html=1;shadow=1;arcSize=10;fillColor=#dae8fc;strokeColor=#6c8ebf;fontColor=#000000;fontSize=13;fontStyle=0;" vertex="1" parent="1">
          <mxGeometry x="100" y="200" width="160" height="60" as="geometry"/>
        </mxCell>
        <mxCell id="3" value="Payments" style="rounded=1;whiteSpace=wrap;html=1;shadow=1;arcSize=10;fillColor=#dae8fc;strokeColor=#6c8ebf;fontColor=#000000;fontSize=13;fontStyle=0;" vertex="1" parent="1">
          <mxGeometry x="380" y="200" width="160" height="60" as="geometry"/>
        </mxCell>
        <mxCell id="4" value="charges" style="edgeStyle=orthogonalEdgeStyle;rounded=1;orthogonalLoop=1;strokeColor=#000000;fontColor=#000000;fontSize=16;fontStyle=0;endArrow=block;endFill=1;shadow=0;" edge="1" source="2" target="3" parent="1">
          <mxGeometry relative="1" as="geometry"/>
        </mxCell>
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

## File Output

- save as `<name>.drawio` using the Write tool
- see `references/example-bce.drawio`, `references/example-layered.drawio`, and `references/example-feature.drawio` for complete per-mapping examples
