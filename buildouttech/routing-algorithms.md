# Rail Journey Routing Algorithms: Research & MVP Recommendations

## Executive Summary

**Key Insight**: Rail schedules are deterministic. LLMs are overkill for standard routing - use proven algorithmic approaches. Reserve LLM capabilities for disruption handling, natural language understanding, and complex multi-constraint scenarios.

**MVP Recommendation**: Start with **RAPTOR** or **CSA** for core routing. Both are simple to implement, require no preprocessing, and handle multi-criteria optimization (time + transfers) natively.

---

## Algorithm Comparison Matrix

| Algorithm | Query Time | Preprocessing | Multi-Criteria | Real-time Updates | Implementation Complexity |
|-----------|------------|---------------|----------------|-------------------|---------------------------|
| **Dijkstra** | 100-500ms | None | Limited | Easy | Low |
| **A*** | 50-200ms | None | Limited | Easy | Low |
| **RAPTOR** | 1-10ms | None | Native | Easy | Medium |
| **CSA** | 1-5ms | None | Native | Very Easy | Low |
| **TBTR** | <1ms | 30s-2min | Native | Moderate | Medium |
| **Contraction Hierarchies** | <1ms | Minutes-Hours | Limited | Hard | High |
| **Transfer Patterns** | <1ms | Hours (quadratic) | Limited | Hard | High |

---

## Detailed Algorithm Analysis

### 1. Graph-Based Classical Algorithms

#### Dijkstra's Algorithm
- **How it works**: Explores nodes in order of distance from source
- **Complexity**: O((V+E)log V) with binary heap
- **Pros**: Simple, correct, no preprocessing
- **Cons**: Too slow for large networks (100-500ms per query), doesn't handle time-dependent costs well
- **Verdict**: Not suitable for production transit routing

#### A* Algorithm
- **How it works**: Dijkstra + heuristic (e.g., geographic distance)
- **Performance**: 2-5x faster than Dijkstra
- **Pros**: Better than Dijkstra, still simple
- **Cons**: Heuristics don't work well for transit (transfer times, schedules)
- **Verdict**: Marginal improvement, still not recommended

#### Contraction Hierarchies (CH)
- **How it works**: Precomputes shortcuts by contracting "unimportant" nodes
- **Query time**: Sub-millisecond
- **Preprocessing**: Minutes to hours depending on network size
- **Pros**: Extremely fast queries, proven at scale (Google Maps for roads)
- **Cons**: Designed for static road networks, struggles with time-dependent transit schedules, complex to update
- **Verdict**: Better for road networks than rail; not recommended for MVP

### 2. Transit-Specific Algorithms (Recommended)

#### RAPTOR (Round-Based Public Transit Routing)
- **How it works**: Operates in rounds (k rounds = k-1 transfers), scans routes not edges
- **Query time**: 1-10ms on large networks (London: order of magnitude faster than Dijkstra-based)
- **Preprocessing**: None required
- **Multi-criteria**: Native support for arrival time + transfers (McRAPTOR extends to fare zones, etc.)

**Advantages**:
- Simple data structures, memory efficient
- Pareto-optimal journeys out of the box
- Easy to handle flexible departure times
- Parallelizable
- Works in fully dynamic scenarios (no preprocessing = easy updates)
- Used by OpenTripPlanner

