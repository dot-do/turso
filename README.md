# turso.do

> Edge-Native SQLite. Distributed. AI-First.

Turso raised $30M to build "SQLite at the edge." But their edge is still centralized regions. Their replication is manual. Their DX is another CLI to learn. And their free tier is limited.

**turso.do** is the open-source alternative. True edge-native on Cloudflare Durable Objects. SQLite per-tenant. Automatic global replication. AI that understands your queries.

## AI-Native API

```typescript
import { turso } from 'turso.do'           // Full SDK
import { turso } from 'turso.do/tiny'      // Minimal client
import { turso } from 'turso.do/sql'       // SQL-only operations
```

Natural language IS the API:

```typescript
import { turso } from 'turso.do'

// Talk to it like a colleague
const users = await turso`users in California`
const active = await turso`orders this week over $100`
const metrics = await turso`revenue by month this year`

// Replication in plain English
await turso`replicate to us-west eu-west ap-southeast`
await turso`primary in us-east, read replicas everywhere`

// Schema evolution without migrations
await turso`add email_verified boolean to users`
await turso`rename column user_name to username in users`

// AI-powered optimization
await turso`what indexes would help these slow queries?`.suggest().apply()
await turso`analyze query patterns and optimize`
```

## The Problem

Turso promises SQLite at the edge but delivers:

| What Turso Offers | The Reality |
|-------------------|-------------|
| **"Edge"** | 30+ regions, but you pick primary |
| **Replication** | Manual setup, replica URLs to manage |
| **Free Tier** | 500 databases, 9GB storage, limited |
| **DX** | Another CLI (turso), another dashboard |
| **Embedded** | libSQL fork, not standard SQLite |
| **Branching** | Paid feature |
| **Point-in-Time** | Paid feature |

### The Architecture Problem

Turso's model:

```
Your App --> Turso Primary (one region) --> Turso Replicas (other regions)
                    |
            Still a round trip to primary for writes
```

turso.do's model:

```
Your App --> Durable Object (closest edge) --> SQLite (in the DO)
                    |
            Zero cold start. Writes at the edge. Always.
```

### The Replication Problem

Turso makes you think about regions:

```bash
turso db create my-db --location iad
turso db replicate my-db --location cdg
turso db replicate my-db --location sin
# Now manage multiple connection URLs
```

turso.do handles it automatically:

```typescript
await turso`replicate globally`
// Done. That's it.
```

### The Migration Problem

Turso uses standard migrations. Schema changes require:

1. Write migration files
2. Test locally
3. Deploy to staging
4. Hope production doesn't break
5. Roll back if it does

turso.do uses AI-powered schema evolution:

```typescript
// Just describe what you want
await turso`add subscription_tier to users`
await turso`split name into first_name and last_name for users`

// AI generates migration, validates safety, applies
```

## The Solution

**turso.do** reimagines distributed SQLite:

```
Turso                               turso.do
-----------------------------------------------------------------
30 regions you manage               Cloudflare's 300+ PoPs automatic
Pick your primary region            Primary is wherever you are
Manual replica URLs                 One URL, automatic routing
libSQL fork                         Standard SQLite
CLI + Dashboard                     Natural language API
Paid branching                      Built-in branching
Paid point-in-time                  Automatic PITR
$29/month Pro plan                  $0 - run your own
```

## One-Click Deploy

```bash
npx create-dotdo turso
```

Your own distributed SQLite. Running on your Cloudflare account. Global replication. Point-in-time recovery. All automatic.

```typescript
import { Turso } from 'turso.do'

export default Turso({
  name: 'my-app',
  domain: 'db.my-app.com',
  replication: 'global',       // or specify regions
  pointInTime: '7d',           // retention period
})
```

## Features

### Queries

```typescript
// Natural language queries
const users = await turso`active users this month`
const revenue = await turso`total revenue by product category`
const churn = await turso`users who haven't logged in for 30 days`

// AI infers the SQL
await turso`users in California`
// SELECT * FROM users WHERE state = 'CA'

await turso`top 10 products by revenue`
// SELECT p.*, SUM(o.amount) as revenue
// FROM products p JOIN orders o ON p.id = o.product_id
// GROUP BY p.id ORDER BY revenue DESC LIMIT 10

// Or write SQL directly
await turso.sql`SELECT * FROM users WHERE active = true`
```

### Replication

```typescript
// Global replication
await turso`replicate globally`

// Specific regions
await turso`replicate to us-west eu-central ap-tokyo`

// Read-your-writes consistency
await turso`primary writes, global reads`

// Check replication status
await turso`replication status`
// Returns: { primary: 'iad', replicas: ['cdg', 'sin', 'nrt'], lag: '12ms' }
```

### Migrations

```typescript
// Schema changes in plain English
await turso`add email_verified boolean to users default false`
await turso`add index on users(email)`
await turso`rename orders to purchases`

// AI validates before applying
await turso`drop column legacy_field from users`.validate()
// Returns: { safe: true, affected_rows: 0, warnings: [] }

// Branching for testing
const branch = await turso`branch for testing`
await branch`add experimental_feature to users`
await branch.test()
await branch.merge() // or branch.delete()
```

### Schema

```typescript
// Introspect your schema
await turso`describe schema`
await turso`what tables reference users?`
await turso`show foreign keys`

