# Global TV Discovery System - Final Architecture Blueprint

## Executive Summary

This document presents the complete architecture blueprint for a **global, cross-platform TV discovery system** implemented entirely in **Rust**, operable through a **unified CLI**, and deployable across TVs, smartphones, and consumer devices. The system allows users to securely authenticate with streaming platforms via **OAuth2/PKCE** and **Device Authorization Grant** flows, with all platform integration occurring through **custom MCP connectors**.

**Core Technologies:**
- **Ruvector + SONA**: Hypergraph, vector, GNN, and Self-Optimizing Neural Architecture
- **E2B Sandboxes**: Secure isolated execution for AI-generated code (Firecracker microVMs)
- **hackathon-tv5**: ARW specification, MCP server foundation, 17+ integrated tools
- **PubNub**: Real-time cross-device synchronization
- **Claude-Flow**: Multi-agent orchestration with SPARC methodology
- **Rust**: Primary implementation language (100%)
- **Google Cloud Platform**: Production infrastructure (GKE Autopilot, Cloud Run, Cloud SQL)

**SONA Intelligence Capabilities:**
- **Runtime Adaptation**: Two-tier LoRA personalization without retraining
- **Tiny Dancer**: FastGRNN semantic routing for optimal query dispatch
- **39 Attention Mechanisms**: Dynamic selection for graphs, transformers, hyperbolic spaces
- **ReasoningBank**: Persistent storage of learned reasoning patterns
- **EWC++**: Elastic Weight Consolidation prevents catastrophic forgetting

---

## Table of Contents

