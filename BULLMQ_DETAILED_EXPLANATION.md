# 🚀 **BullMQ Complete Technical Guide**

## 📋 **Table of Contents**
- [What is BullMQ?](#what-is-bullmq)
- [Why Choose BullMQ?](#why-choose-bullmq)
- [Core API Methods and Terms](#core-api-methods-and-terms)
- [Installation and Setup](#installation-and-setup)
- [Core Concepts](#core-concepts)
- [Basic Usage](#basic-usage)
- [Advanced Features](#advanced-features)
- [Job Management](#job-management)
- [Worker Management](#worker-management)
- [Error Handling and Retries](#error-handling-and-retries)
- [Priority Queues](#priority-queues)
- [Real-time Monitoring](#real-time-monitoring)
- [Performance Optimization](#performance-optimization)
- [Security Best Practices](#security-best-practices)
- [Production Deployment](#production-deployment)
- [Troubleshooting](#troubleshooting)
- [Use Cases](#use-cases)
- [Comparison with Alternatives](#comparison-with-alternatives)
- [Best Practices](#best-practices)
- [API Reference](#api-reference)

---

## 🤔 **What is BullMQ?**

**BullMQ** is a modern, Redis-backed job queue library for Node.js applications that provides advanced job management, real-time monitoring, and fault tolerance. It's the successor to the popular Bull library, built specifically for modern JavaScript/TypeScript applications.

### **Key Characteristics:**
- ✅ **Redis-Backed**: Uses Redis for job storage and state management
- ✅ **TypeScript Support**: Full type safety and IntelliSense support
- ✅ **Advanced Job Management**: Priorities, retries, scheduling, dead letter queues
- ✅ **Real-time Monitoring**: Live job status updates and progress tracking
- ✅ **Fault Tolerance**: Automatic recovery and distributed processing
- ✅ **High Performance**: Optimized for high-throughput scenarios
- ✅ **Production Ready**: Battle-tested in production environments

### **Architecture Overview:**
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Producers     │────│     Redis       │────│   Consumers     │
│  (API, Services)│    │  (Job Storage)  │    │   (Workers)     │
│                 │    │                 │    │                 │
│ ┌─────────────┐ │    │ ┌─────────────┐ │    │ ┌─────────────┐ │
│ │   Queues    │◄├───►│ │ Job States  │◄├───►│ │ Processing  │ │
│ │             │ │    │ │             │ │    │ │             │ │
│ └─────────────┘ │    │ └─────────────┘ │    │ └─────────────┘ │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

---

## 📖 **Theory & Concepts**

### **Background Theory**

#### **What is a Job Queue?**
A **job queue** is a data structure that manages units of work (jobs) in a First-In-First-Out (FIFO) manner, but with advanced features like prioritization, scheduling, and distributed processing. Job queues enable:

- **Asynchronous Processing**: Decouple task submission from execution
- **Scalability**: Distribute work across multiple workers
- **Reliability**: Persist jobs across system failures
- **Load Balancing**: Distribute work based on capacity

#### **Why Redis for Job Queues?**
Redis provides the perfect foundation for job queues due to its:

- **In-Memory Storage**: Sub-millisecond access times
- **Persistence**: Data survives server restarts (RDB/AOF)
- **Atomic Operations**: Thread-safe operations
- **Pub/Sub**: Real-time event notifications
- **Data Structures**: Lists, Sets, Sorted Sets, Hashes
- **Clustering**: Horizontal scaling and high availability

#### **Distributed Systems Principles**
BullMQ implements several distributed systems patterns:

- **Producer-Consumer Pattern**: Producers add jobs, consumers process them
- **Leader Election**: Workers compete for jobs using Redis locks
- **Circuit Breaker**: Automatic failure handling and recovery
- **Event Sourcing**: Complete audit trail of job state changes
- **Saga Pattern**: Complex workflows with compensation logic

---

### **Core Terminology**

#### **1. Queue**
```typescript
// Primary container for jobs
const queue = new Queue('email-queue', { connection: redisConfig });
```
- **Definition**: A named collection of jobs waiting to be processed
- **Purpose**: Organizes jobs by type and processing requirements
- **Characteristics**: FIFO with priority support, persistent storage

#### **2. Job**
```typescript
// Unit of work
const job = await queue.add('send-email', {
  to: 'user@example.com',
  subject: 'Welcome!'
});
```
- **Definition**: A single unit of work with data, options, and metadata
- **Components**:
  - **Data**: Payload containing job-specific information
  - **Options**: Configuration (priority, retries, scheduling)
  - **Metadata**: System-generated info (ID, timestamps, attempts)

#### **3. Worker**
```typescript
// Job processor
const worker = new Worker('email-queue', async (job) => {
  return await sendEmail(job.data);
});
```
- **Definition**: A process that consumes and executes jobs from queues
- **Responsibilities**: Job execution, progress reporting, error handling
- **Characteristics**: Concurrent processing, automatic scaling

#### **4. Producer**
```typescript
// Job creator
await queue.add('process-file', fileData);
```
- **Definition**: Any application component that creates jobs
- **Examples**: API endpoints, scheduled tasks, event handlers
- **Role**: Initiates work without blocking execution

#### **5. Consumer**
```typescript
// Job processor (same as Worker)
const consumer = new Worker('queue-name', processor);
```
- **Definition**: Component that receives and processes jobs
- **Pattern**: Pull-based consumption from queues
- **Scaling**: Multiple consumers can process same queue

---

### **Job Lifecycle States**

#### **Complete State Flow:**
```
CREATED → WAITING → ACTIVE → [COMPLETED | FAILED | DELAYED]
    ↑       ↑         ↑            ↓
    └───────┴─────────┴─────────── RETRY
```

#### **State Definitions:**

**1. WAITING** (Initial State)
```typescript
// Job created and queued for processing
await queue.add('task', data); // Job enters WAITING state
```
- **Characteristics**: Job stored in Redis, waiting for worker
- **Transitions**: → ACTIVE (when worker picks up job)
- **Storage**: Redis sorted set ordered by priority

**2. ACTIVE** (Processing State)
```typescript
// Worker processing the job
const worker = new Worker('queue', async (job) => {
  // Job is now ACTIVE
  await processJob(job);
});
```
- **Characteristics**: Job locked by worker, being executed
- **Time Limits**: Lock duration prevents duplicate processing
- **Progress**: Can report progress updates

**3. COMPLETED** (Success State)
```typescript
// Job finished successfully
return { success: true, result: data }; // Job moves to COMPLETED
```
- **Characteristics**: Job finished without errors
- **Retention**: Configurable retention period
- **Cleanup**: Automatic removal after retention period

**4. FAILED** (Error State)
```typescript
// Job failed during processing
throw new Error('Processing failed'); // Job moves to FAILED
```
- **Characteristics**: Job encountered an error
- **Retry Logic**: May retry based on configuration
- **Dead Letter**: Moves to DLQ if max retries exceeded

**5. DELAYED** (Scheduled State)
```typescript
// Job scheduled for future execution
await queue.add('task', data, { delay: 60000 }); // 1 minute delay
```
- **Characteristics**: Job waits until scheduled time
- **Storage**: Redis sorted set ordered by execution time

**6. PAUSED** (Suspended State)
```typescript
// Queue temporarily stopped
await queue.pause(); // All jobs enter PAUSED state
```
- **Characteristics**: Queue processing suspended
- **Recovery**: Jobs resume when queue is unpaused

---

### **Advanced Concepts**

#### **1. Priority Queues**
```typescript
// Jobs processed in priority order (higher number = higher priority)
await queue.add('urgent', data, { priority: 10 });    // Highest
await queue.add('normal', data, { priority: 5 });     // Medium
await queue.add('background', data, { priority: 1 }); // Lowest
```
- **Mechanism**: Redis sorted sets with priority scoring
- **Use Cases**: Critical tasks, user-facing vs background work
- **Performance**: Slight overhead for priority ordering

#### **2. Job Dependencies**
```typescript
// Job B waits for Job A completion
const jobA = await queue.add('step1', dataA);
const jobB = await queue.add('step2', dataB, {
  parent: { id: jobA.id, queue: 'my-queue' }
});
```
- **Types**:
  - **Parent-Child**: Child waits for parent completion
  - **Multi-Parent**: Child waits for multiple parents
- **Use Cases**: Complex workflows, data pipelines

#### **3. Recurring Jobs**
```typescript
// Cron-based repetition
await queue.add('daily-report', data, {
  repeat: { cron: '0 9 * * *' } // Every day at 9 AM
});

// Interval-based repetition
await queue.add('health-check', data, {
  repeat: { every: 30000 } // Every 30 seconds
});
```
- **Storage**: Separate Redis structures for scheduling
- **Execution**: Automatic job creation at scheduled times

#### **4. Dead Letter Queues (DLQ)**
```typescript
// Failed jobs moved to DLQ after max retries
const dlq = new Queue('dead-letter-queue');
await dlq.add('failed-job', {
  originalJobId: job.id,
  error: error.message,
  failedAt: new Date()
});
```
- **Purpose**: Store permanently failed jobs for inspection
- **Recovery**: Manual reprocessing or analysis
- **Retention**: Configurable cleanup policies

#### **5. Rate Limiting**
```typescript
// Limit job processing rate
const worker = new Worker('api-calls', processor, {
  limiter: {
    max: 100,     // Max jobs
    duration: 1000 // Per second
  }
});
```
- **Types**:
  - **Worker-Level**: Per worker instance
  - **Queue-Level**: Global rate limiting
- **Use Cases**: API rate limiting, resource protection

#### **6. Job Locking**
```typescript
// Prevent duplicate processing
const worker = new Worker('queue', processor, {
  lockDuration: 30000,    // 30 second lock
  lockRenewTime: 15000    // Renew every 15 seconds
});
```
- **Mechanism**: Redis-based distributed locks
- **Purpose**: Prevent race conditions and duplicate processing
- **Recovery**: Stalled job detection and recovery

#### **7. Progress Tracking**
```typescript
// Report job progress
const worker = new Worker('file-upload', async (job) => {
  job.updateProgress(25, { status: 'Uploading' });
  await uploadChunk(chunk1);

  job.updateProgress(50, { status: 'Processing' });
  await processFile();

  job.updateProgress(100, { status: 'Complete' });
  return result;
});
```
- **Real-time**: Progress updates via Redis pub/sub
- **Granular**: Support for step-by-step tracking
- **Persistence**: Progress stored with job state

#### **8. Batch Processing**
```typescript
// Process multiple jobs together
const jobs = await queue.addBulk([
  { name: 'email', data: user1Data },
  { name: 'email', data: user2Data },
  { name: 'email', data: user3Data }
]);
```
- **Efficiency**: Reduce Redis round trips
- **Atomicity**: All-or-nothing job creation
- **Use Cases**: Bulk operations, newsletters

---

### **Architecture Patterns**

#### **1. Producer-Consumer Pattern**
```
Producers → Queue → Consumers
    ↓         ↓         ↓
   API     Storage    Workers
```
- **Decoupling**: Producers and consumers operate independently
- **Scalability**: Scale producers and consumers separately
- **Reliability**: Queue acts as buffer during failures

#### **2. Competing Consumers Pattern**
```
Queue → Worker 1
      → Worker 2
      → Worker 3
```
- **Load Distribution**: Jobs distributed across multiple workers
- **Fault Tolerance**: Workers can fail without affecting others
- **Scalability**: Add more workers to handle increased load

#### **3. Saga Pattern (Complex Workflows)**
```
Job A → Job B → Job C
   ↓       ↓       ↓
Error → Compensation → Rollback
```
- **Long-Running**: Complex multi-step processes
- **Compensation**: Rollback logic for failures
- **State Management**: Track progress across steps

#### **4. Circuit Breaker Pattern**
```typescript
// Automatic failure handling
const worker = new Worker('external-api', async (job) => {
  if (await isCircuitOpen()) {
    throw new Error('Circuit breaker open');
  }

  try {
    return await callExternalAPI(job.data);
  } catch (error) {
    await recordFailure();
    throw error;
  }
});
```
- **Purpose**: Prevent cascade failures
- **States**: Closed (normal), Open (failing), Half-Open (testing)

---

### **Performance Concepts**

#### **1. Throughput**
- **Definition**: Number of jobs processed per unit time
- **Factors**: Worker concurrency, job complexity, Redis performance
- **Optimization**: Connection pooling, batch operations

#### **2. Latency**
- **Definition**: Time from job creation to completion
- **Components**: Queue wait time + processing time
- **Optimization**: Priority queues, worker scaling

#### **3. Concurrency**
- **Definition**: Number of jobs processed simultaneously
- **Limits**: CPU cores, memory, external service limits
- **Optimization**: Optimal concurrency = CPU cores - 1

#### **4. Backpressure**
```typescript
// Prevent queue overflow
const queueSize = await queue.getWaiting().then(jobs => jobs.length);
if (queueSize > MAX_QUEUE_SIZE) {
  // Implement backpressure: reject new jobs or slow down producers
  throw new Error('Queue full');
}
```
- **Purpose**: Prevent system overload
- **Strategies**: Rejection, throttling, buffering

---

### **Data Structures Used**

#### **Redis Data Structures in BullMQ:**

**1. Lists** (`bull:queue:wait`):
```redis
LPUSH bull:email:wait job-id-1
LPUSH bull:email:wait job-id-2
LPOP bull:email:wait  # Worker gets job-id-1
```

**2. Sorted Sets** (`bull:queue:delayed`):
```redis
ZADD bull:email:delayed 1640995200000 job-id-1  # Timestamp as score
ZRANGEBYSCORE bull:email:delayed -inf 1640995200000  # Get due jobs
```

**3. Hashes** (`bull:queue:job-id`):
```redis
HSET bull:email:123 data '{"to":"user@example.com"}'
HSET bull:email:123 opts '{"attempts":3}'
HGETALL bull:email:123
```

**4. Sets** (`bull:queue:active`):
```redis
SADD bull:email:active job-id-1
SREM bull:email:active job-id-1  # Remove when completed
```

**5. Pub/Sub** (`bull:queue:events`):
```redis
PUBLISH bull:email:events '{"event":"completed","jobId":"123"}'
SUBSCRIBE bull:email:events
```

---

### **Security Concepts**

#### **1. Data Sanitization**
```typescript
// Remove sensitive data from jobs
const sanitizeJob = (data: any) => {
  const sanitized = { ...data };
  delete sanitized.password;
  delete sanitized.apiKey;
  delete sanitized.ssn;
  return sanitized;
};
```
- **Purpose**: Prevent sensitive data leakage
- **Implementation**: Input validation and sanitization

#### **2. Access Control**
```typescript
// Role-based job permissions
const secureWorker = new Worker('admin-queue', async (job) => {
  if (!await hasPermission(job.data.userId, 'admin')) {
    throw new Error('Insufficient permissions');
  }
  return await processAdminJob(job);
});
```
- **Purpose**: Prevent unauthorized job processing
- **Implementation**: Authentication and authorization checks

#### **3. Rate Limiting**
```typescript
// Prevent abuse
const rateLimitedWorker = new Worker('api-queue', processor, {
  limiter: {
    max: 100,
    duration: 60000  // 100 requests per minute
  }
});
```
- **Purpose**: Prevent resource exhaustion
- **Implementation**: Token bucket or sliding window algorithms

---

### **Monitoring Concepts**

#### **1. Key Metrics**
- **Queue Depth**: Number of waiting jobs
- **Processing Rate**: Jobs completed per second
- **Error Rate**: Percentage of failed jobs
- **Latency**: Average job processing time

#### **2. Health Checks**
```typescript
// Queue health monitoring
const health = {
  redis: await redis.ping() === 'PONG',
  queue: await queue.getWaiting().then(() => true).catch(() => false),
  workers: activeWorkers.length > 0
};
```

#### **3. Alerting**
- **Threshold Alerts**: Queue depth > threshold
- **Error Alerts**: High failure rate
- **Performance Alerts**: Slow processing times

---

## 🎯 **Why Choose BullMQ?**

### **Advantages Over Traditional Job Queues:**

**1. Modern JavaScript Support**
```typescript
// Full TypeScript support
import { Queue, Worker, Job } from 'bullmq';

interface EmailJobData {
  to: string;
  subject: string;
  body: string;
}

const emailQueue = new Queue<EmailJobData>('email-queue');
// Type-safe job data
```

**2. Advanced Job Control**
```typescript
// Priority queues
await queue.add('email', data, { priority: 10 });

// Delayed execution
await queue.add('report', data, { delay: 60000 });

// Repeat jobs
await queue.add('cleanup', data, {
  repeat: { cron: '0 0 * * *' } // Daily at midnight
});
```

**3. Real-time Monitoring**
```typescript
// Live job updates
queue.on('completed', (jobId, result) => {
  console.log(`Job ${jobId} completed:`, result);
});

queue.on('failed', (jobId, err) => {
  console.log(`Job ${jobId} failed:`, err.message);
});
```

**4. Fault Tolerance**
```typescript
// Automatic retries with backoff
await queue.add('api-call', data, {
  attempts: 3,
  backoff: {
    type: 'exponential',
    delay: 2000
  }
});

// Dead letter queues
const failedJobs = await queue.getJobs(['failed']);
```

### **Performance Benchmarks:**
- **Throughput**: 1000+ jobs/second
- **Latency**: <1ms for job operations
- **Memory Usage**: Efficient Redis storage
- **Scalability**: Horizontal scaling across multiple workers

---

## 🔧 **Core API Methods and Terms**

### **Essential BullMQ Classes and Methods**

BullMQ provides several core classes and methods that developers use to interact with the job queue system:

#### **1. Queue Class**
**Purpose**: Primary class for managing job queues
```typescript
import { Queue } from 'bullmq';

// Create a queue
const queue = new Queue('email-queue', {
  connection: {
    host: 'localhost',
    port: 6379
  },
  defaultJobOptions: {
    removeOnComplete: 100,
    removeOnFail: 50
  }
});

// Add a job
await queue.add('send-email', { email: 'user@example.com', message: 'Hello!' });

// Close the queue
await queue.close();
```

**Key Methods:**
- `add(name, data, options)` - Add a job to the queue
- `addBulk(jobs)` - Add multiple jobs at once
- `getJob(jobId)` - Get a specific job by ID
- `getJobs(types, start, end)` - Get jobs by type and range
- `getJobCounts()` - Get job counts by state
- `obliterate()` - Remove all jobs from queue
- `drain()` - Remove all jobs (delayed, waiting, active)
- `clean(grace, limit, type)` - Clean old jobs
- `close()` - Close the queue connection

#### **2. Worker Class**
**Purpose**: Processes jobs from queues
```typescript
import { Worker } from 'bullmq';

const worker = new Worker('email-queue', async (job) => {
  // Process the job
  console.log('Processing job:', job.id);
  console.log('Job data:', job.data);

  // Simulate work
  await new Promise(resolve => setTimeout(resolve, 1000));

  return { result: 'Email sent successfully' };
}, {
  connection: {
    host: 'localhost',
    port: 6379
  },
  concurrency: 5, // Process 5 jobs simultaneously
  limiter: {
    max: 10, // Max jobs per duration
    duration: 1000 // Duration in milliseconds
  }
});

// Event listeners
worker.on('completed', (job) => {
  console.log(`Job ${job.id} completed`);
});

worker.on('failed', (job, err) => {
  console.log(`Job ${job.id} failed:`, err.message);
});
```

**Key Methods:**
- `run()` - Start processing jobs (called automatically)
- `pause()` - Pause job processing
- `resume()` - Resume job processing
- `close()` - Close the worker
- `isRunning()` - Check if worker is running

#### **3. Job Class**
**Purpose**: Represents a unit of work
```typescript
// Job instance methods (available in worker processor)
async (job) => {
  // Access job properties
  console.log('Job ID:', job.id);
  console.log('Job Name:', job.name);
  console.log('Job Data:', job.data);

  // Update progress
  await job.updateProgress(50);

  // Add log entry
  await job.log('Processing started');

  // Update job data
  await job.update({ processed: true });

  // Move to different queue
  await job.moveToFailed(new Error('Custom error'), 'my-token');

  return result;
}
```

**Key Methods:**
- `updateProgress(progress)` - Update job progress (0-100)
- `log(message)` - Add log entry to job
- `update(data)` - Update job data
- `remove()` - Remove job from queue
- `retry()` - Retry failed job
- `discard()` - Discard job (don't retry)
- `moveToCompleted(returnValue, token)` - Mark as completed
- `moveToFailed(error, token)` - Mark as failed

#### **4. QueueScheduler Class**
**Purpose**: Manages delayed and scheduled jobs
```typescript
import { QueueScheduler } from 'bullmq';

const scheduler = new QueueScheduler('email-queue', {
  connection: {
    host: 'localhost',
    port: 6379
  }
});

// Scheduler automatically manages delayed jobs
// No manual methods needed - it's automatic
```

#### **5. FlowProducer Class**
**Purpose**: Manages job dependencies and workflows
```typescript
import { FlowProducer } from 'bullmq';

const flowProducer = new FlowProducer({
  connection: {
    host: 'localhost',
    port: 6379
  }
});

// Add parent job with children
const flow = await flowProducer.add({
  name: 'process-order',
  queueName: 'order-queue',
  data: { orderId: 123 },
  children: [
    {
      name: 'validate-payment',
      queueName: 'payment-queue',
      data: { orderId: 123 }
    },
    {
      name: 'update-inventory',
      queueName: 'inventory-queue',
      data: { orderId: 123 }
    }
  ]
});
```

**Key Methods:**
- `add(flow)` - Add a flow with parent/child relationships
- `addBulk(flows)` - Add multiple flows
- `getFlow(flowId)` - Get a flow by ID
- `getFlows(types)` - Get flows by type

### **Essential Methods and Operations**

#### **Job Management Methods**
```typescript
// Add jobs with different options
const job1 = await queue.add('urgent-email', data, {
  priority: 10,
  delay: 5000, // 5 seconds delay
  attempts: 3,
  backoff: {
    type: 'exponential',
    delay: 2000
  }
});

const job2 = await queue.add('batch-email', data, {
  repeat: {
    cron: '0 9 * * *' // Every day at 9 AM
  }
});

// Bulk operations
const jobs = await queue.addBulk([
  { name: 'email-1', data: { to: 'user1@example.com' } },
  { name: 'email-2', data: { to: 'user2@example.com' } },
  { name: 'email-3', data: { to: 'user3@example.com' } }
]);
```

#### **Queue Operations**
```typescript
// Get queue statistics
const counts = await queue.getJobCounts();
// Returns: { active: 2, completed: 150, failed: 3, delayed: 5, waiting: 10 }

const waiting = await queue.getJobs(['waiting'], 0, 10);
const active = await queue.getJobs(['active'], 0, 10);
const completed = await queue.getJobs(['completed'], 0, 10);
const failed = await queue.getJobs(['failed'], 0, 10);

// Clean old jobs
await queue.clean(24 * 3600 * 1000, 100, 'completed'); // Clean completed jobs older than 24 hours
await queue.clean(7 * 24 * 3600 * 1000, 50, 'failed'); // Clean failed jobs older than 7 days

// Remove all jobs
await queue.obliterate({ force: true });
```

#### **Worker Control Methods**
```typescript
// Pause and resume
await worker.pause();
await worker.resume();

// Check status
const isRunning = worker.isRunning();
const isPaused = worker.isPaused();

// Get worker stats
const stats = await worker.getWorkerStats();
```

### **Key Terms and Concepts**

#### **1. Job States**
```typescript
// Job lifecycle states
enum JobState {
  WAITING = 'waiting',     // Job is waiting to be processed
  ACTIVE = 'active',       // Job is being processed
  COMPLETED = 'completed', // Job finished successfully
  FAILED = 'failed',       // Job failed
  DELAYED = 'delayed',     // Job is scheduled for later
  STALLED = 'stalled',     // Job processing was interrupted
  PRIORITY = 'priority'    // Job has priority over others
}
```

#### **2. Job Options**
```typescript
interface JobOptions {
  priority?: number;           // Job priority (0-2^53)
  delay?: number;             // Delay in milliseconds
  attempts?: number;          // Maximum retry attempts
  backoff?: {                 // Retry backoff strategy
    type: 'fixed' | 'exponential';
    delay: number;
  };
  lifo?: boolean;             // Last In, First Out
  timeout?: number;           // Job timeout in milliseconds
  jobId?: string;             // Custom job ID
  removeOnComplete?: number;  // Keep completed jobs
  removeOnFail?: number;      // Keep failed jobs
  stackTraceLimit?: number;   // Stack trace limit
}
```

#### **3. Worker Options**
```typescript
interface WorkerOptions {
  concurrency?: number;       // Max concurrent jobs
  limiter?: {                 // Rate limiting
    max: number;
    duration: number;
    groupKey?: string;
  };
  lockDuration?: number;      // Lock duration in milliseconds
  lockRenewTime?: number;     // Lock renewal time
  stalledInterval?: number;   // Stalled check interval
  maxStalledCount?: number;   // Max stalled job count
  autorun?: boolean;          // Auto-start processing
}
```

#### **4. Queue Options**
```typescript
interface QueueOptions {
  connection?: RedisConnection;    // Redis connection
  prefix?: string;                // Key prefix
  defaultJobOptions?: JobOptions; // Default job options
  settings?: {                    // Queue settings
    lockDuration?: number;
    stalledInterval?: number;
    maxStalledCount?: number;
    guardInterval?: number;
    retryProcessDelay?: number;
  };
}
```

#### **5. Flow Options**
```typescript
interface FlowJob {
  name: string;               // Job name
  queueName: string;          // Target queue name
  data?: any;                 // Job data
  prefix?: string;            // Queue prefix
  opts?: JobOptions;          // Job options
  children?: FlowJob[];       // Child jobs
}
```

### **Common Event Types**

#### **Queue Events**
```typescript
queue.on('waiting', (jobId) => {
  console.log(`Job ${jobId} is waiting`);
});

queue.on('active', (job, jobPromise) => {
  console.log(`Job ${job.id} started processing`);
});

queue.on('completed', (job, result) => {
  console.log(`Job ${job.id} completed with result:`, result);
});

queue.on('failed', (job, err) => {
  console.log(`Job ${job.id} failed:`, err.message);
});

queue.on('stalled', (jobId) => {
  console.log(`Job ${jobId} has stalled`);
});
```

#### **Worker Events**
```typescript
worker.on('completed', (job, result) => {
  console.log(`Worker completed job ${job.id}`);
});

worker.on('failed', (job, err) => {
  console.log(`Worker failed job ${job.id}:`, err.message);
});

worker.on('stalled', (jobId) => {
  console.log(`Worker detected stalled job ${jobId}`);
});

worker.on('closing', () => {
  console.log('Worker is closing');
});
```

#### **Job Events**
```typescript
job.on('progress', (progress) => {
  console.log(`Job ${job.id} progress: ${progress}%`);
});

job.on('completed', (result) => {
  console.log(`Job ${job.id} completed:`, result);
});

job.on('failed', (err) => {
  console.log(`Job ${job.id} failed:`, err.message);
});
```

### **Error Handling Methods**

#### **1. onError(callback)**
**Purpose**: Handle worker errors
```typescript
worker.on('error', (err) => {
  console.error('Worker error:', err);
});
```

#### **2. onFailed(callback)**
**Purpose**: Handle job failures
```typescript
worker.on('failed', (job, err) => {
  console.error(`Job ${job.id} failed:`, err.message);

  // Custom retry logic
  if (err.message.includes('temporary')) {
    job.retry();
  }
});
```

#### **3. moveToFailed(error, token)**
**Purpose**: Manually fail a job
```typescript
await job.moveToFailed(new Error('Custom failure reason'), 'my-token');
```

### **Advanced Methods**

#### **1. getState()**
**Purpose**: Get current job state
```typescript
const state = await job.getState();
// Returns: 'waiting', 'active', 'completed', 'failed', 'delayed', etc.
```

#### **2. changePriority(priority)**
**Purpose**: Change job priority
```typescript
await job.changePriority(10);
```

#### **3. extendLock(duration)**
**Purpose**: Extend job lock duration
```typescript
await job.extendLock(30000); // Extend by 30 seconds
```

#### **4. promote()**
**Purpose**: Move delayed job to waiting
```typescript
await job.promote();
```

#### **5. finish()**
**Purpose**: Mark job as finished (internal use)
```typescript
await job.finish();
```

### **Common Patterns**

#### **1. Producer Pattern**
```typescript
class JobProducer {
  constructor(queueName) {
    this.queue = new Queue(queueName, { connection: redisConfig });
  }

  async addJob(name, data, options = {}) {
    try {
      const job = await this.queue.add(name, data, {
        attempts: 3,
        backoff: { type: 'exponential', delay: 2000 },
        removeOnComplete: 100,
        removeOnFail: 50,
        ...options
      });

      console.log(`Added job ${job.id} to queue`);
      return job;
    } catch (error) {
      console.error('Failed to add job:', error);
      throw error;
    }
  }

  async addBulkJobs(jobs) {
    try {
      const addedJobs = await this.queue.addBulk(jobs);
      console.log(`Added ${addedJobs.length} jobs to queue`);
      return addedJobs;
    } catch (error) {
      console.error('Failed to add bulk jobs:', error);
      throw error;
    }
  }

  async close() {
    await this.queue.close();
  }
}
```

#### **2. Consumer Pattern**
```typescript
class JobConsumer {
  constructor(queueName, processor, options = {}) {
    this.worker = new Worker(queueName, processor, {
      concurrency: 5,
      limiter: { max: 10, duration: 1000 },
      ...options
    });

    this.setupEventHandlers();
  }

  setupEventHandlers() {
    this.worker.on('completed', (job) => {
      console.log(`Job ${job.id} completed successfully`);
    });

    this.worker.on('failed', (job, err) => {
      console.error(`Job ${job.id} failed: ${err.message}`);
    });

    this.worker.on('stalled', (jobId) => {
      console.log(`Job ${jobId} has stalled`);
    });
  }

  async pause() {
    await this.worker.pause();
  }

  async resume() {
    await this.worker.resume();
  }

  async close() {
    await this.worker.close();
  }
}
```

#### **3. Job Monitor Pattern**
```typescript
class JobMonitor {
  constructor(queue) {
    this.queue = queue;
    this.interval = setInterval(() => this.checkHealth(), 30000);
  }

  async checkHealth() {
    try {
      const counts = await this.queue.getJobCounts();
      const totalJobs = counts.active + counts.waiting + counts.delayed;

      console.log(`Queue Health: ${totalJobs} total jobs`);
      console.log(`Active: ${counts.active}, Waiting: ${counts.waiting}, Failed: ${counts.failed}`);

      // Alert thresholds
      if (counts.failed > 100) {
        console.error('ALERT: High failure rate detected!');
      }

      if (counts.waiting > 1000) {
        console.warn('WARNING: Queue backlog detected!');
      }

    } catch (error) {
      console.error('Health check failed:', error);
    }
  }

  stop() {
    if (this.interval) {
      clearInterval(this.interval);
    }
  }
}
```

#### **4. Retry Pattern**
```typescript
const retryProcessor = async (job) => {
  try {
    // Attempt to process the job
    const result = await processJob(job.data);

    // Log success
    await job.log(`Successfully processed on attempt ${job.attemptsMade}`);

    return result;

  } catch (error) {
    // Determine if error is retryable
    const isRetryable = !error.message.includes('permanent_failure');

    if (isRetryable && job.attemptsMade < job.opts.attempts) {
      // Log retry attempt
      await job.log(`Attempt ${job.attemptsMade} failed: ${error.message}. Retrying...`);

      // Throw to trigger retry
      throw error;
    } else {
      // Log final failure
      await job.log(`All retry attempts exhausted. Final error: ${error.message}`);

      // Move to failed state
      await job.moveToFailed(error, 'final-failure');

      throw error;
    }
  }
};
```

---

## 📦 **Installation and Setup**

### **Basic Installation:**
```bash
# Install BullMQ
npm install bullmq

# Install Redis client (required)
npm install redis

# TypeScript support
npm install --save-dev @types/bullmq typescript
```

### **Docker Setup:**
```yaml
version: '3.8'
services:
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes

  app:
    build: .
    environment:
      - REDIS_URL=redis://redis:6379
    depends_on:
      - redis

volumes:
  redis_data:
```

### **Environment Configuration:**
```typescript
// config/queue.ts
import { Queue, Worker } from 'bullmq';

export const redisConfig = {
    host: process.env.REDIS_HOST || 'localhost',
    port: parseInt(process.env.REDIS_PORT || '6379'),
    password: process.env.REDIS_PASSWORD,
    db: parseInt(process.env.REDIS_DB || '0'),
  retryDelayOnFailover: 100,
  maxRetriesPerRequest: null,
  lazyConnect: true,
};

export const queueConfig = {
  defaultJobOptions: {
    removeOnComplete: 100,
    removeOnFail: 50,
    attempts: 3,
  },
};
```

---

## 🏗️ **Core Concepts**

### **1. Queues**
Queues are the primary containers for jobs. They manage job storage, state transitions, and worker coordination.

```typescript
import { Queue } from 'bullmq';

// Create a queue
const emailQueue = new Queue('email', {
  connection: redisConfig,
  defaultJobOptions: {
    removeOnComplete: 100,
    attempts: 3,
  },
});

// Add jobs to queue
await emailQueue.add('send-welcome', {
  email: 'user@example.com',
  name: 'John'
});

// Get queue statistics
const waiting = await emailQueue.getWaiting();
const active = await emailQueue.getActive();
const completed = await emailQueue.getCompleted();
```

### **2. Jobs**
Jobs represent units of work to be processed. Each job has data, options, and state information.

```typescript
interface JobData {
  userId: string;
  email: string;
  template: string;
}

interface JobOptions {
  priority?: number;
  delay?: number;
  attempts?: number;
  backoff?: {
    type: 'fixed' | 'exponential';
    delay: number;
  };
}

// Create job with options
const job = await queue.add('email', jobData, {
  priority: 10,
  delay: 5000, // 5 seconds
  attempts: 3,
  backoff: {
    type: 'exponential',
    delay: 2000,
  },
});

// Job states: waiting -> active -> completed/failed
console.log('Job ID:', job.id);
console.log('Job state:', await job.getState());
```

### **3. Workers**
Workers consume jobs from queues and execute the processing logic.

```typescript
import { Worker } from 'bullmq';

// Create worker
const worker = new Worker('email', async (job) => {
  const { userId, email, template } = job.data;

  // Process the job
  await sendEmail(email, template);

  // Update progress (optional)
  job.updateProgress(50);

  // Return result
  return { sent: true, recipient: email };
}, {
  connection: redisConfig,
  concurrency: 5, // Process 5 jobs simultaneously
});

// Handle job completion
worker.on('completed', (job, result) => {
  console.log(`Email sent to ${result.recipient}`);
});

// Handle job failures
worker.on('failed', (job, err) => {
  console.log(`Job ${job.id} failed:`, err.message);
});
```

### **4. Job States**
Jobs transition through various states during their lifecycle:

```typescript
enum JobState {
  WAITING = 'waiting',     // Job is waiting in queue
  ACTIVE = 'active',       // Job is being processed
  COMPLETED = 'completed', // Job finished successfully
  FAILED = 'failed',       // Job failed (may retry)
  DELAYED = 'delayed',     // Job is scheduled for later
  PAUSED = 'paused',       // Queue is paused
}

// Check job state
const state = await job.getState();

// Get jobs by state
const waitingJobs = await queue.getJobs(['waiting']);
const activeJobs = await queue.getJobs(['active']);
const failedJobs = await queue.getJobs(['failed']);
```

---

## 🎯 **Basic Usage**

### **Simple Email Queue Example:**
```typescript
import { Queue, Worker } from 'bullmq';

// 1. Create queue
const emailQueue = new Queue('email', { connection: redisConfig });

// 2. Add job
await emailQueue.add('send-email', {
  to: 'user@example.com',
  subject: 'Welcome!',
  body: 'Welcome to our platform...'
});

// 3. Create worker
const emailWorker = new Worker('email', async (job) => {
  const { to, subject, body } = job.data;

  // Simulate email sending
  console.log(`Sending email to ${to}: ${subject}`);
  await new Promise(resolve => setTimeout(resolve, 1000));

  return { sent: true, recipient: to };
}, { connection: redisConfig });

// 4. Monitor results
emailWorker.on('completed', (job, result) => {
  console.log(`Email sent to ${result.recipient}`);
});
```

### **File Processing Example:**
```typescript
import { Queue, Worker } from 'bullmq';
import * as fs from 'fs';

const fileQueue = new Queue('file-processing', { connection: redisConfig });

const fileWorker = new Worker('file-processing', async (job) => {
  const { filePath, operation } = job.data;

  job.updateProgress(10, { status: 'Reading file' });

  // Read file
  const content = fs.readFileSync(filePath, 'utf8');

  job.updateProgress(50, { status: 'Processing file' });

  // Process file (example: convert to uppercase)
  const processed = content.toUpperCase();

  job.updateProgress(90, { status: 'Saving result' });

  // Save result
  fs.writeFileSync(`${filePath}.processed`, processed);

  return {
    originalSize: content.length,
    processedSize: processed.length,
    operation: 'uppercase'
  };
}, { connection: redisConfig });
```

### **API Rate Limiting Example:**
```typescript
import { Queue, Worker } from 'bullmq';

const apiQueue = new Queue('api-calls', { connection: redisConfig });

const apiWorker = new Worker('api-calls', async (job) => {
  const { endpoint, method, data, headers } = job.data;

  const response = await fetch(endpoint, {
    method,
    headers: {
      'Content-Type': 'application/json',
      ...headers
    },
    body: JSON.stringify(data)
  });

  if (!response.ok) {
    throw new Error(`API call failed: ${response.status}`);
  }

  return await response.json();
}, {
  connection: redisConfig,
  concurrency: 10, // Rate limit to 10 concurrent calls
  limiter: {
    max: 100,     // Max 100 calls
    duration: 1000 // Per second
  }
});

// Usage
await apiQueue.add('api-call', {
  endpoint: 'https://api.example.com/users',
  method: 'POST',
  data: { name: 'John', email: 'john@example.com' }
});
```

---

## ⚡ **Advanced Features**

### **Job Priorities:**
```typescript
// High priority jobs processed first
await queue.add('urgent-email', data, { priority: 10 });
await queue.add('regular-email', data, { priority: 5 });
await queue.add('bulk-email', data, { priority: 1 });

// Priority ranges from 1 (lowest) to higher numbers
```

### **Delayed Jobs:**
```typescript
// Job executes after 5 minutes
await queue.add('scheduled-task', data, {
  delay: 5 * 60 * 1000 // 5 minutes in milliseconds
});

// Job executes at specific time
const executeAt = new Date('2024-01-01T10:00:00');
await queue.add('scheduled-task', data, {
  delay: executeAt.getTime() - Date.now()
});
```

### **Recurring Jobs:**
```typescript
// Cron-based scheduling
await queue.add('daily-report', data, {
  repeat: {
    cron: '0 9 * * *', // Every day at 9 AM
  }
});

// Interval-based scheduling
await queue.add('health-check', data, {
  repeat: {
    every: 30000, // Every 30 seconds
  }
});

// Fixed rate scheduling
await queue.add('cleanup', data, {
  repeat: {
    every: 3600000, // Every hour
    limit: 24 // Limit to 24 executions
  }
});
```

### **Job Dependencies:**
```typescript
// Job B depends on Job A completion
const jobA = await queue.add('task-a', dataA);
const jobB = await queue.add('task-b', dataB, {
  parent: { id: jobA.id, queue: 'my-queue' }
});

// Job C depends on both A and B
const jobC = await queue.add('task-c', dataC, {
  parent: [
    { id: jobA.id, queue: 'my-queue' },
    { id: jobB.id, queue: 'my-queue' }
  ]
});
```

### **Batch Jobs:**
```typescript
// Process multiple jobs as a batch
const jobs = await queue.addBulk([
  { name: 'email', data: { to: 'user1@example.com' } },
  { name: 'email', data: { to: 'user2@example.com' } },
  { name: 'email', data: { to: 'user3@example.com' } },
]);

// Get results for all jobs
const results = await Promise.all(
  jobs.map(job => job.waitUntilFinished())
);
```

---

## 🎛️ **Job Management**

### **Job Operations:**
```typescript
// Get job by ID
const job = await queue.getJob('job-id-123');

// Get job state
const state = await job.getState();

// Update job data
await job.update({
  ...job.data,
  status: 'updated'
});

// Remove job
await job.remove();

// Retry failed job
await job.retry();

// Move job to different queue
await job.changePriority(5);
```

### **Bulk Operations:**
```typescript
// Get multiple jobs
const waitingJobs = await queue.getJobs(['waiting'], 0, 100);
const activeJobs = await queue.getJobs(['active'], 0, 50);

// Bulk job operations
await queue.clean(24 * 60 * 60 * 1000, 100); // Clean old jobs
await queue.obliterate({ force: false }); // Remove all jobs
```

### **Job Search and Filtering:**
```typescript
// Get jobs by pattern
const emailJobs = await queue.getJobs(['waiting', 'active'], 0, 10, true);

// Get completed jobs
const completedJobs = await queue.getCompleted(0, 50);

// Get failed jobs
const failedJobs = await queue.getFailed(0, 50);
```

---

## 👷 **Worker Management**

### **Worker Configuration:**
```typescript
const worker = new Worker('email', processor, {
  connection: redisConfig,
  concurrency: 5,          // Process 5 jobs simultaneously
  limiter: {
    max: 10,              // Max jobs per duration
    duration: 1000        // Per second
  },
  lockDuration: 30000,    // Job lock duration (30s)
  lockRenewTime: 15000,   // Renew lock every 15s
});
```

### **Worker Events:**
```typescript
worker.on('ready', () => {
  console.log('Worker is ready to process jobs');
});

worker.on('active', (job, prev) => {
  console.log(`Job ${job.id} is now active`);
});

worker.on('completed', (job, result) => {
  console.log(`Job ${job.id} completed with result:`, result);
});

worker.on('failed', (job, err) => {
  console.log(`Job ${job.id} failed with error:`, err.message);
});

worker.on('progress', (job, progress) => {
  console.log(`Job ${job.id} progress:`, progress);
});

worker.on('stalled', (jobId) => {
  console.log(`Job ${jobId} has stalled`);
});
```

### **Multiple Workers:**
```typescript
// Different workers for different job types
const emailWorker = new Worker('email-queue', emailProcessor, {
  connection: redisConfig,
  concurrency: 3
});

const fileWorker = new Worker('file-queue', fileProcessor, {
  connection: redisConfig,
  concurrency: 2
});

// Shared worker for multiple queues
const generalWorker = new Worker(['queue1', 'queue2'], generalProcessor, {
  connection: redisConfig,
  concurrency: 5
});
```

### **Worker Lifecycle:**
```typescript
// Graceful shutdown
process.on('SIGTERM', async () => {
  await worker.close();
  console.log('Worker closed gracefully');
});

// Pause worker
await worker.pause();

// Resume worker
await worker.resume();

// Close worker
await worker.close();
```

---

## 🚨 **Error Handling and Retries**

### **Basic Retry Configuration:**
```typescript
await queue.add('api-call', data, {
  attempts: 3,
  backoff: {
    type: 'exponential',
    delay: 2000
  }
});
```

### **Advanced Retry Strategies:**
```typescript
// Custom backoff strategy
const customBackoff = (attemptsMade: number, err: Error) => {
  // Different delays based on error type
  if (err.message.includes('rate limit')) {
    return 60000; // 1 minute for rate limits
  }
  if (err.message.includes('network')) {
    return Math.min(1000 * Math.pow(2, attemptsMade), 30000);
  }
  return 5000; // Default 5 seconds
};

await queue.add('api-call', data, {
  attempts: 5,
  backoff: customBackoff
});
```

### **Error Handling in Workers:**
```typescript
const worker = new Worker('email', async (job) => {
  try {
    const result = await sendEmail(job.data);

    if (!result.success) {
      // Custom error for retry
      throw new Error('Email delivery failed');
    }

    return result;
  } catch (error) {
    // Log error for monitoring
    console.error(`Job ${job.id} failed:`, error);

    // Re-throw for retry or dead letter
    throw error;
  }
}, { connection: redisConfig });
```

### **Dead Letter Queues:**
```typescript
// Create dead letter queue
const dlq = new Queue('dead-letter-queue', { connection: redisConfig });

// Worker with dead letter handling
const worker = new Worker('main-queue', async (job) => {
  try {
    return await processJob(job);
  } catch (error) {
    if (job.attemptsMade >= job.opts.attempts) {
      // Move to dead letter queue
      await dlq.add('failed-job', {
        originalJobId: job.id,
        originalQueue: job.queueName,
        error: error.message,
        data: job.data,
        failedAt: new Date()
      });
    }
    throw error;
  }
}, { connection: redisConfig });
```

---

## 📊 **Real-time Monitoring**

### **Queue Events:**
```typescript
// Monitor queue events
queue.on('waiting', (jobId) => {
  console.log(`Job ${jobId} is waiting`);
});

queue.on('active', (jobId, prev) => {
  console.log(`Job ${jobId} is now active`);
});

queue.on('completed', (jobId, result) => {
  console.log(`Job ${jobId} completed:`, result);
});

queue.on('failed', (jobId, err) => {
  console.log(`Job ${jobId} failed:`, err.message);
});

queue.on('stalled', (jobId) => {
  console.log(`Job ${jobId} has stalled`);
});
```

### **Queue Metrics:**
```typescript
// Get queue statistics
const waitingCount = await queue.getWaiting(0, -1).then(jobs => jobs.length);
const activeCount = await queue.getActive(0, -1).then(jobs => jobs.length);
const completedCount = await queue.getCompleted(0, -1).then(jobs => jobs.length);
const failedCount = await queue.getFailed(0, -1).then(jobs => jobs.length);

// Calculate rates
const stats = {
  waiting: waitingCount,
  active: activeCount,
  completed: completedCount,
  failed: failedCount,
  total: waitingCount + activeCount + completedCount + failedCount
};

console.log('Queue Stats:', stats);
```

### **Performance Monitoring:**
```typescript
// Monitor job processing time
const startTime = Date.now();

worker.on('active', (job) => {
  job.startTime = Date.now();
});

worker.on('completed', (job) => {
  const processingTime = Date.now() - job.startTime;
  console.log(`Job ${job.id} took ${processingTime}ms`);
});

// Monitor queue throughput
setInterval(async () => {
  const completed = await queue.getCompleted(0, -1);
  const throughput = completed.length / ((Date.now() - startTime) / 1000);
  console.log(`Throughput: ${throughput.toFixed(2)} jobs/second`);
}, 60000); // Every minute
```

---

## ⚡ **Performance Optimization**

### **Redis Configuration:**
```redis.conf
# Memory optimization
maxmemory 512mb
maxmemory-policy allkeys-lru

# Connection settings
tcp-keepalive 300
tcp-backlog 511
timeout 300

# Performance tuning
databases 16
save 900 1
save 300 10
save 60 10000
```

### **BullMQ Optimization:**
```typescript
// Optimized queue configuration
const optimizedQueue = new Queue('high-throughput', {
  connection: {
    ...redisConfig,
    maxRetriesPerRequest: null,
    retryDelayOnFailover: 100,
    lazyConnect: true,
  },
  defaultJobOptions: {
    removeOnComplete: 1000,  // Keep more completed jobs
    removeOnFail: 500,       // Keep more failed jobs
    attempts: 3,
    backoff: {
      type: 'exponential',
      delay: 1000,
    },
  },
  settings: {
    lockDuration: 30000,     // 30 second lock
    stalledInterval: 30000,  // 30 second stall check
    maxStalledCount: 1,      // Max stalled attempts
  }
});
```

### **Worker Optimization:**
```typescript
// High-performance worker
const optimizedWorker = new Worker('processing', processor, {
  connection: redisConfig,
  concurrency: Math.max(1, os.cpus().length - 1), // CPU cores - 1
  limiter: {
    max: 1000,      // Max jobs per second
    duration: 1000
  },
  settings: {
    lockDuration: 30000,
    lockRenewTime: 15000,
  }
});
```

### **Connection Pooling:**
```typescript
// Redis connection pooling for high throughput
import { Cluster } from 'ioredis';

const cluster = new Cluster([
  { host: 'redis-1', port: 6379 },
  { host: 'redis-2', port: 6379 },
  { host: 'redis-3', port: 6379 }
], {
  redisOptions: {
    password: process.env.REDIS_PASSWORD
  },
  clusterRetryDelay: 100,
  maxRedirections: 16
});

const queue = new Queue('cluster-queue', {
  connection: cluster,
  defaultJobOptions: {
    removeOnComplete: 100,
    attempts: 3
  }
});
```

---

## 🔐 **Security Best Practices**

### **Redis Security:**
```bash
# Redis configuration
bind 127.0.0.1          # Only localhost
port 6379               # Default port
requirepass strongpassword
timeout 300             # Connection timeout
maxclients 1000         # Max connections
```

### **BullMQ Security:**
```typescript
// Secure job data handling
const secureQueue = new Queue('secure-jobs', {
  connection: {
    ...redisConfig,
    password: process.env.REDIS_PASSWORD,
    tls: {
      ca: fs.readFileSync('/path/to/ca.crt'),
      cert: fs.readFileSync('/path/to/client.crt'),
      key: fs.readFileSync('/path/to/client.key')
    }
  }
});

// Sanitize job data
const sanitizeJobData = (data: any) => {
  // Remove sensitive information
  const sanitized = { ...data };
  delete sanitized.password;
  delete sanitized.apiKey;
  return sanitized;
};
```

### **Access Control:**
```typescript
// Role-based job access
const adminQueue = new Queue('admin-jobs', { connection: redisConfig });
const userQueue = new Queue('user-jobs', { connection: redisConfig });

// Check permissions before processing
const secureWorker = new Worker('admin-jobs', async (job) => {
  // Verify admin permissions
  if (!await checkAdminPermissions(job.data.userId)) {
    throw new Error('Insufficient permissions');
  }

  return await processAdminJob(job.data);
}, { connection: redisConfig });
```

---

## 🚀 **Production Deployment**

### **Docker Compose Setup:**
```yaml
version: '3.8'
services:
  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "--raw", "incr", "ping"]
      interval: 30s
      timeout: 10s
      retries: 3

  app:
    build: .
    environment:
      - REDIS_URL=redis://redis:6379
      - REDIS_PASSWORD=${REDIS_PASSWORD}
    depends_on:
      redis:
        condition: service_healthy
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '1.0'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 512M

  worker:
    build: .
    command: npm run worker
    environment:
      - REDIS_URL=redis://redis:6379
      - REDIS_PASSWORD=${REDIS_PASSWORD}
    depends_on:
      - redis
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: '0.5'
          memory: 512M

volumes:
  redis_data:
```

### **Kubernetes Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bullmq-worker
spec:
  replicas: 3
  selector:
    matchLabels:
      app: bullmq-worker
  template:
    metadata:
      labels:
        app: bullmq-worker
    spec:
      containers:
      - name: worker
        image: myapp:latest
        command: ["npm", "run", "worker"]
        env:
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: redis-secret
              key: url
        resources:
          limits:
            cpu: "500m"
            memory: "512Mi"
          requests:
            cpu: "250m"
            memory: "256Mi"
        livenessProbe:
          exec:
            command: ["node", "healthcheck.js"]
          initialDelaySeconds: 30
          periodSeconds: 30
```

### **Health Checks:**
```typescript
// Application health check
app.get('/health', async (req, res) => {
  try {
    // Check Redis connectivity
    await redis.ping();

    // Check queue health
    const waiting = await queue.getWaiting(0, 1);

    res.json({
      status: 'healthy',
      redis: 'connected',
      queue: 'operational',
      timestamp: new Date().toISOString()
    });
  } catch (error) {
    res.status(503).json({
      status: 'unhealthy',
      error: error.message,
      timestamp: new Date().toISOString()
    });
  }
});
```

---

## 🔧 **Troubleshooting**

### **Common Issues:**

#### **Connection Problems:**
```bash
# Test Redis connectivity
redis-cli -h localhost -p 6379 ping

# Check Redis logs
docker logs redis-container

# Verify BullMQ connection
node -e "
const { Queue } = require('bullmq');
const queue = new Queue('test', { connection: { host: 'localhost', port: 6379 } });
queue.add('test-job', { data: 'test' }).then(() => console.log('Connected'));
"
```

#### **Job Stalling:**
```typescript
// Find stalled jobs
const stalledJobs = await queue.getJobs(['active'], 0, 100, true);

for (const job of stalledJobs) {
  const timeSinceActive = Date.now() - job.processedOn;
  if (timeSinceActive > 5 * 60 * 1000) { // 5 minutes
    console.log(`Stalled job: ${job.id}`);
    await job.moveToFailed(new Error('Job stalled'), true);
  }
}
```

#### **Memory Issues:**
```typescript
// Monitor Redis memory
const redis = new Redis(redisConfig);
const memory = await redis.info('memory');
console.log('Redis memory:', memory);

// Clean up old jobs
await queue.clean(24 * 60 * 60 * 1000, 1000); // 24 hours, 1000 jobs

// Adjust memory settings
await queue.updateSettings({
  removeOnComplete: 100,
  removeOnFail: 50
});
```

#### **Performance Issues:**
```typescript
// Monitor queue performance
const monitorPerformance = async () => {
  const startTime = Date.now();
  const startCompleted = await queue.getCompleted(0, -1).then(jobs => jobs.length);

  setTimeout(async () => {
    const endCompleted = await queue.getCompleted(0, -1).then(jobs => jobs.length);
    const throughput = (endCompleted - startCompleted) / ((Date.now() - startTime) / 1000);
    console.log(`Throughput: ${throughput.toFixed(2)} jobs/second`);
  }, 60000);
};
```

---

## 💡 **Use Cases**

### **1. Email Processing:**
```typescript
const emailQueue = new Queue('email', { connection: redisConfig });

const emailWorker = new Worker('email', async (job) => {
  const { to, subject, template, data } = job.data;

  // Render email template
  const html = await renderTemplate(template, data);

  // Send email
  await sendEmail({
    to,
    subject,
    html
  });

  return { sent: true, recipient: to };
}, { connection: redisConfig });

// Usage
await emailQueue.add('send-welcome', {
  to: 'user@example.com',
  subject: 'Welcome!',
  template: 'welcome-email',
  data: { name: 'John' }
});
```

### **2. File Processing:**
```typescript
const fileQueue = new Queue('file-processing', { connection: redisConfig });

const fileWorker = new Worker('file-processing', async (job) => {
  const { filePath, operations } = job.data;

  let result = fs.readFileSync(filePath);

  for (const op of operations) {
    job.updateProgress(25, { status: `Applying ${op.type}` });

    switch (op.type) {
      case 'resize':
        result = await resizeImage(result, op.width, op.height);
        break;
      case 'compress':
        result = await compressImage(result, op.quality);
        break;
      case 'watermark':
        result = await addWatermark(result, op.text);
        break;
    }
  }

  const outputPath = `${filePath}.processed`;
  fs.writeFileSync(outputPath, result);

  return {
    originalSize: result.length,
    outputPath,
    operations: operations.length
  };
}, { connection: redisConfig });
```

### **3. API Rate Limiting:**
```typescript
const apiQueue = new Queue('api-calls', { connection: redisConfig });

const apiWorker = new Worker('api-calls', async (job) => {
  const { endpoint, method, data, headers } = job.data;

  const response = await fetch(endpoint, {
    method,
    headers: {
      'Content-Type': 'application/json',
      ...headers
    },
    body: JSON.stringify(data)
  });

  if (!response.ok) {
    throw new Error(`API call failed: ${response.status}`);
  }

  return await response.json();
}, {
  connection: redisConfig,
  concurrency: 10, // Rate limit to 10 concurrent calls
  limiter: {
    max: 100,     // Max 100 calls
    duration: 1000 // Per second
  }
});
```

### **4. Report Generation:**
```typescript
const reportQueue = new Queue('reports', { connection: redisConfig });

const reportWorker = new Worker('reports', async (job) => {
  const { type, parameters, userId } = job.data;

  job.updateProgress(10, { status: 'Fetching data' });

  // Fetch data from database
  const data = await fetchReportData(type, parameters);

  job.updateProgress(50, { status: 'Processing data' });

  // Process data
  const processedData = await processReportData(data);

  job.updateProgress(80, { status: 'Generating report' });

  // Generate report (PDF, Excel, etc.)
  const reportBuffer = await generateReport(processedData, type);

  job.updateProgress(95, { status: 'Saving report' });

  // Save to storage
  const reportUrl = await saveReport(reportBuffer, userId, type);

  return {
    reportUrl,
    size: reportBuffer.length,
    generatedAt: new Date().toISOString()
  };
}, { connection: redisConfig });
```

---

## ⚖️ **Comparison with Alternatives**

### **BullMQ vs Bull:**
```typescript
// BullMQ (Modern, TypeScript-first)
import { Queue, Worker, Job } from 'bullmq';

const queue = new Queue<EmailData>('email', {
  connection: redisConfig
});

// Full TypeScript support
interface EmailData {
  to: string;
  subject: string;
  body: string;
}

// Bull (Legacy, JavaScript-focused)
const Queue = require('bull');

const queue = new Queue('email', {
  redis: redisConfig
});

// No built-in TypeScript support
```

### **BullMQ vs Agenda:**
```typescript
// BullMQ (Redis-based, high performance)
const queue = new Queue('email', { connection: redisConfig });
await queue.add('send-email', data, { delay: 60000 });

// Agenda (MongoDB-based, scheduling focused)
const agenda = new Agenda({ db: { address: mongoUrl } });
agenda.define('send email', async (job) => {
  await sendEmail(job.attrs.data);
});
await agenda.schedule('in 1 minute', 'send email', data);
```

### **BullMQ vs Bee-Queue:**
```typescript
// BullMQ (Advanced features, production ready)
const queue = new Queue('email', {
  connection: redisConfig,
  defaultJobOptions: {
    attempts: 3,
    backoff: { type: 'exponential', delay: 2000 }
  }
});

// Bee-Queue (Simple, lightweight)
const queue = new Bee('email', { redis: redisConfig });
queue.process(async (job) => {
  // Simple processing
  return await sendEmail(job.data);
});
```

### **When to Choose BullMQ:**

**✅ Best For:**
- High-throughput applications
- Complex job workflows
- Real-time monitoring requirements
- Priority-based job processing
- Advanced retry and error handling
- TypeScript applications
- Production-scale applications

**❌ Consider Alternatives For:**
- Simple, low-volume job processing
- MongoDB-based applications (use Agenda)
- Lightweight, minimal dependencies (use Bee-Queue)
- Legacy JavaScript applications (Bull might suffice)

---

## 🎯 **Best Practices**

### **1. Job Design:**
```typescript
// Good: Focused, single-responsibility jobs
interface EmailJobData {
  to: string;
  subject: string;
  templateId: string;
  variables: Record<string, any>;
}

await queue.add('send-template-email', {
  to: 'user@example.com',
  subject: 'Welcome!',
  templateId: 'welcome-email',
  variables: { name: 'John', company: 'Acme' }
});

// Bad: Overloaded jobs
await queue.add('process-user', {
  userId: 123,
  sendEmail: true,
  updateProfile: true,
  createReport: true,
  notifyAdmins: true
  // Too many responsibilities
});
```

### **2. Error Handling:**
```typescript
const worker = new Worker('email', async (job) => {
  try {
    // Validate input
    if (!job.data.to || !job.data.subject) {
      throw new Error('Missing required fields: to, subject');
    }

    // Process job
    const result = await sendEmail(job.data);

    if (!result.success) {
      throw new Error(`Email delivery failed: ${result.error}`);
    }

    return result;
  } catch (error) {
    // Log error with context
    console.error(`Job ${job.id} failed:`, {
      error: error.message,
      jobData: job.data,
      attemptsMade: job.attemptsMade
    });

    throw error; // Re-throw for retry
  }
}, {
  connection: redisConfig,
  concurrency: 5
});
```

### **3. Monitoring & Alerting:**
```typescript
// Monitor queue health
const monitorQueue = async () => {
  const waiting = await queue.getWaiting(0, -1);
  const active = await queue.getActive(0, -1);
  const failed = await queue.getFailed(0, -1);

  const metrics = {
    waitingCount: waiting.length,
    activeCount: active.length,
    failedCount: failed.length,
    timestamp: new Date().toISOString()
  };

  // Alert if too many failed jobs
  if (failed.length > 100) {
    await sendAlert('High failure rate detected', metrics);
  }

  // Log metrics
  console.log('Queue metrics:', metrics);
};

// Monitor every 5 minutes
setInterval(monitorQueue, 5 * 60 * 1000);
```

### **4. Resource Management:**
```typescript
// Configure appropriate resource limits
const worker = new Worker('processing', processor, {
  connection: redisConfig,
  concurrency: Math.max(1, os.cpus().length - 1),
  limiter: {
    max: 100,     // Max jobs per second
    duration: 1000
  },
  settings: {
    lockDuration: 30000,    // 30 second job lock
    lockRenewTime: 15000,   // Renew every 15 seconds
    stalledInterval: 30000, // Check for stalled jobs
  }
});

// Graceful shutdown
process.on('SIGTERM', async () => {
  console.log('Shutting down worker...');
  await worker.close();
  process.exit(0);
});
```

### **5. Security:**
```typescript
// Sanitize job data
const sanitizeJobData = (data: any) => {
  const sanitized = { ...data };

  // Remove sensitive fields
  delete sanitized.password;
  delete sanitized.apiKey;
  delete sanitized.ssn;

  // Validate required fields
  if (!sanitized.userId) {
    throw new Error('userId is required');
  }

  return sanitized;
};

// Use in worker
const worker = new Worker('secure-job', async (job) => {
  const cleanData = sanitizeJobData(job.data);
  return await processSecureJob(cleanData);
}, { connection: redisConfig });
```

### **6. Performance:**
```typescript
// Optimize for high throughput
const highThroughputQueue = new Queue('high-volume', {
  connection: {
    ...redisConfig,
    maxRetriesPerRequest: null,
    retryDelayOnFailover: 100
  },
  defaultJobOptions: {
    removeOnComplete: 1000,  // Keep more completed jobs
    removeOnFail: 500,       // Keep more failed jobs
    attempts: 3,
    backoff: {
      type: 'exponential',
      delay: 1000
    }
  }
});

// Batch job processing
const batchJobs = [];
for (let i = 0; i < 1000; i++) {
  batchJobs.push({
    name: 'process-item',
    data: { itemId: i }
  });
}

await queue.addBulk(batchJobs);
```

---

## 📚 **API Reference**

### **Queue Methods:**
```typescript
// Job Management
await queue.add(name, data, options?)           // Add job
await queue.addBulk(jobs)                       // Add multiple jobs
await queue.getJob(jobId)                       // Get job by ID
await queue.getJobs(states, start?, end?)       // Get jobs by state

// Queue Control
await queue.pause()                             // Pause queue
await queue.resume()                            // Resume queue
await queue.clean(grace, limit?, states?)       // Clean old jobs
await queue.obliterate(options?)                // Remove all jobs

// Queue Info
await queue.getWaiting(start?, end?)            // Get waiting jobs
await queue.getActive(start?, end?)             // Get active jobs
await queue.getCompleted(start?, end?)          // Get completed jobs
await queue.getFailed(start?, end?)             // Get failed jobs
```

### **Worker Methods:**
```typescript
// Worker Control
await worker.pause()                            // Pause worker
await worker.resume()                           // Resume worker
await worker.close()                            // Close worker

// Worker Info
worker.concurrency                             // Current concurrency
worker.opts                                     // Worker options
```

### **Job Methods:**
```typescript
// Job Control
await job.retry()                               // Retry job
await job.moveToFailed(error, token?)           // Mark as failed
await job.moveToCompleted(result, token?)       // Mark as completed

// Job Info
await job.getState()                            // Get job state
await job.update(data)                          // Update job data
await job.updateProgress(progress, data?)       // Update progress
await job.remove()                              // Remove job
```

### **Common Options:**
```typescript
interface JobOptions {
  priority?: number;                            // Job priority (1-∞)
  delay?: number;                               // Delay in milliseconds
  attempts?: number;                            // Max retry attempts
  backoff?: {
    type: 'fixed' | 'exponential';              // Backoff type
    delay: number;                              // Base delay
  };
  removeOnComplete?: number | boolean;          // Keep completed jobs
  removeOnFail?: number | boolean;              // Keep failed jobs
  jobId?: string;                               // Custom job ID
  repeat?: {
    cron?: string;                              // Cron expression
    every?: number;                             // Interval in ms
    limit?: number;                             // Max repetitions
  };
}

interface WorkerOptions {
  connection?: RedisConfig;                     // Redis connection
  concurrency?: number;                         // Max concurrent jobs
  limiter?: {
    max: number;                                // Max jobs
    duration: number;                           // Time window
  };
  lockDuration?: number;                        // Job lock duration
  lockRenewTime?: number;                       // Lock renewal interval
}
```

---

## 🎉 **Conclusion**

BullMQ is a powerful, modern job queue library for Node.js that provides:

- **🚀 High Performance**: Redis-backed with optimized throughput
- **🔧 Advanced Features**: Priorities, retries, scheduling, dead letter queues
- **📊 Real-time Monitoring**: Live job status updates and progress tracking
- **🛡️ Fault Tolerance**: Automatic recovery and distributed processing
- **📝 TypeScript Support**: Full type safety and IntelliSense
- **🏗️ Production Ready**: Battle-tested in large-scale applications

### **Key Benefits:**
- **Scalability**: Handle thousands of jobs per second
- **Reliability**: Jobs survive server restarts and crashes
- **Flexibility**: Support for complex job workflows and dependencies
- **Observability**: Comprehensive monitoring and alerting
- **Developer Experience**: Excellent TypeScript support and documentation

### **Use Cases:**
- **Email Processing**: Bulk email sending with retry logic
- **File Processing**: Image resizing, video encoding, document processing
- **API Integration**: Rate-limited API calls with error handling
- **Report Generation**: Scheduled report creation and delivery
- **Background Tasks**: Database cleanup, cache warming, notifications

BullMQ is the modern choice for job queue management in Node.js applications, offering enterprise-grade features with excellent developer experience. Whether you're building a small API service or a large-scale distributed system, BullMQ provides the tools you need for reliable, high-performance job processing.