// AI-powered schema suggestions
await turso`how should I model a multi-tenant SaaS?`
await turso`suggest indexes for my query patterns`
await turso`what's missing from my schema?`
```

## Architecture

### Durable Object per Database

```
DatabaseDO (config, schema, access control)
  |
  +-- SQLite (transactional storage)
  |     |-- Tables, indexes, views
  |     +-- Point-in-time recovery log
  |
  +-- R2 (backup and archive)
  |     |-- Incremental snapshots
  |     +-- WAL archives
  |
  +-- ReplicationDO (per-region)
        |-- Read replica SQLite
        +-- Change stream consumer
```

### Storage Tiers

| Tier | Storage | Use Case | Query Speed |
|------|---------|----------|-------------|
| **Hot** | SQLite in DO | Active queries | <5ms |
| **Warm** | R2 + Index | Historical queries | <50ms |
| **Cold** | R2 Archive | Compliance/audit | <500ms |

### Replication Flow

```
Write Request
     |
     v
Primary DO (nearest edge)
     |
     +---> SQLite write
     |
     +---> Change event to Stream
               |
               v
         Global broadcast
               |
               +---> Replica DO (us-west)
               +---> Replica DO (eu-central)
               +---> Replica DO (ap-tokyo)
```

### Point-in-Time Recovery

Every transaction is logged. Recovery to any timestamp:

```typescript
// Restore to specific time
await turso`restore to 2024-01-15 14:30:00`

// Restore specific tables
await turso`restore users table to yesterday`

// Preview before restore
await turso`show changes since 2024-01-15`
```

## vs Turso

| Feature | Turso | turso.do |
|---------|-------|----------|
| **Regions** | 30 regions, manual | 300+ PoPs, automatic |
| **Primary Selection** | You choose | Automatic, nearest edge |
| **Replication** | Manual replica URLs | One URL, auto-routing |
| **SQLite Version** | libSQL fork | Standard SQLite |
| **Branching** | $29/month | Built-in, free |
| **Point-in-Time** | $29/month | Built-in, free |
| **Free Tier Databases** | 500 | Unlimited |
| **Free Tier Storage** | 9GB total | Per-database limits |
| **API** | SQL + HTTP | Natural language + SQL |
| **Schema Migrations** | Manual files | AI-powered evolution |
| **Lock-in** | Turso platform | Your Cloudflare account |
| **Open Source** | libSQL (Apache) | MIT licensed |

## Use Cases

### Multi-Tenant SaaS

```typescript
// Each tenant gets their own database
const tenantDb = await turso.database(`tenant-${tenantId}`)

// Query within tenant context
await tenantDb`active subscriptions`

// Cross-tenant analytics (admin only)
await turso.admin`total revenue across all tenants`
```

### Local-First Applications

```typescript
// Sync between client SQLite and edge
await turso`sync from local`
await turso`sync to local`

// Conflict resolution
await turso`sync with last-write-wins`
await turso`sync with merge function ${customMerge}`
```

### Serverless APIs

```typescript
// Zero cold start database access
export default {
  async fetch(request, env) {
    const data = await env.TURSO`users where active`
    return Response.json(data)
  }
}
```

### Analytics

```typescript
// Real-time analytics at the edge
await turso`page views by country today`
await turso`conversion funnel last 7 days`
await turso`cohort retention analysis`

// Automatic rollups
await turso`enable hourly rollups for analytics`
```

### Edge Computing

```typescript
// Database co-located with compute
// No network round trip to external database

// Read: <5ms
// Write: <10ms
// Transaction: <15ms
```

## Roadmap

### Core Database
- [x] SQLite per Durable Object
- [x] SQL query interface
- [x] Natural language queries
- [x] Transaction support
- [x] Prepared statements
- [ ] Full-text search
- [ ] JSON functions
- [ ] Virtual tables

### Replication
- [x] Primary election
- [x] Change data capture
- [x] Read replicas
- [x] Global distribution
- [ ] Multi-primary writes
- [ ] Conflict resolution
- [ ] Offline sync

### Recovery
- [x] WAL logging
- [x] Point-in-time recovery
- [x] Incremental backups
- [x] Snapshot restore
- [ ] Cross-region failover
- [ ] Disaster recovery

### Schema
- [x] AI-powered migrations
- [x] Schema introspection
- [x] Index suggestions
- [x] Branching
- [ ] Schema versioning
- [ ] Rollback support
- [ ] Data validation

### Optimization
- [x] Query analysis
- [x] Index recommendations
- [x] Slow query logging
- [ ] Auto-indexing
- [ ] Query caching
- [ ] Connection pooling

## Contributing

turso.do is open source under the MIT license.

```bash
git clone https://github.com/dotdo/turso.do
cd turso.do
pnpm install
pnpm test
```

## License

MIT License - Database freedom for everyone.

---

<p align="center">
  <strong>SQLite at the edge. For real this time.</strong>
  <br />
  True edge-native. AI-powered. Zero config replication.
  <br /><br />
  <a href="https://turso.do">Website</a> |
  <a href="https://docs.turso.do">Docs</a> |
  <a href="https://discord.gg/dotdo">Discord</a> |
  <a href="https://github.com/dotdo/turso.do">GitHub</a>
</p>
