# 🚀 **Redis Deep Dive - Complete Technical Guide**

## 📋 **Table of Contents**
- [What is Redis?](#what-is-redis)
- [Redis in Sentinel](#redis-in-sentinel)
- [Data Types and Commands](#data-types-and-commands)
- [Persistence and Durability](#persistence-and-durability)
- [Memory Management](#memory-management)
- [Replication and Clustering](#replication-and-clustering)
- [Performance Optimization](#performance-optimization)
- [Security Best Practices](#security-best-practices)
- [Monitoring and Troubleshooting](#monitoring-and-troubleshooting)
- [Production Deployment](#production-deployment)
- [Redis vs Other Databases](#redis-vs-other-databases)
- [Advanced Features](#advanced-features)
- [Real-World Use Cases](#real-world-use-cases)

---

## 🤔 **What is Redis?**

**Redis** (Remote Dictionary Server) is an **open-source, in-memory data structure store** that can be used as a database, cache, and message broker. It supports various data structures and provides high-performance operations.

### **Core Characteristics:**
- ✅ **In-Memory**: Lightning-fast data access
- ✅ **Persistent**: Optional disk persistence
- ✅ **Distributed**: Replication and clustering
- ✅ **Versatile**: Multiple data types and use cases
- ✅ **Atomic**: Single-threaded operations guarantee

### **Why Redis?**
```typescript
// Traditional Database (Slow)
await sql`SELECT * FROM users WHERE id = ?` // ~10-100ms

// Redis (Fast)
await redis.get('user:123') // ~1ms (100x faster)
```

---

## 🎯 **Redis in Sentinel**

### **Sentinel's Redis Usage:**
```typescript
// 1. BullMQ Job Storage
redis.set('bull:test-execution:job-123', JSON.stringify(jobData))

// 2. Session Management
redis.setex('session:user-456', 3600, sessionData)

// 3. Caching Layer
redis.set('cache:test-results:789', JSON.stringify(results), 'EX', 300)

// 4. Rate Limiting
redis.incr('rate-limit:api:ip-192.168.1.1')
redis.expire('rate-limit:api:ip-192.168.1.1', 60)

// 5. Distributed Locks
redis.set('lock:resource-abc', 'worker-1', 'NX', 'EX', 30)
```

### **Architecture Integration:**
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   API Service   │────│     Redis       │────│ Integrations    │
│   (Port 3000)   │    │  (Cache & Queue)│    │   Service       │
│                 │    │                 │    │  (Port 3001)    │
│ ┌─────────────┐ │    │ ┌─────────────┐ │    │                 │
│ │ BullMQ Jobs │◄├───►│ │  Job Queue   │◄├───►│ ┌─────────────┐ │
│ │   Storage    │ │    │ │   & State   │ │    │ │  Workers     │ │
│ └─────────────┘ │    │ └─────────────┘ │    │ │  Processing  │ │
└─────────────────┘    └─────────────────┘    │ └─────────────┘ │
                                              └─────────────────┘
```

---

## 📊 **Data Types and Commands**

### **1. Strings (Most Common)**
```bash
# Basic operations
SET user:123:name "John Doe"      # Set string value
GET user:123:name                 # Get string value: "John Doe"
EXISTS user:123:name              # Check if key exists: 1
DEL user:123:name                 # Delete key

# Advanced operations
SETEX cache:weather 300 "sunny"   # Set with expiration (5 min)
INCR counter:visits                # Increment counter: 1, 2, 3...
APPEND log:events "New event"     # Append to string
STRLEN user:123:name              # Get string length: 8

# Bit operations
SETBIT bitmap:active:2024 1 1    # Set bit at position 1
GETBIT bitmap:active:2024 1      # Get bit at position 1: 1
BITCOUNT bitmap:active:2024       # Count set bits
```

### **2. Hashes (Objects)**
```bash
# User profile as hash
HSET user:123 name "John" email "john@example.com" age "30"
HGET user:123 name                    # "John"
HGETALL user:123                     # All fields
HMSET user:123 city "NYC" country "USA"  # Multiple fields
HINCRBY user:123 login_count 1       # Increment field
HEXISTS user:123 email               # Check field exists
HDEL user:123 age                    # Delete field
HLEN user:123                        # Number of fields
```

### **3. Lists (Queues/Stacks)**
```bash
# Queue operations (FIFO)
LPUSH queue:emails "email1"         # Add to left: [email1]
LPUSH queue:emails "email2"         # Add to left: [email2, email1]
RPOP queue:emails                   # Remove from right: email1
LLEN queue:emails                   # Length: 1

# Stack operations (LIFO)
LPUSH stack:tasks "task1"           # Add to left
LPOP stack:tasks                    # Remove from left

# Advanced operations
LRANGE queue:emails 0 -1            # Get all elements
LTRIM queue:emails 0 99             # Keep first 100 elements
BRPOP queue:emails 0                # Blocking pop (wait if empty)
```

### **4. Sets (Unique Collections)**
```bash
# User interests
SADD user:123:interests "redis" "javascript" "nodejs"
SADD user:123:interests "python"           # Add another
SMEMBERS user:123:interests               # Get all: [redis, javascript, nodejs, python]
SISMEMBER user:123:interests "redis"     # Check membership: 1
SCARD user:123:interests                 # Count elements: 4
SREM user:123:interests "python"         # Remove element

# Set operations
SADD user:456:interests "redis" "golang" "kubernetes"
SINTER user:123:interests user:456:interests  # Intersection: [redis]
SUNION user:123:interests user:456:interests  # Union: [redis, javascript, nodejs, golang, kubernetes]
SDIFF user:123:interests user:456:interests   # Difference: [javascript, nodejs]
```

### **5. Sorted Sets (Ordered Sets)**
```bash
# Leaderboard with scores
ZADD leaderboard:game1 100 "user123"     # Score: 100
ZADD leaderboard:game1 150 "user456"     # Score: 150
ZADD leaderboard:game1 75 "user789"      # Score: 75

ZSCORE leaderboard:game1 "user456"       # Get score: 150
ZRANK leaderboard:game1 "user456"         # Get rank: 0 (highest)
ZREVRANK leaderboard:game1 "user456"      # Reverse rank: 2
ZRANGE leaderboard:game1 0 -1 WITHSCORES # Get all with scores
ZREVRANGE leaderboard:game1 0 2          # Top 3 players
ZINCRBY leaderboard:game1 25 "user123"   # Increase score

# Time-based operations
ZADD timeline:posts 1640995200 "post123"  # Timestamp score
ZRANGEBYSCORE timeline:posts 1640995200 1641081600  # Posts in time range
```

### **6. Bitmaps (Compact Boolean Arrays)**
```bash
# User activity tracking (365 days)
SETBIT user:123:active:2024 1 1    # Day 1: active
SETBIT user:123:active:2024 30 1   # Day 30: active
SETBIT user:123:active:2024 365 1  # Day 365: active

GETBIT user:123:active:2024 30     # Was user active on day 30?: 1
BITCOUNT user:123:active:2024       # Total active days: 3
BITPOS user:123:active:2024 0       # First inactive day: 0

# Analytics
BITOP AND result user:123:active:2024 user:456:active:2024  # Days both active
BITOP OR combined user:123:active:2024 user:456:active:2024  # Days either active
```

### **7. HyperLogLog (Unique Counting)**
```bash
# Unique visitor counting (memory efficient)
PFADD page:home:unique_visitors "user123" "user456" "user123"  # Only 2 unique
PFCOUNT page:home:unique_visitors                            # Count: 2

# Merge multiple counters
PFMERGE page:all:unique_visitors page:home:unique_visitors page:about:unique_visitors
```

### **8. Geospatial (Location Data)**
```bash
# Store locations
GEOADD locations:restaurants -122.4194 37.7749 "restaurant1"
GEOADD locations:restaurants -122.4089 37.7833 "restaurant2"

# Find nearby
GEORADIUS locations:restaurants -122.4194 37.7749 5 km WITHCOORD WITHDIST
# Returns: [restaurant1, [-122.4194, 37.7749], 0.0000], [restaurant2, [-122.4089, 37.7833], 1.2567]

GEODIST locations:restaurants "restaurant1" "restaurant2" km  # Distance: 1.2567 km
```

---

## 💾 **Persistence and Durability**

### **Persistence Strategies:**

#### **1. RDB (Redis Database File)**
```redis.conf
# RDB Configuration
save 900 1      # Save if 1 change in 900 seconds
save 300 10     # Save if 10 changes in 300 seconds
save 60 10000   # Save if 10000 changes in 60 seconds

dbfilename dump.rdb
dir /var/lib/redis

# Compression
rdbcompression yes
rdbchecksum yes
```

#### **2. AOF (Append Only File)**
```redis.conf
# AOF Configuration
appendonly yes
appendfilename "appendonly.aof"

# Sync strategies
appendfsync everysec    # Sync every second (balanced)
# appendfsync always    # Sync every write (slow, durable)
# appendfsync no        # OS decides (fast, risky)

# Rewrite settings
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

#### **3. Hybrid Persistence (Redis 5+)**
```redis.conf
# Hybrid persistence combines RDB + AOF
aof-use-rdb-preamble yes
```

### **Durability Trade-offs:**

| Strategy | Performance | Durability | Recovery Time | Use Case |
|----------|-------------|------------|---------------|----------|
| **RDB Only** | Fastest | Medium | Fast | Development |
| **AOF Only** | Medium | High | Slow | Production |
| **Hybrid** | Balanced | High | Medium | Enterprise |

### **Backup & Recovery:**
```bash
# Manual backup
redis-cli BGSAVE                    # Async RDB snapshot
redis-cli LASTSAVE                  # Check last save time

# Copy backup files
cp /var/lib/redis/dump.rdb /backup/
cp /var/lib/redis/appendonly.aof /backup/

# Restore from backup
redis-cli SHUTDOWN NOSAVE          # Graceful shutdown
# Replace files and restart Redis
```

---

## 🧠 **Memory Management**

### **Memory Policies:**

#### **1. Eviction Policies**
```redis.conf
# Memory limit
maxmemory 256mb

# Eviction policies
maxmemory-policy noeviction        # Don't evict (default)
maxmemory-policy allkeys-lru       # LRU on all keys
maxmemory-policy volatile-lru      # LRU on keys with TTL
maxmemory-policy allkeys-random    # Random on all keys
maxmemory-policy volatile-random   # Random on keys with TTL
maxmemory-policy volatile-ttl      # Evict soonest expiring
```

#### **2. Key Expiration**
```bash
# Set expiration
SET session:123 "data" EX 3600      # Expire in 1 hour
EXPIRE user:456 86400               # Expire in 24 hours
PEXPIRE cache:789 300000            # Expire in 5 minutes (milliseconds)

# Check TTL
TTL session:123                     # Seconds remaining: 3545
PTTL session:123                    # Milliseconds remaining: 3545123

# Remove expiration
PERSIST session:123                 # Remove TTL

# Conditional operations
SET key value NX EX 60              # Set only if not exists, expire in 60s
```

### **Memory Optimization Techniques:**

#### **1. Data Structure Optimization**
```bash
# Use hashes for objects (saves memory)
# Instead of: user:123:name "John", user:123:email "john@example.com"
HSET user:123 name "John" email "john@example.com"  # Single hash key

# Use integers for counters
INCR counter                        # Efficient integer storage
```

#### **2. Key Naming Best Practices**
```bash
# Good: Structured, hierarchical
user:123:profile:name
product:456:reviews:789:rating

# Bad: Unstructured
user_profile_name_123
product456reviews789rating
```

#### **3. Memory Monitoring**
```bash
# Memory info
INFO memory

# Key space analysis
MEMORY USAGE user:123
MEMORY STATS

# Find large keys
redis-cli --bigkeys

# Memory fragmentation
MEMORY PURGE                      # Defragment memory
```

---

## 🔄 **Replication and Clustering**

### **Master-Slave Replication:**

#### **Configuration:**
```redis.conf
# Master configuration (no special config needed)

# Slave configuration
replicaof 192.168.1.100 6379      # Point to master
# replicaof <master-ip> <master-port>

# Authentication
masterauth your-password          # If master requires auth

# Replication settings
repl-timeout 60                   # Replication timeout
repl-backlog-size 1mb             # Backlog size
```

#### **Replication Commands:**
```bash
# Check replication status
INFO replication

# Manual failover (Redis 6+)
REPLICAOF NO ONE                  # Promote slave to master

# Replication info
ROLE                              # Check if master or slave
```

### **Redis Cluster:**

#### **Cluster Configuration:**
```redis.conf
# Enable clustering
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000

# Cluster settings
cluster-require-full-coverage no
cluster-migration-barrier 1
```

#### **Cluster Management:**
```bash
# Create cluster
redis-cli --cluster create 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003

# Check cluster status
redis-cli --cluster check 127.0.0.1:7001

# Add node to cluster
redis-cli --cluster add-node 127.0.0.1:7004 127.0.0.1:7001

# Reshard cluster
redis-cli --cluster reshard 127.0.0.1:7001
```

### **Sentinel (High Availability):**
```sentinel.conf
# Sentinel configuration
sentinel monitor mymaster 192.168.1.100 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
sentinel auth-pass mymaster mypassword
```

---

## ⚡ **Performance Optimization**

### **Connection Management:**

#### **1. Connection Pooling**
```typescript
// Node.js connection pooling
import { createClient } from 'redis';

const client = createClient({
  host: 'localhost',
  port: 6379,
  // Connection pooling settings
  maxRetriesPerRequest: null,
  retryDelayOnFailover: 100,
  lazyConnect: true,
  // Pool settings
  max: 10,                    // Maximum connections
  min: 2,                     // Minimum connections
});
```

#### **2. Pipelining**
```typescript
// Send multiple commands at once
const pipeline = redis.multi();
pipeline.set('key1', 'value1');
pipeline.set('key2', 'value2');
pipeline.get('key1');
pipeline.get('key2');

const results = await pipeline.exec();
// Results: [OK, OK, 'value1', 'value2']
```

#### **3. Lua Scripting**
```lua
-- Atomic operations with Lua
local key = KEYS[1]
local current = redis.call('GET', key)
if current then
  redis.call('INCR', key .. ':counter')
  return current
else
  redis.call('SET', key, 'new-value')
  return 'new-value'
end
```

```typescript
// Execute Lua script
const script = `
  local key = KEYS[1]
  local current = redis.call('GET', key)
  if current then
    redis.call('INCR', key .. ':counter')
    return current
  else
    redis.call('SET', key, 'new-value')
    return 'new-value'
  end
`;

redis.eval(script, 1, 'mykey');
```

### **Query Optimization:**

#### **1. Scan vs Keys**
```bash
# DON'T do this in production (blocks server)
KEYS pattern:*

# DO this instead (non-blocking)
SCAN 0 MATCH pattern:* COUNT 100
```

#### **2. Batch Operations**
```typescript
// Instead of multiple round trips
for (const user of users) {
  await redis.set(`user:${user.id}`, JSON.stringify(user));
}

// Use batch operations
const pipeline = redis.multi();
users.forEach(user => {
  pipeline.set(`user:${user.id}`, JSON.stringify(user));
});
await pipeline.exec();
```

#### **3. Expiration Strategies**
```typescript
// Lazy expiration (default)
redis.setex('cache:key', 3600, 'value');

// Active expiration (background process)
redis.config('SET', 'hz', '10');  // Run expiration 10 times/second
```

---

## 🔒 **Security Best Practices**

### **Access Control:**

#### **1. Authentication**
```redis.conf
# Require password
requirepass your-secure-password

# Client authentication
redis-cli -a your-secure-password
```

#### **2. ACL (Access Control Lists) - Redis 6+**
```redis.conf
# Enable ACL
aclfile /etc/redis/acl.conf
```

```acl.conf
# ACL rules
user default off                    # Disable default user
user readonly on +GET +SCAN        # Read-only user
user app on +@all ~app:*           # App user with key pattern
user admin on allcommands allkeys  # Admin user
```

### **Network Security:**

#### **1. Bind to Specific Interface**
```redis.conf
# Only listen on localhost
bind 127.0.0.1

# Multiple interfaces
bind 127.0.0.1 192.168.1.100
```

#### **2. TLS/SSL Encryption**
```redis.conf
# Enable TLS
tls-port 6380
tls-cert-file /path/to/redis.crt
tls-key-file /path/to/redis.key
tls-ca-cert-file /path/to/ca.crt

# Client certificate verification
tls-auth-clients yes
```

#### **3. Firewall Rules**
```bash
# Allow only specific IPs
ufw allow from 192.168.1.100 to any port 6379
ufw allow from 192.168.1.101 to any port 6379
ufw deny from 0.0.0.0/0 to any port 6379
```

### **Data Protection:**

#### **1. Encryption at Rest**
```bash
# Encrypt data files
cryptsetup luksFormat /dev/sdb1
cryptsetup luksOpen /dev/sdb1 redis-data
```

#### **2. Secure Key Management**
```typescript
// Don't hardcode sensitive data
// Use environment variables or secret management
const redisPassword = process.env.REDIS_PASSWORD;

// Rotate keys regularly
const keyRotation = {
  current: 'key-v1',
  previous: 'key-v0',
  next: 'key-v2'
};
```

---

## 📊 **Monitoring and Troubleshooting**

### **Built-in Monitoring:**

#### **1. INFO Command**
```bash
redis-cli INFO
# Returns comprehensive server statistics

redis-cli INFO memory      # Memory statistics
redis-cli INFO stats       # General statistics
redis-cli INFO replication # Replication info
redis-cli INFO persistence # Persistence info
```

#### **2. SLOWLOG**
```bash
# Enable slow log
redis-cli CONFIG SET slowlog-log-slower-than 10000  # Log queries > 10ms

# View slow queries
redis-cli SLOWLOG GET 10

# Get slow log length
redis-cli SLOWLOG LEN
```

#### **3. MONITOR Command**
```bash
# Monitor all commands in real-time
redis-cli MONITOR

# Monitor specific patterns
redis-cli --eval monitor_pattern.lua , pattern
```

### **External Monitoring:**

#### **1. Redis Exporter (Prometheus)**
```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'redis'
    static_configs:
      - targets: ['localhost:9121']
```

#### **2. Health Checks**
```bash
# Ping check
redis-cli --raw ping | grep -q PONG

# Memory usage check
memory_usage=$(redis-cli INFO memory | grep used_memory: | cut -d: -f2)
if [ "$memory_usage" -gt 1073741824 ]; then  # 1GB
  echo "Memory usage too high"
fi
```

### **Common Issues & Solutions:**

#### **1. Memory Issues**
```bash
# Check memory usage
redis-cli INFO memory

# Find large keys
redis-cli --bigkeys

# Clean up expired keys
redis-cli --scan --pattern "*:expired:*" | xargs redis-cli DEL

# Enable memory optimization
redis-cli CONFIG SET maxmemory-policy allkeys-lru
```

#### **2. Connection Issues**
```bash
# Check connection count
redis-cli INFO clients

# Set connection limits
redis-cli CONFIG SET maxclients 10000

# Monitor connection pool
redis-cli CLIENT LIST
```

#### **3. Performance Issues**
```bash
# Check command statistics
redis-cli INFO commandstats

# Find slow commands
redis-cli SLOWLOG GET 5

# Optimize data structures
redis-cli --eval optimize_hashes.lua
```

---

## 🚀 **Production Deployment**

### **Docker Deployment:**
```dockerfile
FROM redis:7-alpine

# Copy custom configuration
COPY redis.conf /etc/redis/redis.conf

# Create data directory
RUN mkdir -p /data
VOLUME /data

# Expose ports
EXPOSE 6379

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD redis-cli ping || exit 1

# Start Redis with custom config
CMD ["redis-server", "/etc/redis/redis.conf"]
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  redis:
    build: .
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    environment:
      - REDIS_PASSWORD=${REDIS_PASSWORD}
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 30s
      timeout: 10s
      retries: 3

volumes:
  redis-data:
```

### **Kubernetes Deployment:**
```yaml
# Redis StatefulSet
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
spec:
  replicas: 1
  serviceName: redis
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7-alpine
        ports:
        - containerPort: 6379
        volumeMounts:
        - name: redis-data
          mountPath: /data
        resources:
          requests:
            memory: "256Mi"
            cpu: "100m"
          limits:
            memory: "512Mi"
            cpu: "200m"
        livenessProbe:
          exec:
            command:
            - redis-cli
            - ping
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          exec:
            command:
            - redis-cli
            - ping
          initialDelaySeconds: 5
          periodSeconds: 5

  volumeClaimTemplates:
  - metadata:
      name: redis-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

### **High Availability Setup:**
```yaml
# Redis Sentinel for HA
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-sentinel
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: sentinel
        image: redis:7-alpine
        command:
        - redis-sentinel
        - /etc/redis/sentinel.conf
        volumeMounts:
        - name: sentinel-config
          mountPath: /etc/redis/
```

---

## ⚖️ **Redis vs Other Databases**

### **Redis vs Traditional RDBMS:**

| Feature | Redis | PostgreSQL | MySQL |
|---------|-------|------------|-------|
| **Data Model** | Key-Value + Structures | Relational | Relational |
| **Performance** | ~1ms | ~10-100ms | ~10-100ms |
| **Persistence** | Optional | Yes | Yes |
| **ACID** | Limited | Full | Full |
| **Scaling** | Horizontal | Vertical | Vertical |
| **Memory Usage** | High | Low | Low |
| **Query Language** | Commands | SQL | SQL |

### **Redis vs Other Caches:**

| Feature | Redis | Memcached | etcd |
|---------|-------|-----------|------|
| **Data Types** | Rich | Simple | Simple |
| **Persistence** | Yes | No | Yes |
| **Clustering** | Yes | Limited | Yes |
| **Pub/Sub** | Yes | No | Yes |
| **Transactions** | Yes | No | Yes |
| **Lua Scripting** | Yes | No | No |

### **When to Use Redis:**

#### **✅ Use Cases:**
- **Caching**: Session storage, API responses
- **Real-time Analytics**: Counters, leaderboards
- **Message Queues**: Job processing (BullMQ)
- **Rate Limiting**: API throttling
- **Geospatial**: Location-based services
- **Pub/Sub**: Real-time notifications

#### **❌ When NOT to Use:**
- **Complex Queries**: Use RDBMS
- **Large Datasets**: Memory constraints
- **ACID Transactions**: Use PostgreSQL
- **Data Relationships**: Use MongoDB
- **Text Search**: Use Elasticsearch

---

## 🔮 **Advanced Features**

### **1. Pub/Sub (Publish/Subscribe)**
```bash
# Subscribe to channel
SUBSCRIBE notifications

# Publish to channel
PUBLISH notifications "New user registered"

# Pattern subscription
PSUBSCRIBE notifications:*

# Channel info
PUBSUB CHANNELS
PUBSUB NUMSUB notifications
```

### **2. Transactions**
```bash
# Multi-command transaction
MULTI
SET user:123:balance 100
INCR user:456:balance
EXEC

# Watch for conflicts
WATCH user:123:balance
MULTI
INCR user:123:balance
EXEC  # Only succeeds if balance unchanged
```

### **3. Streams (Redis 5+)**
```bash
# Add to stream
XADD mystream * sensor-id 1234 temperature 19.8

# Read from stream
XREAD COUNT 10 STREAMS mystream 0

# Consumer groups
XGROUP CREATE mystream mygroup 0
XREADGROUP GROUP mygroup Alice COUNT 1 STREAMS mystream >

# Process messages
XACK mystream mygroup 1526569495631-0
```

### **4. Modules**
```bash
# RedisJSON for JSON operations
JSON.SET user:123 $ {"name":"John","age":30}
JSON.GET user:123 $.name

# RedisSearch for full-text search
FT.CREATE users ON JSON SCHEMA $.name AS name TEXT $.age AS age NUMERIC
FT.SEARCH users "John"

# RedisTimeSeries for time series data
TS.CREATE temperature:room1
TS.ADD temperature:room1 * 23.5
TS.RANGE temperature:room1 - + AGGREGATION avg 3600000
```

### **5. Active-Active Replication (Redis 7+)**
```redis.conf
# Active-Active replication
replicaof <peer1> 6379
replicaof <peer2> 6379

# Conflict resolution
replica-conflict-resolution last-write-wins
```

---

## 🌍 **Real-World Use Cases**

### **1. Session Management**
```typescript
class SessionManager {
  async createSession(userId: string, data: any): Promise<string> {
    const sessionId = uuidv4();
    const key = `session:${sessionId}`;

    await redis.setex(key, 3600, JSON.stringify({
      userId,
      data,
      createdAt: Date.now()
    }));

    return sessionId;
  }

  async getSession(sessionId: string): Promise<any | null> {
    const key = `session:${sessionId}`;
    const data = await redis.get(key);

    if (!data) return null;

    // Extend session
    await redis.expire(key, 3600);

    return JSON.parse(data);
  }
}
```

### **2. Rate Limiting**
```typescript
class RateLimiter {
  constructor(private windowMs: number = 60000, private maxRequests: number = 100) {}

  async isAllowed(key: string): Promise<boolean> {
    const redisKey = `ratelimit:${key}`;
    const now = Date.now();
    const windowStart = now - this.windowMs;

    // Remove old requests outside window
    await redis.zremrangebyscore(redisKey, 0, windowStart);

    // Count current requests
    const requestCount = await redis.zcard(redisKey);

    if (requestCount >= this.maxRequests) {
      return false;
    }

    // Add new request
    await redis.zadd(redisKey, now, now.toString());

    // Set expiration
    await redis.expire(redisKey, Math.ceil(this.windowMs / 1000));

    return true;
  }
}
```

### **3. Distributed Locks**
```typescript
class DistributedLock {
  constructor(private redis: any, private lockTimeout: number = 30000) {}

  async acquire(lockKey: string, ownerId: string): Promise<boolean> {
    const result = await redis.set(
      lockKey,
      ownerId,
      'NX',     // Only set if not exists
      'EX',     // Set expiration
      Math.ceil(this.lockTimeout / 1000)
    );

    return result === 'OK';
  }

  async release(lockKey: string, ownerId: string): Promise<boolean> {
    // Use Lua script for atomic check-and-delete
    const script = `
      if redis.call('GET', KEYS[1]) == ARGV[1] then
        return redis.call('DEL', KEYS[1])
      else
        return 0
      end
    `;

    const result = await redis.eval(script, 1, lockKey, ownerId);
    return result === 1;
  }

  async extend(lockKey: string, ownerId: string, extensionMs: number = 30000): Promise<boolean> {
    const script = `
      if redis.call('GET', KEYS[1]) == ARGV[1] then
        return redis.call('PEXPIRE', KEYS[1], ARGV[2])
      else
        return 0
      end
    `;

    const result = await redis.eval(script, 1, lockKey, ownerId, extensionMs);
    return result === 1;
  }
}
```

### **4. Cache Management**
```typescript
class CacheManager {
  constructor(private redis: any, private defaultTtl: number = 300) {}

  async get<T>(key: string): Promise<T | null> {
    const data = await redis.get(key);
    return data ? JSON.parse(data) : null;
  }

  async set<T>(key: string, value: T, ttl?: number): Promise<void> {
    const serialized = JSON.stringify(value);
    const expireTime = ttl || this.defaultTtl;

    await redis.setex(key, expireTime, serialized);
  }

  async invalidate(pattern: string): Promise<number> {
    const keys = await redis.keys(pattern);
    if (keys.length > 0) {
      return await redis.del(...keys);
    }
    return 0;
  }

  async getOrSet<T>(
    key: string,
    factory: () => Promise<T>,
    ttl?: number
  ): Promise<T> {
    let cached = await this.get<T>(key);
    if (cached) {
      return cached;
    }

    const fresh = await factory();
    await this.set(key, fresh, ttl);
    return fresh;
  }
}
```

---

## 🎯 **Conclusion**

Redis is a **versatile, high-performance data store** that excels at:

### **✅ Strengths:**
- **Lightning-fast** in-memory operations
- **Rich data structures** for complex use cases
- **Persistence options** for durability
- **Clustering** for scalability
- **Pub/Sub** for real-time communication
- **Atomic operations** for consistency

### **🔧 Best Practices:**
- **Choose the right data structure** for your use case
- **Implement proper expiration** strategies
- **Use connection pooling** for performance
- **Monitor memory usage** and set appropriate limits
- **Secure your Redis instance** with authentication and TLS
- **Plan for high availability** with replication and clustering

### **🚀 Production Ready:**
Redis is **battle-tested** in production environments and powers some of the world's largest applications including:
- **Twitter** (timeline caching)
- **GitHub** (job queues, caching)
- **Stack Overflow** (session storage, caching)
- **Instagram** (feed caching)
- **Uber** (real-time location tracking)

This comprehensive guide covers everything from basic operations to advanced production deployment, making it the ultimate Redis reference for developers working with Sentinel or any other Redis-powered application! 🚀

---

## 📚 **Additional Resources**

- [Redis Official Documentation](https://redis.io/documentation)
- [Redis Commands Reference](https://redis.io/commands)
- [Redis Best Practices](https://redis.io/topics/best-practices)
- [Redis Modules](https://redis.io/modules)
- [Redis University](https://university.redis.com/)
