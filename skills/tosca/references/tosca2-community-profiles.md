# TOSCA 2.0 Community Profiles Reference

> ⚠️ **Questo file è uno snapshot** del repository `oasis-open/tosca-community-contributions`.
> I profili sono in evoluzione continua. Verifica sempre lo stato attuale tramite GitHub MCP:
> ```
> github-mcp-server-get_file_contents owner=oasis-open repo=tosca-community-contributions path=profiles
> ```

Repository: https://github.com/oasis-open/tosca-community-contributions/tree/main/profiles

---

## Profili normative OASIS

### `org.oasis-open.simple:2.0`
**Path:** `profiles/org/oasis-open/simple/2.0/`  
**Profile entry:** `profiles/org/oasis-open/simple/2.0/profile.yaml`

Il profilo normativo ufficiale OASIS. Equivalente TOSCA 2.0 del "TOSCA Simple Profile 1.3".
Contiene tutti i tipi base: `Compute`, `SoftwareComponent`, `WebServer`, `Database`, `Network`, `Port`, `LoadBalancer`, ecc.

**Import:**
```yaml
imports:
  - profile: org.oasis-open.simple:2.0
```

---

## Profili Community

### Community TOSCA Abstract + Technology
**Path:** `profiles/community/tosca/`  
**Struttura:**
- `abstract/` — tipi astratti riutilizzabili (MicroService, Container, ecc.)
- `core/` — tipi fondamentali della community
- `technology/` — tipi specifici per tecnologie (Docker, Kubernetes, cloud…)
- `inventory.md` — inventario completo dei tipi disponibili
- `patterns.md` — pattern di design TOSCA 2.0 raccomandati

Per vedere i tipi disponibili:
```
github-mcp-server-get_file_contents path=profiles/community/tosca/inventory.md
```

### OpenTOSCA Community Types
**Path:** `profiles/community/open-tosca.yaml`  
**Profile:** file singolo YAML  
Tipi per l'orchestratore OpenTOSCA. Contiene node types per applicazioni cloud, container, VM.

### EDMM (Essential Deployment Metamodel)
**Path:** `profiles/community/edmm.yaml`  
Profilo per il metamodello di deployment EDMM.

---

## Profili Kubernetes

### Kubernetes API (io/k8s)
**Path:** `profiles/io/k8s/`  
**Struttura:**
```
io/k8s/
├── api/                     ← Kubernetes core API types
├── apiextensions-apiserver/ ← CRD types
├── apimachinery/            ← API machinery types
└── kube-aggregator/         ← API aggregation types
```

Per vedere i tipi Kubernetes disponibili:
```
github-mcp-server-get_file_contents path=profiles/io/k8s/api
```

---

## Profili Cloud (Puccini)

Profili per l'orchestratore [Puccini](https://github.com/tliron/puccini).

**Path:** `profiles/cloud/puccini/`

| Sub-profilo | Path | Contenuto |
|---|---|---|
| Kubernetes | `profiles/cloud/puccini/kubernetes/` | Tipi K8s per Puccini |
| Helm | `profiles/cloud/puccini/helm/` | Tipi Helm chart |
| KubeVirt | `profiles/cloud/puccini/kubevirt/` | VM su Kubernetes |
| OpenStack | `profiles/cloud/puccini/openstack/` | Tipi OpenStack |

---

## Profili Vendor

| Vendor | Path | Note |
|---|---|---|
| Ubicity | `profiles/com/ubicity/` | Profili Ubicity |
| Steampunk | `profiles/si/steampunk/` | Profili XLAB Steampunk |

---

## Come leggere un profilo live

Quando l'utente ha bisogno di tipi per una specifica tecnologia, usa GitHub MCP per leggere il profilo attuale:

```python
# 1. Lista i file nel profilo
github-mcp-server-get_file_contents(
    owner="oasis-open",
    repo="tosca-community-contributions",
    path="profiles/community/tosca/technology"
)

# 2. Leggi il file YAML del profilo
github-mcp-server-get_file_contents(
    owner="oasis-open",
    repo="tosca-community-contributions",
    path="profiles/community/tosca/technology/<filename>.yaml"
)
```

Poi usa il `profile:` name dichiarato nel file per costruire l'import nel file dell'utente.

---

## Import da URL (per profili non ancora distribuiti come package)

Se un profilo non ha ancora un nome profile registrato, si può importare tramite URL raw:

```yaml
imports:
  - profile: org.oasis-open.simple:2.0
  - url: https://raw.githubusercontent.com/oasis-open/tosca-community-contributions/main/profiles/community/tosca/abstract/<file>.yaml
    namespace: community
```

---

## Best Practices per i Profili (da `profiles/README.md`)

1. **Naming:** usa la notazione reverse-domain (`org.mycompany.myprofile:1.0`), NON nomi dotted lunghi come `tosca.nodes.myapp.MyType`.
2. **File organization:** non dividere i tipi per categoria in file separati (node_types.yaml, relationship_types.yaml…) — causa problemi di import circolare. Meglio un file per dominio logico.
3. **Namespaces:** sfrutta i namespace di import TOSCA 2.0 invece di prefissare i nomi dei tipi.
4. **Versioning:** includi la versione nel profile name: `com.example.myprofile:1.0`.
