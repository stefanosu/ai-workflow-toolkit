# System Design Framework

A structured approach to designing systems from scratch.

---

## The Process

```
┌───────────────┐
│ 1. Clarify    │  What exactly are we building?
│   Requirements│  What are the constraints?
└───────┬───────┘
        ↓
┌───────────────┐
│ 2. Estimate   │  How much data? How many users?
│     Scale     │  What are the access patterns?
└───────┬───────┘
        ↓
┌───────────────┐
│ 3. High-Level │  What are the major components?
│    Design     │  How do they communicate?
└───────┬───────┘
        ↓
┌───────────────┐
│ 4. Deep Dive  │  Design critical components in detail
│  Components   │  Address bottlenecks
└───────┬───────┘
        ↓
┌───────────────┐
│ 5. Address    │  Failure modes, scaling, monitoring
│  Trade-offs   │  What breaks first?
└───────────────┘
```

---

## Step 1: Clarify Requirements

### Functional Requirements
What does the system DO?

- What are the core features?
- What inputs does it accept?
- What outputs does it produce?
- Who are the users?

### Non-Functional Requirements
What qualities must it HAVE?

| Category | Questions |
|----------|-----------|
| Scale | How many users? How much data? |
| Performance | Latency requirements? Throughput? |
| Availability | Uptime requirements? |
| Consistency | Strong vs eventual? |
| Durability | Data loss tolerance? |

### Constraints
- Technology constraints (existing stack, team skills)
- Budget constraints
- Timeline constraints
- Regulatory constraints

### Template
```markdown
## Requirements

### Functional
- Users can [action 1]
- System provides [feature 2]
- [...]

### Non-Functional
- Scale: [X users, Y requests/sec]
- Latency: [p99 < Z ms]
- Availability: [99.X%]
- Data: [X TB, growing Y% per month]

### Constraints
- Must use [existing system]
- Budget: [limits]
- Timeline: [deadline]
```

---

## Step 2: Estimate Scale

### Back-of-Envelope Calculations

**Users & Traffic**
```
Daily active users: ___
Actions per user per day: ___
Requests per second: DAU × actions / 86400
Peak multiplier: 2-10x average
```

**Storage**
```
Data per [item]: ___ bytes
Items created per day: ___
Storage per day: items × size
Storage per year: daily × 365
```

**Bandwidth**
```
Average request size: ___ KB
Average response size: ___ KB
Bandwidth: requests/sec × (req + resp size)
```

### Quick Reference
| Value | Approximation |
|-------|---------------|
| Seconds/day | 86,400 ≈ 100,000 |
| Seconds/month | 2.6M ≈ 2.5M |
| Seconds/year | 31M ≈ 30M |
| 1 KB | 1,000 bytes |
| 1 MB | 1,000 KB |
| 1 GB | 1,000 MB |
| 1 TB | 1,000 GB |

---

## Step 3: High-Level Design

### Identify Components

Common building blocks:
- **Load Balancer** - Distribute traffic
- **Web Servers** - Handle requests
- **Application Servers** - Business logic
- **Cache** - Reduce latency, database load
- **Database** - Persistent storage
- **Message Queue** - Async processing
- **CDN** - Static content delivery
- **Search** - Full-text search

### Draw the Diagram

```
[Clients] → [CDN] → [Load Balancer] → [Web Servers]
                                            ↓
                                     [Application Layer]
                                            ↓
                    [Cache] ← → [Database Primary] → [Replicas]
                                            ↓
                                     [Message Queue]
                                            ↓
                                     [Workers]
```

### Define Data Flow
1. Request comes in from client
2. Hits CDN (static) or load balancer (dynamic)
3. Web server handles request
4. Check cache for data
5. If miss, query database
6. Process and return response

---

## Step 4: Deep Dive Components

### Database Design

**Schema Design Questions**
- What entities exist?
- What are the relationships?
- What queries are most common?
- Read-heavy or write-heavy?

**SQL vs NoSQL**
| Use SQL When | Use NoSQL When |
|--------------|----------------|
| Complex queries | Simple queries at scale |
| ACID required | Eventual consistency OK |
| Structured data | Schema flexibility needed |
| Joins needed | Denormalization acceptable |

**Scaling Strategies**
- Vertical: Bigger machine
- Read replicas: Distribute reads
- Sharding: Distribute data
- Caching: Reduce load

### Caching Strategy

**What to Cache**
- Expensive computations
- Frequently accessed data
- Data that changes slowly

**Cache Patterns**
- Cache-aside: App manages cache
- Write-through: Write to cache and DB
- Write-behind: Write to cache, async to DB

**Eviction Policies**
- LRU: Least recently used
- LFU: Least frequently used
- TTL: Time-based expiration

### API Design

**REST Principles**
```
GET    /resources       - List all
GET    /resources/:id   - Get one
POST   /resources       - Create
PUT    /resources/:id   - Replace
PATCH  /resources/:id   - Update
DELETE /resources/:id   - Delete
```

**Pagination**
- Offset-based: `?page=2&limit=20`
- Cursor-based: `?cursor=abc123&limit=20`

---

## Step 5: Address Trade-offs

### CAP Theorem
Pick two:
- **Consistency**: All reads get latest write
- **Availability**: Every request gets a response
- **Partition tolerance**: System works despite network failures

In practice (networks fail), choose CP or AP.

### Scaling Trade-offs

| Approach | Pros | Cons |
|----------|------|------|
| Vertical scaling | Simple | Has limits |
| Horizontal scaling | Unlimited | Complex |
| Caching | Fast reads | Stale data |
| Sharding | Scales writes | Complex queries |
| Replication | Fault tolerant | Consistency lag |

### Failure Modes
- What happens if [component] fails?
- How do we detect failures?
- How do we recover?
- What data could be lost?

### Monitoring & Observability
- Metrics: Request rate, error rate, latency
- Logging: Structured logs, correlation IDs
- Alerting: When to wake someone up
- Dashboards: What to show

---

## Common Patterns

### Rate Limiting
```
┌─────────────────────────────────────┐
│ Token Bucket                        │
│                                     │
│ - Bucket holds N tokens             │
│ - Tokens added at rate R per second │
│ - Request consumes 1 token          │
│ - If empty, request rejected        │
└─────────────────────────────────────┘
```

### Circuit Breaker
```
         ┌──────────┐
    ┌───→│  Closed  │←──────────────┐
    │    └────┬─────┘               │
    │         │ failures > threshold│
    │         ↓                     │
    │    ┌──────────┐          success
    │    │   Open   │───────────────┤
    │    └────┬─────┘               │
    │         │ timeout             │
    │         ↓                     │
    │    ┌──────────┐               │
    └────│Half-Open │───────────────┘
         └──────────┘
```

### Event Sourcing
```
Instead of storing current state:
  { balance: 100 }

Store events:
  { type: "deposit", amount: 150 }
  { type: "withdraw", amount: 50 }

Replay events to reconstruct state.
```

---

## Design Checklist

### Before You Start
- [ ] Requirements clear?
- [ ] Scale estimated?
- [ ] Constraints identified?

### High-Level Design
- [ ] Major components identified?
- [ ] Data flow clear?
- [ ] APIs defined?

### Deep Dive
- [ ] Database schema designed?
- [ ] Critical components detailed?
- [ ] Caching strategy defined?

### Trade-offs
- [ ] Failure modes addressed?
- [ ] Scaling strategy clear?
- [ ] Monitoring planned?

### Wrap-up
- [ ] Can explain every component?
- [ ] Know what would break first?
- [ ] Have alternatives for each decision?
