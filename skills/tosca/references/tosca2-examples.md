# TOSCA 2.0 Working Examples

---

## Example 1: Minimal Service Template (Web App on Compute)

```yaml
tosca_definitions_version: tosca_2_0

imports:
  - profile: org.oasis-open.simple:2.0

description: >-
  Simple web application deployed on a virtual machine.

service_template:
  inputs:
    vm_cpus:
      type: integer
      default: 2
    vm_mem:
      type: scalar-unit.size
      default: 4 GB
    app_port:
      type: integer
      default: 8080

  node_templates:
    my_server:
      type: Compute
      capabilities:
        host:
          properties:
            num_cpus: { get_input: vm_cpus }
            mem_size: { get_input: vm_mem }
        os:
          properties:
            type: linux
            distribution: ubuntu
            version: "22.04"

    my_webserver:
      type: WebServer
      requirements:
        - host: my_server

    my_webapp:
      type: WebApplication
      properties:
        context_root: /app
      capabilities:
        app_endpoint:
          properties:
            port: { get_input: app_port }
      requirements:
        - host: my_webserver

  outputs:
    app_ip:
      description: Public IP of the application.
      value: { get_attribute: [my_server, public_address] }
```

---

## Example 2: Custom Node Types + Service Template

```yaml
tosca_definitions_version: tosca_2_0

imports:
  - profile: org.oasis-open.simple:2.0

description: >-
  Custom types for a Python microservice with PostgreSQL.

node_types:
  PythonMicroservice:
    derived_from: SoftwareComponent
    description: A Python-based microservice component.
    properties:
      app_name:
        type: string
        description: Name of the microservice.
      listen_port:
        type: integer
        default: 5000
      debug_mode:
        type: boolean
        default: false
        required: false
    capabilities:
      api_endpoint:
        type: Endpoint
    requirements:
      - host:
          capability: Compute
          node: Compute
          relationship: HostedOn
          count_range: [1, 1]
      - db:
          capability: Endpoint.Database
          relationship: ConnectsTo
          count_range: [0, 1]
    interfaces:
      Standard:
        type: Lifecycle.Standard
        operations:
          create:
            implementation: scripts/install.sh
          configure:
            inputs:
              PORT: { get_property: [SELF, listen_port] }
              APP: { get_property: [SELF, app_name] }
            implementation: scripts/configure.sh
          start:
            implementation: scripts/start.sh
          stop:
            implementation: scripts/stop.sh

service_template:
  inputs:
    service_name:
      type: string
    db_password:
      type: string

  node_templates:
    app_server:
      type: Compute
      capabilities:
        host:
          properties:
            num_cpus: 4
            mem_size: 8 GB
        os:
          properties:
            type: linux
            distribution: ubuntu

    db_server:
      type: Compute
      capabilities:
        host:
          properties:
            num_cpus: 2
            mem_size: 4 GB

    my_postgres:
      type: DBMS
      properties:
        port: 5432
        root_password: { get_input: db_password }
      requirements:
        - host: db_server

    my_database:
      type: Database
      properties:
        name: mydb
        port: 5432
        user: appuser
        password: { get_input: db_password }
      requirements:
        - host: my_postgres

    my_service:
      type: PythonMicroservice
      properties:
        app_name: { get_input: service_name }
        listen_port: 8080
      requirements:
        - host: app_server
        - db: my_database

  outputs:
    service_endpoint:
      value: { get_attribute: [my_service, api_endpoint, ip_address] }
```

---

## Example 3: Multi-file Project with Profile

### `profile.yaml` — Reusable type library

```yaml
tosca_definitions_version: tosca_2_0

profile: com.example.microservices:1.0

imports:
  - profile: org.oasis-open.simple:2.0

description: >-
  Common microservice types for Example Corp.

capability_types:
  ServiceEndpoint:
    derived_from: Endpoint
    description: Typed endpoint for inter-service communication.
    properties:
      service_name:
        type: string
        required: false

node_types:
  MicroService:
    derived_from: Container.Application
    description: Base type for all microservices.
    properties:
      name:
        type: string
      image:
        type: string
        description: Docker image reference.
      replicas:
        type: integer
        default: 1
        required: false
    capabilities:
      endpoint:
        type: ServiceEndpoint
    requirements:
      - endpoint:
          capability: ServiceEndpoint
          relationship: ConnectsTo
          count_range: [0, UNBOUNDED]
```