1. [Multi-Layer Architecture Overview](#1-multi-layer-architecture-overview)
2. [Layer-1: Micro-Repository Ecosystem](#2-layer-1-micro-repository-ecosystem)
3. [Layer-2: Intelligence Layer](#3-layer-2-intelligence-layer)
4. [Layer-3: Consolidation Layer](#4-layer-3-consolidation-layer)
5. [Layer-4: End-User Experience](#5-layer-4-end-user-experience)
6. [Ruvector Hypergraph Schema](#6-ruvector-hypergraph-schema)
7. [PubNub Channel Topology](#7-pubnub-channel-topology)
8. [Authentication Architecture](#8-authentication-architecture)
9. [Multi-Repo Dependency Graph](#9-multi-repo-dependency-graph)
10. [Cross-Kingdom Data Strategy](#10-cross-kingdom-data-strategy)
11. [Trust-Scoring Approach](#11-trust-scoring-approach)
12. [Privacy-Safe Personalization](#12-privacy-safe-personalization)
13. [Multi-Agent Coordination Plan](#13-multi-agent-coordination-plan)
14. [Cross-Device Flow](#14-cross-device-flow)
15. [CLI Structure](#15-cli-structure)
16. [Integration with hackathon-tv5](#16-integration-with-hackathon-tv5)
17. [Google Cloud Platform Deployment](#17-google-cloud-platform-deployment)
18. [SONA Intelligence Engine Integration](#18-sona-intelligence-engine-integration)
19. [E2B Sandbox Integration](#19-e2b-sandbox-integration)
20. [Implementation Roadmap](#20-implementation-roadmap)

---

## 1. Multi-Layer Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                      DEVELOPER / OPERATOR TOOLS                                  │
│  ┌─────────────────────┐  ┌─────────────────────┐  ┌─────────────────────┐     │
│  │   Media Gateway CLI │  │  hackathon-tv5 CLI  │  │   ARW Chrome Ext    │     │
│  │     (Rust TUI)      │  │  (npx agentics)     │  │    (Inspector)      │     │
│  └─────────────────────┘  └─────────────────────┘  └─────────────────────┘     │
└───────────────────────────────────┬─────────────────────────────────────────────┘
                                    │
┌───────────────────────────────────▼─────────────────────────────────────────────┐
│                           LAYER 4: END-USER EXPERIENCE                           │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐                 │
│  │     Web App     │  │   Mobile Apps   │  │     TV Apps     │                 │
│  │    (Next.js)    │  │  (iOS/Android)  │  │  (Samsung/LG)   │                 │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘                 │
└───────────────────────────────────┬─────────────────────────────────────────────┘
                                    │
┌───────────────────────────────────▼─────────────────────────────────────────────┐
│                        LAYER 3: CONSOLIDATION LAYER                             │
│  ┌───────────────────┐  ┌───────────────────┐  ┌───────────────────┐           │
│  │  Global Metadata  │  │  Unified Avail.   │  │  Personalization  │           │
│  │      Fabric       │  │      Index        │  │      Engine       │           │
│  └───────────────────┘  └───────────────────┘  └───────────────────┘           │
│  ┌───────────────────┐  ┌───────────────────┐                                   │
│  │   Rights Engine   │  │  Search Services  │                                   │
│  └───────────────────┘  └───────────────────┘                                   │
└───────────────────────────────────┬─────────────────────────────────────────────┘
                                    │
┌───────────────────────────────────▼─────────────────────────────────────────────┐
│                        LAYER 2: INTELLIGENCE LAYER                              │
│  ┌───────────────────┐  ┌───────────────────┐  ┌───────────────────┐           │
│  │  Multi-Agent      │  │  Recommendation   │  │  Semantic Search  │           │
│  │  Orchestration    │  │     Engine        │  │  + Embeddings     │           │
│  │  (Claude-Flow)    │  │  (GNN + Hybrid)   │  │                   │           │
│  └───────────────────┘  └───────────────────┘  └───────────────────┘           │
│  ┌───────────────────┐  ┌───────────────────┐                                   │
│  │  Cross-Platform   │  │   ReasoningBank   │                                   │
│  │   Orchestrator    │  │   + AgentDB       │                                   │
│  └───────────────────┘  └───────────────────┘                                   │
└───────────────────────────────────┬─────────────────────────────────────────────┘
                                    │
┌───────────────────────────────────▼─────────────────────────────────────────────┐
│                     LAYER 1: MICRO-REPOSITORY ECOSYSTEM                         │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐ │
│  │Ingestion│  │Metadata │  │ Entity  │  │ Rights  │  │Identity │  │  MCP    │ │
│  │ Modules │  │Normalizer│  │Resolver │  │Validator│  │  /Auth  │  │ Server  │ │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘  └─────────┘  └─────────┘ │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                                         │
│  │ Device  │  │Semantic │  │Simulation│                                         │
│  │  Sync   │  │ Tagger  │  │ Engine  │                                         │
│  └─────────┘  └─────────┘  └─────────┘                                         │
└───────────────────────────────────┬─────────────────────────────────────────────┘
                                    │
┌───────────────────────────────────▼─────────────────────────────────────────────┐
│                              DATA LAYER                                          │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │                           RUVECTOR                                        │  │
│  │  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐              │  │
│  │  │   Hypergraph   │  │ Vector Indexes │  │ GNN Simulation │              │  │
│  │  │   (Cypher)     │  │    (HNSW)      │  │  (GraphSAGE)   │              │  │
│  │  └────────────────┘  └────────────────┘  └────────────────┘              │  │
│  └──────────────────────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │                            PUBNUB                                         │  │
│  │  Real-Time Sync │ Event Streaming │ State Propagation │ Presence         │  │
│  └──────────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Layer-1: Micro-Repository Ecosystem

### 2.1 Multi-Repository Architecture

The Media Gateway platform is organized as **independent micro-repositories** rather than a monorepo. This enables:
- Independent versioning and release cycles
- Separate CI/CD pipelines per component
- Team autonomy and parallel development
- Flexible deployment (deploy only what changes)
- Clear dependency boundaries via published crates/packages

#### Repository Organization

```
github.com/globalbusinessadvisors/
│
├── FOUNDATION REPOSITORIES (Build First)
│   │
│   ├── mg-proto/                        # gRPC/Protobuf definitions
│   │   ├── proto/
│   │   │   ├── content.proto            # Content metadata schemas
│   │   │   ├── search.proto             # Search request/response
│   │   │   ├── recommendation.proto     # Recommendation types
│   │   │   ├── availability.proto       # Platform availability
│   │   │   ├── user.proto               # User profiles and prefs
│   │   │   └── agent.proto              # Agent communication
│   │   ├── generated/                   # Auto-generated code
│   │   │   ├── rust/
│   │   │   ├── typescript/
│   │   │   └── python/
│   │   └── Cargo.toml                   # Publish as mg-proto crate
│   │
│   ├── mg-sdk-rust/                     # Core Rust SDK
│   │   ├── src/
│   │   │   ├── types/                   # Shared domain types
│   │   │   ├── errors/                  # Error types and handling
│   │   │   ├── traits/                  # Core traits (MCPConnector, etc.)
│   │   │   ├── client/                  # HTTP/gRPC client utilities
│   │   │   └── telemetry/               # Logging, tracing, metrics
│   │   ├── Cargo.toml                   # Publish as mg-sdk crate
│   │   └── README.md
│   │
│   └── mg-config/                       # Shared configuration
│       ├── schemas/                     # JSON Schema definitions
│       ├── defaults/                    # Default config files
│       └── validation/                  # Config validation logic
│
├── LAYER 1: DATA INGESTION (14 repos)
│   │
│   ├── mg-ingestion-core/               # MCP connector framework
│   │   ├── src/
│   │   │   ├── connector.rs             # MCPConnector trait
│   │   │   ├── rate_limiter.rs          # Rate limiting
│   │   │   ├── retry.rs                 # Retry logic
│   │   │   └── health.rs                # Health checks
│   │   ├── Cargo.toml                   # Depends on: mg-sdk-rust, mg-proto
│   │   └── README.md
│   │
│   ├── mg-connector-aggregator/         # JustWatch/Watchmode/StreamingAvailability
│   │   ├── src/
│   │   │   ├── justwatch.rs
│   │   │   ├── watchmode.rs
│   │   │   └── streaming_availability.rs
│   │   ├── Cargo.toml                   # Depends on: mg-ingestion-core
│   │   └── README.md
│   │
│   ├── mg-connector-netflix/            # Netflix via aggregators
│   ├── mg-connector-prime/              # Prime Video
│   ├── mg-connector-disney/             # Disney+
│   ├── mg-connector-hulu/               # Hulu
│   ├── mg-connector-apple/              # Apple TV+
│   ├── mg-connector-youtube/            # YouTube (real API)
│   ├── mg-connector-crave/              # Crave (Canada)
│   ├── mg-connector-hbomax/             # HBO Max
│   ├── mg-connector-peacock/            # Peacock
│   ├── mg-connector-paramount/          # Paramount+
│   │
│   ├── mg-metadata-normalizer/          # Platform → Unified schema
│   │   ├── src/
│   │   │   ├── normalizer.rs            # Normalization logic
│   │   │   ├── schemas/                 # Platform-specific schemas
│   │   │   └── unified.rs               # Unified content schema
│   │   └── Cargo.toml                   # Depends on: mg-sdk-rust
│   │
│   └── mg-entity-resolver/              # EIDR/IMDb/TMDb deduplication
│       ├── src/
│       │   ├── resolver.rs              # Entity resolution logic
│       │   ├── matchers/                # Fuzzy matching algorithms
│       │   └── identifiers.rs           # External ID handling
│       └── Cargo.toml                   # Depends on: mg-sdk-rust
│
├── LAYER 1: IDENTITY & AUTH (3 repos)
│   │
│   ├── mg-auth-service/                 # OAuth2/PKCE + Device Grant
│   │   ├── src/
│   │   │   ├── oauth2.rs                # OAuth2 flows
│   │   │   ├── pkce.rs                  # PKCE implementation
│   │   │   ├── device_grant.rs          # RFC 8628 Device Grant
│   │   │   └── providers/               # Platform-specific auth
│   │   ├── Cargo.toml                   # Depends on: mg-sdk-rust
│   │   └── README.md
│   │
│   ├── mg-token-service/                # JWT management
│   │   ├── src/
│   │   │   ├── jwt.rs                   # JWT creation/validation
│   │   │   ├── refresh.rs               # Token refresh logic
│   │   │   └── storage.rs               # Secure token storage
│   │   └── Cargo.toml
│   │
│   └── mg-session-service/              # Cross-device sessions
│       ├── src/
│       │   ├── session.rs               # Session management
│       │   └── sync.rs                  # Cross-device sync
│       └── Cargo.toml
│
├── LAYER 1: DEVICE SYNC (3 repos)
│   │
│   ├── mg-device-gateway/               # WebSocket hub
│   │   ├── src/
│   │   │   ├── websocket.rs             # WebSocket server
│   │   │   ├── pubsub.rs                # PubNub integration
│   │   │   └── presence.rs              # Device presence
│   │   └── Cargo.toml
│   │
│   ├── mg-sync-engine/                  # CRDT-based sync
│   │   ├── src/
│   │   │   ├── crdt.rs                  # CRDT implementations
│   │   │   ├── conflict.rs              # Conflict resolution
│   │   │   └── watchlist.rs             # Watchlist sync
│   │   └── Cargo.toml
│   │
│   └── mg-device-adapters/              # Platform-specific adapters
│       ├── src/
│       │   ├── tizen.rs                 # Samsung Tizen
│       │   ├── webos.rs                 # LG webOS
│       │   ├── roku.rs                  # Roku
│       │   └── chromecast.rs            # Chromecast
│       └── Cargo.toml
│
├── LAYER 2: INTELLIGENCE (8 repos)
│   │
│   ├── mg-agent-orchestrator/           # Claude-Flow integration
│   │   ├── src/
│   │   │   ├── orchestrator.rs          # SPARC methodology
│   │   │   ├── agents/                  # 9 specialized agents
│   │   │   │   ├── swarm_lead.rs
│   │   │   │   ├── content_searcher.rs
│   │   │   │   ├── recommendation_builder.rs
│   │   │   │   ├── availability_checker.rs
│   │   │   │   ├── device_coordinator.rs
│   │   │   │   ├── context_keeper.rs
│   │   │   │   ├── pattern_learner.rs
│   │   │   │   ├── result_merger.rs
│   │   │   │   └── quality_assurer.rs
│   │   │   └── tools/                   # Agent tools
│   │   ├── Cargo.toml                   # Depends on: mg-sdk-rust, mg-e2b-client
│   │   └── README.md
│   │
│   ├── mg-recommendation-engine/        # Hybrid GNN recommendations
│   │   ├── src/
│   │   │   ├── engine.rs                # Main recommendation engine
│   │   │   ├── collaborative.rs         # Collaborative filtering
│   │   │   ├── content_based.rs         # Content-based filtering
│   │   │   ├── gnn.rs                   # GraphSAGE integration
│   │   │   ├── fusion.rs                # RRF fusion
│   │   │   └── diversity.rs             # MMR diversity
│   │   └── Cargo.toml                   # Depends on: mg-ruvector-client, mg-sona-client
│   │
│   ├── mg-semantic-search/              # Vector + Graph search
│   │   ├── src/
│   │   │   ├── search.rs                # Unified search interface
│   │   │   ├── vector.rs                # HNSW vector search
│   │   │   ├── graph.rs                 # Graph traversal
│   │   │   └── hybrid.rs                # Hybrid search
│   │   └── Cargo.toml
│   │
│   ├── mg-embedding-service/            # Content embeddings
│   │   ├── src/
│   │   │   ├── embedder.rs              # Embedding generation
│   │   │   ├── models/                  # Embedding models
│   │   │   └── cache.rs                 # Embedding cache
│   │   └── Cargo.toml
│   │
│   ├── mg-sona-client/                  # SONA intelligence client
│   │   ├── src/
│   │   │   ├── client.rs                # SONA API client
│   │   │   ├── lora.rs                  # Two-Tier LoRA adapters
│   │   │   ├── ewc.rs                   # EWC++ anti-forgetting
│   │   │   ├── attention.rs             # 39 attention mechanisms
│   │   │   └── tiny_dancer.rs           # Semantic routing
│   │   └── Cargo.toml
│   │
│   ├── mg-reasoning-bank/               # Reasoning pattern storage
│   │   ├── src/
│   │   │   ├── bank.rs                  # Pattern storage
│   │   │   ├── patterns.rs              # Pattern types
│   │   │   └── retrieval.rs             # Pattern retrieval
│   │   └── Cargo.toml
│   │
│   ├── mg-agent-db/                     # Agent context storage (Redis)
│   │   ├── src/
│   │   │   ├── store.rs                 # Key-value storage
│   │   │   ├── context.rs               # Agent context
│   │   │   └── memory.rs                # Working memory
│   │   └── Cargo.toml
│   │
│   └── mg-e2b-client/                   # E2B sandbox client
│       ├── src/
│       │   ├── client.rs                # E2B API client
│       │   ├── sandbox.rs               # Sandbox management
│       │   ├── execution.rs             # Code execution
│       │   └── templates.rs             # Custom templates
│       └── Cargo.toml
│
├── LAYER 3: CONSOLIDATION (5 repos)
│   │
│   ├── mg-metadata-fabric/              # Global unified metadata
│   │   ├── src/
│   │   │   ├── fabric.rs                # Metadata aggregation
│   │   │   ├── unified.rs               # Unified content model
│   │   │   └── enrichment.rs            # Metadata enrichment
│   │   └── Cargo.toml
│   │
│   ├── mg-availability-index/           # Cross-platform availability
│   │   ├── src/
│   │   │   ├── index.rs                 # Availability index
│   │   │   ├── regional.rs              # Regional availability
│   │   │   └── pricing.rs               # Pricing information
│   │   └── Cargo.toml
│   │
│   ├── mg-rights-engine/                # Rights aggregation
│   │   ├── src/
│   │   │   ├── engine.rs                # Rights validation
│   │   │   ├── licensing.rs             # License tracking
│   │   │   └── expiration.rs            # Expiration alerts
│   │   └── Cargo.toml
│   │
│   ├── mg-personalization-engine/       # Privacy-safe personalization
│   │   ├── src/
│   │   │   ├── engine.rs                # Personalization logic
│   │   │   ├── federated.rs             # Federated learning
│   │   │   ├── privacy.rs               # Differential privacy
│   │   │   └── preferences.rs           # User preferences
│   │   └── Cargo.toml
│   │
│   └── mg-search-api/                   # Unified search API
│       ├── src/
│       │   ├── api.rs                   # REST/gRPC API
│       │   ├── handlers.rs              # Request handlers
│       │   └── filters.rs               # Search filters
│       └── Cargo.toml
│
├── LAYER 4: END-USER APPLICATIONS (6 repos)
│   │
│   ├── mg-cli/                          # Developer/Operator CLI
│   │   ├── src/
│   │   │   ├── main.rs                  # CLI entry point
│   │   │   ├── commands/                # Command implementations
│   │   │   ├── tui/                     # Terminal UI (ratatui)
│   │   │   └── config.rs                # CLI configuration
│   │   └── Cargo.toml                   # Binary crate
│   │
│   ├── mg-web-app/                      # Next.js web application
│   │   ├── src/
│   │   │   ├── app/                     # Next.js app router
│   │   │   ├── components/              # React components
│   │   │   └── lib/                     # Utilities
│   │   ├── package.json
│   │   └── README.md
│   │
│   ├── mg-mobile-ios/                   # iOS application (Swift)
│   │   ├── MediaGateway/
│   │   │   ├── Views/
│   │   │   ├── ViewModels/
│   │   │   └── Services/
│   │   └── MediaGateway.xcodeproj
│   │
│   ├── mg-mobile-android/               # Android application (Kotlin)
│   │   ├── app/src/main/
│   │   │   ├── java/
│   │   │   └── res/
│   │   └── build.gradle.kts
│   │
│   ├── mg-tv-tizen/                     # Samsung Tizen app
│   │   ├── src/
│   │   └── config.xml
│   │
│   └── mg-tv-webos/                     # LG webOS app
│       ├── src/
│       └── appinfo.json
│
├── DATA LAYER (2 repos)
│   │
│   ├── mg-ruvector-client/              # Ruvector Rust client
│   │   ├── src/
│   │   │   ├── client.rs                # Main client
│   │   │   ├── hypergraph.rs            # Hypergraph operations
│   │   │   ├── vector.rs                # Vector operations
│   │   │   ├── gnn.rs                   # GNN queries
│   │   │   └── cypher.rs                # Cypher query builder
│   │   └── Cargo.toml
│   │
│   └── mg-pubnub-client/                # PubNub Rust client wrapper
│       ├── src/
│       │   ├── client.rs                # PubNub client
│       │   ├── channels.rs              # Channel management
│       │   └── presence.rs              # Presence features
│       └── Cargo.toml
│
├── INFRASTRUCTURE (4 repos)
│   │
│   ├── mg-terraform/                    # Terraform modules
│   │   ├── modules/
│   │   │   ├── gke/                     # GKE Autopilot
│   │   │   ├── cloudrun/                # Cloud Run
│   │   │   ├── cloudsql/                # Cloud SQL PostgreSQL
│   │   │   ├── memorystore/             # Memorystore Valkey
│   │   │   ├── pubsub/                  # Pub/Sub
│   │   │   ├── security/                # IAM, Secrets, Armor
│   │   │   └── observability/           # Logging, Monitoring
│   │   ├── environments/
│   │   │   ├── dev/
│   │   │   ├── staging/
│   │   │   └── prod/
│   │   └── README.md
│   │
│   ├── mg-helm-charts/                  # Helm charts
│   │   ├── charts/
│   │   │   ├── mg-ingestion/
│   │   │   ├── mg-intelligence/
│   │   │   ├── mg-consolidation/
│   │   │   └── mg-api/
│   │   └── README.md
│   │
│   ├── mg-docker/                       # Dockerfiles and compose
│   │   ├── dockerfiles/
│   │   │   ├── rust-service.Dockerfile
│   │   │   ├── web-app.Dockerfile
│   │   │   └── cli.Dockerfile
│   │   ├── docker-compose.dev.yml
│   │   └── docker-compose.test.yml
│   │
│   └── mg-ci-templates/                 # Shared CI/CD templates
│       ├── .github/
│       │   └── workflows/
│       │       ├── rust-build.yml
│       │       ├── rust-test.yml
│       │       └── deploy-gcp.yml
│       └── README.md
│
├── TESTING & SIMULATION (2 repos)
│   │
│   ├── mg-test-utils/                   # Shared test utilities
│   │   ├── src/
│   │   │   ├── fixtures/                # Test fixtures
│   │   │   ├── mocks/                   # Mock implementations
│   │   │   └── helpers/                 # Test helpers
│   │   └── Cargo.toml
│   │
│   └── mg-simulator/                    # Simulation & mock APIs
│       ├── src/
│       │   ├── catalog_gen.rs           # Synthetic catalog generator
│       │   ├── mock_apis/               # Mock streaming APIs
│       │   └── load_gen.rs              # Load testing
│       └── Cargo.toml
│
└── DOCUMENTATION (1 repo)
    │
    └── mg-docs/                         # Documentation site
        ├── docs/
        │   ├── architecture/
        │   ├── api/
        │   ├── guides/
        │   └── tutorials/
        ├── docusaurus.config.js
        └── README.md
```

#### Repository Count Summary

| Category | Count | Purpose |
|----------|-------|---------|
| **Foundation** | 3 | Proto, SDK, Config |
| **Layer 1 - Ingestion** | 14 | MCP connectors, normalizers |
| **Layer 1 - Identity** | 3 | Auth, tokens, sessions |
| **Layer 1 - Device** | 3 | Gateway, sync, adapters |
| **Layer 2 - Intelligence** | 8 | Agents, recommendations, search |
| **Layer 3 - Consolidation** | 5 | Metadata, availability, rights |
| **Layer 4 - Applications** | 6 | CLI, web, mobile, TV |
| **Data Layer** | 2 | Ruvector, PubNub clients |
| **Infrastructure** | 4 | Terraform, Helm, Docker, CI |
| **Testing** | 2 | Test utils, simulator |
| **Documentation** | 1 | Docs site |
| **Total** | **51** | Complete platform |

### 2.2 Dependency Graph

```
                    ┌─────────────────┐
                    │    mg-proto     │
                    │   (Protobuf)    │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │  mg-sdk-rust    │
                    │  (Core Types)   │
                    └────────┬────────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
┌────────▼────────┐ ┌────────▼────────┐ ┌────────▼────────┐
│mg-ingestion-core│ │mg-ruvector-client│ │  mg-e2b-client  │
│  (Connectors)   │ │   (Data Layer)  │ │   (Sandboxes)   │
└────────┬────────┘ └────────┬────────┘ └────────┬────────┘
         │                   │                   │
         │          ┌────────▼────────┐          │
         │          │  mg-sona-client │          │
         │          │  (Intelligence) │          │
         │          └────────┬────────┘          │
         │                   │                   │
┌────────▼───────────────────▼───────────────────▼────────┐
│                  mg-agent-orchestrator                   │
│              (Claude-Flow + 9 Agents)                   │
└─────────────────────────┬───────────────────────────────┘
                          │
         ┌────────────────┼────────────────┐
         │                │                │
┌────────▼────────┐ ┌─────▼─────┐ ┌────────▼────────┐
│mg-recommendation│ │mg-semantic│ │mg-metadata-fabric│
│    -engine      │ │  -search  │ │  (Consolidation) │
└────────┬────────┘ └─────┬─────┘ └────────┬────────┘
         │                │                │
         └────────────────┼────────────────┘
                          │
                 ┌────────▼────────┐
                 │   mg-search-api │
                 │   (Unified API) │
                 └────────┬────────┘
                          │
         ┌────────────────┼────────────────┐
         │                │                │
┌────────▼────────┐ ┌─────▼─────┐ ┌────────▼────────┐
│     mg-cli      │ │mg-web-app │ │   mg-tv-tizen   │
│  (Developer)    │ │ (Next.js) │ │   (Samsung)     │
└─────────────────┘ └───────────┘ └─────────────────┘
```

### 2.3 Repository Wiring Strategy

Each repository communicates via well-defined interfaces:

#### Inter-Repository Communication

| Communication Type | Technology | Use Case |
|-------------------|------------|----------|
| **Sync API Calls** | gRPC (tonic) | Service-to-service |
| **Async Events** | Pub/Sub | Event streaming |
| **Real-time Sync** | PubNub | Device synchronization |
| **Shared Types** | mg-proto | Schema definitions |
| **Client Libraries** | mg-sdk-rust | Rust integration |

#### Versioning Strategy

```yaml
# Each repo follows semantic versioning
# Breaking changes require major version bump
# All repos publish to crates.io (Rust) or npm (TypeScript)

# Example Cargo.toml dependency
[dependencies]
mg-sdk = "1.2.0"           # Pinned version
mg-proto = "1.0.0"         # Stable API
mg-ingestion-core = "0.5"  # Flexible minor
```

#### CI/CD Integration

Each repository has its own CI/CD pipeline that:
1. Runs tests on PR
2. Publishes to package registry on merge to main
3. Triggers dependent repos via webhook (optional)
4. Deploys to GKE/Cloud Run via mg-terraform

### 2.4 MCP Connector Interface (Rust)

```rust
#[async_trait]
pub trait MCPConnector: Send + Sync {
    fn platform_type(&self) -> PlatformType;
    fn get_tool_schemas(&self) -> Vec<MCPToolSchema>;
    async fn initialize(&mut self, credentials: AuthCredentials) -> Result<(), ConnectorError>;
    async fn fetch_catalog(&self, region: &str, cursor: Option<String>, limit: usize) -> Result<MCPResponse, ConnectorError>;
    async fn fetch_content(&self, platform_id: &str) -> Result<ContentMetadata, ConnectorError>;
    async fn search_content(&self, query: &str, region: &str, limit: usize) -> Result<MCPResponse, ConnectorError>;
    async fn health_check(&self) -> Result<ConnectorHealth, ConnectorError>;
    fn rate_limit_config(&self) -> RateLimitConfig;
}
```

---

## 3. Layer-2: Intelligence Layer

### 3.1 Multi-Agent Architecture (9 Agents)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        AGENT ORCHESTRATION SYSTEM                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  COORDINATOR AGENTS:                                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  1. SwarmLead                                                        │   │
│  │     - Overall coordination and task delegation                       │   │
│  │     - SPARC methodology orchestration                                │   │
│  │     - Tools: swarm_monitor, agent_assign, task_create               │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  SPECIALIST AGENTS:                                                         │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐         │
│  │ 2. ContentSearcher│  │ 3. RecommendationBuilder│  │ 4. AvailabilityChecker│  │
│  │    - Semantic     │  │    - Personalized  │  │    - Rights validation│  │
│  │      search       │  │      recommendations│  │    - Deep link check │  │
│  │    - ruvector_    │  │    - gnn_recommend │  │    - rights_check    │  │
│  │      search       │  │    - preference_   │  │    - deeplink_       │  │
│  │                   │  │      model         │  │      validate        │  │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘         │
│                                                                             │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐         │
│  │ 5. DeviceCoordinator│  │ 6. ContextKeeper│  │ 7. PatternLearner│         │
│  │    - Cross-device │  │    - AgentDB     │  │    - ReasoningBank│         │
│  │      commands     │  │      memory      │  │      storage     │         │
│  │    - device_send  │  │    - memory_     │  │    - pattern_    │         │
│  │    - sync_state   │  │      store/get   │  │      store       │         │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘         │
│                                                                             │
│  ┌──────────────────┐  ┌──────────────────┐                                │
│  │ 8. ResultMerger  │  │ 9. QualityAssurer│                                │
│  │    - Stream      │  │    - Trust       │                                │
│  │      aggregation │  │      scoring     │                                │
│  │    - Dedup       │  │    - Validation  │                                │
│  │    - Ranking     │  │    - Edge cases  │                                │
│  └──────────────────┘  └──────────────────┘                                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 SPARC Methodology Integration

```
SPARC PHASE FLOW:

S (Specification)     P (Pseudocode)      A (Architecture)
     │                      │                    │
     ▼                      ▼                    ▼
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│ Parse Query │      │ Design Plan │      │ Spawn Agents│
│ Intent      │ ───► │ Execution   │ ───► │ Parallel    │
│ Entities    │      │ Strategy    │      │ Coordination│
└─────────────┘      └─────────────┘      └─────────────┘
                                                │
                                                ▼
R (Refinement)        C (Completion)     ◄──────┘
     │                      │
     ▼                      ▼
┌─────────────┐      ┌─────────────┐
│ Validate    │      │ Format      │
│ Trust Score │ ───► │ Response    │
│ Edge Cases  │      │ Learn       │
└─────────────┘      └─────────────┘
```

### 3.3 Recommendation Engine (Hybrid)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    HYBRID RECOMMENDATION ARCHITECTURE                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  INPUT: User Context + Query                                                │
│            │                                                                │
│            ▼                                                                │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                    PARALLEL STRATEGY EXECUTION                       │   │
│  │                                                                      │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────┐│   │
│  │  │Collaborative │  │Content-Based │  │     GNN      │  │ Context  ││   │
│  │  │ Filtering    │  │  Filtering   │  │  (Ruvector)  │  │  Aware   ││   │
│  │  │   (35%)      │  │    (25%)     │  │    (30%)     │  │  (10%)   ││   │
│  │  │              │  │              │  │              │  │          ││   │
│  │  │ User-User    │  │  Embedding   │  │  GraphSAGE   │  │  Time    ││   │
│  │  │ Item-Item    │  │  Similarity  │  │  3-layer     │  │  Device  ││   │
│  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └────┬─────┘│   │
│  │         │                 │                 │               │      │   │
│  │         └─────────────────┴─────────────────┴───────────────┘      │   │
│  │                                    │                                │   │
│  └────────────────────────────────────┼────────────────────────────────┘   │
│                                       ▼                                    │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                 RECIPROCAL RANK FUSION (RRF)                         │   │
│  │                                                                      │   │
│  │  RRF(d) = Σ 1/(k + rank_i(d)) * weight_i                            │   │
│  │                                                                      │   │
│  │  Weights: collaborative=0.35, content=0.25, gnn=0.30, context=0.10  │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                       │                                    │
│                                       ▼                                    │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                     POST-PROCESSING                                  │   │
│  │                                                                      │   │
│  │  1. Diversity Injection (MMR, λ=0.85)                               │   │
│  │  2. Trust Filtering (threshold=0.6)                                 │   │
│  │  3. Availability Filtering (user's region + subscriptions)          │   │
│  │  4. Explanation Generation                                          │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                       │                                    │
│                                       ▼                                    │
│                              OUTPUT: Ranked Results                        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

PERFORMANCE METRICS:
- Precision@10: 0.26 (+44% over baseline)
- NDCG@10: 0.54 (+29% improvement)
- Latency: 62ms (p50), 124ms (p99)
- Throughput: 12,000 RPS (with caching)
```

---

## 4. Layer-3: Consolidation Layer

### 4.1 Global Metadata Fabric

```rust
pub struct GlobalMetadataFabric {
    ruvector: Arc<RuvectorClient>,
    availability_index: Arc<AvailabilityIndex>,
    rights_engine: Arc<RightsEngine>,
}

impl GlobalMetadataFabric {
    /// Get unified content with all availability across platforms
    pub async fn get_unified_content(&self, content_id: &str, region: &str) -> Result<UnifiedContent> {
        // 1. Fetch from Ruvector hypergraph
        let content = self.ruvector.get_content(content_id).await?;

        // 2. Get availability across all platforms
        let availability = self.availability_index
            .get_availability(content_id, region)
            .await?;

        // 3. Validate rights
        let validated = self.rights_engine
            .validate_availability(&availability)
            .await?;

        Ok(UnifiedContent {
            metadata: content,
            availability: validated,
            trust_score: self.calculate_trust(&content, &validated),
        })
    }
}
```

### 4.2 Unified Availability Index

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      UNIFIED AVAILABILITY INDEX                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Query: "Where can I watch 'Stranger Things' in US?"                       │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  RESPONSE                                                            │   │
│  ├─────────────────────────────────────────────────────────────────────┤   │
│  │  Content: Stranger Things (2016)                                    │   │
│  │  Unified ID: unified:eidr:10.5240/XXXX                              │   │
│  │                                                                      │   │
│  │  Availability in US:                                                │   │
│  │  ┌─────────────┬────────────┬───────────────┬────────────────────┐ │   │
│  │  │ Platform    │ Model      │ Quality       │ Deep Link          │ │   │
│  │  ├─────────────┼────────────┼───────────────┼────────────────────┤ │   │
│  │  │ Netflix     │ SVOD       │ 4K HDR Atmos  │ netflix.com/t/1234 │ │   │
│  │  │ Trust: 0.95 │ $15.99/mo  │               │                    │ │   │
│  │  ├─────────────┼────────────┼───────────────┼────────────────────┤ │   │
│  │  │ Prime Video │ TVOD       │ 4K HDR        │ amazon.com/v/5678  │ │   │
│  │  │ Trust: 0.88 │ $3.99 rent │               │                    │ │   │
│  │  └─────────────┴────────────┴───────────────┴────────────────────┘ │   │
│  │                                                                      │   │
│  │  Expiring: Netflix license ends 2025-03-15 (100 days)               │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Layer-4: End-User Experience

Layer 4 provides consumer-facing applications for end-users. **Note:** The CLI tools (Media Gateway CLI, hackathon-tv5 CLI) are **developer/operator tools** for platform management, not end-user interfaces. End-users interact through Web, Mobile, and TV applications.

### 5.1 End-User Applications

| Platform | Technology | Target Users |
|----------|------------|--------------|
| **Web App** | Next.js | Desktop/laptop users |
| **Mobile Apps** | iOS (Swift), Android (Kotlin) | Smartphone users |
| **TV Apps** | Samsung Tizen, LG webOS, Roku | Smart TV users |

### 5.2 Developer/Operator CLI Architecture

The CLI is designed for **developers and platform operators** to manage, debug, and operate the Media Gateway platform. It is NOT an end-user interface.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           CLI COMMAND STRUCTURE                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  tv-discover/                                                               │
│  ├── search                 # Semantic content search                       │
│  │   ├── query              # Natural language: "sci-fi like stranger things"│
│  │   ├── filter             # Structured filters                            │
│  │   └── similar            # Find similar to content ID                    │
│  │                                                                          │
│  ├── browse                 # Interactive browsing                          │
│  │   ├── trending           # What's popular now                            │
│  │   ├── new                # Recently added                                │
│  │   ├── expiring           # Leaving soon                                  │
│  │   └── categories         # Browse by genre/mood                          │
│  │                                                                          │
│  ├── recommend              # Personalized recommendations                  │
│  │   ├── for-me             # Based on watch history                        │
│  │   ├── mood               # "I want something light"                      │
│  │   └── social             # What friends are watching                     │
│  │                                                                          │
│  ├── watch                  # Playback control                              │
│  │   ├── now                # Open in streaming app                         │
│  │   ├── queue              # Add to watch queue                            │
│  │   └── list               # Manage watchlist                              │
│  │                                                                          │
│  ├── devices                # Device management                             │
│  │   ├── list               # Show connected devices                        │
│  │   ├── connect            # Pair new device                               │
│  │   ├── sync               # Sync watch progress                           │
│  │   └── cast               # Cast to TV                                    │
│  │                                                                          │
│  ├── accounts               # Platform account management                   │
│  │   ├── list               # Show linked subscriptions                     │
│  │   ├── link               # Add platform subscription                     │
│  │   ├── unlink             # Remove platform                               │
│  │   └── status             # Check account status                          │
│  │                                                                          │
│  ├── preferences            # User preferences                              │
│  │   ├── genres             # Genre preferences                             │
│  │   ├── platforms          # Platform priorities                           │
│  │   ├── region             # Regional settings                             │
│  │   └── export             # Export/backup preferences                     │
│  │                                                                          │
│  └── config                 # CLI configuration                             │
│      ├── init               # First-time setup                              │
│      ├── set                # Set config value                              │
│      └── show               # Show current config                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 5.3 CLI TUI Interface (Developer/Operator View)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  TV DISCOVER                                      Region: US │ ▶ Netflix   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  🔍 Search: sci-fi shows like stranger things_                              │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │ RESULTS (12 found)                                        Trust: ★★★★  │ │
│  ├───────────────────────────────────────────────────────────────────────┤ │
│  │ ▶ Dark (2017)                                             [Netflix]   │ │
│  │   German mystery thriller with sci-fi elements                        │ │
│  │   ⭐ 8.7 │ 3 Seasons │ 4K HDR │ Dolby Atmos                           │ │
│  │   Because you watched: Stranger Things, Black Mirror                  │ │
│  │   [Enter: Watch] [q: Queue] [i: Info] [s: Similar]                    │ │
│  ├───────────────────────────────────────────────────────────────────────┤ │
│  │   The OA (2016)                                           [Netflix]   │ │
│  │   Mind-bending supernatural drama                                     │ │
│  │   ⭐ 7.8 │ 2 Seasons │ 4K                                             │ │
│  │   Leaving Netflix in 45 days                                          │ │
│  ├───────────────────────────────────────────────────────────────────────┤ │
│  │   Black Mirror (2011)                                     [Netflix]   │ │
│  │   Anthology series exploring tech and society                         │ │
│  │   ⭐ 8.8 │ 6 Seasons │ 4K HDR                                         │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  ↑/↓: Navigate │ Enter: Select │ f: Filter │ Tab: Platforms │ ?: Help     │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Ruvector Hypergraph Schema

### 6.1 Node Types

```
NODE TYPES & EMBEDDINGS:
========================

┌──────────────┬────────────────────────────────────────────────────────────┐
│ Node Type    │ Properties & Embedding Dimensions                          │
├──────────────┼────────────────────────────────────────────────────────────┤
│ Content      │ id, title, type, year, runtime, synopsis                   │
│              │ Embedding: 1536-dim (plot + metadata)                      │
├──────────────┼────────────────────────────────────────────────────────────┤
│ TVShow       │ extends Content + seasons, episode_count, status           │
│              │ Embedding: 1536-dim                                        │
├──────────────┼────────────────────────────────────────────────────────────┤
│ Episode      │ extends Content + show_id, season, episode_number          │
│              │ Embedding: 768-dim                                         │
├──────────────┼────────────────────────────────────────────────────────────┤
│ Person       │ id, name, bio, born, photo_url                             │
│              │ Embedding: 768-dim (filmography style)                     │
├──────────────┼────────────────────────────────────────────────────────────┤
│ User         │ id, preferences, region (privacy-protected)                │
│              │ Embedding: 512-dim (preference vector)                     │
├──────────────┼────────────────────────────────────────────────────────────┤
│ Genre        │ id, name, parent_genre                                     │
│              │ Embedding: 256-dim                                         │
├──────────────┼────────────────────────────────────────────────────────────┤
│ Mood         │ id, name, intensity                                        │
│              │ Embedding: 256-dim                                         │
├──────────────┼────────────────────────────────────────────────────────────┤
│ Platform     │ id, name, type, deep_link_pattern                          │
│              │ No embedding (categorical)                                 │
├──────────────┼────────────────────────────────────────────────────────────┤
│ Region       │ code (ISO 3166), name, timezone                            │
│              │ No embedding (categorical)                                 │
├──────────────┼────────────────────────────────────────────────────────────┤
│ License      │ id, licensor, contract_type                                │
│              │ No embedding (metadata only)                               │
└──────────────┴────────────────────────────────────────────────────────────┘
```

### 6.2 Edge Types

```
STANDARD EDGES:
===============
- ACTED_IN (Person → Content) {character, billing_order}
- DIRECTED (Person → Content) {}
- WROTE (Person → Content) {role: "screenplay"|"story"}
- PRODUCED (Person → Content) {}
- BELONGS_TO (Content → Genre) {confidence}
- HAS_MOOD (Content → Mood) {intensity}
- SIMILAR_TO (Content → Content) {similarity_score, method}
- SEQUEL_OF (Content → Content) {order}
- WATCHED (User → Content) {timestamp, progress, completed}
- RATED (User → Content) {rating, timestamp}
- WATCHLIST (User → Content) {added_at, priority}
- FOLLOWS (User → User) {}

HYPEREDGE: AVAILABILITY_WINDOW
==============================
Connects: Content × Platform × Region × TimeWindow × LicenseTerms

┌─────────────────────────────────────────────────────────────────────┐
│                    AVAILABILITY_WINDOW HYPEREDGE                     │
├─────────────────────────────────────────────────────────────────────┤
│ Properties:                                                          │
│   - start_date: DateTime                                            │
│   - end_date: DateTime (nullable = perpetual)                       │
│   - pricing_model: SVOD | TVOD | AVOD | FREE                        │
│   - quality_tiers: [SD, HD, FHD, 4K, HDR, DolbyVision]             │
│   - audio_formats: [Stereo, 5.1, Atmos]                            │
│   - content_rights: {streaming, download, offline_hours}            │
│   - exclusivity: Exclusive | Shared | NonExclusive                  │
│   - deep_link: String                                               │
│   - trust_score: f32                                                │
│   - last_verified: DateTime                                         │
│   - verification_source: PlatformAPI | Aggregator | UserReport      │
└─────────────────────────────────────────────────────────────────────┘
```

### 6.3 GNN Architecture (GraphSAGE)

```
GRAPHSAGE LAYERS:
=================

Layer 1: Input → 512
  - Neighbors sampled: 25
  - Attention heads: 8
  - Activation: ReLU
  - Dropout: 0.2

Layer 2: 512 → 256
  - Neighbors sampled: 15
  - Attention heads: 4
  - Activation: ReLU
  - Dropout: 0.2

Layer 3: 256 → 128
  - Neighbors sampled: 10
  - Attention heads: 2
  - Activation: None
  - Output: Final embeddings

Total Parameters: ~183K
Inference Time: 45ms (p50), 95ms (p99)
```

---

## 7. PubNub Channel Topology

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PUBNUB CHANNEL TOPOLOGY                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  USER CHANNELS (per user):                                                  │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │ user.{userId}.sync                                                     │ │
│  │   - Watch progress updates                                            │ │
│  │   - Watchlist changes                                                 │ │
│  │   - Preference updates                                                │ │
│  │   - Message format: CRDT deltas                                       │ │
│  ├───────────────────────────────────────────────────────────────────────┤ │
│  │ user.{userId}.devices                                                  │ │
│  │   - Device presence (online/offline)                                  │ │
│  │   - Device capabilities                                               │ │
│  │   - Last active timestamp                                             │ │
│  ├───────────────────────────────────────────────────────────────────────┤ │
│  │ user.{userId}.notifications                                            │ │
│  │   - Content expiration alerts                                         │ │
│  │   - New content recommendations                                       │ │
│  │   - Social updates (friend activity)                                  │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  GLOBAL CHANNELS:                                                           │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │ global.trending                                                        │ │
│  │   - Top 100 trending content (updated hourly)                         │ │
│  │   - Platform-agnostic rankings                                        │ │
│  ├───────────────────────────────────────────────────────────────────────┤ │
│  │ global.announcements                                                   │ │
│  │   - System announcements                                              │ │
│  │   - Feature updates                                                   │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  REGIONAL CHANNELS:                                                         │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │ region.{regionCode}.updates                                            │ │
│  │   - New content available in region                                   │ │
│  │   - Content leaving region soon                                       │ │
│  │   - Platform availability changes                                     │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  PLATFORM CHANNELS:                                                         │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │ platform.{platformId}.catalog                                          │ │
│  │   - Catalog updates (additions/removals)                              │ │
│  │   - Pricing changes                                                   │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  CHANNEL GROUPS:                                                            │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │ user-{userId}-all:                                                     │ │
│  │   - user.{userId}.sync                                                │ │
│  │   - user.{userId}.devices                                             │ │
│  │   - user.{userId}.notifications                                       │ │
│  │   - region.{userRegion}.updates                                       │ │
│  │   - global.announcements                                              │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 7.1 CRDT-Based Sync Protocol

```rust
/// Watch progress using Last-Writer-Wins Register
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct WatchProgress {
    pub content_id: String,
    pub progress_seconds: u32,
    pub duration_seconds: u32,
    pub timestamp: i64,      // HLC timestamp for conflict resolution
    pub device_id: String,
}

/// Watchlist using Observed-Remove Set (OR-Set)
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Watchlist {
    pub additions: Vec<WatchlistEntry>,
    pub removals: Vec<WatchlistEntry>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct WatchlistEntry {
    pub content_id: String,
    pub timestamp: i64,
    pub device_id: String,
    pub unique_tag: String,  // For OR-Set semantics
}

impl Watchlist {
    pub fn merge(&mut self, other: Watchlist) {
        // Union of additions
        for entry in other.additions {
            if !self.additions.iter().any(|e| e.unique_tag == entry.unique_tag) {
                self.additions.push(entry);
            }
        }
        // Union of removals
        for entry in other.removals {
            if !self.removals.iter().any(|e| e.unique_tag == entry.unique_tag) {
                self.removals.push(entry);
            }
        }
    }

    pub fn effective_items(&self) -> Vec<&WatchlistEntry> {
        self.additions.iter()
            .filter(|add| {
                !self.removals.iter().any(|rem|
                    rem.content_id == add.content_id && rem.timestamp > add.timestamp
                )
            })
            .collect()
    }
}
```

---

## 8. Authentication Architecture

### 8.1 OAuth 2.0 + PKCE Flow (Web/Mobile)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    OAUTH 2.0 + PKCE FLOW                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Client (Web/Mobile)              Auth Server              Backend          │
│       │                                │                       │            │
│       │ 1. Generate                    │                       │            │
│       │    code_verifier (random)      │                       │            │
│       │    code_challenge = SHA256(v)  │                       │            │
│       │                                │                       │            │
│       │ 2. Redirect to /authorize      │                       │            │
│       │    + code_challenge            │                       │            │
│       │    + code_challenge_method=S256│                       │            │
│       │ ──────────────────────────────>│                       │            │
│       │                                │                       │            │
│       │                    3. User authenticates               │            │
│       │                       (username/password/SSO)          │            │
│       │                                │                       │            │
│       │ 4. Redirect with auth code     │                       │            │
│       │ <──────────────────────────────│                       │            │
│       │                                │                       │            │
│       │ 5. POST /token                 │                       │            │
│       │    + auth_code                 │                       │            │
│       │    + code_verifier             │                       │            │
│       │ ──────────────────────────────>│                       │            │
│       │                                │                       │            │
│       │                    6. Verify SHA256(verifier) ==       │            │
│       │                       code_challenge                   │            │
│       │                                │                       │            │
│       │ 7. Return tokens               │                       │            │
│       │    access_token (15 min)       │                       │            │
│       │    refresh_token (7 days)      │                       │            │
│       │ <──────────────────────────────│                       │            │
│       │                                │                       │            │
│       │ 8. API requests with           │                       │            │
│       │    Bearer token                │                       │            │
│       │ ──────────────────────────────────────────────────────>│            │
│       │                                │                       │            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 8.2 Device Authorization Grant (RFC 8628) for TV/CLI

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    DEVICE AUTHORIZATION GRANT                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  TV/CLI Device              Auth Server              User's Phone           │
│       │                          │                        │                 │
│       │ 1. POST /device/code     │                        │                 │
│       │ ────────────────────────>│                        │                 │
│       │                          │                        │                 │
│       │ 2. Response:             │                        │                 │
│       │    device_code           │                        │                 │
│       │    user_code: "WDJB-MJHT"│                        │                 │
│       │    verification_uri      │                        │                 │
│       │    expires_in: 1800      │                        │                 │
│       │    interval: 5           │                        │                 │
│       │ <────────────────────────│                        │                 │
│       │                          │                        │                 │
│  ┌────┴────┐                     │                        │                 │
│  │ Display │                     │                        │                 │
│  │         │                     │         ┌────────────┐ │                 │
│  │ ┌─────────────────────┐      │         │ User scans │ │                 │
│  │ │ Enter code:         │      │         │ QR or visits│ │                 │
│  │ │                     │      │         │ URL         │ │                 │
│  │ │    WDJB-MJHT        │      │         └────────────┘ │                 │
│  │ │                     │      │                ▼       │                 │
│  │ │ visit:              │      │         ┌────────────┐ │                 │
│  │ │ tv.discover.io/     │      │         │ Enters code│ │                 │
│  │ │ activate            │      │         │ WDJB-MJHT  │ │                 │
│  │ │                     │      │ <───────│            │ │                 │
│  │ │ [QR CODE]           │      │         └────────────┘ │                 │
│  │ └─────────────────────┘      │                        │                 │
│  └────┬────┘                     │                        │                 │
│       │                          │                        │                 │
│       │ 3. Poll every 5s:        │                        │                 │
│       │    POST /token           │                        │                 │
│       │    grant_type=device_code│                        │                 │
│       │    device_code=xxxxx     │                        │                 │
│       │ ────────────────────────>│                        │                 │
│       │                          │                        │                 │
│       │ 4a. Before auth:         │                        │                 │
│       │     "authorization_      │                        │                 │
│       │      pending"            │                        │                 │
│       │ <────────────────────────│                        │                 │
│       │                          │                        │                 │
│       │ 4b. After auth:          │                        │                 │
│       │     access_token         │                        │                 │
│       │     refresh_token        │                        │                 │
│       │ <────────────────────────│                        │                 │
│       │                          │                        │                 │
│  [AUTHENTICATED]                 │                        │                 │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 8.3 Platform Subscription Management

```rust
/// User declares their streaming subscriptions (not OAuth with platforms)
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct UserSubscriptions {
    pub user_id: String,
    pub subscriptions: Vec<PlatformSubscription>,
    pub region: String,
    pub updated_at: DateTime<Utc>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct PlatformSubscription {
    pub platform: Platform,
    pub tier: SubscriptionTier,      // Basic, Standard, Premium
    pub ad_supported: bool,
    pub verified: bool,              // Future: OAuth verification
    pub added_at: DateTime<Utc>,
}

// Benefits:
// 1. Filter recommendations by subscribed platforms
// 2. Prioritize content on user's platforms
// 3. Show pricing for unsubscribed platforms
// 4. No platform API integration required (yet)
```

---

## 9. Multi-Repo Dependency Graph

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        DEPENDENCY GRAPH                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  LAYER 4 (End-User)                                                         │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐                        │
│  │ cli-app │  │ web-app │  │ mobile  │  │ tv-apps │                        │
│  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘                        │
│       │            │            │            │                              │
│       └────────────┴────────────┴────────────┘                              │
│                           │                                                 │
│                           ▼                                                 │
│  LAYER 3 (Consolidation)                                                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                      │
│  │metadata-fabric│  │availability- │  │personalization│                     │
│  │              │  │   index      │  │   -engine    │                      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘                      │
│         │                 │                 │                               │
│         └─────────────────┼─────────────────┘                               │
│                           │                                                 │
│                           ▼                                                 │
│  LAYER 2 (Intelligence)                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                      │
│  │    agent-    │  │recommendation│  │  semantic-   │                      │
│  │ orchestrator │  │   -engine    │  │   search     │                      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘                      │
│         │                 │                 │                               │
│         └─────────────────┼─────────────────┘                               │
│                           │                                                 │
│                           ▼                                                 │
│  LAYER 1 (Micro-Repos)                                                      │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐          │
│  │ingestion│  │metadata-│  │ entity- │  │ rights- │  │  auth-  │          │
│  │-modules │  │normalizer│ │resolver │  │validator│  │ service │          │
│  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘          │
│       │            │            │            │            │                 │
│       └────────────┴────────────┴────────────┴────────────┘                 │
│                                   │                                         │
│                                   ▼                                         │
│  INFRASTRUCTURE                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                  │   │
│  │  │    proto    │  │  sdk-rust   │  │  ruvector-  │                  │   │
│  │  │ (gRPC defs) │  │             │  │   client    │                  │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘                  │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                   │                                         │
│                                   ▼                                         │
│  DATA LAYER                                                                 │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │        RUVECTOR                    │           PUBNUB                 │  │
│  │  (Hypergraph + Vector + GNN)      │    (Real-Time Sync)             │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 10. Cross-Kingdom Data Strategy

### 10.1 Data Kingdoms

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DATA KINGDOMS                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ CONTENT KINGDOM                                                      │   │
│  │ - Movies, TV Shows, Episodes, Seasons                               │   │
│  │ - Metadata: title, description, cast, crew, ratings                 │   │
│  │ - Embeddings: plot, mood, themes                                    │   │
│  │ - Identifiers: EIDR, IMDb, TMDb, Gracenote                         │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                              │                                              │
│                              ▼                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ USER KINGDOM                                                         │   │
│  │ - Profiles, Preferences, Watch History                              │   │
│  │ - Ratings, Watchlists, Social Connections                           │   │
│  │ - Privacy-protected (local storage preferred)                       │   │
│  │ - Federated learning for model updates                              │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                              │                                              │
│                              ▼                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ PLATFORM KINGDOM                                                     │   │
│  │ - Streaming Services (Netflix, Prime, Disney+, etc.)                │   │
│  │ - Pricing Models, Quality Tiers, Features                           │   │
│  │ - Deep Link Patterns, App Integration                               │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                              │                                              │
│                              ▼                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ RIGHTS KINGDOM                                                       │   │
│  │ - Licensing Agreements, Regional Availability                       │   │
│  │ - Time Windows, Exclusivity Terms                                   │   │
│  │ - Quality/Audio Restrictions                                        │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                              │                                              │
│                              ▼                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ DEVICE KINGDOM                                                       │   │
│  │ - Capabilities (resolution, audio formats)                          │   │
│  │ - Sync State, Playback History                                      │   │
│  │ - Authentication Status                                             │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 10.2 Kafka Event Bus

```
KAFKA TOPICS:
=============

content.ingested           # New content from MCP connectors
content.updated            # Metadata updates
content.entity.resolved    # Entity resolution completed
content.embedding.ready    # Embeddings generated

user.preference.updated    # User changed preferences
user.watch.started         # User started watching
user.watch.completed       # User finished watching
user.rating.submitted      # User rated content

rights.availability.changed # Availability window changed
rights.window.expiring     # 30/7/1 day warnings
rights.window.expired      # Rights expired

device.connected           # New device paired
device.playback.state      # Playback position update
device.sync.requested      # Sync requested

CONSUMER GROUPS:
================
ruvector-indexer           # content.*, user.watch.*
recommendation-engine      # user.*, content.embedding.*
notification-service       # rights.window.*
sync-service               # device.*, user.watch.*
```

---

## 11. Trust-Scoring Approach

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        TRUST SCORING SYSTEM                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  OVERALL TRUST SCORE = Σ (component × weight)                              │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ 1. SOURCE RELIABILITY (25%)                                          │   │
│  │    - PlatformAPI: 1.0                                               │   │
│  │    - LicensorData: 0.95                                             │   │
│  │    - ThirdPartyAggregator: 0.70                                     │   │
│  │    - DeepLinkValidation: 0.60                                       │   │
│  │    - WebScraping: 0.40                                              │   │
│  │    - UserReport: 0.20                                               │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ 2. METADATA ACCURACY (25%)                                           │   │
│  │    - Cross-source validation (TMDb, IMDb match)                     │   │
│  │    - Field completeness (synopsis, cast, images)                    │   │
│  │    - User-reported corrections                                      │   │
│  │    - Entity resolution confidence                                   │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ 3. AVAILABILITY CONFIDENCE (20%)                                     │   │
│  │    - Platform confirmation status                                   │   │
│  │    - Deep link validation (HTTP HEAD check)                         │   │
│  │    - User-reported availability                                     │   │
│  │    - Recency of check (exponential decay)                          │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ 4. RECOMMENDATION QUALITY (15%)                                      │   │
│  │    - Click-through rate                                             │   │
│  │    - Watch completion rate                                          │   │
│  │    - User rating correlation                                        │   │
│  │    - A/B test performance                                           │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ 5. USER PREFERENCE CONFIDENCE (15%)                                  │   │
│  │    - Interaction count (more = higher)                              │   │
│  │    - Preference consistency                                         │   │
│  │    - Recency decay                                                  │   │
│  │    - Explicit vs implicit signals                                   │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  TRUST DECAY FUNCTION:                                                      │
│  trust(t) = original_score × (1 - 0.01 × days_since_verification)          │
│                                                                             │
│  FILTERING:                                                                 │
│  - Recommendations with trust < 0.6 are filtered                           │
│  - Availability with trust < 0.5 shows warning                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 12. Privacy-Safe Personalization

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                   PRIVACY-SAFE PERSONALIZATION                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  THREE-TIER DATA ARCHITECTURE:                                              │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ TIER 1: ON-DEVICE (Most Private)                                     │   │
│  │   Data: Full watch history, detailed preferences, viewing patterns   │   │
│  │   Storage: Device local (AES-256 encrypted)                          │   │
│  │   Processing: On-device ML models (TensorFlow Lite)                  │   │
│  │   Sync: Never sent to server                                         │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                              │                                              │
│                              │ Federated Learning                           │
│                              │ (Gradients only, with DP noise)              │
│                              ▼                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ TIER 2: AGGREGATED (Privacy-Preserving)                              │   │
│  │   Data: Anonymized preference clusters, model weights                │   │
│  │   Storage: Ruvector (no PII)                                         │   │
│  │   Processing: Secure aggregation (min 1000 clients)                  │   │
│  │   Privacy: (ε=1.0, δ=1e-5)-differential privacy                      │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                              │                                              │
│                              ▼                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ TIER 3: PUBLIC SIGNALS (Non-Private)                                 │   │
│  │   Data: Content metadata, aggregate popularity, public ratings       │   │
│  │   Storage: Ruvector (public data)                                    │   │
│  │   Processing: Standard algorithms                                    │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  FEDERATED LEARNING PROTOCOL:                                               │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ 1. Server sends global model to devices                              │   │
│  │ 2. Each device trains on LOCAL data only                             │   │
│  │ 3. Device computes gradients with DP noise: ∇L + Laplace(ε)         │   │
│  │ 4. Gradients clipped: ||∇L|| ≤ C                                     │   │
│  │ 5. Secure aggregation: server sees only Σ(encrypted gradients)       │   │
│  │ 6. Server updates global model                                       │   │
│  │ 7. Repeat weekly                                                     │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  USER CONTROLS:                                                             │
│  - View all collected data                                                 │
│  - Delete watch history                                                    │
│  - Opt-out of federated learning                                          │
│  - Export/backup preferences                                              │
│  - Granular privacy settings                                              │
│                                                                             │
│  COMPLIANCE: GDPR, CCPA, VPPA                                              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 13. Multi-Agent Coordination Plan

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     MULTI-AGENT COORDINATION                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  QUERY: "Find light sci-fi shows like Stranger Things on my platforms"     │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ PHASE S: SPECIFICATION (50ms)                                        │   │
│  │   Agent: RequirementsAnalyst                                        │   │
│  │   Output:                                                           │   │
│  │     - intent: SIMILAR                                               │   │
│  │     - reference: "Stranger Things"                                  │   │
│  │     - mood: "light"                                                 │   │
│  │     - genre_filter: "sci-fi"                                        │   │
│  │     - platform_filter: user_subscriptions                           │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                              │                                              │
│                              ▼                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ PHASE P: PSEUDOCODE (30ms)                                           │   │
│  │   Agent: QueryPlanner                                               │   │
│  │   Output:                                                           │   │
│  │     1. Resolve "Stranger Things" → content_id                       │   │
│  │     2. PARALLEL:                                                    │   │
│  │        a. Vector similarity search (mood: light, genre: sci-fi)     │   │
│  │        b. Graph traversal (SIMILAR_TO edges)                        │   │
│  │        c. GNN recommendation                                        │   │
│  │     3. Merge results with RRF                                       │   │
│  │     4. Filter by user's platforms                                   │   │
│  │     5. Apply trust scoring                                          │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                              │                                              │
│                              ▼                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ PHASE A: ARCHITECTURE (200ms parallel)                               │   │
│  │   Agents spawned:                                                   │   │
│  │   ┌──────────────────────────────────────────────────────────────┐ │   │
│  │   │ SwarmLead ─┬─► ContentSearcher (vector search)               │ │   │
│  │   │            ├─► RecommendationBuilder (GNN)                   │ │   │
│  │   │            ├─► AvailabilityChecker (rights validation)       │ │   │
│  │   │            └─► ContextKeeper (memory retrieval)              │ │   │
│  │   └──────────────────────────────────────────────────────────────┘ │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                              │                                              │
│                              ▼                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ PHASE R: REFINEMENT (100ms)                                          │   │
│  │   Agent: QualityAssurer                                             │   │
│  │   Actions:                                                          │   │
│  │     - Validate trust scores (filter < 0.6)                          │   │
│  │     - Remove duplicates                                             │   │
│  │     - Verify deep links                                             │   │
│  │     - Apply diversity (MMR)                                         │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                              │                                              │
│                              ▼                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ PHASE C: COMPLETION (50ms)                                           │   │
│  │   Agent: ResponseFormatter                                          │   │
│  │   Output:                                                           │   │
│  │     - Formatted results with explanations                           │   │
│  │     - "Because you watched Stranger Things..."                      │   │
│  │     - Trust indicators                                              │   │
│  │     - Deep links to platforms                                       │   │
│  │   Memory:                                                           │   │
│  │     - Store query pattern in ReasoningBank                          │   │
│  │     - Update user context in AgentDB                                │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  TOTAL LATENCY: ~430ms (streaming first results at 200ms)                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 14. Cross-Device Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         CROSS-DEVICE FLOW                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  SCENARIO: User searches on phone, watches on TV                           │
│                                                                             │
│  Phone (CLI/App)              PubNub                    Smart TV            │
│       │                          │                          │               │
│       │ 1. User searches         │                          │               │
│       │    "Dark"                │                          │               │
│       │                          │                          │               │
│       │ 2. Select "Watch on TV"  │                          │               │
│       │                          │                          │               │
│       │ 3. Publish to            │                          │               │
│       │    user.{id}.devices     │                          │               │
│       │ ────────────────────────>│                          │               │
│       │    {                     │                          │               │
│       │      action: "open",     │                          │               │
│       │      content_id: "xxx",  │                          │               │
│       │      platform: "netflix",│                          │               │
│       │      deep_link: "...",   │                          │               │
│       │      target: "living_room│                          │               │
│       │              _tv"        │                          │               │
│       │    }                     │                          │               │
│       │                          │                          │               │
│       │                          │ 4. Deliver to subscribed │               │
│       │                          │    TV device             │               │
│       │                          │ ────────────────────────>│               │
│       │                          │                          │               │
│       │                          │                    5. TV receives       │
│       │                          │                       command           │
│       │                          │                          │               │
│       │                          │                    6. Open Netflix      │
│       │                          │                       deep link         │
│       │                          │                          │               │
│       │                          │                    7. Start playback    │
│       │                          │                          │               │
│       │                          │ 8. Playback state        │               │
│       │                          │ <────────────────────────│               │
│       │                          │    {                     │               │
│       │                          │      content_id: "xxx",  │               │
│       │                          │      progress: 0,        │               │
│       │                          │      state: "playing"    │               │
│       │                          │    }                     │               │
│       │                          │                          │               │
│       │ 9. Receive playback state│                          │               │
│       │ <────────────────────────│                          │               │
│       │                          │                          │               │
│       │ 10. Update UI:           │                          │               │
│       │     "Now playing on      │                          │               │
│       │      Living Room TV"     │                          │               │
│       │                          │                          │               │
│                                                                             │
│  SYNC PROTOCOL:                                                             │
│  - All sync messages use CRDT format (LWW/OR-Set)                          │
│  - Conflict resolution: Last-Writer-Wins with HLC timestamps               │
│  - Offline queue: Messages queued locally, synced on reconnect             │
│  - Delta sync: Only changes transmitted, not full state                    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 15. CLI Structure

### 15.1 Technology Stack

```
RUST CLI STACK:
===============
- clap 4.4          # Argument parsing
- ratatui 0.25      # TUI framework
- crossterm 0.27    # Terminal backend
- tonic 0.10        # gRPC client
- tokio 1.35        # Async runtime
- serde/toml        # Configuration
- keyring 2.2       # Secure token storage
- oauth2 4.4        # OAuth2 client
```

### 15.2 Configuration

```toml
# ~/.config/tv-discover/config.toml

[profile]
active = "default"

[profiles.default]
region = "US"
default_platform = "netflix"

[profiles.default.subscriptions]
netflix = "premium"
disney_plus = "standard"
prime_video = "standard"

[auth]
refresh_token_path = "~/.config/tv-discover/tokens"

[display]
theme = "dark"
results_per_page = 20
show_trust_scores = true

[devices]
default_tv = "living_room_tv"
```

### 15.3 Example Usage

```bash
# Search
tv-discover search query "sci-fi like stranger things"
tv-discover search similar tt4574334 --limit 10

# Browse
tv-discover browse trending --region US
tv-discover browse expiring --platform netflix --days 30

# Recommendations
tv-discover recommend for-me --limit 20
tv-discover recommend mood "something light and funny"

# Watch
tv-discover watch now tt4574334 --device living_room_tv
tv-discover watch queue add tt4574334

# Devices
tv-discover devices list
tv-discover devices connect --type samsung-tizen
tv-discover devices cast tt4574334 --to living_room_tv

# Accounts
tv-discover accounts link netflix --tier premium
tv-discover accounts status

# Interactive TUI
tv-discover  # No args = launch TUI
```

---

## 16. Integration with hackathon-tv5

This section details the comprehensive integration with the [Agentics Foundation hackathon-tv5](https://github.com/agenticsorg/hackathon-tv5) toolkit, which provides the foundation for our agentic AI media discovery system.

### 16.1 hackathon-tv5 Overview

The hackathon-tv5 project addresses the **"45-minute decision problem"** — the time users waste deciding what to watch due to content fragmentation across streaming platforms.

**Key Components:**
- **CLI Tool** (`npx agentics-hackathon`): Project initialization, tool installation, MCP server
- **MCP Server**: 6 core tools with STDIO and SSE transport
- **ARW Specification**: Agent-Ready Web protocol for efficient AI-agent interaction
- **Demo Apps**: Media Discovery (Next.js) and ARW Chrome Extension
- **17+ Tools**: AI assistants, orchestration frameworks, databases, and cloud integrations

### 16.2 Integration Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        MEDIA GATEWAY + HACKATHON-TV5                             │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │                    USER EXPERIENCE LAYER                                 │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                   │    │
│  │  │ Media Gateway│  │ hackathon-tv5│  │  ARW Chrome  │                   │    │
│  │  │     CLI      │  │     CLI      │  │  Extension   │                   │    │
│  │  │  (Rust TUI)  │  │ (npx agentics)│  │ (Inspector)  │                   │    │
│  │  └──────────────┘  └──────────────┘  └──────────────┘                   │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                                      │                                           │
│  ┌───────────────────────────────────▼──────────────────────────────────────┐   │
│  │                    MCP PROTOCOL LAYER                                     │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                    │   │
│  │  │hackathon-tv5 │  │ Media Gateway│  │   Claude     │                    │   │
│  │  │  MCP Server  │  │ MCP Connectors│ │   Desktop    │                    │   │
│  │  │  (6 tools)   │  │ (10+ platforms)│ │ Integration  │                    │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘                    │   │
│  └──────────────────────────────────────────────────────────────────────────┘   │
│                                      │                                           │
│  ┌───────────────────────────────────▼──────────────────────────────────────┐   │
│  │                    ARW (AGENT-READY WEB) LAYER                            │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                    │   │
│  │  │   Machine    │  │    OAuth     │  │  AI-* HTTP   │                    │   │
│  │  │    Views     │  │   Actions    │  │   Headers    │                    │   │
│  │  │ (85% tokens) │  │ (Secure ops) │  │(Observability)│                    │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘                    │   │
│  └──────────────────────────────────────────────────────────────────────────┘   │
│                                      │                                           │
│  ┌───────────────────────────────────▼──────────────────────────────────────┐   │
│  │                    TOOL ECOSYSTEM (17+ Tools)                             │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │   │
│  │  │  Claude Flow │  │ Agentic Flow │  │   RuVector   │  │   AgentDB    │  │   │
│  │  │ (101 tools)  │  │ (66 agents)  │  │  (Hypergraph)│  │  (Context)   │  │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘  │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │   │
│  │  │  Google ADK  │  │  Vertex AI   │  │  SONA Engine │  │  SPARC 2.0   │  │   │
│  │  │              │  │     SDK      │  │ (39 attn mech)│ │              │  │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘  │   │
│  └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 16.3 ARW Specification Integration

The Agent-Ready Web (ARW) specification provides efficient agent-to-service communication:

| Benefit | Impact | Implementation |
|---------|--------|----------------|
| **85% Token Reduction** | Lower API costs | Machine Views instead of HTML scraping |
| **10x Faster Discovery** | Improved UX | Structured manifests |
| **OAuth-Enforced Actions** | Secure transactions | Platform auth integration |
| **AI-* Headers** | Traffic observability | Agent monitoring |

```rust
// ARW Machine View Integration
#[derive(Debug, Serialize, Deserialize)]
pub struct ARWMediaView {
    #[serde(rename = "@context")]
    pub context: String,  // "https://agentics.org/arw/v1"
    #[serde(rename = "@type")]
    pub content_type: String,

    pub id: String,
    pub title: String,
    pub availability: Vec<ARWPlatformAvailability>,

    // Pre-computed for semantic search
    pub embeddings: Option<Vec<f32>>,
    pub taxonomy: Vec<String>,
}

#[derive(Debug, Serialize, Deserialize)]
pub struct ARWHeaders {
    #[serde(rename = "AI-Request-ID")]
    pub request_id: String,
    #[serde(rename = "AI-Agent-Name")]
    pub agent_name: String,
    #[serde(rename = "AI-Purpose")]
    pub purpose: String,
    #[serde(rename = "AI-Token-Budget")]
    pub token_budget: Option<u32>,
}
```

### 16.4 hackathon-tv5 MCP Server Tools

| Tool | Purpose | Media Gateway Integration |
|------|---------|---------------------------|
| `get_hackathon_info` | Hackathon details | Project context |
| `get_tracks` | Competition tracks | Track selection |
| `get_available_tools` | Tool catalog | Tool discovery |
| `get_project_status` | Project state | Status monitoring |
| `check_tool_installed` | Tool verification | Dependency check |
| `get_resources` | Resource links | Documentation access |

### 16.5 Tool Ecosystem Integration

**Orchestration Tools (Layer-2):**

| Tool | Integration Point | Purpose |
|------|-------------------|---------|
| **Claude Flow** | Agent Orchestrator | 101 MCP tools, SPARC methodology |
| **Agentic Flow** | Agent Orchestrator | 66 specialized agents |
| **Flow Nexus** | Cross-Platform Orchestrator | Workflow coordination |
| **Google ADK** | Agent Orchestrator | Agent development kit |

**Data Tools (Data Layer):**

| Tool | Integration Point | Purpose |
|------|-------------------|---------|
| **RuVector** | Primary Data Engine | Hypergraph + Vector + GNN |
| **AgentDB** | Memory Layer | Agent context storage |

**Cloud Tools (Infrastructure):**

| Tool | Integration Point | Purpose |
|------|-------------------|---------|
| **Google Cloud CLI** | GCP Deployment | Infrastructure management |
| **Vertex AI SDK** | Intelligence Layer | ML model serving |

### 16.6 MCP Tool Extensions

```rust
// Extend hackathon-tv5 MCP server with streaming-specific tools

pub fn get_streaming_tools() -> Vec<MCPToolSchema> {
    vec![
        MCPToolSchema {
            name: "streaming_search".to_string(),
            description: "Search for content across all streaming platforms".to_string(),
            input_schema: json!({
                "type": "object",
                "properties": {
                    "query": { "type": "string" },
                    "platforms": { "type": "array", "items": { "type": "string" } },
                    "region": { "type": "string" },
                    "limit": { "type": "integer", "default": 20 }
                },
                "required": ["query"]
            }),
        },
        MCPToolSchema {
            name: "streaming_availability".to_string(),
            description: "Check where content is available to stream".to_string(),
            input_schema: json!({
                "type": "object",
                "properties": {
                    "content_id": { "type": "string" },
                    "region": { "type": "string" }
                },
                "required": ["content_id"]
            }),
        },
        MCPToolSchema {
            name: "streaming_recommend".to_string(),
            description: "Get personalized content recommendations".to_string(),
            input_schema: json!({
                "type": "object",
                "properties": {
                    "mood": { "type": "string" },
                    "genres": { "type": "array", "items": { "type": "string" } },
                    "similar_to": { "type": "string" },
                    "limit": { "type": "integer", "default": 10 }
                }
            }),
        },
    ]
}
```

### 16.7 Claude Desktop Configuration

```json
{
  "mcpServers": {
    "agentics-hackathon": {
      "command": "npx",
      "args": ["agentics-hackathon", "mcp"]
    },
    "media-gateway": {
      "command": "./media-gateway",
      "args": ["mcp", "--transport", "stdio"],
      "env": {
        "RUST_LOG": "info",
        "RUVECTOR_URL": "http://localhost:6333"
      }
    }
  }
}
```

### 16.8 Key Integration Points

| hackathon-tv5 Component | Media Gateway Layer | Integration |
|------------------------|---------------------|-------------|
| **ARW Machine Views** | Layer-3 Metadata Fabric | 85% token reduction for content ingestion |
| **MCP Server (6 tools)** | Layer-1 MCP Connectors | Protocol bridge for platform integration |
| **Claude Flow (101 tools)** | Layer-2 Agent Orchestrator | Multi-agent coordination with SPARC |
| **Agentic Flow (66 agents)** | Layer-2 Agent Orchestrator | Specialized agent workflows |
| **RuVector** | Data Layer | Hypergraph + Vector + GNN storage |
| **AgentDB** | Layer-2 Memory | Agent context and session state |
| **Google ADK** | Layer-2 Agent Orchestrator | Agent development framework |
| **Vertex AI SDK** | Layer-2 Intelligence | ML model serving integration |

### 16.9 Hackathon Track Alignment

| Track | Media Gateway Focus | Key Components |
|-------|--------------------|--------------------|
| **Entertainment Discovery** | Core functionality | Search, recommendations, availability |
| **Multi-Agent Systems** | Layer-2 Intelligence | Claude Flow, SONA, 9 specialized agents |
| **Agentic Workflows** | End-to-end flows | Auth → Search → Recommend → Sync |
| **Open Innovation** | Novel features | Cross-device sync, privacy-safe personalization |

> **Full Integration Documentation**: See [`HACKATHON_TV5_INTEGRATION.md`](HACKATHON_TV5_INTEGRATION.md) for complete integration specification, deployment configurations, and implementation examples.

---

## 17. Google Cloud Platform Deployment

### 17.1 Infrastructure Overview

The system is designed for production deployment on **Google Cloud Platform**, leveraging managed services for scalability, security, and operational efficiency.

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              GOOGLE CLOUD PLATFORM                                   │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                      │
│  ┌─────────────────────────────────────────────────────────────────────────────┐   │
│  │              GLOBAL LOAD BALANCER + CLOUD ARMOR                              │   │
│  │                    (WAF + DDoS Protection)                                   │   │
│  └──────────────────────────────┬──────────────────────────────────────────────┘   │
│                                 │                                                   │
│  ┌──────────────────────────────┼──────────────────────────────────────────────┐   │
│  │                              ▼                                               │   │
│  │  ┌────────────────┐    ┌────────────────┐    ┌────────────────┐             │   │
│  │  │   Cloud Run    │    │  GKE Autopilot │    │   Cloud Run    │             │   │
│  │  │  (API Gateway) │    │ (15+ Services) │    │   (Webhooks)   │             │   │
│  │  └────────────────┘    └────────────────┘    └────────────────┘             │   │
│  │                              │                                               │   │
│  │         ISTIO SERVICE MESH + WORKLOAD IDENTITY                              │   │
│  └──────────────────────────────┼──────────────────────────────────────────────┘   │
│                                 │                                                   │
│  ┌──────────────────────────────┼──────────────────────────────────────────────┐   │
│  │                              ▼                                               │   │
│  │  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐                 │   │
│  │  │   Cloud SQL    │  │  Memorystore   │  │    Pub/Sub     │                 │   │
│  │  │  PostgreSQL 15 │  │    Valkey      │  │    Topics      │                 │   │
│  │  └────────────────┘  └────────────────┘  └────────────────┘                 │   │
│  │                                                                              │   │
│  │              PRIVATE VPC + PRIVATE SERVICE ACCESS                           │   │
│  └──────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                      │
│  ┌──────────────────────────────────────────────────────────────────────────────┐   │
│  │  OBSERVABILITY: Cloud Logging │ Cloud Trace │ Cloud Monitoring               │   │
│  └──────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                      │
│  ┌──────────────────────────────────────────────────────────────────────────────┐   │
│  │  CI/CD: Cloud Build │ Artifact Registry │ Cloud Deploy │ Secret Manager     │   │
│  └──────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### 17.2 GCP Services Matrix

| Service | Purpose | Configuration |
|---------|---------|---------------|
| **GKE Autopilot** | Container orchestration for 15+ microservices | Regional, Workload Identity, Istio service mesh |
| **Cloud Run** | Stateless API gateway and webhook handlers | Auto-scaling 0-100, VPC connector, 80 concurrency |
| **Cloud SQL** | PostgreSQL 15 with HA | 4 vCPU, 16GB RAM, Private IP, PITR enabled |
| **Memorystore** | Redis-compatible caching (Valkey) | 10GB, Standard HA, Read replicas |
| **Pub/Sub** | Event streaming between services | Schema enforcement, Dead letter queues |
| **Cloud Armor** | WAF and DDoS protection | Adaptive protection, Rate limiting (1000 req/min) |
| **Secret Manager** | Credential management | Automatic rotation, Workload Identity access |
| **Artifact Registry** | Container image storage | Vulnerability scanning, Cross-region replication |
| **Cloud Build** | CI/CD pipeline | Multi-stage Rust builds with cargo-chef |
| **Cloud Deploy** | Kubernetes deployment automation | Progressive rollouts, Canary deployments |

### 17.3 Service Distribution

**GKE Autopilot** hosts stateful microservices:
- Layer 1: Ingestion, Auth Service, Sync Engine, Entity Resolver
- Layer 2: Recommendation Engine, Agent Orchestrator, Semantic Search
- Layer 3: Metadata Fabric, Availability Index, Rights Engine

**Cloud Run** hosts stateless components:
- API Gateway (public-facing, auto-scales to zero)
- Webhook Handlers (Pub/Sub push subscriptions)
- Batch Jobs (scheduled via Cloud Scheduler)

### 17.4 GKE Autopilot Configuration

```hcl
resource "google_container_cluster" "media_gateway" {
  name     = "media-gateway-cluster"
  location = var.region

  enable_autopilot = true

  workload_identity_config {
    workload_pool = "${var.project_id}.svc.id.goog"
  }

  private_cluster_config {
    enable_private_nodes    = true
    enable_private_endpoint = false
    master_ipv4_cidr_block  = "172.16.0.0/28"
  }

  addons_config {
    http_load_balancing {
      disabled = false
    }
    network_policy_config {
      disabled = false
    }
  }

  release_channel {
    channel = "REGULAR"
  }

  logging_config {
    enable_components = ["SYSTEM_COMPONENTS", "WORKLOADS"]
  }

  monitoring_config {
    enable_components = ["SYSTEM_COMPONENTS", "WORKLOADS"]
    managed_prometheus {
      enabled = true
    }
  }
}
```

### 17.5 Cloud Run API Gateway

```hcl
resource "google_cloud_run_v2_service" "api_gateway" {
  name     = "api-gateway"
  location = var.region

  template {
    scaling {
      min_instance_count = 1
      max_instance_count = 100
    }

    vpc_access {
      connector = google_vpc_access_connector.connector.id
      egress    = "PRIVATE_RANGES_ONLY"
    }

    containers {
      image = "${var.artifact_registry}/api-gateway:${var.image_tag}"

      resources {
        limits = {
          cpu    = "2"
          memory = "1Gi"
        }
        cpu_idle = true
      }

      startup_probe {
        http_get {
          path = "/health"
          port = 8080
        }
      }
    }

    service_account = google_service_account.api_gateway.email
    max_instance_request_concurrency = 80
  }
}
```

### 17.6 Database and Caching

**Cloud SQL PostgreSQL** (Primary Database):
```hcl
resource "google_sql_database_instance" "primary" {
  name             = "media-gateway-db"
  database_version = "POSTGRES_15"
  region           = var.region

  settings {
    tier              = "db-custom-4-16384"
    availability_type = "REGIONAL"
    disk_size         = 200
    disk_type         = "PD_SSD"

    backup_configuration {
      enabled                        = true
      point_in_time_recovery_enabled = true
      backup_retention_settings {
        retained_backups = 30
      }
    }

    ip_configuration {
      ipv4_enabled    = false
      private_network = google_compute_network.vpc.id
    }

    insights_config {
      query_insights_enabled  = true
      record_application_tags = true
    }
  }
}
```

**Memorystore Valkey** (Redis-Compatible Cache):
```hcl
resource "google_redis_instance" "cache" {
  name           = "media-gateway-cache"
  tier           = "STANDARD_HA"
  memory_size_gb = 10
  region         = var.region
  redis_version  = "REDIS_7_2"

  authorized_network      = google_compute_network.vpc.id
  connect_mode            = "PRIVATE_SERVICE_ACCESS"
  transit_encryption_mode = "SERVER_AUTHENTICATION"
  auth_enabled            = true
  replica_count           = 2
  read_replicas_mode      = "READ_REPLICAS_ENABLED"
}
```

### 17.7 Event Streaming with Pub/Sub

```hcl
# Content availability changes
resource "google_pubsub_topic" "content_availability" {
  name                       = "content-availability-changes"
  message_retention_duration = "86400s"

  schema_settings {
    schema   = google_pubsub_schema.content_availability.id
    encoding = "JSON"
  }
}

# Pull subscription for GKE services
resource "google_pubsub_subscription" "availability_processor" {
  name  = "availability-processor"
  topic = google_pubsub_topic.content_availability.name

  ack_deadline_seconds         = 60
  enable_exactly_once_delivery = true

  dead_letter_policy {
    dead_letter_topic     = google_pubsub_topic.dead_letter.id
    max_delivery_attempts = 5
  }
}
```

### 17.8 Security Architecture

**Workload Identity** (Keyless Authentication):
```hcl
resource "google_service_account" "layer1_sa" {
  account_id   = "media-gateway-layer1"
  display_name = "Media Gateway Layer 1 Services"
}

resource "google_service_account_iam_member" "layer1_workload_identity" {
  service_account_id = google_service_account.layer1_sa.name
  role               = "roles/iam.workloadIdentityUser"
  member             = "serviceAccount:${var.project_id}.svc.id.goog[media-gateway-layer1/layer1-ksa]"
}
```

**Cloud Armor Security Policy**:
```hcl
resource "google_compute_security_policy" "api_policy" {
  name = "media-gateway-api-policy"

  adaptive_protection_config {
    layer_7_ddos_defense_config {
      enable = true
    }
  }

  # XSS Protection
  rule {
    action   = "deny(403)"
    priority = 200
    match {
      expr {
        expression = "evaluatePreconfiguredExpr('xss-v33-stable')"
      }
    }
  }

  # SQL Injection Protection
  rule {
    action   = "deny(403)"
    priority = 201
    match {
      expr {
        expression = "evaluatePreconfiguredExpr('sqli-v33-stable')"
      }
    }
  }

  # Rate Limiting
  rule {
    action   = "rate_based_ban"
    priority = 300
    match {
      versioned_expr = "SRC_IPS_V1"
      config {
        src_ip_ranges = ["*"]
      }
    }
    rate_limit_options {
      conform_action = "allow"
      exceed_action  = "deny(429)"
      enforce_on_key = "IP"
      rate_limit_threshold {
        count        = 1000
        interval_sec = 60
      }
      ban_duration_sec = 600
    }
  }
}
```

### 17.9 CI/CD Pipeline

**Cloud Build Configuration** (cloudbuild.yaml):
```yaml
steps:
  # Build Rust services in parallel
  - name: 'gcr.io/cloud-builders/docker'
    id: 'build-ingestion'
    args: ['build', '-t', '${_ARTIFACT_REGISTRY}/ingestion-core:${SHORT_SHA}',
           '-f', 'layer-1/ingestion/Dockerfile', '.']
    waitFor: ['-']

  - name: 'gcr.io/cloud-builders/docker'
    id: 'build-recommendations'
    args: ['build', '-t', '${_ARTIFACT_REGISTRY}/recommendation-engine:${SHORT_SHA}',
           '-f', 'layer-2/recommendation-engine/Dockerfile', '.']
    waitFor: ['-']

  # Push images
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', '--all-tags', '${_ARTIFACT_REGISTRY}/ingestion-core']
    waitFor: ['build-ingestion']

  # Deploy to GKE
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    id: 'deploy-gke'
    entrypoint: 'bash'
    args:
      - '-c'
      - |
        gcloud container clusters get-credentials media-gateway-cluster --region ${_REGION}
        kubectl set image deployment/ingestion-core \
          ingestion-core=${_ARTIFACT_REGISTRY}/ingestion-core:${SHORT_SHA} \
          -n media-gateway-layer1
    waitFor: ['push-images']

options:
  machineType: 'E2_HIGHCPU_8'

timeout: 3600s
```

**Rust Dockerfile** (Distroless):
```dockerfile
FROM rust:1.75-slim as builder
WORKDIR /app
RUN cargo install cargo-chef
COPY . .
RUN cargo chef prepare --recipe-path recipe.json

FROM rust:1.75-slim as cacher
WORKDIR /app
RUN cargo install cargo-chef
COPY --from=builder /app/recipe.json recipe.json
RUN cargo chef cook --release --recipe-path recipe.json

FROM rust:1.75-slim as final-builder
WORKDIR /app
COPY . .
COPY --from=cacher /app/target target
RUN cargo build --release

FROM gcr.io/distroless/cc-debian12:nonroot
COPY --from=final-builder /app/target/release/service /app/service
USER nonroot:nonroot
EXPOSE 8080 50051
ENTRYPOINT ["/app/service"]
```

### 17.10 Observability Stack

**Cloud Logging + Monitoring**:
```hcl
# Log sink for BigQuery analytics
resource "google_logging_project_sink" "bigquery_sink" {
  name        = "media-gateway-logs-bq"
  destination = "bigquery.googleapis.com/projects/${var.project_id}/datasets/${google_bigquery_dataset.logs.dataset_id}"
  filter      = "resource.type=\"k8s_container\" AND resource.labels.namespace_name=~\"media-gateway-.*\""

  bigquery_options {
    use_partitioned_tables = true
  }
}

# Error rate alert
resource "google_monitoring_alert_policy" "high_error_rate" {
  display_name = "High Error Rate - Media Gateway"
  combiner     = "OR"

  conditions {
    display_name = "Error rate > 1%"
    condition_threshold {
      filter          = "metric.type=\"logging.googleapis.com/user/media-gateway/error-rate\""
      comparison      = "COMPARISON_GT"
      threshold_value = 0.01
      duration        = "300s"
    }
  }

  notification_channels = [google_monitoring_notification_channel.email.id]
}
```

### 17.11 Terraform Module Structure

```
terraform/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   └── prod/
└── modules/
    ├── gke/                 # GKE Autopilot cluster
    ├── cloudrun/            # Cloud Run services
    ├── database/            # Cloud SQL PostgreSQL
    ├── cache/               # Memorystore Valkey
    ├── pubsub/              # Pub/Sub topics and subscriptions
    ├── security/            # Workload Identity, Cloud Armor, Secrets
    ├── network/             # VPC, Subnets, Firewall
    └── observability/       # Logging, Monitoring, Alerting
```

### 17.12 Cost Estimates

| Component | Monthly Estimate |
|-----------|-----------------|
| GKE Autopilot (15 services, 2 replicas avg) | $800-1,200 |
| Cloud Run (API Gateway + Webhooks) | $200-400 |
| Cloud SQL (HA, 4 vCPU, 16GB) | $400-500 |
| Memorystore (10GB HA) | $300-350 |
| Pub/Sub (10M messages) | $50-100 |
| Cloud Armor + Logging + Monitoring | $100-150 |
| **Total** | **$1,850-2,700/month** |

### 17.13 Quick Deployment Commands

```bash
# Initialize Terraform
cd terraform/environments/dev
terraform init
terraform plan -out=tfplan
terraform apply tfplan

# Get GKE credentials
gcloud container clusters get-credentials media-gateway-cluster --region us-central1

# Deploy Kubernetes resources
kubectl apply -f k8s/namespaces.yaml
kubectl apply -f k8s/layer1/
kubectl apply -f k8s/layer2/
kubectl apply -f k8s/layer3/

# Deploy Cloud Run
gcloud run deploy api-gateway \
  --image us-central1-docker.pkg.dev/PROJECT_ID/media-gateway/api-gateway:latest \
  --region us-central1

# Run Cloud Build
gcloud builds submit --config cloudbuild.yaml

# Check deployment status
kubectl get pods -n media-gateway-layer1
kubectl get pods -n media-gateway-layer2
kubectl get pods -n media-gateway-layer3
```

> **Full GCP Deployment Documentation**: See [`GCP_DEPLOYMENT_ARCHITECTURE.md`](GCP_DEPLOYMENT_ARCHITECTURE.md) for complete Terraform modules, Kubernetes manifests, and operational procedures.

---

## 18. SONA Intelligence Engine Integration

SONA (Self-Optimizing Neural Architecture) provides runtime adaptation capabilities without retraining, enabling personalized recommendations that evolve with user preferences.

### 18.1 SONA Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         SONA INTELLIGENCE ENGINE                                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                      SEMANTIC ROUTING LAYER                              │   │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐         │   │
│  │  │  Tiny Dancer    │  │   Query Type    │  │   Confidence    │         │   │
│  │  │  (FastGRNN)     │──│   Classifier    │──│    Router       │         │   │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────┘         │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                      │                                          │
│  ┌───────────────────────────────────▼──────────────────────────────────────┐  │
│  │                      ATTENTION SELECTION (39 MECHANISMS)                  │  │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐             │  │
│  │  │   Core    │  │   Graph   │  │Specialized│  │Hyperbolic │             │  │
│  │  │ MultiHead │  │ GraphRoPE │  │  Sparse   │  │  expMap   │             │  │
│  │  │  Flash    │  │EdgeFeatured│ │  Cross    │  │  logMap   │             │  │
│  │  │  Linear   │  │   Node    │  │  Kernel   │  │ mobiusAdd │             │  │
│  │  └───────────┘  └───────────┘  └───────────┘  └───────────┘             │  │
│  └──────────────────────────────────────────────────────────────────────────┘  │
│                                      │                                          │
│  ┌───────────────────────────────────▼──────────────────────────────────────┐  │
│  │                      RUNTIME ADAPTATION LAYER                             │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐          │  │
│  │  │   Two-Tier      │  │     EWC++       │  │  ReasoningBank  │          │  │
│  │  │     LoRA        │  │   (Anti-forget) │  │    Storage      │          │  │
│  │  │  (rank 4-16)    │  │                 │  │                 │          │  │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────┘          │  │
│  └──────────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 18.2 Core SONA Components

| Component | Purpose | Integration Point |
|-----------|---------|-------------------|
| **Two-Tier LoRA** | Memory-efficient user personalization | Layer-3 Personalization Engine |
| **EWC++** | Prevents catastrophic forgetting | Model update pipeline |
| **Tiny Dancer** | FastGRNN semantic query routing | Layer-2 Orchestrator |
| **ReasoningBank** | Persistent reasoning patterns | Ruvector graph storage |
| **39 Attention Mechanisms** | Dynamic attention selection | Recommendation/Search engines |

### 18.3 Two-Tier LoRA Personalization

```rust
/// User-specific LoRA adapter for runtime personalization
pub struct UserLoRAAdapter {
    pub user_id: String,
    pub rank: usize,               // LoRA rank (4-16 typical)
    pub alpha: f32,                // Scaling factor
    pub a_matrices: Vec<Tensor>,   // Low-rank A matrices
    pub b_matrices: Vec<Tensor>,   // Low-rank B matrices
    pub interaction_count: u64,
    pub last_updated: DateTime<Utc>,
}

impl UserLoRAAdapter {
    /// Apply LoRA adaptation to base model weights
    pub fn apply(&self, base_weights: &Tensor) -> Tensor {
        let delta = self.compute_delta();
        base_weights + (self.alpha / self.rank as f32) * delta
    }

    /// Update adapter based on user interaction
    pub async fn update_from_interaction(&mut self, interaction: &UserInteraction) {
        // Incremental gradient update with EWC++ regularization
        let gradient = compute_interaction_gradient(interaction);
        let ewc_penalty = self.compute_ewc_penalty();
        self.apply_gradient(gradient - ewc_penalty);
    }
}
```

**Storage Strategy:**
- **Tier 1 (Hot)**: Active user adapters in Memorystore Valkey (~10KB per user)
- **Tier 2 (Warm)**: Recent adapters in Cloud SQL PostgreSQL
- **Tier 3 (Cold)**: Historical adapters in Cloud Storage

### 18.4 Tiny Dancer Semantic Router

Tiny Dancer uses FastGRNN (Fast Gated Recurrent Neural Network) for low-latency query classification and routing:

```rust
pub struct TinyDancerRouter {
    model: FastGRNN,
    query_types: Vec<QueryType>,
    confidence_threshold: f32,
}

#[derive(Clone, Debug)]
pub enum QueryType {
    ContentDiscovery,      // "Find me action movies"
    PersonalRecommendation,// "What should I watch tonight?"
    MetadataLookup,        // "Who directed Inception?"
    AvailabilityCheck,     // "Where can I watch Breaking Bad?"
    ComparisonQuery,       // "Netflix vs Disney+ for sci-fi"
    TrendingAnalysis,      // "What's popular this week?"
}

impl TinyDancerRouter {
    /// Route query to appropriate handler with confidence score
    pub async fn route(&self, query: &str) -> (QueryType, f32) {
        let embedding = self.model.encode(query);
        let (query_type, confidence) = self.classify(embedding);

        if confidence < self.confidence_threshold {
            return (QueryType::ContentDiscovery, confidence); // Default fallback
        }

        (query_type, confidence)
    }

    /// Select optimal attention mechanism for query type
    pub fn select_attention(&self, query_type: &QueryType) -> AttentionMechanism {
        match query_type {
            QueryType::ContentDiscovery => AttentionMechanism::FlashAttention,
            QueryType::PersonalRecommendation => AttentionMechanism::GraphRoPE,
            QueryType::MetadataLookup => AttentionMechanism::Linear,
            QueryType::AvailabilityCheck => AttentionMechanism::Sparse,
            QueryType::ComparisonQuery => AttentionMechanism::CrossAttention,
            QueryType::TrendingAnalysis => AttentionMechanism::MultiHeadGraph,
        }
    }
}
```

### 18.5 Attention Mechanism Selection

SONA provides 39 attention mechanisms grouped into four categories:

| Category | Mechanisms | Use Case |
|----------|------------|----------|
| **Core (12)** | MultiHead, Flash, Linear, Sliding Window, Local, Global, Chunked, Dilated, Strided, Relative, ALiBi, RoPE | Standard transformer operations |
| **Graph (10)** | GraphRoPE, EdgeFeatured, Node, Bipartite, Heterogeneous, Message Passing, GraphSAGE, GAT, GCN, GIN | Graph neural network queries |
| **Specialized (9)** | Sparse, Cross, Kernel, Landmark, Performer, Longformer, BigBird, Reformer, Linformer | Long-context and efficiency |
| **Hyperbolic (8)** | expMap, logMap, mobiusAddition, mobiusMatVec, hyperbolicMLR, poincareEmbedding, lorentzEmbedding, hyperbolicDistance | Hierarchical/tree structures |

### 18.6 ReasoningBank Integration

ReasoningBank stores learned reasoning patterns for reuse across similar queries:

```rust
pub struct ReasoningPattern {
    pub pattern_id: Uuid,
    pub query_signature: Vec<f32>,      // Query embedding fingerprint
    pub reasoning_steps: Vec<ReasoningStep>,
    pub outcome_quality: f32,           // User feedback score
    pub usage_count: u64,
    pub created_at: DateTime<Utc>,
}

pub struct ReasoningBank {
    ruvector: RuvectorClient,
    pattern_index: String,  // HNSW index for pattern lookup
}

impl ReasoningBank {
    /// Find similar reasoning patterns for query
    pub async fn find_patterns(&self, query_embedding: &[f32], k: usize) -> Vec<ReasoningPattern> {
        self.ruvector.search_similar(
            &self.pattern_index,
            query_embedding,
            k,
            SearchParams::default().with_ef(64)
        ).await
    }

    /// Store new successful reasoning pattern
    pub async fn store_pattern(&self, pattern: ReasoningPattern) {
        self.ruvector.upsert(&self.pattern_index, pattern).await;
    }
}
```

### 18.7 EWC++ Anti-Forgetting

Elastic Weight Consolidation (EWC++) prevents catastrophic forgetting when updating user adapters:

```rust
pub struct EWCRegularizer {
    fisher_information: HashMap<String, Tensor>,  // Per-parameter importance
    optimal_weights: HashMap<String, Tensor>,     // Previous optimal values
    lambda: f32,                                   // Regularization strength
}

impl EWCRegularizer {
    /// Compute EWC penalty for weight update
    pub fn compute_penalty(&self, current_weights: &HashMap<String, Tensor>) -> Tensor {
        let mut penalty = Tensor::zeros(&[1]);

        for (name, weight) in current_weights {
            if let Some(fisher) = self.fisher_information.get(name) {
                if let Some(optimal) = self.optimal_weights.get(name) {
                    let diff = weight - optimal;
                    penalty = penalty + (fisher * diff.pow(2)).sum();
                }
            }
        }

        self.lambda * penalty
    }
}
```

### 18.8 SONA GCP Deployment

| Component | GKE Service | Resources | Scaling |
|-----------|-------------|-----------|---------|
| Tiny Dancer Router | `sona-router` | 0.5 vCPU, 1GB RAM | HPA: 2-20 pods |
| LoRA Adapter Service | `sona-lora` | 2 vCPU, 4GB RAM | HPA: 3-15 pods |
| Attention Selector | `sona-attention` | 1 vCPU, 2GB RAM | HPA: 2-10 pods |
| ReasoningBank API | `sona-reasoning` | 1 vCPU, 2GB RAM | HPA: 2-8 pods |

**Estimated Additional Cost**: $300-500/month for SONA services

### 18.9 SONA Integration Flow

```
User Query
    │
    ▼
┌─────────────────┐
│  Tiny Dancer    │ ◄── Query classification
│  Router         │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  ReasoningBank  │ ◄── Pattern lookup
│  Lookup         │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Attention      │ ◄── Dynamic mechanism selection
│  Selector       │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  LoRA Adapter   │ ◄── User personalization
│  Application    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Base Model     │ ◄── Enhanced inference
│  Inference      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  EWC++ Update   │ ◄── Learn from interaction
│  (async)        │
└─────────────────┘
```

---

## 19. E2B Sandbox Integration

[E2B](https://e2b.dev) provides secure, isolated sandbox environments for executing AI-generated code. This is critical for our multi-agent system where LLM-generated code must run safely without compromising production infrastructure.

### 19.1 Why E2B for Agent Sandboxing

| Challenge | E2B Solution |
|-----------|--------------|
| **Untrusted Code Execution** | Firecracker microVM isolation (~150ms startup) |
| **Resource Exhaustion** | Configurable CPU/memory limits |
| **Network Security** | Sandboxed network with controlled egress |
| **Data Exfiltration** | Isolated filesystem, no host access |
| **Long-Running Tasks** | Sessions up to 24 hours |
| **Scale** | Thousands of concurrent sandboxes |

### 19.2 E2B Integration Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        LAYER 2: INTELLIGENCE + E2B                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │                   CLAUDE-FLOW ORCHESTRATOR                               │    │
│  │  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐           │    │
│  │  │ Discovery  │ │ Analysis   │ │ Recommend  │ │  Search    │           │    │
│  │  │   Agent    │ │   Agent    │ │   Agent    │ │   Agent    │           │    │
│  │  └─────┬──────┘ └─────┬──────┘ └─────┬──────┘ └─────┬──────┘           │    │
│  │        │              │              │              │                   │    │
│  │        └──────────────┴──────────────┴──────────────┘                   │    │
│  │                              │                                          │    │
│  │                    ┌─────────▼─────────┐                               │    │
│  │                    │  E2B SANDBOX      │                               │    │
│  │                    │    MANAGER        │                               │    │
│  │                    └─────────┬─────────┘                               │    │
│  └──────────────────────────────┼──────────────────────────────────────────┘    │
│                                 │                                               │
│  ┌──────────────────────────────▼──────────────────────────────────────────┐   │
│  │                         E2B CLOUD                                        │   │
│  │  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐            │   │
│  │  │   Code         │  │   Data         │  │   Desktop      │            │   │
│  │  │   Interpreter  │  │   Analysis     │  │   Sandbox      │            │   │
│  │  │   (Python/JS)  │  │   (Pandas)     │  │   (Browser)    │            │   │
│  │  └────────────────┘  └────────────────┘  └────────────────┘            │   │
│  └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 19.3 Sandbox Types

| Sandbox Type | Template | Use Case |
|--------------|----------|----------|
| **Code Interpreter** | `e2b/code-interpreter` | General agent code execution |
| **Data Analysis** | `media-gateway-analysis` | Recommendation computations |
| **Search Executor** | `media-gateway-search` | Vector similarity code |
| **Desktop/Browser** | `e2b/desktop` | Web scraping, metadata enrichment |
| **SONA Updater** | `sona-lora-updater` | Safe LoRA weight updates |

### 19.4 Agent Sandbox Workflow

```rust
/// Execute agent-generated code safely in E2B sandbox
pub async fn execute_safely(
    &self,
    task: &AgentTask,
) -> Result<AgentResult, AgentError> {
    // 1. Agent generates code via LLM
    let code = self.agent.generate_code(task).await?;

    // 2. Create isolated sandbox
    let sandbox = self.e2b.create_sandbox(&self.template).await?;

    // 3. Upload inputs
    self.e2b.upload_file(&sandbox.id, "/data/inputs.json", &task.inputs).await?;

    // 4. Execute in sandbox (isolated from production)
    let result = self.e2b.execute_code(&sandbox.id, &code, Language::Python).await?;

    // 5. Download outputs
    let outputs = self.e2b.download_file(&sandbox.id, "/output/results.json").await?;

    // 6. Cleanup
    self.e2b.terminate(&sandbox.id).await?;

    Ok(AgentResult { output: outputs, logs: result.stdout })
}
```

### 19.5 Security Configuration

```rust
pub struct SandboxSecurityConfig {
    pub internet_access: bool,        // Default: false
    pub allowed_hosts: Vec<String>,   // Allowlist for external access
    pub timeout_seconds: u32,         // Max execution time
    pub memory_mb: u32,               // Memory limit
    pub cpu_cores: u8,                // CPU limit
    pub max_upload_bytes: u64,        // Input size limit
    pub max_download_bytes: u64,      // Output size limit
}
```

### 19.6 E2B Cost Estimate

| Usage | Monthly Estimate |
|-------|-----------------|
| Sandbox Hours (~1,000) | $200-400 |
| Data Transfer | ~$50 |
| **Total E2B** | **$250-450** |

> **Full E2B Documentation**: See [`E2B_SANDBOX_INTEGRATION.md`](E2B_SANDBOX_INTEGRATION.md) for complete implementation details, custom templates, and GKE deployment configuration.

---

## 20. Implementation Roadmap

### Phase 1: Foundation
- [ ] Set up multi-repo structure with Cargo workspace
- [ ] Deploy Ruvector cluster (dev environment)
- [ ] Implement basic hypergraph schema
- [ ] Create synthetic catalog generator (10K+ content)
- [ ] Build auth-service with OAuth 2.0 + PKCE
- [ ] Implement basic CLI skeleton (clap + ratatui)

### Phase 2: Core Services (Weeks 5-8)
- [ ] Implement MCP connectors for aggregators (JustWatch, Watchmode)
- [ ] Build metadata-normalizer service
- [ ] Implement entity-resolver with fuzzy matching
- [ ] Create semantic-search service with Ruvector
- [ ] Develop basic recommendation engine
- [ ] Build device-gateway for cross-device sync

### Phase 3: Intelligence (Weeks 9-12)
- [ ] Implement GNN-enhanced recommendations (GraphSAGE)
- [ ] Build Claude-Flow agent orchestrator
- [ ] Integrate SPARC methodology
- [ ] Implement trust-scoring system
- [ ] Create user-preferences service with local storage
- [ ] Build federated learning infrastructure

### Phase 4: Integration (Weeks 13-16)
- [ ] Complete CLI with all features and TUI
- [ ] Build TV apps (Samsung, LG, Roku)
- [ ] Develop mobile apps (iOS, Android)
- [ ] Implement PubNub cross-device sync
- [ ] Create web application
- [ ] Build monitoring and observability stack

### Phase 5: Production (Weeks 17-20)
- [ ] Performance optimization (< 100ms p99 latency)
- [ ] Security audit and penetration testing
- [ ] Load testing (100K concurrent users)
- [ ] Documentation and API reference
- [ ] Beta launch preparation
- [ ] Production deployment

---

## Appendix A: Technology Stack Summary

| Category | Technology | Purpose |
|----------|------------|---------|
| Language | Rust | Primary implementation (100%) |
| Data | Ruvector | Hypergraph + Vector + GNN |
| Intelligence | SONA | Self-Optimizing Neural Architecture |
| Sandboxing | E2B | Firecracker microVM agent isolation |
| Adaptation | Two-Tier LoRA | Runtime user personalization |
| Routing | Tiny Dancer | FastGRNN semantic routing |
| Attention | 39 Mechanisms | Dynamic attention selection |
| Learning | EWC++ | Anti-catastrophic forgetting |
| Patterns | ReasoningBank | Reasoning pattern storage |
| Foundation | hackathon-tv5 | ARW spec, MCP server, 17+ tools |
| Protocol | ARW | Agent-Ready Web (85% token reduction) |
| MCP | hackathon-tv5 MCP | 6 core tools, STDIO/SSE transport |
| Tools | Claude Flow | 101 MCP tools, orchestration |
| Tools | Agentic Flow | 66 specialized agents |
| Real-Time | PubNub | Cross-device sync |
| Events | Google Pub/Sub | Event streaming |
| Cache | Memorystore (Valkey) | Session/query cache + LoRA adapters |
| Database | Cloud SQL PostgreSQL | Persistent storage |
| Auth | Custom OAuth 2.0 | Authentication |
| Agents | Claude-Flow | Multi-agent orchestration |
| Search | Ruvector + Semantic | Hybrid search |
| ML | PyTorch (via tch) | Model serving |
| Gateway | Cloud Run | API gateway |
| Mesh | Istio | Service mesh |
| Container | GKE Autopilot | Orchestration |
| CI/CD | Cloud Build + Deploy | Automation |
| Monitoring | Cloud Monitoring | Observability |
| Tracing | Cloud Trace | Distributed tracing |
| Security | Cloud Armor | WAF + DDoS |
| Secrets | Secret Manager | Credential management |
| Registry | Artifact Registry | Container storage |

---

## Appendix B: Performance Targets

| Metric | Target |
|--------|--------|
| Search latency (p50) | < 50ms |
| Search latency (p99) | < 200ms |
| Recommendation latency (p50) | < 100ms |
| Recommendation latency (p99) | < 500ms |
| CLI startup time | < 100ms |
| Cross-device sync delay | < 2s |
| Throughput | 10K RPS |
| Availability | 99.9% |

---

## Appendix C: Key Differentiators

1. **100% Rust Implementation**: Performance, safety, and consistency
2. **Developer CLI**: Powerful TUI for platform operators and developers
3. **Privacy-First**: Federated learning, no PII collection
4. **Hypergraph Power**: Complex multi-way relationships via Ruvector
5. **Real-Time Sync**: Seamless cross-device experience via PubNub
6. **Trust Transparency**: Every recommendation includes quality score
7. **Simulation-Ready**: Works immediately with synthetic data
8. **MCP-Native**: Full Model Context Protocol integration
9. **Multi-Agent Intelligence**: 9 specialized agents for complex queries
10. **Explainable Recommendations**: Graph paths show "why"
11. **Cloud-Native GCP**: Production-grade infrastructure with GKE Autopilot
12. **Zero-Trust Security**: Workload Identity, Cloud Armor, private networking
13. **SONA Runtime Adaptation**: Personalization without retraining via Two-Tier LoRA
14. **39 Attention Mechanisms**: Dynamic selection for optimal query processing
15. **Anti-Forgetting**: EWC++ ensures stable long-term learning
16. **hackathon-tv5 Foundation**: ARW specification with 85% token reduction
17. **17+ Integrated Tools**: Claude Flow, Agentic Flow, RuVector, AgentDB ecosystem
18. **E2B Sandboxed Execution**: Firecracker microVM isolation for safe agent code execution

---

## Appendix D: GCP Infrastructure Summary

| Service | Configuration | Monthly Cost |
|---------|---------------|--------------|
| GKE Autopilot | 15 services, regional | $800-1,200 |
| Cloud Run | API Gateway, 0-100 instances | $200-400 |
| Cloud SQL | PostgreSQL 15, HA, 4 vCPU | $400-500 |
| Memorystore | Valkey 10GB, HA, 2 replicas | $300-350 |
| Pub/Sub | 10M messages/month | $50-100 |
| Cloud Armor | WAF + DDoS, adaptive | $50 |
| Secret Manager | 20 secrets | $5 |
| Artifact Registry | 50GB images | $5 |
| Cloud Logging | 100GB/month | $50 |
| **Total** | | **$1,850-2,700** |

**Key GCP Features Used:**
- Workload Identity for keyless authentication
- Private Service Access for database connectivity
- VPC-native clusters with pod security policies
- Managed Prometheus for metrics collection
- Cloud Deploy for progressive rollouts
- Binary Authorization for supply chain security

---

## Appendix E: SONA Intelligence Summary

| Component | Technology | Purpose | Integration |
|-----------|------------|---------|-------------|
| Two-Tier LoRA | Low-Rank Adaptation | Per-user personalization | Memorystore + Cloud SQL |
| EWC++ | Elastic Weight Consolidation | Anti-forgetting | Model update pipeline |
| Tiny Dancer | FastGRNN | Query routing | Layer-2 Orchestrator |
| ReasoningBank | Pattern Storage | Reasoning reuse | Ruvector HNSW |
| Attention (39) | Dynamic Selection | Optimal processing | Search/Rec engines |

**SONA Performance Targets:**

| Metric | Target |
|--------|--------|
| Query routing latency | < 5ms |
| LoRA application | < 10ms |
| Attention selection | < 2ms |
| Pattern lookup | < 15ms |
| EWC update (async) | < 100ms |

**SONA Cost Estimate (Monthly):**

| Service | Cost |
|---------|------|
| SONA Router (GKE) | $100-150 |
| LoRA Adapter Service | $100-200 |
| Attention Selector | $50-75 |
| ReasoningBank API | $50-75 |
| **Total SONA** | **$300-500** |

> **Full SONA Documentation**: See [`SONA_INTEGRATION_SPECIFICATION.md`](SONA_INTEGRATION_SPECIFICATION.md) for complete implementation details, Rust code examples, and deployment configurations.

---

*Document Version: 1.4.0*
*Last Updated: December 2025*
*Authors: 9-Agent Architecture Swarm + GCP Integration + SONA Intelligence + hackathon-tv5 + E2B Sandboxing*
