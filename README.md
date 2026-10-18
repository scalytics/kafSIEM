# kafSIEM

Edge-ready operations intelligence with an evidence-linked entity graph.

kafSIEM watches Kafka agent traffic and OSINT feeds, materializes entities and
relationships in SQLite, and serves analyst workflows through a web desk and
typed OpenAPI. Deploy with Docker on a single host. No cluster database
required.

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache--2.0-blue.svg)](LICENSE)
[![OpenAPI](https://img.shields.io/badge/API-OpenAPI-green)](api/openapi.yaml)
[![Docker](https://img.shields.io/badge/deploy-docker--compose-blue)](docker-compose.yml)

Keywords: entity graph, Kafka SIEM, OSINT dashboard, drone fleet operations,
SCADA security, edge deployment, ontology packs, provenance, SQLite analyst API.

## Screenshots

<p align="center">
  <a href="public/images/Scalytics-OSINT-Prediction.png">
    <img src="public/images/Scalytics-OSINT-Prediction.png" alt="kafSIEM OSINT globe and alert feed" width="32%" />
  </a>
  <a href="public/images/Scalytics-Ontology-Drones.png">
    <img src="public/images/Scalytics-Ontology-Drones.png" alt="kafSIEM Operations desk with drones ontology graph" width="32%" />
  </a>
  <a href="public/images/Scalytics-Ontology-SCADA.png">
    <img src="public/images/Scalytics-Ontology-SCADA.png" alt="kafSIEM Operations desk with SCADA ontology graph" width="32%" />
  </a>
</p>
<p align="center">
  <strong>OSINT</strong> | <strong>Operations / Drones</strong> | <strong>Operations / SCADA</strong>
</p>

## Try in 30 seconds

No Kafka, API keys, or Docker required.

```bash
git clone https://github.com/scalytics/kafSIEM.git && cd kafSIEM
make help          # list all targets
make demo-osint    # public OSINT relations demo (fixtures, no Docker)
make demo-ontology # Operations ontology desk
make demo-fusion   # Fusion desk
```

`make demo-osint` opens the OSINT console with bundled alert and incident
fixtures — cyber, malware, ICS/energy, and terror clusters in the Relations
panel. No Kafka or Docker required.

## What you get

- **Entity graph** stored in SQLite: platforms, faults, devices, agents, tasks.
  Graph edges can cite the source Kafka record.
- **Analyst desk**: flow queue, topology graph, map, replay studio, entity
  profiles, provenance drawer.
- **Domain packs**: YAML ontology for drones and SCADA. Detectors, queries,
  views, map layers, report templates. No pack executable code.
- **Typed API**: `/api/v1/...` with generated [OpenAPI](api/openapi.yaml).
  Integrates with existing enterprise platforms.
- **Edge deployment**: collector writes, API reads, shared volume. Docker
  Compose or Helm.

## Who it is for

**Unmanned systems teams** tracking readiness, sorties, EW events, software
lineage, and signoff evidence across a fleet.

**SCADA and critical infrastructure teams** connecting plants, devices, change
audit, alarms, firmware, vulnerabilities, and engineer sessions.

| Pack | Example questions |
|------|-------------------|
| [Drones](docs/packs/drones.md) | What changed since last signoff? Is this fault fleet-wide? Which platforms run software version X? |
| [SCADA](docs/packs/scada.md) | Which changes lack a work order? Is firmware drifting on safety PLCs? What CVEs affect live devices? |

## How it works

```text
KafClaw agents
      |
      v
KafScale / Kafka
      |
      v
kafsiem-collector  --->  /data/agentops.db (+ WAL/SHM)
      |                           |
      | legacy /api/*             v
      +---------------->  kafsiem-api  --->  Caddy + web UI
```

The collector ingests OSINT sources and, when enabled, consumes Kafka traffic.
It is the sole writer for observed operational state. `kafsiem-api` serves
read-only analyst resources and enqueues replay requests. See
[Architecture](docs/architecture.md).

## Operating modes

| Product name | `UI_MODE` | Purpose |
|--------------|-----------|---------|
| OSINT | `OSINT` | Globe-first intelligence, alert feeds, attack-relation clusters |
| Operations | `AGENTOPS` | Kafka agent flow tracking and ontology desk |
| Fusion | `HYBRID` | Operations plus heuristic OSINT correlation |

Set mode via `UI_MODE` and mounted policy files. User-facing copy uses OSINT,
Operations, and Fusion. Runtime env values stay `AGENTOPS` and `HYBRID` for
compatibility.

## Quick start (Docker)

```bash
cp .env.example .env
docker compose up --build
```

UI: `http://localhost:8080`

Make targets for local HTTP dev:

```bash
make dev-start
make dev-stop
make dev-restart
make dev-logs
```

## Remote install

```bash
wget -qO- https://raw.githubusercontent.com/scalytics/kafSIEM/main/deploy/install.sh | bash
```

The installer checks Docker, clones or updates the repo, prompts for profile
(OSINT, Operations, or Fusion), site address, and profile-specific settings,
then starts GHCR images.

## API example

```bash
curl -s http://localhost:8080/api/v1/ontology/packs | jq .
curl -s http://localhost:8080/api/v1/flows | jq .
curl -s http://localhost:8080/api/v1/entities/platform/auv-7 | jq .
```

Full contract: [api/openapi.yaml](api/openapi.yaml). Client notes:
[docs/api-clients.md](docs/api-clients.md).

## Stack context

| Component | Role |
|-----------|------|
| **[KafScale](https://github.com/KafScale/platform)** | External Kafka transport. kafSIEM consumes from it. |
| **[KafClaw](https://github.com/KafClaw/KafClaw)** | External agent plane. Emits group envelopes on Kafka topics. |
| **kafSIEM** | Observer, graph store, API, UI, packs (this repository). |

kafSIEM complements enterprise intelligence platforms. It exposes typed entity
and graph APIs, pack-defined ontology, and provenance back to source records.

Related projects from the same team: [KafGraph](https://github.com/scalytics/KafGraph)
(shared knowledge graph for agents) and [kshark-core](https://github.com/scalytics/kshark-core)
(Kafka connectivity diagnostics, an example of how agents can use our tools to debug
Kafka clusters). All projects are listed at
[scalytics.io/open-source](https://www.scalytics.io/open-source). For a governed,
supported platform built on this open core, see [Lascaris](https://www.scalytics.io/lascaris).
Product page: [scalytics.io/kafsiem](https://www.scalytics.io/kafsiem).

## Documentation

| Topic | Document |
|-------|----------|
| Full doc index | [docs/README.md](docs/README.md) |
| Architecture | [docs/architecture.md](docs/architecture.md) |
| Operations / Docker / VM | [docs/operations.md](docs/operations.md) |
| Operations Kafka config | [docs/agentops-operator-guide.md](docs/agentops-operator-guide.md) |
| API clients | [docs/api-clients.md](docs/api-clients.md) |
| Drones pack | [docs/packs/drones.md](docs/packs/drones.md) |
| SCADA pack | [docs/packs/scada.md](docs/packs/scada.md) |
| OSINT user guide | [docs/userguide.md](docs/userguide.md) |
| Advanced env tuning | [docs/advanced-config.md](docs/advanced-config.md) |
| Minimal env template | [.env.example](.env.example) |

## Development

```bash
make check
make ci
make docker-build
```

Node `26.7.0` and npm `11.19.0` are pinned in `package.json`, `.nvmrc`, and
`.node-version`. The Go collector powers scheduled feed refresh, Docker
runtime, and local collection commands.

Local workflows (`make help` for the full list):

| Goal | Command |
|------|---------|
| OSINT relations demo (fixtures) | `make demo-osint` |
| Full local stack (live collector + API) | `make dev-start` → `http://localhost:8080` |
| Go collector tests | `make go-test` |
| Full CI gate | `make ci` or `make commit-check` |

Browse corroborated clusters: header **Relations** → filter by attack type
(cyber, terror, maritime, conflict). See
[docs/userguide.md](docs/userguide.md#attack-relations-clusters).

## License

Apache-2.0. See [LICENSE](LICENSE).