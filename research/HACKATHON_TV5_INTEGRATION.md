# Hackathon-TV5 Integration Specification

## Overview

This document specifies how Media Gateway integrates with the [Agentics Foundation hackathon-tv5](https://github.com/agenticsorg/hackathon-tv5) toolkit. The hackathon-tv5 project provides CLI tools, MCP servers, and the ARW (Agent-Ready Web) specification that form the foundation for our agentic AI media discovery system.

---

## Table of Contents

1. [Integration Architecture](#1-integration-architecture)
2. [ARW Specification Integration](#2-arw-specification-integration)
3. [MCP Server Integration](#3-mcp-server-integration)
4. [CLI Tool Integration](#4-cli-tool-integration)
5. [Available Tools Integration](#5-available-tools-integration)
6. [Package Dependencies](#6-package-dependencies)
7. [Implementation Mapping](#7-implementation-mapping)
8. [Deployment Configuration](#8-deployment-configuration)

---

## 1. Integration Architecture

### High-Level Integration

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        MEDIA GATEWAY + HACKATHON-TV5                             │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │                    LAYER 4: USER EXPERIENCE                              │    │
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
│  │                    INTELLIGENCE LAYER (SONA + Tools)                      │   │
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

### Integration Points

| Media Gateway Component | hackathon-tv5 Component | Integration Type |
|------------------------|------------------------|------------------|
| Layer-1 MCP Connectors | MCP Server (6 tools) | Protocol Bridge |
| Layer-2 Agent Orchestrator | Claude Flow / Agentic Flow | Tool Integration |
| Layer-2 Recommendation Engine | RuVector + SONA | Data + Intelligence |
| Layer-3 Metadata Fabric | ARW Machine Views | Data Format |
| Layer-4 CLI | hackathon-tv5 CLI | Complementary UIs |
| Authentication | OAuth Actions (ARW) | Security Layer |

---

## 2. ARW Specification Integration

The Agent-Ready Web (ARW) specification provides the protocol layer for agent-to-service communication.

### ARW Benefits for Media Gateway

| Benefit | Impact | Implementation |
|---------|--------|----------------|
| **85% Token Reduction** | Lower API costs, faster responses | Machine Views instead of HTML |
| **10x Faster Discovery** | Improved UX | Structured manifests |
| **OAuth-Enforced Actions** | Secure transactions | Platform auth integration |
| **AI-* Headers** | Traffic observability | Agent monitoring |

### ARW Machine View Schema

```typescript
// ARW Machine View for Media Content
interface ARWMediaView {
  "@context": "https://agentics.org/arw/v1";
  "@type": "MediaContent";

  // Core Metadata (replaces HTML scraping)
  id: string;
  title: string;
  description: string;
  contentType: "movie" | "series" | "episode";

  // Availability (structured, not scraped)
  availability: {
    platforms: ARWPlatformAvailability[];
    regions: string[];
    pricing: ARWPricing;
  };

  // Agent-Optimized Fields
  embeddings?: number[];        // Pre-computed for semantic search
  taxonomy?: string[];          // Structured categories
  relationships?: ARWRelation[];// Graph connections

  // ARW Headers
  headers: {
    "AI-Request-ID": string;
    "AI-Agent-Name": string;
    "AI-Purpose": string;
    "AI-Token-Budget"?: number;
  };
}

interface ARWPlatformAvailability {
  platform: string;
  url: string;
  type: "subscription" | "rent" | "buy" | "free";
  price?: number;
  currency?: string;
  quality: "SD" | "HD" | "4K" | "HDR";
}
```

### ARW Integration in Media Gateway

```rust
// Rust implementation for ARW client
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize)]
pub struct ARWRequest {
    pub endpoint: String,
    pub headers: ARWHeaders,
    pub view_type: ViewType,
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

#[derive(Debug, Serialize, Deserialize)]
pub enum ViewType {
    Machine,  // Structured JSON (85% token reduction)
    Human,    // HTML view
    Hybrid,   // Both available
}

impl ARWClient {
    /// Fetch content using ARW machine view
    pub async fn fetch_machine_view(&self, content_id: &str) -> Result<ARWMediaView> {
        let request = ARWRequest {
            endpoint: format!("/content/{}/machine", content_id),
            headers: ARWHeaders {
                request_id: Uuid::new_v4().to_string(),
                agent_name: "media-gateway".to_string(),
                purpose: "content-discovery".to_string(),
                token_budget: Some(1000),
            },
            view_type: ViewType::Machine,
        };

        self.client.get(&request.endpoint)
            .headers(request.headers.into())
            .send()
            .await?
            .json()
            .await
    }
}
```

---

## 3. MCP Server Integration

### hackathon-tv5 MCP Tools

The hackathon-tv5 MCP server provides 6 core tools:

| Tool | Purpose | Media Gateway Integration |
|------|---------|---------------------------|
| `get_hackathon_info` | Hackathon details | Project context |
| `get_tracks` | Competition tracks | Track selection |
| `get_available_tools` | Tool catalog | Tool discovery |
| `get_project_status` | Project state | Status monitoring |
| `check_tool_installed` | Tool verification | Dependency check |
| `get_resources` | Resource links | Documentation access |

### MCP Server Bridge Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        MCP SERVER BRIDGE                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌────────────────────┐         ┌────────────────────┐                      │
│  │  hackathon-tv5     │         │   Media Gateway    │                      │
│  │    MCP Server      │◄───────►│   MCP Connectors   │                      │
│  │                    │  Bridge │                    │                      │
│  │  - STDIO transport │         │  - Netflix MCP     │                      │
│  │  - SSE transport   │         │  - Prime MCP       │                      │
│  │  - 6 tools         │         │  - Disney+ MCP     │                      │
│  │  - 2 prompts       │         │  - Aggregator MCP  │                      │
│  └────────────────────┘         └────────────────────┘                      │
│            │                              │                                  │
│            │                              │                                  │
│            ▼                              ▼                                  │
│  ┌────────────────────────────────────────────────────────────────────┐     │
│  │                    CLAUDE DESKTOP INTEGRATION                       │     │
│  │                                                                     │     │
│  │  claude_desktop_config.json:                                       │     │
│  │  {                                                                 │     │
│  │    "mcpServers": {                                                 │     │
│  │      "agentics-hackathon": {                                       │     │
│  │        "command": "npx",                                           │     │
│  │        "args": ["agentics-hackathon", "mcp"]                       │     │
│  │      },                                                            │     │
│  │      "media-gateway": {                                            │     │
│  │        "command": "./media-gateway",                               │     │
│  │        "args": ["mcp", "--transport", "stdio"]                     │     │
│  │      }                                                             │     │
│  │    }                                                               │     │
│  │  }                                                                 │     │
│  └────────────────────────────────────────────────────────────────────┘     │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Unified MCP Configuration

```json
{
  "mcpServers": {
    "agentics-hackathon": {
      "command": "npx",
      "args": ["agentics-hackathon", "mcp"],
      "env": {
        "NODE_ENV": "production"
      }
    },
    "media-gateway-core": {
      "command": "./media-gateway",
      "args": ["mcp", "--transport", "stdio"],
      "env": {
        "RUST_LOG": "info",
        "RUVECTOR_URL": "http://localhost:6333"
      }
    },
    "media-gateway-sse": {
      "url": "http://localhost:3001/sse",
      "transport": "sse"
    }
  }
}
```

---

## 4. CLI Tool Integration

### Dual CLI Strategy

Media Gateway provides two complementary CLI experiences:

| CLI | Language | Purpose | Primary Users |
|-----|----------|---------|---------------|
| **media-gateway** | Rust (TUI) | Production operations, search, recommendations | End users |
| **npx agentics-hackathon** | TypeScript | Development, tool setup, hackathon features | Developers |

### CLI Command Mapping

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        CLI COMMAND COMPARISON                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  hackathon-tv5 CLI              Media Gateway CLI                           │
│  ─────────────────              ──────────────────                           │
│  npx agentics-hackathon init    media-gateway init                          │
│  npx agentics-hackathon tools   media-gateway tools --list                  │
│  npx agentics-hackathon status  media-gateway status                        │
│  npx agentics-hackathon mcp     media-gateway mcp serve                     │
│  npx agentics-hackathon info    media-gateway --help                        │
│                                                                              │
│  UNIQUE TO hackathon-tv5        UNIQUE TO media-gateway                     │
│  ─────────────────────          ────────────────────────                     │
│  npx agentics-hackathon discord media-gateway search "action movies"        │
│                                 media-gateway recommend --mood relaxing     │
│                                 media-gateway watch --continue              │
│                                 media-gateway sync --device tv              │
│                                 media-gateway auth --platform netflix       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Integrated Workflow

```bash
# Step 1: Initialize with hackathon-tv5
npx agentics-hackathon init
# Select track: Entertainment Discovery

# Step 2: Install required tools
npx agentics-hackathon tools
# Install: Claude Flow, RuVector, AgentDB

# Step 3: Start MCP servers
npx agentics-hackathon mcp sse --port 3000 &
media-gateway mcp serve --port 3001 &

# Step 4: Use Media Gateway for discovery
media-gateway search "sci-fi like Blade Runner"
media-gateway recommend --personalized

# Step 5: Check project status
npx agentics-hackathon status
```

---

## 5. Available Tools Integration

### hackathon-tv5 Tool Categories

The 17+ tools from hackathon-tv5 integrate with Media Gateway's layers:

#### AI Assistants
| Tool | Integration Point | Purpose |
|------|-------------------|---------|
| Claude Code CLI | Development | Code generation, debugging |
| Gemini CLI | Development | Alternative AI assistant |

#### Orchestration (Layer-2)
| Tool | Integration Point | Purpose |
|------|-------------------|---------|
| **Claude Flow** | Agent Orchestrator | 101 MCP tools, SPARC methodology |
| **Agentic Flow** | Agent Orchestrator | 66 specialized agents |
| Flow Nexus | Cross-Platform Orchestrator | Workflow coordination |
| Google ADK | Agent Orchestrator | Agent development kit |

#### Cloud (Infrastructure)
| Tool | Integration Point | Purpose |
|------|-------------------|---------|
| Google Cloud CLI | GCP Deployment | Infrastructure management |
| Vertex AI SDK | Intelligence Layer | ML model serving |

#### Databases (Data Layer)
| Tool | Integration Point | Purpose |
|------|-------------------|---------|
| **RuVector** | Primary Data Engine | Hypergraph + Vector + GNN |
| **AgentDB** | Memory Layer | Agent context storage |

#### Frameworks (Layer-1/2)
| Tool | Integration Point | Purpose |
|------|-------------------|---------|
| LionPride | Alternative Framework | Python agent orchestration |
| Agentic Framework | Alternative Framework | Python tooling |
| OpenAI Agents SDK | Agent Orchestrator | OpenAI integration |

#### Advanced (Intelligence)
| Tool | Integration Point | Purpose |
|------|-------------------|---------|
| Agentic Synth | Data Generation | Synthetic content creation |
| Strange Loops | Reasoning | Complex reasoning patterns |
| **SPARC 2.0** | Methodology | Structured agent reasoning |

### Tool Installation Automation

```bash
#!/bin/bash
# install-media-gateway-tools.sh

echo "Installing Media Gateway required tools..."

# Core tools (required)
npx agentics-hackathon tools --install claude-flow
npx agentics-hackathon tools --install ruvector
npx agentics-hackathon tools --install agentdb

# Orchestration tools (recommended)
npx agentics-hackathon tools --install agentic-flow
npx agentics-hackathon tools --install google-adk

# Cloud tools (for GCP deployment)
npx agentics-hackathon tools --install gcloud-cli
npx agentics-hackathon tools --install vertex-ai-sdk

# Verify installation
npx agentics-hackathon status
```

---

## 6. Package Dependencies

### hackathon-tv5 Packages Integration

```
packages/
├── schemas/           → Shared type definitions
│   ├── arw-schema/    → ARW specification types
│   ├── mcp-schema/    → MCP protocol types
│   └── media-schema/  → Media content types
│
├── validators/        → Data validation
│   ├── arw-validator/ → ARW compliance checking
│   └── content-validator/
│
├── sdks/              → Client libraries
│   ├── arw-sdk/       → ARW client SDK
│   ├── mcp-sdk/       → MCP client SDK
│   └── media-sdk/     → Media Gateway SDK
│
└── plugins/           → Extension points
    ├── claude-plugin/ → Claude integration
    └── platform-plugins/
```

### Rust Integration with TypeScript Packages

```rust
// Using napi-rs for TypeScript package interop
use napi::bindgen_prelude::*;
use napi_derive::napi;

/// Bridge to hackathon-tv5 ARW SDK
#[napi]
pub struct ARWBridge {
    sdk: JsObject,  // TypeScript ARW SDK instance
}

#[napi]
impl ARWBridge {
    #[napi(constructor)]
    pub fn new(env: Env) -> Result<Self> {
        // Initialize TypeScript ARW SDK
        let sdk = env.get_global()?
            .get_named_property::<JsObject>("arwSdk")?;
        Ok(Self { sdk })
    }

    #[napi]
    pub async fn fetch_machine_view(&self, content_id: String) -> Result<JsObject> {
        // Call TypeScript SDK method
        self.sdk.get_named_property::<JsFunction>("fetchMachineView")?
            .call(None, &[content_id.into_js_value()?])?
            .await
    }
}
```

---

## 7. Implementation Mapping

### Layer Mapping to hackathon-tv5 Components

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                  MEDIA GATEWAY ↔ HACKATHON-TV5 MAPPING                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  MEDIA GATEWAY LAYER          HACKATHON-TV5 COMPONENT                       │
│  ────────────────────         ─────────────────────────                      │
│                                                                              │
│  Layer-1: Ingestion ─────────► MCP Server (platform connectors)             │
│           Auth ──────────────► OAuth Actions (ARW spec)                     │
│           Device Sync ───────► MCP Resources (state sync)                   │
│                                                                              │
│  Layer-2: Agent Orchestrator ► Claude Flow (101 tools)                      │
│           Recommendation ────► RuVector + SONA                              │
│           Semantic Search ───► RuVector (embeddings)                        │
│           Memory ────────────► AgentDB (context)                            │
│                                                                              │
│  Layer-3: Metadata Fabric ───► ARW Machine Views (85% tokens)               │
│           Availability ──────► ARW Platform Availability                    │
│           Personalization ───► SONA LoRA Adapters                           │
│                                                                              │
│  Layer-4: CLI ───────────────► hackathon-tv5 CLI (dev)                      │
│           Web App ───────────► Media Discovery App (Next.js)                │
│           Extensions ────────► ARW Chrome Extension                         │
│                                                                              │
│  Infrastructure: GCP ────────► Google Cloud CLI + Vertex AI SDK             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Hackathon Track Alignment

| Track | Media Gateway Focus | Key Components |
|-------|--------------------|--------------------|
| **Entertainment Discovery** | Core functionality | Search, recommendations, availability |
| **Multi-Agent Systems** | Layer-2 Intelligence | Claude Flow, SONA, 9 specialized agents |
| **Agentic Workflows** | End-to-end flows | Auth → Search → Recommend → Sync |
| **Open Innovation** | Novel features | Cross-device sync, privacy-safe personalization |

---

## 8. Deployment Configuration

### Combined Docker Compose

```yaml
version: '3.8'

services:
  # hackathon-tv5 MCP Server
  hackathon-mcp:
    image: node:20-alpine
    command: npx agentics-hackathon mcp sse --port 3000
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production

  # Media Gateway MCP Server
  media-gateway-mcp:
    build:
      context: ./media-gateway
      dockerfile: Dockerfile
    command: media-gateway mcp serve --port 3001
    ports:
      - "3001:3001"
    environment:
      - RUST_LOG=info
      - RUVECTOR_URL=http://ruvector:6333
    depends_on:
      - ruvector

  # RuVector Database
  ruvector:
    image: ruvector/ruvector:latest
    ports:
      - "6333:6333"
      - "6334:6334"
    volumes:
      - ruvector_data:/data

  # AgentDB (Redis-compatible)
  agentdb:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - agentdb_data:/data

  # Media Discovery App (from hackathon-tv5)
  media-discovery:
    build:
      context: ./hackathon-tv5/apps/media-discovery
      dockerfile: Dockerfile
    ports:
      - "3002:3000"
    environment:
      - NEXT_PUBLIC_MCP_URL=http://hackathon-mcp:3000
      - NEXT_PUBLIC_GATEWAY_URL=http://media-gateway-mcp:3001

volumes:
  ruvector_data:
  agentdb_data:
```

### GKE Deployment Integration

```yaml
# k8s/hackathon-tv5/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hackathon-mcp
  namespace: media-gateway
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hackathon-mcp
  template:
    metadata:
      labels:
        app: hackathon-mcp
    spec:
      containers:
      - name: hackathon-mcp
        image: gcr.io/PROJECT_ID/hackathon-mcp:latest
        ports:
        - containerPort: 3000
        resources:
          requests:
            cpu: "100m"
            memory: "256Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
        env:
        - name: NODE_ENV
          value: "production"
        - name: MCP_TRANSPORT
          value: "sse"
---
apiVersion: v1
kind: Service
metadata:
  name: hackathon-mcp
  namespace: media-gateway
spec:
  selector:
    app: hackathon-mcp
  ports:
  - port: 3000
    targetPort: 3000
  type: ClusterIP
```

---

## Summary

The hackathon-tv5 integration provides:

1. **ARW Protocol**: 85% token reduction, structured machine views
2. **MCP Foundation**: 6 tools + extensible server architecture
3. **Tool Ecosystem**: 17+ tools across AI, orchestration, databases, and frameworks
4. **Dual CLI**: Development (TypeScript) + Production (Rust)
5. **Reference Apps**: Media Discovery app and ARW Chrome Extension

This integration positions Media Gateway as the production-grade implementation of the hackathon-tv5 vision, combining:
- Rust performance with TypeScript developer experience
- SONA intelligence with ARW efficiency
- GCP infrastructure with MCP flexibility

---

*Document Version: 1.0.0*
*Last Updated: December 2025*
*Integration Target: [agenticsorg/hackathon-tv5](https://github.com/agenticsorg/hackathon-tv5)*
