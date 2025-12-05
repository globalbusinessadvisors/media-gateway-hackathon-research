# Google Cloud Platform Deployment Architecture

## Executive Summary

This document defines the complete GCP infrastructure for deploying the Media Gateway 4-layer microservices architecture. The deployment leverages **GKE Autopilot** for container orchestration and **Cloud Run** for serverless components, with supporting services including Cloud SQL, Memorystore, Pub/Sub, and comprehensive observability.

**Key Decisions:**
- **GKE Autopilot** for Layer 1-3 microservices (15+ services)
- **Cloud Run** for stateless API endpoints and event handlers
- **Cloud SQL PostgreSQL** for primary databases
- **Memorystore for Valkey** for caching and real-time state
- **Pub/Sub** for event-driven communication (complement to PubNub)
- **Workload Identity** for keyless authentication
- **Distroless containers** for security and minimal footprint

---

## Table of Contents

1. [Infrastructure Overview](#1-infrastructure-overview)
2. [GKE Cluster Architecture](#2-gke-cluster-architecture)
3. [Cloud Run Configuration](#3-cloud-run-configuration)
4. [Database Architecture](#4-database-architecture)
5. [Caching Layer](#5-caching-layer)
6. [Event Streaming](#6-event-streaming)
7. [Security Architecture](#7-security-architecture)
8. [Observability Stack](#8-observability-stack)
9. [CI/CD Pipeline](#9-cicd-pipeline)
10. [Terraform Modules](#10-terraform-modules)
11. [Kubernetes Manifests](#11-kubernetes-manifests)
12. [Cost Optimization](#12-cost-optimization)

---

## 1. Infrastructure Overview

### 1.1 Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              GOOGLE CLOUD PLATFORM                                   │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                      │
│  ┌─────────────────────────────────────────────────────────────────────────────┐   │
│  │                         GLOBAL LOAD BALANCER                                 │   │
│  │                    Cloud Armor (WAF + DDoS Protection)                       │   │
│  └────────────────────────────────┬────────────────────────────────────────────┘   │
│                                   │                                                 │
│  ┌────────────────────────────────┼────────────────────────────────────────────┐   │
│  │                                ▼                                             │   │
│  │  ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐       │   │
│  │  │   Cloud Run      │    │   GKE Autopilot  │    │   Cloud Run      │       │   │
│  │  │   (API Gateway)  │    │   (Core Services)│    │   (Webhooks)     │       │   │
│  │  └────────┬─────────┘    └────────┬─────────┘    └────────┬─────────┘       │   │
│  │           │                       │                       │                  │   │
│  │           └───────────────────────┼───────────────────────┘                  │   │
│  │                                   │                                          │   │
│  │                        ┌──────────▼──────────┐                              │   │
│  │                        │   ISTIO SERVICE     │                              │   │
│  │                        │       MESH          │                              │   │
│  │                        └──────────┬──────────┘                              │   │
│  │                                   │                                          │   │
│  │  ┌────────────────────────────────┼────────────────────────────────────┐    │   │
│  │  │                                ▼                                     │    │   │
│  │  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌────────────┐  │    │   │
│  │  │  │  Layer 1    │  │  Layer 2    │  │  Layer 3    │  │  Layer 4   │  │    │   │
│  │  │  │ Ingestion   │  │ Intelligence│  │Consolidation│  │    CLI     │  │    │   │
│  │  │  │ Auth, Sync  │  │ Recommend   │  │  Metadata   │  │   Apps     │  │    │   │
│  │  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └─────┬──────┘  │    │   │
│  │  │         │                │                │               │         │    │   │
│  │  │         └────────────────┼────────────────┼───────────────┘         │    │   │
│  │  │                          │                │                          │    │   │
│  │  │              GKE AUTOPILOT CLUSTER                                   │    │   │
│  │  └──────────────────────────┼────────────────┼──────────────────────────┘    │   │
│  │                             │                │                               │   │
│  │  PRIVATE VPC NETWORK        │                │                               │   │
│  └─────────────────────────────┼────────────────┼───────────────────────────────┘   │
│                                │                │                                    │
│  ┌─────────────────────────────┼────────────────┼───────────────────────────────┐   │
│  │                             ▼                ▼                                │   │
│  │  ┌───────────────┐    ┌───────────────┐    ┌───────────────┐                 │   │
│  │  │  Cloud SQL    │    │  Memorystore  │    │   Pub/Sub     │                 │   │
│  │  │  PostgreSQL   │    │    Valkey     │    │   Topics      │                 │   │
│  │  │  (Primary DB) │    │   (Cache)     │    │  (Events)     │                 │   │
│  │  └───────────────┘    └───────────────┘    └───────────────┘                 │   │
│  │                                                                               │   │
│  │  DATA LAYER (Private Service Access)                                         │   │
│  └───────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                      │
│  ┌───────────────────────────────────────────────────────────────────────────────┐   │
│  │  OBSERVABILITY                                                                │   │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐              │   │
│  │  │   Cloud    │  │   Cloud    │  │   Cloud    │  │   Error    │              │   │
│  │  │  Logging   │  │   Trace    │  │ Monitoring │  │ Reporting  │              │   │
│  │  └────────────┘  └────────────┘  └────────────┘  └────────────┘              │   │
│  └───────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                      │
│  ┌───────────────────────────────────────────────────────────────────────────────┐   │
│  │  CI/CD                                                                        │   │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐              │   │
│  │  │   Cloud    │  │  Artifact  │  │   Cloud    │  │  Secret    │              │   │
│  │  │   Build    │  │  Registry  │  │   Deploy   │  │  Manager   │              │   │
│  │  └────────────┘  └────────────┘  └────────────┘  └────────────┘              │   │
│  └───────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Service Distribution

| Layer | Service | Deployment Target | Reason |
|-------|---------|-------------------|--------|
| Layer 1 | ingestion-* | GKE Autopilot | Long-running, stateful connections |
| Layer 1 | auth-service | GKE Autopilot | Session management, token rotation |
| Layer 1 | sync-engine | GKE Autopilot | WebSocket connections |
| Layer 1 | entity-resolver | GKE Autopilot | ML workloads, GPU optional |
| Layer 2 | recommendation-engine | GKE Autopilot | GNN computation, memory-intensive |
| Layer 2 | agent-orchestrator | GKE Autopilot | Long-running agent sessions |
| Layer 2 | semantic-search | GKE Autopilot | Vector operations |
| Layer 3 | metadata-fabric | GKE Autopilot | High availability |
| Layer 3 | availability-index | GKE Autopilot | Real-time updates |
| Layer 4 | api-gateway | Cloud Run | Stateless, auto-scaling |
| Layer 4 | webhook-handlers | Cloud Run | Event-driven, burst traffic |
| Infra | batch-jobs | Cloud Run Jobs | Scheduled tasks |

---

## 2. GKE Cluster Architecture

### 2.1 Cluster Configuration

**GKE Autopilot** is selected for the following reasons:
- Simplified node management (Google handles patching, scaling)
- Pay-per-pod pricing (no idle node costs)
- Production-ready defaults (security policies, monitoring)
- 5,000 pods/cluster capacity (sufficient for 15+ services)

```hcl
# terraform/modules/gke/main.tf

resource "google_container_cluster" "media_gateway" {
  name     = "media-gateway-cluster"
  location = var.region

  # Enable Autopilot mode
  enable_autopilot = true

  # Workload Identity for keyless auth
  workload_identity_config {
    workload_pool = "${var.project_id}.svc.id.goog"
  }

  # Private cluster configuration
  private_cluster_config {
    enable_private_nodes    = true
    enable_private_endpoint = false
    master_ipv4_cidr_block  = "172.16.0.0/28"
  }

  # Network configuration
  network    = google_compute_network.vpc.id
  subnetwork = google_compute_subnetwork.gke.id

  ip_allocation_policy {
    cluster_secondary_range_name  = "pods"
    services_secondary_range_name = "services"
  }

  # Addons
  addons_config {
    http_load_balancing {
      disabled = false
    }
    horizontal_pod_autoscaling {
      disabled = false
    }
    network_policy_config {
      disabled = false
    }
  }

  # Release channel for automatic upgrades
  release_channel {
    channel = "REGULAR"
  }

  # Maintenance window
  maintenance_policy {
    recurring_window {
      start_time = "2025-01-01T04:00:00Z"
      end_time   = "2025-01-01T08:00:00Z"
      recurrence = "FREQ=WEEKLY;BYDAY=SA,SU"
    }
  }

  # Logging and monitoring
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

### 2.2 Namespace Strategy

```yaml
# k8s/namespaces.yaml

---
apiVersion: v1
kind: Namespace
metadata:
  name: media-gateway-layer1
  labels:
    layer: "1"
    istio-injection: enabled
---
apiVersion: v1
kind: Namespace
metadata:
  name: media-gateway-layer2
  labels:
    layer: "2"
    istio-injection: enabled
---
apiVersion: v1
kind: Namespace
metadata:
  name: media-gateway-layer3
  labels:
    layer: "3"
    istio-injection: enabled
---
apiVersion: v1
kind: Namespace
metadata:
  name: media-gateway-layer4
  labels:
    layer: "4"
    istio-injection: enabled
---
apiVersion: v1
kind: Namespace
metadata:
  name: media-gateway-infra
  labels:
    layer: infra
```

### 2.3 Resource Quotas

```yaml
# k8s/resource-quotas.yaml

apiVersion: v1
kind: ResourceQuota
metadata:
  name: layer1-quota
  namespace: media-gateway-layer1
spec:
  hard:
    requests.cpu: "20"
    requests.memory: 40Gi
    limits.cpu: "40"
    limits.memory: 80Gi
    pods: "100"
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: layer2-quota
  namespace: media-gateway-layer2
spec:
  hard:
    requests.cpu: "30"
    requests.memory: 60Gi
    limits.cpu: "60"
    limits.memory: 120Gi
    pods: "50"
```

---

## 3. Cloud Run Configuration

### 3.1 API Gateway Service

```hcl
# terraform/modules/cloudrun/api_gateway.tf

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

      ports {
        container_port = 8080
      }

      env {
        name  = "GKE_ENDPOINT"
        value = "http://istio-ingressgateway.istio-system.svc.cluster.local"
      }

      env {
        name = "DB_PASSWORD"
        value_source {
          secret_key_ref {
            secret  = google_secret_manager_secret.db_password.secret_id
            version = "latest"
          }
        }
      }

      resources {
        limits = {
          cpu    = "2"
          memory = "1Gi"
        }
        cpu_idle = true  # Scale to zero
      }

      startup_probe {
        http_get {
          path = "/health"
          port = 8080
        }
        initial_delay_seconds = 5
        timeout_seconds       = 3
        period_seconds        = 10
        failure_threshold     = 3
      }

      liveness_probe {
        http_get {
          path = "/health"
          port = 8080
        }
        period_seconds    = 30
        timeout_seconds   = 3
        failure_threshold = 3
      }
    }

    service_account = google_service_account.api_gateway.email

    max_instance_request_concurrency = 80
    timeout                          = "30s"
  }

  traffic {
    type    = "TRAFFIC_TARGET_ALLOCATION_TYPE_LATEST"
    percent = 100
  }
}

# IAM for public access
resource "google_cloud_run_service_iam_member" "api_gateway_public" {
  location = google_cloud_run_v2_service.api_gateway.location
  service  = google_cloud_run_v2_service.api_gateway.name
  role     = "roles/run.invoker"
  member   = "allUsers"
}
```

### 3.2 Webhook Handler (Event-Driven)

```hcl
# terraform/modules/cloudrun/webhook_handler.tf

resource "google_cloud_run_v2_service" "webhook_handler" {
  name     = "webhook-handler"
  location = var.region

  template {
    scaling {
      min_instance_count = 0  # Scale to zero
      max_instance_count = 50
    }

    vpc_access {
      connector = google_vpc_access_connector.connector.id
      egress    = "ALL_TRAFFIC"
    }

    containers {
      image = "${var.artifact_registry}/webhook-handler:${var.image_tag}"

      resources {
        limits = {
          cpu    = "1"
          memory = "512Mi"
        }
        cpu_idle = true
      }
    }

    service_account = google_service_account.webhook_handler.email

    max_instance_request_concurrency = 100
    timeout                          = "60s"
  }
}

# Pub/Sub push subscription
resource "google_pubsub_subscription" "webhook_push" {
  name  = "webhook-push-subscription"
  topic = google_pubsub_topic.platform_events.name

  push_config {
    push_endpoint = google_cloud_run_v2_service.webhook_handler.uri

    oidc_token {
      service_account_email = google_service_account.pubsub_invoker.email
    }
  }

  ack_deadline_seconds = 60

  retry_policy {
    minimum_backoff = "10s"
    maximum_backoff = "600s"
  }

  dead_letter_policy {
    dead_letter_topic     = google_pubsub_topic.dead_letter.id
    max_delivery_attempts = 5
  }
}
```

### 3.3 Cloud Run Jobs (Batch Processing)

```hcl
# terraform/modules/cloudrun/batch_jobs.tf

resource "google_cloud_run_v2_job" "catalog_sync" {
  name     = "catalog-sync-job"
  location = var.region

  template {
    template {
      containers {
        image = "${var.artifact_registry}/catalog-sync:${var.image_tag}"

        resources {
          limits = {
            cpu    = "4"
            memory = "4Gi"
          }
        }

        env {
          name  = "BATCH_SIZE"
          value = "1000"
        }
      }

      vpc_access {
        connector = google_vpc_access_connector.connector.id
        egress    = "ALL_TRAFFIC"
      }

      service_account = google_service_account.batch_jobs.email
      timeout         = "3600s"  # 1 hour
      max_retries     = 3
    }

    parallelism = 5
    task_count  = 10
  }
}

# Cloud Scheduler trigger
resource "google_cloud_scheduler_job" "catalog_sync_trigger" {
  name             = "catalog-sync-trigger"
  schedule         = "0 */6 * * *"  # Every 6 hours
  time_zone        = "UTC"
  attempt_deadline = "3600s"

  http_target {
    http_method = "POST"
    uri         = "https://${var.region}-run.googleapis.com/apis/run.googleapis.com/v1/namespaces/${var.project_id}/jobs/catalog-sync-job:run"

    oauth_token {
      service_account_email = google_service_account.scheduler.email
    }
  }
}
```

---

## 4. Database Architecture

### 4.1 Cloud SQL PostgreSQL

```hcl
# terraform/modules/database/cloudsql.tf

resource "google_sql_database_instance" "primary" {
  name             = "media-gateway-db"
  database_version = "POSTGRES_15"
  region           = var.region

  settings {
    tier              = "db-custom-4-16384"  # 4 vCPU, 16GB RAM
    availability_type = "REGIONAL"           # High availability
    disk_size         = 200
    disk_type         = "PD_SSD"
    disk_autoresize   = true

    backup_configuration {
      enabled                        = true
      start_time                     = "02:00"
      location                       = var.region
      point_in_time_recovery_enabled = true
      transaction_log_retention_days = 7

      backup_retention_settings {
        retained_backups = 30
        retention_unit   = "COUNT"
      }
    }

    ip_configuration {
      ipv4_enabled    = false
      private_network = google_compute_network.vpc.id

      authorized_networks {
        name  = "allow-none"
        value = "0.0.0.0/0"
      }
    }

    database_flags {
      name  = "max_connections"
      value = "500"
    }

    database_flags {
      name  = "log_checkpoints"
      value = "on"
    }

    database_flags {
      name  = "log_connections"
      value = "on"
    }

    database_flags {
      name  = "log_disconnections"
      value = "on"
    }

    insights_config {
      query_insights_enabled  = true
      query_string_length     = 4500
      record_application_tags = true
      record_client_address   = true
    }

    maintenance_window {
      day          = 7  # Sunday
      hour         = 3  # 3 AM
      update_track = "stable"
    }
  }

  deletion_protection = true
}

# Databases per layer
resource "google_sql_database" "layer1_db" {
  name     = "layer1"
  instance = google_sql_database_instance.primary.name
}

resource "google_sql_database" "layer2_db" {
  name     = "layer2"
  instance = google_sql_database_instance.primary.name
}

resource "google_sql_database" "layer3_db" {
  name     = "layer3"
  instance = google_sql_database_instance.primary.name
}

# Users
resource "google_sql_user" "app_user" {
  name     = "media_gateway_app"
  instance = google_sql_database_instance.primary.name
  password = random_password.db_password.result
}

resource "random_password" "db_password" {
  length  = 32
  special = true
}

resource "google_secret_manager_secret_version" "db_password" {
  secret      = google_secret_manager_secret.db_password.id
  secret_data = random_password.db_password.result
}
```

### 4.2 Database Schema Extensions

```sql
-- migrations/001_enable_extensions.sql

-- Enable vector operations for semantic search
CREATE EXTENSION IF NOT EXISTS vector;

-- Enable UUID generation
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Enable full-text search
CREATE EXTENSION IF NOT EXISTS pg_trgm;

-- Enable JSON operations
CREATE EXTENSION IF NOT EXISTS jsonb_plpgsql;
```

---

## 5. Caching Layer

### 5.1 Memorystore for Valkey (Redis-compatible)

```hcl
# terraform/modules/cache/memorystore.tf

resource "google_redis_instance" "cache" {
  name           = "media-gateway-cache"
  tier           = "STANDARD_HA"  # High availability
  memory_size_gb = 10
  region         = var.region
  redis_version  = "REDIS_7_2"

  authorized_network = google_compute_network.vpc.id
  connect_mode       = "PRIVATE_SERVICE_ACCESS"

  transit_encryption_mode = "SERVER_AUTHENTICATION"
  auth_enabled            = true

  replica_count       = 2
  read_replicas_mode  = "READ_REPLICAS_ENABLED"

  maintenance_policy {
    weekly_maintenance_window {
      day = "SUNDAY"
      start_time {
        hours   = 3
        minutes = 0
      }
    }
  }

  persistence_config {
    persistence_mode    = "RDB"
    rdb_snapshot_period = "ONE_HOUR"
  }

  labels = {
    environment = var.environment
    layer       = "infrastructure"
  }
}

# Store auth string in Secret Manager
resource "google_secret_manager_secret_version" "redis_auth" {
  secret      = google_secret_manager_secret.redis_auth.id
  secret_data = google_redis_instance.cache.auth_string
}
```

### 5.2 Cache Key Strategy

```rust
// src/cache/keys.rs

pub mod cache_keys {
    /// Content metadata cache (TTL: 1 hour)
    pub fn content_metadata(content_id: &str) -> String {
        format!("content:metadata:{}", content_id)
    }

    /// Platform availability cache (TTL: 15 minutes)
    pub fn platform_availability(content_id: &str, region: &str) -> String {
        format!("availability:{}:{}", content_id, region)
    }

    /// User session cache (TTL: 24 hours)
    pub fn user_session(user_id: &str) -> String {
        format!("session:{}", user_id)
    }

    /// Recommendation cache (TTL: 30 minutes)
    pub fn recommendations(user_id: &str) -> String {
        format!("recs:{}", user_id)
    }

    /// Rate limiting (TTL: sliding window)
    pub fn rate_limit(client_ip: &str, endpoint: &str) -> String {
        format!("ratelimit:{}:{}", client_ip, endpoint)
    }

    /// Feature flags (TTL: 5 minutes)
    pub fn feature_flags() -> String {
        "config:feature_flags".to_string()
    }
}
```

---

## 6. Event Streaming

### 6.1 Pub/Sub Topics

```hcl
# terraform/modules/pubsub/topics.tf

# Content availability changes
resource "google_pubsub_topic" "content_availability" {
  name = "content-availability-changes"

  message_retention_duration = "86400s"  # 24 hours

  schema_settings {
    schema   = google_pubsub_schema.content_availability.id
    encoding = "JSON"
  }

  labels = {
    layer = "layer1"
  }
}

resource "google_pubsub_schema" "content_availability" {
  name       = "content-availability-schema"
  type       = "AVRO"
  definition = <<EOF
{
  "type": "record",
  "name": "ContentAvailabilityEvent",
  "fields": [
    {"name": "content_id", "type": "string"},
    {"name": "platform", "type": "string"},
    {"name": "region", "type": "string"},
    {"name": "available", "type": "boolean"},
    {"name": "timestamp", "type": "long"}
  ]
}
EOF
}

# User activity events
resource "google_pubsub_topic" "user_activity" {
  name                       = "user-activity-events"
  message_retention_duration = "604800s"  # 7 days
}

# Recommendation updates
resource "google_pubsub_topic" "recommendation_updates" {
  name                       = "recommendation-updates"
  message_retention_duration = "86400s"
}

# Dead letter queue
resource "google_pubsub_topic" "dead_letter" {
  name                       = "dead-letter-queue"
  message_retention_duration = "1209600s"  # 14 days
}
```

### 6.2 Subscriptions

```hcl
# terraform/modules/pubsub/subscriptions.tf

# Pull subscription for GKE services
resource "google_pubsub_subscription" "availability_processor" {
  name  = "availability-processor"
  topic = google_pubsub_topic.content_availability.name

  ack_deadline_seconds       = 60
  message_retention_duration = "604800s"  # 7 days
  retain_acked_messages      = false

  expiration_policy {
    ttl = ""  # Never expire
  }

  retry_policy {
    minimum_backoff = "10s"
    maximum_backoff = "600s"
  }

  dead_letter_policy {
    dead_letter_topic     = google_pubsub_topic.dead_letter.id
    max_delivery_attempts = 5
  }

  enable_exactly_once_delivery = true
}

# Push subscription for Cloud Run
resource "google_pubsub_subscription" "activity_webhook" {
  name  = "activity-webhook"
  topic = google_pubsub_topic.user_activity.name

  push_config {
    push_endpoint = "${google_cloud_run_v2_service.webhook_handler.uri}/activity"

    oidc_token {
      service_account_email = google_service_account.pubsub_invoker.email
    }

    attributes = {
      x-goog-version = "v1"
    }
  }

  ack_deadline_seconds = 60
}
```

---

## 7. Security Architecture

### 7.1 Workload Identity

```hcl
# terraform/modules/security/workload_identity.tf

# Service accounts per layer
resource "google_service_account" "layer1_sa" {
  account_id   = "media-gateway-layer1"
  display_name = "Media Gateway Layer 1 Services"
}

resource "google_service_account" "layer2_sa" {
  account_id   = "media-gateway-layer2"
  display_name = "Media Gateway Layer 2 Services"
}

resource "google_service_account" "layer3_sa" {
  account_id   = "media-gateway-layer3"
  display_name = "Media Gateway Layer 3 Services"
}

# IAM bindings for Layer 1
resource "google_project_iam_member" "layer1_sql" {
  project = var.project_id
  role    = "roles/cloudsql.client"
  member  = "serviceAccount:${google_service_account.layer1_sa.email}"
}

resource "google_project_iam_member" "layer1_pubsub" {
  project = var.project_id
  role    = "roles/pubsub.publisher"
  member  = "serviceAccount:${google_service_account.layer1_sa.email}"
}

resource "google_project_iam_member" "layer1_secrets" {
  project = var.project_id
  role    = "roles/secretmanager.secretAccessor"
  member  = "serviceAccount:${google_service_account.layer1_sa.email}"
}

# Workload Identity binding
resource "google_service_account_iam_member" "layer1_workload_identity" {
  service_account_id = google_service_account.layer1_sa.name
  role               = "roles/iam.workloadIdentityUser"
  member             = "serviceAccount:${var.project_id}.svc.id.goog[media-gateway-layer1/layer1-ksa]"
}
```

### 7.2 Cloud Armor Security Policy

```hcl
# terraform/modules/security/cloud_armor.tf

resource "google_compute_security_policy" "api_policy" {
  name = "media-gateway-api-policy"

  # Adaptive protection (ML-based)
  adaptive_protection_config {
    layer_7_ddos_defense_config {
      enable = true
    }
  }

  # Rule 1: Block known bad actors
  rule {
    action   = "deny(403)"
    priority = 100
    match {
      versioned_expr = "SRC_IPS_V1"
      config {
        src_ip_ranges = var.blocked_ip_ranges
      }
    }
    description = "Block known malicious IPs"
  }

  # Rule 2: XSS protection
  rule {
    action   = "deny(403)"
    priority = 200
    match {
      expr {
        expression = "evaluatePreconfiguredExpr('xss-v33-stable')"
      }
    }
    description = "XSS protection"
  }

  # Rule 3: SQL injection protection
  rule {
    action   = "deny(403)"
    priority = 201
    match {
      expr {
        expression = "evaluatePreconfiguredExpr('sqli-v33-stable')"
      }
    }
    description = "SQL injection protection"
  }

  # Rule 4: Rate limiting
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
    description = "Rate limiting - 1000 req/min per IP"
  }

  # Rule 5: Bot management
  rule {
    action   = "deny(403)"
    priority = 400
    match {
      expr {
        expression = "evaluatePreconfiguredExpr('rce-v33-stable')"
      }
    }
    description = "Remote code execution protection"
  }

  # Default rule: Allow
  rule {
    action   = "allow"
    priority = 2147483647
    match {
      versioned_expr = "SRC_IPS_V1"
      config {
        src_ip_ranges = ["*"]
      }
    }
    description = "Default allow"
  }
}
```

### 7.3 Secret Manager

```hcl
# terraform/modules/security/secrets.tf

locals {
  secrets = {
    db_password       = "db-password"
    redis_auth        = "redis-auth"
    jwt_signing_key   = "jwt-signing-key"
    oauth_client_id   = "oauth-client-id"
    oauth_secret      = "oauth-client-secret"
    pubnub_publish    = "pubnub-publish-key"
    pubnub_subscribe  = "pubnub-subscribe-key"
    aggregator_api    = "aggregator-api-key"
  }
}

resource "google_secret_manager_secret" "secrets" {
  for_each  = local.secrets
  secret_id = each.value

  replication {
    auto {}
  }

  labels = {
    environment = var.environment
    managed_by  = "terraform"
  }
}

# Grant access to service accounts
resource "google_secret_manager_secret_iam_member" "layer1_access" {
  for_each  = google_secret_manager_secret.secrets
  secret_id = each.value.secret_id
  role      = "roles/secretmanager.secretAccessor"
  member    = "serviceAccount:${google_service_account.layer1_sa.email}"
}
```

---

## 8. Observability Stack

### 8.1 Cloud Logging Configuration

```hcl
# terraform/modules/observability/logging.tf

# Log sink for BigQuery (analytics)
resource "google_logging_project_sink" "bigquery_sink" {
  name                   = "media-gateway-logs-bq"
  destination            = "bigquery.googleapis.com/projects/${var.project_id}/datasets/${google_bigquery_dataset.logs.dataset_id}"
  filter                 = "resource.type=\"k8s_container\" AND resource.labels.namespace_name=~\"media-gateway-.*\""
  unique_writer_identity = true

  bigquery_options {
    use_partitioned_tables = true
  }
}

resource "google_bigquery_dataset" "logs" {
  dataset_id = "media_gateway_logs"
  location   = var.region

  default_table_expiration_ms = 2592000000  # 30 days

  labels = {
    environment = var.environment
  }
}

# Log-based metric for error rate
resource "google_logging_metric" "error_rate" {
  name   = "media-gateway/error-rate"
  filter = <<EOF
resource.type="k8s_container"
resource.labels.namespace_name=~"media-gateway-.*"
severity>=ERROR
EOF

  metric_descriptor {
    metric_kind = "DELTA"
    value_type  = "INT64"
    labels {
      key         = "service"
      value_type  = "STRING"
      description = "Service name"
    }
  }

  label_extractors = {
    "service" = "EXTRACT(resource.labels.container_name)"
  }
}
```

### 8.2 Cloud Monitoring Dashboards

```hcl
# terraform/modules/observability/dashboards.tf

resource "google_monitoring_dashboard" "overview" {
  dashboard_json = jsonencode({
    displayName = "Media Gateway Overview"
    gridLayout = {
      columns = 3
      widgets = [
        {
          title = "Request Rate"
          xyChart = {
            dataSets = [{
              timeSeriesQuery = {
                timeSeriesFilter = {
                  filter = "metric.type=\"run.googleapis.com/request_count\" resource.type=\"cloud_run_revision\""
                }
              }
            }]
          }
        },
        {
          title = "Error Rate"
          xyChart = {
            dataSets = [{
              timeSeriesQuery = {
                timeSeriesFilter = {
                  filter = "metric.type=\"logging.googleapis.com/user/media-gateway/error-rate\""
                }
              }
            }]
          }
        },
        {
          title = "Latency P99"
          xyChart = {
            dataSets = [{
              timeSeriesQuery = {
                timeSeriesFilter = {
                  filter = "metric.type=\"run.googleapis.com/request_latencies\" resource.type=\"cloud_run_revision\""
                  aggregation = {
                    perSeriesAligner   = "ALIGN_PERCENTILE_99"
                    alignmentPeriod    = "60s"
                    crossSeriesReducer = "REDUCE_MEAN"
                  }
                }
              }
            }]
          }
        }
      ]
    }
  })
}
```

### 8.3 Alerting Policies

```hcl
# terraform/modules/observability/alerts.tf

resource "google_monitoring_alert_policy" "high_error_rate" {
  display_name = "High Error Rate - Media Gateway"
  combiner     = "OR"

  conditions {
    display_name = "Error rate > 1%"

    condition_threshold {
      filter          = "metric.type=\"logging.googleapis.com/user/media-gateway/error-rate\" resource.type=\"k8s_container\""
      comparison      = "COMPARISON_GT"
      threshold_value = 0.01
      duration        = "300s"  # 5 minutes

      aggregations {
        alignment_period   = "60s"
        per_series_aligner = "ALIGN_RATE"
      }
    }
  }

  notification_channels = [google_monitoring_notification_channel.email.id]

  alert_strategy {
    auto_close = "1800s"  # 30 minutes
  }
}

resource "google_monitoring_alert_policy" "high_latency" {
  display_name = "High Latency - Media Gateway"
  combiner     = "OR"

  conditions {
    display_name = "P99 latency > 1s"

    condition_threshold {
      filter          = "metric.type=\"run.googleapis.com/request_latencies\" resource.type=\"cloud_run_revision\""
      comparison      = "COMPARISON_GT"
      threshold_value = 1000  # 1 second
      duration        = "300s"

      aggregations {
        alignment_period     = "60s"
        per_series_aligner   = "ALIGN_PERCENTILE_99"
        cross_series_reducer = "REDUCE_MEAN"
      }
    }
  }

  notification_channels = [google_monitoring_notification_channel.email.id]
}

resource "google_monitoring_notification_channel" "email" {
  display_name = "Media Gateway Alerts"
  type         = "email"

  labels = {
    email_address = var.alert_email
  }
}
```

---

## 9. CI/CD Pipeline

### 9.1 Cloud Build Configuration

```yaml
# cloudbuild.yaml

substitutions:
  _REGION: us-central1
  _ARTIFACT_REGISTRY: ${_REGION}-docker.pkg.dev/${PROJECT_ID}/media-gateway

steps:
  # Step 1: Build all Rust services in parallel
  - name: 'gcr.io/cloud-builders/docker'
    id: 'build-layer1-ingestion'
    args:
      - 'build'
      - '-t'
      - '${_ARTIFACT_REGISTRY}/ingestion-core:${SHORT_SHA}'
      - '-t'
      - '${_ARTIFACT_REGISTRY}/ingestion-core:latest'
      - '-f'
      - 'layer-1/ingestion/Dockerfile'
      - '.'
    waitFor: ['-']

  - name: 'gcr.io/cloud-builders/docker'
    id: 'build-layer1-auth'
    args:
      - 'build'
      - '-t'
      - '${_ARTIFACT_REGISTRY}/auth-service:${SHORT_SHA}'
      - '-t'
      - '${_ARTIFACT_REGISTRY}/auth-service:latest'
      - '-f'
      - 'layer-1/identity/auth-service/Dockerfile'
      - '.'
    waitFor: ['-']

  - name: 'gcr.io/cloud-builders/docker'
    id: 'build-layer2-recommendations'
    args:
      - 'build'
      - '-t'
      - '${_ARTIFACT_REGISTRY}/recommendation-engine:${SHORT_SHA}'
      - '-t'
      - '${_ARTIFACT_REGISTRY}/recommendation-engine:latest'
      - '-f'
      - 'layer-2/recommendation-engine/Dockerfile'
      - '.'
    waitFor: ['-']

  # Step 2: Push images
  - name: 'gcr.io/cloud-builders/docker'
    id: 'push-images'
    args: ['push', '--all-tags', '${_ARTIFACT_REGISTRY}/ingestion-core']
    waitFor: ['build-layer1-ingestion']

  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', '--all-tags', '${_ARTIFACT_REGISTRY}/auth-service']
    waitFor: ['build-layer1-auth']

  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', '--all-tags', '${_ARTIFACT_REGISTRY}/recommendation-engine']
    waitFor: ['build-layer2-recommendations']

  # Step 3: Deploy to GKE
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    id: 'deploy-gke'
    entrypoint: 'bash'
    args:
      - '-c'
      - |
        gcloud container clusters get-credentials media-gateway-cluster --region ${_REGION}
        kubectl set image deployment/ingestion-core ingestion-core=${_ARTIFACT_REGISTRY}/ingestion-core:${SHORT_SHA} -n media-gateway-layer1
        kubectl set image deployment/auth-service auth-service=${_ARTIFACT_REGISTRY}/auth-service:${SHORT_SHA} -n media-gateway-layer1
        kubectl set image deployment/recommendation-engine recommendation-engine=${_ARTIFACT_REGISTRY}/recommendation-engine:${SHORT_SHA} -n media-gateway-layer2
    waitFor: ['push-images']

  # Step 4: Deploy Cloud Run services
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    id: 'deploy-cloudrun'
    entrypoint: 'bash'
    args:
      - '-c'
      - |
        gcloud run deploy api-gateway \
          --image ${_ARTIFACT_REGISTRY}/api-gateway:${SHORT_SHA} \
          --region ${_REGION} \
          --platform managed
    waitFor: ['push-images']

images:
  - '${_ARTIFACT_REGISTRY}/ingestion-core:${SHORT_SHA}'
  - '${_ARTIFACT_REGISTRY}/auth-service:${SHORT_SHA}'
  - '${_ARTIFACT_REGISTRY}/recommendation-engine:${SHORT_SHA}'

options:
  machineType: 'E2_HIGHCPU_8'
  logging: CLOUD_LOGGING_ONLY

timeout: 3600s
```

### 9.2 Dockerfile Template (Rust Services)

```dockerfile
# Dockerfile.rust-service

# Stage 1: Build dependencies
FROM rust:1.75-slim as planner
WORKDIR /app
RUN cargo install cargo-chef
COPY . .
RUN cargo chef prepare --recipe-path recipe.json

# Stage 2: Cache dependencies
FROM rust:1.75-slim as cacher
WORKDIR /app
RUN cargo install cargo-chef
COPY --from=planner /app/recipe.json recipe.json
RUN cargo chef cook --release --recipe-path recipe.json

# Stage 3: Build application
FROM rust:1.75-slim as builder
WORKDIR /app
COPY . .
COPY --from=cacher /app/target target
COPY --from=cacher /usr/local/cargo /usr/local/cargo
RUN cargo build --release

# Stage 4: Runtime
FROM gcr.io/distroless/cc-debian12:nonroot

LABEL org.opencontainers.image.source="https://github.com/globalbusinessadvisors/media-gateway"
LABEL org.opencontainers.image.description="Media Gateway Microservice"

COPY --from=builder /app/target/release/service /app/service

USER nonroot:nonroot
EXPOSE 8080 50051

ENTRYPOINT ["/app/service"]
```

---

## 10. Terraform Modules

### 10.1 Module Structure

```
terraform/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   └── prod/
├── modules/
│   ├── gke/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── cloudrun/
│   │   ├── main.tf
│   │   ├── api_gateway.tf
│   │   ├── webhook_handler.tf
│   │   └── batch_jobs.tf
│   ├── database/
│   │   ├── cloudsql.tf
│   │   └── outputs.tf
│   ├── cache/
│   │   └── memorystore.tf
│   ├── pubsub/
│   │   ├── topics.tf
│   │   └── subscriptions.tf
│   ├── security/
│   │   ├── workload_identity.tf
│   │   ├── cloud_armor.tf
│   │   └── secrets.tf
│   ├── network/
│   │   ├── vpc.tf
│   │   └── firewall.tf
│   └── observability/
│       ├── logging.tf
│       ├── dashboards.tf
│       └── alerts.tf
└── shared/
    ├── providers.tf
    └── backend.tf
```

### 10.2 Root Module

```hcl
# terraform/environments/prod/main.tf

terraform {
  required_version = ">= 1.5.0"

  backend "gcs" {
    bucket = "media-gateway-terraform-state"
    prefix = "prod"
  }

  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
    google-beta = {
      source  = "hashicorp/google-beta"
      version = "~> 5.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.25"
    }
  }
}

provider "google" {
  project = var.project_id
  region  = var.region
}

provider "google-beta" {
  project = var.project_id
  region  = var.region
}

# Network
module "network" {
  source     = "../../modules/network"
  project_id = var.project_id
  region     = var.region
}

# GKE Cluster
module "gke" {
  source     = "../../modules/gke"
  project_id = var.project_id
  region     = var.region
  network    = module.network.vpc_id
  subnetwork = module.network.gke_subnet_id
}

# Cloud Run Services
module "cloudrun" {
  source            = "../../modules/cloudrun"
  project_id        = var.project_id
  region            = var.region
  vpc_connector     = module.network.vpc_connector_id
  artifact_registry = var.artifact_registry
  image_tag         = var.image_tag
}

# Database
module "database" {
  source     = "../../modules/database"
  project_id = var.project_id
  region     = var.region
  network    = module.network.vpc_id
}

# Cache
module "cache" {
  source     = "../../modules/cache"
  project_id = var.project_id
  region     = var.region
  network    = module.network.vpc_id
}

# Pub/Sub
module "pubsub" {
  source     = "../../modules/pubsub"
  project_id = var.project_id
}

# Security
module "security" {
  source     = "../../modules/security"
  project_id = var.project_id
}

# Observability
module "observability" {
  source      = "../../modules/observability"
  project_id  = var.project_id
  region      = var.region
  alert_email = var.alert_email
}
```

---

## 11. Kubernetes Manifests

### 11.1 Layer 1 Deployment Example

```yaml
# k8s/layer1/ingestion-core.yaml

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: layer1-ksa
  namespace: media-gateway-layer1
  annotations:
    iam.gke.io/gcp-service-account: media-gateway-layer1@PROJECT_ID.iam.gserviceaccount.com

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ingestion-core
  namespace: media-gateway-layer1
  labels:
    app: ingestion-core
    layer: "1"
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ingestion-core
  template:
    metadata:
      labels:
        app: ingestion-core
        layer: "1"
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "9090"
    spec:
      serviceAccountName: layer1-ksa
      securityContext:
        runAsNonRoot: true
        runAsUser: 65532
        fsGroup: 65532

      containers:
        - name: ingestion-core
          image: us-central1-docker.pkg.dev/PROJECT_ID/media-gateway/ingestion-core:latest
          imagePullPolicy: Always

          ports:
            - name: http
              containerPort: 8080
            - name: grpc
              containerPort: 50051
            - name: metrics
              containerPort: 9090

          env:
            - name: RUST_LOG
              value: "info,ingestion_core=debug"
            - name: DB_HOST
              valueFrom:
                secretKeyRef:
                  name: db-credentials
                  key: host
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-credentials
                  key: password
            - name: REDIS_HOST
              value: "media-gateway-cache.redis.svc.cluster.local"
            - name: PUBSUB_PROJECT_ID
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace

          resources:
            requests:
              cpu: 250m
              memory: 256Mi
            limits:
              cpu: 1000m
              memory: 512Mi

          livenessProbe:
            httpGet:
              path: /health/live
              port: http
            initialDelaySeconds: 15
            periodSeconds: 20
            timeoutSeconds: 5
            failureThreshold: 3

          readinessProbe:
            httpGet:
              path: /health/ready
              port: http
            initialDelaySeconds: 5
            periodSeconds: 10
            timeoutSeconds: 3
            failureThreshold: 3

          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop:
                - ALL

---
apiVersion: v1
kind: Service
metadata:
  name: ingestion-core
  namespace: media-gateway-layer1
spec:
  type: ClusterIP
  selector:
    app: ingestion-core
  ports:
    - name: http
      port: 80
      targetPort: 8080
    - name: grpc
      port: 50051
      targetPort: 50051

---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: ingestion-core-hpa
  namespace: media-gateway-layer1
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ingestion-core
  minReplicas: 2
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

---

## 12. Cost Optimization

### 12.1 Estimated Monthly Costs

| Component | Configuration | Estimated Cost |
|-----------|---------------|----------------|
| GKE Autopilot | 15 services, avg 2 replicas | $800-1,200 |
| Cloud Run | API Gateway + Webhooks | $200-400 |
| Cloud SQL | db-custom-4-16384, HA | $400-500 |
| Memorystore | 10GB, Standard HA | $300-350 |
| Pub/Sub | 10M messages/month | $50-100 |
| Cloud Armor | Standard tier | $50 |
| Artifact Registry | 50GB storage | $5 |
| Cloud Logging | 100GB/month | $50 |
| **Total** | | **$1,855-2,605/month** |

### 12.2 Cost Optimization Strategies

1. **GKE Autopilot Spot Pods**
   - Use Spot VMs for non-critical workloads
   - 60-80% cost reduction for fault-tolerant services

2. **Cloud Run Min Instances**
   - Set `min_instance_count = 0` for low-traffic services
   - Pay only when processing requests

3. **Committed Use Discounts**
   - 1-year commitment: 37% discount
   - 3-year commitment: 55% discount

4. **Right-sizing**
   - Monitor actual usage with Cloud Monitoring
   - Adjust resource requests/limits monthly

5. **Regional vs Multi-Regional**
   - Use single region for development
   - Multi-regional only for production

---

## Appendix: Quick Start Commands

```bash
# Initialize Terraform
cd terraform/environments/dev
terraform init
terraform plan
terraform apply

# Get GKE credentials
gcloud container clusters get-credentials media-gateway-cluster --region us-central1

# Deploy to GKE
kubectl apply -f k8s/namespaces.yaml
kubectl apply -f k8s/layer1/
kubectl apply -f k8s/layer2/
kubectl apply -f k8s/layer3/

# Check deployment status
kubectl get pods -n media-gateway-layer1
kubectl get pods -n media-gateway-layer2
kubectl get pods -n media-gateway-layer3

# View logs
kubectl logs -f deployment/ingestion-core -n media-gateway-layer1

# Deploy Cloud Run
gcloud run deploy api-gateway \
  --image us-central1-docker.pkg.dev/PROJECT_ID/media-gateway/api-gateway:latest \
  --region us-central1

# Run Cloud Build
gcloud builds submit --config cloudbuild.yaml
```

---

## References

- [GKE Autopilot Documentation](https://cloud.google.com/kubernetes-engine/docs/concepts/autopilot-overview)
- [Cloud Run Documentation](https://cloud.google.com/run/docs)
- [Cloud SQL for PostgreSQL](https://cloud.google.com/sql/docs/postgres)
- [Memorystore for Redis](https://cloud.google.com/memorystore/docs/redis)
- [Cloud Pub/Sub](https://cloud.google.com/pubsub/docs)
- [Workload Identity](https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity)
- [Cloud Armor](https://cloud.google.com/armor/docs)
- [Cloud Build](https://cloud.google.com/build/docs)
