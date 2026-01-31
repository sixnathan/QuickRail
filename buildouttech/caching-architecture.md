# Rail Route/Price Caching Architecture

**Target**: Sub-800ms response time | **Strategy**: Multi-layer caching with route overlap detection

---

## Architecture Diagram

```
                                    [EU Users]
                                        |
                              [Edge Cache Layer]
                         Cloudflare/Fastly (50-100ms)
                          - Popular route responses
                          - Static fare tables
                          - Station metadata
                                        |
                    +-------------------+-------------------+
                    |                                       |
              [Cache HIT]                            [Cache MISS]
              Return ~80ms                                  |
                                                           v
                                              [Application Layer]
                                              +------------------+
                                              |   Route Cache    |
                                              |   (Dragonfly)    |
                                              |                  |
                                              | - Graph segments |
                                              | - Precomputed    |
                                              |   Pareto sets    |
                                              | - Overlap index  |
                                              +------------------+
                                                       |
                                    +------------------+------------------+
                                    |                                     |
                              [Cache HIT]                          [Cache MISS]
                              Return ~150ms                               |
                                                                         v
                                                          [Routing Engine]
                                                          RAPTOR/CSA (1-10ms)
                                                                  |
                                                                  v
                                                          [Price Engine]
                                                          API calls (200-400ms)
                                                                  |
                                                                  v
                                                          [Cache Write]
                                                          Store result + indexes
```

---

## 1. Route Caching Strategy

### Overlap Detection Algorithm

```
Key Insight: Routes A->C and B->C share segment B->C
Cache segments, not just full routes
```

**Segment-Based Caching**:
```
Route: London -> Paris -> Berlin
Cached Segments:
  - london_paris_{departure_window}
  - paris_berlin_{departure_window}
  - london_berlin_{departure_window} (full route)
```

**Overlap Index Structure**:
```
Key: segment:{origin}_{destination}_{time_bucket}
Value: {
  pareto_set: [...optimal journeys...],
  graph_edges: [...connection data...],
  computed_at: timestamp
}
```

**Time Bucketing**: 15-minute windows for departure times
- Query for 14:23 -> hits bucket 14:15-14:30
- Reduces key cardinality by 4x vs exact times

### Graph Caching

| Cache Layer | Data | TTL | Invalidation |
|-------------|------|-----|--------------|
| **Connection Graph** | Station links, transfer times | 24h | Schedule update event |
| **Pareto Sets** | Precomputed optimal routes | 1h | Price change event |
| **Segment Results** | Individual route segments | 15min | Staleness-based |

### TTL Policies

| Data Type | TTL | Rationale |
|-----------|-----|-----------|
| Static graph (stations, transfers) | 24h | Changes infrequently |
| Schedule-based routes | 4h | GTFS updates 4-6x/day |
| Price-inclusive results | 15min | Price volatility |
| Disruption-affected routes | 0 (no cache) | Real-time accuracy required |

---

## 2. Price Caching Strategy

### Staleness Tolerance Matrix

| Price Type | Max Staleness | Strategy |
|------------|---------------|----------|
| Base fares | 1h | TTL-based |
| Dynamic pricing | 5min | SWR (stale-while-revalidate) |
| Promotional fares | 15min | Event-driven + TTL fallback |
| Sold-out segments | 0 | Real-time check required |

### Invalidation Triggers

1. **Event-Driven** (primary):
   - Price update webhook from rail operator
   - Inventory threshold crossed (low availability)
   - Promotion start/end

2. **TTL Fallback** (safety net):
   - Max age regardless of events
   - Jittered TTL: `base_ttl + random(0, 0.2 * base_ttl)`

3. **SWR for Hot Routes**:
   - Serve stale immediately
   - Background refresh
   - Max stale age: 2x TTL

### Cache Key Design

```
price:{operator}:{origin}:{dest}:{date}:{class}:{flex_type}
Example: price:eurostar:london:paris:2024-03-15:standard:anytime
```

---

## 3. In-Memory Store Recommendation

### Comparison: Redis vs KeyDB vs Dragonfly

| Factor | Redis Cluster | KeyDB | Dragonfly |
|--------|--------------|-------|-----------|
| **Throughput** | Baseline | ~5x (claimed) | 25x (claimed) |
| **Memory Efficiency** | 1x | 1x | 0.5x (better compression) |
| **Multi-threading** | Per-shard | Native | Native |
| **Active Maintenance** | Yes | Stale (1.5y+) | Active |
| **API Compatibility** | N/A | 100% Redis | 100% Redis |
| **License** | AGPL v3 | BSD-3 | BSD-3 |
| **Ops Complexity** | High (cluster) | Medium | Low (single node scales) |

### Recommendation: **Dragonfly**

**Why**:
1. Single-node handles TB-scale (no cluster complexity)
2. Native multi-threading (uses all cores efficiently)
3. 80% lower infrastructure cost vs Redis cluster
4. Active development, production-proven
5. Drop-in Redis replacement (same client libs)

**Caveat**: Requires modern kernel (5.x+). For legacy infra, use Redis Cluster.

**Alternative**: If on AWS with managed preference, ElastiCache for Redis with cluster mode.

---

## 4. Edge Caching for EU Latency

### Provider Comparison

