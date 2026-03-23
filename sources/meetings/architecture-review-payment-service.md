# Architecture Review — Payment Service Extraction

> Date: February 14, 2026 | Type: Architecture Review | Duration: 120 minutes

## Meeting Details

| Field | Value |
|-------|-------|
| **Date** | February 14, 2026 |
| **Time** | 2:00 PM – 4:00 PM EST |
| **Type** | Monthly Architecture Review |
| **Facilitator** | David Park, VP Engineering |
| **Participants** | David Park (VP Eng), Ravi Krishnan (Sr Backend), James Liu (DevOps/SRE), Priya Sharma (Security), Alex Tran (Frontend Lead), Carlos Mendez (Sr Backend), Mei Chen (Staff Engineer), Fatima Malik (EM) |

## Summary

Monthly architecture review focused on the payment-service extraction from the monolith (INI-202). Reviewed three data synchronization options: dual-write, event sourcing with Kafka, and CDC via Debezium. Team selected CDC with Debezium as the migration strategy. Also reviewed service mesh options (Istio vs. Linkerd) and selected Istio for its traffic management capabilities.

This was a critical decision meeting. The data sync strategy determines how we maintain consistency between the monolith and the new service during the transition period (estimated 3–4 months of dual operation).

## Key Topics

### 1. Data Synchronization Strategy

Mei Chen presented three options for keeping the monolith and payment-service in sync during the strangler fig migration:

**Option A — Dual Write**: Application writes to both the monolith database and the new service database. Simplest to implement, but creates consistency risks. If one write fails, we have divergent state. Requires distributed transactions or saga pattern.

**Option B — Event Sourcing**: Rewrite payment processing to be event-sourced. Events published to Kafka, consumed by both monolith and new service. Cleanest long-term architecture, but requires significant rewrite of existing payment logic. Estimated 8–10 additional weeks.

**Option C — CDC (Debezium)**: Capture database changes from the monolith's PostgreSQL WAL and stream them via Debezium to Kafka topics. New service consumes events and maintains its own read store. No changes to existing monolith write path. Eventual consistency with sub-second lag.

**Mei**: "CDC gives us the best risk profile. We don't touch the monolith's write path, so we can't break what's working today. The tradeoff is eventual consistency — the new service's data will lag by 100–500ms."

**David**: "For payment processing, can we tolerate that lag?"

**Mei**: "For reads, absolutely. For writes, the new service talks directly to its own database. During migration, the monolith is still the system of record. We only flip the system of record when we're confident in the new service."

**Ravi**: "What about the refund flow? That reads the original transaction and writes a refund. If the read is stale..."

**Mei**: "Good catch. Refunds will stay in the monolith until the new service is the system of record. We route at the API gateway level — refund endpoints point to the monolith, charge endpoints point to the new service once we cut over."

### 2. Service Mesh Selection

James presented Istio vs. Linkerd evaluation:

| Criteria | Istio | Linkerd |
|----------|-------|---------|
| mTLS | Yes | Yes |
| Traffic splitting | Advanced (percentage, header-based) | Basic (percentage only) |
| Circuit breaking | Yes | Yes |
| Resource overhead | Higher (~200MB/sidecar) | Lower (~50MB/sidecar) |
| Community/Support | Larger ecosystem | Smaller but focused |
| Complexity | Higher | Lower |

**James**: "Istio is heavier, but we need the traffic splitting capabilities for the strangler fig migration. We need to send 10% of traffic to the new service, then 25%, then 50%, then 100%. Istio lets us do that with VirtualService rules. Linkerd's traffic splitting is more limited."

**David**: "The resource overhead concerns me. We're already tight on our EKS node group."

**James**: "I've budgeted for a node group expansion. Two additional m5.xlarge nodes, ~$280/month. Worth it for the migration tooling."

### 3. Database Strategy

Ravi proposed database-per-service with a transition plan:

- **Phase 1**: New service gets a separate schema in the same RDS instance (saves cost, simplifies ops)
- **Phase 2**: When the service is stable (Q3), migrate to a dedicated RDS instance
- **Phase 3**: Right-size the dedicated instance based on actual usage patterns

**Priya**: "From a PCI perspective, the payment-service database must be in the CDE. Same RDS instance is actually simpler for our PCI boundary — one fewer network segment to audit."

**David**: "Let's do Phase 1 now. We can always separate later. Premature optimization of infrastructure is how startups burn cash."

### 4. API Gateway Configuration

James presented the Kong configuration for routing:

- Default route: all traffic to monolith
- Per-endpoint override: `/v1/payments/charge` → new service (with percentage-based rollout)
- Health check: if new service is unhealthy, automatic fallback to monolith
- Latency budget: if new service P99 > 500ms, circuit breaker trips and traffic routes to monolith

**Carlos**: "What about API versioning? If we're going to break the API contract during decomposition..."

**David**: "We're not. The new service implements the exact same API contract. From the merchant's perspective, nothing changes. That's the whole point of strangler fig."

## Decisions

1. **CDC with Debezium selected** as the data synchronization strategy for monolith decomposition. Dual-write rejected (consistency risk too high). Event sourcing rejected (timeline too long).

2. **Istio selected** as the service mesh. Two additional m5.xlarge nodes approved for EKS cluster ($280/month).

3. **Database-per-schema** (Phase 1) — payment-service gets a dedicated schema in the existing RDS instance. Separate RDS instance deferred to Q3.

4. **Refund endpoints stay in monolith** until payment-service is the system of record. Gateway routing handles the split.

5. **Rollout strategy confirmed**: 10% → 25% → 50% → 100% traffic, with at least 1 week at each stage. Automated rollback if error rate exceeds 0.1% or P99 latency exceeds 500ms.

## Action Items

| Action | Owner | Due |
|--------|-------|-----|
| Debezium connector configuration for payments tables | Mei Chen | Feb 21 |
| Kafka topic design for payment events | Ravi Krishnan | Feb 21 |
| Istio installation and configuration in staging EKS | James Liu | Feb 24 |
| EKS node group expansion (2x m5.xlarge) | James Liu | Feb 19 |
| Payment-service database schema design | Carlos Mendez | Feb 21 |
| Kong routing rules for payment endpoints | James Liu | Feb 28 |
| Update PCI network diagram with new service topology | Priya Sharma | Feb 28 |
| Document CDC lag monitoring and alerting thresholds | Mei Chen | Feb 28 |

## Architecture Decision Record

**ADR-2026-003**: CDC via Debezium for Monolith Decomposition Data Sync

- **Context**: Need data synchronization between monolith and extracted services during migration
- **Decision**: Use Debezium CDC streaming PostgreSQL WAL to Kafka
- **Consequences**: Accept eventual consistency (< 500ms lag). Monolith remains system of record during migration. No changes to monolith write path (reduced risk). Must monitor CDC lag and have alerting for replication failures.
- **Status**: Accepted, February 14, 2026
