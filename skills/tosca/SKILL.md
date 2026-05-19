---
name: tosca
description: >
  Expert skill for writing, editing, and validating TOSCA 2.0 files — the OASIS standard for cloud topology
  and orchestration. ALWAYS invoke this skill when the user mentions TOSCA in any form: service templates,
  node types, capability types, relationship types, profiles, topology modeling, substitution mappings,
  or the oasis-open TOSCA spec. Also invoke it when the user asks to model an application or infrastructure
  as a TOSCA file, fix TOSCA syntax errors (like topology_template vs service_template, occurrences vs
  count_range), design a TOSCA profile or type library, look up community profiles in the
  oasis-open/tosca-community-contributions GitHub repo, or understand how to import
  org.oasis-open.simple:2.0 or any other TOSCA profile. This skill is exclusively for TOSCA 2.0
  (tosca_definitions_version: tosca_2_0) and never produces TOSCA 1.x syntax.
---

# TOSCA 2.0 Skill

You are an expert in the OASIS TOSCA (Topology and Orchestration Specification for Cloud Applications) Version 2.0 standard.
Your job is to produce correct, well-structured TOSCA 2.0 files.

> ⚠️ This skill is strictly for **TOSCA 2.0**. Never use TOSCA 1.x syntax (e.g., `tosca_simple_yaml_1_0`, `tosca_simple_yaml_1_3`, or `tosca.nodes.*` / `tosca.capabilities.*` namespaced types). When in doubt, refuse to downgrade.

## Key TOSCA 2.0 concepts

See `references/tosca2-types.md` for the full catalog of normative types from the `org.oasis-open.simple:2.0` profile.
See `references/tosca2-examples.md` for complete working examples.
See `references/tosca2-community-profiles.md` for the catalog of community profiles in the OASIS GitHub repo.

---

## TOSCA 2.0 File Structure

Every TOSCA 2.0 file starts with `tosca_definitions_version: tosca_2_0` and may contain:

```yaml
tosca_definitions_version: tosca_2_0

# Optional: declare this file as a reusable profile
profile: <profile-name>:<version>  # e.g. org.mycompany.myprofile:1.0

metadata:
  template_name: <name>
  template_author: <author>
  template_version: <semver>

description: >-
  Human-readable description of this file.

imports:
  - profile: org.oasis-open.simple:2.0          # import normative types
  - url: ./other-types.yaml                       # import a local file
    namespace: ns                                  # optional namespace prefix

# One or more type definition sections (all optional):
data_types: ...
artifact_types: ...
capability_types: ...
interface_types: ...
relationship_types: ...
node_types: ...
group_types: ...
policy_types: ...

# The service template (replaces topology_template from TOSCA 1.x):
service_template:
  description: ...
  inputs: ...
  node_templates: ...
  relationship_templates: ...
  groups: ...
  policies: ...
  outputs: ...
  substitution_mappings: ...
```

### Critical differences from TOSCA 1.x

| Feature | TOSCA 1.x | TOSCA 2.0 |
|---|---|---|
| Version keyword | `tosca_simple_yaml_1_3` | `tosca_2_0` |
| Topology section | `topology_template` | `service_template` |
| Normative type names | `tosca.nodes.Compute` | `Compute` (when profile imported) |
| Requirement occurrences | `occurrences: [0, 1]` | `count_range: [0, 1]` |
| Profile import | not supported | `profile: org.oasis-open.simple:2.0` |
| Namespacing | not supported | `namespace: myns` in imports |

---

## Type Definitions

### Node Type

```yaml
node_types:
  MyApp:
    derived_from: SoftwareComponent    # or Root, Compute, etc.
    description: My application node type.
    properties:
      port:
        type: integer
        description: The listening port.
        default: 8080
        required: false
        constraints:
          - in_range: [1024, 65535]
    attributes:
      url:
        type: string
    capabilities:
      endpoint:
        type: Endpoint
    requirements:
      - host:
          capability: Compute
          node: Compute
          relationship: HostedOn
          count_range: [1, 1]
      - dependency:
          capability: Node
          count_range: [0, UNBOUNDED]
    interfaces:
      Standard:
        type: Lifecycle.Standard
        operations:
          create:
            implementation: scripts/create.sh
          start:
            inputs:
              PORT: { get_property: [SELF, port] }
            implementation: scripts/start.sh
```

### Capability Type

```yaml
capability_types:
  MyEndpoint:
    derived_from: Endpoint
    description: Custom endpoint capability.
    properties:
      api_version:
        type: string
        default: "v1"
```

### Relationship Type