**Reference Implementation**: [planarnetwork/raptor](https://github.com/planarnetwork/raptor)

#### Connection Scan Algorithm (CSA)
- **How it works**: Scans connections (individual vehicle departures) sorted by departure time
- **Query time**: 1-5ms
- **Preprocessing**: None (just sort connections once)
- **Core insight**: Replace Dijkstra's priority queue with a sorted list - faster array operations

**Advantages**:
- Extremely simple to implement
- Excellent cache efficiency
- Easy to incorporate delays (just update affected connections)
- Handles the Minimum Expected Arrival Time (MEAT) problem for uncertain delays
- Used by Deutsche Bahn

**Reference Implementation**: [dbsystel/practical-csa](https://github.com/dbsystel/practical-csa)

#### Trip-Based Transit Routing (TBTR)
- **How it works**: Models trips as nodes, transfers as edges; BFS-like traversal
- **Query time**: <1ms (70ms for full 24-hour profiles)
- **Preprocessing**: 30s-2min

**Advantages**:
- Fastest query times among non-heavily-preprocessed algorithms
- Good for computing travel time profiles (all departure times in a day)
- HypTBTR variant: 23-37% faster than base TBTR

**Cons**: Requires preprocessing, more complex than RAPTOR/CSA

### 3. Heavy Preprocessing Algorithms

#### Transfer Patterns
- **Query time**: Few milliseconds
- **Preprocessing**: Quadratic complexity (hours on large networks)
- **Used by**: Google Maps (transit)
- **Verdict**: Overkill for MVP; consider later for massive scale

#### ULTRA (UnLimited TRAnsfers)
- **Purpose**: Multi-modal routing (transit + walking/cycling/e-scooter)
- **Performance**: Fastest known algorithm for multi-modal journey planning
- **Verdict**: Consider for future multi-modal expansion

---

## Multi-Criteria Optimization

### Criteria for Rail Routing
1. **Arrival time** - Primary
2. **Number of transfers** - Critical for user experience
3. **Price/fare** - Complex due to fare zone structures
4. **Walking distance** - For first/last mile
5. **Carbon footprint** - Emerging criterion

### Algorithmic Approaches

**Pareto-Optimal Search**: RAPTOR and CSA naturally return Pareto-optimal sets (no journey is dominated on all criteria).

**Price Optimization**: Recent research shows price-optimal routing (POEAP) can be solved in <400ms using McRAPTOR with Conditional Fare Networks. Note: PSEAP (price-sensitive earliest arrival) is NP-hard in general.

**Weighted Sum**: OTP uses a configurable cost function mixing time, waiting, walking, transfers. Simple but loses Pareto-optimality.

### MVP Recommendation
Start with **arrival time + transfers** (native in RAPTOR/CSA). Add price as post-processing filter on Pareto set rather than integrated optimization.

---

## Precomputation Strategies

### For Fast Queries
| Strategy | Preprocessing Time | Query Speedup | Update Flexibility |
|----------|-------------------|---------------|-------------------|
| None (RAPTOR/CSA) | 0 | Baseline | Instant |
| TBTR trip-transfers | 30s-2min | 10x | Minutes |
| Hub-based | Minutes | 100x | Hard |
| Transfer Patterns | Hours | 1000x | Very Hard |
| Arc-Flag TB | Minutes | 100x | Moderate |

### MVP Strategy
**No precomputation** for launch. RAPTOR/CSA give 1-10ms queries which is more than sufficient for user-facing applications. Add TBTR preprocessing if analytics/batch queries needed.

---

## Open-Source Implementations

### Recommended for Evaluation

| Project | Algorithms | Language | License | Best For |
|---------|-----------|----------|---------|----------|
| **[OpenTripPlanner](https://github.com/opentripplanner/OpenTripPlanner)** | RAPTOR, A* | Java | LGPL | Full-featured trip planner |
| **[R5](https://github.com/conveyal/r5)** | RAPTOR variants | Java | MIT | Accessibility analysis, batch routing |
| **[Navitia](https://github.com/hove-io/navitia)** | RAPTOR | C++/Python | AGPL | Journey planner framework |
| **[transnetlab/transit-routing](https://github.com/transnetlab/transit-routing)** | RAPTOR, TBTR, HypRAPTOR | Python | - | Research, prototyping |
| **[gtfsrouter](https://github.com/UrbanAnalyst/gtfsrouter)** | CSA | R | GPL | One-to-many routing |
| **[practical-csa](https://github.com/dbsystel/practical-csa)** | CSA | - | - | Reference implementation |

### Comparison: OTP vs R5 vs Navitia

| Aspect | OpenTripPlanner | R5 | Navitia |
|--------|----------------|-----|---------|
| **Focus** | Individual trip planning | Accessibility analysis | Passenger information systems |
| **Architecture** | Single component | Analysis-focused | Modular, distributed |
| **Best for MVP** | Yes - if need full planner | No - overkill | Yes - if building API |
| **Deployment** | Simple (single JAR) | Moderate | Complex (distributed) |

---

## LLM vs Algorithmic Routing

### When Algorithmic Routing Wins (90%+ of cases)

| Scenario | Why Algorithmic |
|----------|-----------------|
| Standard A→B journey | Deterministic schedules, solved problem |
| Departure time optimization | RAPTOR handles natively |
| Multi-transfer journeys | Pareto-optimal search |
| Bulk/batch queries | Speed (1000s of queries/second) |
| Real-time updates | CSA handles delays elegantly |

### When LLM Adds Value

| Scenario | Why LLM |
|----------|---------|
| **Disruption reasoning** | "Train cancelled, find alternatives considering..." |
| **Natural language queries** | "Get me to Paris avoiding the strike" |
| **Complex constraints** | "Wheelchair accessible, quiet coach, near dining car" |
| **Ambiguous destinations** | "The museum near that famous tower" |
| **Itinerary explanation** | "Why is this route better than..." |
| **Multi-modal recommendations** | "Should I take the train or fly given my luggage?" |

### Recommended Architecture

```
User Query → NLU Layer (LLM) → Structured Query
                                    ↓
                            Algorithmic Router (RAPTOR/CSA)
                                    ↓
Results → Post-processing → Explanation Layer (LLM, optional)
```

**LLM handles**:
- Query understanding and intent extraction
- Entity resolution (station names, dates, constraints)
- Result explanation and recommendations
- Disruption impact analysis

**Algorithm handles**:
- Actual route computation
- Schedule lookups
- Transfer optimization
- Real-time delay incorporation

---

## MVP Implementation Roadmap

### Phase 1: Core Routing (Week 1-2)
1. **Choose CSA** for simplicity or **RAPTOR** for better multi-criteria support
2. Ingest GTFS data for target rail networks
3. Implement basic A→B routing with departure time
4. Return Pareto-optimal journeys (time vs transfers)

### Phase 2: Real-time Updates (Week 3)
1. Integrate GTFS-Realtime feed
2. Update connection/trip data with delays
3. Handle cancellations by marking trips unavailable

### Phase 3: LLM Integration (Week 4)
1. NLU layer for query understanding
2. Station/date entity extraction
3. Constraint parsing (accessibility, preferences)
4. Pass structured query to algorithmic router

### Phase 4: Disruption Handling (Future)
1. LLM-powered disruption impact analysis
2. Alternative route suggestions with reasoning
3. Proactive rebooking recommendations

---

## Key Recommendations

1. **Don't use LLM for routing computation** - Schedules are deterministic; algorithms are faster, cheaper, and provably correct.

2. **Start with RAPTOR or CSA** - No preprocessing, fast enough (<10ms), handles multi-criteria, easy to update.

3. **Use LLM for understanding and explaining** - Query parsing, constraint extraction, result explanation, disruption handling.

4. **Avoid heavy preprocessing initially** - Transfer Patterns and Contraction Hierarchies add complexity without benefit at MVP scale.

5. **Leverage GTFS ecosystem** - Standard data format means existing tools work out of the box.

6. **Plan for multi-criteria from day one** - Users care about transfers and price, not just arrival time.

---

## References

### Academic Papers
- [RAPTOR: Round-Based Public Transit Routing](https://www.microsoft.com/en-us/research/wp-content/uploads/2012/01/raptor_alenex.pdf) - Microsoft Research
- [Connection Scan Algorithm](https://arxiv.org/abs/1703.05997) - Dibbelt et al.
- [Trip-Based Public Transit Routing](https://arxiv.org/abs/1504.07149) - Witt
- [Price Optimal Routing in Public Transportation](https://arxiv.org/abs/2204.01326)
- [Contraction Hierarchies](https://link.springer.com/chapter/10.1007/978-3-540-68552-4_24)

### Open Source Projects
- [OpenTripPlanner](https://github.com/opentripplanner/OpenTripPlanner)
- [R5 Routing Engine](https://github.com/conveyal/r5)
- [Navitia](https://github.com/hove-io/navitia)
- [transit-routing (Python implementations)](https://github.com/transnetlab/transit-routing)
- [planarnetwork/raptor](https://github.com/planarnetwork/raptor)
- [practical-csa](https://github.com/dbsystel/practical-csa)

### Documentation
- [GTFS Reference](https://gtfs.org/documentation/realtime/reference/)
- [OTP vs R5 Comparison](https://github.com/conveyal/r5/issues/575)
- [Navitia vs OTP Comparison](https://github.com/hove-io/navitia/wiki/OpenTripPlanner-and-Navitia-comparison)
