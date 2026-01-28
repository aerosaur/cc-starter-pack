---
name: database-sql
description: Expert guidance for database design, SQL optimization, and data modeling with PostgreSQL, MySQL, and ORMs (Prisma, TypeORM). Covers schema design, indexing, query optimization, migrations, transactions, and performance tuning. Use when designing databases, writing complex queries, optimizing performance, or debugging database issues.
allowed-tools: Read, Grep, Glob, Edit, Write
---

# Database & SQL

Complete toolkit for database design, SQL development, and performance optimization with PostgreSQL and modern ORMs.

## Core Capabilities

### Database Design
- Entity-relationship modeling
- Normalization (1NF, 2NF, 3NF, BCNF)
- Denormalization strategies
- Schema design patterns
- Data type selection
- Constraint design (FK, UK, CHECK)
- Index strategy planning

### SQL Operations
- Complex SELECT queries
- JOINs (INNER, LEFT, RIGHT, FULL OUTER)
- Subqueries and CTEs (Common Table Expressions)
- Window functions (ROW_NUMBER, RANK, LAG, LEAD)
- Aggregate functions (GROUP BY, HAVING)
- UNION, INTERSECT, EXCEPT
- Transactions and ACID properties

### Performance Optimization
- Query optimization techniques
- Index design and analysis
- EXPLAIN plan interpretation
- Query profiling
- Connection pooling
- Database tuning parameters
- Partitioning strategies

### ORM Usage
- Prisma schema design
- Type-safe queries
- Migrations management
- Seeding strategies
- Relation loading (eager vs lazy)
- Raw SQL when needed
- Performance considerations

## Tech Stack

**Databases:** PostgreSQL (primary), MySQL, SQLite
**ORMs:** Prisma, TypeORM, Sequelize, Drizzle
**Tools:** pgAdmin, DBeaver, TablePlus
**Migration:** Prisma Migrate, Flyway, Liquibase
**Testing:** pg-mem, jest-mock-extended

## PostgreSQL Best Practices

### Schema Design
```sql
-- Use appropriate data types
CREATE TABLE users (
  id BIGSERIAL PRIMARY KEY,
  email VARCHAR(255) NOT NULL UNIQUE,
  created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
  metadata JSONB DEFAULT '{}'::jsonb
);

-- Add constraints
ALTER TABLE users
  ADD CONSTRAINT email_format CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}$');

-- Create indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_created_at ON users(created_at DESC);
CREATE INDEX idx_users_metadata ON users USING GIN (metadata);
```

### Query Optimization
```sql
-- Use EXPLAIN ANALYZE to profile queries
EXPLAIN ANALYZE
SELECT u.*, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.created_at > NOW() - INTERVAL '30 days'
GROUP BY u.id;

-- Use CTEs for readability
WITH recent_users AS (
  SELECT id, email
  FROM users
  WHERE created_at > NOW() - INTERVAL '30 days'
),
user_orders AS (
  SELECT user_id, COUNT(*) as order_count
  FROM orders
  WHERE user_id IN (SELECT id FROM recent_users)
  GROUP BY user_id
)
SELECT ru.*, COALESCE(uo.order_count, 0) as orders
FROM recent_users ru
LEFT JOIN user_orders uo ON ru.id = uo.user_id;
```

### Indexing Strategy
```sql
-- Single column index
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- Composite index (order matters!)
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- Partial index for common queries
CREATE INDEX idx_active_users ON users(email)
WHERE active = true;

-- Expression index
CREATE INDEX idx_users_lower_email ON users(LOWER(email));

-- Covering index (include columns)
CREATE INDEX idx_orders_covering ON orders(user_id)
INCLUDE (total, created_at);
```

## Prisma Patterns

### Schema Design
```prisma
// Define models with relations
model User {
  id        Int       @id @default(autoincrement())
  email     String    @unique
  profile   Profile?
  orders    Order[]
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt

  @@index([email])
  @@map("users")
}

model Profile {
  id      Int    @id @default(autoincrement())
  bio     String?
  userId  Int    @unique
  user    User   @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("profiles")
}

model Order {
  id        Int      @id @default(autoincrement())
  total     Decimal  @db.Decimal(10, 2)
  status    String
  userId    Int
  user      User     @relation(fields: [userId], references: [id])
  items     OrderItem[]
  createdAt DateTime @default(now())

  @@index([userId, status])
  @@map("orders")
}
```

### Query Patterns
```typescript
// Efficient relation loading
const users = await prisma.user.findMany({
  include: {
    profile: true,
    orders: {
      where: { status: 'completed' },
      take: 10,
      orderBy: { createdAt: 'desc' }
    }
  }
});

// Transactions
await prisma.$transaction(async (tx) => {
  const user = await tx.user.create({ data: userData });
  await tx.profile.create({ data: { userId: user.id, ...profileData } });
  return user;
});

// Raw SQL when needed
const result = await prisma.$queryRaw`
  SELECT u.*, COUNT(o.id) as order_count
  FROM users u
  LEFT JOIN orders o ON u.id = o.user_id
  GROUP BY u.id
  HAVING COUNT(o.id) > ${threshold}
`;

// Batch operations
await prisma.user.createMany({
  data: userData,
  skipDuplicates: true
});
```

## Performance Optimization