### `main.yaml` — Service template using the profile

```yaml
tosca_definitions_version: tosca_2_0

imports:
  - profile: com.example.microservices:1.0
    namespace: svc

description: >-
  E-commerce platform service template.

service_template:
  inputs:
    frontend_replicas:
      type: integer
      default: 2

  node_templates:
    frontend:
      type: svc:MicroService
      directives: [substitute]
      properties:
        name: frontend
        image: example/frontend:latest
        replicas: { get_input: frontend_replicas }
      capabilities:
        endpoint:
          properties:
            port: 80
      requirements:
        - endpoint: catalog_service
        - endpoint: cart_service

    catalog_service:
      type: svc:MicroService
      directives: [substitute]
      properties:
        name: catalog
        image: example/catalog:latest
      capabilities:
        endpoint:
          properties:
            port: 8080

    cart_service:
      type: svc:MicroService
      directives: [substitute]
      properties:
        name: cart
        image: example/cart:latest
      capabilities:
        endpoint:
          properties:
            port: 8081
```

---

## Example 4: Substitution Mapping (Node Implementation)

A service template that implements a node type via substitution:

### `compute-node.yaml` — Implements `Abstract.Compute`

```yaml
tosca_definitions_version: tosca_2_0

imports:
  - profile: org.oasis-open.simple:2.0

description: >-
  Substitution template: implements an Abstract.Compute with a VM + block storage.

service_template:
  inputs:
    cpus:
      type: integer
      default: 2
    memory:
      type: scalar-unit.size
      default: 4 GB
    disk:
      type: scalar-unit.size
      default: 50 GB

  node_templates:
    the_vm:
      type: Compute
      capabilities:
        host:
          properties:
            num_cpus: { get_input: cpus }
            mem_size: { get_input: memory }

    the_disk:
      type: Storage.BlockStorage
      properties:
        size: { get_input: disk }
      requirements:
        - attachment:
            node: the_vm
            relationship:
              type: AttachesTo
              properties:
                location: /data

  substitution_mappings:
    node_type: Abstract.Compute
    capabilities:
      host: [the_vm, host]
    requirements:
      local_storage: [the_vm, local_storage]
```

---

## Example 5: Using Namespaces and Multiple Imports

```yaml
tosca_definitions_version: tosca_2_0

imports:
  - profile: org.oasis-open.simple:2.0
    namespace: tosca
  - url: ./custom-types.yaml
    namespace: custom

description: >-
  Template using both normative and custom types with namespaces.

service_template:
  node_templates:
    web_server:
      type: tosca:WebServer
      requirements:
        - host:
            node: my_vm

    my_app:
      type: custom:MySpecialApp
      requirements:
        - host: web_server

    my_vm:
      type: tosca:Compute
      capabilities:
        host:
          properties:
            num_cpus: 2
            mem_size: 2 GB
```

---

## Example 6: Network Topology

```yaml
tosca_definitions_version: tosca_2_0

imports:
  - profile: org.oasis-open.simple:2.0

description: >-
  Application with explicit network topology.

service_template:
  node_templates:
    private_net:
      type: Network
      properties:
        ip_version: 4
        cidr: 192.168.10.0/24
        dhcp_enabled: true

    app_server:
      type: Compute
      capabilities:
        host:
          properties:
            num_cpus: 2
            mem_size: 4 GB

    app_port:
      type: Port
      properties:
        order: 0
        is_default: true
      requirements:
        - link: private_net
        - binding: app_server

    db_server:
      type: Compute
      capabilities:
        host:
          properties:
            num_cpus: 2
            mem_size: 8 GB

    db_port:
      type: Port
      properties:
        order: 0
      requirements:
        - link: private_net
        - binding: db_server

    my_app:
      type: SoftwareComponent
      requirements:
        - host: app_server

    my_db:
      type: DBMS
      properties:
        port: 5432
      requirements:
        - host: db_server
```
