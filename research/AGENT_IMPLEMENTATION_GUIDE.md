# Agent Orchestrator Implementation Guide
## Practical Implementation Details for Layer-2 Multi-Agent System

---

## Table of Contents

1. [Quick Start Guide](#1-quick-start-guide)
2. [Configuration Files](#2-configuration-files)
3. [Example Agent Implementations](#3-example-agent-implementations)
4. [Testing Strategies](#4-testing-strategies)
5. [Deployment Patterns](#5-deployment-patterns)
6. [Monitoring and Observability](#6-monitoring-and-observability)

---

## 1. Quick Start Guide

### 1.1 Project Setup

```bash
# Create agent-orchestrator service
mkdir -p services/agent-orchestrator
cd services/agent-orchestrator

# Initialize package.json
npm init -y

# Install dependencies
npm install claude-flow@^2.7.41 \
  ruvector@^0.1.31 \
  @grpc/grpc-js \
  @grpc/proto-loader \
  ioredis \
  kafkajs \
  winston \
  prom-client

# Install dev dependencies
npm install -D \
  typescript \
  @types/node \
  ts-node \
  nodemon \
  jest \
  @types/jest
```

### 1.2 TypeScript Configuration

**tsconfig.json:**

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```

### 1.3 Directory Structure Setup

```bash
mkdir -p src/{orchestrator,agents/{base,coordinators,specialists,memory},sparc,mcp/{tools},streaming,memory,utils}
mkdir -p config proto tests/{unit,integration,scenarios}
```

---

## 2. Configuration Files

### 2.1 Agent Configuration

**config/agents.yaml:**

```yaml
agents:
  # Coordinator Agents
  SwarmLead:
    type: coordinator
    maxConcurrency: 1
    priority: 10
    timeout: 30000
    capabilities:
      - task_delegation
      - progress_monitoring
      - failure_handling
    hooks:
      beforeExecution: enrichWithContext
      afterExecution: updateMetrics
      onError: handleFailure

  ResultMerger:
    type: coordinator
    maxConcurrency: 3
    priority: 8
    timeout: 5000
    capabilities:
      - result_aggregation
      - deduplication
      - ranking

  # Specialist Agents
  ContentSearcher:
    type: specialist
    maxConcurrency: 5
    priority: 7
    timeout: 10000
    capabilities:
      - semantic_search
      - availability_filtering
    tools:
      - ruvector_search
      - filter_apply
    cache:
      enabled: true
      ttl: 3600

  RecommendationBuilder:
    type: specialist
    maxConcurrency: 3
    priority: 7
    timeout: 15000
    capabilities:
      - gnn_recommend
      - preference_modeling
      - collaborative_filtering
    tools:
      - gnn_recommend
      - preference_model
    cache:
      enabled: true
      ttl: 7200

  AvailabilityChecker:
    type: specialist
    maxConcurrency: 5
    priority: 6
    timeout: 5000
    capabilities:
      - rights_validation
      - deeplink_verification
    tools:
      - rights_check
      - deeplink_validate
    cache:
      enabled: true
      ttl: 1800

  DeviceCoordinator:
    type: specialist
    maxConcurrency: 2
    priority: 5
    timeout: 10000
    capabilities:
      - device_communication
      - playback_control
      - state_sync
    tools:
      - device_send
      - sync_state

  # Memory Agents
  ContextKeeper:
    type: memory
    maxConcurrency: 10
    priority: 9
    timeout: 1000
    capabilities:
      - context_storage
      - context_retrieval
      - session_management
    tools:
      - memory_store
      - memory_retrieve
    storage:
      backend: agentdb
      ttl: 3600

  PatternLearner:
    type: memory
    maxConcurrency: 2
    priority: 3
    timeout: 5000
    capabilities:
      - pattern_detection
      - reflexion
      - optimization
    tools:
      - pattern_store
      - pattern_search
      - reflexion_update
    storage:
      backend: reasoningbank
      ttl: 604800  # 7 days
```

### 2.2 MCP Tool Definitions

**config/tools.yaml:**

```yaml
tools:
  ruvector_search:
    name: ruvector_search
    description: Semantic search over content catalog
    service:
      name: semantic-search
      endpoint: semantic-search.default.svc.cluster.local:50051
      protocol: grpc
      timeout: 5000
      retries: 3
    schema:
      input:
        query:
          type: string
          required: true
          description: Search query text
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
            minTrustScore:
              type: number
              minimum: 0
              maximum: 1
              default: 0.7
        limit:
          type: number
          default: 20
          minimum: 1
          maximum: 100
        offset:
          type: number
          default: 0
          minimum: 0
      output:
        type: object
        properties:
          results:
            type: array
          total:
            type: number

  gnn_recommend:
    name: gnn_recommend
    description: GNN-based recommendations
    service:
      name: recommendation-engine
      endpoint: recommendation-engine.default.svc.cluster.local:50051
      protocol: grpc
      timeout: 10000
      retries: 2
    schema:
      input:
        userId:
          type: string
          required: true
        contentContext:
          type: array
          items:
            type: string
          default: []
        numRecommendations:
          type: number
          default: 20
        diversityFactor:
          type: number
          minimum: 0
          maximum: 1
          default: 0.3
      output:
        type: object
        properties:
          recommendations:
            type: array

  rights_check:
    name: rights_check
    description: Validate content availability
    service:
      name: rights-validator
      endpoint: rights-validator.default.svc.cluster.local:50051
      protocol: grpc
      timeout: 3000
      retries: 3
    schema:
      input:
        contentId:
          type: string
          required: true
        region:
          type: string
          required: true
      output:
        type: object
        properties:
          available:
            type: boolean
          platforms:
            type: array
          restrictions:
            type: array
          expiresAt:
            type: string

  device_send:
    name: device_send
    description: Send commands to devices
    service:
      name: device-gateway
      endpoint: device-gateway.default.svc.cluster.local:50051
      protocol: grpc
      timeout: 5000
      retries: 2
    schema:
      input:
        deviceId:
          type: string
          required: true
        deviceType:
          type: string
          required: true
        command:
          type: object
          required: true
      output:
        type: object
        properties:
          success:
            type: boolean
          error:
            type: string
          state:
            type: object

  memory_store:
    name: memory_store
    description: Store data in memory databases
    service:
      name: agent-orchestrator
      endpoint: localhost:50051
      protocol: internal
      timeout: 1000
    schema:
      input:
        database:
          type: string
          enum: [agentdb, reasoningbank]
          required: true
        collection:
          type: string
          required: true
        key:
          type: string
          required: true
        value:
          type: any
          required: true
        ttl:
          type: number
          default: 3600
        merge:
          type: boolean
          default: false
      output:
        type: object
        properties:
          success:
            type: boolean
          key:
            type: string
```

### 2.3 SPARC Phase Configuration

**config/sparc.yaml:**

```yaml
sparc:
  # S: Specification Phase
  specification:
    enabled: true
    timeout: 1000
    model: claude-opus-4-5-20251101
    prompts:
      intentAnalysis: |
        Analyze the user's query and determine their intent.

        Query: {query}
        Context: {context}

        Classify the intent and extract:
        - Primary intent (SEARCH, RECOMMEND, BROWSE, etc.)
        - Secondary intents
        - Required data domains
        - Output format preference
        - Urgency level
      entityExtraction: |
        Extract entities from the query.

        Query: {query}

        Extract:
        - Content titles
        - Genres
        - People (actors, directors)
        - Platforms
        - Time references

  # P: Pseudocode Phase
  pseudocode:
    enabled: true
    timeout: 2000
    cachePatterns: true
    patternSimilarityThreshold: 0.85
    optimization:
      parallelization: auto
      fallbackGeneration: true

  # A: Architecture Phase
  architecture:
    enabled: true
    timeout: 1000
    agentSelection:
      strategy: capability-based
      loadBalancing: true
    communication:
      channels:
        buffer: 1000
        timeout: 30000
      hooks:
        enabled: true

  # R: Refinement Phase
  refinement:
    enabled: true
    timeout: 3000
    validation:
      enabled: true
      strictMode: false
    trustScoring:
      enabled: true
      minScore: 0.7
      weights:
        sourceReliability: 0.25
        metadataAccuracy: 0.25
        availabilityConfidence: 0.20
        recommendationQuality: 0.15
        preferenceConfidence: 0.15
    diversity:
      enabled: true
      maxSameGenre: 3
      maxSamePlatform: 5
      noveltyBoost: 0.2

  # C: Completion Phase
  completion:
    enabled: true
    timeout: 2000
    formatting:
      supportedFormats:
        - minimal
        - list
        - detailed
      defaultFormat: list
    explanations:
      enabled: true
      maxExplanations: 5
    memoryUpdate:
      enabled: true
      async: true
    patternLearning:
      enabled: true
      async: true
```

### 2.4 Service Configuration

**config/service.yaml:**

```yaml
service:
  name: agent-orchestrator
  version: 1.0.0
  port: 50051
  protocol: grpc

  # Claude-Flow Configuration
  claudeFlow:
    modelId: claude-opus-4-5-20251101
    apiKey: ${ANTHROPIC_API_KEY}
    streaming: true
    maxTokens: 4096
    temperature: 0.7

  # Task Queue Configuration
  taskQueue:
    maxConcurrency: 10
    priorityLevels: 5
    backpressure:
      enabled: true
      threshold: 100
      strategy: drop_oldest

  # Agent Registry Configuration
  registry:
    healthCheckInterval: 30000
    unhealthyThreshold: 3
    recovery:
      enabled: true
      backoffMs: 1000
      maxRetries: 5

  # Memory Backends
  memory:
    agentdb:
      type: redis
      host: ${REDIS_HOST}
      port: 6379
      db: 0
      password: ${REDIS_PASSWORD}
      keyPrefix: agentdb:
    reasoningbank:
      type: ruvector
      endpoint: ${RUVECTOR_ENDPOINT}
      collection: reasoning_patterns

  # Streaming Configuration
  streaming:
    enabled: true
    chunkSize: 10
    bufferSize: 100
    timeout: 30000

  # Monitoring
  monitoring:
    enabled: true
    metrics:
      port: 9090
      path: /metrics
    tracing:
      enabled: true
      endpoint: ${JAEGER_ENDPOINT}
      serviceName: agent-orchestrator
    logging:
      level: info
      format: json
      destination: stdout
```

---

## 3. Example Agent Implementations

### 3.1 Base Agent Class

**src/agents/base/BaseAgent.ts:**

```typescript
import { EventEmitter } from 'events';
import { MCPExecutor } from '../../mcp/MCPExecutor';
import { Logger } from '../../utils/Logger';
import { MetricsCollector } from '../../utils/MetricsCollector';

export interface Task {
  id: string;
  action: string;
  input: any;
  timeout: number;
  priority: number;
}

export interface ExecutionContext {
  queryId: string;
  userId: string;
  query: any;
  userContext: any;
  patterns: any[];
  timestamp: number;
  metadata: Record<string, any>;
}

export interface AgentResult {
  type: string;
  [key: string]: any;
}

export interface AgentHooks {
  beforeExecution?: (context: ExecutionContext) => Promise<ExecutionContext>;
  afterExecution?: (result: AgentResult) => Promise<AgentResult>;
  onError?: (error: Error, context: ExecutionContext) => Promise<void>;
}

export abstract class BaseAgent extends EventEmitter {
  abstract name: string;
  abstract type: 'coordinator' | 'specialist' | 'memory';

  protected mcpExecutor: MCPExecutor;
  protected logger: Logger;
  protected metrics: MetricsCollector;
  public hooks: AgentHooks = {};

  constructor(
    mcpExecutor: MCPExecutor,
    logger: Logger,
    metrics: MetricsCollector
  ) {
    super();
    this.mcpExecutor = mcpExecutor;
    this.logger = logger;
    this.metrics = metrics;
  }

  async execute(task: Task, context: ExecutionContext): Promise<AgentResult> {
    const startTime = Date.now();

    // Emit start event
    this.emit('start', { agent: this.name, task });

    try {
      // Before execution hook
      if (this.hooks.beforeExecution) {
        context = await this.hooks.beforeExecution(context);
      }

      // Execute agent logic
      this.logger.info(`Executing ${this.name}`, { taskId: task.id });
      const result = await this.executeInternal(task, context);

      // After execution hook
      let finalResult = result;
      if (this.hooks.afterExecution) {
        finalResult = await this.hooks.afterExecution(result);
      }

      // Emit complete event
      const duration = Date.now() - startTime;
      this.emit('complete', { agent: this.name, task, result: finalResult, duration });

      // Record metrics
      this.metrics.recordAgentExecution(this.name, duration, true);

      return finalResult;
    } catch (error) {
      // Error hook
      if (this.hooks.onError) {
        await this.hooks.onError(error as Error, context);
      }

      // Emit error event
      const duration = Date.now() - startTime;
      this.emit('error', { agent: this.name, task, error, duration });

      // Record metrics
      this.metrics.recordAgentExecution(this.name, duration, false);

      this.logger.error(`Error in ${this.name}`, { error, taskId: task.id });
      throw error;
    }
  }

  protected abstract executeInternal(
    task: Task,
    context: ExecutionContext
  ): Promise<AgentResult>;

  // Helper methods
  protected async executeTool(
    toolName: string,
    input: any
  ): Promise<any> {
    const startTime = Date.now();

    try {
      const result = await this.mcpExecutor.execute(toolName, input);
      const duration = Date.now() - startTime;

      this.metrics.recordToolExecution(toolName, duration, true);
      return result;
    } catch (error) {
      const duration = Date.now() - startTime;
      this.metrics.recordToolExecution(toolName, duration, false);
      throw error;
    }
  }

  protected async executeToolWithRetry(
    toolName: string,
    input: any,
    retries: number = 3,
    backoffMs: number = 1000
  ): Promise<any> {
    for (let i = 0; i < retries; i++) {
      try {
        return await this.executeTool(toolName, input);
      } catch (error) {
        if (i === retries - 1) throw error;

        const delay = backoffMs * Math.pow(2, i);
        this.logger.warn(`Tool ${toolName} failed, retrying in ${delay}ms`, {
          attempt: i + 1,
          error
        });

        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
  }
}
```

### 3.2 ContentSearcher Implementation

**src/agents/specialists/ContentSearcher.ts:**

```typescript
import { BaseAgent, Task, ExecutionContext, AgentResult } from '../base/BaseAgent';

interface SearchParams {
  embedQuery: string;
  genres: string[];
  platforms: string[];
  mood?: string;
  limit: number;
  offset: number;
}

export class ContentSearcher extends BaseAgent {
  name = 'ContentSearcher';
  type = 'specialist' as const;

  protected async executeInternal(
    task: Task,
    context: ExecutionContext
  ): Promise<AgentResult> {
    // Parse search query
    const searchParams = await this.parseSearchQuery(task.query, context);

    this.logger.info('Executing semantic search', {
      query: searchParams.embedQuery,
      filters: {
        genres: searchParams.genres,
        platforms: searchParams.platforms
      }
    });

    // Execute search using MCP tool
    const searchResults = await this.executeTool('ruvector_search', {
      query: searchParams.embedQuery,
      filters: {
        genres: searchParams.genres,
        platforms: searchParams.platforms,
        region: context.userContext.region,
        minTrustScore: 0.7
      },
      limit: searchParams.limit,
      offset: searchParams.offset
    });

    // Apply availability filter
    const filteredResults = this.applyAvailabilityFilter(
      searchResults.results,
      context.userContext.subscriptions || []
    );

    return {
      type: 'search_result',
      results: filteredResults,
      totalFound: searchResults.total,
      query: searchParams
    };
  }

  private async parseSearchQuery(
    query: string,
    context: ExecutionContext
  ): Promise<SearchParams> {
    // Extract structured parameters from natural language query
    // This could use an LLM or rule-based extraction

    return {
      embedQuery: query,
      genres: this.extractGenres(query),
      platforms: context.userContext.subscriptions || [],
      mood: this.extractMood(query),
      limit: 20,
      offset: 0
    };
  }

  private extractGenres(query: string): string[] {
    // Simple genre extraction (in production, use LLM)
    const genreKeywords: Record<string, string[]> = {
      'sci-fi': ['sci-fi', 'science fiction', 'futuristic'],
      'action': ['action', 'thriller', 'adventure'],
      'drama': ['drama', 'emotional'],
      'comedy': ['comedy', 'funny', 'humor']
    };

    const genres: string[] = [];
    const lowerQuery = query.toLowerCase();

    for (const [genre, keywords] of Object.entries(genreKeywords)) {
      if (keywords.some(keyword => lowerQuery.includes(keyword))) {
        genres.push(genre);
      }
    }

    return genres;
  }

  private extractMood(query: string): string | undefined {
    const moodKeywords: Record<string, string[]> = {
      'dark': ['dark', 'gritty', 'intense'],
      'light': ['light', 'fun', 'cheerful'],
      'mysterious': ['mystery', 'mysterious', 'suspense']
    };

    const lowerQuery = query.toLowerCase();

    for (const [mood, keywords] of Object.entries(moodKeywords)) {
      if (keywords.some(keyword => lowerQuery.includes(keyword))) {
        return mood;
      }
    }

    return undefined;
  }

  private applyAvailabilityFilter(
    results: any[],
    subscriptions: string[]
  ): any[] {
    return results.map(result => {
      // Calculate availability score
      const availabilityScore = this.calculateAvailabilityScore(
        result.availability,
        subscriptions
      );

      return {
        ...result,
        availabilityScore
      };
    }).sort((a, b) => b.availabilityScore - a.availabilityScore);
  }

  private calculateAvailabilityScore(
    availability: any[],
    subscriptions: string[]
  ): number {
    if (!availability || availability.length === 0) {
      return 0;
    }

    // Higher score if available on user's subscriptions
    const onSubscription = availability.some(a =>
      subscriptions.includes(a.platform)
    );

    if (onSubscription) {
      return 1.0;
    }

    // Lower score if only available for rent/purchase
    const freeAvailable = availability.some(a =>
      a.pricingModel === 'FREE' || a.pricingModel === 'INCLUDED'
    );

    return freeAvailable ? 0.5 : 0.2;
  }
}
```

### 3.3 SwarmLead Implementation

**src/agents/coordinators/SwarmLead.ts:**

```typescript
import { BaseAgent, Task, ExecutionContext, AgentResult } from '../base/BaseAgent';
import { AgentRegistry } from '../../orchestrator/AgentRegistry';
import { EventEmitter } from 'events';

interface ExecutionStrategy {
  requiredAgents: string[];
  executionMode: 'parallel' | 'sequential';
  timeout: number;
  fallbackStrategy?: ExecutionStrategy;
}

interface QueryIntent {
  requiresSearch: boolean;
  requiresRecommendations: boolean;
  requiresAvailabilityCheck: boolean;
  requiresDeviceControl: boolean;
  requiresRealtime: boolean;
}

export class SwarmLead extends BaseAgent {
  name = 'SwarmLead';
  type = 'coordinator' as const;

  private registry: AgentRegistry;

  constructor(
    mcpExecutor: any,
    logger: any,
    metrics: any,
    registry: AgentRegistry
  ) {
    super(mcpExecutor, logger, metrics);
    this.registry = registry;
  }

  protected async executeInternal(
    task: Task,
    context: ExecutionContext
  ): Promise<AgentResult> {
    // Determine execution strategy
    const strategy = await this.determineStrategy(task.query, context);

    this.logger.info('SwarmLead strategy determined', {
      strategy,
      queryId: context.queryId
    });

    // Create agent tasks
    const agentTasks = this.createAgentTasks(strategy, task, context);

    // Execute agents based on strategy
    const results = await this.executeAgents(agentTasks, strategy);

    // Aggregate results
    const finalResult = await this.aggregateResults(results);

    return {
      type: 'swarm_lead_result',
      strategy,
      results: finalResult,
      agentMetrics: this.collectAgentMetrics()
    };
  }

  private async determineStrategy(
    query: string,
    context: ExecutionContext
  ): Promise<ExecutionStrategy> {
    // Check for cached patterns
    const patterns = context.patterns;

    if (patterns && patterns.length > 0) {
      this.logger.info('Using cached pattern', {
        patternId: patterns[0].queryEmbedding
      });

      return patterns[0].executionStrategy;
    }

    // Analyze query to determine strategy
    const intent = await this.analyzeIntent(query);

    const requiredAgents = this.selectAgents(intent);

    return {
      requiredAgents,
      executionMode: intent.requiresRealtime ? 'parallel' : 'sequential',
      timeout: 30000,
      fallbackStrategy: this.createFallbackStrategy(intent)
    };
  }

  private async analyzeIntent(query: string): Promise<QueryIntent> {
    // Simple intent detection (in production, use LLM)
    const lowerQuery = query.toLowerCase();

    return {
      requiresSearch: lowerQuery.includes('find') ||
                      lowerQuery.includes('search') ||
                      lowerQuery.includes('show'),
      requiresRecommendations: lowerQuery.includes('recommend') ||
                                lowerQuery.includes('suggest') ||
                                lowerQuery.includes('what should'),
      requiresAvailabilityCheck: true, // Always check
      requiresDeviceControl: lowerQuery.includes('play') ||
                             lowerQuery.includes('watch on'),
      requiresRealtime: true // Default to realtime
    };
  }

  private selectAgents(intent: QueryIntent): string[] {
    const agents: string[] = [];

    // Always start with context
    agents.push('ContextKeeper');

    if (intent.requiresSearch) {
      agents.push('ContentSearcher');
    }

    if (intent.requiresRecommendations) {
      agents.push('RecommendationBuilder');
    }

    if (intent.requiresAvailabilityCheck) {
      agents.push('AvailabilityChecker');
    }

    if (intent.requiresDeviceControl) {
      agents.push('DeviceCoordinator');
    }

    return agents;
  }

  private createAgentTasks(
    strategy: ExecutionStrategy,
    parentTask: Task,
    context: ExecutionContext
  ): Map<string, Task> {
    const tasks = new Map<string, Task>();

    for (const agentName of strategy.requiredAgents) {
      tasks.set(agentName, {
        id: `${parentTask.id}-${agentName}`,
        action: this.getDefaultAction(agentName),
        input: this.prepareAgentInput(agentName, parentTask, context),
        timeout: 10000,
        priority: 5
      });
    }

    return tasks;
  }

  private getDefaultAction(agentName: string): string {
    const actionMap: Record<string, string> = {
      'ContextKeeper': 'retrieve',
      'ContentSearcher': 'search',
      'RecommendationBuilder': 'recommend',
      'AvailabilityChecker': 'check',
      'DeviceCoordinator': 'send'
    };

    return actionMap[agentName] || 'execute';
  }

  private prepareAgentInput(
    agentName: string,
    parentTask: Task,
    context: ExecutionContext
  ): any {
    // Prepare input specific to each agent
    switch (agentName) {
      case 'ContextKeeper':
        return { key: 'user_context' };
      case 'ContentSearcher':
        return { query: parentTask.input.query };
      case 'RecommendationBuilder':
        return { userId: context.userId };
      case 'AvailabilityChecker':
        return { contentIds: [] }; // Will be filled by search results
      default:
        return parentTask.input;
    }
  }

  private async executeAgents(
    tasks: Map<string, Task>,
    strategy: ExecutionStrategy
  ): Promise<Map<string, AgentResult>> {
    const results = new Map<string, AgentResult>();

    if (strategy.executionMode === 'parallel') {
      // Execute all agents in parallel
      const promises = Array.from(tasks.entries()).map(async ([name, task]) => {
        const agent = await this.registry.spawn(name, task, {} as ExecutionContext);
        const result = await agent.execute();
        return [name, result] as [string, AgentResult];
      });

      const agentResults = await Promise.all(promises);

      for (const [name, result] of agentResults) {
        results.set(name, result);
      }
    } else {
      // Execute sequentially
      for (const [name, task] of tasks.entries()) {
        const agent = await this.registry.spawn(name, task, {} as ExecutionContext);
        const result = await agent.execute();
        results.set(name, result);

        // Update subsequent tasks based on results
        this.updateTasksWithResults(tasks, name, result);
      }
    }

    return results;
  }

  private updateTasksWithResults(
    tasks: Map<string, Task>,
    completedAgent: string,
    result: AgentResult
  ): void {
    // Update AvailabilityChecker input with search results
    if (completedAgent === 'ContentSearcher' && tasks.has('AvailabilityChecker')) {
      const availTask = tasks.get('AvailabilityChecker')!;
      availTask.input.contentIds = result.results.map((r: any) => r.contentId);
    }
  }

  private async aggregateResults(
    results: Map<string, AgentResult>
  ): Promise<any> {
    // Combine results from all agents
    const searchResults = results.get('ContentSearcher');
    const recommendations = results.get('RecommendationBuilder');
    const availability = results.get('AvailabilityChecker');

    // Merge and deduplicate
    const allResults = [
      ...(searchResults?.results || []),
      ...(recommendations?.recommendations || [])
    ];

    // Apply availability information
    if (availability) {
      for (const result of allResults) {
        const avail = availability.availability.find(
          (a: any) => a.contentId === result.contentId
        );

        if (avail) {
          result.availability = avail.platforms;
        }
      }
    }

    // Deduplicate by contentId
    const uniqueResults = Array.from(
      new Map(allResults.map(r => [r.contentId, r])).values()
    );

    return uniqueResults;
  }

  private createFallbackStrategy(intent: QueryIntent): ExecutionStrategy {
    // Simplified fallback: just search if recommendations fail
    return {
      requiredAgents: ['ContextKeeper', 'ContentSearcher'],
      executionMode: 'sequential',
      timeout: 15000
    };
  }

  private collectAgentMetrics(): any {
    // Collect metrics from all spawned agents
    return {
      totalAgents: this.registry.getActiveAgentCount(),
      executionTime: this.metrics.getAverageExecutionTime()
    };
  }
}
```

---

## 4. Testing Strategies

### 4.1 Unit Tests

**tests/unit/agents/ContentSearcher.test.ts:**

```typescript
import { ContentSearcher } from '../../../src/agents/specialists/ContentSearcher';
import { MCPExecutor } from '../../../src/mcp/MCPExecutor';
import { Logger } from '../../../src/utils/Logger';
import { MetricsCollector } from '../../../src/utils/MetricsCollector';

jest.mock('../../../src/mcp/MCPExecutor');
jest.mock('../../../src/utils/Logger');
jest.mock('../../../src/utils/MetricsCollector');

describe('ContentSearcher', () => {
  let agent: ContentSearcher;
  let mockExecutor: jest.Mocked<MCPExecutor>;
  let mockLogger: jest.Mocked<Logger>;
  let mockMetrics: jest.Mocked<MetricsCollector>;

  beforeEach(() => {
    mockExecutor = new MCPExecutor({} as any) as jest.Mocked<MCPExecutor>;
    mockLogger = new Logger() as jest.Mocked<Logger>;
    mockMetrics = new MetricsCollector() as jest.Mocked<MetricsCollector>;

    agent = new ContentSearcher(mockExecutor, mockLogger, mockMetrics);
  });

  describe('execute', () => {
    it('should execute search and return results', async () => {
      // Arrange
      const task = {
        id: 'test-task',
        action: 'search',
        input: { query: 'sci-fi shows' },
        timeout: 10000,
        priority: 5
      };

      const context = {
        queryId: 'test-query',
        userId: 'user-123',
        query: 'sci-fi shows',
        userContext: {
          region: 'US',
          subscriptions: ['netflix', 'prime']
        },
        patterns: [],
        timestamp: Date.now(),
        metadata: {}
      };

      const mockSearchResults = {
        results: [
          {
            contentId: 'tt1',
            title: 'Stranger Things',
            relevanceScore: 0.95,
            availability: [
              { platform: 'netflix', pricingModel: 'INCLUDED' }
            ]
          },
          {
            contentId: 'tt2',
            title: 'The Expanse',
            relevanceScore: 0.88,
            availability: [
              { platform: 'prime', pricingModel: 'INCLUDED' }
            ]
          }
        ],
        total: 2
      };

      mockExecutor.execute.mockResolvedValue(mockSearchResults);

      // Act
      const result = await agent.execute(task, context);

      // Assert
      expect(result.type).toBe('search_result');
      expect(result.results).toHaveLength(2);
      expect(result.totalFound).toBe(2);
      expect(mockExecutor.execute).toHaveBeenCalledWith(
        'ruvector_search',
        expect.objectContaining({
          query: 'sci-fi shows',
          filters: expect.objectContaining({
            region: 'US',
            minTrustScore: 0.7
          })
        })
      );
    });

    it('should handle errors gracefully', async () => {
      // Arrange
      const task = {
        id: 'test-task',
        action: 'search',
        input: { query: 'sci-fi shows' },
        timeout: 10000,
        priority: 5
      };

      const context = {
        queryId: 'test-query',
        userId: 'user-123',
        query: 'sci-fi shows',
        userContext: { region: 'US', subscriptions: [] },
        patterns: [],
        timestamp: Date.now(),
        metadata: {}
      };

      mockExecutor.execute.mockRejectedValue(new Error('Search failed'));

      // Act & Assert
      await expect(agent.execute(task, context)).rejects.toThrow('Search failed');
    });
  });
});
```

### 4.2 Integration Tests

**tests/integration/agent-orchestrator.test.ts:**

```typescript
import { AgentOrchestrator } from '../../src/orchestrator/AgentOrchestrator';
import { AgentRegistry } from '../../src/orchestrator/AgentRegistry';
import { MCPToolRegistry } from '../../src/mcp/MCPToolRegistry';

describe('Agent Orchestrator Integration', () => {
  let orchestrator: AgentOrchestrator;

  beforeAll(async () => {
    // Set up test environment
    orchestrator = new AgentOrchestrator({
      agentConfig: require('../../config/agents.yaml'),
      toolConfig: require('../../config/tools.yaml'),
      sparcConfig: require('../../config/sparc.yaml')
    });

    await orchestrator.initialize();
  });

  afterAll(async () => {
    await orchestrator.shutdown();
  });

  it('should execute a simple search query', async () => {
    const query = {
      userId: 'test-user',
      text: 'Find sci-fi shows',
      region: 'US'
    };

    const results = [];
    for await (const chunk of await orchestrator.executeQuery(query)) {
      results.push(chunk);
    }

    expect(results).not.toHaveLength(0);

    const finalResult = results[results.length - 1];
    expect(finalResult.type).toBe('final_results');
    expect(finalResult.payload.results).toBeInstanceOf(Array);
  });

  it('should handle complex multi-agent queries', async () => {
    const query = {
      userId: 'test-user',
      text: 'Recommend something like Stranger Things and play it on my TV',
      region: 'US',
      devices: [{ deviceId: 'tv-1', type: 'samsung' }]
    };

    const results = [];
    for await (const chunk of await orchestrator.executeQuery(query)) {
      results.push(chunk);
    }

    // Should have executed multiple agents
    const agentStarts = results.filter(r => r.type === 'agent_start');
    expect(agentStarts.length).toBeGreaterThan(2);

    // Should have ContentSearcher, RecommendationBuilder, DeviceCoordinator
    const agentNames = agentStarts.map(r => r.payload.agent);
    expect(agentNames).toContain('ContentSearcher');
    expect(agentNames).toContain('RecommendationBuilder');
    expect(agentNames).toContain('DeviceCoordinator');
  });
});
```

### 4.3 Scenario Tests

**tests/scenarios/recommendation-flow.test.ts:**

```typescript
import { AgentOrchestrator } from '../../src/orchestrator/AgentOrchestrator';

describe('Recommendation Flow Scenarios', () => {
  let orchestrator: AgentOrchestrator;

  beforeAll(async () => {
    orchestrator = new AgentOrchestrator({
      agentConfig: require('../../config/agents.yaml'),
      toolConfig: require('../../config/tools.yaml'),
      sparcConfig: require('../../config/sparc.yaml')
    });

    await orchestrator.initialize();
  });

  describe('New User (Cold Start)', () => {
    it('should provide generic recommendations for new users', async () => {
      const query = {
        userId: 'new-user',
        text: 'What should I watch?',
        region: 'US'
      };

      const results = [];
      for await (const chunk of await orchestrator.executeQuery(query)) {
        if (chunk.type === 'final_results') {
          results.push(...chunk.payload.results);
        }
      }

      expect(results).not.toHaveLength(0);

      // Should return popular/trending content for cold start
      expect(results.length).toBeGreaterThan(5);
    });
  });

  describe('Existing User (Warm Start)', () => {
    it('should provide personalized recommendations', async () => {
      // First, establish user preferences
      await orchestrator.updateUserPreferences('existing-user', {
        preferredGenres: ['sci-fi', 'thriller'],
        watchHistory: ['tt1', 'tt2', 'tt3']
      });

      const query = {
        userId: 'existing-user',
        text: 'Recommend something for me',
        region: 'US'
      };

      const results = [];
      for await (const chunk of await orchestrator.executeQuery(query)) {
        if (chunk.type === 'final_results') {
          results.push(...chunk.payload.results);
        }
      }

      expect(results).not.toHaveLength(0);

      // Should have personalized results
      // Check if results match user preferences
      const genres = results.flatMap((r: any) => r.genres || []);
      expect(genres).toContain('sci-fi');
    });
  });

  describe('Pattern Learning', () => {
    it('should learn from successful queries', async () => {
      const query1 = {
        userId: 'test-user',
        text: 'Find sci-fi shows like Stranger Things',
        region: 'US'
      };

      // Execute query once
      for await (const chunk of await orchestrator.executeQuery(query1)) {
        // Consume stream
      }

      // Execute similar query
      const query2 = {
        userId: 'test-user',
        text: 'Find sci-fi shows like Dark',
        region: 'US'
      };

      const metrics1 = await orchestrator.getMetrics();

      for await (const chunk of await orchestrator.executeQuery(query2)) {
        // Consume stream
      }

      const metrics2 = await orchestrator.getMetrics();

      // Second query should be faster due to pattern learning
      expect(metrics2.averageExecutionTime).toBeLessThan(metrics1.averageExecutionTime);
    });
  });
});
```

---

## 5. Deployment Patterns

### 5.1 Docker Configuration

**Dockerfile:**

```dockerfile
FROM node:20-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source
COPY dist ./dist
COPY config ./config
COPY proto ./proto

# Set environment
ENV NODE_ENV=production

# Expose ports
EXPOSE 50051 9090

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:9090/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"

# Start service
CMD ["node", "dist/server.js"]
```

**docker-compose.yaml:**

```yaml
version: '3.8'

services:
  agent-orchestrator:
    build: .
    ports:
      - "50051:50051"
      - "9090:9090"
    environment:
      - NODE_ENV=production
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - REDIS_HOST=redis
      - REDIS_PASSWORD=${REDIS_PASSWORD}
      - RUVECTOR_ENDPOINT=ruvector:50051
      - JAEGER_ENDPOINT=jaeger:6831
    depends_on:
      - redis
      - ruvector
      - jaeger
    volumes:
      - ./config:/app/config:ro
    networks:
      - tv-discovery

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis-data:/data
    networks:
      - tv-discovery

  ruvector:
    image: ruvector/ruvector:latest
    ports:
      - "50052:50051"
    volumes:
      - ruvector-data:/data
    networks:
      - tv-discovery

  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "6831:6831/udp"
      - "16686:16686"
    networks:
      - tv-discovery

volumes:
  redis-data:
  ruvector-data:

networks:
  tv-discovery:
    driver: bridge
```

### 5.2 Kubernetes Deployment

**k8s/deployment.yaml:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: agent-orchestrator
  namespace: tv-discovery
spec:
  replicas: 3
  selector:
    matchLabels:
      app: agent-orchestrator
  template:
    metadata:
      labels:
        app: agent-orchestrator
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "9090"
        prometheus.io/path: "/metrics"
    spec:
      containers:
      - name: agent-orchestrator
        image: tv-discovery/agent-orchestrator:latest
        ports:
        - containerPort: 50051
          name: grpc
        - containerPort: 9090
          name: metrics
        env:
        - name: NODE_ENV
          value: "production"
        - name: ANTHROPIC_API_KEY
          valueFrom:
            secretKeyRef:
              name: anthropic-api
              key: api-key
        - name: REDIS_HOST
          value: "redis.tv-discovery.svc.cluster.local"
        - name: REDIS_PASSWORD
          valueFrom:
            secretKeyRef:
              name: redis-secret
              key: password
        - name: RUVECTOR_ENDPOINT
          value: "ruvector.tv-discovery.svc.cluster.local:50051"
        - name: JAEGER_ENDPOINT
          value: "jaeger-agent.tv-discovery.svc.cluster.local:6831"
        resources:
          requests:
            cpu: 500m
            memory: 1Gi
          limits:
            cpu: 2000m
            memory: 4Gi
        livenessProbe:
          httpGet:
            path: /health
            port: 9090
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 9090
          initialDelaySeconds: 5
          periodSeconds: 5
        volumeMounts:
        - name: config
          mountPath: /app/config
          readOnly: true
      volumes:
      - name: config
        configMap:
          name: agent-orchestrator-config

---
apiVersion: v1
kind: Service
metadata:
  name: agent-orchestrator
  namespace: tv-discovery
spec:
  selector:
    app: agent-orchestrator
  ports:
  - name: grpc
    port: 50051
    targetPort: 50051
  - name: metrics
    port: 9090
    targetPort: 9090
  type: ClusterIP
```

---

## 6. Monitoring and Observability

### 6.1 Metrics Collection

**src/utils/MetricsCollector.ts:**

```typescript
import { Counter, Histogram, Gauge, Registry } from 'prom-client';

export class MetricsCollector {
  private registry: Registry;

  // Agent execution metrics
  private agentExecutionDuration: Histogram;
  private agentExecutionTotal: Counter;
  private agentExecutionErrors: Counter;

  // Tool execution metrics
  private toolExecutionDuration: Histogram;
  private toolExecutionTotal: Counter;
  private toolExecutionErrors: Counter;

  // Queue metrics
  private queueDepth: Gauge;
  private queueWaitTime: Histogram;

  // Agent metrics
  private activeAgents: Gauge;

  constructor() {
    this.registry = new Registry();

    // Initialize metrics
    this.agentExecutionDuration = new Histogram({
      name: 'agent_execution_duration_ms',
      help: 'Agent execution duration in milliseconds',
      labelNames: ['agent_name', 'success'],
      buckets: [10, 50, 100, 500, 1000, 5000, 10000]
    });

    this.agentExecutionTotal = new Counter({
      name: 'agent_execution_total',
      help: 'Total agent executions',
      labelNames: ['agent_name', 'success']
    });

    this.agentExecutionErrors = new Counter({
      name: 'agent_execution_errors_total',
      help: 'Total agent execution errors',
      labelNames: ['agent_name', 'error_type']
    });

    this.toolExecutionDuration = new Histogram({
      name: 'tool_execution_duration_ms',
      help: 'Tool execution duration in milliseconds',
      labelNames: ['tool_name', 'success'],
      buckets: [10, 50, 100, 500, 1000, 5000]
    });

    this.toolExecutionTotal = new Counter({
      name: 'tool_execution_total',
      help: 'Total tool executions',
      labelNames: ['tool_name', 'success']
    });

    this.toolExecutionErrors = new Counter({
      name: 'tool_execution_errors_total',
      help: 'Total tool execution errors',
      labelNames: ['tool_name', 'error_type']
    });

    this.queueDepth = new Gauge({
      name: 'task_queue_depth',
      help: 'Current task queue depth',
      labelNames: ['priority']
    });

    this.queueWaitTime = new Histogram({
      name: 'task_queue_wait_time_ms',
      help: 'Task queue wait time in milliseconds',
      buckets: [10, 50, 100, 500, 1000, 5000]
    });

    this.activeAgents = new Gauge({
      name: 'active_agents',
      help: 'Number of active agents',
      labelNames: ['agent_name']
    });

    // Register metrics
    this.registry.registerMetric(this.agentExecutionDuration);
    this.registry.registerMetric(this.agentExecutionTotal);
    this.registry.registerMetric(this.agentExecutionErrors);
    this.registry.registerMetric(this.toolExecutionDuration);
    this.registry.registerMetric(this.toolExecutionTotal);
    this.registry.registerMetric(this.toolExecutionErrors);
    this.registry.registerMetric(this.queueDepth);
    this.registry.registerMetric(this.queueWaitTime);
    this.registry.registerMetric(this.activeAgents);
  }

  recordAgentExecution(agentName: string, duration: number, success: boolean): void {
    this.agentExecutionDuration.observe(
      { agent_name: agentName, success: success.toString() },
      duration
    );

    this.agentExecutionTotal.inc({
      agent_name: agentName,
      success: success.toString()
    });
  }

  recordToolExecution(toolName: string, duration: number, success: boolean): void {
    this.toolExecutionDuration.observe(
      { tool_name: toolName, success: success.toString() },
      duration
    );

    this.toolExecutionTotal.inc({
      tool_name: toolName,
      success: success.toString()
    });
  }

  recordAgentError(agentName: string, errorType: string): void {
    this.agentExecutionErrors.inc({
      agent_name: agentName,
      error_type: errorType
    });
  }

  setQueueDepth(priority: number, depth: number): void {
    this.queueDepth.set({ priority: priority.toString() }, depth);
  }

  recordQueueWait(duration: number): void {
    this.queueWaitTime.observe(duration);
  }

  setActiveAgents(agentName: string, count: number): void {
    this.activeAgents.set({ agent_name: agentName }, count);
  }

  getMetrics(): Promise<string> {
    return this.registry.metrics();
  }

  getAverageExecutionTime(): number {
    // Implementation to calculate average execution time
    return 0; // Placeholder
  }
}
```

### 6.2 Grafana Dashboard

**monitoring/grafana-dashboard.json:**

```json
{
  "dashboard": {
    "title": "Agent Orchestrator",
    "panels": [
      {
        "title": "Agent Execution Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(agent_execution_total[5m])",
            "legendFormat": "{{agent_name}} - {{success}}"
          }
        ]
      },
      {
        "title": "Agent Execution Duration (p95)",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(agent_execution_duration_ms_bucket[5m]))",
            "legendFormat": "{{agent_name}}"
          }
        ]
      },
      {
        "title": "Task Queue Depth",
        "type": "graph",
        "targets": [
          {
            "expr": "task_queue_depth",
            "legendFormat": "Priority {{priority}}"
          }
        ]
      },
      {
        "title": "Active Agents",
        "type": "graph",
        "targets": [
          {
            "expr": "active_agents",
            "legendFormat": "{{agent_name}}"
          }
        ]
      }
    ]
  }
}
```

---

**Document Version: 1.0.0**
**Last Updated: December 2025**
**Authors: Implementation Team**
