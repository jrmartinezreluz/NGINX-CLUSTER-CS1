# Architecture diagrams

Visual reference for **NGINX-CLUSTER-CS1**: NGINX Plus active/passive pair with Keepalived VIP, L7 app proxying, and optional MySQL stream.

---

## High availability overview

```mermaid
flowchart TB
    Clients[Clients / Browsers]

    subgraph VIP_Layer["Floating VIP (Keepalived VRRP)"]
        VIP["VIP x.x.x.x<br/>eth1"]
    end

    subgraph NGINX_Cluster["NGINX Plus — active/passive"]
        PRI[nginx-primary<br/>MASTER priority 150]
        SEC[nginx-secondary<br/>BACKUP priority 100]
    end

    subgraph Apps["Backend pools"]
        A1[App1 servers :3000]
        A2[App2 servers :3000]
        DB[(MySQL :3306)]
    end

    Clients -->|HTTP 80 / HTTPS 443| VIP
    VIP -.->|active holder| PRI
    PRI <-->|VRRP| SEC
    PRI & SEC --> A1 & A2
    PRI & SEC -->|stream 3306| DB
```

| Node | Keepalived role | Typical priority |
|------|-----------------|------------------|
| `nginx-primary` | `MASTER` | 150 |
| `nginx-secondary` | `BACKUP` | 100 |

Only the node holding the VIP serves traffic on `listen <VIP>:80/443`.

---

## Traffic paths

### Application HTTP(S)

```mermaid
flowchart LR
    User[User]
    VIP[VIP / DNS]
    NGX[NGINX Plus]
    U1[upstream app1_backend]
    U2[upstream app2_backend]

    User -->|app1.ghc.com| VIP --> NGX
    NGX --> U1
    User -->|app2.ghc.com| VIP --> NGX
    NGX --> U2
```

- **App1:** `least_conn` to backend pool (`app1_backend`)
- **App2:** `ip_hash` to backend pool (`app2_backend`)

### NGINX Plus dashboard

```mermaid
sequenceDiagram
    participant Admin
    participant VIP as Floating VIP
    participant NGX as NGINX Plus
    participant API as Plus API /status

    Admin->>VIP: GET https://host/dashboard.html
    VIP->>NGX: TLS terminate
    NGX->>Admin: dashboard UI
    Admin->>VIP: /api/ /status
    NGX->>API: Plus monitoring API
```

Use `nginx-dashboard-conf-open.yml` for lab validation (`allow all`), then `fix-dashboard-api.yml` or restricted template for production.

### MySQL TCP (stream)

```mermaid
flowchart LR
    App[Application clients]
    VIP[VIP :3306]
    NGX[NGINX stream]
    DB1[(MySQL primary)]
    DB2[(MySQL secondary)]

    App --> VIP --> NGX
    NGX -->|least_conn| DB1 & DB2
```

---

## Ansible deployment pipeline

```mermaid
flowchart TD
    A[nginx-uninstall.yml] --> B[nginx-install.yml<br/>Plus + license]
    B --> C[Copy SSL certs to /etc/nginx/ssl]
    C --> D[nginx-conf.yml<br/>base nginx.conf]
    D --> E[nginx-deploy-conf.yml<br/>upstreams + vhosts + stream]
    E --> F[nginx-dashboard-conf-open.yml<br/>optional validation]
    F --> G[fix-dashboard-api.yml<br/>if API issues]
    G --> H[nginx-keepalived-ha.yml<br/>VIP failover]
    H --> I[curl VIP / dashboard / failover test]
```

---

## Repository map

```mermaid
flowchart LR
    subgraph Repo["NGINX-CLUSTER-CS1"]
        DOC[docs/architecture.md]
        AN[ansible/nginx/]
    end

    AN --> PB[playbooks *.yml]
    AN --> TMPL[templates/*.j2]
    AN --> FL[files/<br/>license + certs]
    AN --> GV[group_vars/]
```

---

## Related files

| File | Purpose |
|------|---------|
| [README.md](../README.md) | Quick start |
| `ansible/nginx/files/README.md` | Required license and TLS files |
| `ansible/nginx/group_vars/all.yml.example` | VIP, backends, SSL paths |
