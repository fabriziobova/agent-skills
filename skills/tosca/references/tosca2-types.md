# TOSCA 2.0 Normative Types Reference

Source: `org.oasis-open.simple:2.0` profile  
GitHub: https://github.com/oasis-open/tosca-community-contributions/tree/main/profiles/org/oasis-open/simple/2.0

---

## Node Types

All inherit from `Root` unless stated otherwise.

### Root
Base node type. All TOSCA nodes ultimately derive from this.
- **Attributes**: `state: string`
- **Capabilities**: `feature: Node`
- **Requirements**: `dependency` → capability: `Node`, relationship: `DependsOn`, count_range: `[0, UNBOUNDED]`
- **Interfaces**: `Standard: Lifecycle.Standard`

### Abstract.Compute
Abstract compute resource (no storage/network requirements).
- **derived_from**: `Root`
- **Capabilities**: `host: Compute` (valid_source_types: [])

### Compute
Real or virtual server with CPU, memory, storage.
- **derived_from**: `Abstract.Compute`
- **Attributes**: `private_address`, `public_address`, `networks` (map of NetworkInfo), `ports` (map of PortInfo)
- **Requirements**: `local_storage` → capability: `Attachment`, node: `Storage.BlockStorage`, relationship: `AttachesTo`, count_range: `[0, UNBOUNDED]`
- **Capabilities**: `host: Compute` (valid_source_types: [SoftwareComponent]), `endpoint: Endpoint.Admin`, `os: OperatingSystem`, `scalable: Scalable`, `binding: Bindable`

### SoftwareComponent
Generic software running on a Compute node.
- **derived_from**: `Root`
- **Properties**: `component_version: version` (optional), `admin_credential: Credential` (optional)
- **Requirements**: `host` → capability: `Compute`, node: `Compute`, relationship: `HostedOn`, count_range: `[0, 1]`

### WebServer
Web server hosting WebApplication nodes.
- **derived_from**: `SoftwareComponent`
- **Capabilities**: `data_endpoint: Endpoint`, `admin_endpoint: Endpoint.Admin`, `host: Compute` (valid_source_types: [WebApplication])

### WebApplication
Web application hosted on a WebServer.
- **derived_from**: `Root`
- **Properties**: `context_root: string` (optional)
- **Capabilities**: `app_endpoint: Endpoint`
- **Requirements**: `host` → capability: `Compute`, node: `WebServer`, relationship: `HostedOn`

### DBMS
Relational database management system.
- **derived_from**: `SoftwareComponent`
- **Properties**: `root_password: string` (optional), `port: integer` (optional)
- **Capabilities**: `host: Compute` (valid_source_types: [Database])

### Database
Logical database hosted on a DBMS.
- **derived_from**: `Root`
- **Properties**: `name: string`, `port: integer`, `user: string` (optional), `password: string` (optional)
- **Requirements**: `host` → capability: `Compute`, node: `DBMS`, relationship: `HostedOn`
- **Capabilities**: `database_endpoint: Endpoint.Database`

### Abstract.Storage
Abstract storage resource.
- **derived_from**: `Root`
- **Properties**: `size: scalar-unit.size` (≥ 0 MB)

### Storage.ObjectStorage
Object/blob storage.
- **derived_from**: `Abstract.Storage`
- **Properties**: `name: string`, `maxsize: scalar-unit.size` (optional, ≥ 0 GB)
- **Capabilities**: `storage_endpoint: Endpoint`

### Storage.BlockStorage
Block storage device (server-local).
- **derived_from**: `Abstract.Storage`
- **Properties**: `size: scalar-unit.size` (≥ 1 MB), `volume_id: string` (optional), `snapshot_id: string` (optional)
- **Capabilities**: `attachment: Attachment`

### Container.Runtime
Container runtime (e.g. Docker).
- **derived_from**: `SoftwareComponent`
- **Capabilities**: `host: Compute`, `scalable: Scalable`

### Container.Application
Application requiring container-level virtualization.
- **derived_from**: `Root`
- **Requirements**: `host` → capability: `Compute`, node: `Container.Runtime`, relationship: `HostedOn`

### Network
Logical network service.
- **derived_from**: `Root`
- **Properties**: `ip_version: integer` (4 or 6, default 4), `cidr`, `start_ip`, `end_ip`, `gateway_ip`, `network_name`, `network_id`, `segmentation_id`, `network_type`, `physical_network`, `dhcp_enabled: boolean` (default false)
- **Capabilities**: `link: Linkable`

### Port
Virtual NIC that connects a Compute node to a Network.
- **derived_from**: `Root`
- **Properties**: `ip_address` (optional), `order: integer` (default 0), `is_default: boolean`, `ip_range_start`, `ip_range_end`
- **Requirements**: `link` → capability: `Linkable`, relationship: `LinksTo` / `binding` → capability: `Bindable`, relationship: `BindsTo`