```yaml
relationship_types:
  MyConnectsTo:
    derived_from: ConnectsTo
    description: Custom connection relationship.
    valid_target_types: [Endpoint]
    properties:
      timeout:
        type: integer
        default: 30
    interfaces:
      Configure:
        type: Relationship.Configure
        operations:
          pre_configure_source:
            implementation: scripts/configure.sh
```

### Data Type

```yaml
data_types:
  MyCredential:
    derived_from: map
    description: Custom credential data type.
    properties:
      user:
        type: string
      token:
        type: string
```

---

## Service Template

### Node Template

```yaml
service_template:
  inputs:
    app_port:
      type: integer
      default: 8080

  node_templates:
    my_app:
      type: MyApp
      description: The main application instance.
      metadata:
        tag: production
      directives: [substitute]       # optional: substitute, select, create
      properties:
        port: { get_input: app_port }
      attributes:
        url: { get_attribute: [SELF, endpoint, ip_address] }
      capabilities:
        endpoint:
          properties:
            port: 8080
      requirements:
        - host: my_server
      interfaces:
        Standard:
          operations:
            create:
              implementation: scripts/create.sh
      artifacts:
        app_war:
          type: Deployment.Web
          file: app.war

    my_server:
      type: Compute
      capabilities:
        host:
          properties:
            num_cpus: 2
            mem_size: 4 GB
        os:
          properties:
            type: linux
            distribution: ubuntu
```

### Outputs

```yaml
  outputs:
    app_url:
      description: The application URL.
      value: { get_attribute: [my_app, endpoint, ip_address] }
```

### Groups and Policies

```yaml
  groups:
    app_group:
      type: Root
      members: [my_app, my_server]

  policies:
    scale_policy:
      type: Scaling
      targets: [app_group]
      properties:
        min_instances: 1
        max_instances: 5
```

### Substitution Mappings

Used when a service template acts as a node implementation:

```yaml
  substitution_mappings:
    node_type: MyApp
    capabilities:
      endpoint: [my_app, endpoint]
    requirements:
      host: [my_app, host]
```

---

## Intrinsic Functions

```yaml
# Get a template input value
value: { get_input: input_name }

# Get a property from a node
value: { get_property: [node_name, property_name] }
value: { get_property: [SELF, property_name] }
value: { get_property: [TARGET, property_name] }
value: { get_property: [SOURCE, property_name] }

# Get a runtime attribute
value: { get_attribute: [node_name, attribute_name] }
value: { get_attribute: [SELF, endpoint, ip_address] }

# Concatenate strings
value: { concat: ["http://", { get_attribute: [server, public_address] }, ":8080"] }

# Join a list
value: { join: [["item1", "item2"], ","] }

# Token extraction
value: { token: [{ get_attribute: [SELF, ip_address] }, ".", 0] }

# Select from condition
value: { if: [condition_expression, true_value, false_value] }
```

---

## Imports and Profiles

### Importing the OASIS normative profile

```yaml
imports:
  - profile: org.oasis-open.simple:2.0
```

This gives you access to: `Root`, `Compute`, `SoftwareComponent`, `WebServer`, `WebApplication`, `DBMS`, `Database`, `Network`, `Port`, `Container.Runtime`, `Container.Application`, `Storage.BlockStorage`, `Storage.ObjectStorage`, `LoadBalancer` node types, and all normative capability and relationship types.

### Importing with namespace

```yaml
imports:
  - profile: org.oasis-open.simple:2.0
    namespace: tosca
  - url: ./my-types.yaml
    namespace: local
```

When using namespaces, prefix type names:
```yaml
node_templates:
  my_server:
    type: tosca:Compute
  my_app:
    type: local:MyCustomType
```

### Defining a profile

```yaml
tosca_definitions_version: tosca_2_0
profile: com.example.myprofile:1.0

imports:
  - profile: org.oasis-open.simple:2.0

node_types:
  MySpecialApp:
    derived_from: SoftwareComponent
    ...
```

---

## Community Profiles — OASIS GitHub Repository

The TOSCA 2.0 profile ecosystem is **actively evolving**. The canonical source of community profiles is:

> **https://github.com/oasis-open/tosca-community-contributions**

### When to consult the repo

Whenever the user asks about:
- A specific technology (Kubernetes, OpenStack, Helm, cloud providers, telco…)
- Whether a profile already exists for something
- The current types in a community profile
- Best practices for designing profiles

**Use the GitHub MCP tool** (`github-mcp-server-get_file_contents`) to browse the repository live. Don't guess — the repo changes frequently and the snapshot in `references/tosca2-community-profiles.md` may be stale.