### Query Optimization Checklist
- [ ] Use indexes on WHERE clause columns
- [ ] Avoid SELECT *; specify needed columns
- [ ] Use LIMIT for pagination
- [ ] Use appropriate JOIN types
- [ ] Check EXPLAIN plan for sequential scans
- [ ] Consider materialized views for complex queries
- [ ] Use connection pooling
- [ ] Batch operations when possible
- [ ] Cache frequently accessed data
- [ ] Use prepared statements

### Index Analysis
```sql
-- Find missing indexes
SELECT schemaname, tablename, indexname, indexdef
FROM pg_indexes
WHERE schemaname = 'public'
ORDER BY tablename, indexname;

-- Find unused indexes
SELECT schemaname, tablename, indexname
FROM pg_stat_user_indexes
WHERE idx_scan = 0 AND indexrelname NOT LIKE 'pg_toast_%';

-- Index size
SELECT tablename, indexname, pg_size_pretty(pg_relation_size(indexrelid))
FROM pg_indexes
JOIN pg_stat_user_indexes USING (indexrelname)
WHERE schemaname = 'public'
ORDER BY pg_relation_size(indexrelid) DESC;
```

### Connection Pooling
```typescript
// Prisma connection pool
const prisma = new PrismaClient({
  datasources: {
    db: {
      url: process.env.DATABASE_URL + '?connection_limit=10'
    }
  }
});

// PgBouncer configuration
[databases]
mydb = host=localhost port=5432 dbname=mydb pool_size=25

[pgbouncer]
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 25
```

## Migrations Best Practices

### Prisma Migrations
```bash
# Create migration
npx prisma migrate dev --name add_user_profile

# Apply to production
npx prisma migrate deploy

# Reset database (dev only!)
npx prisma migrate reset

# Generate Prisma Client
npx prisma generate
```

### Safe Migration Patterns
```sql
-- Add column with default (safe)
ALTER TABLE users ADD COLUMN status VARCHAR(20) DEFAULT 'active';

-- Add NOT NULL constraint (safe with default)
ALTER TABLE users ADD COLUMN email VARCHAR(255) DEFAULT '' NOT NULL;

-- Add index concurrently (safe in production)
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);

-- Drop column (consider data loss!)
-- First: Verify column not used in code
-- Then: Remove from code
-- Finally: Drop column
ALTER TABLE users DROP COLUMN old_column;
```

## Data Integrity

### Constraints
```sql
-- Foreign key with cascade
ALTER TABLE orders
  ADD CONSTRAINT fk_orders_user
  FOREIGN KEY (user_id) REFERENCES users(id)
  ON DELETE CASCADE;

-- Check constraint
ALTER TABLE users
  ADD CONSTRAINT check_age
  CHECK (age >= 18 AND age < 120);

-- Unique constraint (composite)
ALTER TABLE user_roles
  ADD CONSTRAINT unique_user_role
  UNIQUE (user_id, role_id);
```

### Transactions
```typescript
// ACID transaction
await prisma.$transaction(
  async (tx) => {
    // Deduct from sender
    await tx.account.update({
      where: { id: fromAccountId },
      data: { balance: { decrement: amount } }
    });

    // Add to receiver
    await tx.account.update({
      where: { id: toAccountId },
      data: { balance: { increment: amount } }
    });

    // Log transaction
    await tx.transaction.create({
      data: { fromAccountId, toAccountId, amount }
    });
  },
  { isolationLevel: 'Serializable' }
);
```

## Common Queries

### Pagination
```sql
-- Offset pagination (simple but slow for large offsets)
SELECT * FROM users
ORDER BY created_at DESC
LIMIT 20 OFFSET 100;

-- Cursor pagination (fast, scalable)
SELECT * FROM users
WHERE created_at < '2024-01-01 00:00:00'
ORDER BY created_at DESC
LIMIT 20;
```

### Aggregations
```sql
-- Group by with aggregates
SELECT
  DATE_TRUNC('day', created_at) as date,
  COUNT(*) as user_count,
  AVG(age) as avg_age
FROM users
GROUP BY DATE_TRUNC('day', created_at)
HAVING COUNT(*) > 10
ORDER BY date DESC;
```

### Window Functions
```sql
-- Ranking
SELECT
  user_id,
  total,
  ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC) as order_rank
FROM orders;

-- Running totals
SELECT
  created_at,
  amount,
  SUM(amount) OVER (ORDER BY created_at) as running_total
FROM transactions;
```

## Debugging

```bash
# Show slow queries (PostgreSQL)
SELECT query, mean_exec_time, calls
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;

# Show active connections
SELECT * FROM pg_stat_activity
WHERE state = 'active';

# Kill long-running query
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE state = 'active' AND query_start < NOW() - INTERVAL '5 minutes';
```

## Backup & Recovery

```bash
# Backup database
pg_dump -h localhost -U postgres mydb > backup.sql

# Restore database
psql -h localhost -U postgres mydb < backup.sql

# Backup specific tables
pg_dump -h localhost -U postgres -t users -t orders mydb > tables_backup.sql
```

## Resources

- PostgreSQL Docs: https://www.postgresql.org/docs/
- Prisma Docs: https://www.prisma.io/docs/
- Use The Index, Luke: https://use-the-index-luke.com/
- Database Indexing Explained: https://www.freecodecamp.org/news/database-indexing-at-a-glance/
