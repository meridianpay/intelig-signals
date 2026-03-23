# Sprint 29 Planning — Foundation & Fraud

> Date: February 21, 2026 | Type: Sprint Planning | Duration: 90 minutes

## Meeting Details

| Field | Value |
|-------|-------|
| **Date** | February 21, 2026 |
| **Time** | 9:00 AM – 10:30 AM EST |
| **Type** | Sprint Planning |
| **Facilitator** | Fatima Malik, Engineering Manager |
| **Participants** | David Park (VP Eng), Sarah Kim (VP Product), Ravi Krishnan (Sr ML Eng), Carlos Mendez (Sr Backend), Priya Sharma (Security Lead), Alex Tran (Frontend Lead), James Liu (DevOps), Fatima Malik (EM), + 19 engineers |

## Summary

Sprint 29 planning for the Payments Engineering organization (27 engineers). Three sprint goals established: (1) deploy fraud detection ML model in shadow mode, (2) extract payment-service to staging with CDC pipeline, (3) ship merchant onboarding automation to production. Total capacity: 94 points across 6 initiative workstreams plus tech debt buffer.

Major discussion around capacity allocation between monolith decomposition (INI-202) and multi-currency (INI-204) — both are P0 but compete for backend engineering time. Resolved by having the multi-currency team focus on EUR sandbox testing (less backend-heavy) while decomposition team pushes payment-service to staging.

## Key Topics

### 1. Fraud Detection Shadow Mode (INI-201)
Ravi presented the model training results: XGBoost ensemble trained on 18.4M transactions, 147 features, AUC-ROC of 0.94 on holdout set. Plan is to deploy as a gRPC sidecar to the payment processing pipeline. Every transaction gets scored; results logged but no blocking. Target: demonstrate 40% improvement over rules engine on live traffic by end of sprint.

**Ravi**: "The model looks strong on historical data, but live traffic always surprises. Shadow mode for two weeks gives us confidence without risk. If we see 40% improvement, we cutover in Sprint 30."

**David**: "What's the rollback plan if the sidecar impacts latency?"

**Ravi**: "Circuit breaker with 100ms timeout. If the model doesn't respond, we fall through to the rules engine. Zero impact on transaction processing."

### 2. Payment-Service Extraction (INI-202)
David walked through the extraction plan. Payment-service is the first bounded context being pulled from the monolith. CDC pipeline via Debezium is the data sync mechanism. Target: payment-service running in staging with all 251 integration tests passing by Feb 28.

**David**: "We're doing strangler fig. The monolith still processes everything in production. The new service in staging needs to handle the same traffic pattern without data inconsistencies. Debezium gives us the event stream."

**James**: "CDC lag target?"

**David**: "Under 500ms P99. If we can hit that, we're confident in production cutover with 10% traffic in Sprint 30."

### 3. Merchant Onboarding Automation (INI-203)
Carlos presented the final integration: Middesk KYB + risk scoring model + PandaDoc contracts + self-serve provisioning. All components tested individually. Sprint 29 goal is to wire them end-to-end and ship to production.

**Sarah**: "What's our confidence level on the auto-approval rate?"

**Carlos**: "Based on the last 200 merchant applications run through the model retroactively, we'd auto-approve 67%. Our target was 60%. The risk scoring model is conservative by design — we'd rather manual-review a good merchant than auto-approve a risky one."

### 4. Capacity Allocation Discussion
Tension between INI-202 (monolith decomposition, 6 engineers) and INI-204 (multi-currency, 5 engineers). Both need Ravi's attention. Resolution: Ravi focuses on fraud (INI-201) and EUR sandbox for multi-currency. Backend capacity for multi-currency shifts to settlement engine design (less hands-on-keyboard, more architecture).

**Fatima**: "We're at 94 points capacity. If we overcommit, the decomposition timeline slips, and that blocks multi-currency production launch in Q2. I'd rather we under-promise on multi-currency this sprint."

**Sarah**: "Agreed. EUR in sandbox by end of sprint is enough. The real pressure is Q2 production launch."

### 5. PCI DSS 4.0 Status
Priya reported 4 of 6 gaps closed. Two remaining: HSM-backed tokenization (gap #5) and key rotation automation (gap #6). Audit date confirmed: March 24–28. Sprint 29 will focus on incident response plan update and starting HSM integration.

**Priya**: "The Thales HSM is racked and has network connectivity. This sprint we're doing the integration spike — understand the SDK, test key generation, build the tokenization flow. Full implementation in Sprint 30."

**David**: "Priya, flag me immediately if HSM work is going slower than expected. We cannot miss the audit date."

## Decisions

1. **Sprint goals confirmed**: Shadow mode fraud detection, payment-service to staging, merchant onboarding to production
2. **Capacity split**: INI-202 gets 6 engineers (largest allocation), INI-204 gets 5 but focused on design work, INI-201 gets 4
3. **Multi-currency scope for Sprint 29**: EUR sandbox only, no production work. Settlement engine v2 design document by end of sprint.
4. **Tech debt allocation**: 2 engineers reserved for unplanned work and tech debt (James Liu + rotating)
5. **Settlement reports dependency**: Acknowledged design system v2 dependency. If design system ships by Mar 7, settlement reports stay in Sprint 29. If not, push to Sprint 30.

## Action Items

| Action | Owner | Due |
|--------|-------|-----|
| Deploy fraud model sidecar to staging | Ravi Krishnan | Feb 24 |
| Activate shadow mode on production traffic | Ravi Krishnan | Feb 26 |
| Payment-service staging deployment | David Park | Feb 28 |
| CDC pipeline validation (lag < 500ms P99) | James Liu | Mar 1 |
| Merchant onboarding end-to-end testing | Carlos Mendez | Feb 26 |
| Merchant onboarding production deploy | Carlos Mendez | Mar 1 |
| HSM integration spike document | Priya Sharma | Mar 5 |
| EUR sandbox end-to-end transaction test | Ravi Krishnan | Mar 3 |
| Settlement engine v2 design document | Alex Tran | Mar 7 |
| Check design system v2 ETA with design team | Fatima Malik | Feb 24 |
