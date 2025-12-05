<div align="center">

# Media Gateway

**A Global Cross-Platform TV Discovery System**

[![Rust](https://img.shields.io/badge/rust-100%25-orange.svg)](https://www.rust-lang.org/)
[![Repositories](https://img.shields.io/badge/repos-51%20micro--repos-blue.svg)](#multi-repository-architecture)
[![hackathon-tv5](https://img.shields.io/badge/hackathon--tv5-Agentics%20Foundation-00A67E.svg)](https://github.com/agenticsorg/hackathon-tv5)
[![E2B](https://img.shields.io/badge/E2B-Sandboxed%20Agents-FF6B35.svg)](https://e2b.dev)
[![GCP](https://img.shields.io/badge/GCP-GKE%20%7C%20Cloud%20Run-4285F4.svg)](https://cloud.google.com/)
[![SONA](https://img.shields.io/badge/SONA-Intelligence%20Engine-purple.svg)](#sona-intelligence-engine)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Architecture](https://img.shields.io/badge/architecture-4--layer-green.svg)](#architecture)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

*Eliminate the 45 minutes people waste daily deciding what to watch.*

[Architecture](#architecture) · [51 Repositories](#multi-repository-architecture) · [hackathon-tv5](#hackathon-tv5-integration) · [E2B Sandboxes](#e2b-sandbox-integration) · [SONA Intelligence](#sona-intelligence-engine) · [GCP Deployment](#google-cloud-deployment) · [Documentation](#documentation)

</div>

---

## Overview

Media Gateway is a comprehensive architecture blueprint for a unified TV content discovery platform. It aggregates streaming services (Netflix, Prime Video, Disney+, Hulu, Apple TV+, YouTube, Crave, HBO Max, and more) into a single, intelligent discovery experience accessible via CLI, web, mobile, and smart TV applications.

### Key Features

- **51 Micro-Repositories** — Independent versioning, separate CI/CD, parallel development across teams
- **Unified Content Discovery** — Search across 10+ streaming platforms simultaneously
- **Intelligent Recommendations** — Hybrid engine combining collaborative filtering, content-based analysis, and Graph Neural Networks
- **hackathon-tv5 Integration** — Built on Agentics Foundation toolkit with ARW specification and 17+ tools
- **ARW Protocol** — Agent-Ready Web with 85% token reduction for efficient AI-agent interaction
- **E2B Sandboxed Agents** — Firecracker microVM isolation for safe AI-generated code execution
- **SONA Intelligence Engine** — Self-Optimizing Neural Architecture with runtime adaptation via Two-Tier LoRA
- **39 Attention Mechanisms** — Dynamic selection for optimal query processing (Graph, Hyperbolic, Transformer)
- **Cross-Device Sync** — Real-time watchlist and progress synchronization via PubNub
- **Privacy-First Design** — Federated learning with differential privacy for personalization
- **Rust-Native Performance** — 100% Rust implementation for reliability and speed
- **Developer CLI** — Full-featured terminal interface for platform operators (not end-users)
- **Cloud-Native Deployment** — GKE Autopilot and Cloud Run on Google Cloud Platform

---

## Architecture

Media Gateway implements a **4-layer architecture** designed for scalability, modularity, and cross-platform deployment:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     DEVELOPER / OPERATOR TOOLS                              │
│         CLI (Rust TUI) │ hackathon-tv5 CLI │ ARW Chrome Extension          │
├─────────────────────────────────────────────────────────────────────────────┤
│                       LAYER 4: END-USER EXPERIENCE                          │
│         Web App (Next.js) │ Mobile (iOS/Android) │ TV (Tizen/LG)           │
├─────────────────────────────────────────────────────────────────────────────┤
│                       LAYER 3: CONSOLIDATION                                │
│    Global Metadata │ Availability Index │ Rights Engine │ Personalization  │
├─────────────────────────────────────────────────────────────────────────────┤
│                       LAYER 2: INTELLIGENCE                                 │
│    Multi-Agent Orchestration │ Recommendation Engine │ Semantic Search     │
├─────────────────────────────────────────────────────────────────────────────┤
│                       LAYER 1: MICRO-REPOSITORIES                           │
│    MCP Connectors │ Normalizers │ Entity Resolver │ Auth │ MCP Server      │
├─────────────────────────────────────────────────────────────────────────────┤
│                            DATA LAYER                                       │
│              Ruvector (Hypergraph + Vector + GNN) │ PubNub                 │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Core Technologies

| Component | Technology | Purpose |
|-----------|------------|---------|
| **Runtime** | Rust | Primary implementation language |
| **Foundation** | [hackathon-tv5](https://github.com/agenticsorg/hackathon-tv5) | ARW spec, MCP server, 17+ tools |
| **Protocol** | ARW | Agent-Ready Web (85% token reduction) |
| **Sandboxing** | [E2B](https://e2b.dev) | Firecracker microVM agent isolation |
| **Data Engine** | Ruvector | Hypergraph, vector indexes, GNN simulation |
| **Intelligence** | SONA | Self-Optimizing Neural Architecture |
| **Personalization** | Two-Tier LoRA | Runtime adaptation without retraining |
| **Routing** | Tiny Dancer | FastGRNN semantic query routing |
| **Attention** | 39 Mechanisms | Dynamic attention selection |
| **Real-Time Sync** | PubNub | Cross-device state synchronization |
| **Orchestration** | Claude-Flow | Multi-agent AI coordination (SPARC methodology) |
| **Authentication** | OAuth2 + PKCE | Secure streaming platform authorization |
| **Service Communication** | gRPC (tonic) | Inter-service messaging |
| **Container Orchestration** | GKE Autopilot | Managed Kubernetes for microservices |
| **Serverless** | Cloud Run | Auto-scaling API gateway and webhooks |

---

## hackathon-tv5 Integration

Media Gateway is built on the [Agentics Foundation hackathon-tv5](https://github.com/agenticsorg/hackathon-tv5) toolkit, which provides the foundational infrastructure for agentic AI media discovery.

### What is hackathon-tv5?

hackathon-tv5 addresses the **"45-minute decision problem"** — the time users waste deciding what to watch due to content fragmentation across streaming platforms. It provides:

- **CLI Tool** (`npx agentics-hackathon`): Project initialization, tool installation, MCP server
- **MCP Server**: 6 core tools with STDIO and SSE transport
- **ARW Specification**: Agent-Ready Web protocol for efficient AI interaction
- **Demo Apps**: Media Discovery (Next.js) and ARW Chrome Extension
- **17+ Tools**: AI assistants, orchestration frameworks, databases

### ARW (Agent-Ready Web) Benefits

| Benefit | Impact |
|---------|--------|
| **85% Token Reduction** | Machine views instead of HTML scraping |
| **10x Faster Discovery** | Structured manifests for quick parsing |
| **OAuth-Enforced Actions** | Secure platform transactions |
| **AI-* Headers** | Full agent traffic observability |

### Tool Ecosystem

hackathon-tv5 provides 17+ tools across six categories:

| Category | Tools | Integration |
|----------|-------|-------------|
| **AI Assistants** | Claude Code CLI, Gemini CLI | Development |
| **Orchestration** | Claude Flow (101 tools), Agentic Flow (66 agents) | Layer-2 Intelligence |
| **Cloud** | Google Cloud CLI, Vertex AI SDK | Infrastructure |
| **Databases** | RuVector, AgentDB | Data Layer |
| **Frameworks** | LionPride, OpenAI Agents SDK | Alternative agents |
| **Advanced** | SPARC 2.0, Strange Loops | Reasoning patterns |

### Quick Start with hackathon-tv5

```bash
# Initialize project with hackathon-tv5
npx agentics-hackathon init
# Select track: Entertainment Discovery

# Install recommended tools
npx agentics-hackathon tools --install claude-flow
npx agentics-hackathon tools --install ruvector

# Start MCP servers
npx agentics-hackathon mcp sse --port 3000

# Check status
npx agentics-hackathon status
```

### Claude Desktop Integration

```json
{
  "mcpServers": {
    "agentics-hackathon": {
      "command": "npx",
      "args": ["agentics-hackathon", "mcp"]
    },
    "media-gateway": {
      "command": "./media-gateway",
      "args": ["mcp", "--transport", "stdio"]
    }
  }
}
```

---

## Google Cloud Deployment

Media Gateway is designed for production deployment on **Google Cloud Platform**, leveraging GKE Autopilot for container orchestration and Cloud Run for serverless components.

### Infrastructure Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         GOOGLE CLOUD PLATFORM                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │              GLOBAL LOAD BALANCER + CLOUD ARMOR                      │   │
│  │                    (WAF + DDoS Protection)                           │   │
│  └──────────────────────────────┬──────────────────────────────────────┘   │
│                                 │                                           │
│  ┌──────────────────────────────┼──────────────────────────────────────┐   │
│  │                              ▼                                       │   │
│  │  ┌────────────────┐    ┌────────────────┐    ┌────────────────┐     │   │
│  │  │   Cloud Run    │    │  GKE Autopilot │    │   Cloud Run    │     │   │
│  │  │  (API Gateway) │    │ (15+ Services) │    │   (Webhooks)   │     │   │
│  │  └────────────────┘    └────────────────┘    └────────────────┘     │   │
│  │                              │                                       │   │
│  │         ISTIO SERVICE MESH + WORKLOAD IDENTITY                      │   │
│  └──────────────────────────────┼──────────────────────────────────────┘   │
│                                 │                                           │
│  ┌──────────────────────────────┼──────────────────────────────────────┐   │
│  │                              ▼                                       │   │
│  │  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐         │   │
│  │  │   Cloud SQL    │  │  Memorystore   │  │    Pub/Sub     │         │   │
│  │  │  PostgreSQL 15 │  │    Valkey      │  │    Topics      │         │   │
│  │  └────────────────┘  └────────────────┘  └────────────────┘         │   │
│  │                                                                      │   │
│  │              PRIVATE VPC + PRIVATE SERVICE ACCESS                   │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  OBSERVABILITY: Cloud Logging │ Cloud Trace │ Cloud Monitoring       │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  CI/CD: Cloud Build │ Artifact Registry │ Cloud Deploy │ Secret Mgr │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### GCP Services

| Service | Purpose | Configuration |
|---------|---------|---------------|
| **GKE Autopilot** | Container orchestration for 15+ microservices | Regional, Workload Identity enabled |
| **Cloud Run** | Stateless API gateway and webhook handlers | Auto-scaling, VPC connector |
| **Cloud SQL** | PostgreSQL 15 with HA | Private IP, Point-in-time recovery |
| **Memorystore** | Redis-compatible caching (Valkey) | 10GB, Standard HA, Read replicas |
| **Pub/Sub** | Event streaming between services | Schema enforcement, Dead letter queues |
| **Cloud Armor** | WAF and DDoS protection | Adaptive protection, Rate limiting |
| **Secret Manager** | Credential management | Automatic rotation, IAM integration |
| **Artifact Registry** | Container image storage | Vulnerability scanning |
| **Cloud Build** | CI/CD pipeline | Multi-stage Rust builds |
| **Cloud Deploy** | Kubernetes deployment automation | Progressive rollouts |

### Deployment Patterns

**GKE Autopilot** hosts the core microservices:
- Layer 1: Ingestion, Auth, Sync Engine, Entity Resolver
- Layer 2: Recommendation Engine, Agent Orchestrator, Semantic Search
- Layer 3: Metadata Fabric, Availability Index, Rights Engine

**Cloud Run** hosts stateless components:
- API Gateway (public-facing)
- Webhook Handlers (Pub/Sub push)
- Batch Jobs (scheduled via Cloud Scheduler)

### Infrastructure as Code

All GCP resources are provisioned via **Terraform** modules:

```
terraform/
├── environments/
│   ├── dev/
│   ├── staging/
│   └── prod/
└── modules/
    ├── gke/           # GKE Autopilot cluster
    ├── cloudrun/      # Cloud Run services
    ├── database/      # Cloud SQL PostgreSQL
    ├── cache/         # Memorystore Valkey
    ├── pubsub/        # Pub/Sub topics and subscriptions
    ├── security/      # Workload Identity, Cloud Armor, Secrets
    ├── network/       # VPC, Subnets, Firewall
    └── observability/ # Logging, Monitoring, Alerting
```

### Quick Deploy

```bash
# Initialize Terraform
cd terraform/environments/dev
terraform init

# Plan and apply
terraform plan -out=tfplan
terraform apply tfplan

# Get GKE credentials
gcloud container clusters get-credentials media-gateway-cluster --region us-central1

# Deploy services
kubectl apply -f k8s/namespaces.yaml
kubectl apply -f k8s/layer1/
kubectl apply -f k8s/layer2/
kubectl apply -f k8s/layer3/

# Deploy Cloud Run
gcloud run deploy api-gateway \
  --image us-central1-docker.pkg.dev/PROJECT_ID/media-gateway/api-gateway:latest \
  --region us-central1
```

### Estimated Costs

| Component | Monthly Estimate |
|-----------|-----------------|
| GKE Autopilot (15 services) | $800-1,200 |
| Cloud Run (API + Webhooks) | $200-400 |
| Cloud SQL (HA, 4 vCPU) | $400-500 |
| Memorystore (10GB HA) | $300-350 |
| Pub/Sub (10M msgs) | $50-100 |
| SONA Intelligence Services | $300-500 |
| E2B Sandboxes | $250-450 |
| Other (Armor, Logging, etc.) | $100-150 |
| **Total** | **$2,400-3,650** |

---

## E2B Sandbox Integration

Media Gateway uses [E2B](https://e2b.dev) to provide secure, isolated sandbox environments for AI agent code execution. This ensures LLM-generated code runs safely without compromising production infrastructure.

### Why E2B?

| Challenge | E2B Solution |
|-----------|--------------|
| **Untrusted Code** | Firecracker microVM isolation |
| **Resource Limits** | Configurable CPU/memory |
| **Network Security** | Sandboxed with controlled egress |
| **Fast Startup** | ~150ms boot time |
| **Scale** | Thousands concurrent sandboxes |

### Sandbox Types

| Type | Use Case |
|------|----------|
| **Code Interpreter** | General agent code execution (Python/JS) |
| **Data Analysis** | Recommendation computations |
| **Desktop/Browser** | Web scraping, metadata enrichment |
| **SONA Updater** | Safe LoRA weight updates |

### Agent Execution Flow

```
Agent Task → LLM Generates Code → E2B Sandbox (isolated) → Results → Agent Response
```

All agent-generated code executes in isolated Firecracker microVMs, never on production infrastructure.

---

## SONA Intelligence Engine

Media Gateway integrates the **SONA (Self-Optimizing Neural Architecture)** from Ruvector to provide runtime adaptation and personalization without model retraining.

### SONA Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       SONA INTELLIGENCE ENGINE                               │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  SEMANTIC ROUTING (Tiny Dancer)                                       │  │
│  │  FastGRNN query classification → Optimal handler dispatch             │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                    │                                         │
│  ┌─────────────────────────────────▼─────────────────────────────────────┐  │
│  │  ATTENTION SELECTION (39 Mechanisms)                                   │  │
│  │  Core │ Graph │ Specialized │ Hyperbolic                              │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                    │                                         │
│  ┌─────────────────────────────────▼─────────────────────────────────────┐  │
│  │  RUNTIME ADAPTATION                                                    │  │
│  │  Two-Tier LoRA │ EWC++ (Anti-forgetting) │ ReasoningBank              │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Core SONA Components

| Component | Purpose | Key Benefit |
|-----------|---------|-------------|
| **Two-Tier LoRA** | Low-rank adaptation for user personalization | ~10KB per user, no retraining required |
| **Tiny Dancer** | FastGRNN-based semantic query routing | <5ms routing latency |
| **39 Attention Mechanisms** | Dynamic attention for different query types | Optimal processing for graphs, text, hierarchies |
| **ReasoningBank** | Persistent storage of reasoning patterns | Learn once, reuse across similar queries |
| **EWC++** | Elastic Weight Consolidation | Prevents catastrophic forgetting during updates |

### Attention Mechanism Categories

| Category | Count | Use Cases |
|----------|-------|-----------|
| **Core** | 12 | MultiHead, Flash, Linear, RoPE, ALiBi for standard transformers |
| **Graph** | 10 | GraphRoPE, GAT, GCN for content relationship queries |
| **Specialized** | 9 | Sparse, Cross, Longformer for complex queries |
| **Hyperbolic** | 8 | expMap, mobiusAddition for hierarchical content taxonomies |

### How SONA Improves Recommendations

1. **Query Understanding** — Tiny Dancer classifies user intent in <5ms
2. **Personalization** — LoRA adapters apply user preferences without retraining
3. **Pattern Reuse** — ReasoningBank caches successful reasoning for similar queries
4. **Continuous Learning** — EWC++ enables safe incremental updates

---

## Documentation

### Research & Architecture

| Document | Description |
|----------|-------------|
| [`FINAL_ARCHITECTURE_BLUEPRINT.md`](research/FINAL_ARCHITECTURE_BLUEPRINT.md) | Complete 20-section system architecture |
| [`SONA_INTEGRATION_SPECIFICATION.md`](research/SONA_INTEGRATION_SPECIFICATION.md) | SONA Intelligence Engine integration guide |
| [`HACKATHON_TV5_INTEGRATION.md`](research/HACKATHON_TV5_INTEGRATION.md) | hackathon-tv5 toolkit integration specification |
| [`E2B_SANDBOX_INTEGRATION.md`](research/E2B_SANDBOX_INTEGRATION.md) | E2B sandbox integration for secure agent execution |
| [`ARCHITECTURE_BLUEPRINT.md`](research/ARCHITECTURE_BLUEPRINT.md) | Core architecture design and patterns |
| [`streaming-platform-research.md`](research/streaming-platform-research.md) | API analysis for 10+ streaming platforms |
| [`GCP_DEPLOYMENT_ARCHITECTURE.md`](research/GCP_DEPLOYMENT_ARCHITECTURE.md) | Google Cloud deployment guide with Terraform |

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

This research repository contains architecture documentation for the Media Gateway platform:

```
media-gateway-hackathon-research/
├── research/                    # Architecture documentation
│   ├── FINAL_ARCHITECTURE_BLUEPRINT.md
│   ├── SONA_INTEGRATION_SPECIFICATION.md
│   ├── HACKATHON_TV5_INTEGRATION.md
│   ├── E2B_SANDBOX_INTEGRATION.md
│   ├── GCP_DEPLOYMENT_ARCHITECTURE.md
│   └── ...
├── .gitignore
├── LICENSE
└── README.md
```

### Multi-Repository Architecture

The Media Gateway platform is implemented as **51 independent micro-repositories** rather than a monorepo. This enables independent versioning, separate CI/CD pipelines, and parallel development.

#### Repository Organization

```
github.com/globalbusinessadvisors/
│
├── FOUNDATION (3 repos)
│   ├── mg-proto                 # gRPC/Protobuf definitions
│   ├── mg-sdk-rust              # Core Rust SDK
│   └── mg-config                # Shared configuration
│
├── LAYER 1: INGESTION (14 repos)
│   ├── mg-ingestion-core        # MCP connector framework
│   ├── mg-connector-aggregator  # JustWatch/Watchmode/StreamingAvailability
│   ├── mg-connector-netflix     # Netflix via aggregators
│   ├── mg-connector-prime       # Prime Video
│   ├── mg-connector-disney      # Disney+
│   ├── mg-connector-*           # ... (10 platform connectors)
│   ├── mg-metadata-normalizer   # Platform → Unified schema
│   └── mg-entity-resolver       # EIDR/IMDb/TMDb deduplication
│
├── LAYER 1: IDENTITY (3 repos)
│   ├── mg-auth-service          # OAuth2/PKCE + Device Grant
│   ├── mg-token-service         # JWT management
│   └── mg-session-service       # Cross-device sessions
│
├── LAYER 1: DEVICE (3 repos)
│   ├── mg-device-gateway        # WebSocket hub + PubNub
│   ├── mg-sync-engine           # CRDT-based sync
│   └── mg-device-adapters       # Tizen, webOS, Roku, Chromecast
│
├── LAYER 2: INTELLIGENCE (8 repos)
│   ├── mg-agent-orchestrator    # Claude-Flow + 9 specialized agents
│   ├── mg-recommendation-engine # Hybrid GNN recommendations
│   ├── mg-semantic-search       # Vector + Graph search
│   ├── mg-embedding-service     # Content embeddings
│   ├── mg-sona-client           # SONA intelligence (LoRA, EWC++, 39 attention)
│   ├── mg-reasoning-bank        # Reasoning pattern storage
│   ├── mg-agent-db              # Agent context storage (Redis)
│   └── mg-e2b-client            # E2B sandbox client
│
├── LAYER 3: CONSOLIDATION (5 repos)
│   ├── mg-metadata-fabric       # Global unified metadata
│   ├── mg-availability-index    # Cross-platform availability
│   ├── mg-rights-engine         # Rights aggregation
│   ├── mg-personalization-engine # Privacy-safe personalization
│   └── mg-search-api            # Unified search API
│
├── LAYER 4: APPLICATIONS (6 repos)
│   ├── mg-cli                   # Developer/Operator CLI (Rust TUI)
│   ├── mg-web-app               # Next.js web application
│   ├── mg-mobile-ios            # iOS application (Swift)
│   ├── mg-mobile-android        # Android application (Kotlin)
│   ├── mg-tv-tizen              # Samsung Tizen app
│   └── mg-tv-webos              # LG webOS app
│
├── DATA LAYER (2 repos)
│   ├── mg-ruvector-client       # Ruvector Rust client
│   └── mg-pubnub-client         # PubNub Rust client wrapper
│
├── INFRASTRUCTURE (4 repos)
│   ├── mg-terraform             # Terraform modules (GKE, Cloud Run, etc.)
│   ├── mg-helm-charts           # Helm charts
│   ├── mg-docker                # Dockerfiles and compose
│   └── mg-ci-templates          # Shared CI/CD workflows
│
├── TESTING (2 repos)
│   ├── mg-test-utils            # Shared test utilities
│   └── mg-simulator             # Simulation & mock APIs
│
└── DOCUMENTATION (1 repo)
    └── mg-docs                  # Documentation site (Docusaurus)
```

#### Repository Count by Category

| Category | Repos | Purpose |
|----------|-------|---------|
| Foundation | 3 | Proto, SDK, Config |
| Layer 1 - Ingestion | 14 | MCP connectors |
| Layer 1 - Identity | 3 | Auth, tokens |
| Layer 1 - Device | 3 | Sync, adapters |
| Layer 2 - Intelligence | 8 | AI/ML services |
| Layer 3 - Consolidation | 5 | Aggregation |
| Layer 4 - Applications | 6 | End-user apps |
| Data Layer | 2 | Data clients |
| Infrastructure | 4 | DevOps |
| Testing | 2 | QA |
| Documentation | 1 | Docs |
| **Total** | **51** | |

#### Dependency Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         REPOSITORY DEPENDENCY GRAPH                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  BUILD ORDER (Foundation → Services → Applications)                         │
│                                                                              │
│  ┌─────────────┐                                                            │
│  │  mg-proto   │ ◄── Build first (Protobuf schemas)                         │
│  └──────┬──────┘                                                            │
│         │                                                                    │
│  ┌──────▼──────┐                                                            │
│  │ mg-sdk-rust │ ◄── Core types, traits, utilities                          │
│  └──────┬──────┘                                                            │
│         │                                                                    │
│  ┌──────┴──────────────────────────────────────┐                            │
│  │                                              │                            │
│  ▼                                              ▼                            │
│  ┌─────────────────┐              ┌─────────────────┐                       │
│  │mg-ingestion-core│              │mg-ruvector-client│                       │
│  └────────┬────────┘              └────────┬────────┘                       │
│           │                                │                                 │
│  ┌────────┴────────┐              ┌────────▼────────┐                       │
│  │ mg-connector-*  │              │  mg-sona-client │                       │
│  │ (10+ platforms) │              └────────┬────────┘                       │
│  └────────┬────────┘                       │                                 │
│           │                                │                                 │
│           └────────────┬───────────────────┘                                │
│                        │                                                     │
│               ┌────────▼────────┐                                           │
│               │mg-agent-        │                                           │
│               │orchestrator     │ ◄── 9 specialized agents                  │
│               └────────┬────────┘                                           │
│                        │                                                     │
│  ┌─────────────────────┼─────────────────────┐                              │
│  │                     │                     │                              │
│  ▼                     ▼                     ▼                              │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐                        │
│  │mg-recommend- │ │mg-semantic-  │ │mg-metadata-  │                        │
│  │ation-engine  │ │search        │ │fabric        │                        │
│  └──────┬───────┘ └──────┬───────┘ └──────┬───────┘                        │
│         │                │                │                                 │
│         └────────────────┼────────────────┘                                │
│                          │                                                  │
│                 ┌────────▼────────┐                                        │
│                 │  mg-search-api  │ ◄── Unified API gateway                │
│                 └────────┬────────┘                                        │
│                          │                                                  │
│  ┌───────────────────────┼───────────────────────┐                         │
│  │                       │                       │                         │
│  ▼                       ▼                       ▼                         │
│  ┌──────────┐    ┌──────────────┐    ┌──────────────┐                     │
│  │  mg-cli  │    │  mg-web-app  │    │ mg-tv-tizen  │                     │
│  └──────────┘    └──────────────┘    └──────────────┘                     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### Inter-Repository Communication

| Method | Technology | Use Case |
|--------|------------|----------|
| **Sync RPC** | gRPC (tonic) | Service-to-service calls |
| **Async Events** | Google Pub/Sub | Event streaming, notifications |
| **Real-time** | PubNub | Device sync, presence |
| **Shared Types** | mg-proto crate | Schema definitions |
| **Client SDK** | mg-sdk-rust crate | Common utilities |

#### Versioning Strategy

All repositories follow **semantic versioning** and publish to crates.io (Rust) or npm (TypeScript):

```toml
# Example Cargo.toml for a service repository
[package]
name = "mg-recommendation-engine"
version = "0.1.0"

[dependencies]
mg-sdk = "1.0"              # Core SDK
mg-proto = "1.0"            # Protobuf types
mg-ruvector-client = "0.5"  # Data layer
mg-sona-client = "0.3"      # Intelligence
```

#### Build Order for New Developers

```bash
# 1. Clone and build foundation repos first
git clone https://github.com/globalbusinessadvisors/mg-proto.git
cd mg-proto && cargo build && cargo publish --dry-run

git clone https://github.com/globalbusinessadvisors/mg-sdk-rust.git
cd mg-sdk-rust && cargo build && cargo publish --dry-run

# 2. Build data layer clients
git clone https://github.com/globalbusinessadvisors/mg-ruvector-client.git
git clone https://github.com/globalbusinessadvisors/mg-pubnub-client.git

# 3. Build your target service (e.g., recommendation engine)
git clone https://github.com/globalbusinessadvisors/mg-recommendation-engine.git
cd mg-recommendation-engine && cargo build

# 4. Or use the dev environment with all services
git clone https://github.com/globalbusinessadvisors/mg-docker.git
cd mg-docker && docker-compose -f docker-compose.dev.yml up
```

#### Repository Links

| Repository | Purpose | Status |
|------------|---------|--------|
| `mg-proto` | Protobuf schemas | 🔜 Planned |
| `mg-sdk-rust` | Core Rust SDK | 🔜 Planned |
| `mg-ingestion-core` | MCP connector framework | 🔜 Planned |
| `mg-agent-orchestrator` | Claude-Flow + 9 agents | 🔜 Planned |
| `mg-recommendation-engine` | Hybrid GNN recommendations | 🔜 Planned |
| `mg-search-api` | Unified search API | 🔜 Planned |
| `mg-cli` | Developer CLI | 🔜 Planned |
| `mg-web-app` | Next.js web app | 🔜 Planned |
| `mg-terraform` | GCP infrastructure | 🔜 Planned |

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

- Rust 1.75+
- Docker & Docker Compose
- Google Cloud SDK (`gcloud`)
- Terraform 1.5+
- kubectl
- Access to aggregator APIs (JustWatch, Watchmode, or Streaming Availability)

### Local Development

```bash
# Clone the repository
git clone https://github.com/globalbusinessadvisors/media-gateway.git
cd media-gateway

# Review the architecture
cat research/FINAL_ARCHITECTURE_BLUEPRINT.md

# Review GCP deployment
cat research/GCP_DEPLOYMENT_ARCHITECTURE.md
```

### GCP Setup

```bash
# Authenticate with GCP
gcloud auth login
gcloud config set project YOUR_PROJECT_ID

# Enable required APIs
gcloud services enable \
  container.googleapis.com \
  run.googleapis.com \
  sqladmin.googleapis.com \
  redis.googleapis.com \
  pubsub.googleapis.com \
  secretmanager.googleapis.com \
  cloudbuild.googleapis.com \
  artifactregistry.googleapis.com

# Initialize Terraform
cd terraform/environments/dev
terraform init
```

---

## Contributing

We welcome contributions! Please see our [Contributing Guide](CONTRIBUTING.md) for details.

### Development Process

1. Review the [Architecture Blueprint](research/FINAL_ARCHITECTURE_BLUEPRINT.md)
2. Review the [GCP Deployment Guide](research/GCP_DEPLOYMENT_ARCHITECTURE.md)
3. Check the [Implementation Roadmap](research/CLI_IMPLEMENTATION_ROADMAP.md)
4. Pick an unassigned component from the roadmap
5. Submit a PR with tests and documentation

### Code Style

- Rust: Follow `rustfmt` and `clippy` recommendations
- Terraform: Follow HashiCorp style guide
- Kubernetes: Use `kubectl diff` before applying
- Documentation: Markdown with Mermaid diagrams where applicable
- Commits: Conventional Commits format

---

## Roadmap

### Phase 1: Foundation
- [ ] Core MCP connector traits and ingestion framework
- [ ] Ruvector client library and schema
- [ ] Authentication service (OAuth2/PKCE + Device Grant)
- [ ] CLI core with TUI framework
- [ ] GCP infrastructure (Terraform modules)

### Phase 2: Intelligence
- [ ] Recommendation engine with GraphSAGE
- [ ] Semantic search and embeddings
- [ ] Multi-agent orchestration (Claude-Flow)
- [ ] GKE deployment with Istio

### Phase 3: Integration
- [ ] Cross-device sync (PubNub + CRDT)
- [ ] Platform-specific MCP connectors
- [ ] Rights validation engine
- [ ] Cloud Run API gateway

### Phase 4: Experience
- [ ] Production CLI application
- [ ] Web application (Next.js)
- [ ] Smart TV applications
- [ ] Cloud Deploy pipelines

---

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---

## Acknowledgments

- **[Agentics Foundation hackathon-tv5](https://github.com/agenticsorg/hackathon-tv5)** — ARW specification, MCP server, and tool ecosystem
- **[E2B](https://e2b.dev)** — Firecracker microVM sandboxes for secure agent code execution
- **Google Cloud Platform** — Infrastructure and managed services
- **Ruvector + SONA** — Hypergraph, vector database, and Self-Optimizing Neural Architecture
- **PubNub** — Real-time messaging infrastructure
- **Claude-Flow** — Multi-agent orchestration framework
- **JustWatch** — Streaming availability data

---

<div align="center">

**[Documentation](research/) · [GCP Guide](research/GCP_DEPLOYMENT_ARCHITECTURE.md) · [Issues](https://github.com/globalbusinessadvisors/media-gateway/issues) · [Discussions](https://github.com/globalbusinessadvisors/media-gateway/discussions)**

</div>
