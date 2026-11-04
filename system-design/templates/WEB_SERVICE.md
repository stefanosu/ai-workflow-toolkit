# Web Service Template

Use this template when designing a typical web application or API service.

---

## Architecture Overview

```
                        ┌─────────────┐
                        │     CDN     │ (static assets)
                        └──────┬──────┘
                               │
┌──────────┐            ┌──────┴──────┐
│ Clients  │───────────→│Load Balancer│
└──────────┘            └──────┬──────┘
                               │
                    ┌──────────┴──────────┐
                    │                     │
              ┌─────┴─────┐         ┌─────┴─────┐
              │  Web App  │         │  Web App  │
              │  Server 1 │         │  Server 2 │
              └─────┬─────┘         └─────┬─────┘
                    │                     │
                    └──────────┬──────────┘
                               │
                    ┌──────────┴──────────┐
                    │                     │
              ┌─────┴─────┐         ┌─────┴─────┐
              │   Cache   │         │  Database │
              │  (Redis)  │         │  Primary  │
              └───────────┘         └─────┬─────┘
                                          │
                                   ┌──────┴──────┐
                                   │   Replicas  │
                                   └─────────────┘
```

---

## Components

### Load Balancer
- **Purpose**: Distribute traffic, health checks, SSL termination
- **Options**: AWS ALB, Nginx, HAProxy
- **Scaling**: Managed services auto-scale

### Application Servers
- **Purpose**: Handle HTTP requests, business logic
- **Scaling**: Horizontal, behind load balancer
- **Considerations**: Stateless for easy scaling

### Cache Layer
- **Purpose**: Reduce database load, improve latency
- **Options**: Redis, Memcached
- **Pattern**: Cache-aside (check cache, fall back to DB)

### Database
- **Purpose**: Persistent data storage
- **Scaling**:
  - Read replicas for read-heavy workloads
  - Sharding for write-heavy workloads
- **Backup**: Regular snapshots, point-in-time recovery

### CDN
- **Purpose**: Serve static assets globally
- **Options**: CloudFlare, AWS CloudFront, Fastly
- **Cache**: Long TTL for versioned assets

---

## Data Flow

### Read Request
```
1. Client → Load Balancer
2. Load Balancer → App Server (round robin)
3. App Server → Check Cache
4. Cache HIT → Return cached data
5. Cache MISS → Query Database
6. Store in Cache → Return to client
```

### Write Request
```
1. Client → Load Balancer
2. Load Balancer → App Server
3. App Server → Validate input
4. App Server → Write to Database
5. Invalidate/Update Cache
6. Return success to client
```

---

## Scaling Strategy

### Vertical (Quick Wins)
- Upgrade database instance
- Add more memory to cache
- Use faster storage (SSD)

### Horizontal (Sustainable)
- Add more app servers behind LB
- Add read replicas for database
- Distribute cache (Redis Cluster)

### Optimization
- Add caching for hot data
- Optimize slow queries (indexes)
- Async processing for slow operations

---

## Failure Handling

| Component | Failure Mode | Mitigation |
|-----------|--------------|------------|
| App Server | Instance dies | LB routes to healthy instances |
| Database Primary | Crashes | Promote replica, restore from backup |
| Cache | Redis down | Fall back to database (slower) |
| Network | Partition | Circuit breaker, graceful degradation |

---

## Checklist

- [ ] Load balancer configured with health checks
- [ ] At least 2 app server instances
- [ ] Database backups scheduled
- [ ] Cache TTLs set appropriately
- [ ] Static assets on CDN
- [ ] Monitoring and alerting set up
- [ ] Logs centralized
- [ ] Rate limiting in place
