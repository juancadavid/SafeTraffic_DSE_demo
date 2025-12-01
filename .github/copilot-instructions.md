# SafeTraffic_DSE_demo • AI Working Notes

## Context snapshot
- This repo is a **Modelio 5.4 local project** (`project.conf`) with the `ModelerModule 9.4.00` and `ToscaDesigner 0.5.01` modules active. Always open the workspace inside Modelio when making structural changes; the `.exml` files are generated and reference-linked by GUID.
- Project data lives under `data/fragments/SafeTraffic_DSE_demo`. Subfolders: `admin/` (metamodel + access control), `model/` (one file per UML element), `blobs/` (binary diagram payloads), `.index/` (lookup tables). Treat everything here as Modelio-managed artifacts.
- `data/localmodel/*` mirrors the current local database state; do not edit it manually and avoid committing ad-hoc experiments from that folder.

## Structural map
- `data/fragments/SafeTraffic_DSE_demo/model/Standard.Project/132ed670-…` defines the root project and points to the top-level package `example.eu.myrtus` (`Standard.Package/8b2e3731-…`).
- The TOSCA model lives inside that package:
  - `nodetypes` (`Standard.Package/006805f6-…`) stores every **node type** as a `Standard.Class` stereotyped `TNodeType`. Example: `myrtus.dpe.compute` in `Standard.Class/95b62343-…`.
  - `TrafficApplication` (`Standard.Class/50e090a8-…`, stereotype `TTopologyTemplate`) contains **groups** (`compute_nodes`, `software_component_nodes`, etc., stereotype `TGroup`) and concrete **node templates** like `FogComputeNode1` (`Standard.Class/8ef5707f-…`, stereotype `TNodeTemplate`).
  - Property metadata for node/template elements sits in child `Infrastructure.TypedPropertyTable` objects whose `Content` field is a Java `.properties` blob (e.g., `minInstances`, `targetNamespace`).
- Diagrams referenced from the package live in `Standard.StaticDiagram/*` files plus matching binary blobs under `data/fragments/.../blobs/`. Edit diagrams only through Modelio so GUID links stay valid.

## Modeling conventions
- **Never hand-edit GUIDs or the `mc` (metaclass) attributes** inside `.exml`. Create, rename, or remove elements exclusively via Modelio. Manual tweaks are only safe inside `<ATT name="Content">` blocks of property tables when you know the schema.
- Every TOSCA node type class must:
  1. Carry the `TNodeType` stereotype.
  2. Store its description/namespace flags inside a `TypedPropertyTable` referencing `TEntityTypePropertyTable` (see `Standard.Class/95b62343-…`).
  3. Declare each TOSCA property as a `Standard.Attribute` stereotyped `PropertyDefinitionType`, pointing to primitive data types in `Standard.DataType`.
- Node templates must:
  - Be contained by a group within the topology template and use the `TNodeTemplate` stereotype.
  - Include `OwnedAttribute` entries stereotyped `TPropertyDef` for concrete values. These attributes often depend on the node type attributes via `Infrastructure.Dependency` objects named `type`; Modelio builds these references—do not edit the `DependsOn` UID manually.
  - Reference their type through a `DependsOnDependency` named `nodeType`, pointing to the correct node type class.
- Policies, requirements, and capabilities are modeled as nested classes under each node (`OwnedElement` collections). Follow the same stereotype patterns (e.g., `TRequirementType`, `TCapabilityType`) already used in neighboring files to stay consistent.

## Working in Modelio
- Launch Modelio 5.4, choose **Open Workspace**, and point to `c:/Users/jcadavid/modelio/workspace`. Open the `SafeTraffic_DSE_demo` project and ensure the **ToscaDesigner** module is active (check the Modules view). 
- Use ToscaDesigner palettes to create or edit TOSCA entities; the module automatically emits the correct stereotypes, property tables, and diagram glyphs.
- When you finish edits, save the project, then close Modelio to flush `.exml`, `.index`, and `localmodel` updates before running git status. Committing while Modelio is still open can leave dangling lock files.

## Validation & exports
- To validate the model, run the **Audit** or ToscaDesigner validation inside Modelio (Audit at startup is disabled in `project.conf`, so trigger it manually via `Analyze ▶ Audit`). Resolve warnings inside the tool before committing.
- ToscaDesigner can export service templates/XMI directly; use this instead of editing serialized files. Keep any exported deliverables outside the repository unless explicitly requested.

## Practical tips
- The GUID-based file names make diffing noisy; narrow diffs by focusing on `<ATT name="Name">`, `<ATT name="Value">`, and property-table `Content` entries—these contain the semantic changes reviewers care about.
- If you must compare versions, search by human-readable names in the file (e.g., `FogComputeNode1`) rather than by file name.
- Large metamodel descriptors under `data/fragments/.../admin/` are regenerated when modules update; avoid touching them unless a Modelio upgrade requires it.
- Prefer small, isolated edits (add/update one node type or template at a time). This keeps GUID churn manageable and helps auditors trace dependencies.
