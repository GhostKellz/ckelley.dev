# Security Posture

How the site and its host are protected. The guiding principle is a **split between a public data
plane and a private management plane**: visitors only ever touch Cloudflare and NGINX over HTTPS,
while all administration, deployment, and security signalling happen over a private Tailscale
network.

This is a **production** posture — the origin runs on a PVE bare-metal cloud node, and the
CrowdSec and observability stacks it reports into are shared across the production PVE cluster
with HA and redundancy.

## Trust Planes

```mermaid
%%{init: {'theme':'base','themeVariables':{'fontFamily':'Inter, system-ui, sans-serif','primaryColor':'#112240','primaryBorderColor':'#64ffda','primaryTextColor':'#e6f1ff','lineColor':'#64ffda','fontSize':'14px'}}}%%
flowchart TB
    subgraph PUBLIC["Public Internet"]
        USER(["Site visitors"]):::public
        CF["Cloudflare edge<br/>WAF · CDN · TLS"]:::external
        LE["Let's Encrypt<br/>ACME CA"]:::external
        OTHER["SSH / mgmt ports<br/>direct host access"]:::deny
    end

    subgraph TAILNET["Tailscale Tailnet — WireGuard, private"]
        ADMIN["Dev workstation<br/>admin + deploy"]:::dev
        CROWD["Central CrowdSec LAPI<br/>production PVE cluster"]:::security
        OBS["Observability stack<br/>Prometheus · Loki · Wazuh · Grafana"]:::security
    end

    subgraph VM["PVE1 NGINX VM — hardened origin"]
        direction TB
        NGX["NGINX<br/>TLS · security headers"]:::host
        BNC["CrowdSec nginx bouncer"]:::security
        ACMECL["acme.sh<br/>DNS-01 renewer"]:::host
        AGENTS["node_exporter · syslog-ng · Wazuh agent"]:::host
        NGX --- BNC
    end

    CFDNS["Cloudflare DNS<br/>scoped API token"]:::external

    USER ==>|"HTTPS 443"| CF
    CF ==>|"proxied origin pull · 443 only"| NGX
    ADMIN -.->|"SSH key auth<br/>Tailscale-only"| VM
    BNC -.->|"pull ban decisions"| CROWD
    AGENTS -.->|"metrics · logs · events"| OBS
    ACMECL -->|"DNS-01 TXT challenge"| CFDNS
    LE -.->|"issue / renew cert"| ACMECL
    OTHER -.->|"DENY at firewall"| VM

    classDef dev fill:#112240,stroke:#64ffda,stroke-width:1.5px,color:#e6f1ff;
    classDef host fill:#0a192f,stroke:#64ffda,stroke-width:1.5px,color:#e6f1ff;
    classDef security fill:#06281f,stroke:#10B981,stroke-width:1.5px,color:#6ee7b7;
    classDef public fill:#020c1b,stroke:#8892b0,stroke-width:1.5px,color:#ccd6f6;
    classDef external fill:#1a1300,stroke:#fbbf24,stroke-width:1.5px,color:#fcd34d;
    classDef deny fill:#2a0a0a,stroke:#EF4444,stroke-width:1.5px,color:#fca5a5;
```

**Reading the diagram**

- **Solid edges** are the public data path (visitors → Cloudflare → NGINX origin). **Dotted
  edges** are private control/observe flows (admin SSH, CrowdSec decisions, telemetry,
  certificate renewal).
- The only inbound public exposure is **HTTPS/443**, and the origin only accepts it from
  Cloudflare. SSH and management ports are blocked at the firewall (default-deny) and reachable
  only across the Tailscale mesh.
