# MCP Protocol Specification
## Model Context Protocol Integration for TV Discovery System

---

## Table of Contents

1. [Overview](#1-overview)
2. [Protocol Definitions](#2-protocol-definitions)
3. [Tool Specifications](#3-tool-specifications)
4. [Service Communication Patterns](#4-service-communication-patterns)
5. [Error Handling](#5-error-handling)
6. [Security and Authentication](#6-security-and-authentication)

---

## 1. Overview

### 1.1 MCP Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                    MCP ARCHITECTURE                                   │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Agent Orchestrator                                                  │
│         │                                                            │
│         │ MCP Tool Registry                                          │
│         ▼                                                            │
│  ┌─────────────────────────────────────────────┐                    │
│  │  MCP Executor                                │                    │
│  │  - Tool discovery                            │                    │
│  │  - Input validation                          │                    │
│  │  - Output validation                         │                    │
│  │  - Error handling                            │                    │
│  └────────────────┬────────────────────────────┘                    │
│                   │                                                  │
│                   │ gRPC / HTTP / Internal                           │
│                   ▼                                                  │
│  ┌────────────────────────────────────────────────────┐             │
│  │  Service Layer                                      │             │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────┐ │             │
│  │  │  Semantic    │  │  Recommend   │  │  Rights  │ │             │
│  │  │  Search      │  │  Engine      │  │Validator │ │             │
│  │  └──────────────┘  └──────────────┘  └──────────┘ │             │
│  └────────────────────────────────────────────────────┘             │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

### 1.2 Tool Lifecycle

```
┌──────────────────────────────────────────────────────────────────────┐
│                    TOOL LIFECYCLE                                     │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. REGISTRATION                                                     │
│     - Tool metadata defined in YAML                                  │
│     - Schema validation rules                                        │
│     - Service endpoint configuration                                 │
│                                                                      │
│  2. DISCOVERY                                                        │
│     - Agent queries tool registry                                    │
│     - Capability matching                                            │
│     - Tool selection                                                 │
│                                                                      │
│  3. VALIDATION                                                       │
│     - Input schema validation                                        │
│     - Type checking                                                  │
│     - Required field verification                                    │
│                                                                      │
│  4. EXECUTION                                                        │
│     - Service client creation                                        │
│     - Request serialization                                          │
│     - Timeout management                                             │
│     - Retry logic                                                    │
│                                                                      │
│  5. RESPONSE HANDLING                                                │
│     - Output validation                                              │
│     - Error parsing                                                  │
│     - Result transformation                                          │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 2. Protocol Definitions

### 2.1 gRPC Service Definitions

**proto/search.proto:**

```protobuf
syntax = "proto3";

package tv.discovery.search;

service SemanticSearch {
  // Streaming search for progressive results
  rpc Search(SearchRequest) returns (stream SearchResponse);

  // Batch search for multiple queries
  rpc BatchSearch(BatchSearchRequest) returns (BatchSearchResponse);

  // Find similar content
  rpc FindSimilar(SimilarRequest) returns (SimilarResponse);
}

message SearchRequest {
  string query = 1;
  SearchFilters filters = 2;
  int32 limit = 3;
  int32 offset = 4;
  string request_id = 5;
}

message SearchFilters {
  repeated string genres = 1;
  repeated string platforms = 2;
  string region = 3;
  double min_trust_score = 4;
  DateRange release_date_range = 5;
  repeated string content_types = 6; // movie, series, etc.
}

message DateRange {
  string start_date = 1; // ISO 8601
  string end_date = 2;   // ISO 8601
}

message SearchResponse {
  repeated ContentResult results = 1;
  int32 total_found = 2;
  int32 offset = 3;
  SearchMetadata metadata = 4;
}

message ContentResult {
  string content_id = 1;
  string title = 2;
  string description = 3;
  double relevance_score = 4;
  double trust_score = 5;
  TrustBreakdown trust_breakdown = 6;
  repeated Availability availability = 7;
  ContentMetadata content_metadata = 8;
}

message TrustBreakdown {
  double source_reliability = 1;
  double metadata_accuracy = 2;
  double availability_confidence = 3;
  double recommendation_quality = 4;
  double preference_confidence = 5;
}

message Availability {
  string platform = 1;
  string pricing_model = 2; // INCLUDED, FREE, RENT, BUY
  repeated string quality_tiers = 3; // SD, HD, 4K, HDR
  string deep_link = 4;
  int64 expires_at = 5; // Unix timestamp
}

message ContentMetadata {
  string content_type = 1; // movie, series, episode
  repeated string genres = 2;
  repeated Person cast = 3;
  repeated Person crew = 4;
  double rating = 5;
  int32 release_year = 6;
  string release_date = 7;
  int32 runtime_minutes = 8;
  string mood = 9;
}

message Person {
  string id = 1;
  string name = 2;
  string role = 3; // actor, director, etc.
}

message SearchMetadata {
  int64 execution_time_ms = 1;
  string query_id = 2;
  repeated string agents_used = 3;
}

message BatchSearchRequest {
  repeated SearchRequest requests = 1;
}

message BatchSearchResponse {
  repeated SearchResponse responses = 1;
}

message SimilarRequest {
  string content_id = 1;
  int32 limit = 2;
  SearchFilters filters = 3;
}

message SimilarResponse {
  repeated ContentResult results = 1;
  int32 total_found = 2;
}
```

**proto/recommendation.proto:**

```protobuf
syntax = "proto3";

package tv.discovery.recommendation;

service RecommendationEngine {
  // Get personalized recommendations
  rpc Recommend(RecommendRequest) returns (RecommendResponse);

  // Get recommendations with explanations
  rpc RecommendWithExplanations(RecommendRequest) returns (stream ExplainedRecommendation);

  // Update user preferences
  rpc UpdatePreferences(UpdatePreferencesRequest) returns (UpdatePreferencesResponse);
}

message RecommendRequest {
  string user_id = 1;
  repeated string content_context = 2; // Content IDs for context
  int32 num_recommendations = 3;
  double diversity_factor = 4; // 0.0 to 1.0
  RecommendFilters filters = 5;
  string request_id = 6;
}

message RecommendFilters {
  repeated string genres = 1;
  repeated string platforms = 2;
  string region = 3;
  repeated string excluded_content_ids = 4;
  double min_rating = 5;
}

message RecommendResponse {
  repeated Recommendation recommendations = 1;
  UserProfile user_profile = 2;
  RecommendMetadata metadata = 3;
}

message Recommendation {
  string content_id = 1;
  string title = 2;
  string description = 3;
  double score = 4;
  repeated string genres = 5;
  double rating = 6;
  repeated Availability availability = 7;
  RecommendationReason reason = 8;
}

message RecommendationReason {
  string explanation = 1;
  repeated string based_on_content_ids = 2;
  repeated string matched_preferences = 3;
  double novelty_score = 4;
}

message UserProfile {
  string user_id = 1;
  map<string, double> genre_scores = 2;
  repeated string favorite_content_ids = 3;
  int64 last_updated = 4;
}

message RecommendMetadata {
  int64 execution_time_ms = 1;
  string algorithm_version = 2;
  repeated string models_used = 3; // gnn, collaborative, etc.
}

message ExplainedRecommendation {
  Recommendation recommendation = 1;
  DetailedExplanation explanation = 2;
}

message DetailedExplanation {
  string narrative = 1;
  repeated ExplanationFactor factors = 2;
  double confidence = 3;
}

message ExplanationFactor {
  string factor_type = 1; // preference_match, similar_to_watched, trending
  double weight = 2;
  string description = 3;
}

message UpdatePreferencesRequest {
  string user_id = 1;
  UserPreferences preferences = 2;
}

message UserPreferences {
  map<string, double> genre_preferences = 1;
  repeated string favorite_actors = 2;
  repeated string favorite_directors = 3;
  repeated string excluded_genres = 4;
}

message UpdatePreferencesResponse {
  bool success = 1;
  UserProfile updated_profile = 2;
}
```

**proto/rights.proto:**

```protobuf
syntax = "proto3";

package tv.discovery.rights;

service RightsValidator {
  // Check content availability
  rpc CheckAvailability(AvailabilityRequest) returns (AvailabilityResponse);

  // Batch availability check
  rpc BatchCheckAvailability(BatchAvailabilityRequest) returns (BatchAvailabilityResponse);

  // Validate deep link
  rpc ValidateDeepLink(DeepLinkRequest) returns (DeepLinkResponse);

  // Get upcoming expirations
  rpc GetExpiringContent(ExpirationRequest) returns (ExpirationResponse);
}

message AvailabilityRequest {
  string content_id = 1;
  string region = 2;
  string request_id = 3;
}

message AvailabilityResponse {
  string content_id = 1;
  string region = 2;
  bool available = 3;
  repeated PlatformAvailability platforms = 4;
  repeated string restrictions = 5;
  int64 checked_at = 6;
}

message PlatformAvailability {
  string platform = 1;
  string pricing_model = 2;
  repeated string quality_tiers = 3;
  string deep_link = 4;
  int64 available_from = 5;
  int64 available_until = 6;
  bool exclusive = 7;
  LicenseInfo license_info = 8;
}

message LicenseInfo {
  string license_id = 1;
  string territory = 2;
  repeated string rights = 3; // streaming, download, etc.
}

message BatchAvailabilityRequest {
  repeated AvailabilityRequest requests = 1;
}

message BatchAvailabilityResponse {
  repeated AvailabilityResponse responses = 1;
}

message DeepLinkRequest {
  string content_id = 1;
  string platform = 2;
  string device_type = 3; // web, mobile, tv
}

message DeepLinkResponse {
  string url = 1;
  bool valid = 2;
  int64 validated_at = 3;
  string error_message = 4;
}

message ExpirationRequest {
  string region = 1;
  int32 days_ahead = 2; // How many days to look ahead
  repeated string platforms = 3;
}

message ExpirationResponse {
  repeated ExpiringContent content = 1;
}

message ExpiringContent {
  string content_id = 1;
  string title = 2;
  string platform = 3;
  int64 expires_at = 4;
  int32 days_remaining = 5;
}
```

**proto/device.proto:**

```protobuf
syntax = "proto3";

package tv.discovery.device;

service DeviceGateway {
  // Send command to device
  rpc SendCommand(DeviceCommandRequest) returns (DeviceCommandResponse);

  // Sync playback state
  rpc SyncPlayback(SyncPlaybackRequest) returns (SyncPlaybackResponse);

  // Get device state
  rpc GetDeviceState(DeviceStateRequest) returns (DeviceStateResponse);

  // Stream device events
  rpc StreamDeviceEvents(DeviceEventsRequest) returns (stream DeviceEvent);
}

message DeviceCommandRequest {
  string device_id = 1;
  string device_type = 2; // samsung, lg, roku, etc.
  DeviceCommand command = 3;
  string request_id = 4;
}

message DeviceCommand {
  string action = 1; // play, pause, stop, seek, sync_playback
  map<string, string> payload = 2;
}

message DeviceCommandResponse {
  bool success = 1;
  string error = 2;
  DeviceState state = 3;
  int64 executed_at = 4;
}

message DeviceState {
  string device_id = 1;
  string status = 2; // idle, playing, paused
  PlaybackState playback = 3;
  int64 last_updated = 4;
}

message PlaybackState {
  string content_id = 1;
  int32 position_seconds = 2;
  int32 duration_seconds = 3;
  double playback_rate = 4;
  int64 timestamp = 5;
}

message SyncPlaybackRequest {
  repeated string device_ids = 1;
  PlaybackState state = 2;
}

message SyncPlaybackResponse {
  map<string, bool> sync_status = 1; // device_id -> success
  repeated string errors = 2;
}

message DeviceStateRequest {
  string device_id = 1;
}

message DeviceStateResponse {
  DeviceState state = 1;
}

message DeviceEventsRequest {
  repeated string device_ids = 1;
  repeated string event_types = 2; // playback, connection, error
}

message DeviceEvent {
  string device_id = 1;
  string event_type = 2;
  int64 timestamp = 3;
  map<string, string> data = 4;
}
```

**proto/memory.proto:**

```protobuf
syntax = "proto3";

package tv.discovery.memory;

service MemoryService {
  // Store data
  rpc Store(StoreRequest) returns (StoreResponse);

  // Retrieve data
  rpc Retrieve(RetrieveRequest) returns (RetrieveResponse);

  // Query data
  rpc Query(QueryRequest) returns (QueryResponse);

  // Delete data
  rpc Delete(DeleteRequest) returns (DeleteResponse);
}

message StoreRequest {
  string database = 1; // agentdb, reasoningbank
  string collection = 2;
  string key = 3;
  bytes value = 4; // Serialized data
  int32 ttl = 5; // Seconds
  bool merge = 6;
  map<string, string> metadata = 7;
}

message StoreResponse {
  bool success = 1;
  string key = 2;
  string error = 3;
}

message RetrieveRequest {
  string database = 1;
  string collection = 2;
  string key = 3;
}

message RetrieveResponse {
  bool found = 1;
  bytes value = 2;
  map<string, string> metadata = 3;
  int64 stored_at = 4;
}

message QueryRequest {
  string database = 1;
  string collection = 2;
  map<string, string> filters = 3;
  int32 limit = 4;
  int32 offset = 5;
  string sort_by = 6;
  bool sort_desc = 7;
}

message QueryResponse {
  repeated QueryResult results = 1;
  int32 total_found = 2;
}

message QueryResult {
  string key = 1;
  bytes value = 2;
  map<string, string> metadata = 3;
}

message DeleteRequest {
  string database = 1;
  string collection = 2;
  string key = 3;
}

message DeleteResponse {
  bool success = 1;
  string error = 2;
}
```

---

## 3. Tool Specifications

### 3.1 Ruvector Search Tool

**Complete Tool Specification:**

```yaml
tool:
  name: ruvector_search
  version: 1.0.0
  description: |
    Semantic search over content catalog using Ruvector's
    vector + graph capabilities. Supports hybrid search combining
    vector similarity and graph traversal.

  service:
    name: semantic-search
    endpoint: semantic-search.default.svc.cluster.local:50051
    protocol: grpc
    timeout: 5000
    retries: 3
    circuit_breaker:
      enabled: true
      failure_threshold: 5
      success_threshold: 2
      timeout: 60000

  input_schema:
    type: object
    required:
      - query
    properties:
      query:
        type: string
        description: Natural language search query
        minLength: 1
        maxLength: 500
        example: "sci-fi shows like Stranger Things"

      filters:
        type: object
        description: Search filters
        properties:
          genres:
            type: array
            items:
              type: string
              enum: [action, comedy, drama, sci-fi, thriller, horror, romance]
            description: Filter by genres

          platforms:
            type: array
            items:
              type: string
            description: Filter by platforms

          region:
            type: string
            pattern: "^[A-Z]{2}$"
            description: Two-letter region code (ISO 3166-1 alpha-2)
            example: "US"

          minTrustScore:
            type: number
            minimum: 0.0
            maximum: 1.0
            default: 0.7
            description: Minimum trust score for results

          contentTypes:
            type: array
            items:
              type: string
              enum: [movie, series, episode, documentary]
            description: Filter by content types

          releaseDateRange:
            type: object
            properties:
              start:
                type: string
                format: date
                description: Start date (YYYY-MM-DD)
              end:
                type: string
                format: date
                description: End date (YYYY-MM-DD)

      limit:
        type: integer
        minimum: 1
        maximum: 100
        default: 20
        description: Maximum number of results

      offset:
        type: integer
        minimum: 0
        default: 0
        description: Offset for pagination

  output_schema:
    type: object
    properties:
      results:
        type: array
        items:
          type: object
          properties:
            contentId:
              type: string
            title:
              type: string
            description:
              type: string
            relevanceScore:
              type: number
            trustScore:
              type: number
            availability:
              type: array
              items:
                type: object

      total:
        type: integer
        description: Total number of matching results

      metadata:
        type: object
        properties:
          executionTimeMs:
            type: integer
          queryId:
            type: string

  error_codes:
    - code: INVALID_QUERY
      description: Query is empty or malformed
      http_status: 400

    - code: INVALID_REGION
      description: Region code is invalid
      http_status: 400

    - code: SERVICE_UNAVAILABLE
      description: Ruvector service is unavailable
      http_status: 503

    - code: TIMEOUT
      description: Search timed out
      http_status: 504

    - code: INTERNAL_ERROR
      description: Internal server error
      http_status: 500

  examples:
    - name: Simple search
      input:
        query: "sci-fi shows"
        limit: 10

      output:
        results:
          - contentId: "tt1"
            title: "Stranger Things"
            relevanceScore: 0.95
        total: 42

    - name: Filtered search
      input:
        query: "action movies"
        filters:
          genres: ["action"]
          platforms: ["netflix", "prime"]
          region: "US"
          minTrustScore: 0.8
        limit: 20

      output:
        results:
          - contentId: "tt2"
            title: "The Matrix"
            relevanceScore: 0.92
        total: 156
```

### 3.2 GNN Recommend Tool

**Complete Tool Specification:**

```yaml
tool:
  name: gnn_recommend
  version: 1.0.0
  description: |
    Graph Neural Network-based personalized recommendations.
    Uses Ruvector's built-in GNN capabilities for collaborative
    and content-based filtering.

  service:
    name: recommendation-engine
    endpoint: recommendation-engine.default.svc.cluster.local:50051
    protocol: grpc
    timeout: 10000
    retries: 2

  input_schema:
    type: object
    required:
      - userId
    properties:
      userId:
        type: string
        description: User identifier
        minLength: 1

      contentContext:
        type: array
        items:
          type: string
        default: []
        description: Content IDs to use as context
        maxItems: 10

      numRecommendations:
        type: integer
        minimum: 1
        maximum: 100
        default: 20
        description: Number of recommendations to return

      diversityFactor:
        type: number
        minimum: 0.0
        maximum: 1.0
        default: 0.3
        description: |
          Diversity vs relevance trade-off.
          0.0 = maximize relevance
          1.0 = maximize diversity

      filters:
        type: object
        properties:
          genres:
            type: array
            items:
              type: string

          platforms:
            type: array
            items:
              type: string

          region:
            type: string

          excludedContentIds:
            type: array
            items:
              type: string
            description: Content to exclude from recommendations

  output_schema:
    type: object
    properties:
      recommendations:
        type: array
        items:
          type: object
          properties:
            contentId:
              type: string
            score:
              type: number
            reason:
              type: string
            basedOnContentIds:
              type: array
              items:
                type: string

      metadata:
        type: object
        properties:
          executionTimeMs:
            type: integer
          algorithmVersion:
            type: string
          modelsUsed:
            type: array
            items:
              type: string

  error_codes:
    - code: USER_NOT_FOUND
      description: User profile not found
      http_status: 404

    - code: INSUFFICIENT_DATA
      description: Insufficient user data for recommendations
      http_status: 400

    - code: MODEL_ERROR
      description: GNN model execution error
      http_status: 500

  examples:
    - name: Basic recommendations
      input:
        userId: "user-123"
        numRecommendations: 10

      output:
        recommendations:
          - contentId: "tt1"
            score: 0.87
            reason: "Based on your preferences"
        metadata:
          modelsUsed: ["gnn", "collaborative"]

    - name: Context-based recommendations
      input:
        userId: "user-123"
        contentContext: ["tt-stranger-things", "tt-dark"]
        numRecommendations: 20
        diversityFactor: 0.4

      output:
        recommendations:
          - contentId: "tt-the-oa"
            score: 0.92
            reason: "Similar to Stranger Things and Dark"
            basedOnContentIds: ["tt-stranger-things", "tt-dark"]
```

### 3.3 Memory Store Tool

**Complete Tool Specification:**

```yaml
tool:
  name: memory_store
  version: 1.0.0
  description: |
    Store data in AgentDB (Redis) or ReasoningBank (Ruvector).
    Supports TTL, merge operations, and metadata tagging.

  service:
    name: agent-orchestrator
    endpoint: localhost:50051
    protocol: internal
    timeout: 1000

  input_schema:
    type: object
    required:
      - database
      - collection
      - key
      - value
    properties:
      database:
        type: string
        enum: [agentdb, reasoningbank]
        description: Target database

      collection:
        type: string
        description: Collection/namespace
        pattern: "^[a-zA-Z0-9_-]+$"

      key:
        type: string
        description: Storage key
        minLength: 1
        maxLength: 256

      value:
        type: any
        description: Value to store (will be serialized)

      ttl:
        type: integer
        minimum: 0
        default: 3600
        description: Time to live in seconds (0 = no expiration)

      merge:
        type: boolean
        default: false
        description: Merge with existing value (for objects)

      metadata:
        type: object
        additionalProperties:
          type: string
        description: Additional metadata

  output_schema:
    type: object
    properties:
      success:
        type: boolean
      key:
        type: string
      error:
        type: string

  error_codes:
    - code: INVALID_DATABASE
      description: Database name is invalid
      http_status: 400

    - code: STORAGE_FULL
      description: Storage quota exceeded
      http_status: 507

    - code: SERIALIZATION_ERROR
      description: Failed to serialize value
      http_status: 400

  examples:
    - name: Store user context
      input:
        database: agentdb
        collection: context
        key: "user-123:preferences"
        value:
          genres: ["sci-fi", "thriller"]
          platforms: ["netflix", "prime"]
        ttl: 7200

      output:
        success: true
        key: "user-123:preferences"

    - name: Store query pattern
      input:
        database: reasoningbank
        collection: patterns
        key: "pattern-456"
        value:
          queryEmbedding: [0.1, 0.2, ...]
          executionStrategy: { ... }
        ttl: 604800  # 7 days
        metadata:
          userId: "user-123"
          queryType: "search"

      output:
        success: true
        key: "pattern-456"
```

---

## 4. Service Communication Patterns

### 4.1 Request/Response Pattern

```typescript
// Example: Synchronous search request
async function searchContent(query: string): Promise<SearchResults> {
  const client = createGrpcClient<SemanticSearchClient>(
    'semantic-search.default.svc.cluster.local:50051'
  );

  const request: SearchRequest = {
    query,
    filters: {
      region: 'US',
      minTrustScore: 0.7
    },
    limit: 20,
    offset: 0,
    requestId: generateRequestId()
  };

  try {
    const response = await client.Search(request);
    return response;
  } catch (error) {
    throw new SearchError('Search failed', error);
  }
}
```

### 4.2 Streaming Pattern

```typescript
// Example: Streaming search results
async function* searchContentStreaming(
  query: string
): AsyncIterable<ContentResult> {
  const client = createGrpcClient<SemanticSearchClient>(
    'semantic-search.default.svc.cluster.local:50051'
  );

  const request: SearchRequest = {
    query,
    filters: { region: 'US' },
    limit: 100,
    requestId: generateRequestId()
  };

  const stream = client.Search(request);

  for await (const response of stream) {
    for (const result of response.results) {
      yield result;
    }
  }
}
```

### 4.3 Batch Pattern

```typescript
// Example: Batch availability check
async function checkAvailabilityBatch(
  contentIds: string[],
  region: string
): Promise<Map<string, AvailabilityResponse>> {
  const client = createGrpcClient<RightsValidatorClient>(
    'rights-validator.default.svc.cluster.local:50051'
  );

  const batchRequest: BatchAvailabilityRequest = {
    requests: contentIds.map(contentId => ({
      contentId,
      region,
      requestId: generateRequestId()
    }))
  };

  const batchResponse = await client.BatchCheckAvailability(batchRequest);

  const resultMap = new Map<string, AvailabilityResponse>();
  for (const response of batchResponse.responses) {
    resultMap.set(response.contentId, response);
  }

  return resultMap;
}
```

### 4.4 Circuit Breaker Pattern

```typescript
// Circuit breaker for service resilience
class CircuitBreaker {
  private state: 'CLOSED' | 'OPEN' | 'HALF_OPEN' = 'CLOSED';
  private failureCount: number = 0;
  private successCount: number = 0;
  private lastFailureTime: number = 0;

  constructor(
    private failureThreshold: number = 5,
    private successThreshold: number = 2,
    private timeout: number = 60000
  ) {}

  async execute<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === 'OPEN') {
      if (Date.now() - this.lastFailureTime > this.timeout) {
        this.state = 'HALF_OPEN';
        this.successCount = 0;
      } else {
        throw new Error('Circuit breaker is OPEN');
      }
    }

    try {
      const result = await fn();

      if (this.state === 'HALF_OPEN') {
        this.successCount++;
        if (this.successCount >= this.successThreshold) {
          this.state = 'CLOSED';
          this.failureCount = 0;
        }
      }

      return result;
    } catch (error) {
      this.failureCount++;
      this.lastFailureTime = Date.now();

      if (this.failureCount >= this.failureThreshold) {
        this.state = 'OPEN';
      }

      throw error;
    }
  }
}

// Usage with MCP tool
const circuitBreaker = new CircuitBreaker(5, 2, 60000);

async function executeToolWithCircuitBreaker(
  toolName: string,
  input: any
): Promise<any> {
  return await circuitBreaker.execute(async () => {
    return await mcpExecutor.execute(toolName, input);
  });
}
```

---

## 5. Error Handling

### 5.1 Error Response Format

```typescript
interface MCPError {
  code: string;
  message: string;
  details?: Record<string, any>;
  timestamp: number;
  requestId?: string;
  retryable: boolean;
}

// Standard error codes
enum ErrorCode {
  // Client errors (4xx)
  INVALID_INPUT = 'INVALID_INPUT',
  INVALID_QUERY = 'INVALID_QUERY',
  INVALID_REGION = 'INVALID_REGION',
  USER_NOT_FOUND = 'USER_NOT_FOUND',
  INSUFFICIENT_DATA = 'INSUFFICIENT_DATA',

  // Server errors (5xx)
  INTERNAL_ERROR = 'INTERNAL_ERROR',
  SERVICE_UNAVAILABLE = 'SERVICE_UNAVAILABLE',
  TIMEOUT = 'TIMEOUT',
  MODEL_ERROR = 'MODEL_ERROR',
  STORAGE_ERROR = 'STORAGE_ERROR',

  // Custom errors
  CIRCUIT_BREAKER_OPEN = 'CIRCUIT_BREAKER_OPEN',
  RATE_LIMIT_EXCEEDED = 'RATE_LIMIT_EXCEEDED'
}
```

### 5.2 Error Handling Strategy

```typescript
class MCPErrorHandler {
  async handleError(error: Error, context: { toolName: string; input: any }): Promise<void> {
    if (error instanceof GrpcError) {
      return this.handleGrpcError(error, context);
    } else if (error instanceof TimeoutError) {
      return this.handleTimeoutError(error, context);
    } else if (error instanceof ValidationError) {
      return this.handleValidationError(error, context);
    } else {
      return this.handleUnknownError(error, context);
    }
  }

  private async handleGrpcError(
    error: GrpcError,
    context: { toolName: string; input: any }
  ): Promise<void> {
    const mcpError: MCPError = {
      code: this.mapGrpcStatusToErrorCode(error.code),
      message: error.message,
      details: error.details,
      timestamp: Date.now(),
      requestId: context.input.requestId,
      retryable: this.isRetryable(error.code)
    };

    // Log error
    logger.error('MCP tool execution failed', {
      toolName: context.toolName,
      error: mcpError
    });

    // Emit error event
    eventBus.emit('mcp.error', mcpError);

    throw mcpError;
  }

  private mapGrpcStatusToErrorCode(grpcStatus: number): ErrorCode {
    switch (grpcStatus) {
      case 3: // INVALID_ARGUMENT
        return ErrorCode.INVALID_INPUT;
      case 4: // DEADLINE_EXCEEDED
        return ErrorCode.TIMEOUT;
      case 5: // NOT_FOUND
        return ErrorCode.USER_NOT_FOUND;
      case 14: // UNAVAILABLE
        return ErrorCode.SERVICE_UNAVAILABLE;
      default:
        return ErrorCode.INTERNAL_ERROR;
    }
  }

  private isRetryable(grpcStatus: number): boolean {
    // Retryable errors: UNAVAILABLE, DEADLINE_EXCEEDED, RESOURCE_EXHAUSTED
    return [14, 4, 8].includes(grpcStatus);
  }
}
```

---

## 6. Security and Authentication

### 6.1 Service-to-Service Authentication

```yaml
# mTLS Configuration
mtls:
  enabled: true
  ca_cert: /etc/certs/ca.crt
  client_cert: /etc/certs/client.crt
  client_key: /etc/certs/client.key
  verify_peer: true
```

```typescript
// gRPC client with mTLS
function createSecureGrpcClient<T>(
  endpoint: string,
  serviceName: string
): T {
  const credentials = grpc.credentials.createSsl(
    fs.readFileSync('/etc/certs/ca.crt'),
    fs.readFileSync('/etc/certs/client.key'),
    fs.readFileSync('/etc/certs/client.crt')
  );

  return new ServiceClient(endpoint, credentials);
}
```

### 6.2 Request Authentication

```typescript
// Add authentication metadata to requests
function addAuthMetadata(metadata: grpc.Metadata): grpc.Metadata {
  const token = getServiceToken();
  metadata.add('authorization', `Bearer ${token}`);
  metadata.add('x-service-name', 'agent-orchestrator');
  metadata.add('x-request-id', generateRequestId());

  return metadata;
}

// Example usage
async function authenticatedSearch(query: string): Promise<SearchResults> {
  const client = createGrpcClient<SemanticSearchClient>(endpoint);

  const metadata = new grpc.Metadata();
  addAuthMetadata(metadata);

  const request: SearchRequest = { query, ... };

  return await client.Search(request, metadata);
}
```

### 6.3 Rate Limiting

```typescript
// Rate limiter per tool
class RateLimiter {
  private buckets: Map<string, TokenBucket> = new Map();

  async checkLimit(toolName: string, userId: string): Promise<boolean> {
    const key = `${toolName}:${userId}`;
    const bucket = this.getBucket(key, toolName);

    return bucket.consume();
  }

  private getBucket(key: string, toolName: string): TokenBucket {
    if (!this.buckets.has(key)) {
      const limits = this.getToolLimits(toolName);
      this.buckets.set(key, new TokenBucket(limits));
    }

    return this.buckets.get(key)!;
  }

  private getToolLimits(toolName: string): RateLimitConfig {
    const limits: Record<string, RateLimitConfig> = {
      ruvector_search: { capacity: 100, refillRate: 10 },
      gnn_recommend: { capacity: 50, refillRate: 5 },
      rights_check: { capacity: 200, refillRate: 20 }
    };

    return limits[toolName] || { capacity: 100, refillRate: 10 };
  }
}

// Token bucket implementation
class TokenBucket {
  private tokens: number;

  constructor(private config: RateLimitConfig) {
    this.tokens = config.capacity;
    this.startRefill();
  }

  consume(): boolean {
    if (this.tokens > 0) {
      this.tokens--;
      return true;
    }
    return false;
  }

  private startRefill(): void {
    setInterval(() => {
      this.tokens = Math.min(
        this.tokens + this.config.refillRate,
        this.config.capacity
      );
    }, 1000);
  }
}
```

---

**Document Version: 1.0.0**
**Last Updated: December 2025**
**Authors: MCP Integration Team**
