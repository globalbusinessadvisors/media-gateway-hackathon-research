<div align="center">

# Media Gateway

**A Global Cross-Platform TV Discovery System**

[![Rust](https://img.shields.io/badge/rust-100%25-orange.svg)](https://www.rust-lang.org/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Architecture](https://img.shields.io/badge/architecture-4--layer-green.svg)](#architecture)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

*Eliminate the 45 minutes people waste daily deciding what to watch.*

[Architecture](#architecture) · [Documentation](#documentation) · [Getting Started](#getting-started) · [Contributing](#contributing)

</div>

---

## Overview

Media Gateway is a comprehensive architecture blueprint for a unified TV content discovery platform. It aggregates streaming services (Netflix, Prime Video, Disney+, Hulu, Apple TV+, YouTube, Crave, HBO Max, and more) into a single, intelligent discovery experience accessible via CLI, web, mobile, and smart TV applications.

### Key Features

- **Unified Content Discovery** — Search across 10+ streaming platforms simultaneously
- **Intelligent Recommendations** — Hybrid engine combining collaborative filtering, content-based analysis, and Graph Neural Networks
- **Cross-Device Sync** — Real-time watchlist and progress synchronization via PubNub
- **Privacy-First Design** — Federated learning with differential privacy for personalization
- **Rust-Native Performance** — 100% Rust implementation for reliability and speed
- **CLI-First Experience** — Full-featured terminal interface with rich TUI

---

## Architecture

Media Gateway implements a **4-layer architecture** designed for scalability, modularity, and cross-platform deployment:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       LAYER 4: END-USER EXPERIENCE                          │
│    ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐                  │
│    │   CLI    │  │   Web    │  │  Mobile  │  │    TV    │                  │
│    │(Rust TUI)│  │(Next.js) │  │(iOS/Droid)│ │(Tizen/LG)│                  │
│    └──────────┘  └──────────┘  └──────────┘  └──────────┘                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                       LAYER 3: CONSOLIDATION                                │
│    Global Metadata │ Availability Index │ Rights Engine │ Personalization  │
├─────────────────────────────────────────────────────────────────────────────┤
│                       LAYER 2: INTELLIGENCE                                 │
│    Multi-Agent Orchestration │ Recommendation Engine │ Semantic Search     │
├─────────────────────────────────────────────────────────────────────────────┤
│                       LAYER 1: MICRO-REPOSITORIES                           │
│    MCP Connectors │ Normalizers │ Entity Resolver │ Auth │ Device Sync     │
├─────────────────────────────────────────────────────────────────────────────┤
│                            DATA LAYER                                       │
│              Ruvector (Hypergraph + Vector + GNN) │ PubNub                 │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Core Technologies

| Component | Technology | Purpose |
|-----------|------------|---------|
| **Runtime** | Rust | Primary implementation language |
| **Data Engine** | Ruvector | Hypergraph, vector indexes, GNN simulation |
| **Real-Time Sync** | PubNub | Cross-device state synchronization |
| **Orchestration** | Claude-Flow | Multi-agent AI coordination (SPARC methodology) |
| **Authentication** | OAuth2 + PKCE | Secure streaming platform authorization |
| **Service Communication** | gRPC (tonic) | Inter-service messaging |

---

## Documentation

### Research & Architecture

| Document | Description |
|----------|-------------|
| [`FINAL_ARCHITECTURE_BLUEPRINT.md`](research/FINAL_ARCHITECTURE_BLUEPRINT.md) | Complete 17-section system architecture |
| [`ARCHITECTURE_BLUEPRINT.md`](research/ARCHITECTURE_BLUEPRINT.md) | Core architecture design and patterns |
| [`streaming-platform-research.md`](research/streaming-platform-research.md) | API analysis for 10+ streaming platforms |

### Component Specifications

| Document | Description |
|----------|-------------|
| [`RUVECTOR_KNOWLEDGE_GRAPH_SPEC.md`](research/RUVECTOR_KNOWLEDGE_GRAPH_SPEC.md) | Hypergraph schema and GNN architecture |
| [`RECOMMENDATION_ENGINE_SPEC.md`](research/RECOMMENDATION_ENGINE_SPEC.md) | Hybrid recommendation system design |
| [`MCP_PROTOCOL_SPECIFICATION.md`](research/MCP_PROTOCOL_SPECIFICATION.md) | Model Context Protocol connector patterns |
| [`LAYER2_MULTI_AGENT_ARCHITECTURE.md`](research/LAYER2_MULTI_AGENT_ARCHITECTURE.md) | Intelligence layer agent coordination |

### CLI & Implementation

| Document | Description |
|----------|-------------|
| [`CLI_ARCHITECTURE_SPECIFICATION.md`](research/CLI_ARCHITECTURE_SPECIFICATION.md) | Terminal interface design |
| [`CLI_RUST_IMPLEMENTATION_EXAMPLES.md`](research/CLI_RUST_IMPLEMENTATION_EXAMPLES.md) | Rust code examples and patterns |
| [`CLI_IMPLEMENTATION_ROADMAP.md`](research/CLI_IMPLEMENTATION_ROADMAP.md) | Development phases and milestones |
| [`AGENT_IMPLEMENTATION_GUIDE.md`](research/AGENT_IMPLEMENTATION_GUIDE.md) | Multi-agent system implementation |

---

## Repository Structure

```
media-gateway/
├── research/                    # Architecture documentation
│   ├── FINAL_ARCHITECTURE_BLUEPRINT.md
│   ├── ARCHITECTURE_BLUEPRINT.md
│   ├── streaming-platform-research.md
│   ├── RUVECTOR_KNOWLEDGE_GRAPH_SPEC.md
│   ├── RECOMMENDATION_ENGINE_SPEC.md
│   ├── MCP_PROTOCOL_SPECIFICATION.md
│   ├── CLI_ARCHITECTURE_SPECIFICATION.md
│   └── ...
├── .gitignore
├── LICENSE
└── README.md
```

### Planned Repository Layout

The full implementation will follow a multi-repository structure:

```
tv-discovery/
├── layer-1/
│   ├── ingestion/               # MCP connectors for each platform
│   ├── processing/              # Normalizers, entity resolver, semantic tagger
│   ├── validation/              # Rights validator, trust scorer
│   ├── identity/                # Auth service (OAuth2/PKCE, Device Grant)
│   ├── device/                  # Device gateway, sync engine (CRDT)
│   ├── cli/                     # CLI core and unified application
│   └── simulation/              # Mock APIs and synthetic data
├── layer-2/
│   ├── agent-orchestrator/      # Claude-Flow integration
│   ├── recommendation-engine/   # Hybrid GNN recommendations
│   ├── semantic-search/         # Vector + graph search
│   └── memory/                  # AgentDB, ReasoningBank
├── layer-3/
│   ├── metadata-fabric/         # Global unified metadata
│   ├── availability-index/      # Cross-platform availability
│   ├── rights-engine/           # Rights aggregation
│   └── personalization-engine/  # Federated learning
├── layer-4/
│   ├── cli-app/                 # Production CLI
│   ├── web-app/                 # Next.js web client
│   └── tv-apps/                 # Samsung Tizen, LG webOS, Roku
└── infrastructure/
    ├── proto/                   # gRPC definitions
    ├── sdk-rust/                # Rust SDK
    └── deploy/                  # Kubernetes, Helm, Terraform
```

---

## Key Concepts

### Ruvector Hypergraph

The system uses Ruvector as its core data engine, combining:

- **Hypergraph Storage** — Nodes (Content, Person, User, Genre, Platform, Region) with typed edges
- **Vector Indexes** — HNSW indexes for 768-dim content embeddings (semantic search)
- **GNN Simulation** — 3-layer GraphSAGE for recommendation generation

### Authentication Flows

Two authentication mechanisms support all device types:

1. **OAuth2 + PKCE** — For web and mobile applications
2. **Device Authorization Grant (RFC 8628)** — For TVs and CLI

### Cross-Device Synchronization

PubNub channel topology enables real-time sync:

- `user.{userId}.sync` — Watchlist and preference updates
- `user.{userId}.devices` — Device presence and coordination
- `global.catalog.*` — Content availability changes
- `region.{code}.*` — Regional content updates

### Privacy-Safe Personalization

Three-tier data architecture:

1. **On-Device** — Detailed viewing history (never leaves device)
2. **Federated** — Model updates aggregated with differential privacy
3. **Server** — Anonymized aggregate patterns only

---

## Getting Started

> **Note:** This repository currently contains architecture documentation and research. Implementation is in active development.

### Prerequisites

- Rust 1.75+ (for implementation)
- Docker & Docker Compose (for local development)
- Access to aggregator APIs (JustWatch, Watchmode, or Streaming Availability)

### Quick Start

```bash
# Clone the repository
git clone https://github.com/globalbusinessadvisors/media-gateway.git
cd media-gateway

# Review the architecture
cat research/FINAL_ARCHITECTURE_BLUEPRINT.md

# Start with the CLI specification
cat research/CLI_ARCHITECTURE_SPECIFICATION.md
```

---

## Contributing

We welcome contributions! Please see our [Contributing Guide](CONTRIBUTING.md) for details.

### Development Process

1. Review the [Architecture Blueprint](research/FINAL_ARCHITECTURE_BLUEPRINT.md)
2. Check the [Implementation Roadmap](research/CLI_IMPLEMENTATION_ROADMAP.md)
3. Pick an unassigned component from the roadmap
4. Submit a PR with tests and documentation

### Code Style

- Rust: Follow `rustfmt` and `clippy` recommendations
- Documentation: Markdown with Mermaid diagrams where applicable
- Commits: Conventional Commits format

---

## Roadmap

### Phase 1: Foundation
- [ ] Core MCP connector traits and ingestion framework
- [ ] Ruvector client library and schema
- [ ] Authentication service (OAuth2/PKCE + Device Grant)
- [ ] CLI core with TUI framework

### Phase 2: Intelligence
- [ ] Recommendation engine with GraphSAGE
- [ ] Semantic search and embeddings
- [ ] Multi-agent orchestration (Claude-Flow)

### Phase 3: Integration
- [ ] Cross-device sync (PubNub + CRDT)
- [ ] Platform-specific MCP connectors
- [ ] Rights validation engine

### Phase 4: Experience
- [ ] Production CLI application
- [ ] Web application (Next.js)
- [ ] Smart TV applications

---

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---

## Acknowledgments

- **Ruvector** — Hypergraph and vector database engine
- **PubNub** — Real-time messaging infrastructure
- **Claude-Flow** — Multi-agent orchestration framework
- **JustWatch** — Streaming availability data

---

<div align="center">

**[Documentation](research/) · [Issues](https://github.com/globalbusinessadvisors/media-gateway/issues) · [Discussions](https://github.com/globalbusinessadvisors/media-gateway/discussions)**

</div>
