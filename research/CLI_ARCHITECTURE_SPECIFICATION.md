# CLI ARCHITECTURE SPECIFICATION
## Global TV Discovery System - Unified Rust CLI

**Version:** 1.0
**Date:** 2025-12-05
**Status:** Design Specification

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [CLI Core Architecture](#cli-core-architecture)
3. [Rust TUI Framework](#rust-tui-framework)
4. [Streaming Results Display](#streaming-results-display)
5. [gRPC Client Architecture](#grpc-client-architecture)
6. [Authentication Flow](#authentication-flow)
7. [Configuration Management](#configuration-management)
8. [Rust Implementation Details](#rust-implementation-details)
9. [Cross-Platform Considerations](#cross-platform-considerations)
10. [Code Examples](#code-examples)

---

## Executive Summary

The TV Discovery CLI (`tv-discover` or `tvd`) is a unified, cross-platform command-line interface built entirely in Rust. It provides both scriptable commands and an interactive TUI for discovering, browsing, and managing TV content across multiple streaming platforms.

### Key Features
- Cross-platform support (macOS, Linux, Windows)
- Interactive TUI using ratatui
- Scriptable commands for automation
- Real-time streaming results from backend agents
- Device Authorization Grant flow
- Secure credential management
- gRPC communication with backend services

---

## CLI Core Architecture

### Micro-Repository Structure

```
cli-core/
├── Cargo.toml
├── Cargo.lock
├── README.md
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── release.yml
├── src/
│   ├── main.rs                    # Entry point
│   ├── lib.rs                     # Library exports
│   ├── cli/
│   │   ├── mod.rs                 # CLI module root
│   │   ├── app.rs                 # Clap application definition
│   │   ├── commands/
│   │   │   ├── mod.rs
│   │   │   ├── search.rs          # Search commands
│   │   │   ├── browse.rs          # Browse commands
│   │   │   ├── recommend.rs       # Recommendation commands
│   │   │   ├── watch.rs           # Watch commands
│   │   │   ├── devices.rs         # Device management
│   │   │   ├── accounts.rs        # Account management
│   │   │   ├── preferences.rs     # Preferences management
│   │   │   └── config.rs          # Configuration commands
│   │   └── validators.rs          # Input validation
│   ├── tui/
│   │   ├── mod.rs                 # TUI module root
│   │   ├── app.rs                 # TUI application state
│   │   ├── ui/
│   │   │   ├── mod.rs
│   │   │   ├── layout.rs          # Layout definitions
│   │   │   ├── search.rs          # Search UI
│   │   │   ├── results.rs         # Results list UI
│   │   │   ├── detail.rs          # Detail view UI
│   │   │   ├── devices.rs         # Device selector UI
│   │   │   └── widgets/
│   │   │       ├── mod.rs
│   │   │       ├── input.rs       # Input widget
│   │   │       ├── list.rs        # List widget
│   │   │       ├── progress.rs    # Progress indicators
│   │   │       └── spinner.rs     # Spinner/thinking indicator
│   │   ├── events.rs              # Event handling
│   │   └── state.rs               # State management
│   ├── grpc/
│   │   ├── mod.rs                 # gRPC module root
│   │   ├── client.rs              # gRPC client wrapper
│   │   ├── streaming.rs           # Streaming handlers
│   │   ├── retry.rs               # Retry logic
│   │   └── proto/
│   │       ├── mod.rs
│   │       └── generated/         # Generated from .proto files
│   ├── auth/
│   │   ├── mod.rs                 # Auth module root
│   │   ├── device_flow.rs         # Device Authorization Grant
│   │   ├── token_manager.rs       # Token management
│   │   └── keyring.rs             # Secure storage
│   ├── config/
│   │   ├── mod.rs                 # Config module root
│   │   ├── file.rs                # Config file handling
│   │   ├── env.rs                 # Environment variables
│   │   ├── profiles.rs            # Profile management
│   │   └── schema.rs              # Config schema
│   ├── models/
│   │   ├── mod.rs                 # Data models
│   │   ├── content.rs             # Content models
│   │   ├── device.rs              # Device models
│   │   └── user.rs                # User models
│   ├── error.rs                   # Error types
│   └── utils/
│       ├── mod.rs
│       ├── logger.rs              # Logging setup
│       └── terminal.rs            # Terminal utilities
├── proto/
│   ├── discovery.proto            # Service definitions
│   ├── auth.proto
│   └── streaming.proto
├── tests/
│   ├── integration/
│   │   ├── mod.rs
│   │   ├── search_test.rs
│   │   └── auth_test.rs
│   └── fixtures/
└── benches/
    └── cli_benchmark.rs
```

### Command Structure Hierarchy

```
tv-discover (tvd)
├── search                          # Search for content
│   ├── query <QUERY>              # Free-text search
│   ├── filter [OPTIONS]           # Filtered search
│   └── similar <CONTENT_ID>       # Find similar content
├── browse                          # Browse content
│   ├── trending [CATEGORY]        # Trending content
│   ├── new [DAYS]                 # New releases
│   ├── expiring [DAYS]            # Expiring soon
│   └── categories                 # Browse by category
├── recommend                       # Get recommendations
│   ├── for-me                     # Personalized recommendations
│   ├── mood <MOOD>                # Mood-based recommendations
│   └── social                     # Social recommendations
├── watch                           # Watch management
│   ├── now <CONTENT_ID>           # Watch now
│   ├── queue                      # View watch queue
│   │   ├── add <CONTENT_ID>       # Add to queue
│   │   ├── remove <CONTENT_ID>    # Remove from queue
│   │   └── clear                  # Clear queue
│   └── list                       # View watch list
│       ├── add <CONTENT_ID>       # Add to list
│       └── remove <CONTENT_ID>    # Remove from list
├── devices                         # Device management
│   ├── list                       # List devices
│   ├── connect <DEVICE_ID>        # Connect to device
│   ├── sync                       # Sync devices
│   └── cast <CONTENT_ID>          # Cast to device
├── accounts                        # Account management
│   ├── list                       # List linked accounts
│   ├── link <PROVIDER>            # Link account
│   ├── unlink <PROVIDER>          # Unlink account
│   └── status                     # Show account status
├── preferences                     # User preferences
│   ├── genres                     # Manage genre preferences
│   │   ├── add <GENRE>            # Add preferred genre
│   │   ├── remove <GENRE>         # Remove genre
│   │   └── list                   # List genres
│   ├── platforms                  # Manage platform preferences
│   │   ├── add <PLATFORM>         # Add platform
│   │   ├── remove <PLATFORM>      # Remove platform
│   │   └── list                   # List platforms
│   ├── region <REGION>            # Set region
│   └── export                     # Export preferences
└── config                          # Configuration
    ├── init                       # Initialize config
    ├── set <KEY> <VALUE>          # Set config value
    ├── show [KEY]                 # Show config
    └── profile <NAME>             # Switch profile
```

---

## Rust TUI Framework

### Component Architecture

The TUI is built using **ratatui** with a **crossterm** backend, following a component-based architecture with unidirectional data flow.

### Core Components

#### 1. Search Input Component
```
┌─ Search ───────────────────────────────────────────────────────┐
│ > thriller sci-fi available:netflix                            │
│   Filters: [Genre: Thriller, Sci-Fi] [Platform: Netflix]      │
└────────────────────────────────────────────────────────────────┘
```

Features:
- Autocomplete for filters
- Syntax highlighting
- History navigation (↑/↓)
- Real-time validation

#### 2. Results List Component
```
┌─ Results (42 found) ───────────────────────────────────────────┐
│ ▸ Stranger Things (TV Series)                     Netflix ★8.7 │
│   Black Mirror (TV Series)                         Netflix ★8.8 │
│   The Expanse (TV Series)                    Prime Video ★8.5 │
│   Dark (TV Series)                                 Netflix ★8.8 │
│   Westworld (TV Series)                                HBO ★8.5 │
│                                                                 │
│ [↑/↓: Navigate] [Enter: Details] [Space: Add to Queue]        │
└────────────────────────────────────────────────────────────────┘
```

Features:
- Virtualized scrolling for large lists
- Multi-selection support
- Sort indicators
- Platform badges
- Rating display

#### 3. Detail View Component
```
┌─ Stranger Things ──────────────────────────────────────────────┐
│ [Netflix] ★ 8.7/10 (IMDb)                        TV-14 | 2016- │
│                                                                 │
│ Genres: Sci-Fi, Horror, Drama, Mystery                         │
│ Seasons: 4 | Episodes: 34 | Runtime: ~50min                   │
│                                                                 │
│ When a young boy disappears, his mother, a police chief and   │
│ his friends must confront terrifying supernatural forces in    │
│ order to get him back.                                         │
│                                                                 │
│ Cast: Millie Bobby Brown, Finn Wolfhard, Winona Ryder         │
│                                                                 │
│ ┌─ Actions ───────────────────────────────────────────────────┐│
│ │ [W] Watch Now  [Q] Add to Queue  [L] Add to List           ││
│ │ [S] Similar    [C] Cast to Device                          ││
│ └─────────────────────────────────────────────────────────────┘│
└────────────────────────────────────────────────────────────────┘
```

#### 4. Device Selector Component
```
┌─ Cast to Device ───────────────────────────────────────────────┐
│ ▸ Living Room TV (Roku Ultra)                      [Connected] │
│   Bedroom TV (Apple TV 4K)                            [Idle]   │
│   Kitchen Display (Chromecast)                        [Idle]   │
│                                                                 │
│ [Enter: Select] [Esc: Cancel]                                  │
└────────────────────────────────────────────────────────────────┘
```

#### 5. Progress Indicators

**Streaming Results:**
```
┌─ Searching... ─────────────────────────────────────────────────┐
│ ⠋ Netflix: Searching...                                  [12%] │
│ ✓ Prime Video: 8 results                                [100%] │
│ ⠙ HBO Max: Searching...                                  [45%] │
│ ✓ Hulu: 5 results                                       [100%] │
│ ⠸ Disney+: Searching...                                  [67%] │
└────────────────────────────────────────────────────────────────┘
```

**Thinking Indicator:**
```
⠋ Processing recommendations...
⠙ Analyzing viewing history...
⠹ Generating mood-based suggestions...
⠸ Fetching social recommendations...
```

### Keyboard Navigation

```
Global:
  Ctrl+C, q          - Quit
  Ctrl+L             - Redraw screen
  /                  - Focus search
  Esc                - Back/Cancel
  Tab                - Next component
  Shift+Tab          - Previous component

Search:
  Enter              - Submit search
  ↑/↓                - History navigation
  Ctrl+U             - Clear input

Lists:
  ↑/↓, j/k           - Navigate items
  PageUp/PageDown    - Page navigation
  Home/End           - First/Last item
  Enter              - Select/Open
  Space              - Toggle selection

Detail View:
  w                  - Watch now
  q                  - Add to queue
  l                  - Add to list
  s                  - Find similar
  c                  - Cast to device
```

### Mouse Support

- Click to focus components
- Click to select items
- Scroll wheel for lists
- Click buttons for actions

---

## Streaming Results Display

### Real-Time Result Streaming Architecture

The CLI receives streaming results from backend agents and progressively renders them as they arrive.

### Streaming Flow

```
┌──────────┐         ┌──────────┐         ┌──────────┐
│   CLI    │ gRPC    │ Gateway  │ Agent   │  Agent   │
│  Client  │◄────────┤  Server  │◄────────┤ (Netflix)│
└──────────┘ Stream  └──────────┘         └──────────┘
     │                                           │
     │ StreamSearchRequest                       │
     ├──────────────────────────────────────────►│
     │                                           │
     │ SearchResultChunk (1)                     │
     │◄──────────────────────────────────────────┤
     │ [Display partial results]                 │
     │                                           │
     │ SearchResultChunk (2)                     │
     │◄──────────────────────────────────────────┤
     │ [Update display]                          │
     │                                           │
     │ SearchResultChunk (final)                 │
     │◄──────────────────────────────────────────┤
     │ [Complete display]                        │
```

### Progressive Rendering States

1. **Initial State** (No results yet)
```
┌─ Searching for "stranger things"... ──────────────────────────┐
│                                                                 │
│   ⠋ Initializing search across platforms...                   │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

2. **First Results Arrive**
```
┌─ Searching for "stranger things"... ──────────────────────────┐
│ ▸ Stranger Things (TV Series)                     Netflix ★8.7 │
│   Stranger Things: Behind the Scenes              Netflix ★7.2 │
│                                                                 │
│   ⠙ Netflix: 2 results | Prime: Searching... | Hulu: Waiting  │
└────────────────────────────────────────────────────────────────┘
```

3. **Multiple Agents Returning**
```
┌─ Searching for "stranger things"... ──────────────────────────┐
│ ▸ Stranger Things (TV Series)                     Netflix ★8.7 │
│   Stranger Things: Behind the Scenes              Netflix ★7.2 │
│   The Stranger (Film)                        Prime Video ★6.8 │
│   Strange Angel (TV Series)                        Hulu ★7.1 │
│                                                                 │
│   ✓ Netflix: 2 | ✓ Prime: 1 | ✓ Hulu: 1 | ⠸ HBO: Searching...│
└────────────────────────────────────────────────────────────────┘
```

4. **Complete**
```
┌─ Results (12 found in 2.3s) ───────────────────────────────────┐
│ ▸ Stranger Things (TV Series)                     Netflix ★8.7 │
│   Stranger Things: Behind the Scenes              Netflix ★7.2 │
│   The Stranger (Film)                        Prime Video ★6.8 │
│   Strange Angel (TV Series)                        Hulu ★7.1 │
│   ...                                                           │
│                                                                 │
│   ✓ All platforms searched | Sort: Relevance                   │
└────────────────────────────────────────────────────────────────┘
```

### Thinking Indicators

Custom spinner characters for different states:
```rust
const SPINNER_FRAMES: &[&str] = &["⠋", "⠙", "⠹", "⠸", "⠼", "⠴", "⠦", "⠧", "⠇", "⠏"];
const THINKING_MESSAGES: &[&str] = &[
    "Searching across platforms...",
    "Analyzing content metadata...",
    "Filtering results...",
    "Ranking by relevance...",
];
```

### Partial Result Display Strategy

- **Minimum Display**: Show first result after 200ms (perceived responsiveness)
- **Batch Updates**: Update UI every 100ms or every 5 results (whichever comes first)
- **Smooth Scrolling**: Auto-scroll to new results only if user is at bottom
- **Stable Positions**: Lock item positions once displayed (no jumping)
- **Background Loading**: Continue loading while user browses early results

---

## gRPC Client Architecture

### Service Definitions

#### discovery.proto
```protobuf
syntax = "proto3";

package tv_discovery.v1;

service DiscoveryService {
  // Search operations
  rpc Search(SearchRequest) returns (stream SearchResult);
  rpc GetContentDetails(ContentDetailsRequest) returns (ContentDetails);

  // Browse operations
  rpc BrowseTrending(BrowseRequest) returns (stream BrowseResult);
  rpc BrowseNew(BrowseRequest) returns (stream BrowseResult);
  rpc BrowseExpiring(BrowseRequest) returns (stream BrowseResult);

  // Recommendations
  rpc GetRecommendations(RecommendationRequest) returns (stream Recommendation);
  rpc GetSimilar(SimilarRequest) returns (stream SimilarContent);

  // Watch management
  rpc GetWatchQueue(WatchQueueRequest) returns (WatchQueue);
  rpc AddToQueue(AddToQueueRequest) returns (AddToQueueResponse);
  rpc RemoveFromQueue(RemoveFromQueueRequest) returns (RemoveFromQueueResponse);

  // Device operations
  rpc ListDevices(ListDevicesRequest) returns (ListDevicesResponse);
  rpc ConnectDevice(ConnectDeviceRequest) returns (ConnectDeviceResponse);
  rpc CastToDevice(CastRequest) returns (CastResponse);
}

message SearchRequest {
  string query = 1;
  repeated string filters = 2;
  SearchOptions options = 3;
}

message SearchResult {
  oneof result {
    SearchChunk chunk = 1;
    SearchComplete complete = 2;
    SearchError error = 3;
  }
}

message SearchChunk {
  string agent_id = 1;
  repeated ContentItem items = 2;
  int32 total_found = 3;
  float progress = 4;
}

message SearchComplete {
  int32 total_results = 1;
  float duration_seconds = 2;
}

message ContentItem {
  string id = 1;
  string title = 2;
  ContentType type = 3;
  repeated string genres = 4;
  float rating = 5;
  string platform = 6;
  string thumbnail_url = 7;
  int32 year = 8;
}

enum ContentType {
  CONTENT_TYPE_UNSPECIFIED = 0;
  MOVIE = 1;
  TV_SERIES = 2;
  DOCUMENTARY = 3;
  SPECIAL = 4;
}

message SearchOptions {
  int32 max_results = 1;
  SortOrder sort_order = 2;
  repeated string platforms = 3;
}

enum SortOrder {
  SORT_ORDER_UNSPECIFIED = 0;
  RELEVANCE = 1;
  RATING = 2;
  RELEASE_DATE = 3;
  TITLE = 4;
}
```

#### auth.proto
```protobuf
syntax = "proto3";

package tv_discovery.auth.v1;

service AuthService {
  // Device Authorization Grant flow
  rpc InitiateDeviceAuth(DeviceAuthRequest) returns (DeviceAuthResponse);
  rpc PollDeviceAuth(PollRequest) returns (PollResponse);
  rpc RefreshToken(RefreshTokenRequest) returns (RefreshTokenResponse);
  rpc RevokeToken(RevokeTokenRequest) returns (RevokeTokenResponse);
}

message DeviceAuthRequest {
  string client_id = 1;
  repeated string scopes = 2;
}

message DeviceAuthResponse {
  string device_code = 1;
  string user_code = 2;
  string verification_uri = 3;
  string verification_uri_complete = 4;
  int32 expires_in = 5;
  int32 interval = 6;
}

message PollRequest {
  string device_code = 1;
}

message PollResponse {
  oneof result {
    TokenResponse token = 1;
    PollStatus status = 2;
  }
}

message PollStatus {
  enum Status {
    STATUS_UNSPECIFIED = 0;
    AUTHORIZATION_PENDING = 1;
    SLOW_DOWN = 2;
    EXPIRED = 3;
    ACCESS_DENIED = 4;
  }
  Status status = 1;
  string message = 2;
}

message TokenResponse {
  string access_token = 1;
  string refresh_token = 2;
  int64 expires_at = 3;
  string token_type = 4;
}
```

### gRPC Client Implementation

```rust
// src/grpc/client.rs

use tonic::{
    transport::{Channel, ClientTlsConfig, Endpoint},
    metadata::MetadataValue,
    Request, Status,
};
use tower::ServiceBuilder;
use std::time::Duration;

pub struct GrpcClient {
    channel: Channel,
    discovery_client: DiscoveryServiceClient<Channel>,
    auth_client: AuthServiceClient<Channel>,
}

impl GrpcClient {
    pub async fn new(endpoint: &str) -> Result<Self, Box<dyn std::error::Error>> {
        let channel = Endpoint::from_shared(endpoint.to_string())?
            .timeout(Duration::from_secs(30))
            .connect_timeout(Duration::from_secs(10))
            .tls_config(ClientTlsConfig::new())?
            .connect()
            .await?;

        let discovery_client = DiscoveryServiceClient::new(channel.clone());
        let auth_client = AuthServiceClient::new(channel.clone());

        Ok(Self {
            channel,
            discovery_client,
            auth_client,
        })
    }

    pub async fn search(
        &mut self,
        query: String,
        filters: Vec<String>,
    ) -> Result<impl Stream<Item = Result<SearchResult, Status>>, Status> {
        let request = Request::new(SearchRequest {
            query,
            filters,
            options: Some(SearchOptions::default()),
        });

        let response = self.discovery_client.search(request).await?;
        Ok(response.into_inner())
    }

    pub fn with_auth_token(mut self, token: &str) -> Self {
        let token_value = MetadataValue::try_from(format!("Bearer {}", token))
            .expect("Invalid token format");

        // Add interceptor for auth
        self
    }
}
```

### Streaming RPC Handler

```rust
// src/grpc/streaming.rs

use futures_util::StreamExt;
use tokio::sync::mpsc;

pub struct StreamingHandler {
    tx: mpsc::Sender<SearchResultChunk>,
}

impl StreamingHandler {
    pub fn new() -> (Self, mpsc::Receiver<SearchResultChunk>) {
        let (tx, rx) = mpsc::channel(100);
        (Self { tx }, rx)
    }

    pub async fn handle_stream(
        &self,
        mut stream: impl Stream<Item = Result<SearchResult, Status>> + Unpin,
    ) -> Result<(), Box<dyn std::error::Error>> {
        while let Some(result) = stream.next().await {
            match result {
                Ok(search_result) => {
                    match search_result.result {
                        Some(Result::Chunk(chunk)) => {
                            self.tx.send(chunk).await?;
                        }
                        Some(Result::Complete(complete)) => {
                            // Handle completion
                            break;
                        }
                        Some(Result::Error(error)) => {
                            // Handle error
                            return Err(error.into());
                        }
                        None => {}
                    }
                }
                Err(status) => {
                    return Err(status.into());
                }
            }
        }
        Ok(())
    }
}
```

### Connection Pooling and Retry

```rust
// src/grpc/retry.rs

use tower::{retry::Policy, ServiceBuilder};
use std::time::Duration;

#[derive(Clone)]
pub struct RetryPolicy {
    max_retries: usize,
    backoff: Duration,
}

impl RetryPolicy {
    pub fn new(max_retries: usize) -> Self {
        Self {
            max_retries,
            backoff: Duration::from_millis(100),
        }
    }
}

impl<Req: Clone, Res, E> Policy<Req, Res, E> for RetryPolicy {
    type Future = futures::future::Ready<Self>;

    fn retry(&self, _req: &Req, result: Result<&Res, &E>) -> Option<Self::Future> {
        match result {
            Ok(_) => None,
            Err(_) => {
                if self.max_retries > 0 {
                    Some(futures::future::ready(RetryPolicy {
                        max_retries: self.max_retries - 1,
                        backoff: self.backoff * 2,
                    }))
                } else {
                    None
                }
            }
        }
    }

    fn clone_request(&self, req: &Req) -> Option<Req> {
        Some(req.clone())
    }
}

// Connection pool configuration
pub fn create_channel_pool(
    endpoints: Vec<String>,
) -> Result<Channel, Box<dyn std::error::Error>> {
    // Implement connection pooling with load balancing
    // Use tower's balance layer for load balancing across endpoints
    unimplemented!("Connection pooling implementation")
}
```

---

## Authentication Flow

### Device Authorization Grant Implementation

The CLI implements OAuth 2.0 Device Authorization Grant (RFC 8628) for secure authentication.

### Flow Diagram

```
┌─────────┐                                  ┌──────────┐                 ┌─────────┐
│   CLI   │                                  │   Auth   │                 │ Browser │
│ Client  │                                  │  Server  │                 │  User   │
└────┬────┘                                  └────┬─────┘                 └────┬────┘
     │                                            │                            │
     │ 1. Initiate Device Auth                    │                            │
     ├───────────────────────────────────────────►│                            │
     │                                            │                            │
     │ 2. Device Code + User Code                 │                            │
     │◄───────────────────────────────────────────┤                            │
     │                                            │                            │
     │ 3. Display User Code & URL                 │                            │
     │    "Visit https://auth.tv/device"          │                            │
     │    "Enter code: WDJB-MJHT"                 │                            │
     │                                            │                            │
     │                                            │ 4. User visits URL         │
     │                                            │◄───────────────────────────┤
     │                                            │                            │
     │                                            │ 5. Enter code              │
     │                                            │◄───────────────────────────┤
     │                                            │                            │
     │ 6. Poll for authorization (every 5s)       │ 7. User authorizes         │
     ├───────────────────────────────────────────►│◄───────────────────────────┤
     │      (pending...)                          │                            │
     ├───────────────────────────────────────────►│                            │
     │      (pending...)                          │                            │
     ├───────────────────────────────────────────►│                            │
     │                                            │                            │
     │ 8. Access Token + Refresh Token            │                            │
     │◄───────────────────────────────────────────┤                            │
     │                                            │                            │
     │ 9. Store in keyring                        │                            │
     │                                            │                            │
```

### CLI Display During Auth

```
┌─ Authentication Required ──────────────────────────────────────┐
│                                                                 │
│   To authenticate, please visit:                               │
│                                                                 │
│   https://auth.tv-discover.com/device                          │
│                                                                 │
│   And enter the code:                                          │
│                                                                 │
│        ╔════════════╗                                          │
│        ║  WDJB-MJHT ║                                          │
│        ╚════════════╝                                          │
│                                                                 │
│   Or scan this QR code:                                        │
│   ┌─────────────────┐                                          │
│   │ █▀▀▀▀▀█ █▀ █▀▀ │                                          │
│   │ █ ███ █ ▀█ █▀█ │                                          │
│   │ █ ▀▀▀ █ ▀▀ ▀▀█ │                                          │
│   │ ▀▀▀▀▀▀▀ █ █ ▀ █ │                                          │
│   └─────────────────┘                                          │
│                                                                 │
│   ⠋ Waiting for authorization...                              │
│                                                                 │
│   Code expires in: 14:32                                       │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

### Implementation

```rust
// src/auth/device_flow.rs

use crate::grpc::auth_client::AuthServiceClient;
use std::time::{Duration, SystemTime};
use tokio::time;

pub struct DeviceAuthFlow {
    client: AuthServiceClient,
}

impl DeviceAuthFlow {
    pub fn new(client: AuthServiceClient) -> Self {
        Self { client }
    }

    pub async fn authenticate(&mut self) -> Result<TokenResponse, AuthError> {
        // Step 1: Initiate device authorization
        let device_auth = self.client
            .initiate_device_auth(DeviceAuthRequest {
                client_id: CLIENT_ID.to_string(),
                scopes: vec!["read".to_string(), "write".to_string()],
            })
            .await?
            .into_inner();

        // Step 2: Display user code and verification URL
        self.display_auth_prompt(&device_auth)?;

        // Step 3: Poll for authorization
        let token = self.poll_for_authorization(
            device_auth.device_code,
            Duration::from_secs(device_auth.interval as u64),
            Duration::from_secs(device_auth.expires_in as u64),
        ).await?;

        Ok(token)
    }

    fn display_auth_prompt(&self, auth: &DeviceAuthResponse) -> Result<(), AuthError> {
        println!("\n┌─ Authentication Required ────────────────────────────┐");
        println!("│                                                       │");
        println!("│   To authenticate, please visit:                     │");
        println!("│                                                       │");
        println!("│   {}                       │", auth.verification_uri);
        println!("│                                                       │");
        println!("│   And enter the code:                                │");
        println!("│                                                       │");
        println!("│        ╔════════════╗                                │");
        println!("│        ║  {}  ║                                │", auth.user_code);
        println!("│        ╚════════════╝                                │");
        println!("│                                                       │");

        if let Some(uri_complete) = &auth.verification_uri_complete {
            self.display_qr_code(uri_complete)?;
        }

        println!("│                                                       │");
        println!("│   Code expires in: {}:{}                          │",
            auth.expires_in / 60, auth.expires_in % 60);
        println!("│                                                       │");
        println!("└───────────────────────────────────────────────────────┘\n");

        Ok(())
    }

    async fn poll_for_authorization(
        &mut self,
        device_code: String,
        interval: Duration,
        expires_in: Duration,
    ) -> Result<TokenResponse, AuthError> {
        let start_time = SystemTime::now();
        let mut poll_interval = time::interval(interval);

        loop {
            poll_interval.tick().await;

            // Check if expired
            if start_time.elapsed()? >= expires_in {
                return Err(AuthError::DeviceCodeExpired);
            }

            let response = self.client
                .poll_device_auth(PollRequest {
                    device_code: device_code.clone(),
                })
                .await?
                .into_inner();

            match response.result {
                Some(poll_response::Result::Token(token)) => {
                    println!("\n✓ Authentication successful!\n");
                    return Ok(token);
                }
                Some(poll_response::Result::Status(status)) => {
                    match status.status() {
                        PollStatus::AuthorizationPending => {
                            // Continue polling
                            self.update_waiting_indicator();
                        }
                        PollStatus::SlowDown => {
                            // Increase interval
                            poll_interval = time::interval(interval * 2);
                        }
                        PollStatus::AccessDenied => {
                            return Err(AuthError::AccessDenied);
                        }
                        PollStatus::Expired => {
                            return Err(AuthError::DeviceCodeExpired);
                        }
                        _ => {}
                    }
                }
                None => {}
            }
        }
    }

    fn update_waiting_indicator(&self) {
        // Display spinner animation
        print!("\r⠋ Waiting for authorization...");
        std::io::stdout().flush().unwrap();
    }
}
```

### Token Management

```rust
// src/auth/token_manager.rs

use serde::{Deserialize, Serialize};
use std::time::{SystemTime, UNIX_EPOCH};

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct TokenStore {
    pub access_token: String,
    pub refresh_token: String,
    pub expires_at: u64,
    pub token_type: String,
}

impl TokenStore {
    pub fn from_response(response: TokenResponse) -> Self {
        Self {
            access_token: response.access_token,
            refresh_token: response.refresh_token,
            expires_at: response.expires_at,
            token_type: response.token_type,
        }
    }

    pub fn is_expired(&self) -> bool {
        let now = SystemTime::now()
            .duration_since(UNIX_EPOCH)
            .unwrap()
            .as_secs();

        // Add 60 second buffer
        now >= (self.expires_at - 60)
    }

    pub fn needs_refresh(&self) -> bool {
        self.is_expired()
    }
}

pub struct TokenManager {
    keyring: Keyring,
    auth_client: AuthServiceClient,
}

impl TokenManager {
    pub fn new(keyring: Keyring, auth_client: AuthServiceClient) -> Self {
        Self { keyring, auth_client }
    }

    pub async fn get_valid_token(&mut self) -> Result<String, AuthError> {
        let mut token_store = self.keyring.get_token()?;

        if token_store.needs_refresh() {
            token_store = self.refresh_token(token_store).await?;
            self.keyring.store_token(&token_store)?;
        }

        Ok(token_store.access_token)
    }

    async fn refresh_token(&mut self, token_store: TokenStore) -> Result<TokenStore, AuthError> {
        let response = self.auth_client
            .refresh_token(RefreshTokenRequest {
                refresh_token: token_store.refresh_token,
            })
            .await?
            .into_inner();

        Ok(TokenStore::from_response(response))
    }
}
```

### Secure Token Storage (Keyring)

```rust
// src/auth/keyring.rs

use keyring::Entry;
use serde_json;

const SERVICE_NAME: &str = "tv-discover-cli";
const TOKEN_KEY: &str = "auth_token";

pub struct Keyring {
    entry: Entry,
}

impl Keyring {
    pub fn new() -> Result<Self, KeyringError> {
        let entry = Entry::new(SERVICE_NAME, TOKEN_KEY)?;
        Ok(Self { entry })
    }

    pub fn store_token(&self, token: &TokenStore) -> Result<(), KeyringError> {
        let json = serde_json::to_string(token)?;
        self.entry.set_password(&json)?;
        Ok(())
    }

    pub fn get_token(&self) -> Result<TokenStore, KeyringError> {
        let json = self.entry.get_password()?;
        let token = serde_json::from_str(&json)?;
        Ok(token)
    }

    pub fn delete_token(&self) -> Result<(), KeyringError> {
        self.entry.delete_password()?;
        Ok(())
    }

    pub fn has_token(&self) -> bool {
        self.entry.get_password().is_ok()
    }
}
```

---

## Configuration Management

### Configuration File Structure

**Location:** `~/.config/tv-discover/config.toml`

```toml
# TV Discover CLI Configuration

[auth]
# Authentication endpoint
endpoint = "https://auth.tv-discover.com"
client_id = "cli-client"

[api]
# API Gateway endpoint
endpoint = "https://api.tv-discover.com:50051"
# Connection timeout in seconds
timeout = 30
# Enable TLS
use_tls = true

[search]
# Default maximum results
max_results = 50
# Default sort order: relevance, rating, release_date, title
default_sort = "relevance"

[display]
# Enable colors
colors = true
# Enable mouse support
mouse = true
# Results per page
page_size = 20
# Theme: dark, light, auto
theme = "auto"

[platforms]
# Preferred platforms (searched first)
preferred = ["Netflix", "Prime Video", "HBO Max"]
# Excluded platforms
excluded = []

[preferences]
# Preferred genres
genres = ["Sci-Fi", "Thriller", "Documentary"]
# Preferred content languages
languages = ["en", "es"]
# Content region
region = "US"

[devices]
# Default device for casting
default_device_id = ""

[cache]
# Enable result caching
enabled = true
# Cache TTL in seconds
ttl = 3600
# Cache directory
directory = "~/.cache/tv-discover"

[logging]
# Log level: trace, debug, info, warn, error
level = "info"
# Log file location
file = "~/.local/share/tv-discover/logs/cli.log"

# Profile support
[profiles.work]
platforms.preferred = ["Netflix", "YouTube"]
display.theme = "light"

[profiles.home]
platforms.preferred = ["Netflix", "Prime Video", "HBO Max", "Disney+"]
devices.default_device_id = "living-room-tv"
```

### Configuration Schema

```rust
// src/config/schema.rs

use serde::{Deserialize, Serialize};
use std::collections::HashMap;

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Config {
    pub auth: AuthConfig,
    pub api: ApiConfig,
    pub search: SearchConfig,
    pub display: DisplayConfig,
    pub platforms: PlatformsConfig,
    pub preferences: PreferencesConfig,
    pub devices: DevicesConfig,
    pub cache: CacheConfig,
    pub logging: LoggingConfig,
    #[serde(default)]
    pub profiles: HashMap<String, ProfileConfig>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct AuthConfig {
    pub endpoint: String,
    pub client_id: String,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct ApiConfig {
    pub endpoint: String,
    #[serde(default = "default_timeout")]
    pub timeout: u64,
    #[serde(default = "default_true")]
    pub use_tls: bool,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct SearchConfig {
    #[serde(default = "default_max_results")]
    pub max_results: usize,
    #[serde(default)]
    pub default_sort: SortOrder,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct DisplayConfig {
    #[serde(default = "default_true")]
    pub colors: bool,
    #[serde(default = "default_true")]
    pub mouse: bool,
    #[serde(default = "default_page_size")]
    pub page_size: usize,
    #[serde(default)]
    pub theme: Theme,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct PlatformsConfig {
    #[serde(default)]
    pub preferred: Vec<String>,
    #[serde(default)]
    pub excluded: Vec<String>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct PreferencesConfig {
    #[serde(default)]
    pub genres: Vec<String>,
    #[serde(default)]
    pub languages: Vec<String>,
    pub region: String,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct DevicesConfig {
    #[serde(default)]
    pub default_device_id: String,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct CacheConfig {
    #[serde(default = "default_true")]
    pub enabled: bool,
    #[serde(default = "default_cache_ttl")]
    pub ttl: u64,
    pub directory: String,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct LoggingConfig {
    #[serde(default)]
    pub level: LogLevel,
    pub file: String,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct ProfileConfig {
    #[serde(flatten)]
    pub overrides: HashMap<String, toml::Value>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
#[serde(rename_all = "snake_case")]
pub enum SortOrder {
    Relevance,
    Rating,
    ReleaseDate,
    Title,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
#[serde(rename_all = "lowercase")]
pub enum Theme {
    Dark,
    Light,
    Auto,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
#[serde(rename_all = "lowercase")]
pub enum LogLevel {
    Trace,
    Debug,
    Info,
    Warn,
    Error,
}

// Default value functions
fn default_timeout() -> u64 { 30 }
fn default_true() -> bool { true }
fn default_max_results() -> usize { 50 }
fn default_page_size() -> usize { 20 }
fn default_cache_ttl() -> u64 { 3600 }

impl Default for Config {
    fn default() -> Self {
        Self {
            auth: AuthConfig {
                endpoint: "https://auth.tv-discover.com".to_string(),
                client_id: "cli-client".to_string(),
            },
            api: ApiConfig {
                endpoint: "https://api.tv-discover.com:50051".to_string(),
                timeout: 30,
                use_tls: true,
            },
            search: SearchConfig {
                max_results: 50,
                default_sort: SortOrder::Relevance,
            },
            display: DisplayConfig {
                colors: true,
                mouse: true,
                page_size: 20,
                theme: Theme::Auto,
            },
            platforms: PlatformsConfig {
                preferred: vec![
                    "Netflix".to_string(),
                    "Prime Video".to_string(),
                    "HBO Max".to_string(),
                ],
                excluded: vec![],
            },
            preferences: PreferencesConfig {
                genres: vec![],
                languages: vec!["en".to_string()],
                region: "US".to_string(),
            },
            devices: DevicesConfig {
                default_device_id: String::new(),
            },
            cache: CacheConfig {
                enabled: true,
                ttl: 3600,
                directory: "~/.cache/tv-discover".to_string(),
            },
            logging: LoggingConfig {
                level: LogLevel::Info,
                file: "~/.local/share/tv-discover/logs/cli.log".to_string(),
            },
            profiles: HashMap::new(),
        }
    }
}
```

### Configuration File Handling

```rust
// src/config/file.rs

use std::fs;
use std::path::PathBuf;
use dirs::config_dir;

pub struct ConfigManager {
    config_path: PathBuf,
    config: Config,
}

impl ConfigManager {
    pub fn new() -> Result<Self, ConfigError> {
        let config_path = Self::get_config_path()?;
        let config = Self::load_or_create(&config_path)?;

        Ok(Self { config_path, config })
    }

    pub fn with_profile(profile_name: &str) -> Result<Self, ConfigError> {
        let mut manager = Self::new()?;
        manager.apply_profile(profile_name)?;
        Ok(manager)
    }

    fn get_config_path() -> Result<PathBuf, ConfigError> {
        let config_dir = config_dir()
            .ok_or(ConfigError::NoConfigDir)?
            .join("tv-discover");

        fs::create_dir_all(&config_dir)?;
        Ok(config_dir.join("config.toml"))
    }

    fn load_or_create(path: &PathBuf) -> Result<Config, ConfigError> {
        if path.exists() {
            let contents = fs::read_to_string(path)?;
            let config: Config = toml::from_str(&contents)?;
            Ok(config)
        } else {
            let config = Config::default();
            let toml_string = toml::to_string_pretty(&config)?;
            fs::write(path, toml_string)?;
            Ok(config)
        }
    }

    pub fn get(&self) -> &Config {
        &self.config
    }

    pub fn set<T: Serialize>(&mut self, key: &str, value: T) -> Result<(), ConfigError> {
        // Update config using key path (e.g., "display.theme")
        // This would use a library like `config-rs` for nested key access
        self.save()?;
        Ok(())
    }

    pub fn save(&self) -> Result<(), ConfigError> {
        let toml_string = toml::to_string_pretty(&self.config)?;
        fs::write(&self.config_path, toml_string)?;
        Ok(())
    }

    fn apply_profile(&mut self, profile_name: &str) -> Result<(), ConfigError> {
        if let Some(profile) = self.config.profiles.get(profile_name) {
            // Merge profile overrides into main config
            // Implementation would deep merge the overrides
            Ok(())
        } else {
            Err(ConfigError::ProfileNotFound(profile_name.to_string()))
        }
    }
}
```

### Environment Variable Overrides

```rust
// src/config/env.rs

use std::env;

pub struct EnvOverrides;

impl EnvOverrides {
    pub fn apply(config: &mut Config) {
        // API endpoint override
        if let Ok(endpoint) = env::var("TVD_API_ENDPOINT") {
            config.api.endpoint = endpoint;
        }

        // Auth endpoint override
        if let Ok(endpoint) = env::var("TVD_AUTH_ENDPOINT") {
            config.auth.endpoint = endpoint;
        }

        // Log level override
        if let Ok(level) = env::var("TVD_LOG_LEVEL") {
            if let Ok(log_level) = level.parse::<LogLevel>() {
                config.logging.level = log_level;
            }
        }

        // Timeout override
        if let Ok(timeout) = env::var("TVD_TIMEOUT") {
            if let Ok(timeout_secs) = timeout.parse::<u64>() {
                config.api.timeout = timeout_secs;
            }
        }

        // Profile override
        if let Ok(profile) = env::var("TVD_PROFILE") {
            // Would need to load and apply profile
        }
    }
}
```

---

## Rust Implementation Details

### Cargo.toml

```toml
[package]
name = "tv-discover-cli"
version = "0.1.0"
edition = "2021"
authors = ["TV Discover Team"]
description = "Unified CLI for global TV content discovery"
license = "MIT"
repository = "https://github.com/tv-discover/cli-core"

[dependencies]
# CLI Framework
clap = { version = "4.4", features = ["derive", "env", "wrap_help"] }

# TUI Framework
ratatui = "0.25"
crossterm = { version = "0.27", features = ["event-stream"] }

# gRPC
tonic = { version = "0.10", features = ["tls", "tls-roots"] }
prost = "0.12"
tower = { version = "0.4", features = ["retry", "balance"] }

# Async Runtime
tokio = { version = "1.35", features = ["full"] }
tokio-stream = "0.1"
futures = "0.3"
futures-util = "0.3"

# Serialization
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
toml = "0.8"

# Error Handling
thiserror = "1.0"
anyhow = "1.0"

# Authentication
oauth2 = "4.4"
keyring = "2.2"

# Configuration
dirs = "5.0"
config = "0.13"

# Logging
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter", "json"] }

# Utilities
chrono = "0.4"
uuid = { version = "1.6", features = ["v4"] }
url = "2.5"
reqwest = { version = "0.11", features = ["json"] }

# QR Code generation
qrcode = "0.13"

# Terminal utilities
console = "0.15"
indicatif = "0.17"

[build-dependencies]
tonic-build = "0.10"

[dev-dependencies]
mockall = "0.12"
wiremock = "0.5"
tempfile = "3.8"

[profile.release]
opt-level = 3
lto = true
codegen-units = 1
strip = true

[[bin]]
name = "tvd"
path = "src/main.rs"
```

### Main Entry Point

```rust
// src/main.rs

use clap::Parser;
use tv_discover_cli::{
    cli::App,
    config::ConfigManager,
    error::Result,
    utils::logger,
};

#[tokio::main]
async fn main() -> Result<()> {
    // Initialize logging
    logger::init()?;

    // Load configuration
    let config = ConfigManager::new()?;

    // Parse CLI arguments
    let app = App::parse();

    // Execute command
    app.execute(config).await?;

    Ok(())
}
```

### CLI Application Definition

```rust
// src/cli/app.rs

use clap::{Parser, Subcommand};
use crate::cli::commands::*;

#[derive(Parser)]
#[command(name = "tvd")]
#[command(author, version, about, long_about = None)]
#[command(propagate_version = true)]
pub struct App {
    #[command(subcommand)]
    pub command: Commands,

    /// Use interactive TUI mode
    #[arg(short, long, global = true)]
    pub interactive: bool,

    /// Output format (json, yaml, table)
    #[arg(short, long, global = true, default_value = "table")]
    pub format: OutputFormat,

    /// Enable verbose logging
    #[arg(short, long, global = true)]
    pub verbose: bool,

    /// Use specific profile
    #[arg(short, long, global = true, env = "TVD_PROFILE")]
    pub profile: Option<String>,
}

#[derive(Subcommand)]
pub enum Commands {
    /// Search for content
    Search(SearchCommand),

    /// Browse content
    Browse(BrowseCommand),

    /// Get recommendations
    Recommend(RecommendCommand),

    /// Watch management
    Watch(WatchCommand),

    /// Device management
    Devices(DevicesCommand),

    /// Account management
    Accounts(AccountsCommand),

    /// Preferences management
    Preferences(PreferencesCommand),

    /// Configuration management
    Config(ConfigCommand),
}

#[derive(Clone, Debug, ValueEnum)]
pub enum OutputFormat {
    Json,
    Yaml,
    Table,
}

impl App {
    pub async fn execute(self, config: ConfigManager) -> Result<()> {
        if self.interactive {
            // Launch TUI mode
            crate::tui::run(config).await?;
        } else {
            // Execute command in scriptable mode
            match self.command {
                Commands::Search(cmd) => cmd.execute(config).await?,
                Commands::Browse(cmd) => cmd.execute(config).await?,
                Commands::Recommend(cmd) => cmd.execute(config).await?,
                Commands::Watch(cmd) => cmd.execute(config).await?,
                Commands::Devices(cmd) => cmd.execute(config).await?,
                Commands::Accounts(cmd) => cmd.execute(config).await?,
                Commands::Preferences(cmd) => cmd.execute(config).await?,
                Commands::Config(cmd) => cmd.execute(config).await?,
            }
        }
        Ok(())
    }
}
```

### Search Command Implementation

```rust
// src/cli/commands/search.rs

use clap::{Args, Subcommand};
use crate::{
    config::ConfigManager,
    grpc::client::GrpcClient,
    error::Result,
};

#[derive(Args)]
pub struct SearchCommand {
    #[command(subcommand)]
    pub action: SearchAction,
}

#[derive(Subcommand)]
pub enum SearchAction {
    /// Free-text search
    Query {
        /// Search query
        query: String,

        /// Filter by genre
        #[arg(short, long)]
        genre: Option<Vec<String>>,

        /// Filter by platform
        #[arg(short, long)]
        platform: Option<Vec<String>>,

        /// Filter by year
        #[arg(short, long)]
        year: Option<u32>,

        /// Minimum rating
        #[arg(short, long)]
        rating: Option<f32>,

        /// Maximum results
        #[arg(short = 'n', long, default_value = "20")]
        max_results: usize,
    },

    /// Filtered search
    Filter {
        /// Filter criteria
        #[arg(short, long)]
        filters: Vec<String>,
    },

    /// Find similar content
    Similar {
        /// Content ID
        content_id: String,

        /// Maximum results
        #[arg(short = 'n', long, default_value = "10")]
        max_results: usize,
    },
}

impl SearchCommand {
    pub async fn execute(self, config: ConfigManager) -> Result<()> {
        match self.action {
            SearchAction::Query {
                query,
                genre,
                platform,
                year,
                rating,
                max_results,
            } => {
                self.execute_query(
                    config,
                    query,
                    genre,
                    platform,
                    year,
                    rating,
                    max_results,
                ).await
            }
            SearchAction::Filter { filters } => {
                self.execute_filter(config, filters).await
            }
            SearchAction::Similar { content_id, max_results } => {
                self.execute_similar(config, content_id, max_results).await
            }
        }
    }

    async fn execute_query(
        &self,
        config: ConfigManager,
        query: String,
        genre: Option<Vec<String>>,
        platform: Option<Vec<String>>,
        year: Option<u32>,
        rating: Option<f32>,
        max_results: usize,
    ) -> Result<()> {
        // Create gRPC client
        let mut client = GrpcClient::new(&config.get().api.endpoint).await?;

        // Build filters
        let mut filters = Vec::new();
        if let Some(genres) = genre {
            filters.push(format!("genre:{}", genres.join(",")));
        }
        if let Some(platforms) = platform {
            filters.push(format!("platform:{}", platforms.join(",")));
        }
        if let Some(year) = year {
            filters.push(format!("year:{}", year));
        }
        if let Some(rating) = rating {
            filters.push(format!("rating:>={}", rating));
        }

        // Execute search
        println!("Searching for '{}'...\n", query);

        let mut stream = client.search(query, filters).await?;
        let mut total_results = 0;

        while let Some(result) = stream.next().await {
            match result? {
                SearchResult::Chunk(chunk) => {
                    for item in chunk.items {
                        total_results += 1;
                        self.print_result(&item);

                        if total_results >= max_results {
                            break;
                        }
                    }
                }
                SearchResult::Complete(complete) => {
                    println!("\nFound {} results in {:.2}s",
                        complete.total_results,
                        complete.duration_seconds
                    );
                }
                SearchResult::Error(error) => {
                    eprintln!("Error: {}", error.message);
                }
            }
        }

        Ok(())
    }

    fn print_result(&self, item: &ContentItem) {
        println!("{} ({}) - {} ★{:.1}",
            item.title,
            item.year,
            item.platform,
            item.rating
        );
    }
}
```

### TUI Application

```rust
// src/tui/app.rs

use crossterm::{
    event::{self, DisableMouseCapture, EnableMouseCapture, Event, KeyCode},
    execute,
    terminal::{disable_raw_mode, enable_raw_mode, EnterAlternateScreen, LeaveAlternateScreen},
};
use ratatui::{
    backend::{Backend, CrosstermBackend},
    Terminal,
};
use std::io;

pub struct TuiApp {
    state: AppState,
}

pub struct AppState {
    pub current_view: View,
    pub search_input: String,
    pub results: Vec<ContentItem>,
    pub selected_index: usize,
    pub loading: bool,
}

pub enum View {
    Search,
    Results,
    Detail,
    Devices,
}

impl TuiApp {
    pub fn new() -> Self {
        Self {
            state: AppState {
                current_view: View::Search,
                search_input: String::new(),
                results: Vec::new(),
                selected_index: 0,
                loading: false,
            },
        }
    }

    pub async fn run(mut self) -> Result<()> {
        // Setup terminal
        enable_raw_mode()?;
        let mut stdout = io::stdout();
        execute!(stdout, EnterAlternateScreen, EnableMouseCapture)?;
        let backend = CrosstermBackend::new(stdout);
        let mut terminal = Terminal::new(backend)?;

        // Run app
        let res = self.run_app(&mut terminal).await;

        // Restore terminal
        disable_raw_mode()?;
        execute!(
            terminal.backend_mut(),
            LeaveAlternateScreen,
            DisableMouseCapture
        )?;
        terminal.show_cursor()?;

        res
    }

    async fn run_app<B: Backend>(&mut self, terminal: &mut Terminal<B>) -> Result<()> {
        loop {
            terminal.draw(|f| self.ui(f))?;

            if let Event::Key(key) = event::read()? {
                match key.code {
                    KeyCode::Char('q') => return Ok(()),
                    KeyCode::Char('c') if key.modifiers.contains(event::KeyModifiers::CONTROL) => {
                        return Ok(());
                    }
                    _ => self.handle_input(key).await?,
                }
            }
        }
    }

    fn ui<B: Backend>(&self, f: &mut Frame<B>) {
        match self.state.current_view {
            View::Search => self.render_search(f),
            View::Results => self.render_results(f),
            View::Detail => self.render_detail(f),
            View::Devices => self.render_devices(f),
        }
    }

    async fn handle_input(&mut self, key: event::KeyEvent) -> Result<()> {
        match self.state.current_view {
            View::Search => self.handle_search_input(key).await,
            View::Results => self.handle_results_input(key).await,
            View::Detail => self.handle_detail_input(key).await,
            View::Devices => self.handle_devices_input(key).await,
        }
    }
}
```

### Error Handling

```rust
// src/error.rs

use thiserror::Error;

#[derive(Error, Debug)]
pub enum CliError {
    #[error("gRPC error: {0}")]
    Grpc(#[from] tonic::Status),

    #[error("Authentication error: {0}")]
    Auth(#[from] AuthError),

    #[error("Configuration error: {0}")]
    Config(#[from] ConfigError),

    #[error("IO error: {0}")]
    Io(#[from] std::io::Error),

    #[error("Serialization error: {0}")]
    Serialization(String),

    #[error("Invalid input: {0}")]
    InvalidInput(String),

    #[error("Network error: {0}")]
    Network(String),
}

#[derive(Error, Debug)]
pub enum AuthError {
    #[error("Device code expired")]
    DeviceCodeExpired,

    #[error("Access denied")]
    AccessDenied,

    #[error("Token expired")]
    TokenExpired,

    #[error("Keyring error: {0}")]
    Keyring(String),
}

#[derive(Error, Debug)]
pub enum ConfigError {
    #[error("No config directory found")]
    NoConfigDir,

    #[error("Profile not found: {0}")]
    ProfileNotFound(String),

    #[error("Invalid configuration: {0}")]
    Invalid(String),
}

pub type Result<T> = std::result::Result<T, CliError>;
```

---

## Cross-Platform Considerations

### Platform-Specific Features

#### macOS
```rust
#[cfg(target_os = "macos")]
mod platform {
    pub fn get_keyring() -> Result<Keyring> {
        // Use macOS Keychain
        Keyring::new()
    }

    pub fn open_browser(url: &str) -> Result<()> {
        std::process::Command::new("open")
            .arg(url)
            .spawn()?;
        Ok(())
    }
}
```

#### Linux
```rust
#[cfg(target_os = "linux")]
mod platform {
    pub fn get_keyring() -> Result<Keyring> {
        // Use Secret Service API (libsecret)
        Keyring::new()
    }

    pub fn open_browser(url: &str) -> Result<()> {
        std::process::Command::new("xdg-open")
            .arg(url)
            .spawn()?;
        Ok(())
    }
}
```

#### Windows
```rust
#[cfg(target_os = "windows")]
mod platform {
    pub fn get_keyring() -> Result<Keyring> {
        // Use Windows Credential Manager
        Keyring::new()
    }

    pub fn open_browser(url: &str) -> Result<()> {
        std::process::Command::new("cmd")
            .args(&["/c", "start", url])
            .spawn()?;
        Ok(())
    }
}
```

### Terminal Compatibility

The CLI uses `crossterm` backend which provides cross-platform terminal support:
- Windows Terminal
- Terminal.app (macOS)
- iTerm2 (macOS)
- GNOME Terminal (Linux)
- Konsole (Linux)
- Alacritty
- WezTerm

### Build Configuration

```toml
# .cargo/config.toml

[target.x86_64-pc-windows-msvc]
rustflags = ["-C", "link-arg=/SUBSYSTEM:CONSOLE"]

[target.x86_64-apple-darwin]
rustflags = ["-C", "link-arg=-mmacosx-version-min=10.15"]

[target.x86_64-unknown-linux-gnu]
rustflags = ["-C", "link-arg=-Wl,-rpath,$ORIGIN"]
```

---

## Code Examples

### Example 1: Basic Search (Scriptable)

```bash
# Simple search
tvd search query "stranger things"

# Search with filters
tvd search query "thriller" --genre sci-fi --platform netflix --rating 8.0

# JSON output for scripting
tvd search query "breaking bad" --format json | jq '.[] | {title, rating}'

# Find similar content
tvd search similar tt0903747
```

### Example 2: Interactive TUI Mode

```bash
# Launch interactive mode
tvd --interactive

# Or just
tvd -i
```

### Example 3: Device Management

```bash
# List devices
tvd devices list

# Connect to device
tvd devices connect living-room-tv

# Cast content
tvd devices cast tt0903747 --device living-room-tv
```

### Example 4: Configuration

```bash
# Initialize config
tvd config init

# Set preference
tvd config set display.theme dark

# Show config
tvd config show

# Use profile
tvd --profile work search query "documentaries"
```

### Example 5: Authentication

```bash
# Authenticate (starts device flow)
tvd accounts link

# Output:
# ┌─ Authentication Required ────────────────────────────┐
# │   Visit: https://auth.tv-discover.com/device         │
# │   Code: WDJB-MJHT                                    │
# │   ⠋ Waiting for authorization...                    │
# └──────────────────────────────────────────────────────┘
```

### Example 6: Complete Rust Implementation Example

```rust
// Complete example of search flow

use tv_discover_cli::*;

#[tokio::main]
async fn main() -> Result<()> {
    // 1. Load config
    let config = ConfigManager::new()?;

    // 2. Initialize gRPC client
    let mut client = GrpcClient::new(&config.get().api.endpoint).await?;

    // 3. Authenticate if needed
    let token_manager = TokenManager::new(
        Keyring::new()?,
        client.auth_client(),
    );

    if !token_manager.has_valid_token().await? {
        let auth_flow = DeviceAuthFlow::new(client.auth_client());
        let token = auth_flow.authenticate().await?;
        token_manager.store_token(token)?;
    }

    // 4. Get valid token
    let token = token_manager.get_valid_token().await?;
    client = client.with_auth_token(&token);

    // 5. Execute search
    let mut stream = client.search(
        "stranger things".to_string(),
        vec!["genre:sci-fi".to_string()],
    ).await?;

    // 6. Process streaming results
    while let Some(result) = stream.next().await {
        match result? {
            SearchResult::Chunk(chunk) => {
                println!("Received {} results from {}",
                    chunk.items.len(),
                    chunk.agent_id
                );

                for item in chunk.items {
                    println!("  - {} ({})", item.title, item.platform);
                }
            }
            SearchResult::Complete(complete) => {
                println!("\nSearch complete: {} total results in {:.2}s",
                    complete.total_results,
                    complete.duration_seconds
                );
            }
            SearchResult::Error(error) => {
                eprintln!("Error: {}", error.message);
            }
        }
    }

    Ok(())
}
```

---

## TUI Mockups (ASCII Art)

### Full Application Layout

```
┌─ TV Discover ──────────────────────────────────────────────────────────────┐
│                                                                             │
│  Search: thriller available:netflix ★>8.0                        [Ctrl+/]  │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │ Filters: [Genre: Thriller] [Platform: Netflix] [Rating: >8.0]        │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  Results (8 found in 1.2s)                                    Sort: Rating │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │ ▸ Black Mirror (TV Series)                          Netflix ★8.8      │ │
│  │   Stranger Things (TV Series)                       Netflix ★8.7      │ │
│  │   Mindhunter (TV Series)                            Netflix ★8.6      │ │
│  │   The Haunting of Hill House (TV Series)            Netflix ★8.6      │ │
│  │   Dark (TV Series)                                  Netflix ★8.8      │ │
│  │   Ozark (TV Series)                                 Netflix ★8.5      │ │
│  │   You (TV Series)                                   Netflix ★8.1      │ │
│  │   The Sinner (TV Series)                            Netflix ★8.0      │ │
│  │                                                                        │ │
│  │ [↑/↓: Navigate] [Enter: Details] [W: Watch] [Q: Queue] [/: Search]   │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  Status: Ready | Profile: home | Devices: 3 connected                      │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Detail View

```
┌─ Black Mirror ─────────────────────────────────────────────────────────────┐
│                                                                             │
│  [Netflix] ★ 8.8/10 (IMDb) | ★ 85% (RT)               TV-MA | 2011-2019   │
│                                                                             │
│  Genres: Sci-Fi, Thriller, Drama                                           │
│  Seasons: 5 | Episodes: 22 | Runtime: 60min                                │
│  Creator: Charlie Brooker                                                  │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ An anthology series exploring a twisted, high-tech near-future     │   │
│  │ where humanity's greatest innovations and darkest instincts        │   │
│  │ collide.                                                           │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  Cast:                                                                      │
│    Daniel Kaluuya, Jessica Brown Findlay, Bryce Dallas Howard             │
│                                                                             │
│  Episodes:                                                                  │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │  S01E01  The National Anthem                              ★8.2       │ │
│  │  S01E02  Fifteen Million Merits                           ★8.2       │ │
│  │  S01E03  The Entire History of You                        ★8.5       │ │
│  │  S02E01  Be Right Back                                    ★8.0       │ │
│  │  ...                                                                  │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  ┌─ Actions ──────────────────────────────────────────────────────────────┐│
│  │ [W] Watch Now    [Q] Add to Queue     [L] Add to List                ││
│  │ [S] Similar      [C] Cast to Device   [Esc] Back                     ││
│  └────────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────────┘
```

### Streaming Search Progress

```
┌─ Searching for "breaking bad"... ──────────────────────────────────────────┐
│                                                                             │
│  ┌─ Agent Status ─────────────────────────────────────────────────────────┐│
│  │ ✓ Netflix        5 results    [████████████████████] 100%             ││
│  │ ⠙ Prime Video   Searching...  [████████░░░░░░░░░░░░]  45%             ││
│  │ ⠸ HBO Max       Searching...  [██████░░░░░░░░░░░░░░]  32%             ││
│  │ ✓ Hulu          3 results    [████████████████████] 100%             ││
│  │ ⠋ Disney+       Waiting...     [░░░░░░░░░░░░░░░░░░░░]   0%             ││
│  └────────────────────────────────────────────────────────────────────────┘│
│                                                                             │
│  Results so far (8):                                                        │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │ ▸ Breaking Bad (TV Series)                          Netflix ★9.5      │ │
│  │   El Camino: A Breaking Bad Movie                   Netflix ★7.3      │ │
│  │   Better Call Saul (TV Series)                      Netflix ★9.0      │ │
│  │   Breaking Bad: Behind the Scenes                   Netflix ★8.1      │ │
│  │   Breaking Bad (TV Series)                     Prime Video ★9.5      │ │
│  │   Metastasis (TV Series)                            Hulu ★7.8      │ │
│  │   Breaking Bad: The Movie                           Hulu ★6.2      │ │
│  │   ...                                                                 │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  [Esc] Cancel search                                                        │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Conclusion

This specification provides a complete architecture for the TV Discovery CLI built in Rust. The design emphasizes:

1. **Cross-platform compatibility** through careful use of platform-agnostic libraries
2. **Dual interface** supporting both scriptable commands and interactive TUI
3. **Real-time streaming** with progressive result display
4. **Secure authentication** using Device Authorization Grant and keyring storage
5. **Flexible configuration** with profiles and environment variable support
6. **Modern Rust practices** using async/await, proper error handling, and type safety

The implementation leverages the Rust ecosystem's best tools:
- `clap` for powerful CLI parsing
- `ratatui` for beautiful TUI
- `tonic` for efficient gRPC communication
- `tokio` for async runtime
- `keyring` for secure credential storage

This architecture provides a solid foundation for building a production-ready CLI that can scale with the platform while maintaining excellent user experience.