### How to navigate the repo

The profiles directory is organized by reverse-domain namespace:

```
profiles/
├── org/oasis-open/simple/2.0/   ← normative OASIS profile (stable)
├── community/tosca/             ← community abstract + technology types
│   ├── abstract/
│   ├── core/
│   └── technology/
├── community/edmm.yaml          ← Essential Deployment Metamodel
├── community/open-tosca.yaml    ← OpenTOSCA community types
├── io/k8s/                      ← Kubernetes API profiles
├── cloud/puccini/               ← Puccini orchestrator profiles
│   ├── kubernetes/
│   ├── helm/
│   ├── kubevirt/
│   └── openstack/
├── com/ubicity/                 ← Ubicity profiles
└── si/steampunk/                ← Steampunk profiles
```

### Workflow for profile-aware requests

When the user needs types for a specific technology:

1. **Check the repo** — call `github-mcp-server-get_file_contents` with `owner: oasis-open`, `repo: tosca-community-contributions`, `path: profiles` to list available profiles.
2. **Drill into the relevant directory** — e.g. `path: profiles/io/k8s` for Kubernetes types.
3. **Read the profile file** — get the actual YAML to understand available types before writing the user's file.
4. **Import it** — reference the profile by its declared `profile:` name. Example:
   ```yaml
   imports:
     - profile: org.oasis-open.simple:2.0
     - url: https://raw.githubusercontent.com/oasis-open/tosca-community-contributions/main/profiles/io/k8s/api/v1/profile.yaml
       namespace: k8s
   ```
5. **Prefer community types over inventing custom ones** — if a profile already models what the user needs, use it.

### Profile naming convention (TOSCA 2.0 best practice)

Profile names use **reverse domain notation**, NOT dotted type names:

```yaml
# ✅ GOOD — profile-level naming
profile: com.example.myapp:1.0

node_types:
  AppServer:       # short, clean name; namespacing handled by profile import
    derived_from: SoftwareComponent

# ❌ BAD — TOSCA 1.x style (avoid even in 2.0)
node_types:
  com.example.myapp.nodes.AppServer:
    derived_from: tosca.nodes.SoftwareComponent
```

---

## Workflow

When asked to create a TOSCA 2.0 file:

1. **Clarify the goal** — understand what infrastructure/application is being modeled and what the user wants to deploy or describe.
2. **Determine the file purpose** — is it a service template (deploying something), a type library (defining reusable types), or a profile?
3. **Check for existing community profiles** — if the technology has a community profile in the OASIS repo, read it via GitHub MCP before writing custom types. Read `references/tosca2-community-profiles.md` for a snapshot, then verify live if freshness matters.
4. **Choose the right types** — prefer normative types from `org.oasis-open.simple:2.0`, then community profiles, before inventing custom ones. Consult `references/tosca2-types.md` for the normative catalog.
5. **Import the profile** — always include `imports: - profile: org.oasis-open.simple:2.0` unless the file IS a profile defining types at that level.
6. **Write the file** — produce a clean, commented YAML file that validates against TOSCA 2.0 rules.
7. **Review for common mistakes** — check the list below.

### Common mistakes to avoid

- ❌ Using `topology_template` → use `service_template`
- ❌ Using `tosca.nodes.Compute` → use `Compute` (with profile imported)
- ❌ Using `occurrences:` in requirements → use `count_range:`
- ❌ Missing `tosca_definitions_version: tosca_2_0` at top
- ❌ Forgetting to import `org.oasis-open.simple:2.0` when using normative types
- ❌ Using `get_input` in a type definition (only valid in service_template)
- ❌ Using `tosca_simple_yaml_1_3` or any 1.x version string
- ❌ Using `valid_source_node_types` → use `valid_source_types` in capabilities
- ❌ Using string `UNBOUNDED` without enclosing in a list: `count_range: [0, UNBOUNDED]`

---

## Reference Files

- **`references/tosca2-types.md`** — Full catalog of TOSCA 2.0 normative types (node, capability, relationship, interface, artifact, data) from `org.oasis-open.simple:2.0`. Read when you need to know what types are available.
- **`references/tosca2-examples.md`** — Complete working TOSCA 2.0 examples (web app, microservices, custom profile). Read when the user needs a full example or when you want to check patterns.
- **`references/tosca2-community-profiles.md`** — Snapshot of available community profiles in the OASIS GitHub repo (`oasis-open/tosca-community-contributions`). Use as a starting point, then verify live with GitHub MCP if the profile may have changed.
