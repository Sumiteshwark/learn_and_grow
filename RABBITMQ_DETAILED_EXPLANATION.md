# 🚀 **RabbitMQ Implementation in Sentinel - Complete Technical Guide**

## 📋 **Table of Contents**
- [What is RabbitMQ?](#what-is-rabbitmq)
- [Why RabbitMQ in Sentinel?](#why-rabbitmq-in-sentinel)
- [Architecture Overview](#architecture-overview)
- [Core Components](#core-components)
- [Configuration Deep Dive](#configuration-deep-dive)
- [Message Lifecycle](#message-lifecycle)
- [Processing Pipeline](#processing-pipeline)
- [Error Handling & Retry Logic](#error-handling--retry-logic)
- [Dead Letter Exchange](#dead-letter-exchange)
- [Monitoring & Health Checks](#monitoring--health-checks)
- [API Integration](#api-integration)
- [Performance Optimization](#performance-optimization)
- [Migration Strategy](#migration-strategy)
- [Production Deployment](#production-deployment)
- [Troubleshooting Guide](#troubleshooting-guide)

---

## 🤔 **What is RabbitMQ?**

**RabbitMQ** is an **open-source message broker** that implements the Advanced Message Queuing Protocol (AMQP). It provides:

### **Core Features:**
- ✅ **Message Persistence**: Messages survive broker restarts
- ✅ **Routing Flexibility**: Exchange types for different routing patterns
- ✅ **Reliability**: Message acknowledgments and publisher confirms
- ✅ **Scalability**: Clustering and load balancing capabilities
- ✅ **Protocol Support**: AMQP, MQTT, STOMP, HTTP
- ✅ **Management UI**: Web-based monitoring interface
- ✅ **Plugin Architecture**: Extensible with custom plugins

### **Why Not Other Solutions?**
- **vs Redis**: Better for complex routing and guaranteed delivery
- **vs Kafka**: More suitable for traditional messaging patterns
- **vs BullMQ**: Native message broker with AMQP compliance
- **vs SQS**: Self-hosted with full control and customization

---

## 🎯 **Why RabbitMQ in Sentinel?**

### **Sentinel's Requirements:**
```typescript
// Sentinel needs to handle:
// 1. Test execution jobs (potentially long-running)
// 2. High throughput during peak hours
// 3. Message persistence across deployments
// 4. Complex routing scenarios
// 5. Priority-based processing
// 6. Comprehensive error handling
```

### **RabbitMQ Benefits for Sentinel:**
- ✅ **Routing Flexibility**: Direct, topic, headers, and fanout exchanges
- ✅ **Reliability**: Message acknowledgments and persistence
- ✅ **Observability**: Rich management UI and monitoring
- ✅ **Scalability**: Clustering and load balancing
- ✅ **Protocol Support**: Multiple protocols for different use cases
- ✅ **Plugin Ecosystem**: Extendable with custom plugins

### **Real-World Impact:**
```typescript
// Before: Synchronous processing (blocking)
const result = await executeTest(testDefinition); // Blocks API response

// After: Asynchronous processing (non-blocking)
const correlationId = await messageBroker.publishTest(testDefinition);
// API responds immediately with correlationId
// User can check progress/status later
```

---

## 🏗️ **Architecture Overview**

### **Service Architecture:**
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   API Service   │────│    RabbitMQ     │────│ Integrations    │
│   (Port 3000)   │    │  Message Broker │    │   Service       │
│                 │    │                 │    │  (Port 3001)    │
│ ┌─────────────┐ │    │ ┌─────────────┐ │    │                 │
│ │  Publisher  │◄├───►│ │  Exchanges  │◄├───►│ ┌─────────────┐ │
│ │             │ │    │ │  & Queues   │ │    │ │  Consumer    │ │
│ └─────────────┘ │    │ └─────────────┘ │    │ │             │ │
└─────────────────┘    └─────────────────┘    │ └─────────────┘ │
                                              └─────────────────┘
```

### **Data Flow:**
```
1. User submits test → API Service
2. API publishes message → RabbitMQ Exchange
3. Exchange routes message → Appropriate Queue
4. Consumer picks message → Integrations Service
5. Integrations processes → Browser Engine
6. Results published → Response Exchange
7. User queries status → API Service
```

### **Component Responsibilities:**

#### **API Service (Publisher):**
- Accepts test execution requests
- Publishes messages to exchanges with routing keys
- Provides status/result endpoints
- Handles message correlation and tracking

#### **RabbitMQ (Message Broker):**
- Message routing and queuing
- Message persistence and durability
- Connection management
- Monitoring and management interface

#### **Integrations Service (Consumer):**
- Worker processes that consume messages
- Browser automation via Playwright
- Progress reporting and acknowledgments
- Error handling and dead lettering

---

## 🔧 **Core Components**

### **1. Connection Configuration (`rabbitmq.config.ts`)**

```typescript
// Environment-based configuration
export const rabbitMQConfig: RabbitMQConfig = {
  protocol: 'amqp',
  hostname: process.env.RABBITMQ_HOST || 'localhost',
  port: parseInt(process.env.RABBITMQ_PORT || '5672'),
  username: process.env.RABBITMQ_USERNAME || 'guest',
  password: process.env.RABBITMQ_PASSWORD || 'guest',
  vhost: process.env.RABBITMQ_VHOST || '/',
  heartbeat: 60,           // Heartbeat interval
  connection_timeout: 60000, // Connection timeout
  channelMax: 100,         // Max channels per connection
  frameMax: 0,             // Max frame size
};
```

#### **Exchange Types:**
```typescript
export const EXCHANGE_TYPES = {
  direct: 'direct',      // Route based on exact routing key match
  topic: 'topic',        // Route based on pattern matching
  headers: 'headers',    // Route based on message headers
  fanout: 'fanout',      // Route to all bound queues
};
```

### **2. Message Structure (`message.types.ts`)**

#### **Message Data Structure:**
```typescript
interface MessageData {
  testDefinition: TestDefinition;    // What to test
  options: MessageOptions;          // How to run it
  metadata: MessageMetadata;        // Who/when/why
}

interface MessageOptions {
  priority: 'high' | 'medium' | 'low';
  expiration?: number;              // Message TTL
  persistent: boolean;              // Message persistence
  mandatory: boolean;               // Mandatory delivery
  immediate: boolean;               // Immediate delivery
  correlationId?: string;           // Message correlation
  replyTo?: string;                 // Reply queue
}

interface MessageMetadata {
  tenantId: string;                // Multi-tenant support
  userId?: string;                 // Who triggered the test
  requestId?: string;              // Request correlation
  timestamp: string;               // When message was sent
  source?: 'api' | 'scheduler' | 'webhook';
}
```

#### **Message Properties:**
```typescript
interface MessageProperties {
  contentType: 'application/json';
  contentEncoding: 'utf-8';
  headers: {
    'x-test-priority': string;
    'x-tenant-id': string;
    'x-user-id'?: string;
    'x-request-id'?: string;
    'x-retry-count': number;
    'x-max-retries': number;
    'x-original-routing-key': string;
  };
  deliveryMode: 2; // Persistent delivery
  priority: number; // 0-255 priority
  correlationId: string;
  replyTo?: string;
  expiration?: string;
  messageId: string;
  timestamp: number;
  type: string;
  userId: string;
  appId: string;
}
```

### **3. Message Broker Service (`rabbitmq.service.ts`)**

#### **Main Service Interface:**
```typescript
class RabbitMQService {
  async publishTest(testDefinition, options): Promise<string> {
    // 1. Prepare message
    const message = {
      testDefinition,
      options: { priority: 'medium', persistent: true, ... },
      metadata: { tenantId, userId, timestamp: now() }
    };

    // 2. Setup correlation tracking
    const correlationId = generateCorrelationId();

    // 3. Publish to exchange
    await this.channel.publish(
      EXCHANGES.testExecution,
      this.getRoutingKey(options.priority),
      Buffer.from(JSON.stringify(message)),
      {
        persistent: true,
        correlationId,
        headers: this.buildHeaders(message)
      }
    );

    return correlationId;
  }

  async getTestStatus(correlationId): Promise<MessageStatus> {
    // Query message status from tracking store
    return await this.messageTracker.getStatus(correlationId);
  }
}
```

### **4. Message Consumer (`test-consumer.processor.ts`)**

#### **Consumer Processing Pipeline:**
```typescript
export async function testMessageConsumer(channel: Channel, msg: ConsumeMessage) {
  const startTime = Date.now();
  const correlationId = msg.properties.correlationId;
  const retryCount = parseInt(msg.properties.headers['x-retry-count'] || '0');

  try {
    // 1. Parse message
    const messageData = JSON.parse(msg.content.toString());
    const { testDefinition, options } = messageData;

    // 2. Validate message
    validateMessageData(messageData);

    // 3. Process test with timeout protection
    const result = await Promise.race([
      executeTestWithProgress(channel, msg, testDefinition, options),
      createTimeoutPromise(options.timeout || 60000, correlationId)
    ]);

    // 4. Publish results
    await publishTestResult(channel, correlationId, result);

    // 5. Acknowledge message
    channel.ack(msg);

  } catch (error) {
    // Handle errors with retry logic
    await handleConsumerError(channel, msg, error, retryCount);
  }
}
```

---

## ⚙️ **Configuration Deep Dive**

### **Environment Variables:**
```bash
# RabbitMQ Connection
RABBITMQ_HOST=localhost
RABBITMQ_PORT=5672
RABBITMQ_USERNAME=guest
RABBITMQ_PASSWORD=guest
RABBITMQ_VHOST=/

# Queue Configuration
RABBITMQ_QUEUE_HIGH=test.execution.high
RABBITMQ_QUEUE_MEDIUM=test.execution.medium
RABBITMQ_QUEUE_LOW=test.execution.low

# Exchange Configuration
RABBITMQ_EXCHANGE_TEST_EXECUTION=test.execution
RABBITMQ_EXCHANGE_DEAD_LETTER=test.dead.letter
RABBITMQ_EXCHANGE_RESULTS=test.results

# Consumer Settings
RABBITMQ_CONSUMER_PREFETCH=5          # Messages per consumer
RABBITMQ_CONSUMER_TIMEOUT=600000      # 10 minutes default

# Message Settings
RABBITMQ_MESSAGE_TTL=86400000         # 24 hours
RABBITMQ_MAX_RETRIES=3                # Maximum retry attempts
```

### **Exchange and Queue Topology:**
```typescript
const TOPOLOGY = {
  exchanges: {
    testExecution: {
      name: 'test.execution',
      type: 'direct',
      durable: true,
      autoDelete: false
    },
    deadLetter: {
      name: 'test.dead.letter',
      type: 'direct',
      durable: true,
      autoDelete: false
    },
    results: {
      name: 'test.results',
      type: 'direct',
      durable: true,
      autoDelete: false
    }
  },
  queues: {
    high: {
      name: 'test.execution.high',
      durable: true,
      arguments: {
        'x-message-ttl': 86400000,    // 24 hours
        'x-max-priority': 255,
        'x-dead-letter-exchange': 'test.dead.letter'
      }
    },
    medium: {
      name: 'test.execution.medium',
      durable: true,
      arguments: {
        'x-message-ttl': 86400000,
        'x-max-priority': 255,
        'x-dead-letter-exchange': 'test.dead.letter'
      }
    },
    low: {
      name: 'test.execution.low',
      durable: true,
      arguments: {
        'x-message-ttl': 86400000,
        'x-max-priority': 255,
        'x-dead-letter-exchange': 'test.dead.letter'
      }
    }
  }
};
```

### **Priority-Based Routing:**
```typescript
const PRIORITY_ROUTING = {
  high: {
    routingKey: 'test.execution.high',
    queue: 'test.execution.high',
    maxPriority: 255,
    prefetchCount: 10
  },
  medium: {
    routingKey: 'test.execution.medium',
    queue: 'test.execution.medium',
    maxPriority: 255,
    prefetchCount: 5
  },
  low: {
    routingKey: 'test.execution.low',
    queue: 'test.execution.low',
    maxPriority: 255,
    prefetchCount: 2
  }
};
```

---

## 🔄 **Message Lifecycle**

### **Complete Message Flow:**

```
1. PUBLISHED     → Message sent to exchange (unroutable if no bindings)
2. ROUTED        → Exchange routes to appropriate queue(s)
3. QUEUED        → Message stored in queue
4. CONSUMED      → Consumer receives message (unacked state)
5. PROCESSING    → Consumer processes message
6. ACKNOWLEDGED  → Consumer acknowledges successful processing
   └─ NACKED     → Consumer rejects message (retry or dead letter)
      └─ DEAD    → Message moved to dead letter exchange
```

### **Message States:**
```typescript
enum MessageState {
  PUBLISHED    = 'published',    // Message published to exchange
  ROUTED       = 'routed',       // Message routed to queue
  QUEUED       = 'queued',       // Message in queue
  CONSUMED     = 'consumed',     // Message consumed by worker
  PROCESSING   = 'processing',   // Message being processed
  ACKNOWLEDGED = 'acknowledged', // Message successfully processed
  NACKED       = 'nacked',       // Message rejected by consumer
  DEAD         = 'dead',         // Message in dead letter queue
  EXPIRED      = 'expired'       // Message expired
}
```

### **Progress Tracking:**
```typescript
interface MessageProgress {
  correlationId: string;        // Message correlation ID
  state: MessageState;          // Current message state
  percentage?: number;          // Processing percentage (0-100)
  currentStep?: number;         // Current test step
  totalSteps?: number;          // Total test steps
  stepName?: string;           // Current step name
  message?: string;            // Progress message
  updatedAt: string;           // Last update timestamp
  retryCount?: number;         // Number of retries
  errorMessage?: string;       // Error message if failed
}
```

---

## 🔬 **Processing Pipeline**

### **Step-by-Step Execution:**

```typescript
async function processTestMessage(channel: Channel, msg: ConsumeMessage) {
  // 1. Initialize
  const startTime = Date.now();
  const correlationId = msg.properties.correlationId;
  updateProgress(correlationId, { percentage: 0, stepName: 'Initializing' });

  // 2. Parse and validate
  const messageData = JSON.parse(msg.content.toString());
  validateMessageData(messageData);
  updateProgress(correlationId, { percentage: 5, stepName: 'Validation complete' });

  // 3. Setup browser context
  const browser = await setupBrowser();
  updateProgress(correlationId, { percentage: 10, stepName: 'Browser ready' });

  // 4. Execute test steps
  const { testDefinition } = messageData;
  for (let i = 0; i < testDefinition.steps.length; i++) {
    const step = testDefinition.steps[i];
    updateProgress(correlationId, {
      percentage: 10 + (i / testDefinition.steps.length) * 80,
      currentStep: i + 1,
      stepName: step.name || `Step ${i + 1}`
    });

    await executeStep(step, browser);
  }

  // 5. Cleanup
  updateProgress(correlationId, { percentage: 95, stepName: 'Cleaning up' });
  await browser.close();

  // 6. Publish results
  updateProgress(correlationId, { percentage: 100, stepName: 'Complete' });
  await publishResults(channel, correlationId, result);

  // 7. Acknowledge
  channel.ack(msg);
}
```

### **Timeout Protection:**
```typescript
const timeoutPromise = createTimeoutPromise(timeout, correlationId);
const result = await Promise.race([
  executeTestWithProgress(channel, msg, testDefinition, options),
  timeoutPromise  // Will throw if timeout exceeded
]);
```

### **Progress Updates:**
```typescript
async function updateProgress(correlationId: string, progress: Partial<MessageProgress>) {
  const fullProgress = {
    correlationId,
    state: 'processing',
    updatedAt: new Date().toISOString(),
    ...progress
  };

  // Publish progress to results exchange
  await channel.publish(
    EXCHANGES.results,
    `progress.${correlationId}`,
    Buffer.from(JSON.stringify(fullProgress)),
    { persistent: true }
  );

  // Store in tracking cache
  await progressCache.set(correlationId, fullProgress);
}
```

---

## 🚨 **Error Handling & Retry Logic**

### **Error Categorization:**
```typescript
function categorizeError(error: Error, retryCount: number): MessageError {
  if (error.message.includes('timeout')) {
    return {
      code: 'TIMEOUT',
      message: 'Test execution timed out',
      category: 'timeout',
      retryable: retryCount < 3,
      nackType: 'requeue'
    };
  }

  if (error.message.includes('network')) {
    return {
      code: 'NETWORK_ERROR',
      message: 'Network connectivity issue',
      category: 'network',
      retryable: retryCount < 5,
      nackType: 'requeue'
    };
  }

  if (error.message.includes('browser')) {
    return {
      code: 'BROWSER_ERROR',
      message: 'Browser automation error',
      category: 'browser',
      retryable: retryCount < 2,
      nackType: 'requeue'
    };
  }

  if (error.message.includes('validation')) {
    return {
      code: 'VALIDATION_ERROR',
      message: 'Invalid test definition',
      category: 'validation',
      retryable: false,
      nackType: 'dead'
    };
  }

  return {
    code: 'UNKNOWN_ERROR',
    message: error.message,
    category: 'unknown',
    retryable: retryCount < 3,
    nackType: 'requeue'
  };
}
```

### **Retry Strategy:**

#### **Exponential Backoff:**
```typescript
const retryStrategy = {
  network: {
    attempts: 5,
    delay: (attempt: number) => Math.min(1000 * Math.pow(2, attempt), 30000)
  },
  timeout: {
    attempts: 3,
    delay: (attempt: number) => Math.min(2000 * Math.pow(2, attempt), 60000)
  },
  browser: {
    attempts: 2,
    delay: (attempt: number) => 3000
  },
  system: {
    attempts: 1,
    delay: (attempt: number) => 60000
  },
  validation: {
    attempts: 0,
    delay: () => 0
  }
};
```

#### **NACK with Requeue:**
```typescript
async function handleConsumerError(channel: Channel, msg: ConsumeMessage, error: Error, retryCount: number) {
  const messageError = categorizeError(error, retryCount);

  if (messageError.retryable && retryCount < messageError.maxRetries) {
    // Requeue with delay
    const delay = calculateRetryDelay(messageError.category, retryCount);
    await channel.publish(
      EXCHANGES.retry,
      msg.fields.routingKey,
      msg.content,
      {
        ...msg.properties,
        headers: {
          ...msg.properties.headers,
          'x-retry-count': retryCount + 1,
          'x-delay': delay
        }
      }
    );
    channel.ack(msg);
  } else {
    // Move to dead letter
    channel.nack(msg, false, false);
  }
}
```

### **Dead Letter Exchange Integration:**
```typescript
// Dead letter consumer
async function deadLetterConsumer(channel: Channel, msg: ConsumeMessage) {
  const messageData = JSON.parse(msg.content.toString());
  const error = msg.properties.headers['x-death']?.[0]?.reason || 'unknown';

  // Store in dead letter store
  await deadLetterStore.add({
    originalMessage: messageData,
    routingKey: msg.fields.routingKey,
    exchange: msg.fields.exchange,
    error: error,
    timestamp: new Date().toISOString(),
    headers: msg.properties.headers
  });

  channel.ack(msg);
}
```

---

## ⚰️ **Dead Letter Exchange**

### **Purpose:**
- Store permanently failed messages
- Prevent infinite retry loops
- Enable manual inspection and recovery
- Track failure patterns for debugging

### **Implementation:**
```typescript
class DeadLetterExchange {
  async setup(channel: Channel) {
    // Declare dead letter exchange
    await channel.assertExchange('test.dead.letter', 'direct', { durable: true });

    // Declare dead letter queue
    await channel.assertQueue('test.dead.letter.queue', {
      durable: true,
      arguments: {
        'x-message-ttl': 7 * 24 * 60 * 60 * 1000, // 7 days
        'x-dead-letter-exchange': '', // No further dead lettering
      }
    });

    // Bind dead letter queue
    await channel.bindQueue('test.dead.letter.queue', 'test.dead.letter', '#');
  }

  async getFailedMessages(limit: number = 50, offset: number = 0) {
    const messages = await this.store.find({})
      .sort({ timestamp: -1 })
      .limit(limit)
      .skip(offset);

    return messages.map(msg => ({
      id: msg._id,
      originalMessage: msg.originalMessage,
      routingKey: msg.routingKey,
      error: msg.error,
      timestamp: msg.timestamp,
      retryCount: msg.headers['x-retry-count'] || 0
    }));
  }

  async requeueFailedMessage(messageId: string) {
    const dlqMessage = await this.store.findById(messageId);
    if (!dlqMessage) throw new Error('DLQ message not found');

    // Republish to original exchange
    await this.channel.publish(
      dlqMessage.exchange,
      dlqMessage.routingKey,
      Buffer.from(JSON.stringify(dlqMessage.originalMessage)),
      {
        persistent: true,
        headers: {
          ...dlqMessage.headers,
          'x-retry-count': 0, // Reset retry count
          'x-requeued-from-dlq': true
        }
      }
    );

    // Remove from DLQ
    await this.store.findByIdAndDelete(messageId);
  }
}
```

---

## 📊 **Monitoring & Health Checks**

### **Queue Metrics:**
```typescript
interface QueueMetrics {
  queue: string;
  messages: number;              // Total messages
  messages_ready: number;        // Ready to consume
  messages_unacknowledged: number; // Being processed
  consumers: number;             // Active consumers
  publish_rate: number;          // Messages published per second
  delivery_rate: number;         // Messages delivered per second
  ack_rate: number;              // Messages acknowledged per second
  redeliver_rate: number;        // Messages redelivered per second
  memory_usage: number;          // Queue memory usage
  message_bytes: number;         // Total message size
}
```

### **Health Check Logic:**
```typescript
function checkRabbitMQHealth(metrics: QueueMetrics[]): 'healthy' | 'degraded' | 'unhealthy' {
  const totalQueued = metrics.reduce((sum, q) => sum + q.messages_ready, 0);
  const totalUnacked = metrics.reduce((sum, q) => sum + q.messages_unacknowledged, 0);
  const totalConsumers = metrics.reduce((sum, q) => sum + q.consumers, 0);

  // Check queue depth
  if (totalQueued > 1000) return 'degraded';
  if (totalQueued > 5000) return 'unhealthy';

  // Check unacknowledged messages
  if (totalUnacked > 100) return 'degraded';
  if (totalUnacked > 500) return 'unhealthy';

  // Check consumer availability
  if (totalConsumers === 0) return 'unhealthy';

  // Check processing rates
  const avgDeliveryRate = metrics.reduce((sum, q) => sum + q.delivery_rate, 0) / metrics.length;
  if (avgDeliveryRate < 1) return 'degraded';

  return 'healthy';
}
```

### **Real-time Monitoring:**
```typescript
// Connection event listeners
connection.on('error', (err) => {
  console.error('RabbitMQ connection error:', err.message);
});

connection.on('close', () => {
  console.warn('RabbitMQ connection closed');
});

// Channel event listeners
channel.on('error', (err) => {
  console.error('Channel error:', err.message);
});

// Consumer event listeners
channel.consume(queueName, async (msg) => {
  if (msg) {
    console.log(`Received message: ${msg.properties.correlationId}`);
  }
}, { noAck: false });
```

---

## 🔗 **API Integration**

### **REST API Endpoints:**

#### **Execute Test:**
```http
POST /api/v1/tests/execute
Authorization: Bearer <api-key>
Content-Type: application/json

{
  "testDefinition": { /* test data */ },
  "options": {
    "priority": "high",
    "timeout": 30000,
    "tags": ["smoke", "critical"]
  }
}

Response:
{
  "success": true,
  "message": "Test execution started successfully",
  "timestamp": "2024-01-01T10:00:00.000Z",
  "data": {
    "correlationId": "msg-12345"
  }
}
```

#### **Check Status:**
```http
GET /api/v1/tests/correlation/msg-12345/status
Authorization: Bearer <api-key>

Response:
{
  "success": true,
  "message": "Test status retrieved successfully",
  "timestamp": "2024-01-01T10:00:05.000Z",
  "data": {
    "correlationId": "msg-12345",
    "status": "processing",
    "priority": "high",
    "progress": {
      "percentage": 65,
      "currentStep": 3,
      "totalSteps": 5,
      "stepName": "Login verification",
      "message": "Checking user credentials"
    },
    "startedAt": "2024-01-01T10:00:00.000Z"
  }
}
```

#### **Queue Management:**
```http
GET /api/v1/rabbitmq/queues/stats      # Get queue metrics
POST /api/v1/rabbitmq/queues/purge     # Purge queue
GET /api/v1/rabbitmq/exchanges         # List exchanges
GET /api/v1/rabbitmq/dead-letters      # Get dead letter messages
POST /api/v1/rabbitmq/dead-letters/{id}/requeue  # Requeue dead letter
```

### **Service Integration:**
```typescript
// In test execution service
const messageBroker = getRabbitMQService();

const correlationId = await messageBroker.publishTest(testDefinition, {
  priority: 'high',
  userId: request.auth?.userId,
  requestId: request.headers['x-request-id']
});

// Status checking
const status = await messageBroker.getTestStatus(correlationId);
```

---

## ⚡ **Performance Optimization**

### **RabbitMQ Configuration:**
```yaml
# rabbitmq.conf
# Memory optimization
vm_memory_high_watermark.relative = 0.6
vm_memory_high_watermark_paging_ratio = 0.5

# Disk free limit
disk_free_limit.relative = 2.0

# Queue settings
queue_index_embed_msgs_below = 4096

# Connection settings
tcp_listen_options.backlog = 128
tcp_listen_options.nodelay = true

# Performance tuning
hipe_compile = true
```

### **Consumer Concurrency:**
```typescript
// Optimal prefetch count based on workload
const optimalPrefetch = Math.max(1, Math.floor(os.cpus().length / 2));

const consumerOptions = {
  prefetch: optimalPrefetch,
  noAck: false,
  exclusive: false
};

// Multiple consumers per queue
for (let i = 0; i < optimalPrefetch; i++) {
  await channel.consume(queueName, messageHandler, consumerOptions);
}
```

### **Connection Pooling:**
```typescript
class RabbitMQConnectionPool {
  private pool: amqp.Connection[] = [];
  private maxConnections: number = 10;

  async getConnection(): Promise<amqp.Connection> {
    // Return existing connection or create new one
    if (this.pool.length > 0) {
      return this.pool.pop()!;
    }

    if (this.pool.length < this.maxConnections) {
      const connection = await amqp.connect(this.config);
      this.setupConnectionEventHandlers(connection);
      return connection;
    }

    // Wait for available connection
    return new Promise((resolve) => {
      const checkPool = () => {
        if (this.pool.length > 0) {
          resolve(this.pool.pop()!);
        } else {
          setTimeout(checkPool, 100);
        }
      };
      checkPool();
    });
  }

  releaseConnection(connection: amqp.Connection) {
    this.pool.push(connection);
  }
}
```

### **Message Batching:**
```typescript
class MessageBatchPublisher {
  private batch: any[] = [];
  private batchSize: number = 100;
  private batchTimeout: number = 5000; // 5 seconds

  async addToBatch(message: any, routingKey: string) {
    this.batch.push({ message, routingKey });

    if (this.batch.length >= this.batchSize) {
      await this.flush();
    }
  }

  async flush() {
    if (this.batch.length === 0) return;

    // Publish all messages in batch
    const publishPromises = this.batch.map(({ message, routingKey }) =>
      this.channel.publish(
        this.exchange,
        routingKey,
        Buffer.from(JSON.stringify(message)),
        { persistent: true }
      )
    );

    await Promise.all(publishPromises);
    this.batch = [];
  }
}
```

---

## 🔄 **Migration Strategy**

### **Hybrid Approach:**
```typescript
class HybridMessageBrokerService {
  constructor(
    private rabbitMQService: RabbitMQService,
    private fallbackService: DirectTestExecutorService
  ) {}

  async publishTest(testDefinition, options) {
    // Check if RabbitMQ is enabled
    if (process.env.USE_RABBITMQ === 'true') {
      try {
        return await this.rabbitMQService.publishTest(testDefinition, options);
      } catch (rabbitError) {
        console.warn('RabbitMQ unavailable, falling back to direct execution');
        return await this.fallbackService.executeTest(testDefinition, options);
      }
    } else {
      // Fallback to direct execution
      return await this.fallbackService.executeTest(testDefinition, options);
    }
  }
}
```

### **Gradual Rollout:**
```bash
# Phase 1: Enable RabbitMQ for low-priority tests
export RABBITMQ_LOW_PRIORITY_ONLY=true

# Phase 2: Enable for all tests
export USE_RABBITMQ=true

# Phase 3: Disable direct execution
export USE_DIRECT_EXECUTION=false

# Phase 4: Enable advanced features
export RABBITMQ_ADVANCED_ROUTING=true
export RABBITMQ_DEAD_LETTER_PROCESSING=true
```

### **Rollback Plan:**
```typescript
// Automatic fallback if RabbitMQ is unavailable
async function publishWithFallback(testDefinition, options) {
  try {
    return await rabbitMQService.publishTest(testDefinition, options);
  } catch (rabbitError) {
    console.warn('RabbitMQ unavailable, falling back to direct execution');
    return await directService.executeTest(testDefinition, options);
  }
}
```

---

## 🚀 **Production Deployment**

### **Infrastructure Setup:**
```yaml
# docker-compose.yml for production
version: '3.8'
services:
  rabbitmq:
    image: rabbitmq:3.12-management-alpine
    environment:
      RABBITMQ_DEFAULT_USER: sentinel
      RABBITMQ_DEFAULT_PASS: ${RABBITMQ_PASSWORD}
      RABBITMQ_DEFAULT_VHOST: sentinel
    volumes:
      - rabbitmq-data:/var/lib/rabbitmq
      - ./rabbitmq/rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf
      - ./rabbitmq/enabled_plugins:/etc/rabbitmq/enabled_plugins
    ports:
      - "5672:5672"   # AMQP port
      - "15672:15672" # Management UI
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "ping"]
      interval: 30s
      timeout: 10s
      retries: 5

  api:
    build: ./packages/@sentinel/services/api
    environment:
      - RABBITMQ_HOST=rabbitmq
      - RABBITMQ_PORT=5672
      - RABBITMQ_USERNAME=sentinel
      - RABBITMQ_PASSWORD=${RABBITMQ_PASSWORD}
      - USE_RABBITMQ=true
    depends_on:
      rabbitmq:
        condition: service_healthy

  integrations:
    build: ./packages/@sentinel/integrations
    environment:
      - RABBITMQ_HOST=rabbitmq
      - RABBITMQ_PORT=5672
      - RABBITMQ_USERNAME=sentinel
      - RABBITMQ_PASSWORD=${RABBITMQ_PASSWORD}
    depends_on:
      rabbitmq:
        condition: service_healthy

volumes:
  rabbitmq-data:
```

### **Kubernetes Deployment:**
```yaml
# RabbitMQ StatefulSet
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sentinel-rabbitmq
spec:
  replicas: 3
  serviceName: sentinel-rabbitmq
  template:
    spec:
      containers:
      - name: rabbitmq
        image: rabbitmq:3.12-management-alpine
        ports:
        - containerPort: 5672
          name: amqp
        - containerPort: 15672
          name: management
        volumeMounts:
        - name: rabbitmq-data
          mountPath: /var/lib/rabbitmq
        resources:
          requests:
            memory: "256Mi"
            cpu: "100m"
          limits:
            memory: "512Mi"
            cpu: "200m"
        env:
        - name: RABBITMQ_ERLANG_COOKIE
          value: "sentinel-cluster-cookie"
        - name: RABBITMQ_NODENAME
          value: "rabbit@sentinel-rabbitmq"
```

### **Clustering Configuration:**
```yaml
# Clustering for high availability
apiVersion: v1
kind: ConfigMap
metadata:
  name: rabbitmq-config
data:
  rabbitmq.conf: |
    cluster_formation.peer_discovery_backend = k8s
    cluster_formation.k8s.host = kubernetes.default.svc.cluster.local
    cluster_formation.k8s.address_type = hostname
    cluster_formation.k8s.service_name = sentinel-rabbitmq
    cluster_formation.node_cleanup.interval = 10
    cluster_formation.node_cleanup.only_log_warning = false
```

---

## 🔧 **Troubleshooting Guide**

### **Common Issues:**

#### **1. Connection Errors:**
```bash
# Check RabbitMQ connectivity
curl -u guest:guest http://localhost:15672/api/aliveness-test/

# Check connection status
rabbitmqctl list_connections

# View connection details
rabbitmqctl list_consumers
```

#### **2. Queue Backlog:**
```typescript
// Check queue depth
const queueInfo = await channel.assertQueue(queueName, { passive: true });
console.log('Queue depth:', queueInfo.messageCount);

// Monitor consumer lag
const consumerInfo = await channel.checkQueue(queueName);
console.log('Unacked messages:', consumerInfo.consumerCount);

// Scale consumers dynamically
if (queueInfo.messageCount > 100) {
  await addMoreConsumers(queueName);
}
```

#### **3. Message Loss:**
```typescript
// Enable publisher confirms
await channel.confirmChannel();

// Publish with mandatory flag
await channel.publish(exchange, routingKey, content, {
  mandatory: true,
  persistent: true
});

// Handle unroutable messages
channel.on('return', (msg) => {
  console.error('Message returned:', msg.properties.correlationId);
});
```

#### **4. Consumer Performance:**
```typescript
// Monitor consumer utilization
const consumerStats = await getConsumerStats();
consumerStats.forEach(stat => {
  if (stat.messagesPerSecond < 10) {
    console.warn(`Slow consumer: ${stat.consumerTag}`);
  }
});

// Adjust prefetch count
await channel.prefetch(10); // Increase for better throughput

// Monitor memory usage
const memUsage = process.memoryUsage();
if (memUsage.heapUsed > 500 * 1024 * 1024) { // 500MB
  console.warn('High memory usage detected');
}
```

### **Debug Commands:**
```bash
# Check RabbitMQ status
rabbitmqctl status

# List queues and their status
rabbitmqctl list_queues name messages consumers

# List exchanges
rabbitmqctl list_exchanges

# List bindings
rabbitmqctl list_bindings

# Check cluster status
rabbitmqctl cluster_status

# View logs
docker logs sentinel-rabbitmq
```

### **Performance Tuning:**
```typescript
// Optimize channel usage
const channelPool = new ChannelPool(maxChannels);

async function getChannel(): Promise<Channel> {
  return await channelPool.acquire();
}

// Connection tuning
const connection = await amqp.connect({
  heartbeat: 60,
  connection_timeout: 60000,
  channelMax: 100,
  frameMax: 0x1000,
});

// Queue optimization
await channel.assertQueue(queueName, {
  durable: true,
  arguments: {
    'x-max-priority': 255,
    'x-message-ttl': 86400000,
    'x-overflow': 'reject-publish',
    'x-queue-mode': 'lazy' // For large queues
  }
});
```

---

## 🎯 **Best Practices**

### **1. Message Design:**
- Keep messages small and focused
- Include comprehensive metadata
- Use correlation IDs for tracking
- Handle failures gracefully
- Set appropriate priorities

### **2. Connection Management:**
- Use connection pooling
- Implement proper error handling
- Set up heartbeats
- Monitor connection health
- Handle reconnections gracefully

### **3. Error Handling:**
- Implement proper retry strategies
- Use dead letter exchanges
- Categorize errors appropriately
- Provide detailed error information
- Set up comprehensive logging

### **4. Performance:**
- Use appropriate prefetch counts
- Implement message batching
- Monitor queue depths
- Set up proper indexing
- Use lazy queues for large datasets

### **5. Monitoring:**
- Set up comprehensive alerting
- Monitor queue depths and rates
- Track consumer performance
- Alert on connection issues
- Monitor message processing times

---

## 📚 **Additional Resources**

- [RabbitMQ Official Documentation](https://www.rabbitmq.com/documentation.html)
- [RabbitMQ Tutorials](https://www.rabbitmq.com/tutorials/)
- [RabbitMQ Management UI](https://www.rabbitmq.com/management.html)
- [AMQP 0-9-1 Protocol](https://www.rabbitmq.com/amqp-0-9-1-reference.html)
- [RabbitMQ Best Practices](https://www.rabbitmq.com/relocate.html)

---

## 🎉 **Conclusion**

RabbitMQ provides Sentinel with a robust, scalable message broker that can handle thousands of concurrent test executions while maintaining reliability and observability. The implementation includes:

- ✅ **Flexible routing** with multiple exchange types
- ✅ **Message persistence** and acknowledgments
- ✅ **Real-time monitoring** through management UI
- ✅ **Clustering support** for high availability
- ✅ **Error handling** with dead letter exchanges
- ✅ **Production readiness** with health checks and metrics
- ✅ **Migration support** with hybrid execution modes

This RabbitMQ implementation enables Sentinel to process test executions asynchronously, providing better user experience and system reliability for large-scale test automation workflows.