| Factor | Cloudflare | Fastly |
|--------|------------|--------|
| **EU PoPs** | 100+ | 60+ |
| **Purge Speed** | <30s | <150ms |
| **API Caching** | Workers + Cache API | Surrogate Keys |
| **P95 Latency (EU)** | ~40ms | ~25ms |
| **Cost** | Lower | Higher |
| **Best For** | Security + CDN bundle | Pure performance |

### Recommendation: **Fastly for API, Cloudflare for general CDN**

- Fastly's surrogate keys enable surgical invalidation
- Sub-150ms purge critical for price changes
- Cloudflare Workers for edge compute (A/B, geolocation routing)

### Edge Cache Strategy

```
Cache-Control: public, max-age=60, stale-while-revalidate=300
Surrogate-Key: route:lon-par price:eurostar:2024-03
```

**Cacheable at Edge**:
- Popular route results (top 1000 O-D pairs)
- Station autocomplete responses
- Static fare tables
- Disruption status (short TTL)

**Never Cache at Edge**:
- Personalized prices (logged-in users)
- Booking/payment flows
- Real-time seat availability

---

## 5. Cache Warming Strategy

### Popular Route Identification

```sql
-- Top routes by query volume (run daily)
SELECT origin, destination,
       COUNT(*) as queries,
       AVG(response_time_ms) as avg_latency
FROM route_queries
WHERE timestamp > NOW() - INTERVAL 7 DAYS
GROUP BY origin, destination
ORDER BY queries DESC
LIMIT 1000;
```

### Warming Schedule

| Trigger | Action | Routes |
|---------|--------|--------|
| **Daily 04:00 UTC** | Full warm of top 1000 O-D pairs | All departure windows, next 7 days |
| **Schedule Update** | Re-warm affected routes | Routes using updated services |
| **Deploy** | Warm critical routes | Top 100 O-D pairs |
| **Price Event** | Re-warm affected routes | Routes with price change |

### Implementation

```typescript
interface WarmingJob {
  routes: Array<{origin: string, destination: string}>
  departureDates: string[]  // next 7 days
  timeWindows: string[]     // 06:00, 09:00, 12:00, 15:00, 18:00, 21:00
  priority: 'critical' | 'standard' | 'background'
}

// Warm in parallel with rate limiting
async function warmCache(job: WarmingJob): Promise<void> {
  const tasks = []
  for (const route of job.routes) {
    for (const date of job.departureDates) {
      for (const time of job.timeWindows) {
        tasks.push(computeAndCacheRoute(route, date, time))
      }
    }
  }
  // Process with concurrency limit
  await pLimit(50)(tasks)
}
```

### Warming Metrics

- Target: 95% of queries hit warm cache
- Monitor: Cache hit rate by route popularity tier
- Alert: Hit rate < 80% on top 100 routes

---

## 6. Latency Estimates

### Request Flow Latencies

| Scenario | P50 | P95 | P99 |
|----------|-----|-----|-----|
| **Edge HIT (warm)** | 50ms | 80ms | 120ms |
| **Dragonfly HIT** | 120ms | 180ms | 250ms |
| **Route compute (no price)** | 150ms | 220ms | 350ms |
| **Full compute (with price API)** | 400ms | 650ms | 900ms |
| **Cold path (everything misses)** | 600ms | 850ms | 1200ms |

### Achieving Sub-800ms

```
Target: P95 < 800ms

Strategy:
1. Edge cache: 60% of requests (P95 = 80ms)
2. Dragonfly cache: 30% of requests (P95 = 180ms)
3. Compute path: 10% of requests (P95 = 650ms)

Weighted P95 = 0.6(80) + 0.3(180) + 0.1(650) = 167ms
```

**Key Optimizations**:
1. Parallel price API calls (not sequential)
2. Precompute Pareto sets for popular routes
3. SWR at edge for near-instant responses
4. Circuit breaker on slow price APIs (serve cached)

---

## 7. Implementation Checklist

### Phase 1: Foundation
- [ ] Deploy Dragonfly (single node, 64GB+ RAM)
- [ ] Implement segment-based cache keys
- [ ] Add TTL policies per data type
- [ ] Set up cache metrics (hit rate, latency)

### Phase 2: Edge Layer
- [ ] Configure Fastly with surrogate keys
- [ ] Implement cache-control headers
- [ ] Set up purge webhooks from price system
- [ ] Deploy to EU PoPs

### Phase 3: Warming
- [ ] Build route popularity analytics
- [ ] Implement daily warming job
- [ ] Add deploy-time warming hook
- [ ] Set up event-driven re-warming

### Phase 4: Optimization
- [ ] A/B test TTL values
- [ ] Implement SWR for hot routes
- [ ] Add circuit breakers
- [ ] Monitor P95/P99 continuously

---

## References

- [Netflix Cache Warming at Scale](https://netflixtechblog.com/cache-warming-agility-for-a-stateful-service-2d3b1da82642)
- [AWS Caching Best Practices](https://aws.amazon.com/caching/best-practices/)
- [Fastly Surrogate Keys](https://www.fastly.com/documentation/solutions/tutorials/enabling-api-caching/)
- [Dragonfly vs Redis Benchmarks](https://github.com/centminmod/redis-comparison-benchmarks)
- [Cache Invalidation Strategies](https://www.designgurus.io/blog/cache-invalidation-strategies)