### LoadBalancer
Load balancer distributing traffic across application instances.
- **derived_from**: `Root`
- **Properties**: `algorithm: string` (optional, experimental)
- **Capabilities**: `client: Endpoint.Public` (count_range: [0, UNBOUNDED])
- **Requirements**: `application` → capability: `Endpoint`, relationship: `RoutesTo`, count_range: [0, UNBOUNDED]

---

## Capability Types

### Node
Base capability. All nodes have a `feature: Node` capability.

### Container
Indicates the node can host other nodes.

### Compute
Hosting on a named compute resource. Extends Container.
- **Properties**: `num_cpus: integer`, `cpu_frequency: scalar-unit.frequency`, `disk_size: scalar-unit.size`, `mem_size: scalar-unit.size`

### Endpoint
Network endpoint (layer 4–7).
- **Properties**: `protocol: string` (default tcp), `port: PortDef` (optional), `secure: boolean` (default false), `url_path: string` (optional), `port_name: string` (optional), `network_name: string` (default PRIVATE), `initiator: string` (source/target/peer), `ports: map of PortSpec` (optional)
- **Attributes**: `ip_address: string`

### Endpoint.Public
Public internet-accessible endpoint. Extends `Endpoint`. `network_name` defaults to PUBLIC.
- **Properties**: `floating: boolean` (experimental), `dns_name: string` (experimental)

### Endpoint.Admin
Administrator endpoint. Extends `Endpoint`. `secure` defaults to true.

### Endpoint.Database
Database endpoint. Extends `Endpoint`.

### Attachment
Attachment capability (for BlockStorage).

### OperatingSystem
OS capability for Compute nodes.
- **Properties**: `architecture: string`, `type: string`, `distribution: string`, `version: version`

### Scalable
Scalability capability.
- **Properties**: `min_instances: integer` (default 1), `max_instances: integer` (default 1), `default_instances: integer` (optional)

### Network
Network addressability capability.

### Bindable (extends Node)
Node can be bound to a network port.

### Linkable (extends Node)
Node can be pointed to by a LinksTo relationship.

---

## Relationship Types

All inherit from `Root` unless stated otherwise.

### Root
Base relationship. Has `Configure: Relationship.Configure` interface.

### DependsOn (extends Root)
General dependency between two nodes.
- **valid_target_types**: [Node]

### HostedOn (extends Root)
Hosting relationship.
- **valid_target_types**: [Container]

### ConnectsTo (extends Root)
Network connection between two nodes.
- **valid_target_types**: [Endpoint]
- **Properties**: `credential: Credential` (optional)

### AttachesTo (extends Root)
Attachment (e.g. block storage to compute).
- **valid_target_types**: [Attachment]
- **Properties**: `location: string` (min_length 1), `device: string` (optional)

### LinksTo (extends DependsOn)
Association between Port and Network.
- **valid_target_types**: [Linkable]

### BindsTo (extends DependsOn)
Association between Port and Compute.
- **valid_target_types**: [Bindable]

### RoutesTo (extends ConnectsTo)
Network routing between endpoints.
- **valid_target_types**: [Endpoint]

---

## Interface Types

### Lifecycle.Standard
Standard lifecycle for node instances.
- **Operations**: `create`, `configure`, `start`, `modify`, `stop`, `delete`

### Relationship.Configure
Lifecycle for relationships.
- **Operations**: `pre_configure_source`, `pre_configure_target`, `post_configure_source`, `post_configure_target`, `add_target`, `add_source`, `target_changed`, `remove_target`

---

## Artifact Types

### Root
Base artifact type.

### Deployment
Deployment artifact base. Extends Root.

### Deployment.Image
Machine/container image. Extends Deployment.

### Deployment.Image.VM
Virtual machine image. Extends Deployment.Image.

### Implementation
Implementation artifact base. Extends Root.

### Implementation.Bash
Shell script (`application/x-sh`). Extends Implementation.

### Implementation.Python
Python script (`application/x-python`). Extends Implementation.

### Deployment.Web
Web application archive (`.war`). Extends Deployment.

---

## Commonly Used Data Types (from `org.oasis-open.simple:2.0`)

| Type | Description |
|---|---|
| `Credential` | `protocol`, `token_type`, `token`, `keys` (map), `user` |
| `NetworkInfo` | `network_name`, `network_id`, `addresses` (list of string) |
| `PortInfo` | `port_name`, `port_id`, `network_id`, `mac_address`, `addresses` (list) |
| `PortDef` | Port number (integer, 1–65535) |
| `PortSpec` | `protocol`, `source`, `source_range`, `target`, `target_range` |

---

## Scalar Units

TOSCA 2.0 supports scalar-unit types for sizes and frequencies:

- `scalar-unit.size`: e.g. `10 GB`, `512 MB`, `2 TB`
- `scalar-unit.frequency`: e.g. `2.4 GHz`, `1 MHz`
- `scalar-unit.time`: e.g. `10 s`, `500 ms`
- `scalar-unit.bitrate`: e.g. `10 Gbps`, `100 Mbps`
