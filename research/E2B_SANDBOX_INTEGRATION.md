# E2B Sandbox Integration Specification

## Overview

This document specifies how Media Gateway integrates [E2B](https://e2b.dev) to provide secure, isolated sandbox environments for AI agent code execution. E2B enables our multi-agent system to safely execute LLM-generated code without compromising system security.

---

## Table of Contents

1. [Why E2B](#1-why-e2b)
2. [Integration Architecture](#2-integration-architecture)
3. [Sandbox Types](#3-sandbox-types)
4. [Agent Sandbox Workflows](#4-agent-sandbox-workflows)
5. [Security Model](#5-security-model)
6. [Rust Integration](#6-rust-integration)
7. [GCP Deployment](#7-gcp-deployment)
8. [Cost Estimation](#8-cost-estimation)

---

## 1. Why E2B

### The Problem

Our multi-agent system (Claude-Flow with 9 specialized agents) generates and executes code for:
- Data analysis and visualization
- Content recommendation computations
- Semantic search queries
- Platform API interactions
- User preference processing

Executing LLM-generated code directly on production infrastructure poses significant security risks.

### E2B Solution

| Challenge | E2B Solution |
|-----------|--------------|
| **Untrusted Code Execution** | Firecracker microVM isolation |
| **Resource Exhaustion** | Configurable CPU/memory limits |
| **Network Security** | Sandboxed network with controlled egress |
| **Data Exfiltration** | Isolated filesystem, no host access |
| **Long-Running Tasks** | Sessions up to 24 hours |
| **Scale** | Sub-200ms startup, thousands concurrent |

### Key E2B Features

- **Firecracker microVMs**: Hardware-level isolation in ~150ms startup
- **Multi-language**: Python, JavaScript, R, Java, Bash
- **Persistence**: Pause/resume sandbox state
- **File Operations**: Upload/download with streaming
- **Internet Access**: Configurable per-sandbox
- **Custom Templates**: Docker-based environment customization
- **MCP Integration**: Native Model Context Protocol support

---

## 2. Integration Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        MEDIA GATEWAY + E2B SANDBOX                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │                    LAYER 2: INTELLIGENCE                                 │    │
│  │  ┌──────────────────────────────────────────────────────────────────┐   │    │
│  │  │                   CLAUDE-FLOW ORCHESTRATOR                        │   │    │
│  │  │  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐    │   │    │
│  │  │  │ Discovery  │ │ Analysis   │ │ Recommend  │ │  Search    │    │   │    │
│  │  │  │   Agent    │ │   Agent    │ │   Agent    │ │   Agent    │    │   │    │
│  │  │  └─────┬──────┘ └─────┬──────┘ └─────┬──────┘ └─────┬──────┘    │   │    │
│  │  │        │              │              │              │            │   │    │
│  │  │        └──────────────┴──────────────┴──────────────┘            │   │    │
│  │  │                              │                                    │   │    │
│  │  │                    ┌─────────▼─────────┐                         │   │    │
│  │  │                    │  E2B SANDBOX      │                         │   │    │
│  │  │                    │    MANAGER        │                         │   │    │
│  │  │                    └─────────┬─────────┘                         │   │    │
│  │  └──────────────────────────────┼────────────────────────────────────┘   │    │
│  └─────────────────────────────────┼────────────────────────────────────────┘    │
│                                    │                                             │
│  ┌─────────────────────────────────▼────────────────────────────────────────┐   │
│  │                         E2B CLOUD                                         │   │
│  │  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐              │   │
│  │  │   Code         │  │   Data         │  │   Desktop      │              │   │
│  │  │   Interpreter  │  │   Analysis     │  │   Sandbox      │              │   │
│  │  │   Sandbox      │  │   Sandbox      │  │   (Browser)    │              │   │
│  │  │                │  │                │  │                │              │   │
│  │  │  Python/JS/R   │  │  Pandas/NumPy  │  │  Playwright    │              │   │
│  │  │  ~150ms start  │  │  Visualization │  │  Computer Use  │              │   │
│  │  └────────────────┘  └────────────────┘  └────────────────┘              │   │
│  │                                                                           │   │
│  │  ┌────────────────────────────────────────────────────────────────────┐  │   │
│  │  │  CUSTOM TEMPLATES                                                   │  │   │
│  │  │  • media-gateway-agent: Ruvector client, recommendation libs       │  │   │
│  │  │  • content-analyzer: FFmpeg, media processing tools                │  │   │
│  │  │  • search-executor: Vector search, embedding models                │  │   │
│  │  └────────────────────────────────────────────────────────────────────┘  │   │
│  └───────────────────────────────────────────────────────────────────────────┘   │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Integration Points

| Media Gateway Component | E2B Integration | Purpose |
|------------------------|-----------------|---------|
| **Claude-Flow Agents** | Code Interpreter | Execute generated analysis code |
| **Recommendation Engine** | Data Analysis Sandbox | Run recommendation algorithms |
| **Semantic Search** | Custom Template | Execute vector similarity code |
| **Content Analyzer** | Custom Template | Process media metadata |
| **SONA LoRA Updates** | Isolated Compute | Safe model weight updates |
| **Web Scraping** | Desktop Sandbox | Browser automation for metadata |

---

## 3. Sandbox Types

### 3.1 Code Interpreter Sandbox

For general-purpose code execution by agents.

```python
from e2b_code_interpreter import Sandbox

async def execute_agent_code(code: str, context: dict) -> ExecutionResult:
    """Execute agent-generated code in isolated sandbox."""
    with Sandbox() as sandbox:
        # Upload context data
        sandbox.files.write("/data/context.json", json.dumps(context))

        # Execute code
        execution = sandbox.run_code(code)

        # Return results
        return ExecutionResult(
            output=execution.text,
            logs=execution.logs,
            error=execution.error,
            artifacts=sandbox.files.list("/output/")
        )
```

### 3.2 Data Analysis Sandbox

For recommendation computations and analytics.

```python
from e2b_code_interpreter import Sandbox

async def run_recommendation_analysis(user_id: str, preferences: dict):
    """Run personalized recommendation analysis in sandbox."""
    with Sandbox(template="media-gateway-analysis") as sandbox:
        # Install dependencies
        sandbox.run_code("!pip install pandas numpy scikit-learn")

        # Upload user data
        sandbox.files.write("/data/preferences.json", json.dumps(preferences))

        # Execute recommendation algorithm
        result = sandbox.run_code("""
import pandas as pd
import numpy as np
from sklearn.metrics.pairwise import cosine_similarity

# Load preferences
prefs = pd.read_json('/data/preferences.json')

# Compute recommendations
# ... recommendation logic ...

# Save results
recommendations.to_json('/output/recommendations.json')
        """)

        # Download results
        return sandbox.files.read("/output/recommendations.json")
```

### 3.3 Desktop Sandbox (Browser Automation)

For web scraping and metadata enrichment.

```python
from e2b_desktop import Sandbox

async def scrape_streaming_metadata(url: str):
    """Use browser automation to scrape streaming platform metadata."""
    with Sandbox() as sandbox:
        # Launch browser
        sandbox.run_code("""
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.goto(url)

    # Extract metadata
    title = page.query_selector('h1').inner_text()
    description = page.query_selector('.description').inner_text()

    # Save results
    with open('/output/metadata.json', 'w') as f:
        json.dump({'title': title, 'description': description}, f)

    browser.close()
        """)

        return sandbox.files.read("/output/metadata.json")
```

### 3.4 Custom Templates

#### Media Gateway Agent Template

```dockerfile
# e2b-templates/media-gateway-agent/Dockerfile
FROM e2b/code-interpreter:latest

# Install Media Gateway dependencies
RUN pip install \
    ruvector-client \
    numpy \
    pandas \
    scikit-learn \
    sentence-transformers \
    httpx

# Install system tools
RUN apt-get update && apt-get install -y \
    ffmpeg \
    jq \
    curl

# Copy agent utilities
COPY agent_utils.py /opt/media-gateway/
ENV PYTHONPATH="/opt/media-gateway:$PYTHONPATH"

# Configure sandbox
ENV E2B_TIMEOUT=300
ENV E2B_MEMORY=2048
```

#### Template Configuration

```toml
# e2b-templates/media-gateway-agent/e2b.toml
[template]
name = "media-gateway-agent"
dockerfile = "Dockerfile"

[template.resources]
cpu_count = 2
memory_mb = 2048

[template.network]
internet_access = true
allowed_hosts = [
    "api.ruvector.io",
    "api.justwatch.com",
    "api.themoviedb.org"
]
```

---

## 4. Agent Sandbox Workflows

### 4.1 Discovery Agent Workflow

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  User Query     │────▶│  Discovery      │────▶│  E2B Sandbox    │
│  "sci-fi like   │     │  Agent          │     │  (Code Exec)    │
│   Stranger..."  │     │                 │     │                 │
└─────────────────┘     └────────┬────────┘     └────────┬────────┘
                                 │                       │
                                 │  Generate            │  Execute
                                 │  Search Code         │  Safely
                                 ▼                       ▼
                        ┌─────────────────┐     ┌─────────────────┐
                        │  LLM (Claude)   │     │  Results        │
                        │  Code Gen       │────▶│  (JSON)         │
                        └─────────────────┘     └─────────────────┘
```

### 4.2 Recommendation Agent Workflow

```python
class RecommendationAgent:
    def __init__(self, e2b_manager: E2BSandboxManager):
        self.e2b = e2b_manager
        self.template = "media-gateway-agent"

    async def generate_recommendations(
        self,
        user_id: str,
        context: RecommendationContext
    ) -> List[Recommendation]:
        # Step 1: Generate recommendation code
        code = await self.llm.generate(
            prompt=f"Generate recommendation algorithm for user preferences: {context}",
            system="You are a recommendation system expert. Output Python code only."
        )

        # Step 2: Execute in E2B sandbox
        result = await self.e2b.execute(
            template=self.template,
            code=code,
            inputs={
                "user_preferences": context.preferences,
                "content_embeddings": context.embeddings,
                "watch_history": context.history
            },
            timeout=30
        )

        # Step 3: Parse and validate results
        if result.error:
            return await self.fallback_recommendations(user_id)

        return self.parse_recommendations(result.output)
```

### 4.3 SONA LoRA Update Workflow

```python
class SONAUpdateAgent:
    """Safely update user LoRA adapters in E2B sandbox."""

    async def update_lora_adapter(
        self,
        user_id: str,
        interaction: UserInteraction
    ):
        # Execute LoRA update in isolated sandbox
        result = await self.e2b.execute(
            template="sona-lora-updater",
            code="""
import torch
from lora_utils import load_adapter, compute_gradient, apply_ewc

# Load current adapter
adapter = load_adapter(user_id)

# Compute gradient from interaction
gradient = compute_gradient(interaction, adapter)

# Apply EWC++ regularization
ewc_penalty = compute_ewc_penalty(adapter)
safe_gradient = gradient - ewc_penalty

# Update adapter weights
adapter.apply_gradient(safe_gradient)

# Save updated adapter
adapter.save(f'/output/{user_id}_adapter.pt')
            """,
            inputs={
                "user_id": user_id,
                "interaction": interaction.to_dict()
            }
        )

        # Upload updated adapter to storage
        if not result.error:
            await self.storage.upload_adapter(
                user_id,
                result.artifacts[f"{user_id}_adapter.pt"]
            )
```

---

## 5. Security Model

### 5.1 Isolation Layers

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           E2B SECURITY MODEL                                     │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  LAYER 1: FIRECRACKER microVM                                           │    │
│  │  • Hardware-level isolation                                             │    │
│  │  • Separate kernel per sandbox                                          │    │
│  │  • No shared memory with host                                           │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                                                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  LAYER 2: NETWORK ISOLATION                                              │    │
│  │  • Default: No internet access                                          │    │
│  │  • Configurable allowlist for specific hosts                            │    │
│  │  • No access to internal GCP network                                    │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                                                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  LAYER 3: RESOURCE LIMITS                                                │    │
│  │  • CPU: Configurable (1-8 cores)                                        │    │
│  │  • Memory: Configurable (512MB - 8GB)                                   │    │
│  │  • Timeout: Max 24 hours (configurable)                                 │    │
│  │  • Disk: Isolated filesystem, cleared on termination                    │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                                                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  LAYER 4: INPUT/OUTPUT VALIDATION                                        │    │
│  │  • Sanitize all inputs before sandbox upload                            │    │
│  │  • Validate outputs before processing                                   │    │
│  │  • Size limits on file transfers                                        │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 5.2 Security Configuration

```rust
/// E2B sandbox security configuration
pub struct SandboxSecurityConfig {
    /// Allow internet access from sandbox
    pub internet_access: bool,

    /// Allowed external hosts (if internet enabled)
    pub allowed_hosts: Vec<String>,

    /// Maximum execution time (seconds)
    pub timeout_seconds: u32,

    /// Maximum memory (MB)
    pub memory_mb: u32,

    /// Maximum CPU cores
    pub cpu_cores: u8,

    /// Maximum file upload size (bytes)
    pub max_upload_bytes: u64,

    /// Maximum file download size (bytes)
    pub max_download_bytes: u64,
}

impl Default for SandboxSecurityConfig {
    fn default() -> Self {
        Self {
            internet_access: false,
            allowed_hosts: vec![],
            timeout_seconds: 60,
            memory_mb: 1024,
            cpu_cores: 2,
            max_upload_bytes: 10 * 1024 * 1024,  // 10MB
            max_download_bytes: 50 * 1024 * 1024, // 50MB
        }
    }
}
```

---

## 6. Rust Integration

### 6.1 E2B Client Wrapper

```rust
use reqwest::Client;
use serde::{Deserialize, Serialize};
use tokio::time::{timeout, Duration};

/// E2B Sandbox Manager for Rust
pub struct E2BSandboxManager {
    client: Client,
    api_key: String,
    base_url: String,
}

#[derive(Debug, Serialize)]
pub struct CreateSandboxRequest {
    pub template: String,
    pub timeout: u32,
    #[serde(skip_serializing_if = "Option::is_none")]
    pub metadata: Option<serde_json::Value>,
}

#[derive(Debug, Deserialize)]
pub struct Sandbox {
    pub id: String,
    pub template: String,
    pub status: SandboxStatus,
}

#[derive(Debug, Deserialize)]
#[serde(rename_all = "lowercase")]
pub enum SandboxStatus {
    Running,
    Paused,
    Terminated,
}

#[derive(Debug, Deserialize)]
pub struct ExecutionResult {
    pub stdout: String,
    pub stderr: String,
    pub exit_code: i32,
    pub error: Option<String>,
}

impl E2BSandboxManager {
    pub fn new(api_key: String) -> Self {
        Self {
            client: Client::new(),
            api_key,
            base_url: "https://api.e2b.dev".to_string(),
        }
    }

    /// Create a new sandbox from template
    pub async fn create_sandbox(
        &self,
        template: &str,
        config: &SandboxSecurityConfig,
    ) -> Result<Sandbox, E2BError> {
        let request = CreateSandboxRequest {
            template: template.to_string(),
            timeout: config.timeout_seconds,
            metadata: None,
        };

        let response = self.client
            .post(format!("{}/sandboxes", self.base_url))
            .header("X-E2B-API-Key", &self.api_key)
            .json(&request)
            .send()
            .await?;

        Ok(response.json().await?)
    }

    /// Execute code in sandbox
    pub async fn execute_code(
        &self,
        sandbox_id: &str,
        code: &str,
        language: Language,
    ) -> Result<ExecutionResult, E2BError> {
        let response = self.client
            .post(format!("{}/sandboxes/{}/code", self.base_url, sandbox_id))
            .header("X-E2B-API-Key", &self.api_key)
            .json(&serde_json::json!({
                "code": code,
                "language": language.as_str(),
            }))
            .send()
            .await?;

        Ok(response.json().await?)
    }

    /// Upload file to sandbox
    pub async fn upload_file(
        &self,
        sandbox_id: &str,
        path: &str,
        content: &[u8],
    ) -> Result<(), E2BError> {
        self.client
            .post(format!("{}/sandboxes/{}/files", self.base_url, sandbox_id))
            .header("X-E2B-API-Key", &self.api_key)
            .json(&serde_json::json!({
                "path": path,
                "content": base64::encode(content),
            }))
            .send()
            .await?;

        Ok(())
    }

    /// Download file from sandbox
    pub async fn download_file(
        &self,
        sandbox_id: &str,
        path: &str,
    ) -> Result<Vec<u8>, E2BError> {
        let response = self.client
            .get(format!("{}/sandboxes/{}/files/{}", self.base_url, sandbox_id, path))
            .header("X-E2B-API-Key", &self.api_key)
            .send()
            .await?;

        let data: FileResponse = response.json().await?;
        Ok(base64::decode(&data.content)?)
    }

    /// Terminate sandbox
    pub async fn terminate(&self, sandbox_id: &str) -> Result<(), E2BError> {
        self.client
            .delete(format!("{}/sandboxes/{}", self.base_url, sandbox_id))
            .header("X-E2B-API-Key", &self.api_key)
            .send()
            .await?;

        Ok(())
    }
}

#[derive(Debug, Clone, Copy)]
pub enum Language {
    Python,
    JavaScript,
    TypeScript,
    R,
    Bash,
}

impl Language {
    fn as_str(&self) -> &'static str {
        match self {
            Language::Python => "python",
            Language::JavaScript => "javascript",
            Language::TypeScript => "typescript",
            Language::R => "r",
            Language::Bash => "bash",
        }
    }
}
```

### 6.2 Agent Integration

```rust
use crate::e2b::{E2BSandboxManager, SandboxSecurityConfig, Language};
use crate::agents::Agent;

/// Agent with E2B sandbox execution capability
pub struct SandboxedAgent {
    agent: Box<dyn Agent>,
    e2b: E2BSandboxManager,
    template: String,
    security_config: SandboxSecurityConfig,
}

impl SandboxedAgent {
    pub fn new(
        agent: Box<dyn Agent>,
        e2b: E2BSandboxManager,
        template: &str,
    ) -> Self {
        Self {
            agent,
            e2b,
            template: template.to_string(),
            security_config: SandboxSecurityConfig::default(),
        }
    }

    /// Execute agent-generated code safely in E2B sandbox
    pub async fn execute_safely(
        &self,
        task: &AgentTask,
    ) -> Result<AgentResult, AgentError> {
        // Step 1: Agent generates code
        let generated_code = self.agent.generate_code(task).await?;

        // Step 2: Create sandbox
        let sandbox = self.e2b
            .create_sandbox(&self.template, &self.security_config)
            .await?;

        // Step 3: Upload task inputs
        if let Some(inputs) = &task.inputs {
            self.e2b
                .upload_file(&sandbox.id, "/data/inputs.json", inputs.as_bytes())
                .await?;
        }

        // Step 4: Execute code in sandbox
        let result = self.e2b
            .execute_code(&sandbox.id, &generated_code, Language::Python)
            .await?;

        // Step 5: Download outputs
        let outputs = if result.exit_code == 0 {
            self.e2b
                .download_file(&sandbox.id, "/output/results.json")
                .await
                .ok()
        } else {
            None
        };

        // Step 6: Cleanup
        self.e2b.terminate(&sandbox.id).await?;

        // Step 7: Return results
        Ok(AgentResult {
            success: result.exit_code == 0,
            output: outputs,
            logs: result.stdout,
            error: result.error,
        })
    }
}
```

---

## 7. GCP Deployment

### 7.1 Architecture with E2B

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              GCP + E2B DEPLOYMENT                                │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │                         GKE AUTOPILOT                                    │    │
│  │  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐            │    │
│  │  │ Agent          │  │ Recommendation │  │ Search         │            │    │
│  │  │ Orchestrator   │  │ Service        │  │ Service        │            │    │
│  │  │                │  │                │  │                │            │    │
│  │  │  E2B Client ◄──┼──┼── E2B Client ◄─┼──┼── E2B Client   │            │    │
│  │  └───────┬────────┘  └───────┬────────┘  └───────┬────────┘            │    │
│  │          │                   │                   │                      │    │
│  └──────────┼───────────────────┼───────────────────┼──────────────────────┘    │
│             │                   │                   │                           │
│             └───────────────────┼───────────────────┘                           │
│                                 │                                                │
│                                 ▼                                                │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │                         E2B CLOUD                                        │    │
│  │                    (External Service)                                    │    │
│  │                                                                          │    │
│  │  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐            │    │
│  │  │ Sandbox Pool   │  │ Sandbox Pool   │  │ Sandbox Pool   │            │    │
│  │  │ (Discovery)    │  │ (Analysis)     │  │ (Search)       │            │    │
│  │  │                │  │                │  │                │            │    │
│  │  │ 10 warm        │  │ 5 warm         │  │ 5 warm         │            │    │
│  │  │ instances      │  │ instances      │  │ instances      │            │    │
│  │  └────────────────┘  └────────────────┘  └────────────────┘            │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 7.2 Kubernetes Configuration

```yaml
# k8s/e2b-config/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: e2b-config
  namespace: media-gateway
data:
  E2B_BASE_URL: "https://api.e2b.dev"
  E2B_DEFAULT_TIMEOUT: "60"
  E2B_DEFAULT_MEMORY: "1024"
  E2B_TEMPLATES: |
    discovery: media-gateway-agent
    analysis: media-gateway-analysis
    search: media-gateway-search
    browser: e2b/desktop
---
# k8s/e2b-config/secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: e2b-credentials
  namespace: media-gateway
type: Opaque
stringData:
  E2B_API_KEY: "${E2B_API_KEY}"
---
# k8s/agent-orchestrator/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: agent-orchestrator
  namespace: media-gateway
spec:
  replicas: 3
  selector:
    matchLabels:
      app: agent-orchestrator
  template:
    metadata:
      labels:
        app: agent-orchestrator
    spec:
      containers:
      - name: orchestrator
        image: gcr.io/PROJECT_ID/agent-orchestrator:latest
        envFrom:
        - configMapRef:
            name: e2b-config
        - secretRef:
            name: e2b-credentials
        resources:
          requests:
            cpu: "500m"
            memory: "1Gi"
          limits:
            cpu: "2"
            memory: "4Gi"
```

### 7.3 Secret Manager Integration

```hcl
# terraform/modules/e2b/main.tf
resource "google_secret_manager_secret" "e2b_api_key" {
  project   = var.project_id
  secret_id = "e2b-api-key"

  replication {
    auto {}
  }
}

resource "google_secret_manager_secret_version" "e2b_api_key_version" {
  secret      = google_secret_manager_secret.e2b_api_key.id
  secret_data = var.e2b_api_key
}

# Grant GKE workload identity access
resource "google_secret_manager_secret_iam_member" "e2b_key_accessor" {
  project   = var.project_id
  secret_id = google_secret_manager_secret.e2b_api_key.secret_id
  role      = "roles/secretmanager.secretAccessor"
  member    = "serviceAccount:${var.gke_service_account}"
}
```

---

## 8. Cost Estimation

### 8.1 E2B Pricing Model

| Tier | Features | Cost |
|------|----------|------|
| **Free** | 100 sandbox hours/month | $0 |
| **Pro** | 24-hour sessions, priority support | Usage-based |
| **Enterprise** | Self-hosted, SLA, custom limits | Custom |

### 8.2 Estimated Usage

| Agent Type | Sandboxes/Day | Avg Duration | Monthly Hours |
|------------|---------------|--------------|---------------|
| Discovery Agent | 1,000 | 30s | 250 |
| Recommendation Agent | 500 | 60s | 250 |
| Search Agent | 2,000 | 15s | 250 |
| Analysis Agent | 200 | 120s | 200 |
| SONA Updates | 100 | 60s | 50 |
| **Total** | **3,800** | - | **1,000** |

### 8.3 Monthly Cost Estimate

| Component | Cost |
|-----------|------|
| E2B Sandbox Hours (1,000 hrs) | $200-400 |
| Custom Templates | Included |
| Data Transfer | ~$50 |
| **Total E2B** | **$250-450/month** |

---

## Summary

E2B integration provides:

1. **Security**: Firecracker microVM isolation for untrusted code execution
2. **Flexibility**: Custom templates for different agent workloads
3. **Performance**: Sub-200ms sandbox startup for low-latency execution
4. **Scale**: Thousands of concurrent sandboxes for high-throughput
5. **Safety**: Isolated environment for SONA LoRA updates and ML computations

This enables our multi-agent system to safely execute LLM-generated code while maintaining production system integrity.

---

*Document Version: 1.0.0*
*Last Updated: December 2025*
*Integration Target: [E2B](https://e2b.dev)*
