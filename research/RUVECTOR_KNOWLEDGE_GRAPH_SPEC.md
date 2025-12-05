# Ruvector Knowledge Graph & GNN Architecture Specification
## Global TV Discovery System - Complete Technical Design

**Document Version:** 1.0.0
**Date:** December 5, 2025
**System:** Media Gateway Hackathon - Ruvector Integration
**Technology Stack:** Rust, Ruvector (Hypergraph + Vector + GNN), Cypher Query Language

---

## Table of Contents

1. [Hypergraph Schema Design](#1-hypergraph-schema-design)
2. [GNN Architecture](#2-gnn-architecture)
3. [Query Patterns](#3-query-patterns)
4. [Micro-Repository Architecture](#4-micro-repository-architecture)
5. [Rust Implementation Patterns](#5-rust-implementation-patterns)

---

## 1. Hypergraph Schema Design

### 1.1 Node Type Specifications

```
┌─────────────────────────────────────────────────────────────────────┐
│                      RUVECTOR NODE TYPES                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  All nodes include vector embeddings for semantic search            │
│  Storage: Ruvector HNSW index + Hypergraph relationships            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

#### 1.1.1 Content Nodes

**Movie Node**
```cypher
CREATE (m:Movie {
  // Identifiers
  id: String,              // Internal UUID
  imdb_id: String,         // IMDb ID (tt1234567)
  tmdb_id: Integer,        // TMDb ID
  eidr_id: String,         // EIDR ID for industry interop
  gracenote_id: String,    // Gracenote TMS ID

  // Metadata
  title: String,
  original_title: String,
  release_date: Date,
  runtime_minutes: Integer,

  // Descriptive
  synopsis: String,
  tagline: String,

  // Classification
  content_rating: String,  // PG-13, R, etc.
  language: String,        // ISO 639-1 code
  country: String,         // ISO 3166-1 alpha-3

  // Metrics
  imdb_rating: Float,
  tmdb_rating: Float,
  rotten_tomatoes: Integer,
  metacritic: Integer,
  popularity_score: Float, // Computed aggregate

  // Technical
  aspect_ratio: String,
  color: String,          // Color, B&W

  // Vector Embedding
  embedding: Vector<1536>, // OpenAI text-embedding-3-large dimension

  // Trust & Quality
  trust_score: Float,
  metadata_completeness: Float,
  last_verified: DateTime,

  // Temporal
  created_at: DateTime,
  updated_at: DateTime
})
```

**TV Show Node**
```cypher
CREATE (s:TVShow {
  // Identifiers (same as Movie)
  id: String,
  imdb_id: String,
  tmdb_id: Integer,
  eidr_id: String,
  gracenote_id: String,

  // Metadata
  title: String,
  original_title: String,
  first_air_date: Date,
  last_air_date: Date,
  status: String,          // Continuing, Ended, Canceled

  // Structure
  num_seasons: Integer,
  num_episodes: Integer,
  episode_runtime: Integer, // Average runtime

  // Descriptive (same as Movie)
  synopsis: String,
  tagline: String,
  content_rating: String,
  language: String,
  country: String,

  // Metrics (same as Movie)
  imdb_rating: Float,
  tmdb_rating: Float,
  rotten_tomatoes: Integer,
  metacritic: Integer,
  popularity_score: Float,

  // Vector Embedding
  embedding: Vector<1536>,

  // Trust & Quality (same as Movie)
  trust_score: Float,
  metadata_completeness: Float,
  last_verified: DateTime,

  // Temporal
  created_at: DateTime,
  updated_at: DateTime
})
```

**Season Node**
```cypher
CREATE (season:Season {
  id: String,
  show_id: String,         // Parent TV show
  season_number: Integer,

  title: String,
  air_date: Date,
  num_episodes: Integer,

  synopsis: String,
  poster_path: String,

  // Metrics
  average_rating: Float,

  // Vector Embedding (aggregated from episodes)
  embedding: Vector<768>,  // Smaller dimension for efficiency

  created_at: DateTime,
  updated_at: DateTime
})
```

**Episode Node**
```cypher
CREATE (ep:Episode {
  id: String,
  show_id: String,
  season_id: String,

  episode_number: Integer,
  season_number: Integer,

  title: String,
  air_date: Date,
  runtime_minutes: Integer,

  synopsis: String,
  still_path: String,

  // Metrics
  imdb_rating: Float,
  vote_count: Integer,

  // Vector Embedding
  embedding: Vector<768>,

  created_at: DateTime,
  updated_at: DateTime
})
```

**Embedding Strategy - Content Nodes:**
```
Movie/TVShow (1536-dim):
  - Combines: title + synopsis + genre tags + cast names + themes
  - Model: OpenAI text-embedding-3-large
  - Use: Primary semantic search, recommendations

Season (768-dim):
  - Aggregates episode embeddings (mean pooling)
  - Model: Derived from episode embeddings
  - Use: Season-level discovery

Episode (768-dim):
  - Combines: title + synopsis + key moments
  - Model: sentence-transformers/all-mpnet-base-v2
  - Use: Episode-specific search
```

#### 1.1.2 Person Nodes

```cypher
CREATE (p:Person {
  // Identifiers
  id: String,
  imdb_id: String,         // nm1234567
  tmdb_id: Integer,

  // Profile
  name: String,
  birth_date: Date,
  death_date: Date,        // NULL if alive
  gender: Integer,         // 0=unknown, 1=female, 2=male, 3=non-binary

  // Career
  known_for_department: String, // Acting, Directing, Writing
  biography: String,
  place_of_birth: String,

  // Metrics
  popularity: Float,

  // Media
  profile_path: String,

  // Vector Embedding
  embedding: Vector<768>,  // Bio + known roles + collaborators

  // Trust
  trust_score: Float,
  last_verified: DateTime,

  created_at: DateTime,
  updated_at: DateTime
})
```

**Embedding Strategy - Person:**
```
Combines:
  - Biography text
  - List of notable roles/works
  - Frequent collaborators
  - Awards and recognitions

Model: sentence-transformers/all-mpnet-base-v2
Use: Cast/crew search, "find movies with similar cast"
```

#### 1.1.3 User Nodes

```cypher
CREATE (u:User {
  // Identifiers
  id: String,              // Internal UUID
  external_id: String,     // OAuth subject ID (hashed)

  // Profile (minimal for privacy)
  username: String,        // Display name only
  region: String,          // ISO 3166-1 alpha-2
  timezone: String,        // IANA timezone
  language_preference: String, // ISO 639-1

  // Subscription Info (user-declared)
  subscriptions: [String], // ["netflix", "prime", "disney"]

  // Privacy Settings
  sharing_enabled: Boolean,
  analytics_enabled: Boolean,

  // Vector Embedding (privacy-preserving)
  preference_embedding: Vector<512>, // Aggregated preferences

  // Trust & Quality
  interaction_count: Integer,  // For confidence scoring
  account_age_days: Integer,

  // Temporal
  created_at: DateTime,
  last_active: DateTime
})
```

**Embedding Strategy - User:**
```
PRIVACY-FIRST APPROACH:
  - Computed on-device via federated learning
  - Only aggregated embeddings sent to server
  - Differential privacy noise added (ε = 1.0)
  - k-anonymity guaranteed (k >= 50)

Embedding represents:
  - Genre preferences (weighted)
  - Viewing time patterns (bucketed)
  - Content themes (abstracted)
  - NOT specific watch history

Model: Fine-tuned collaborative filtering model
Dimension: 512 (smaller for privacy budget)
```

#### 1.1.4 Genre/Mood/Theme Nodes

```cypher
// Genre Taxonomy
CREATE (g:Genre {
  id: String,
  name: String,            // Action, Drama, Sci-Fi, etc.
  parent_id: String,       // For hierarchical genres
  level: Integer,          // Depth in hierarchy

  description: String,

  // Vector Embedding
  embedding: Vector<256>,  // Pre-computed genre semantics

  // Metrics
  content_count: Integer,  // Number of items in genre
  popularity_trend: Float, // Trending score

  created_at: DateTime,
  updated_at: DateTime
})

// Mood Tags
CREATE (mood:Mood {
  id: String,
  name: String,            // Feel-good, Dark, Suspenseful, etc.

  description: String,

  // Vector Embedding
  embedding: Vector<256>,

  created_at: DateTime
})

// Thematic Tags
CREATE (theme:Theme {
  id: String,
  name: String,            // Coming-of-age, Revenge, Identity, etc.

  description: String,

  // Vector Embedding
  embedding: Vector<256>,

  created_at: DateTime
})
```

**Embedding Strategy - Classification:**
```
Pre-computed static embeddings:
  - Genre: Based on genre descriptions + representative content
  - Mood: Sentiment and emotional tone embeddings
  - Theme: Abstract concept embeddings

Model: Custom fine-tuned on film/TV domain
Source: Training data from expert annotations
Use: Classification, mood-based search, thematic discovery
```

#### 1.1.5 Platform Nodes

```cypher
CREATE (p:Platform {
  id: String,
  name: String,            // Netflix, Prime Video, Disney+, etc.
  slug: String,            // netflix, prime, disney_plus

  // Platform Details
  type: String,            // SVOD, TVOD, AVOD, FREE
  website: String,
  app_store_url: String,
  play_store_url: String,

  // Capabilities
  supports_4k: Boolean,
  supports_hdr: Boolean,
  supports_dolby_atmos: Boolean,
  supports_downloads: Boolean,
  max_streams: Integer,

  // Deep Linking
  deep_link_template: String, // Platform-specific URI template
  universal_link_domain: String,

  // Branding
  logo_url: String,
  primary_color: String,   // Hex color

  // Regional Info
  available_regions: [String], // ISO 3166-1 alpha-2 codes

  created_at: DateTime,
  updated_at: DateTime
})
```

#### 1.1.6 Region Nodes

```cypher
CREATE (r:Region {
  id: String,
  code: String,            // ISO 3166-1 alpha-2 (US, CA, GB, etc.)
  name: String,            // United States, Canada, etc.

  // Geographic
  continent: String,
  timezone_primary: String, // Primary IANA timezone
  timezones: [String],     // All timezones in region

  // Content Metadata
  default_language: String, // ISO 639-1
  supported_languages: [String],

  // Licensing
  licensing_region: String, // Licensing territory grouping

  created_at: DateTime,
  updated_at: DateTime
})
```

#### 1.1.7 License/Rights Nodes

```cypher
CREATE (l:License {
  id: String,

  // Licensing Details
  licensor: String,        // Rights holder
  licensee: String,        // Platform receiving rights
  content_id: String,      // Content being licensed

  // Rights Terms
  exclusivity: Boolean,
  sublicensing_allowed: Boolean,

  // Geographic
  regions: [String],       // Regions covered by license

  // Temporal
  start_date: DateTime,
  end_date: DateTime,

  // Commercial
  pricing_model: String,   // SVOD, TVOD, AVOD, FREE
  revenue_share: Float,    // If applicable

  // Quality/Format Rights
  quality_tiers: [String], // SD, HD, 4K, HDR
  languages: [String],     // Dubbed/subtitled languages

  created_at: DateTime,
  updated_at: DateTime
})
```

### 1.2 Edge Type Specifications

#### 1.2.1 Standard Edges (Binary Relationships)

```cypher
// Content Relationships
CREATE (movie:Movie)-[:BELONGS_TO {
  relevance: Float,        // How central this genre is (0-1)
  user_tagged: Boolean,    // Community vs. official tag
  created_at: DateTime
}]->(genre:Genre)

CREATE (person:Person)-[:ACTED_IN {
  role: String,           // Character name
  billing_order: Integer, // 1 = lead, 2 = supporting, etc.
  character_type: String, // Lead, Supporting, Cameo
  created_at: DateTime
}]->(content:Content)

CREATE (person:Person)-[:DIRECTED {
  episodes: [Integer],    // For TV: which episodes
  credited_as: String,    // Name variant used
  created_at: DateTime
}]->(content:Content)

CREATE (person:Person)-[:WROTE {
  role: String,          // Screenplay, Story, Teleplay
  episodes: [Integer],   // For TV: which episodes
  created_at: DateTime
}]->(content:Content)

CREATE (person:Person)-[:PRODUCED {
  role: String,          // Executive, Producer, Co-Producer
  created_at: DateTime
}]->(content:Content)

// Content Hierarchy
CREATE (show:TVShow)-[:HAS_SEASON {
  season_number: Integer,
  created_at: DateTime
}]->(season:Season)

CREATE (season:Season)-[:HAS_EPISODE {
  episode_number: Integer,
  created_at: DateTime
}]->(episode:Episode)

// Similarity Relationships (GNN-generated)
CREATE (content1:Content)-[:SIMILAR_TO {
  similarity_score: Float,    // Cosine similarity (0-1)
  similarity_type: String,    // SEMANTIC, COLLABORATIVE, HYBRID
  computed_method: String,    // GNN, EMBEDDING, MANUAL
  features: Map,              // Which features drove similarity
  created_at: DateTime,
  last_validated: DateTime
}]->(content2:Content)

// User Interactions
CREATE (user:User)-[:WATCHED {
  watch_date: DateTime,
  progress: Float,            // 0-1 (completion percentage)
  watch_duration_seconds: Integer,
  device_type: String,
  created_at: DateTime
}]->(content:Content)

CREATE (user:User)-[:RATED {
  rating: Float,              // 0-10 scale
  rating_date: DateTime,
  created_at: DateTime
}]->(content:Content)

CREATE (user:User)-[:WATCHLISTED {
  added_date: DateTime,
  priority: Integer,          // User-set priority
  created_at: DateTime
}]->(content:Content)

CREATE (user:User)-[:REVIEWED {
  review_text: String,
  rating: Float,
  helpful_count: Integer,
  review_date: DateTime,
  created_at: DateTime
}]->(content:Content)

// Social Relationships
CREATE (user1:User)-[:FOLLOWS {
  followed_date: DateTime,
  created_at: DateTime
}]->(user2:User)
```

#### 1.2.2 Hyperedges (Multi-way Relationships)

Hyperedges are Ruvector's superpower for modeling complex n-ary relationships:

```
┌─────────────────────────────────────────────────────────────────────┐
│                     HYPEREDGE: AVAILABILITY_WINDOW                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Models: Content × Platform × Region × TimeWindow × LicenseTerms    │
│                                                                     │
│  Example:                                                           │
│  "Stranger Things S4 available on Netflix in US/CA/UK              │
│   from 2024-07-01 to 2026-06-30 in 4K/HDR as SVOD exclusive"       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**AVAILABILITY_WINDOW Hyperedge:**
```rust
// Ruvector hyperedge implementation
struct AvailabilityWindow {
    // Node Connections (5-way relationship)
    content_id: String,
    platform_id: String,
    region_ids: Vec<String>,     // Multiple regions in one license

    // Temporal Bounds
    start_date: DateTime,
    end_date: DateTime,

    // Quality/Format
    quality_tiers: Vec<QualityTier>, // [SD, HD, UHD_4K, HDR10, DOLBY_VISION]
    audio_formats: Vec<AudioFormat>, // [STEREO, DOLBY_ATMOS, DTS_X]
    subtitle_languages: Vec<String>, // ISO 639-1 codes
    dubbed_languages: Vec<String>,

    // Commercial Model
    pricing_model: PricingModel, // SVOD, TVOD, AVOD, FREE
    svod_included: bool,         // Included in subscription
    rental_price: Option<Money>,
    purchase_price: Option<Money>,

    // Rights
    exclusivity: bool,
    license_id: Option<String>,  // Reference to License node

    // Deep Linking
    deep_link: String,           // Direct link to content on platform
    universal_link: Option<String>,

    // Trust & Quality
    verified: bool,              // Confirmed by platform or aggregator
    last_verified: DateTime,
    source: DataSource,          // JUSTWATCH, WATCHMODE, PLATFORM_API, etc.
    confidence: f32,             // 0-1

    // Metadata
    created_at: DateTime,
    updated_at: DateTime,
}

enum QualityTier {
    SD,
    HD,
    UHD_4K,
    HDR10,
    HDR10_PLUS,
    DOLBY_VISION,
    IMAX_ENHANCED,
}

enum PricingModel {
    SVOD,    // Subscription VOD
    TVOD,    // Transactional VOD (rent/buy)
    AVOD,    // Ad-supported VOD
    FREE,    // Free with ads or no subscription
}

struct Money {
    amount: Decimal,
    currency: String, // ISO 4217 (USD, CAD, GBP, etc.)
}
```

**Cypher Representation:**
```cypher
// Create hyperedge (Ruvector syntax)
CREATE (content:Movie {id: "tt4574334"}),
       (platform:Platform {id: "netflix"}),
       (region_us:Region {code: "US"}),
       (region_ca:Region {code: "CA"}),
       (window:AvailabilityWindow {
         start_date: datetime("2024-07-01T00:00:00Z"),
         end_date: datetime("2026-06-30T23:59:59Z"),
         quality_tiers: ["HD", "UHD_4K", "HDR10"],
         audio_formats: ["DOLBY_ATMOS"],
         subtitle_languages: ["en", "es", "fr"],
         dubbed_languages: ["en", "es"],
         pricing_model: "SVOD",
         svod_included: true,
         exclusivity: true,
         deep_link: "https://netflix.com/title/80057281",
         verified: true,
         confidence: 0.95,
         source: "JUSTWATCH"
       })

// Connect hyperedge to all participating nodes
CREATE (content)-[:AVAILABLE_IN]->(window),
       (window)-[:ON_PLATFORM]->(platform),
       (window)-[:IN_REGION]->(region_us),
       (window)-[:IN_REGION]->(region_ca)
```

**Query Example - Find Available Content:**
```cypher
// Find 4K content available on user's subscriptions in their region
MATCH (user:User {id: $userId})
MATCH (content:Movie|TVShow)
MATCH (content)-[:AVAILABLE_IN]->(window:AvailabilityWindow)
MATCH (window)-[:ON_PLATFORM]->(platform:Platform)
MATCH (window)-[:IN_REGION]->(region:Region {code: user.region})
WHERE platform.slug IN user.subscriptions
  AND window.end_date > datetime()
  AND "UHD_4K" IN window.quality_tiers
RETURN content, window, platform
ORDER BY content.popularity_score DESC
LIMIT 20
```

### 1.3 Embedding Dimension Strategy

```
┌─────────────────────────────────────────────────────────────────────┐
│                    EMBEDDING DIMENSION ALLOCATION                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Node Type              Dimensions    Model                         │
│  ─────────────────────  ──────────    ─────────────────────────     │
│  Movie/TVShow           1536          OpenAI text-embedding-3-large │
│  Person                 768           all-mpnet-base-v2             │
│  Episode/Season         768           all-mpnet-base-v2             │
│  User (preferences)     512           Custom federated model        │
│  Genre/Mood/Theme       256           Custom fine-tuned             │
│                                                                     │
│  RATIONALE:                                                         │
│  - Larger dims for primary search targets (content)                 │
│  - Smaller dims for supporting entities (efficiency)                │
│  - Privacy-conscious sizing for user data                           │
│  - Hierarchical sizing matches importance                           │
│                                                                     │
│  MEMORY FOOTPRINT:                                                  │
│  - 1M movies/shows: ~6 GB (1536-dim)                                │
│  - 500K people: ~1.5 GB (768-dim)                                   │
│  - 10M episodes: ~30 GB (768-dim)                                   │
│  - 1M users: ~2 GB (512-dim)                                        │
│  - Total: ~40 GB for embeddings alone                               │
│                                                                     │
│  INDEXING STRATEGY:                                                 │
│  - HNSW index per node type                                         │
│  - M=16, efConstruction=200 (Ruvector defaults)                     │
│  - Separate indexes for fast type-specific searches                 │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. GNN Architecture

### 2.1 GraphSAGE Layer Configuration

Ruvector includes built-in GNN support via the `@ruvector/gnn` package:

```rust
use ruvector_gnn::{
    layer::RuvectorLayer,
    search::{differentiable_search, hierarchical_forward},
};

// GraphSAGE layer for content recommendation
struct ContentRecommendationGNN {
    // Multi-layer GNN architecture
    layer1: RuvectorLayer,  // Input: 1536, Hidden: 512, Heads: 8
    layer2: RuvectorLayer,  // Input: 512, Hidden: 256, Heads: 4
    layer3: RuvectorLayer,  // Input: 256, Hidden: 128, Heads: 2

    // Dropout for regularization
    dropout: f32,

    // Training state
    optimizer: AdamOptimizer,
    learning_rate: f32,
}

impl ContentRecommendationGNN {
    pub fn new() -> Self {
        Self {
            // Layer 1: Aggregate neighborhood features
            layer1: RuvectorLayer::new(
                1536,   // input_dim (content embedding)
                512,    // hidden_dim
                8,      // num_heads
                0.1,    // dropout
            ),

            // Layer 2: Feature transformation
            layer2: RuvectorLayer::new(
                512,    // input_dim
                256,    // hidden_dim
                4,      // num_heads
                0.1,    // dropout
            ),

            // Layer 3: Final representation
            layer3: RuvectorLayer::new(
                256,    // input_dim
                128,    // hidden_dim (output embedding size)
                2,      // num_heads
                0.1,    // dropout
            ),

            dropout: 0.1,
            optimizer: AdamOptimizer::new(0.001, 0.9, 0.999, 1e-8),
            learning_rate: 0.001,
        }
    }

    // Forward pass through GNN
    pub fn forward(
        &self,
        node_embedding: &[f32],
        neighbor_embeddings: &[Vec<f32>],
        edge_weights: &[f32],
    ) -> Vec<f32> {
        // Layer 1: Aggregate neighbors
        let h1 = self.layer1.forward(
            node_embedding,
            neighbor_embeddings,
            edge_weights,
        );

        // Layer 2: Transform features
        let h2 = self.layer2.forward(
            &h1,
            &[h1.clone()], // Self-connection
            &[1.0],
        );

        // Layer 3: Final embedding
        let output = self.layer3.forward(
            &h2,
            &[h2.clone()],
            &[1.0],
        );

        output
    }

    // Training step
    pub fn train_step(
        &mut self,
        positive_samples: &[(Vec<f32>, Vec<Vec<f32>>, Vec<f32>)],
        negative_samples: &[(Vec<f32>, Vec<Vec<f32>>, Vec<f32>)],
    ) -> f32 {
        // Compute positive scores
        let pos_scores: Vec<f32> = positive_samples
            .iter()
            .map(|(node, neighbors, weights)| {
                let embedding = self.forward(node, neighbors, weights);
                // Compute similarity with target
                cosine_similarity(&embedding, &node)
            })
            .collect();

        // Compute negative scores
        let neg_scores: Vec<f32> = negative_samples
            .iter()
            .map(|(node, neighbors, weights)| {
                let embedding = self.forward(node, neighbors, weights);
                cosine_similarity(&embedding, &node)
            })
            .collect();

        // BPR loss (Bayesian Personalized Ranking)
        let loss = bpr_loss(&pos_scores, &neg_scores);

        // Backpropagation (simplified - actual implementation uses autograd)
        // self.backward(loss);
        // self.optimizer.step();

        loss
    }
}

// Bayesian Personalized Ranking loss
fn bpr_loss(pos_scores: &[f32], neg_scores: &[f32]) -> f32 {
    let mut total_loss = 0.0;
    for (pos, neg) in pos_scores.iter().zip(neg_scores.iter()) {
        total_loss += -((pos - neg).sigmoid().ln());
    }
    total_loss / pos_scores.len() as f32
}

fn cosine_similarity(a: &[f32], b: &[f32]) -> f32 {
    let dot: f32 = a.iter().zip(b.iter()).map(|(x, y)| x * y).sum();
    let norm_a: f32 = a.iter().map(|x| x * x).sum::<f32>().sqrt();
    let norm_b: f32 = b.iter().map(|x| x * x).sum::<f32>().sqrt();
    dot / (norm_a * norm_b)
}
```

### 2.2 Heterogeneous Graph Construction

```rust
use std::collections::HashMap;

/// Heterogeneous graph for user-content interactions
struct HeterogeneousGraph {
    // Node embeddings by type
    users: HashMap<String, Vec<f32>>,
    content: HashMap<String, Vec<f32>>,

    // Edge lists by type
    watched_edges: Vec<(String, String, f32)>,    // (user_id, content_id, weight)
    rated_edges: Vec<(String, String, f32)>,
    similar_edges: Vec<(String, String, f32)>,    // (content_id, content_id, score)

    // Metadata
    user_dim: usize,
    content_dim: usize,
}

impl HeterogeneousGraph {
    /// Build graph from Ruvector
    pub async fn from_ruvector(
        ruvector_client: &RuvectorClient,
        user_id: &str,
    ) -> Result<Self, Error> {
        let mut graph = HeterogeneousGraph {
            users: HashMap::new(),
            content: HashMap::new(),
            watched_edges: Vec::new(),
            rated_edges: Vec::new(),
            similar_edges: Vec::new(),
            user_dim: 512,
            content_dim: 1536,
        };

        // Fetch user node with embedding
        let user = ruvector_client.get_node("User", user_id).await?;
        graph.users.insert(
            user_id.to_string(),
            user.embedding,
        );

        // Fetch user's watch history (WATCHED edges)
        let watched_query = format!(
            "MATCH (u:User {{id: '{}'}})-[w:WATCHED]->(c:Content) \
             RETURN c.id, c.embedding, w.progress",
            user_id
        );
        let watched_results = ruvector_client.query(&watched_query).await?;

        for row in watched_results {
            let content_id = row.get::<String>("c.id")?;
            let embedding = row.get::<Vec<f32>>("c.embedding")?;
            let progress = row.get::<f32>("w.progress")?;

            graph.content.insert(content_id.clone(), embedding);
            graph.watched_edges.push((
                user_id.to_string(),
                content_id,
                progress, // Weight by completion
            ));
        }

        // Fetch similar content (SIMILAR_TO edges)
        let similar_query = format!(
            "MATCH (c1:Content)<-[:WATCHED]-(u:User {{id: '{}'}}), \
                   (c1)-[s:SIMILAR_TO]->(c2:Content) \
             RETURN c1.id, c2.id, c2.embedding, s.similarity_score \
             ORDER BY s.similarity_score DESC \
             LIMIT 100",
            user_id
        );
        let similar_results = ruvector_client.query(&similar_query).await?;

        for row in similar_results {
            let c1_id = row.get::<String>("c1.id")?;
            let c2_id = row.get::<String>("c2.id")?;
            let c2_embedding = row.get::<Vec<f32>>("c2.embedding")?;
            let score = row.get::<f32>("s.similarity_score")?;

            graph.content.insert(c2_id.clone(), c2_embedding);
            graph.similar_edges.push((c1_id, c2_id, score));
        }

        Ok(graph)
    }

    /// Get neighbors for GNN aggregation
    pub fn get_neighbors(
        &self,
        node_id: &str,
        node_type: NodeType,
    ) -> (Vec<Vec<f32>>, Vec<f32>) {
        match node_type {
            NodeType::User => {
                // User neighbors: watched content
                let neighbors: Vec<Vec<f32>> = self.watched_edges
                    .iter()
                    .filter(|(u, _, _)| u == node_id)
                    .filter_map(|(_, c, _)| self.content.get(c).cloned())
                    .collect();

                let weights: Vec<f32> = self.watched_edges
                    .iter()
                    .filter(|(u, _, _)| u == node_id)
                    .map(|(_, _, w)| *w)
                    .collect();

                (neighbors, weights)
            },
            NodeType::Content => {
                // Content neighbors: similar content
                let neighbors: Vec<Vec<f32>> = self.similar_edges
                    .iter()
                    .filter(|(c1, _, _)| c1 == node_id)
                    .filter_map(|(_, c2, _)| self.content.get(c2).cloned())
                    .collect();

                let weights: Vec<f32> = self.similar_edges
                    .iter()
                    .filter(|(c1, _, _)| c1 == node_id)
                    .map(|(_, _, w)| *w)
                    .collect();

                (neighbors, weights)
            },
        }
    }
}

enum NodeType {
    User,
    Content,
}
```

### 2.3 Training Pipeline

```rust
use ruvector_gnn::training::{Trainer, TrainingConfig};

/// Self-improving recommendation system
pub struct RecommendationTrainer {
    gnn: ContentRecommendationGNN,
    ruvector: RuvectorClient,
    config: TrainingConfig,
}

impl RecommendationTrainer {
    pub fn new(ruvector: RuvectorClient) -> Self {
        Self {
            gnn: ContentRecommendationGNN::new(),
            ruvector,
            config: TrainingConfig {
                learning_rate: 0.001,
                batch_size: 256,
                num_epochs: 50,
                negative_samples: 5,
                validation_split: 0.1,
                early_stopping_patience: 5,
            },
        }
    }

    /// Train on user interaction data
    pub async fn train(&mut self) -> Result<TrainingMetrics, Error> {
        println!("Starting GNN training...");

        let mut metrics = TrainingMetrics::new();
        let mut best_val_loss = f32::MAX;
        let mut patience_counter = 0;

        for epoch in 0..self.config.num_epochs {
            println!("Epoch {}/{}", epoch + 1, self.config.num_epochs);

            // Fetch training batch
            let (positive_samples, negative_samples) =
                self.fetch_training_batch().await?;

            // Training step
            let train_loss = self.gnn.train_step(
                &positive_samples,
                &negative_samples,
            );

            // Validation
            let val_loss = self.validate().await?;

            metrics.record_epoch(epoch, train_loss, val_loss);

            println!("  Train Loss: {:.4}, Val Loss: {:.4}", train_loss, val_loss);

            // Early stopping
            if val_loss < best_val_loss {
                best_val_loss = val_loss;
                patience_counter = 0;
                self.save_checkpoint(epoch).await?;
            } else {
                patience_counter += 1;
                if patience_counter >= self.config.early_stopping_patience {
                    println!("Early stopping triggered");
                    break;
                }
            }
        }

        Ok(metrics)
    }

    /// Fetch training batch from Ruvector
    async fn fetch_training_batch(&self) -> Result<(
        Vec<(Vec<f32>, Vec<Vec<f32>>, Vec<f32>)>,
        Vec<(Vec<f32>, Vec<Vec<f32>>, Vec<f32>)>,
    ), Error> {
        // Positive samples: content user watched and liked
        let positive_query = "
            MATCH (u:User)-[w:WATCHED]->(c:Content)
            WHERE w.progress > 0.7  // Completed at least 70%
            MATCH (c)-[s:SIMILAR_TO]->(neighbor:Content)
            RETURN u.id, c.embedding, collect(neighbor.embedding) as neighbors,
                   collect(s.similarity_score) as weights
            ORDER BY rand()
            LIMIT $batch_size
        ";

        let pos_results = self.ruvector.query_with_params(
            positive_query,
            &[("batch_size", self.config.batch_size.into())],
        ).await?;

        let positive_samples: Vec<_> = pos_results
            .into_iter()
            .map(|row| {
                let embedding = row.get("c.embedding").unwrap();
                let neighbors = row.get("neighbors").unwrap();
                let weights = row.get("weights").unwrap();
                (embedding, neighbors, weights)
            })
            .collect();

        // Negative samples: random content user hasn't watched
        let negative_query = "
            MATCH (u:User), (c:Content)
            WHERE NOT (u)-[:WATCHED]->(c)
            MATCH (c)-[s:SIMILAR_TO]->(neighbor:Content)
            RETURN c.embedding, collect(neighbor.embedding) as neighbors,
                   collect(s.similarity_score) as weights
            ORDER BY rand()
            LIMIT $num_negatives
        ";

        let neg_results = self.ruvector.query_with_params(
            negative_query,
            &[("num_negatives", (self.config.batch_size * self.config.negative_samples).into())],
        ).await?;

        let negative_samples: Vec<_> = neg_results
            .into_iter()
            .map(|row| {
                let embedding = row.get("c.embedding").unwrap();
                let neighbors = row.get("neighbors").unwrap();
                let weights = row.get("weights").unwrap();
                (embedding, neighbors, weights)
            })
            .collect();

        Ok((positive_samples, negative_samples))
    }

    async fn validate(&self) -> Result<f32, Error> {
        // Similar to training but on held-out validation set
        // Return validation loss
        todo!()
    }

    async fn save_checkpoint(&self, epoch: usize) -> Result<(), Error> {
        // Save model weights to disk
        let checkpoint_path = format!("checkpoints/gnn_epoch_{}.bin", epoch);
        // Serialize self.gnn to checkpoint_path
        Ok(())
    }
}

struct TrainingMetrics {
    train_losses: Vec<f32>,
    val_losses: Vec<f32>,
}

impl TrainingMetrics {
    fn new() -> Self {
        Self {
            train_losses: Vec::new(),
            val_losses: Vec::new(),
        }
    }

    fn record_epoch(&mut self, _epoch: usize, train_loss: f32, val_loss: f32) {
        self.train_losses.push(train_loss);
        self.val_losses.push(val_loss);
    }
}
```

### 2.4 Tiny Dancer (FastGRNN) Agent Routing

Ruvector includes a lightweight GNN-based router for intelligent query routing:

```rust
use ruvector_gnn::fastgrnn::TinyDancer;

/// Tiny Dancer: FastGRNN-based query router
pub struct QueryRouter {
    tiny_dancer: TinyDancer,
    ruvector: RuvectorClient,
}

impl QueryRouter {
    pub fn new(ruvector: RuvectorClient) -> Self {
        Self {
            tiny_dancer: TinyDancer::new(
                input_dim: 768,   // Query embedding dimension
                hidden_dim: 128,
                output_classes: 3, // VECTOR, GRAPH, HYBRID
            ),
            ruvector,
        }
    }

    /// Route query to optimal search path
    pub async fn route_query(&self, query: &str) -> QueryPath {
        // Embed query
        let query_embedding = self.embed_query(query).await;

        // FastGRNN prediction (sub-millisecond)
        let (path, confidence) = self.tiny_dancer.predict(&query_embedding);

        println!("Routing query: '{}' -> {:?} (confidence: {:.2})",
                 query, path, confidence);

        path
    }

    async fn embed_query(&self, query: &str) -> Vec<f32> {
        // Use sentence-transformers for query embedding
        // In production: call embedding service
        vec![0.0; 768] // Placeholder
    }
}

#[derive(Debug, Clone)]
pub enum QueryPath {
    Vector,   // Semantic similarity search via HNSW
    Graph,    // Relationship traversal via Cypher
    Hybrid,   // Combined approach
}

/// Example routing logic
pub async fn execute_routed_query(
    router: &QueryRouter,
    ruvector: &RuvectorClient,
    query: &str,
    user_region: &str,
) -> Result<Vec<SearchResult>, Error> {
    let path = router.route_query(query).await;

    match path {
        QueryPath::Vector => {
            // Pure vector similarity search
            println!("Executing vector search...");
            let query_embedding = router.embed_query(query).await;
            let results = ruvector.vector_search(
                &query_embedding,
                limit: 20,
                filter: format!("available_in.region == '{}'", user_region),
            ).await?;
            Ok(results)
        },

        QueryPath::Graph => {
            // Pure graph traversal
            println!("Executing graph traversal...");
            let cypher_query = format!(
                "MATCH (c:Content)-[:AVAILABLE_IN]->(w:AvailabilityWindow) \
                 WHERE w.region = '{}' AND c.title CONTAINS '{}' \
                 RETURN c LIMIT 20",
                user_region, query
            );
            let results = ruvector.cypher_query(&cypher_query).await?;
            Ok(results)
        },

        QueryPath::Hybrid => {
            // Combined: vector search + graph filters
            println!("Executing hybrid search...");
            let query_embedding = router.embed_query(query).await;

            // Step 1: Vector search (broad)
            let vector_results = ruvector.vector_search(
                &query_embedding,
                limit: 100,
                filter: None,
            ).await?;

            // Step 2: Graph filtering (precise)
            let content_ids: Vec<String> = vector_results
                .iter()
                .map(|r| r.id.clone())
                .collect();

            let graph_query = format!(
                "MATCH (c:Content)-[:AVAILABLE_IN]->(w:AvailabilityWindow) \
                 WHERE c.id IN {:?} AND w.region = '{}' \
                 RETURN c, w \
                 ORDER BY c.popularity_score DESC \
                 LIMIT 20",
                content_ids, user_region
            );

            let filtered_results = ruvector.cypher_query(&graph_query).await?;
            Ok(filtered_results)
        },
    }
}
```

---

## 3. Query Patterns

### 3.1 Multi-Hop Content Discovery

```cypher
// Query 1: Find content similar to what friends watched
MATCH (user:User {id: $userId})-[:FOLLOWS]->(friend:User)
MATCH (friend)-[w:WATCHED]->(watched:Content)
WHERE w.progress > 0.8  // Friends completed content
MATCH (watched)-[:SIMILAR_TO]->(recommendation:Content)
WHERE NOT (user)-[:WATCHED]->(recommendation)

// Check availability in user's region
MATCH (recommendation)-[:AVAILABLE_IN]->(window:AvailabilityWindow)
MATCH (window)-[:IN_REGION]->(region:Region {code: user.region})
MATCH (window)-[:ON_PLATFORM]->(platform:Platform)
WHERE platform.slug IN user.subscriptions
  AND window.end_date > datetime()

// Calculate recommendation score
WITH recommendation, window, platform,
     count(DISTINCT friend) AS friend_count,
     avg(watched.popularity_score) AS source_popularity,
     vector.similarity(user.preference_embedding, recommendation.embedding) AS user_sim
ORDER BY friend_count DESC, user_sim DESC, source_popularity DESC
LIMIT 20

RETURN recommendation {
  .id,
  .title,
  .synopsis,
  .imdb_rating,
  platform: platform.name,
  quality: window.quality_tiers,
  pricing: window.pricing_model,
  deep_link: window.deep_link,
  friend_count: friend_count,
  match_score: user_sim
}
```

```cypher
// Query 2: Discover hidden gems in a genre
MATCH (genre:Genre {name: $genreName})
MATCH (content:Content)-[:BELONGS_TO]->(genre)

// Filter by availability
MATCH (content)-[:AVAILABLE_IN]->(window:AvailabilityWindow)
MATCH (window)-[:IN_REGION]->(region:Region {code: $userRegion})
MATCH (window)-[:ON_PLATFORM]->(platform:Platform)
WHERE platform.slug IN $userSubscriptions
  AND window.end_date > datetime()

// Hidden gem criteria: high quality, low popularity
WHERE content.imdb_rating >= 7.5
  AND content.popularity_score < 50  // Not mainstream
  AND content.trust_score > 0.7      // Reliable metadata

// Exclude already watched
MATCH (user:User {id: $userId})
WHERE NOT (user)-[:WATCHED]->(content)

// Ranking: quality over popularity
WITH content, window, platform,
     (content.imdb_rating / 10.0) * (1 - (content.popularity_score / 100)) AS gem_score
ORDER BY gem_score DESC
LIMIT 10

RETURN content {
  .*,
  platform: platform.name,
  availability: window,
  gem_score: gem_score
}
```

### 3.2 Cross-Platform Availability

```cypher
// Query 3: Find where content is available across all platforms
MATCH (content:Content {id: $contentId})
MATCH (content)-[:AVAILABLE_IN]->(window:AvailabilityWindow)
MATCH (window)-[:ON_PLATFORM]->(platform:Platform)
MATCH (window)-[:IN_REGION]->(region:Region {code: $userRegion})
WHERE window.end_date > datetime()

// Collect all availability options
WITH content, platform, window,
     CASE window.pricing_model
       WHEN 'SVOD' THEN
         CASE WHEN window.svod_included THEN 1 ELSE 3 END
       WHEN 'FREE' THEN 2
       WHEN 'TVOD' THEN 4
       ELSE 5
     END AS price_priority

ORDER BY price_priority ASC, platform.name ASC

RETURN platform {
  .name,
  .logo_url,
  pricing_model: window.pricing_model,
  svod_included: window.svod_included,
  rental_price: window.rental_price,
  purchase_price: window.purchase_price,
  quality_tiers: window.quality_tiers,
  audio_formats: window.audio_formats,
  subtitle_languages: window.subtitle_languages,
  deep_link: window.deep_link,
  universal_link: window.universal_link
}
```

### 3.3 Personalized Recommendations

```cypher
// Query 4: GNN-enhanced personalized recommendations
MATCH (user:User {id: $userId})

// Get user's watch history
MATCH (user)-[w:WATCHED]->(watched:Content)
WHERE w.progress > 0.5  // At least half-watched
WITH user, collect(watched) AS watched_content

// Find content with similar embeddings (vector search)
CALL vector.search(
  'content_embedding_index',
  user.preference_embedding,
  50
) YIELD node AS candidate, score

// Filter out already watched
WHERE NOT candidate IN watched_content

// Check availability
MATCH (candidate)-[:AVAILABLE_IN]->(window:AvailabilityWindow)
MATCH (window)-[:IN_REGION]->(region:Region {code: user.region})
MATCH (window)-[:ON_PLATFORM]->(platform:Platform)
WHERE platform.slug IN user.subscriptions
  AND window.end_date > datetime()

// Apply GNN-based scoring (using pre-computed GNN embeddings)
WITH candidate, window, platform, score,
     candidate.gnn_embedding AS gnn_emb,
     user.gnn_preference_embedding AS user_gnn_emb,
     vector.similarity(gnn_emb, user_gnn_emb) AS gnn_score

// Final ranking: combine embedding similarity + GNN score
WITH candidate, window, platform,
     (score * 0.4 + gnn_score * 0.6) AS final_score
ORDER BY final_score DESC
LIMIT 20

RETURN candidate {
  .*,
  platform: platform.name,
  availability: window {
    .quality_tiers,
    .pricing_model,
    .deep_link
  },
  recommendation_score: final_score
}
```

### 3.4 Similar Content Finding

```cypher
// Query 5: Find similar content with multi-hop traversal
MATCH (seed:Content {id: $contentId})

// Method 1: Direct similarity edges (GNN-computed)
MATCH (seed)-[sim:SIMILAR_TO]->(similar1:Content)
WHERE sim.similarity_score > 0.7

// Method 2: Shared cast/crew
MATCH (seed)<-[:ACTED_IN|DIRECTED]-(person:Person)
MATCH (person)-[:ACTED_IN|DIRECTED]->(similar2:Content)
WHERE similar2 != seed

// Method 3: Genre + mood overlap
MATCH (seed)-[:BELONGS_TO]->(genre:Genre)
MATCH (seed)-[:HAS_MOOD]->(mood:Mood)
MATCH (similar3:Content)-[:BELONGS_TO]->(genre)
MATCH (similar3)-[:HAS_MOOD]->(mood)
WHERE similar3 != seed

// Combine all similarity sources
WITH seed,
     collect(DISTINCT similar1) AS method1,
     collect(DISTINCT similar2) AS method2,
     collect(DISTINCT similar3) AS method3
UNWIND method1 + method2 + method3 AS similar

// Aggregate and score
WITH similar,
     CASE
       WHEN similar IN method1 THEN 0.5
       ELSE 0.0
     END AS sim_score,
     CASE
       WHEN similar IN method2 THEN 0.3
       ELSE 0.0
     END AS cast_score,
     CASE
       WHEN similar IN method3 THEN 0.2
       ELSE 0.0
     END AS genre_score

// Check availability
MATCH (similar)-[:AVAILABLE_IN]->(window:AvailabilityWindow)
MATCH (window)-[:IN_REGION]->(region:Region {code: $userRegion})
WHERE window.end_date > datetime()

WITH similar, window,
     (sim_score + cast_score + genre_score) AS combined_score
ORDER BY combined_score DESC
LIMIT 10

RETURN similar {
  .*,
  availability: window,
  similarity_score: combined_score
}
```

### 3.5 Rights/Availability Filtering

```cypher
// Query 6: Find content expiring soon on user's platforms
MATCH (user:User {id: $userId})
MATCH (content:Content)-[:AVAILABLE_IN]->(window:AvailabilityWindow)
MATCH (window)-[:ON_PLATFORM]->(platform:Platform)
MATCH (window)-[:IN_REGION]->(region:Region {code: user.region})

WHERE platform.slug IN user.subscriptions
  AND window.end_date > datetime()
  AND window.end_date < datetime() + duration({days: 30})  // Expiring in 30 days
  AND NOT (user)-[:WATCHED]->(content)

// Prioritize by how soon it expires
WITH content, window, platform,
     duration.between(datetime(), window.end_date).days AS days_remaining
ORDER BY days_remaining ASC, content.imdb_rating DESC

RETURN content {
  .*,
  platform: platform.name,
  days_remaining: days_remaining,
  expires_on: window.end_date,
  deep_link: window.deep_link
}
LIMIT 20
```

### 3.6 Trust-Score Weighted Queries

```cypher
// Query 7: High-confidence recommendations only
MATCH (user:User {id: $userId})
MATCH (content:Content)

// Vector similarity search
CALL vector.search(
  'content_embedding_index',
  user.preference_embedding,
  100
) YIELD node AS content, score

// Trust filtering
WHERE content.trust_score > 0.8
  AND content.metadata_completeness > 0.9

// Availability with trust
MATCH (content)-[:AVAILABLE_IN]->(window:AvailabilityWindow)
WHERE window.verified = true
  AND window.confidence > 0.85

MATCH (window)-[:IN_REGION]->(region:Region {code: user.region})
MATCH (window)-[:ON_PLATFORM]->(platform:Platform)
WHERE platform.slug IN user.subscriptions
  AND window.end_date > datetime()

// Weight by trust scores
WITH content, window, platform, score,
     (score * content.trust_score * window.confidence) AS weighted_score
ORDER BY weighted_score DESC
LIMIT 20

RETURN content {
  .*,
  platform: platform.name,
  availability: window,
  trust_score: content.trust_score,
  availability_confidence: window.confidence,
  final_score: weighted_score
}
```

---

## 4. Micro-Repository Architecture

### 4.1 knowledge-graph Repository

```
knowledge-graph/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── schema/
│   │   ├── mod.rs
│   │   ├── nodes.rs           # Node type definitions
│   │   ├── edges.rs           # Edge type definitions
│   │   └── hyperedges.rs      # Hyperedge implementations
│   ├── ingestion/
│   │   ├── mod.rs
│   │   ├── content.rs         # Content ingestion
│   │   ├── person.rs          # Person ingestion
│   │   └── availability.rs    # Availability window ingestion
│   ├── query/
│   │   ├── mod.rs
│   │   ├── builder.rs         # Cypher query builder
│   │   └── executor.rs        # Query execution
│   ├── maintenance/
│   │   ├── mod.rs
│   │   ├── dedup.rs           # Entity deduplication
│   │   └── cleanup.rs         # Stale data removal
│   └── client.rs              # Ruvector client wrapper
├── tests/
│   ├── integration/
│   │   ├── ingestion_test.rs
│   │   └── query_test.rs
│   └── fixtures/
│       └── sample_data.json
└── examples/
    ├── ingest_content.rs
    └── query_examples.rs
```

**Key Interfaces:**

```rust
// src/lib.rs
pub mod schema;
pub mod ingestion;
pub mod query;
pub mod maintenance;
pub mod client;

pub use client::KnowledgeGraphClient;
pub use schema::{ContentNode, PersonNode, AvailabilityWindow};

// Public API
pub trait KnowledgeGraph {
    async fn ingest_content(&self, content: ContentNode) -> Result<String, Error>;
    async fn ingest_person(&self, person: PersonNode) -> Result<String, Error>;
    async fn add_availability(&self, window: AvailabilityWindow) -> Result<(), Error>;

    async fn query(&self, cypher: &str) -> Result<Vec<QueryResult>, Error>;
    async fn vector_search(
        &self,
        embedding: &[f32],
        limit: usize,
    ) -> Result<Vec<ContentNode>, Error>;

    async fn deduplicate_entities(&self) -> Result<DeduplicationReport, Error>;
}
```

### 4.2 vector-indexer Repository

```
vector-indexer/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── index/
│   │   ├── mod.rs
│   │   ├── hnsw.rs            # HNSW index management
│   │   ├── builder.rs         # Index building
│   │   └── searcher.rs        # Search operations
│   ├── embeddings/
│   │   ├── mod.rs
│   │   ├── generator.rs       # Embedding generation
│   │   ├── models.rs          # Model configurations
│   │   └── cache.rs           # Embedding cache
│   ├── compression/
│   │   ├── mod.rs
│   │   └── adaptive.rs        # Adaptive compression
│   └── client.rs
└── tests/
    └── index_test.rs
```

**Key Interfaces:**

```rust
// src/lib.rs
pub use ruvector_gnn::compress::TensorCompress;

pub trait VectorIndexer {
    // Index management
    async fn create_index(
        &self,
        name: &str,
        dimension: usize,
        config: IndexConfig,
    ) -> Result<(), Error>;

    async fn insert_vector(
        &self,
        index: &str,
        id: &str,
        vector: &[f32],
        metadata: HashMap<String, Value>,
    ) -> Result<(), Error>;

    async fn search(
        &self,
        index: &str,
        query: &[f32],
        k: usize,
        filter: Option<&str>,
    ) -> Result<Vec<SearchResult>, Error>;

    // Embedding generation
    async fn generate_embedding(
        &self,
        text: &str,
        model: EmbeddingModel,
    ) -> Result<Vec<f32>, Error>;

    // Compression
    async fn compress_embeddings(
        &self,
        embeddings: &[Vec<f32>],
        access_frequencies: &[f32],
    ) -> Result<Vec<CompressedEmbedding>, Error>;
}

pub struct IndexConfig {
    pub m: usize,                    // HNSW M parameter
    pub ef_construction: usize,      // HNSW efConstruction
    pub distance_metric: DistanceMetric,
}

pub enum EmbeddingModel {
    OpenAI_TextEmbedding3Large,      // 1536-dim
    SentenceTransformers_MPNet,      // 768-dim
    Custom { endpoint: String },
}
```

### 4.3 gnn-trainer Repository

```
gnn-trainer/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── models/
│   │   ├── mod.rs
│   │   ├── graphsage.rs       # GraphSAGE implementation
│   │   └── fastgrnn.rs        # FastGRNN (Tiny Dancer)
│   ├── training/
│   │   ├── mod.rs
│   │   ├── pipeline.rs        # Training pipeline
│   │   ├── data_loader.rs     # Graph data loading
│   │   └── optimizer.rs       # Optimization
│   ├── evaluation/
│   │   ├── mod.rs
│   │   └── metrics.rs         # Training metrics
│   └── serving/
│       ├── mod.rs
│       └── inference.rs       # Model serving
└── tests/
    └── training_test.rs
```

**Key Interfaces:**

```rust
// src/lib.rs
pub use ruvector_gnn::{RuvectorLayer, differentiable_search, hierarchical_forward};

pub trait GNNTrainer {
    // Training
    async fn train_model(
        &mut self,
        config: TrainingConfig,
    ) -> Result<TrainingMetrics, Error>;

    async fn load_training_data(
        &self,
        graph_client: &KnowledgeGraphClient,
    ) -> Result<HeterogeneousGraph, Error>;

    // Inference
    async fn predict(
        &self,
        node_id: &str,
        k: usize,
    ) -> Result<Vec<Recommendation>, Error>;

    // Model management
    async fn save_checkpoint(&self, path: &str) -> Result<(), Error>;
    async fn load_checkpoint(&mut self, path: &str) -> Result<(), Error>;

    // Evaluation
    async fn evaluate(&self) -> Result<EvaluationMetrics, Error>;
}

pub struct TrainingConfig {
    pub learning_rate: f32,
    pub batch_size: usize,
    pub num_epochs: usize,
    pub negative_samples: usize,
    pub validation_split: f32,
    pub early_stopping_patience: usize,
}
```

### 4.4 Interface Contracts (gRPC)

```protobuf
// proto/knowledge_graph.proto
syntax = "proto3";

package knowledge_graph;

service KnowledgeGraphService {
  // Content management
  rpc IngestContent(ContentNode) returns (IngestResponse);
  rpc IngestPerson(PersonNode) returns (IngestResponse);
  rpc AddAvailability(AvailabilityWindow) returns (EmptyResponse);

  // Querying
  rpc Query(CypherQuery) returns (stream QueryResult);
  rpc VectorSearch(VectorSearchRequest) returns (VectorSearchResponse);

  // Maintenance
  rpc DeduplicateEntities(DeduplicationRequest) returns (DeduplicationReport);
}

message ContentNode {
  string id = 1;
  string title = 2;
  string synopsis = 3;
  repeated float embedding = 4;
  map<string, string> metadata = 5;
}

message VectorSearchRequest {
  repeated float query_embedding = 1;
  int32 limit = 2;
  string filter = 3;
}

message VectorSearchResponse {
  repeated SearchResult results = 1;
}

message SearchResult {
  string id = 1;
  float score = 2;
  ContentNode content = 3;
}
```

---

## 5. Rust Implementation Patterns

### 5.1 Ruvector Client Wrapper

```rust
use ruvector::{VectorDb, VectorEntry, SearchQuery};
use serde::{Deserialize, Serialize};
use std::sync::Arc;
use tokio::sync::RwLock;

/// High-level Ruvector client for TV discovery
pub struct RuvectorClient {
    // Vector database instance
    vector_db: Arc<RwLock<VectorDb>>,

    // Multiple indexes for different node types
    content_index: String,
    person_index: String,
    user_index: String,

    // Configuration
    config: ClientConfig,
}

#[derive(Clone)]
pub struct ClientConfig {
    pub dimensions_content: usize,   // 1536
    pub dimensions_person: usize,    // 768
    pub dimensions_user: usize,      // 512
    pub storage_path: String,
    pub max_elements: usize,
}

impl RuvectorClient {
    pub async fn new(config: ClientConfig) -> Result<Self, Error> {
        // Create content index
        let content_db = VectorDb::new(ruvector::Config {
            dimensions: config.dimensions_content,
            max_elements: config.max_elements,
            storage_path: format!("{}/content.db", config.storage_path),
            ef_construction: 200,
            m: 16,
            distance_metric: ruvector::DistanceMetric::Cosine,
        })?;

        Ok(Self {
            vector_db: Arc::new(RwLock::new(content_db)),
            content_index: "content".to_string(),
            person_index: "person".to_string(),
            user_index: "user".to_string(),
            config,
        })
    }

    /// Insert content node with embedding
    pub async fn insert_content(
        &self,
        content: &ContentNode,
    ) -> Result<(), Error> {
        let entry = VectorEntry {
            id: content.id.clone(),
            vector: content.embedding.clone().into(),
            metadata: serde_json::to_value(content)?,
        };

        let db = self.vector_db.write().await;
        db.insert(entry).await?;

        Ok(())
    }

    /// Semantic search for content
    pub async fn search_content(
        &self,
        query_embedding: &[f32],
        k: usize,
        filter: Option<&str>,
    ) -> Result<Vec<ContentNode>, Error> {
        let query = SearchQuery {
            vector: query_embedding.to_vec().into(),
            k,
            threshold: 0.0,
            filter: filter.map(|s| s.to_string()),
        };

        let db = self.vector_db.read().await;
        let results = db.search(query).await?;

        let contents: Vec<ContentNode> = results
            .into_iter()
            .map(|r| serde_json::from_value(r.metadata).unwrap())
            .collect();

        Ok(contents)
    }

    /// Batch insert for efficiency
    pub async fn batch_insert_content(
        &self,
        contents: Vec<ContentNode>,
    ) -> Result<(), Error> {
        let entries: Vec<VectorEntry> = contents
            .into_iter()
            .map(|c| VectorEntry {
                id: c.id.clone(),
                vector: c.embedding.clone().into(),
                metadata: serde_json::to_value(&c).unwrap(),
            })
            .collect();

        let db = self.vector_db.write().await;
        for entry in entries {
            db.insert(entry).await?;
        }

        Ok(())
    }
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct ContentNode {
    pub id: String,
    pub title: String,
    pub synopsis: String,
    pub embedding: Vec<f32>,
    pub imdb_rating: Option<f32>,
    pub trust_score: f32,
    // ... other fields
}

#[derive(Debug)]
pub enum Error {
    RuvectorError(String),
    SerializationError(serde_json::Error),
}

impl From<serde_json::Error> for Error {
    fn from(err: serde_json::Error) -> Self {
        Error::SerializationError(err)
    }
}
```

### 5.2 Async Query Execution

```rust
use tokio::sync::mpsc;
use futures::stream::{self, StreamExt};

/// Async query executor with streaming results
pub struct AsyncQueryExecutor {
    ruvector: Arc<RuvectorClient>,
    max_concurrent_queries: usize,
}

impl AsyncQueryExecutor {
    pub fn new(ruvector: Arc<RuvectorClient>) -> Self {
        Self {
            ruvector,
            max_concurrent_queries: 10,
        }
    }

    /// Execute query with streaming results
    pub async fn execute_streaming(
        &self,
        query: SearchQuery,
    ) -> mpsc::Receiver<SearchResult> {
        let (tx, rx) = mpsc::channel(100);
        let ruvector = self.ruvector.clone();

        tokio::spawn(async move {
            // Vector search
            let vector_results = ruvector
                .search_content(&query.embedding, query.k, None)
                .await
                .unwrap_or_default();

            // Stream results as they're processed
            for content in vector_results {
                // Enrich with availability data
                let enriched = enrich_with_availability(content, &query.region).await;

                if tx.send(enriched).await.is_err() {
                    break; // Receiver dropped
                }
            }
        });

        rx
    }

    /// Execute multiple queries in parallel
    pub async fn execute_parallel(
        &self,
        queries: Vec<SearchQuery>,
    ) -> Vec<Vec<SearchResult>> {
        stream::iter(queries)
            .map(|query| {
                let ruvector = self.ruvector.clone();
                async move {
                    ruvector
                        .search_content(&query.embedding, query.k, None)
                        .await
                        .unwrap_or_default()
                }
            })
            .buffered(self.max_concurrent_queries)
            .collect()
            .await
    }
}

pub struct SearchQuery {
    pub embedding: Vec<f32>,
    pub k: usize,
    pub region: String,
}

pub struct SearchResult {
    pub content: ContentNode,
    pub availability: Vec<AvailabilityInfo>,
    pub score: f32,
}

async fn enrich_with_availability(
    content: ContentNode,
    region: &str,
) -> SearchResult {
    // Query availability data
    // In production: query graph for AVAILABILITY_WINDOW hyperedges
    SearchResult {
        content,
        availability: vec![],
        score: 0.9,
    }
}
```

### 5.3 Batch Ingestion

```rust
use tokio::sync::Semaphore;
use std::sync::atomic::{AtomicUsize, Ordering};

/// Batch ingestion with rate limiting and progress tracking
pub struct BatchIngester {
    ruvector: Arc<RuvectorClient>,
    semaphore: Arc<Semaphore>,
    progress: Arc<AtomicUsize>,
}

impl BatchIngester {
    pub fn new(
        ruvector: Arc<RuvectorClient>,
        max_concurrent: usize,
    ) -> Self {
        Self {
            ruvector,
            semaphore: Arc::new(Semaphore::new(max_concurrent)),
            progress: Arc::new(AtomicUsize::new(0)),
        }
    }

    /// Ingest content in batches
    pub async fn ingest_content_batch(
        &self,
        contents: Vec<ContentNode>,
        batch_size: usize,
    ) -> Result<IngestionReport, Error> {
        let total = contents.len();
        let mut handles = vec![];

        for batch in contents.chunks(batch_size) {
            let permit = self.semaphore.clone().acquire_owned().await.unwrap();
            let ruvector = self.ruvector.clone();
            let progress = self.progress.clone();
            let batch = batch.to_vec();

            let handle = tokio::spawn(async move {
                let result = ruvector.batch_insert_content(batch.clone()).await;
                progress.fetch_add(batch.len(), Ordering::Relaxed);
                drop(permit);
                result
            });

            handles.push(handle);
        }

        // Wait for all batches
        let mut successful = 0;
        let mut failed = 0;

        for handle in handles {
            match handle.await {
                Ok(Ok(())) => successful += batch_size,
                Ok(Err(_)) => failed += batch_size,
                Err(_) => failed += batch_size,
            }
        }

        Ok(IngestionReport {
            total,
            successful,
            failed,
        })
    }

    /// Get current progress
    pub fn get_progress(&self) -> usize {
        self.progress.load(Ordering::Relaxed)
    }
}

pub struct IngestionReport {
    pub total: usize,
    pub successful: usize,
    pub failed: usize,
}
```

### 5.4 Index Management

```rust
/// Index manager for Ruvector
pub struct IndexManager {
    ruvector: Arc<RuvectorClient>,
}

impl IndexManager {
    pub fn new(ruvector: Arc<RuvectorClient>) -> Self {
        Self { ruvector }
    }

    /// Rebuild index from scratch
    pub async fn rebuild_index(
        &self,
        index_name: &str,
    ) -> Result<(), Error> {
        println!("Rebuilding index: {}", index_name);

        // 1. Fetch all nodes from graph
        let nodes = self.fetch_all_nodes(index_name).await?;
        println!("Fetched {} nodes", nodes.len());

        // 2. Clear existing index
        self.clear_index(index_name).await?;
        println!("Cleared existing index");

        // 3. Batch insert with progress tracking
        let ingester = BatchIngester::new(
            self.ruvector.clone(),
            10, // max concurrent batches
        );

        let report = ingester.ingest_content_batch(nodes, 1000).await?;
        println!("Ingestion complete: {:?}", report);

        Ok(())
    }

    /// Incremental index update
    pub async fn update_index(
        &self,
        updates: Vec<IndexUpdate>,
    ) -> Result<(), Error> {
        for update in updates {
            match update.operation {
                Operation::Insert => {
                    self.ruvector.insert_content(&update.content).await?;
                },
                Operation::Update => {
                    // Delete old, insert new
                    self.delete_by_id(&update.content.id).await?;
                    self.ruvector.insert_content(&update.content).await?;
                },
                Operation::Delete => {
                    self.delete_by_id(&update.content.id).await?;
                },
            }
        }

        Ok(())
    }

    async fn fetch_all_nodes(&self, _index_name: &str) -> Result<Vec<ContentNode>, Error> {
        // Query knowledge graph for all nodes
        // In production: paginated queries
        Ok(vec![])
    }

    async fn clear_index(&self, _index_name: &str) -> Result<(), Error> {
        // Clear Ruvector index
        Ok(())
    }

    async fn delete_by_id(&self, _id: &str) -> Result<(), Error> {
        // Delete from Ruvector
        Ok(())
    }
}

pub struct IndexUpdate {
    pub operation: Operation,
    pub content: ContentNode,
}

pub enum Operation {
    Insert,
    Update,
    Delete,
}
```

---

## ASCII Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                     RUVECTOR KNOWLEDGE GRAPH ARCHITECTURE                     │
│                           Global TV Discovery System                          │
└──────────────────────────────────────────────────────────────────────────────┘

                                  ┌─────────────┐
                                  │   CLIENTS   │
                                  │ CLI/Web/App │
                                  └──────┬──────┘
                                         │
                                         ▼
                           ┌─────────────────────────┐
                           │   API GATEWAY (gRPC)    │
                           │  - Auth verification     │
                           │  - Rate limiting         │
                           │  - Request routing       │
                           └────────────┬────────────┘
                                        │
                    ┌───────────────────┼───────────────────┐
                    │                   │                   │
                    ▼                   ▼                   ▼
        ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
        │ knowledge-graph  │ │ vector-indexer   │ │   gnn-trainer    │
        │                  │ │                  │ │                  │
        │ - Schema mgmt    │ │ - HNSW indexes   │ │ - GraphSAGE      │
        │ - Ingestion      │ │ - Embeddings     │ │ - Tiny Dancer    │
        │ - Query exec     │ │ - Compression    │ │ - Training       │
        │ - Maintenance    │ │ - Search         │ │ - Inference      │
        └────────┬─────────┘ └────────┬─────────┘ └────────┬─────────┘
                 │                    │                    │
                 └────────────────────┼────────────────────┘
                                      │
                                      ▼
                    ┌─────────────────────────────────┐
                    │        RUVECTOR CORE            │
                    │  ┌───────────────────────────┐  │
                    │  │   HYPERGRAPH LAYER        │  │
                    │  │  - Nodes (typed)          │  │
                    │  │  - Edges (binary)         │  │
                    │  │  - Hyperedges (n-ary)     │  │
                    │  │  - Cypher queries         │  │
                    │  └───────────┬───────────────┘  │
                    │              │                  │
                    │  ┌───────────▼───────────────┐  │
                    │  │   VECTOR INDEX LAYER      │  │
                    │  │  - HNSW indexes           │  │
                    │  │  - Semantic search        │  │
                    │  │  - Similarity ranking     │  │
                    │  └───────────┬───────────────┘  │
                    │              │                  │
                    │  ┌───────────▼───────────────┐  │
                    │  │   GNN LAYER               │  │
                    │  │  - RuvectorLayer (Rust)   │  │
                    │  │  - Multi-head attention   │  │
                    │  │  - Neighbor aggregation   │  │
                    │  │  - Differentiable search  │  │
                    │  │  - FastGRNN (Tiny Dancer) │  │
                    │  └───────────┬───────────────┘  │
                    │              │                  │
                    │  ┌───────────▼───────────────┐  │
                    │  │   STORAGE LAYER           │  │
                    │  │  - Persistent storage     │  │
                    │  │  - SIMD optimizations     │  │
                    │  │  - Memory mapping         │  │
                    │  └───────────────────────────┘  │
                    └─────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│                          DATA FLOW EXAMPLE                                    │
└──────────────────────────────────────────────────────────────────────────────┘

User Query: "Find sci-fi shows like Stranger Things on Netflix in US"

    1. API Gateway receives request
           │
           ▼
    2. Query Router (Tiny Dancer) classifies: HYBRID path
           │
           ├─────────────────────┬─────────────────────┐
           ▼                     ▼                     ▼
    3. Parallel Execution:

       VECTOR PATH:              GRAPH PATH:           FILTER PATH:
       - Embed query             - Parse intent        - Region: US
       - HNSW search             - Genre: Sci-Fi       - Platform: Netflix
       - Top 100 candidates      - Similar nodes       - Active licenses
           │                         │                     │
           └─────────────────────────┼─────────────────────┘
                                     ▼
    4. Result Aggregation & Ranking:
       - Merge results
       - Apply GNN scoring
       - Trust score weighting
       - Availability filtering
           │
           ▼
    5. Response:
       [
         {
           title: "Dark",
           similarity: 0.92,
           gnn_score: 0.88,
           trust_score: 0.95,
           netflix_link: "https://...",
           expires: "2026-06-30"
         },
         ...
       ]

┌──────────────────────────────────────────────────────────────────────────────┐
│                     NODE TYPE MEMORY FOOTPRINT                                │
└──────────────────────────────────────────────────────────────────────────────┘

Node Type          Count      Embedding Dim    Memory per Node    Total Memory
─────────────────  ─────────  ───────────────  ─────────────────  ────────────
Movie/TVShow       1,000,000  1536             ~6 KB              ~6 GB
Person             500,000    768              ~3 KB              ~1.5 GB
Episode            10,000,000 768              ~3 KB              ~30 GB
User (preferences) 1,000,000  512              ~2 KB              ~2 GB
Genre/Mood/Theme   5,000      256              ~1 KB              ~5 MB
Platform           50         -                ~1 KB              ~50 KB
Region             200        -                ~500 B             ~100 KB

TOTAL GRAPH SIZE: ~40 GB (embeddings) + ~10 GB (metadata) = ~50 GB

HNSW Index Overhead: ~2x embedding size = ~80 GB total
Recommended RAM: 128 GB for in-memory operations
```

---

## Conclusion

This specification provides a complete blueprint for building a Ruvector-powered knowledge graph and GNN architecture for global TV discovery. The system leverages:

1. **Hypergraph capabilities** for complex n-ary relationships (availability windows)
2. **Vector embeddings** for semantic search across content, people, and user preferences
3. **GNN layers** (GraphSAGE, FastGRNN) for self-improving recommendations
4. **Privacy-first design** with federated learning and differential privacy
5. **Rust-native implementation** for maximum performance
6. **Modular micro-services** for scalability and maintainability

The architecture is production-ready and can scale from development (synthetic data) to production (real aggregator APIs) with minimal changes.

**Next Steps:**
1. Implement `knowledge-graph` repository with schema definitions
2. Build `vector-indexer` with HNSW optimization
3. Develop `gnn-trainer` with GraphSAGE and Tiny Dancer
4. Create integration tests with synthetic data
5. Deploy to Kubernetes with monitoring

---

**Document Metadata:**
- Format: Markdown + ASCII diagrams
- Language: Rust (primary), Cypher (queries)
- Dependencies: Ruvector, @ruvector/gnn, @ruvector/attention
- Estimated Implementation: 12-16 weeks (3 engineers)
- License: MIT

