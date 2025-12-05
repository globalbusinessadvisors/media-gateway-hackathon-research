# Global TV Discovery System - Architecture Blueprint

## Executive Summary

This document presents a comprehensive architecture blueprint for a global, cross-platform TV discovery system designed to eliminate the 45 minutes people waste daily deciding what to watch. The system unifies content from Netflix, Prime Video, Disney+, Hulu, Apple TV+, YouTube, Crave, and other streaming platforms into a seamless discovery experience operated through a unified CLI.

**Core Technology Stack:**
- **Data Layer:** Ruvector (vector + graph engine with GNN support)
- **Orchestration:** Claude-Flow with SPARC methodology
- **CLI Framework:** Bubble Tea (Go) + Commander.js (Node.js)
- **Service Mesh:** Linkerd
- **Event Streaming:** Apache Kafka
- **Search:** Ruvector semantic search + Elasticsearch

---

## Table of Contents

1. [Multi-Repository Layout](#1-multi-repository-layout)
2. [Ruvector Data Layer Architecture](#2-ruvector-data-layer-architecture)
3. [Cross-Kingdom Data Integration Strategy](#3-cross-kingdom-data-integration-strategy)
4. [Trust-Scoring Methodology](#4-trust-scoring-methodology)
5. [Privacy-Safe Personalization Plan](#5-privacy-safe-personalization-plan)
6. [Agent Coordination Model](#6-agent-coordination-model)
7. [Unified CLI Architecture](#7-unified-cli-architecture)
8. [Device Integration Architecture](#8-device-integration-architecture)
9. [Authentication Flow](#9-authentication-flow)
10. [Implementation Roadmap](#10-implementation-roadmap)

---

## 1. Multi-Repository Layout

### 1.1 Repository Organization Strategy

We adopt a **hybrid multi-repo approach** with a shared infrastructure monorepo:

```
tv-discovery/
├── infrastructure/          # Monorepo for shared infrastructure
│   ├── packages/
│   │   ├── proto/          # gRPC/Protobuf definitions
│   │   ├── sdk-node/       # Node.js SDK
│   │   ├── sdk-go/         # Go SDK
│   │   ├── shared-types/   # TypeScript type definitions
│   │   └── ruvector-client/# Ruvector client library
│   ├── deploy/
│   │   ├── kubernetes/     # K8s manifests
│   │   ├── helm/           # Helm charts
│   │   └── terraform/      # Infrastructure as Code
│   └── tools/
│       ├── cli-core/       # CLI shared components
│       └── testing/        # Integration test harness
│
├── services/               # Individual service repos (multi-repo)
│   ├── ingestion-netflix/
│   ├── ingestion-prime/
│   ├── ingestion-disney/
│   ├── ingestion-aggregator/
│   ├── metadata-normalizer/
│   ├── entity-resolver/
│   ├── rights-validator/
│   ├── auth-service/
│   ├── vector-indexer/
│   ├── knowledge-graph/
│   ├── semantic-search/
│   ├── recommendation-engine/
│   ├── user-preferences/
│   ├── agent-orchestrator/
│   └── device-gateway/
│
├── clients/                # Client applications (multi-repo)
│   ├── cli-unified/        # Main CLI application
│   ├── web-app/
│   ├── mobile-ios/
│   ├── mobile-android/
│   └── tv-apps/
│
└── simulation/             # Simulation repos for development
    ├── catalog-simulator/
    └── aggregator-mock/
```

### 1.2 Detailed Repository Specifications

#### Core Services

| Repository | Purpose | Technology | Dependencies |
|------------|---------|------------|--------------|
| `ingestion-netflix` | Simulates Netflix catalog ingestion | Go, Kafka | proto, ruvector-client |
| `ingestion-prime` | Simulates Prime Video ingestion | Go, Kafka | proto, ruvector-client |
| `ingestion-disney` | Simulates Disney+ ingestion | Go, Kafka | proto, ruvector-client |
| `ingestion-aggregator` | Integrates with JustWatch/Watchmode APIs | Go, Kafka | proto, ruvector-client |
| `metadata-normalizer` | Normalizes metadata to unified schema | Rust | proto, ruvector-client |
| `entity-resolver` | Deduplication and entity matching | Python, ML | proto, ruvector-client |
| `rights-validator` | Regional rights and availability | Go | proto, ruvector-client |
| `auth-service` | OAuth 2.0 authentication | Go, JWT | proto |
| `vector-indexer` | Manages Ruvector vector indexes | Rust | ruvector |
| `knowledge-graph` | Builds content relationship graphs | Rust | ruvector |
| `semantic-search` | Semantic query processing | Rust | ruvector |
| `recommendation-engine` | Hybrid recommendation system | Python, PyTorch | ruvector-client |
| `user-preferences` | User preference modeling | Go | proto, ruvector-client |
| `agent-orchestrator` | Claude-Flow agent coordination | TypeScript | claude-flow |
| `device-gateway` | Device communication hub | Go, WebSocket | proto |

#### Shared Infrastructure

| Package | Purpose | Technology |
|---------|---------|------------|
| `proto` | gRPC service definitions | Protocol Buffers |
| `sdk-node` | Node.js service SDK | TypeScript |
| `sdk-go` | Go service SDK | Go |
| `shared-types` | Common type definitions | TypeScript |
| `ruvector-client` | Ruvector client wrapper | Rust/Go/TS bindings |

### 1.3 Versioning Strategy

**Semantic Versioning with BOM (Bill of Materials):**

```yaml
# infrastructure/versions.yaml
bom:
  version: "1.0.0"
  services:
    ingestion-netflix: "^1.2.0"
    metadata-normalizer: "^2.0.0"
    entity-resolver: "^1.5.0"
    ruvector: "^0.8.0"
  infrastructure:
    kubernetes: "1.28"
    linkerd: "2.14"
    kafka: "3.6"
```

**Release Train Model:**
- Major releases: Quarterly
- Minor releases: Bi-weekly
- Patch releases: As needed
- All services maintain backward compatibility for 2 major versions

---

## 2. Ruvector Data Layer Architecture

### 2.1 Core Schema Design

Ruvector's hypergraph capabilities allow modeling complex multi-way relationships:

```
┌─────────────────────────────────────────────────────────────────┐
│                    RUVECTOR SCHEMA                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  NODES (with Vector Embeddings):                                │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │   Content   │  │   Person    │  │    User     │             │
│  │ - id        │  │ - id        │  │ - id        │             │
│  │ - title     │  │ - name      │  │ - profile   │             │
│  │ - type      │  │ - bio       │  │ - prefs     │             │
│  │ - embedding │  │ - embedding │  │ - embedding │             │
│  │   [1536]    │  │   [768]     │  │   [512]     │             │
│  └─────────────┘  └─────────────┘  └─────────────┘             │
│                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │   Genre     │  │  Platform   │  │   Region    │             │
│  │ - id        │  │ - id        │  │ - code      │             │
│  │ - name      │  │ - name      │  │ - name      │             │
│  │ - embedding │  │ - type      │  │ - timezone  │             │
│  │   [256]     │  │             │  │             │             │
│  └─────────────┘  └─────────────┘  └─────────────┘             │
│                                                                 │
│  HYPEREDGES (Multi-way Relationships):                          │
│  ┌──────────────────────────────────────────────┐              │
│  │  AVAILABILITY_WINDOW                          │              │
│  │  Connects: Content × Platform × Region ×      │              │
│  │            TimeWindow × LicenseTerms          │              │
│  │  Properties:                                  │              │
│  │  - start_date, end_date                       │              │
│  │  - quality_tiers: [SD, HD, 4K, HDR]          │              │
│  │  - pricing_model: [SVOD, TVOD, AVOD, FREE]   │              │
│  │  - exclusivity: boolean                       │              │
│  └──────────────────────────────────────────────┘              │
│                                                                 │
│  STANDARD EDGES:                                                │
│  - ACTED_IN (Person → Content)                                 │
│  - DIRECTED (Person → Content)                                 │
│  - BELONGS_TO (Content → Genre)                                │
│  - SIMILAR_TO (Content → Content, similarity_score)            │
│  - WATCHED (User → Content, timestamp, progress)               │
│  - RATED (User → Content, rating, timestamp)                   │
│  - FOLLOWS (User → User)                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Cypher Query Patterns

**Multi-hop Content Discovery:**
```cypher
// Find available shows similar to user's watched content
MATCH (u:User {id: $userId})-[:WATCHED]->(watched:Content)
MATCH (watched)-[:SIMILAR_TO]->(recommended:Content)
MATCH (recommended)-[:BELONGS_TO]->(g:Genre)
WHERE NOT (u)-[:WATCHED]->(recommended)
  AND EXISTS {
    MATCH (recommended)-[:AVAILABLE_IN]->(:AvailabilityWindow {
      region: $userRegion,
      end_date: > datetime()
    })
  }
WITH recommended,
     vector.similarity(u.embedding, recommended.embedding) AS userSim,
     collect(DISTINCT g.name) AS genres,
     count(watched) AS basedOnCount
ORDER BY userSim * basedOnCount DESC
LIMIT 20
RETURN recommended, genres, userSim, basedOnCount
```

**Cross-Platform Availability Query:**
```cypher
// Find where content is available across all platforms
MATCH (c:Content {id: $contentId})
MATCH (c)-[avail:AVAILABLE_IN]->(window:AvailabilityWindow)
MATCH (window)-[:ON_PLATFORM]->(p:Platform)
MATCH (window)-[:IN_REGION]->(r:Region)
WHERE r.code = $regionCode
  AND window.end_date > datetime()
RETURN p.name AS platform,
       window.quality_tiers AS quality,
       window.pricing_model AS pricing,
       window.start_date AS availableFrom,
       window.end_date AS availableUntil
ORDER BY
  CASE window.pricing_model
    WHEN 'INCLUDED' THEN 1
    WHEN 'FREE' THEN 2
    WHEN 'RENT' THEN 3
    WHEN 'BUY' THEN 4
  END
```

### 2.3 GNN-Enhanced Recommendations

Ruvector's built-in GNN layers enable self-improving recommendations:

```
┌─────────────────────────────────────────────────────────────┐
│              GNN RECOMMENDATION PIPELINE                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. FEATURE COLLECTION                                      │
│     ┌──────────┐    ┌──────────┐    ┌──────────┐           │
│     │ Content  │    │  User    │    │ Context  │           │
│     │ Features │    │ Features │    │ Features │           │
│     │ - genre  │    │ - history│    │ - time   │           │
│     │ - cast   │    │ - prefs  │    │ - device │           │
│     │ - mood   │    │ - social │    │ - mood   │           │
│     └────┬─────┘    └────┬─────┘    └────┬─────┘           │
│          │               │               │                  │
│          └───────────────┼───────────────┘                  │
│                          ▼                                  │
│  2. GRAPH CONSTRUCTION                                      │
│     ┌─────────────────────────────────────┐                │
│     │  Heterogeneous Graph                 │                │
│     │  - User nodes                        │                │
│     │  - Content nodes                     │                │
│     │  - Interaction edges (WATCHED, RATED)│                │
│     │  - Similarity edges (SIMILAR_TO)     │                │
│     └─────────────────┬───────────────────┘                │
│                       ▼                                     │
│  3. GNN LAYERS (GraphSAGE in Ruvector)                     │
│     ┌─────────────────────────────────────┐                │
│     │  Layer 1: Neighbor Aggregation       │                │
│     │  Layer 2: Feature Transformation     │                │
│     │  Layer 3: Attention Weighting        │                │
│     └─────────────────┬───────────────────┘                │
│                       ▼                                     │
│  4. OUTPUT EMBEDDINGS                                       │
│     ┌─────────────────────────────────────┐                │
│     │  User embedding ⊗ Content embeddings │                │
│     │  = Personalized ranking scores       │                │
│     └─────────────────────────────────────┘                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 2.4 Tiny Dancer (FastGRNN) Agent Routing

Ruvector's built-in AI agent router uses FastGRNN for intelligent query routing:

```
┌─────────────────────────────────────────────────────────────┐
│              TINY DANCER ROUTING ARCHITECTURE                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  User Query: "Find sci-fi shows like Stranger Things        │
│               available on my subscriptions"                 │
│                          │                                  │
│                          ▼                                  │
│  ┌───────────────────────────────────────────┐             │
│  │         QUERY CLASSIFIER (FastGRNN)        │             │
│  │         - P50 latency: 309µs               │             │
│  │         - Trained on query patterns        │             │
│  └───────────────────────────────────────────┘             │
│                          │                                  │
│          ┌───────────────┼───────────────┐                 │
│          ▼               ▼               ▼                 │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐       │
│  │ VECTOR PATH  │ │ GRAPH PATH   │ │ HYBRID PATH  │       │
│  │              │ │              │ │              │       │
│  │ Semantic     │ │ Relationship │ │ Combined     │       │
│  │ similarity   │ │ traversal    │ │ approach     │       │
│  │ search       │ │ (Cypher)     │ │              │       │
│  └──────┬───────┘ └──────┬───────┘ └──────┬───────┘       │
│         │                │                │                │
│         └────────────────┼────────────────┘                │
│                          ▼                                  │
│  ┌───────────────────────────────────────────┐             │
│  │         RESULT AGGREGATOR                  │             │
│  │         - Rank fusion                      │             │
│  │         - Availability filtering           │             │
│  │         - Personalization boost            │             │
│  └───────────────────────────────────────────┘             │
│                          │                                  │
│                          ▼                                  │
│                    Ranked Results                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 2.5 Aggregator Infrastructure Simulation

Since most streaming platforms lack public APIs, we simulate their infrastructure for development:

```
┌─────────────────────────────────────────────────────────────┐
│           AGGREGATOR SIMULATION ARCHITECTURE                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  SIMULATION MODE (Development/Testing):                     │
│  ┌─────────────────────────────────────────────┐           │
│  │  Catalog Simulator                           │           │
│  │  - Generates realistic content catalogs      │           │
│  │  - Simulates update patterns (daily drops)   │           │
│  │  - Models regional availability variations   │           │
│  │  - Mimics rights expiration/acquisition      │           │
│  └──────────────────────┬──────────────────────┘           │
│                         │                                   │
│                         ▼                                   │
│  ┌─────────────────────────────────────────────┐           │
│  │  Ingestion Pipeline (Same as Production)     │           │
│  │  - Content ingestion adapters                │           │
│  │  - Metadata normalization                    │           │
│  │  - Entity resolution                         │           │
│  │  - Ruvector indexing                         │           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
│  PRODUCTION MODE (With Real Data):                          │
│  ┌─────────────────────────────────────────────┐           │
│  │  Real Data Sources                           │           │
│  │  - Streaming Availability API (JustWatch)    │           │
│  │  - Watchmode API                             │           │
│  │  - TMDb for metadata enrichment              │           │
│  │  - User-authorized platform data             │           │
│  └──────────────────────┬──────────────────────┘           │
│                         │                                   │
│                         ▼                                   │
│  ┌─────────────────────────────────────────────┐           │
│  │  SAME Ingestion Pipeline                     │           │
│  │  (Minimal configuration changes)             │           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
│  SWITCHOVER STRATEGY:                                       │
│  1. Feature flags control data source selection             │
│  2. Gradual migration: simulated → real per platform        │
│  3. A/B testing between simulated and real data             │
│  4. Fallback to simulation if real source fails             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Cross-Kingdom Data Integration Strategy

### 3.1 Data Domain Definitions

```
┌─────────────────────────────────────────────────────────────┐
│                    DATA KINGDOMS                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐                                            │
│  │  CONTENT    │  Movies, TV Shows, Episodes, Seasons       │
│  │  KINGDOM    │  Metadata: title, description, cast,       │
│  │             │  crew, ratings, runtime, release date      │
│  └──────┬──────┘                                            │
│         │                                                   │
│  ┌──────┴──────┐                                            │
│  │  USER       │  Profiles, Preferences, Watch History,     │
│  │  KINGDOM    │  Ratings, Watchlists, Social Connections   │
│  └──────┬──────┘                                            │
│         │                                                   │
│  ┌──────┴──────┐                                            │
│  │  PLATFORM   │  Streaming Services, Pricing Models,       │
│  │  KINGDOM    │  Quality Tiers, Features, Integrations     │
│  └──────┬──────┘                                            │
│         │                                                   │
│  ┌──────┴──────┐                                            │
│  │  RIGHTS     │  Licensing Agreements, Regional            │
│  │  KINGDOM    │  Availability, Time Windows, Exclusivity   │
│  └──────┬──────┘                                            │
│         │                                                   │
│  ┌──────┴──────┐                                            │
│  │  DEVICE     │  Capabilities, Sync State, Playback        │
│  │  KINGDOM    │  History, Preferences, Authentication      │
│  └─────────────┘                                            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 Integration Event Bus Architecture

```
┌─────────────────────────────────────────────────────────────┐
│              EVENT-DRIVEN INTEGRATION                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                    KAFKA TOPICS                             │
│                                                             │
│  ┌─────────────────────────────────────────────┐           │
│  │  content.ingested                            │           │
│  │  content.updated                             │           │
│  │  content.metadata.enriched                   │           │
│  │  content.entity.resolved                     │           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
│  ┌─────────────────────────────────────────────┐           │
│  │  user.preference.updated                     │           │
│  │  user.watch.started                          │           │
│  │  user.watch.completed                        │           │
│  │  user.rating.submitted                       │           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
│  ┌─────────────────────────────────────────────┐           │
│  │  rights.availability.changed                 │           │
│  │  rights.window.expiring                      │           │
│  │  rights.window.expired                       │           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
│  ┌─────────────────────────────────────────────┐           │
│  │  device.connected                            │           │
│  │  device.playback.state                       │           │
│  │  device.sync.requested                       │           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
│  CONSUMER GROUPS:                                           │
│  ┌─────────────────────────────────────────────┐           │
│  │  - ruvector-indexer (content.*, user.*)      │           │
│  │  - recommendation-engine (user.*, content.*) │           │
│  │  - notification-service (rights.*)           │           │
│  │  - sync-service (device.*, user.watch.*)     │           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 3.3 Data Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CROSS-KINGDOM DATA FLOW                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  EXTERNAL SOURCES          PROCESSING              RUVECTOR         │
│                                                                     │
│  ┌──────────┐                                                       │
│  │ JustWatch│─────┐                                                 │
│  │   API    │     │                                                 │
│  └──────────┘     │        ┌──────────────┐                        │
│                   ├───────▶│   Ingestion  │                        │
│  ┌──────────┐     │        │   Service    │                        │
│  │ Watchmode│─────┤        └──────┬───────┘                        │
│  │   API    │     │               │                                 │
│  └──────────┘     │               ▼                                 │
│                   │        ┌──────────────┐     ┌──────────────┐   │
│  ┌──────────┐     │        │  Metadata    │────▶│   Content    │   │
│  │  TMDb    │─────┘        │  Normalizer  │     │   Kingdom    │   │
│  │   API    │              └──────────────┘     │  (Ruvector)  │   │
│  └──────────┘                     │             └──────────────┘   │
│                                   ▼                                 │
│                           ┌──────────────┐                         │
│                           │   Entity     │                         │
│  ┌──────────┐             │  Resolver    │                         │
│  │  User    │             └──────────────┘                         │
│  │ Actions  │                    │                                  │
│  └────┬─────┘                    │                                  │
│       │                          ▼                                  │
│       │               ┌──────────────────┐    ┌──────────────┐    │
│       └──────────────▶│    User Prefs    │───▶│    User      │    │
│                       │    Service       │    │   Kingdom    │    │
│                       └──────────────────┘    │  (Ruvector)  │    │
│                                               └──────────────┘    │
│                                                                     │
│  ┌──────────┐        ┌──────────────┐        ┌──────────────┐     │
│  │ Platform │───────▶│   Rights     │───────▶│   Rights     │     │
│  │   Data   │        │  Validator   │        │   Kingdom    │     │
│  └──────────┘        └──────────────┘        │  (Ruvector)  │     │
│                                               └──────────────┘     │
│                                                                     │
│  ┌──────────┐        ┌──────────────┐        ┌──────────────┐     │
│  │  Device  │───────▶│   Device     │───────▶│   Device     │     │
│  │  Events  │        │   Gateway    │        │   Kingdom    │     │
│  └──────────┘        └──────────────┘        │  (Ruvector)  │     │
│                                               └──────────────┘     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 4. Trust-Scoring Methodology

### 4.1 Trust Score Components

```
┌─────────────────────────────────────────────────────────────┐
│                 TRUST SCORING SYSTEM                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  OVERALL TRUST SCORE = weighted average of:                 │
│                                                             │
│  ┌─────────────────────────────────────────────┐           │
│  │  1. SOURCE RELIABILITY (weight: 0.25)        │           │
│  │     - API uptime history                     │           │
│  │     - Data freshness (last update time)      │           │
│  │     - Historical accuracy rate               │           │
│  │     - Official vs unofficial source          │           │
│  │                                              │           │
│  │  Score = (uptime × 0.3) + (freshness × 0.3)  │           │
│  │        + (accuracy × 0.3) + (official × 0.1) │           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
│  ┌─────────────────────────────────────────────┐           │
│  │  2. METADATA ACCURACY (weight: 0.25)         │           │
│  │     - Cross-source validation (TMDb, IMDb)   │           │
│  │     - Field completeness score               │           │
│  │     - User-reported corrections              │           │
│  │     - Entity resolution confidence           │           │
│  │                                              │           │
│  │  Score = (validation × 0.4) + (complete × 0.3)│          │
│  │        + (corrections × 0.2) + (entity × 0.1)│           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
│  ┌─────────────────────────────────────────────┐           │
│  │  3. AVAILABILITY CONFIDENCE (weight: 0.20)   │           │
│  │     - Platform confirmation status           │           │
│  │     - Deep link validation                   │           │
│  │     - User-reported availability             │           │
│  │     - Recency of availability check          │           │
│  │                                              │           │
│  │  Score = (platform × 0.4) + (deeplink × 0.3) │           │
│  │        + (userReport × 0.2) + (recency × 0.1)│           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
│  ┌─────────────────────────────────────────────┐           │
│  │  4. RECOMMENDATION QUALITY (weight: 0.15)    │           │
│  │     - Click-through rate                     │           │
│  │     - Watch completion rate                  │           │
│  │     - User rating correlation                │           │
│  │     - A/B test performance                   │           │
│  │                                              │           │
│  │  Score = (CTR × 0.25) + (completion × 0.35)  │           │
│  │        + (rating × 0.25) + (AB × 0.15)       │           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
│  ┌─────────────────────────────────────────────┐           │
│  │  5. USER PREFERENCE CONFIDENCE (weight: 0.15)│           │
│  │     - Interaction count (more = higher)      │           │
│  │     - Preference consistency                 │           │
│  │     - Recency decay (recent = higher)        │           │
│  │     - Explicit vs implicit signals           │           │
│  │                                              │           │
│  │  Score = (count × 0.25) + (consistency × 0.25)│          │
│  │        + (recency × 0.25) + (explicit × 0.25)│           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 Trust Score Storage in Ruvector

```cypher
// Trust score as node properties
CREATE (content:Content {
  id: "tt1234567",
  title: "Example Show",
  trust_score: 0.87,
  trust_breakdown: {
    source_reliability: 0.92,
    metadata_accuracy: 0.88,
    availability_confidence: 0.85,
    recommendation_quality: 0.82,
    preference_confidence: 0.78
  },
  trust_updated_at: datetime()
})

// Trust-aware query
MATCH (c:Content)
WHERE c.trust_score > 0.7
  AND c.trust_breakdown.availability_confidence > 0.8
RETURN c
ORDER BY c.trust_score DESC
```

---

## 5. Privacy-Safe Personalization Plan

### 5.1 Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│           PRIVACY-PRESERVING PERSONALIZATION                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  PRINCIPLE: Data minimization + Local-first processing      │
│                                                             │
│  ┌─────────────────────────────────────────────┐           │
│  │  TIER 1: ON-DEVICE (Most Private)            │           │
│  │  - Full watch history                        │           │
│  │  - Detailed preferences                      │           │
│  │  - Viewing patterns                          │           │
│  │  - Personal embeddings                       │           │
│  │                                              │           │
│  │  Storage: Device local storage (encrypted)   │           │
│  │  Processing: On-device ML models             │           │
│  └─────────────────────────────────────────────┘           │
│                          │                                  │
│                          │ Federated Learning               │
│                          │ (model updates only)             │
│                          ▼                                  │
│  ┌─────────────────────────────────────────────┐           │
│  │  TIER 2: AGGREGATED (Privacy-Preserving)     │           │
│  │  - Anonymized preference clusters            │           │
│  │  - Differential privacy aggregates           │           │
│  │  - Global model weights                      │           │
│  │                                              │           │
│  │  Storage: Ruvector (no PII)                  │           │
│  │  Processing: Secure aggregation              │           │
│  └─────────────────────────────────────────────┘           │
│                          │                                  │
│                          ▼                                  │
│  ┌─────────────────────────────────────────────┐           │
│  │  TIER 3: PUBLIC SIGNALS (Non-Private)        │           │
│  │  - Content metadata                          │           │
│  │  - Aggregate popularity scores               │           │
│  │  - Public ratings and reviews                │           │
│  │                                              │           │
│  │  Storage: Ruvector (public data)             │           │
│  │  Processing: Standard algorithms             │           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 5.2 Federated Learning Implementation

```
┌─────────────────────────────────────────────────────────────┐
│           FEDERATED LEARNING PIPELINE                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  STEP 1: MODEL DISTRIBUTION                                 │
│  ┌─────────────────┐                                        │
│  │  Central Server  │                                        │
│  │  (Our Backend)   │                                        │
│  └────────┬────────┘                                        │
│           │ Global Model (v1)                               │
│           ▼                                                 │
│  ┌────────┬────────┬────────┬────────┐                     │
│  │Device 1│Device 2│Device 3│Device N│                     │
│  └────────┴────────┴────────┴────────┘                     │
│                                                             │
│  STEP 2: LOCAL TRAINING                                     │
│  Each device trains on LOCAL data:                          │
│  - Watch history                                            │
│  - Ratings                                                  │
│  - Search queries                                           │
│  - Dwell time                                               │
│                                                             │
│  STEP 3: GRADIENT COLLECTION (with Differential Privacy)    │
│  ┌────────┬────────┬────────┬────────┐                     │
│  │ ∇L₁ +ε│ ∇L₂ +ε│ ∇L₃ +ε│ ∇Lₙ +ε│  (Laplace noise)      │
│  └───┬────┴───┬────┴───┬────┴───┬────┘                     │
│      │        │        │        │                          │
│      └────────┴────────┴────────┘                          │
│                   │                                         │
│                   ▼                                         │
│  ┌─────────────────────────────────────┐                   │
│  │     Secure Aggregation Server        │                   │
│  │  - Aggregates encrypted gradients    │                   │
│  │  - Cannot see individual updates     │                   │
│  │  - Produces global model update      │                   │
│  └─────────────────────────────────────┘                   │
│                   │                                         │
│                   ▼                                         │
│  STEP 4: MODEL UPDATE                                       │
│  ┌─────────────────┐                                        │
│  │  Global Model   │                                        │
│  │  (v2)           │                                        │
│  └─────────────────┘                                        │
│                                                             │
│  PRIVACY GUARANTEES:                                        │
│  - ε-differential privacy (ε = 1.0)                         │
│  - Minimum 1000 devices per aggregation                     │
│  - No raw data leaves device                                │
│  - Gradient clipping to bound sensitivity                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 5.3 Local Preference Storage

```typescript
// On-device preference schema
interface LocalUserPreferences {
  // Encrypted with device-specific key
  userId: string; // Hashed, non-reversible

  // Genre preferences (local computation)
  genreScores: Map<string, number>;

  // Watch history (never sent to server)
  watchHistory: WatchEvent[];

  // Computed embeddings (used for local recommendations)
  userEmbedding: Float32Array; // 512-dim

  // Federated learning state
  localModelVersion: number;
  gradientBuffer: Float32Array[]; // Accumulated gradients

  // Sync state
  lastSyncTimestamp: Date;
}

// What gets sent to server (privacy-safe)
interface SyncPayload {
  // Anonymized user cluster ID (k-anonymity, k >= 50)
  clusterHint: string;

  // Model gradients with differential privacy noise
  encryptedGradients: EncryptedData;

  // Generic preference signals (no specific content)
  preferenceSignals: {
    averageWatchDuration: number; // Bucketed
    activeTimeOfDay: string; // Bucketed
    deviceType: string;
  };
}
```

---

## 6. Agent Coordination Model

### 6.1 Claude-Flow Integration Architecture

```
┌─────────────────────────────────────────────────────────────┐
│           MULTI-AGENT ORCHESTRATION (Claude-Flow)            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  SPARC METHODOLOGY INTEGRATION:                             │
│                                                             │
│  ┌─────────────────────────────────────────────┐           │
│  │  S - SPECIFICATION PHASE                     │           │
│  │      Agent: Requirements Analyst             │           │
│  │      - Parses user query intent              │           │
│  │      - Identifies required data domains      │           │
│  │      - Determines output format              │           │
│  └─────────────────┬───────────────────────────┘           │
│                    ▼                                        │
│  ┌─────────────────────────────────────────────┐           │
│  │  P - PSEUDOCODE PHASE                        │           │
│  │      Agent: Query Planner                    │           │
│  │      - Designs query execution plan          │           │
│  │      - Identifies parallel opportunities     │           │
│  │      - Plans fallback strategies             │           │
│  └─────────────────┬───────────────────────────┘           │
│                    ▼                                        │
│  ┌─────────────────────────────────────────────┐           │
│  │  A - ARCHITECTURE PHASE                      │           │
│  │      Agent: Execution Orchestrator           │           │
│  │      - Spawns specialized worker agents      │           │
│  │      - Manages agent communication           │           │
│  │      - Coordinates parallel execution        │           │
│  └─────────────────┬───────────────────────────┘           │
│                    ▼                                        │
│  ┌─────────────────────────────────────────────┐           │
│  │  R - REFINEMENT PHASE                        │           │
│  │      Agent: Quality Assurer                  │           │
│  │      - Validates results against spec        │           │
│  │      - Applies trust scoring                 │           │
│  │      - Handles edge cases                    │           │
│  └─────────────────┬───────────────────────────┘           │
│                    ▼                                        │
│  ┌─────────────────────────────────────────────┐           │
│  │  C - COMPLETION PHASE                        │           │
│  │      Agent: Response Formatter               │           │
│  │      - Formats final output                  │           │
│  │      - Generates explanations                │           │
│  │      - Updates ReasoningBank memory          │           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 6.2 Agent Types and Responsibilities

```
┌─────────────────────────────────────────────────────────────┐
│                    AGENT REGISTRY                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  COORDINATOR AGENTS:                                        │
│  ┌─────────────────────────────────────────────┐           │
│  │  SwarmLead                                   │           │
│  │  - Manages overall task execution            │           │
│  │  - Delegates to specialist agents            │           │
│  │  - Monitors progress and handles failures    │           │
│  │  - Tools: swarm_monitor, agent_assign,       │           │
│  │           task_create, memory_store          │           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
│  SPECIALIST AGENTS:                                         │
│  ┌─────────────────────────────────────────────┐           │
│  │  ContentSearcher                             │           │
│  │  - Executes semantic search queries          │           │
│  │  - Interfaces with Ruvector                  │           │
│  │  - Applies availability filters              │           │
│  │  - Tools: ruvector_search, filter_apply      │           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
│  ┌─────────────────────────────────────────────┐           │
│  │  RecommendationBuilder                       │           │
│  │  - Generates personalized recommendations    │           │
│  │  - Combines collaborative + content-based    │           │
│  │  - Applies user preference model             │           │
│  │  - Tools: gnn_recommend, preference_model    │           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
│  ┌─────────────────────────────────────────────┐           │
│  │  AvailabilityChecker                         │           │
│  │  - Validates content availability            │           │
│  │  - Checks regional restrictions              │           │
│  │  - Verifies deep links                       │           │
│  │  - Tools: rights_check, deeplink_validate    │           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
│  ┌─────────────────────────────────────────────┐           │
│  │  DeviceCoordinator                           │           │
│  │  - Manages device communication              │           │
│  │  - Handles playback control                  │           │
│  │  - Synchronizes state across devices         │           │
│  │  - Tools: device_send, sync_state            │           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
│  MEMORY AGENTS:                                             │
│  ┌─────────────────────────────────────────────┐           │
│  │  ContextKeeper (AgentDB-backed)              │           │
│  │  - Maintains conversation context            │           │
│  │  - Stores user session state                 │           │
│  │  - Retrieves relevant history                │           │
│  │  - Tools: memory_store, memory_retrieve      │           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
│  ┌─────────────────────────────────────────────┐           │
│  │  PatternLearner (ReasoningBank-backed)       │           │
│  │  - Identifies recurring query patterns       │           │
│  │  - Optimizes future responses                │           │
│  │  - Performs reflexion and self-improvement   │           │
│  │  - Tools: pattern_store, reflexion_update    │           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 6.3 Agent Communication via Hooks

```typescript
// Agent hook definitions for inter-agent communication
interface AgentHooks {
  // Pre-execution hooks
  beforeSearch: (query: SearchQuery) => Promise<SearchQuery>;
  beforeRecommend: (context: UserContext) => Promise<UserContext>;

  // Post-execution hooks
  afterSearch: (results: SearchResults) => Promise<SearchResults>;
  afterRecommend: (recs: Recommendations) => Promise<Recommendations>;

  // Memory hooks
  onMemoryStore: (key: string, value: any) => Promise<void>;
  onMemoryRetrieve: (key: string) => Promise<any>;

  // Error hooks
  onError: (error: Error, context: ExecutionContext) => Promise<void>;
}

// Example: Search agent with hooks
const searchAgent = {
  name: "ContentSearcher",
  hooks: {
    beforeSearch: async (query) => {
      // Enrich query with user context from memory
      const userContext = await memoryAgent.retrieve("user/context");
      return { ...query, userContext };
    },
    afterSearch: async (results) => {
      // Apply trust scoring to results
      return results.map(r => ({
        ...r,
        trustScore: await trustScorer.score(r)
      }));
    }
  }
};
```

### 6.4 Stream-JSON Chaining for Real-Time Updates

```
┌─────────────────────────────────────────────────────────────┐
│           STREAM-JSON AGENT CHAINING                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  User Query: "What should I watch tonight?"                 │
│                          │                                  │
│                          ▼                                  │
│  ┌──────────────────────────────────────┐                  │
│  │  SwarmLead (Orchestrator)             │                  │
│  │  Spawns parallel agent chain          │                  │
│  └──────────────────────────────────────┘                  │
│              │              │              │                │
│              ▼              ▼              ▼                │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐       │
│  │ ContentSearch│ │ Recommend    │ │ Availability │       │
│  │    Agent     │ │   Agent      │ │   Agent      │       │
│  └──────┬───────┘ └──────┬───────┘ └──────┬───────┘       │
│         │                │                │                │
│         │ Stream         │ Stream         │ Stream         │
│         │ Results        │ Results        │ Results        │
│         │                │                │                │
│         └────────────────┼────────────────┘                │
│                          │                                  │
│                          ▼                                  │
│  ┌──────────────────────────────────────┐                  │
│  │  Result Merger Agent                  │                  │
│  │  - Receives streaming results         │                  │
│  │  - Merges and deduplicates            │                  │
│  │  - Ranks by relevance + availability  │                  │
│  │  - Streams to client as ready         │                  │
│  └──────────────────────────────────────┘                  │
│                          │                                  │
│                          ▼                                  │
│  ┌──────────────────────────────────────┐                  │
│  │  CLI/Client                           │                  │
│  │  - Receives progressive results       │                  │
│  │  - Updates UI incrementally           │                  │
│  │  - Shows "thinking" indicators        │                  │
│  └──────────────────────────────────────┘                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 7. Unified CLI Architecture

### 7.1 Command Structure

```
tv-discover/
├── search      # Semantic content search
│   ├── query   # Natural language search
│   ├── filter  # Filtered search
│   └── similar # Find similar content
│
├── browse      # Interactive browsing
│   ├── trending    # What's popular now
│   ├── new         # Recently added
│   ├── expiring    # Leaving soon
│   └── categories  # Browse by genre/mood
│
├── recommend   # Personalized recommendations
│   ├── for-me      # Based on preferences
│   ├── mood        # Mood-based suggestions
│   └── social      # What friends are watching
│
├── watch       # Playback control
│   ├── now         # Start watching (opens platform)
│   ├── queue       # Add to watch queue
│   └── list        # Manage watchlist
│
├── devices     # Device management
│   ├── list        # Show connected devices
│   ├── connect     # Connect new device
│   ├── sync        # Sync watch progress
│   └── cast        # Cast to device
│
├── accounts    # Platform account management
│   ├── list        # Show linked accounts
│   ├── link        # Link new platform
│   ├── unlink      # Remove platform
│   └── status      # Check account status
│
├── preferences # User preferences
│   ├── genres      # Genre preferences
│   ├── platforms   # Platform priorities
│   ├── region      # Regional settings
│   └── export      # Export preferences
│
└── config      # CLI configuration
    ├── init        # Initialize configuration
    ├── set         # Set config value
    └── show        # Show current config
```

### 7.2 Interactive TUI Design (Bubble Tea)

```
┌─────────────────────────────────────────────────────────────┐
│  TV DISCOVER                          Region: US │ ▶ Netflix │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  🔍 Search: sci-fi shows like stranger things_              │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ RESULTS (12 found)                          Trust: ★★★★ │
│  ├─────────────────────────────────────────────────────┤   │
│  │ ▶ Dark (2017)                                       │   │
│  │   German mystery thriller with sci-fi elements      │   │
│  │   ⭐ 8.7 │ 3 Seasons │ Netflix                      │   │
│  │   [Watch Now] [Add to List] [More Info]             │   │
│  ├─────────────────────────────────────────────────────┤   │
│  │   The OA (2016)                                     │   │
│  │   Mind-bending supernatural drama                   │   │
│  │   ⭐ 7.8 │ 2 Seasons │ Netflix                      │   │
│  │   [Watch Now] [Add to List] [More Info]             │   │
│  ├─────────────────────────────────────────────────────┤   │
│  │   Black Mirror (2011)                               │   │
│  │   Anthology series exploring tech and society       │   │
│  │   ⭐ 8.8 │ 6 Seasons │ Netflix                      │   │
│  │   [Watch Now] [Add to List] [More Info]             │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ↑/↓: Navigate │ Enter: Select │ f: Filter │ ?: Help       │
└─────────────────────────────────────────────────────────────┘
```

### 7.3 CLI Implementation Architecture

```go
// Bubble Tea model structure
package main

import (
    tea "github.com/charmbracelet/bubbletea"
    "github.com/charmbracelet/bubbles/list"
    "github.com/charmbracelet/bubbles/textinput"
)

type Model struct {
    // UI State
    searchInput  textinput.Model
    resultsList  list.Model
    deviceList   list.Model

    // Application State
    currentView  View
    user         *UserContext
    searchQuery  string
    results      []ContentResult

    // Agent Connection
    agentClient  *AgentOrchestratorClient

    // Streaming State
    streaming    bool
    streamChan   chan StreamUpdate
}

type View int

const (
    ViewSearch View = iota
    ViewBrowse
    ViewDevice
    ViewPreferences
)

func (m Model) Init() tea.Cmd {
    return tea.Batch(
        textinput.Blink,
        m.loadUserContext,
        m.connectAgentOrchestrator,
    )
}

func (m Model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    switch msg := msg.(type) {
    case tea.KeyMsg:
        switch msg.String() {
        case "enter":
            if m.currentView == ViewSearch {
                return m, m.executeSearch
            }
        case "ctrl+c", "q":
            return m, tea.Quit
        }
    case SearchResultsMsg:
        // Handle streaming search results
        m.results = append(m.results, msg.Results...)
        m.resultsList.SetItems(toListItems(m.results))
    case StreamCompleteMsg:
        m.streaming = false
    }

    // Update active component
    var cmd tea.Cmd
    switch m.currentView {
    case ViewSearch:
        m.searchInput, cmd = m.searchInput.Update(msg)
    case ViewBrowse:
        m.resultsList, cmd = m.resultsList.Update(msg)
    }

    return m, cmd
}

func (m Model) View() string {
    // Render based on current view
    switch m.currentView {
    case ViewSearch:
        return m.renderSearchView()
    case ViewBrowse:
        return m.renderBrowseView()
    case ViewDevice:
        return m.renderDeviceView()
    default:
        return m.renderSearchView()
    }
}
```

### 7.4 CLI-to-Backend Communication

```
┌─────────────────────────────────────────────────────────────┐
│           CLI COMMUNICATION ARCHITECTURE                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  CLI (Bubble Tea)                                           │
│       │                                                     │
│       │ gRPC (streaming)                                    │
│       ▼                                                     │
│  ┌─────────────────────────────────────────────┐           │
│  │  API Gateway                                 │           │
│  │  - Authentication validation                 │           │
│  │  - Rate limiting                             │           │
│  │  - Request routing                           │           │
│  └─────────────────────────────────────────────┘           │
│       │                                                     │
│       │ gRPC                                                │
│       ▼                                                     │
│  ┌─────────────────────────────────────────────┐           │
│  │  Agent Orchestrator Service                  │           │
│  │  (Claude-Flow)                               │           │
│  │  - Spawns agents for query                   │           │
│  │  - Manages agent lifecycle                   │           │
│  │  - Streams results back                      │           │
│  └─────────────────────────────────────────────┘           │
│       │                                                     │
│       │ Internal gRPC / Kafka                               │
│       ▼                                                     │
│  ┌─────────────┬─────────────┬─────────────┐               │
│  │ Semantic    │ Recommend   │ Device      │               │
│  │ Search      │ Engine      │ Gateway     │               │
│  │ Service     │ Service     │ Service     │               │
│  └─────────────┴─────────────┴─────────────┘               │
│       │                                                     │
│       ▼                                                     │
│  ┌─────────────────────────────────────────────┐           │
│  │           RUVECTOR DATA LAYER                │           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 8. Device Integration Architecture

### 8.1 Supported Platforms

```
┌─────────────────────────────────────────────────────────────┐
│               DEVICE INTEGRATION MATRIX                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  SMART TVs:                                                 │
│  ┌──────────────┬────────────┬─────────────────────┐       │
│  │ Platform     │ Protocol   │ Features            │       │
│  ├──────────────┼────────────┼─────────────────────┤       │
│  │ Samsung Tizen│ WebSocket  │ Deep link, playback │       │
│  │ LG webOS     │ WebSocket  │ Deep link, playback │       │
│  │ Android TV   │ ADB/Cast   │ Full integration    │       │
│  │ Roku         │ ECP        │ Deep link, playback │       │
│  │ Fire TV      │ ADB        │ Deep link, playback │       │
│  │ Apple TV     │ AirPlay    │ Cast only           │       │
│  │ Google TV    │ Cast       │ Full integration    │       │
│  └──────────────┴────────────┴─────────────────────┘       │
│                                                             │
│  MOBILE:                                                    │
│  ┌──────────────┬────────────┬─────────────────────┐       │
│  │ Platform     │ Protocol   │ Features            │       │
│  ├──────────────┼────────────┼─────────────────────┤       │
│  │ iOS          │ REST/WS    │ Full app            │       │
│  │ Android      │ REST/WS    │ Full app            │       │
│  └──────────────┴────────────┴─────────────────────┘       │
│                                                             │
│  OTHER:                                                     │
│  ┌──────────────┬────────────┬─────────────────────┐       │
│  │ Platform     │ Protocol   │ Features            │       │
│  ├──────────────┼────────────┼─────────────────────┤       │
│  │ Web Browser  │ REST/WS    │ Full web app        │       │
│  │ Desktop CLI  │ gRPC       │ Full CLI            │       │
│  └──────────────┴────────────┴─────────────────────┘       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 8.2 Device Gateway Architecture

```
┌─────────────────────────────────────────────────────────────┐
│               DEVICE GATEWAY SERVICE                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────┐           │
│  │          CONNECTION MANAGER                  │           │
│  │  - Maintains WebSocket connections           │           │
│  │  - Handles device authentication             │           │
│  │  - Manages connection pools                  │           │
│  └──────────────────────┬──────────────────────┘           │
│                         │                                   │
│        ┌────────────────┼────────────────┐                 │
│        ▼                ▼                ▼                 │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐             │
│  │  TV      │    │  Mobile  │    │  Browser │             │
│  │  Adapter │    │  Adapter │    │  Adapter │             │
│  │          │    │          │    │          │             │
│  │ Tizen    │    │  iOS     │    │ WebSocket│             │
│  │ webOS    │    │  Android │    │          │             │
│  │ Roku ECP │    │          │    │          │             │
│  │ Fire TV  │    │          │    │          │             │
│  │ Cast     │    │          │    │          │             │
│  └──────────┘    └──────────┘    └──────────┘             │
│                                                             │
│  ┌─────────────────────────────────────────────┐           │
│  │          SYNC ENGINE                         │           │
│  │  - Watch progress synchronization            │           │
│  │  - Watchlist sync                            │           │
│  │  - Preference sync                           │           │
│  │  - Conflict resolution (last-write-wins)     │           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
│  ┌─────────────────────────────────────────────┐           │
│  │          PLAYBACK CONTROLLER                 │           │
│  │  - Deep link generation                      │           │
│  │  - Playback state management                 │           │
│  │  - Cast/stream initiation                    │           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 8.3 Cross-Device Synchronization Protocol

```
┌─────────────────────────────────────────────────────────────┐
│           SYNC PROTOCOL (CRDT-based)                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  DATA STRUCTURES (Conflict-free):                           │
│                                                             │
│  WatchProgress (LWW-Register):                              │
│  {                                                          │
│    contentId: "tt1234567",                                  │
│    progress: 1847,  // seconds                              │
│    duration: 3600,                                          │
│    timestamp: 1701792000000,  // for conflict resolution    │
│    deviceId: "device-123"                                   │
│  }                                                          │
│                                                             │
│  Watchlist (OR-Set):                                        │
│  {                                                          │
│    additions: [                                             │
│      { contentId: "tt1234567", timestamp: ..., device: ...} │
│    ],                                                       │
│    removals: [                                              │
│      { contentId: "tt7654321", timestamp: ..., device: ...} │
│    ]                                                        │
│  }                                                          │
│                                                             │
│  SYNC FLOW:                                                 │
│                                                             │
│  Device A                Server              Device B       │
│     │                      │                    │           │
│     │──── Connect ────────▶│                    │           │
│     │◀─── State Vector ────│                    │           │
│     │                      │                    │           │
│     │──── Delta Update ───▶│                    │           │
│     │                      │──── Push Update ──▶│           │
│     │                      │                    │           │
│     │                      │◀─── Ack ──────────│           │
│     │◀─── Confirm ─────────│                    │           │
│     │                      │                    │           │
│  (Real-time sync via WebSocket)                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 9. Authentication Flow

### 9.1 OAuth 2.0 with PKCE (System Authentication)

```
┌─────────────────────────────────────────────────────────────┐
│           AUTHENTICATION ARCHITECTURE                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  IMPORTANT: We authenticate to OUR system, NOT platforms    │
│  Platform content is accessed via deep links, not APIs      │
│                                                             │
│  ┌─────────────────────────────────────────────┐           │
│  │  FLOW 1: Standard OAuth 2.0 + PKCE          │           │
│  │  (Web, Mobile, Desktop with browser)        │           │
│  │                                              │           │
│  │  1. Generate code_verifier (random)          │           │
│  │  2. Generate code_challenge = SHA256(verifier)│          │
│  │  3. Redirect to /authorize with challenge    │           │
│  │  4. User authenticates                       │           │
│  │  5. Receive authorization code               │           │
│  │  6. Exchange code + verifier for tokens      │           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
│  ┌─────────────────────────────────────────────┐           │
│  │  FLOW 2: Device Authorization Grant         │           │
│  │  (Smart TVs, CLI, devices without browser)  │           │
│  │                                              │           │
│  │  1. Device requests device code              │           │
│  │  2. Display user_code to user                │           │
│  │  3. User visits verification URI on phone    │           │
│  │  4. User enters code and authenticates       │           │
│  │  5. Device polls for token (every 5 sec)     │           │
│  │  6. Receive access + refresh tokens          │           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 9.2 Device Authorization Flow (RFC 8628)

```
┌─────────────────────────────────────────────────────────────┐
│           DEVICE AUTHORIZATION FLOW                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  TV/CLI Device              Auth Server            Phone    │
│       │                          │                   │      │
│       │─── POST /device/code ───▶│                   │      │
│       │                          │                   │      │
│       │◀── {                     │                   │      │
│       │      device_code: "...", │                   │      │
│       │      user_code: "WDJB-MJHT",                 │      │
│       │      verification_uri:   │                   │      │
│       │        "https://tv.discover/activate",       │      │
│       │      expires_in: 1800,   │                   │      │
│       │      interval: 5         │                   │      │
│       │    } ────────────────────│                   │      │
│       │                          │                   │      │
│  ┌────┴────┐                     │                   │      │
│  │ Display │                     │                   │      │
│  │ WDJB-   │                     │      ┌────────┐  │      │
│  │ MJHT    │                     │      │ User   │  │      │
│  │         │                     │      │ visits │  │      │
│  │ Visit:  │                     │◀─────│ URL    │──│      │
│  │ tv.disc │                     │      │ enters │  │      │
│  │ over/   │                     │      │ code   │  │      │
│  │ activate│                     │      └────────┘  │      │
│  └────┬────┘                     │                   │      │
│       │                          │                   │      │
│       │─── POST /token ─────────▶│                   │      │
│       │    (device_code)         │                   │      │
│       │                          │                   │      │
│       │◀── {                     │                   │      │
│       │      access_token: "...",│                   │      │
│       │      refresh_token: "...",                   │      │
│       │      expires_in: 3600    │                   │      │
│       │    } ────────────────────│                   │      │
│       │                          │                   │      │
│  [Authenticated!]                │                   │      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 9.3 Platform Account Linking (Metadata Only)

```
┌─────────────────────────────────────────────────────────────┐
│           PLATFORM LINKING STRATEGY                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  NOTE: We do NOT access platform APIs directly              │
│  We store user's platform SUBSCRIPTIONS for filtering       │
│                                                             │
│  ┌─────────────────────────────────────────────┐           │
│  │  User declares subscriptions (manual)        │           │
│  │                                              │           │
│  │  {                                           │           │
│  │    userId: "user-123",                       │           │
│  │    subscriptions: [                          │           │
│  │      { platform: "netflix", tier: "premium" },│          │
│  │      { platform: "prime", tier: "standard" },│           │
│  │      { platform: "disney", tier: "standard" }│           │
│  │    ],                                        │           │
│  │    region: "US"                              │           │
│  │  }                                           │           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
│  BENEFITS:                                                  │
│  - Filter recommendations by subscribed platforms           │
│  - Prioritize content on platforms user has                 │
│  - Show pricing for content on unsubscribed platforms       │
│  - No platform API integration required                     │
│                                                             │
│  FUTURE (with platform APIs):                               │
│  - OAuth with platform for watch history import             │
│  - Automatic subscription detection                         │
│  - Direct playback integration                              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 9.4 Token Management

```typescript
// Token management implementation
interface TokenManager {
  // Store tokens securely
  storeTokens(tokens: AuthTokens): Promise<void>;

  // Get current access token (auto-refresh if needed)
  getAccessToken(): Promise<string>;

  // Refresh tokens
  refreshTokens(): Promise<AuthTokens>;

  // Revoke tokens on logout
  revokeTokens(): Promise<void>;
}

interface AuthTokens {
  accessToken: string;
  refreshToken: string;
  accessTokenExpiresAt: Date;
  refreshTokenExpiresAt: Date;
  scope: string[];
}

// Token storage by platform
const tokenStorage = {
  cli: 'keychain/keyring', // OS secure storage
  web: 'httpOnly cookies', // Secure cookies
  mobile: 'secure enclave', // iOS Keychain / Android Keystore
  tv: 'encrypted local storage' // Device-specific encryption
};
```

---

## 10. Implementation Roadmap

### 10.1 Phase 1: Foundation (Weeks 1-4)

**Deliverables:**
- [ ] Set up multi-repo structure with infrastructure monorepo
- [ ] Deploy Ruvector cluster (dev environment)
- [ ] Implement basic content schema in Ruvector
- [ ] Create catalog simulator with 10K+ content items
- [ ] Build auth-service with OAuth 2.0 + PKCE
- [ ] Implement basic CLI skeleton (Bubble Tea)

**Success Metrics:**
- Ruvector cluster operational
- 10K content items indexed
- Auth flow functional
- CLI connects to backend

### 10.2 Phase 2: Core Services (Weeks 5-8)

**Deliverables:**
- [ ] Implement ingestion-aggregator (JustWatch/Watchmode integration)
- [ ] Build metadata-normalizer service
- [ ] Implement entity-resolver with fuzzy matching
- [ ] Create semantic-search service with Ruvector
- [ ] Develop basic recommendation-engine (collaborative filtering)
- [ ] Build device-gateway for cross-device sync

**Success Metrics:**
- Real aggregator data flowing
- Entity resolution accuracy > 90%
- Search latency < 100ms
- Basic recommendations functional

### 10.3 Phase 3: Intelligence (Weeks 9-12)

**Deliverables:**
- [ ] Implement GNN-enhanced recommendations
- [ ] Build Claude-Flow agent orchestrator
- [ ] Integrate SPARC methodology for query processing
- [ ] Implement trust-scoring system
- [ ] Create user-preferences service with local storage
- [ ] Build federated learning infrastructure

**Success Metrics:**
- GNN recommendations improve CTR by 20%
- Agent orchestration latency < 200ms
- Trust scores correlate with user satisfaction
- Privacy-preserving sync operational

### 10.4 Phase 4: Integration (Weeks 13-16)

**Deliverables:**
- [ ] Complete CLI with all features
- [ ] Build TV apps (Samsung, LG, Roku)
- [ ] Develop mobile apps (iOS, Android)
- [ ] Implement cross-device sync
- [ ] Create web application
- [ ] Build monitoring and observability stack

**Success Metrics:**
- All device platforms supported
- Cross-device sync < 2 second delay
- 99.9% uptime
- Complete observability

### 10.5 Phase 5: Production (Weeks 17-20)

**Deliverables:**
- [ ] Performance optimization (< 50ms p99 latency)
- [ ] Security audit and penetration testing
- [ ] Load testing (100K concurrent users)
- [ ] Documentation and API reference
- [ ] Beta launch preparation
- [ ] Production deployment

**Success Metrics:**
- p99 latency < 50ms
- Security audit passed
- 100K concurrent users supported
- Production-ready deployment

---

## Appendix A: Technology Stack Summary

| Layer | Technology | Purpose |
|-------|------------|---------|
| Data | Ruvector | Vector + Graph + GNN database |
| Events | Apache Kafka | Event streaming |
| Cache | Redis | Session and query caching |
| Search | Ruvector + Elasticsearch | Hybrid search |
| Auth | Custom OAuth 2.0 | Authentication |
| Gateway | Kong/Envoy | API gateway |
| Mesh | Linkerd | Service mesh |
| Container | Kubernetes | Orchestration |
| CI/CD | GitHub Actions | Automation |
| Monitoring | Prometheus + Grafana | Observability |
| Tracing | Jaeger | Distributed tracing |
| Logging | ELK Stack | Centralized logging |

---

## Appendix B: API Contract Examples

### Search API (gRPC)

```protobuf
service SemanticSearch {
  // Streaming search results
  rpc Search(SearchRequest) returns (stream SearchResult);

  // Get similar content
  rpc FindSimilar(SimilarRequest) returns (SimilarResponse);
}

message SearchRequest {
  string query = 1;
  repeated string genres = 2;
  repeated string platforms = 3;
  string region = 4;
  int32 limit = 5;
  float min_trust_score = 6;
}

message SearchResult {
  string content_id = 1;
  string title = 2;
  string description = 3;
  float relevance_score = 4;
  float trust_score = 5;
  repeated Availability availability = 6;
}

message Availability {
  string platform = 1;
  string pricing_model = 2;
  repeated string quality_tiers = 3;
  string deep_link = 4;
}
```

### Device Sync API (WebSocket)

```typescript
// WebSocket message types
type SyncMessage =
  | { type: 'WATCH_PROGRESS', payload: WatchProgress }
  | { type: 'WATCHLIST_UPDATE', payload: WatchlistUpdate }
  | { type: 'PREFERENCE_UPDATE', payload: PreferenceUpdate }
  | { type: 'SYNC_REQUEST', payload: SyncRequest }
  | { type: 'SYNC_RESPONSE', payload: SyncResponse };

interface WatchProgress {
  contentId: string;
  progress: number;
  duration: number;
  timestamp: number;
  deviceId: string;
}

interface WatchlistUpdate {
  action: 'ADD' | 'REMOVE';
  contentId: string;
  timestamp: number;
  deviceId: string;
}
```

---

*Document Version: 1.0.0*
*Last Updated: December 2025*
*Authors: Architecture Swarm*