- Certificate issuance never needs inbound HTTP — acme.sh proves control via **DNS-01** by
  writing TXT records in Cloudflare DNS (see [deployment.md](deployment.md#tls-certificates)).

## Controls Summary

| Layer | Control |
|-------|---------|
| **Edge (public)** | Cloudflare (Pro): WAF, rate limiting, bot management, TLS termination |
| **Transport** | TLS Full (Strict) Cloudflare↔origin via the Let's Encrypt origin certificate; HTTP→HTTPS redirect |
| **HTTP headers** | `X-Frame-Options`, `X-Content-Type-Options`, `X-XSS-Protection`, `Referrer-Policy` |
| **Host access** | Tailscale-only SSH, key-based auth, no password auth, port 22 not public |
| **Edge protection** | CrowdSec nginx bouncer enforcing decisions from a central CrowdSec LAPI |
| **Certificates** | acme.sh + Let's Encrypt, DNS-01 via a scoped Cloudflare API token |
| **Attack surface** | Static files only — no app runtime, database, or CMS to exploit |
| **Observability** | node_exporter, syslog-ng→Loki, Wazuh agent, CrowdSec metrics → central stack |
| **Secrets** | Cloudflare token, SSH keys, origin cert key live on the host only; never committed |

## CrowdSec — NGINX Bouncer

The PVE1 NGINX VM runs the **CrowdSec NGINX bouncer**, which checks every incoming request
against a decision list and blocks known-bad IPs before they reach the site. Rather than running
a full standalone CrowdSec install on the public edge, the bouncer **queries a central CrowdSec
security engine (LAPI + database) on the production PVE cluster over Tailscale** — the
decision-making and threat intelligence stay on the private management plane. This is the same
pattern used across every production edge (see
[Production Topology](#crowdsec--production-topology--threat-feed) below).

```mermaid
%%{init: {'theme':'base','themeVariables':{'fontFamily':'Inter, system-ui, sans-serif','primaryColor':'#112240','primaryBorderColor':'#64ffda','primaryTextColor':'#e6f1ff','lineColor':'#64ffda','fontSize':'14px'}}}%%
flowchart TB
    VISITOR(["Visitor / bot"]):::public

    subgraph VM["PVE1 NGINX VM — public edge"]
        direction TB
        NGX["NGINX"]:::host
        BNC["nginx bouncer<br/>Lua / OpenResty"]:::security
        CACHE["local decision<br/>cache · TTL"]:::host
        NGX --> BNC
        BNC -.- CACHE
    end

    subgraph PVE["Production PVE — CrowdSec brain, over Tailscale"]
        direction TB
        AGENT["CrowdSec agent<br/>parsers + scenarios"]:::security
        LAPI["CrowdSec LAPI<br/>:8080"]:::security
        DB["security engine + DB<br/>decisions · community blocklist"]:::security
        AGENT --> LAPI
        LAPI -.- DB
    end

    OK(["200 — served"]):::secure
    BLOCK(["403 / 429 — dropped"]):::deny

    VISITOR ==>|"HTTPS request"| NGX
    BNC <-.->|"decisions API · API key · over Tailscale"| LAPI
    BNC ==>|"clean"| OK
    BNC ==>|"banned"| BLOCK

    classDef host fill:#0a192f,stroke:#64ffda,stroke-width:1.5px,color:#e6f1ff;
    classDef security fill:#06281f,stroke:#10B981,stroke-width:1.5px,color:#6ee7b7;
    classDef public fill:#020c1b,stroke:#8892b0,stroke-width:1.5px,color:#ccd6f6;
    classDef secure fill:#06281f,stroke:#10B981,stroke-width:1.5px,color:#6ee7b7;
    classDef deny fill:#2a0a0a,stroke:#EF4444,stroke-width:1.5px,color:#fca5a5;

    %% link indices follow edge declaration order (in-subgraph edges count first)
    %% 0 NGX->BNC · 1 BNC-.-CACHE · 2 AGENT->LAPI · 3 LAPI-.-DB
    %% 4 VISITOR->NGX · 5 decisions API (amber) · 6 clean (green) · 7 banned (red)
    linkStyle 5 stroke:#F59E0B,stroke-width:2px;
    linkStyle 6 stroke:#10B981,stroke-width:2.5px;
    linkStyle 7 stroke:#EF4444,stroke-width:2.5px;
```

**How it works**

1. A request hits NGINX. The bouncer (Lua module / OpenResty) intercepts it before it is served.
2. The bouncer consults its local decision cache; on a miss or refresh interval it queries the
   **central CrowdSec LAPI over Tailscale** (`GET /v1/decisions`, authenticated with a bouncer
   API key). Because the call rides the tailnet, the LAPI is never exposed to the internet.
3. If the source IP carries an active **ban** decision, the request is dropped (`403`/`429`).
   Clean traffic is served normally.
4. The CrowdSec **agent** on the production PVE cluster maintains the decision list from parsed
   signals and the CrowdSec **community blocklist**, so the public edge benefits from shared
   threat intelligence without putting the brain on the public host.

> Log acquisition (which signals the agent parses) can run either locally on the VM shipping
> events to the central LAPI, or centrally — the enforcement path shown here (bouncer → central
> LAPI over Tailscale) is the part specific to this deployment.

## CrowdSec — Production Topology & Threat Feed

This site's bouncer is one node in a wider production setup. A **single CrowdSec security engine
+ database (LAPI)** runs centrally on the production PVE cluster; every public edge runs only a
lightweight **bouncer** that consults it over Tailscale. The same brain also publishes a **public
blocklist mirror** at `https://threat.cktechnology.io/crowdsec.txt`, which any host — including
third-party or non-tailnet edges — can pull as a plain IP feed.

```mermaid
%%{init: {'theme':'base','themeVariables':{'fontFamily':'Inter, system-ui, sans-serif','primaryColor':'#112240','primaryBorderColor':'#64ffda','primaryTextColor':'#e6f1ff','lineColor':'#64ffda','fontSize':'14px'}}}%%
flowchart TB
    COMMUNITY["CrowdSec<br/>community blocklist"]:::external

    subgraph PVE["Production PVE cluster — control plane, over Tailscale"]
        direction TB
        ENGINE["CrowdSec security engine<br/>parsers · scenarios · agent"]:::security
        LAPI["LAPI + database<br/>:8080 · decisions"]:::security
        ENGINE --> LAPI
    end

    FEED["threat.cktechnology.io/crowdsec.txt<br/>public IP blocklist mirror"]:::external

    subgraph EDGE["Production edge — bouncers only"]
        direction TB
        CKD["PVE1 NGINX VM · ckelley.dev<br/>nginx + bouncer"]:::host
        PVEWEB["PVE virtualized webservers<br/>nginx + bouncer · HA"]:::host
        EDGE3["other production edge devices"]:::host
    end

    FORTI["FortiGate firewalls<br/>external threat-feed connector"]:::external
    EXT(["Any other host serving traffic<br/>(off-tailnet, pulls public feed)"]):::public

    COMMUNITY -.->|"enrich decisions"| ENGINE
    CKD <-.->|"decisions API · Tailscale"| LAPI
    PVEWEB <-.->|"decisions API · Tailscale"| LAPI
    EDGE3 <-.->|"decisions API · Tailscale"| LAPI
    LAPI ==>|"publish decisions + community blocklist"| FEED
    FEED ==>|"HTTPS pull"| EDGE
    FEED ==>|"threat feed import"| FORTI
    FEED ==>|"HTTPS pull"| EXT

    classDef host fill:#0a192f,stroke:#64ffda,stroke-width:1.5px,color:#e6f1ff;
    classDef security fill:#06281f,stroke:#10B981,stroke-width:1.5px,color:#6ee7b7;
    classDef public fill:#020c1b,stroke:#8892b0,stroke-width:1.5px,color:#ccd6f6;
    classDef external fill:#1a1300,stroke:#fbbf24,stroke-width:1.5px,color:#fcd34d;

    %% link indices follow edge declaration order (ENGINE->LAPI inside subgraph is index 0)
    %% 0 ENGINE->LAPI · 1 COMMUNITY->ENGINE · 2/3/4 decisions API (amber)
    %% 5/6/7/8 feed publish + pulls (teal)
    linkStyle 2 stroke:#F59E0B,stroke-width:2px;
    linkStyle 3 stroke:#F59E0B,stroke-width:2px;
    linkStyle 4 stroke:#F59E0B,stroke-width:2px;
    linkStyle 5 stroke:#64ffda,stroke-width:2.5px;
    linkStyle 6 stroke:#64ffda,stroke-width:2.5px;
    linkStyle 7 stroke:#64ffda,stroke-width:2.5px;
    linkStyle 8 stroke:#64ffda,stroke-width:2.5px;
```

- **One brain, many edges.** Only the central engine holds the database and runs the
  decision-making logic. Edges (this PVE1 NGINX VM, PVE-hosted virtualized webservers behind HA,
  and the other production edge devices) carry just a bouncer, so there is no per-host CrowdSec DB
  to manage or expose.
- **Two distribution paths.** Tailnet edges pull live decisions over the private decisions API
  (amber, authenticated, never public). Anything that can't join the tailnet — third-party or
  external hosts — pulls the same intelligence from the public `crowdsec.txt` mirror (teal, plain
  HTTPS), so the threat feed travels even where Tailscale doesn't.
- **Block at scale via the firewalls.** The mirror publishes the engine's **decisions plus the
  community blocklist** as a single IP list. **FortiGate firewalls import it as an external threat
  feed** wherever applicable, so malicious actors are dropped at the network edge — before they
  ever reach a web server — across every site that consumes the feed.
- **Shared intelligence.** The CrowdSec community blocklist enriches the central engine, and
  every edge inherits it automatically — local scenarios plus crowd-sourced reputation.

## Observability — Metrics, Logs, and SIEM

Every production host (this PVE1 web VM included) ships telemetry to a central observability stack
reached over Tailscale. The stack is **observe-only**: it reads existing endpoints rather than
running security components itself.

```mermaid
%%{init: {'theme':'base','themeVariables':{'fontFamily':'Inter, system-ui, sans-serif','primaryColor':'#112240','primaryBorderColor':'#64ffda','primaryTextColor':'#e6f1ff','lineColor':'#64ffda','fontSize':'14px'}}}%%
flowchart TB
    subgraph HOSTS["Every production host — PVE1 web VM, PVE webservers, edge"]
        direction TB
        NE["node_exporter<br/>:9100 host metrics"]:::host
        LOGS["syslog-ng shipper<br/>nginx + system logs"]:::host
        WA["Wazuh agent<br/>FIM · security events"]:::security
    end

    CSL["CrowdSec LAPI<br/>:6060 cs_* metrics"]:::security

    subgraph OBS["Observability server — over Tailscale"]
        direction TB
        PROM["Prometheus<br/>:9090 · 15s scrape"]:::host
        LOKI["Loki<br/>:3100 log store"]:::host
        WI["Wazuh indexer<br/>:9200 OpenSearch"]:::security
        GRAF["Grafana<br/>:3000 dashboards + alerts"]:::host
    end

    NE -.->|"scrape"| PROM
    CSL -.->|"scrape decisions/alerts"| PROM
    LOGS ==>|"push logs"| LOKI
    WA ==>|"agent then indexer"| WI
    PROM --> GRAF
    LOKI --> GRAF
    WI -.->|"OpenSearch datasource"| GRAF

    classDef host fill:#0a192f,stroke:#64ffda,stroke-width:1.5px,color:#e6f1ff;
    classDef security fill:#06281f,stroke:#10B981,stroke-width:1.5px,color:#6ee7b7;
```

- **Metrics** — `node_exporter` on each host and the **CrowdSec LAPI** (`:6060`, `cs_*`
  decision/alert metrics) are scraped by **Prometheus**, which drives alerting.
- **Logs** — `syslog-ng` ships nginx and system logs to **Loki** with low-cardinality labels
  (`host`, `app`, `severity`) for fast query-time filtering in Grafana. (This mirrors the real
  `LogPipelineFlow` shown on the `/architecture` page.)
- **SIEM** — **Wazuh** agents report file-integrity and security events to the Wazuh indexer
  (OpenSearch); **Grafana** reads it directly as a datasource alongside Prometheus and Loki,
  giving one pane for infra health, logs, and security signals.

## Zero Trust Management Plane

Across every environment the rule is the same: **nothing administrative is reachable from the
public internet.** The CrowdSec LAPI, the observability UIs (Grafana/Prometheus/Alertmanager),
the Wazuh indexer, and all SSH live only on the **Tailscale mesh**, gated by tight tailnet
**ACLs** so each host can reach exactly the services it needs and no more. SSH is **key-based
only** (no password auth, port 22 not public). Public ingress is limited to HTTPS on the edge —
and the origin only accepts it from Cloudflare; everything else is **default-deny** at the host
firewall. Least privilege plus full observability is the whole point — see
[Trust Planes](#trust-planes).

## What This Buys Us

- **Minimal public attack surface** — one port (443), static files, no server-side code, with
  Cloudflare's WAF and bot management in front.
- **No public management plane** — SSH and the CrowdSec brain live behind Tailscale.
- **Automated, hands-off TLS** — DNS-01 renewals with no inbound exposure.
- **Resilient** — the site is fully reproducible from this repo with `pnpm build`, and the
  CrowdSec/observability posture is shared across HA production edges.
