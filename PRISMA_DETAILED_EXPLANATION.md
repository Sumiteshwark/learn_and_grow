# 🚀 **Prisma ORM Deep Dive - Complete Technical Guide**

## 📋 **Table of Contents**
- [What is Prisma?](#what-is-prisma)
- [Why Prisma in Sentinel?](#why-prisma-in-sentinel)
- [Schema Definition](#schema-definition)
- [Data Modeling](#data-modeling)
- [Database Migrations](#database-migrations)
- [Client API & Queries](#client-api--queries)
- [Relationships & Joins](#relationships--joins)
- [Advanced Querying](#advanced-querying)
- [Transactions & ACID](#transactions--acid)
- [Performance Optimization](#performance-optimization)
- [Caching Strategies](#caching-strategies)
- [Error Handling](#error-handling)
- [Production Deployment](#production-deployment)
- [Migration Strategies](#migration-strategies)
- [Monitoring & Observability](#monitoring--observability)
- [Troubleshooting Guide](#troubleshooting-guide)
- [Best Practices](#best-practices)
- [Prisma vs Other ORMs](#prisma-vs-other-orms)

---

## 🤔 **What is Prisma?**

**Prisma** is a next-generation **ORM (Object-Relational Mapping)** tool that simplifies database operations through a type-safe, auto-generated query builder. It provides:

### **Core Features:**
- ✅ **Type-Safe**: Full TypeScript support with generated types
- ✅ **Schema-Driven**: Single source of truth for database schema
- ✅ **Auto-Generated**: Client API generated from schema
- ✅ **Migration System**: Database schema versioning
- ✅ **Introspection**: Generate schema from existing database
- ✅ **Multi-Database**: PostgreSQL, MySQL, SQLite, SQL Server, MongoDB

### **Why Prisma?**
```typescript
// Traditional SQL (Error-prone)
const users = await sql`
  SELECT u.*, p.title as project_title
  FROM users u
  LEFT JOIN projects p ON u.id = p.user_id
  WHERE u.email = ?
`;

// Prisma (Type-safe & Intuitive)
const users = await prisma.user.findMany({
  where: { email: email },
  include: { projects: true }
});
```

---

## 🎯 **Why Prisma in Sentinel?**

### **Sentinel's Database Requirements:**
```typescript
// Sentinel needs to handle:
// 1. Complex test execution data with relationships
// 2. User management and authentication
// 3. Test results storage and querying
// 4. Multi-tenant data isolation
// 5. Performance-critical read operations
// 6. Type-safe database operations
// 7. Schema evolution and migrations
```

### **Prisma Benefits for Sentinel:**
- ✅ **Type Safety**: Prevents runtime errors with compile-time checks
- ✅ **Developer Experience**: Auto-completion and IntelliSense
- ✅ **Performance**: Optimized queries with connection pooling
- ✅ **Migrations**: Safe schema evolution with rollback support
- ✅ **Relationships**: Easy handling of complex data relationships
- ✅ **Multi-Database**: Flexible database backend selection

### **Real-World Impact:**
```typescript
// Before: Raw SQL queries (Error-prone)
const testResult = await db.query(`
  SELECT tr.*, t.name as test_name, u.email as user_email
  FROM test_results tr
  JOIN tests t ON tr.test_id = t.id
  JOIN users u ON tr.user_id = u.id
  WHERE tr.id = $1
`, [testResultId]);

// After: Type-safe Prisma queries (Reliable)
const testResult = await prisma.testResult.findUnique({
  where: { id: testResultId },
  include: {
    test: { select: { name: true } },
    user: { select: { email: true } }
  }
});
```

---

## 🏗️ **Schema Definition**

### **Prisma Schema Structure:**
```prisma
// schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  role      UserRole @default(USER)

  // Relations
  tests         Test[]
  testResults   TestResult[]
  organizations Organization[]

  // Metadata
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Test {
  id          String   @id @default(cuid())
  name        String
  description String?
  type        TestType

  // Relations
  userId      String
  user        User     @relation(fields: [userId], references: [id])

  testResults TestResult[]
  tags        Tag[]

  // Metadata
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}

model TestResult {
  id        String     @id @default(cuid())
  status    TestStatus
  duration  Int?
  error     String?

  // Relations
  testId    String
  test      Test       @relation(fields: [testId], references: [id])

  userId    String
  user      User       @relation(fields: [userId], references: [id])

  // Metadata
  createdAt DateTime   @default(now())
  updatedAt DateTime   @updatedAt
}

enum UserRole {
  ADMIN
  USER
  VIEWER
}

enum TestType {
  SMOKE
  REGRESSION
  PERFORMANCE
  SECURITY
}

enum TestStatus {
  PENDING
  RUNNING
  PASSED
  FAILED
  CANCELLED
}
```

### **Advanced Schema Features:**
```prisma
model TestExecution {
  id          String     @id @default(cuid())
  testId      String
  status      ExecutionStatus @default(PENDING)

  // JSON fields for flexible data
  configuration Json?
  results       Json?

  // Arrays
  tags         String[]

  // Composite unique constraints
  @@unique([testId, status])

  // Indexes for performance
  @@index([status, createdAt])
  @@index([testId])

  // Full-text search
  @@fulltext([configuration])
}

model AuditLog {
  id        String   @id @default(cuid())
  action    String
  entity    String
  entityId  String
  userId    String?
  changes   Json

  // Partitioning for large tables
  @@map("audit_logs")
}
```

---

## 🔧 **Data Modeling**

### **Entity Relationships:**
```typescript
// One-to-One Relationship
model User {
  id          String  @id @default(cuid())
  email       String  @unique
  profile     Profile?
}

model Profile {
  id          String  @id @default(cuid())
  bio         String?
  avatar      String?
  userId      String  @unique
  user        User    @relation(fields: [userId], references: [id])
}

// One-to-Many Relationship
model Organization {
  id          String @id @default(cuid())
  name        String
  users       User[]
}

// Many-to-Many Relationship
model Test {
  id    String @id @default(cuid())
  name  String
  tags  Tag[]
}

model Tag {
  id    String @id @default(cuid())
  name  String
  tests Test[]
}
```

### **Advanced Modeling Patterns:**

#### **Self-Referencing Relationships:**
```prisma
model TestSuite {
  id        String      @id @default(cuid())
  name      String
  parentId  String?
  parent    TestSuite?  @relation("TestSuiteHierarchy", fields: [parentId], references: [id])
  children  TestSuite[] @relation("TestSuiteHierarchy")
}
```

#### **Polymorphic Relationships:**
```prisma
model Comment {
  id        String   @id @default(cuid())
  content   String
  authorId  String

  // Polymorphic fields
  commentableId   String
  commentableType String

  // Union type for type safety
  @@index([commentableType, commentableId])
}
```

#### **Composite Types:**
```prisma
type Address {
  street  String
  city    String
  state   String
  zipCode String
  country String
}

model User {
  id      String  @id @default(cuid())
  address Address
}
```

---

## 🔄 **Database Migrations**

### **Migration Workflow:**
```bash
# 1. Initialize Prisma (if not done)
npx prisma init

# 2. Create migration
npx prisma migrate dev --name add-user-preferences

# 3. Apply to production
npx prisma migrate deploy

# 4. Generate client
npx prisma generate
```

### **Migration Files:**
```sql
-- 20240101120000_add_user_preferences/migration.sql
-- CreateTable
CREATE TABLE "UserPreferences" (
    "id" TEXT NOT NULL,
    "userId" TEXT NOT NULL,
    "theme" TEXT NOT NULL DEFAULT 'light',
    "notifications" BOOLEAN NOT NULL DEFAULT true,
    "createdAt" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP,
    "updatedAt" TIMESTAMP(3) NOT NULL,

    CONSTRAINT "UserPreferences_pkey" PRIMARY KEY ("id")
);

-- CreateIndex
CREATE UNIQUE INDEX "UserPreferences_userId_key" ON "UserPreferences"("userId");

-- AddForeignKey
ALTER TABLE "UserPreferences" ADD CONSTRAINT "UserPreferences_userId_fkey" FOREIGN KEY ("userId") REFERENCES "User"("id") ON DELETE CASCADE ON UPDATE CASCADE;
```

### **Migration Best Practices:**
```typescript
// Migration script with data transformation
const migration = async () => {
  // Rename column with data migration
  await prisma.$executeRaw`
    ALTER TABLE "User" RENAME COLUMN "firstName" TO "givenName";
  `;

  // Update existing data
  await prisma.user.updateMany({
    data: {
      givenName: prisma.user.firstName // This won't work - use raw SQL
    }
  });

  // Raw SQL for complex transformations
  await prisma.$executeRaw`
    UPDATE "User"
    SET "givenName" = "firstName"
    WHERE "givenName" IS NULL;
  `;
};
```

### **Migration Rollback:**
```typescript
// Custom rollback script
const rollback = async () => {
  // Drop new table
  await prisma.$executeRaw`DROP TABLE IF EXISTS "UserPreferences";`;

  // Restore renamed column
  await prisma.$executeRaw`
    ALTER TABLE "User" RENAME COLUMN "givenName" TO "firstName";
  `;
};
```

---

## 🔍 **Client API & Queries**

### **Basic CRUD Operations:**
```typescript
const prisma = new PrismaClient();

// Create
const user = await prisma.user.create({
  data: {
    email: 'john@example.com',
    name: 'John Doe',
    role: 'USER'
  }
});

// Read
const user = await prisma.user.findUnique({
  where: { email: 'john@example.com' }
});

const users = await prisma.user.findMany({
  where: { role: 'USER' },
  orderBy: { createdAt: 'desc' },
  take: 10
});

// Update
const updatedUser = await prisma.user.update({
  where: { id: 'user-123' },
  data: { name: 'John Smith' }
});

// Delete
await prisma.user.delete({
  where: { id: 'user-123' }
});
```

### **Query Builder Features:**
```typescript
// Complex filtering
const tests = await prisma.test.findMany({
  where: {
    AND: [
      { status: 'ACTIVE' },
      { createdAt: { gte: new Date('2024-01-01') } },
      {
        OR: [
          { name: { contains: 'smoke' } },
          { tags: { some: { name: 'critical' } } }
        ]
      }
    ]
  },
  include: {
    user: true,
    testResults: {
      orderBy: { createdAt: 'desc' },
      take: 5
    }
  }
});

// Aggregation
const stats = await prisma.testResult.groupBy({
  by: ['status'],
  _count: { status: true },
  where: { createdAt: { gte: new Date(Date.now() - 24 * 60 * 60 * 1000) } }
});

// Raw queries for complex operations
const complexResult = await prisma.$queryRaw`
  SELECT
    DATE(createdAt) as date,
    status,
    COUNT(*) as count
  FROM "TestResult"
  WHERE createdAt >= NOW() - INTERVAL '30 days'
  GROUP BY DATE(createdAt), status
  ORDER BY date DESC;
`;
```

---

## 🔗 **Relationships & Joins**

### **Eager Loading (include):**
```typescript
// Single relation
const user = await prisma.user.findUnique({
  where: { id: 'user-123' },
  include: { profile: true }
});

// Multiple relations
const test = await prisma.test.findUnique({
  where: { id: 'test-123' },
  include: {
    user: true,
    testResults: true,
    tags: true
  }
});

// Nested includes
const organization = await prisma.organization.findUnique({
  where: { id: 'org-123' },
  include: {
    users: {
      include: {
        tests: {
          include: {
            testResults: {
              where: { status: 'PASSED' }
            }
          }
        }
      }
    }
  }
});
```

### **Lazy Loading (select):**
```typescript
// Selective field loading
const user = await prisma.user.findUnique({
  where: { id: 'user-123' },
  select: {
    id: true,
    email: true,
    name: true,
    // Exclude sensitive data
    // password: false
  }
});

// Computed fields
const testWithStats = await prisma.test.findUnique({
  where: { id: 'test-123' },
  select: {
    id: true,
    name: true,
    _count: {
      select: {
        testResults: true
      }
    }
  }
});
```

### **Advanced Relationship Queries:**
```typescript
// Many-to-many with filtering
const testWithFilteredTags = await prisma.test.findUnique({
  where: { id: 'test-123' },
  include: {
    tags: {
      where: {
        name: { in: ['smoke', 'critical'] }
      }
    }
  }
});

// Self-referencing queries
const testSuiteWithChildren = await prisma.testSuite.findUnique({
  where: { id: 'suite-123' },
  include: {
    children: {
      include: {
        children: true // Recursive loading
      }
    }
  }
});
```

---

## 🔬 **Advanced Querying**

### **Filtering & Sorting:**
```typescript
// Advanced filtering
const advancedQuery = await prisma.test.findMany({
  where: {
    AND: [
      { status: 'ACTIVE' },
      {
        OR: [
          { name: { contains: 'login' } },
          { description: { contains: 'authentication' } }
        ]
      },
      { createdAt: { gte: new Date('2024-01-01') } },
      { user: { role: 'ADMIN' } }
    ]
  },
  orderBy: [
    { priority: 'desc' },
    { createdAt: 'desc' }
  ],
  take: 20,
  skip: 0
});
```

### **Aggregations & Grouping:**
```typescript
// Count operations
const totalTests = await prisma.test.count();
const activeTests = await prisma.test.count({
  where: { status: 'ACTIVE' }
});

// Group by operations
const testsByStatus = await prisma.test.groupBy({
  by: ['status'],
  _count: { status: true },
  orderBy: { _count: { status: 'desc' } }
});

// Advanced aggregations
const testStats = await prisma.testResult.aggregate({
  where: { createdAt: { gte: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000) } },
  _count: { id: true },
  _avg: { duration: true },
  _min: { duration: true },
  _max: { duration: true }
});
```

### **Raw SQL Queries:**
```typescript
// Raw query for complex operations
const complexStats = await prisma.$queryRaw`
  WITH test_durations AS (
    SELECT
      test_id,
      AVG(duration) as avg_duration,
      COUNT(*) as total_runs
    FROM "TestResult"
    WHERE status = 'PASSED'
    GROUP BY test_id
  )
  SELECT
    t.name,
    td.avg_duration,
    td.total_runs
  FROM "Test" t
  JOIN test_durations td ON t.id = td.test_id
  WHERE td.avg_duration > 1000
  ORDER BY td.avg_duration DESC;
`;

// Raw execute for DDL operations
await prisma.$executeRaw`
  CREATE INDEX CONCURRENTLY IF NOT EXISTS "test_status_created_at_idx"
  ON "Test" ("status", "createdAt");
`;
```

---

## 🔐 **Transactions & ACID**

### **Implicit Transactions:**
```typescript
// Single operation (automatic transaction)
const user = await prisma.user.create({
  data: {
    email: 'john@example.com',
    name: 'John Doe',
    profile: {
      create: {
        bio: 'Software Engineer'
      }
    }
  }
});
```

### **Explicit Transactions:**
```typescript
// Interactive transaction
const result = await prisma.$transaction(async (tx) => {
  // Create user
  const user = await tx.user.create({
    data: { email: 'john@example.com', name: 'John Doe' }
  });

  // Create test
  const test = await tx.test.create({
    data: {
      name: 'Login Test',
      userId: user.id
    }
  });

  // Create test result
  const testResult = await tx.testResult.create({
    data: {
      testId: test.id,
      userId: user.id,
      status: 'PASSED'
    }
  });

  return { user, test, testResult };
});
```

### **Batch Transactions:**
```typescript
// Batch operations
const batchResult = await prisma.$transaction([
  prisma.user.create({ data: { email: 'user1@example.com' } }),
  prisma.user.create({ data: { email: 'user2@example.com' } }),
  prisma.test.create({ data: { name: 'Batch Test', userId: 'user-123' } })
]);
```

### **Transaction Options:**
```typescript
// Transaction with timeout
const result = await prisma.$transaction(
  async (tx) => {
    // Complex operations
    return await tx.user.update({
      where: { id: 'user-123' },
      data: { name: 'Updated Name' }
    });
  },
  {
    maxWait: 5000,      // Max time to wait for transaction
    timeout: 10000,     // Max time for transaction execution
    isolationLevel: Prisma.TransactionIsolationLevel.Serializable
  }
);
```

---

## ⚡ **Performance Optimization**

### **Connection Pooling:**
```typescript
// Optimized Prisma Client configuration
const prisma = new PrismaClient({
  log: ['query', 'info', 'warn', 'error'],
  datasources: {
    db: {
      url: process.env.DATABASE_URL,
    },
  },
});

// Connection pool settings (environment variables)
process.env.DATABASE_URL = 'postgresql://user:pass@localhost:5432/db?connection_limit=10&pool_timeout=0&connection_timeout=60000';
```

### **Query Optimization:**
```typescript
// Efficient pagination
const users = await prisma.user.findMany({
  take: 50,
  skip: 0,
  select: {
    id: true,
    email: true,
    name: true
    // Only select needed fields
  }
});

// Cursor-based pagination for large datasets
const users = await prisma.user.findMany({
  take: 50,
  cursor: { id: 'user-123' },
  orderBy: { id: 'asc' }
});

// Optimized includes
const test = await prisma.test.findUnique({
  where: { id: 'test-123' },
  include: {
    user: {
      select: { id: true, name: true, email: true }
    },
    testResults: {
      where: { status: 'PASSED' },
      orderBy: { createdAt: 'desc' },
      take: 10
    }
  }
});
```

### **Indexing Strategy:**
```prisma
model Test {
  id          String   @id @default(cuid())
  name        String
  status      String
  createdAt   DateTime @default(now())

  // Single column indexes
  @@index([name])
  @@index([status])

  // Composite indexes
  @@index([status, createdAt])
  @@index([createdAt, status])

  // Unique indexes
  @@unique([name, status])

  // Full-text search indexes
  @@fulltext([name])
}
```

### **Query Analysis:**
```typescript
// Enable query logging
const prisma = new PrismaClient({
  log: [
    { emit: 'event', level: 'query' },
    { emit: 'stdout', level: 'error' },
    { emit: 'stdout', level: 'info' },
    { emit: 'stdout', level: 'warn' }
  ]
});

// Listen to query events
prisma.$on('query', (e) => {
  console.log('Query: ' + e.query);
  console.log('Duration: ' + e.duration + 'ms');
});
```

---

## 💾 **Caching Strategies**

### **Prisma with Redis Caching:**
```typescript
class CachedPrismaClient {
  constructor(private prisma: PrismaClient, private redis: Redis) {}

  async findUserWithCache(id: string) {
    const cacheKey = `user:${id}`;

    // Try cache first
    const cached = await this.redis.get(cacheKey);
    if (cached) {
      return JSON.parse(cached);
    }

    // Fetch from database
    const user = await this.prisma.user.findUnique({
      where: { id },
      include: { profile: true }
    });

    // Cache result (TTL: 5 minutes)
    if (user) {
      await this.redis.setex(cacheKey, 300, JSON.stringify(user));
    }

    return user;
  }

  async invalidateUserCache(id: string) {
    await this.redis.del(`user:${id}`);
  }
}
```

### **Application-Level Caching:**
```typescript
// Cache frequently accessed data
class TestCache {
  private cache = new Map<string, { data: any; expiry: number }>();

  async getTest(id: string) {
    const cached = this.cache.get(id);
    if (cached && cached.expiry > Date.now()) {
      return cached.data;
    }

    const test = await prisma.test.findUnique({ where: { id } });
    if (test) {
      this.cache.set(id, {
        data: test,
        expiry: Date.now() + 5 * 60 * 1000 // 5 minutes
      });
    }

    return test;
  }

  invalidate(id: string) {
    this.cache.delete(id);
  }
}
```

---

## 🚨 **Error Handling**

### **Prisma Error Types:**
```typescript
import { PrismaClientKnownRequestError, PrismaClientValidationError } from '@prisma/client';

try {
  const user = await prisma.user.create({
    data: { email: 'invalid-email' }
  });
} catch (error) {
  if (error instanceof PrismaClientKnownRequestError) {
    // Known database errors
    switch (error.code) {
      case 'P2002':
        // Unique constraint violation
        throw new Error('Email already exists');
      case 'P2025':
        // Record not found
        throw new Error('User not found');
      default:
        throw new Error(`Database error: ${error.message}`);
    }
  } else if (error instanceof PrismaClientValidationError) {
    // Validation errors
    throw new Error('Invalid data provided');
  } else {
    // Unknown errors
    console.error('Unexpected error:', error);
    throw new Error('Internal server error');
  }
}
```

### **Custom Error Handling:**
```typescript
class PrismaErrorHandler {
  static handle(error: any): never {
    console.error('Prisma Error:', error);

    if (error instanceof PrismaClientKnownRequestError) {
      throw this.handleKnownError(error);
    }

    if (error instanceof PrismaClientValidationError) {
      throw this.handleValidationError(error);
    }

    throw new Error('Database operation failed');
  }

  private static handleKnownError(error: PrismaClientKnownRequestError) {
    const errorMap = {
      P2002: 'Unique constraint violation',
      P2025: 'Record not found',
      P2028: 'Transaction API error',
      P2014: 'Invalid data',
    };

    const message = errorMap[error.code as keyof typeof errorMap] || 'Database error';
    return new DatabaseError(message, error.code);
  }

  private static handleValidationError(error: PrismaClientValidationError) {
    return new ValidationError('Invalid data provided');
  }
}

// Usage
try {
  await prisma.user.create({ data: userData });
} catch (error) {
  PrismaErrorHandler.handle(error);
}
```

---

## 🚀 **Production Deployment**

### **Environment Configuration:**
```typescript
// production.ts
const prisma = new PrismaClient({
  datasources: {
    db: {
      url: process.env.DATABASE_URL,
    },
  },
  log: process.env.NODE_ENV === 'development'
    ? ['query', 'info', 'warn', 'error']
    : ['error'],
});

// Connection pool settings for production
process.env.DATABASE_URL = 'postgresql://user:pass@host:5432/db?connection_limit=20&pool_timeout=20&connection_timeout=60000';
```

### **Docker Deployment:**
```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci --only=production

# Copy source code
COPY . .

# Generate Prisma client
RUN npx prisma generate

# Run migrations
RUN npx prisma migrate deploy

# Build application
RUN npm run build

EXPOSE 3000

CMD ["npm", "start"]
```

### **Kubernetes Deployment:**
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sentinel-api
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: api
        image: sentinel/api:latest
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: database-secret
              key: url
        resources:
          requests:
            memory: "256Mi"
            cpu: "100m"
          limits:
            memory: "512Mi"
            cpu: "500m"
```

### **Database Connection Pooling:**
```typescript
// Connection pool configuration
const prisma = new PrismaClient({
  datasources: {
    db: {
      url: process.env.DATABASE_URL,
    },
  },
});

// Environment variables for connection pooling
// PostgreSQL: connection_limit=10&pool_timeout=0
// MySQL: connectionLimit=10&acquireTimeout=60000
// SQLite: No additional parameters needed
```

---

## 🔄 **Migration Strategies**

### **Zero-Downtime Migrations:**
```typescript
// Migration with data transformation
const zeroDowntimeMigration = async () => {
  // 1. Add new column (nullable)
  await prisma.$executeRaw`
    ALTER TABLE "User" ADD COLUMN "newEmail" TEXT;
  `;

  // 2. Backfill data in batches
  const batchSize = 1000;
  let hasMore = true;

  while (hasMore) {
    const users = await prisma.user.findMany({
      where: { newEmail: null },
      take: batchSize,
      select: { id: true, email: true }
    });

    if (users.length === 0) {
      hasMore = false;
      break;
    }

    // Update in batches
    await prisma.$transaction(
      users.map(user =>
        prisma.user.update({
          where: { id: user.id },
          data: { newEmail: user.email }
        })
      )
    );
  }

  // 3. Make column non-nullable
  await prisma.$executeRaw`
    ALTER TABLE "User" ALTER COLUMN "newEmail" SET NOT NULL;
  `;

  // 4. Rename columns
  await prisma.$executeRaw`
    ALTER TABLE "User" DROP COLUMN "email";
    ALTER TABLE "User" RENAME COLUMN "newEmail" TO "email";
  `;
};
```

### **Rollback Strategies:**
```typescript
// Migration rollback script
const rollbackMigration = async () => {
  // Reverse the migration steps
  await prisma.$executeRaw`
    ALTER TABLE "User" RENAME COLUMN "email" TO "newEmail";
    ALTER TABLE "User" ADD COLUMN "email" TEXT;
    UPDATE "User" SET "email" = "newEmail";
    ALTER TABLE "User" DROP COLUMN "newEmail";
  `;
};
```

---

## 📊 **Monitoring & Observability**

### **Query Performance Monitoring:**
```typescript
// Enable detailed logging
const prisma = new PrismaClient({
  log: [
    {
      emit: 'event',
      level: 'query',
    },
  ],
});

// Monitor query performance
prisma.$on('query', (e) => {
  if (e.duration > 1000) { // Log slow queries
    console.warn(`Slow query (${e.duration}ms): ${e.query}`);
  }
});
```

### **Health Checks:**
```typescript
// Database health check
const healthCheck = async () => {
  try {
    await prisma.$queryRaw`SELECT 1`;
    return { status: 'healthy', timestamp: new Date() };
  } catch (error) {
    console.error('Database health check failed:', error);
    return { status: 'unhealthy', timestamp: new Date(), error: error.message };
  }
};

// Connection pool metrics
const getConnectionMetrics = async () => {
  const metrics = await prisma.$metrics.json();
  return {
    activeConnections: metrics.counters[0].value,
    idleConnections: metrics.gauges[0].value,
    totalQueries: metrics.counters[1].value,
    queryDuration: metrics.histograms[0].value,
  };
};
```

### **Custom Metrics:**
```typescript
class PrismaMetrics {
  private metrics = {
    queryCount: 0,
    slowQueries: 0,
    errorCount: 0,
    connectionPoolSize: 0
  };

  recordQuery(duration: number, error?: Error) {
    this.metrics.queryCount++;

    if (duration > 1000) {
      this.metrics.slowQueries++;
    }

    if (error) {
      this.metrics.errorCount++;
    }
  }

  getMetrics() {
    return { ...this.metrics };
  }
}
```

---

## 🔧 **Troubleshooting Guide**

### **Common Issues:**

#### **Connection Pool Exhaustion:**
```typescript
// Symptoms: "Can't reach database server" errors
// Solution: Increase connection pool size
process.env.DATABASE_URL = 'postgresql://user:pass@host:5432/db?connection_limit=20';

// Monitor connection usage
const metrics = await prisma.$metrics.json();
console.log('Active connections:', metrics.pools[0].active);
```

#### **Slow Queries:**
```typescript
// Enable query logging
const prisma = new PrismaClient({
  log: [{ emit: 'event', level: 'query' }]
});

// Add indexes for slow queries
await prisma.$executeRaw`
  CREATE INDEX CONCURRENTLY "user_email_idx" ON "User" ("email");
  CREATE INDEX CONCURRENTLY "test_status_created_at_idx" ON "Test" ("status", "createdAt");
`;
```

#### **Migration Issues:**
```bash
# Check migration status
npx prisma migrate status

# Resolve migration conflicts
npx prisma migrate resolve --applied 20240101120000_migration_name

# Reset database (development only)
npx prisma migrate reset
```

#### **Schema Drift:**
```bash
# Check schema drift
npx prisma db pull

# Re-sync schema
npx prisma db push

# Generate new client
npx prisma generate
```

---

## 🎯 **Best Practices**

### **1. Schema Design:**
- Use meaningful names for models and fields
- Define appropriate data types
- Add constraints and validations
- Use enums for fixed values
- Plan for future extensibility

### **2. Query Optimization:**
- Select only needed fields
- Use includes judiciously
- Implement proper pagination
- Add appropriate indexes
- Monitor query performance

### **3. Error Handling:**
- Handle Prisma errors appropriately
- Provide meaningful error messages
- Implement proper logging
- Use transactions for data consistency
- Plan for graceful degradation

### **4. Performance:**
- Use connection pooling
- Implement caching strategies
- Optimize database queries
- Monitor resource usage
- Plan for scaling

### **5. Security:**
- Never expose sensitive data in queries
- Use parameterized queries
- Implement proper authentication
- Sanitize user inputs
- Regular security audits

---

## ⚖️ **Prisma vs Other ORMs**

### **Prisma vs TypeORM:**
```typescript
// Prisma (Type-safe, schema-driven)
const users = await prisma.user.findMany({
  where: { role: 'ADMIN' },
  include: { posts: true }
});

// TypeORM (Manual type definitions)
const userRepository = dataSource.getRepository(User);
const users = await userRepository.find({
  where: { role: 'ADMIN' },
  relations: ['posts']
});
```

### **Prisma vs Sequelize:**
```typescript
// Prisma (Generated client)
const user = await prisma.user.create({
  data: { email: 'user@example.com' }
});

// Sequelize (Manual model definitions)
const User = sequelize.define('User', {
  email: DataTypes.STRING
});
const user = await User.create({ email: 'user@example.com' });
```

### **Prisma vs Raw SQL:**
```typescript
// Prisma (Type-safe abstraction)
const users = await prisma.user.findMany({
  where: { createdAt: { gte: new Date('2024-01-01') } },
  orderBy: { createdAt: 'desc' },
  take: 10
});

// Raw SQL (Error-prone, no type safety)
const users = await db.query(`
  SELECT * FROM users
  WHERE created_at >= $1
  ORDER BY created_at DESC
  LIMIT 10
`, ['2024-01-01']);
```

### **When to Choose Prisma:**
- ✅ TypeScript projects
- ✅ Complex relationships
- ✅ Schema-driven development
- ✅ Multi-database support
- ✅ Strong developer experience

### **When NOT to Choose Prisma:**
- ❌ Legacy projects without TypeScript
- ❌ Simple CRUD applications
- ❌ Heavy customization requirements
- ❌ Very specific database features

---

## 📚 **Additional Resources**

- [Prisma Official Documentation](https://www.prisma.io/docs)
- [Prisma Studio](https://www.prisma.io/studio) - Database GUI
- [Prisma Migrate](https://www.prisma.io/migrate) - Migration tool
- [Prisma Client API Reference](https://www.prisma.io/docs/reference/api-reference/prisma-client-reference)
- [Prisma Best Practices](https://www.prisma.io/docs/guides/performance-and-optimization)

---

## 🎉 **Conclusion**

Prisma provides Sentinel with a **modern, type-safe ORM** that simplifies database operations while maintaining excellent performance and developer experience. The implementation includes:

- ✅ **Type-Safe Database Operations** with full TypeScript support
- ✅ **Schema-Driven Development** with single source of truth
- ✅ **Advanced Querying** with relationships and aggregations
- ✅ **Migration System** with versioning and rollback support
- ✅ **Performance Optimization** with connection pooling and caching
- ✅ **Production Readiness** with health checks and monitoring
- ✅ **Developer Experience** with auto-completion and IntelliSense

This Prisma implementation enables Sentinel to handle complex database operations reliably and efficiently, supporting the platform's requirements for test automation data management, user management, and multi-tenant operations.
