# Sprint 30 Planning — Scale & Multi-Currency

> Date: March 8, 2026 | Type: Sprint Planning | Duration: 60 minutes

## Meeting Details

| Field | Value |
|-------|-------|
| **Date** | March 8, 2026 |
| **Time** | 10:00 AM – 11:00 AM EST |
| **Type** | Sprint Planning |
| **Facilitator** | Fatima Malik, VP Engineering |
| **Participants** | Levi Garner (CTO), Sarah Kim (Staff Engineer), Ravi Krishnan (Senior Engineer), Tyler Brooks (QA Lead) |
| **Recorded by** | Fathom |

## Summary

Sprint 30 planning for the Payments Engineering organization (27 engineers). Three sprint goals established: (1) ship multi-currency support for EUR and GBP with real-time FX conversion, (2) achieve PCI DSS 4.0 compliance with full tokenization and TLS 1.3 enforcement, (3) launch the rebuilt merchant portal to 20% of merchants via feature flag. Total capacity: 27 engineers across 6 team allocations.

Board-level urgency on all three goals. Multi-currency is behind Q1 commitment, merchant portal was promised to top 20 merchants by April, and PCI external audit is end of March with zero margin for delay. Levi made PCI non-negotiable — Ravi has full resource priority.

## Key Topics

### 1. Multi-Currency FX Support (INI-204)
Sarah presented the implementation plan. FX conversion endpoint is scaffolded. Sprint work focuses on integrating a real-time rate provider with 15-second cache and staleness warnings. Primary concern: FX rate provider reliability — two outages last quarter. Decision to build secondary provider failover to eliminate single point of failure.

**Sarah**: "I'm targeting under 50 milliseconds for conversion latency. The primary FX rate provider had two outages last quarter. I want to build a secondary provider failover this sprint."

**Levi**: "Do it. We can't ship multi-currency with a single point of failure on rate data."

### 2. PCI DSS 4.0 Compliance (INI-206)
Ravi outlined the tokenization migration: 1.2 million stored card records migrating to HSM-backed AES-256 encryption. Using dual-read migration pattern — old format and new format side by side — to avoid downtime. TLS 1.3 enforcement already in progress with Jordan handling certificate rotation. External consultant pre-audit scheduled mid-sprint.

**Ravi**: "If we hit our targets, we should pass with zero critical findings. Diana's external consultant review is scheduled for mid-sprint. Any critical findings there and we're in trouble for the March audit window."

**Levi**: "That's non-negotiable. Ravi, whatever you need — take it."

### 3. Merchant Portal Rebuild (INI-205)
Fatima is personally leading this. Launching new React portal to 20% of merchants via feature flag. Critical technical challenge: transaction monitoring view with virtual scrolling for merchants with 50,000+ records. Page load target: under 2 seconds. Legacy portal stays up for 30 days during transition — no merchant gets cut over without fallback.

**Tyler**: "I'll need to load test that specifically. Can we get a staging dataset with 50K records?"

**Fatima**: "Yes, Alex can set that up. Tyler, plan for load testing starting the 18th."

### 4. Capacity Allocation
27 engineers allocated across 6 workstreams: Payments (6) for multi-currency and SDK hooks, Platform Core (6) for monolith decomposition and PCI tokenization, Merchant Experience (5) for portal rebuild, Data & ML (4) for fraud model tuning (targeting <1.5% false positives from current 1.8%), Infrastructure (3) for HSM and TLS, QA & Security (3) for pre-audit and load testing.

### 5. Continuing Commitments
PR review SLA of 24-hour first review continues from Sprint 29. Monolith decomposition scaling from 10% to 25% traffic via feature flag.

## Decisions

1. **Sprint goals confirmed**: Ship multi-currency (EUR/GBP), achieve PCI DSS 4.0 compliance, launch merchant portal to 20%
2. **FX failover approved**: Build secondary FX rate provider — single point of failure is unacceptable for production
3. **PCI is non-negotiable**: Ravi gets priority on any resource conflicts — audit window cannot slip
4. **Portal rollout strategy**: 20% feature flag, legacy stays up 30 days, no forced cutover
5. **Tokenization approach**: Dual-read migration (zero downtime), HSM-backed AES-256
6. **PR review SLA**: 24-hour first review continues
7. **Monolith decomposition**: Scale from 10% to 25% traffic this sprint

## Action Items

| Action | Owner | Due |
|--------|-------|-----|
| Deploy FX conversion endpoint to staging | Sarah Kim | Mar 12 |
| HSM key provisioning set up in staging | Ravi Krishnan | Mar 11 |
| Merchant portal feature flag configuration | Fatima Malik | Mar 13 |
| PCI tokenization dual-read migration test | Jordan Hayes | Mar 15 |
| FX rate cache staleness monitoring dashboard | Alex Torres | Mar 14 |
| Load test merchant portal with 50K records | Tyler Brooks | Mar 18 |
| Secondary FX provider failover integration | Sarah Kim | Mar 20 |
| TLS 1.3 certificate rotation across all services | Ravi Krishnan | Mar 19 |

## Risks & Blockers

| Risk | Status | Owner |
|------|--------|-------|
| FX provider API access — support ticket open | Waiting (expected Monday) | Sarah Kim |
| HSM hardware provisioned, needs configuration | No blocker | Ravi Krishnan |
| External pre-audit could surface critical findings | Scheduled mid-sprint | Ravi Krishnan |

---

## Transcript

**Fatima Malik** [00:00:12]: Alright everyone, let's get Sprint 30 kicked off. We've got three big goals this sprint and two weeks to hit them. Levi, do you want to frame it?

**Levi Garner** [00:00:24]: Yeah. So the board is watching three things right now. First, multi-currency — we promised EUR and GBP support by end of Q1 and we're behind. Second, the merchant portal rebuild — we told our top 20 merchants they'd have the new portal by April. Third, PCI 4.0 — our external audit is scheduled for the end of March and we cannot miss that window.

**Sarah Kim** [00:01:02]: On multi-currency, we've got the FX conversion endpoint scaffolded. The main work this sprint is integrating the real-time rate provider and building the 15-second cache with staleness warnings. I'm targeting under 50 milliseconds for conversion latency.

**Fatima Malik** [00:01:28]: What's the risk there?

**Sarah Kim** [00:01:31]: The primary FX rate provider. They had two outages last quarter. I want to build a secondary provider failover this sprint so we're not single-threaded on that dependency.

**Levi Garner** [00:01:45]: Do it. We can't ship multi-currency with a single point of failure on rate data.

**Fatima Malik** [00:01:52]: Okay, so Goal 1 is ship multi-currency — EUR and GBP, real-time FX, 15-second cache, failover provider. Sarah owns it. Key result: process 1,000 test transactions across all three currencies with under 50ms conversion latency. Sarah, are you good with that?

**Sarah Kim** [00:02:15]: Yes. I'll have the conversion endpoint on staging by the 12th.

**Fatima Malik** [00:02:22]: Perfect. Ravi, PCI status?

**Ravi Krishnan** [00:02:26]: Tokenization is the big one. We've got 1.2 million stored card records that need to migrate to HSM-backed AES-256 encryption. I'm doing a dual-read migration so we don't have downtime — old format and new format side by side until we verify everything converted cleanly.

**Levi Garner** [00:02:48]: What's the timeline on that?

**Ravi Krishnan** [00:02:51]: I can have the migration script running by the 11th. The HSM key provisioning needs to happen first — I'll set that up in staging by Monday. TLS 1.3 enforcement is already in progress, Jordan's handling the certificate rotation across all services.

**Fatima Malik** [00:03:12]: And the pre-audit?

**Ravi Krishnan** [00:03:15]: If we hit our targets, we should pass with zero critical findings. Diana's external consultant review is scheduled for mid-sprint. Any critical findings there and we're in trouble for the March audit window.

**Levi Garner** [00:03:32]: That's non-negotiable. Ravi, whatever you need — take it.

**Fatima Malik** [00:03:38]: Goal 2 is PCI DSS 4.0 compliance. Ravi owns it. Complete tokenization, enforce TLS 1.3, pass the internal pre-audit with zero critical findings. 100% of stored card data tokenized.

**Fatima Malik** [00:03:55]: Now the portal. I'm taking this one personally. We're launching the new React merchant portal to 20% of merchants via feature flag. Page load has to be under 2 seconds. The big technical challenge is the transaction monitoring view — some of our merchants have 50,000-plus records and we need virtual scrolling to handle that.

**Tyler Brooks** [00:04:18]: I'll need to load test that specifically. Can we get a staging dataset with 50K records?

**Fatima Malik** [00:04:25]: Yes, Alex can set that up. Tyler, plan for load testing starting the 18th. The legacy portal stays up for 30 days during the transition — nobody gets cut over without a fallback.

**Levi Garner** [00:04:42]: What's our capacity allocation looking like?

**Fatima Malik** [00:04:46]: Six engineers on payments for multi-currency and SDK hooks. Six on platform core for the monolith decomposition, service mesh, and PCI tokenization. Five on merchant experience for the portal rebuild and feature flag rollout. Four on data and ML — the fraud model tuning continues, we're at 1.8% false positives and targeting under 1.5%. Three on infrastructure for HSM provisioning, TLS enforcement, and staging environments. Three on QA and security for the PCI pre-audit, load testing, and integration test coverage. 27 engineers total.

**Levi Garner** [00:05:28]: And we're continuing the PR review SLA from last sprint?

**Fatima Malik** [00:05:32]: Yes, 24-hour first review on all PRs. That's enforced. Also continuing the monolith decomposition — payments service is at 10% traffic via feature flag, we're scaling to 25% this sprint.

**Levi Garner** [00:05:48]: Good. Let's document the action items.

**Fatima Malik** [00:05:52]: Action items. Sarah — deploy FX conversion endpoint to staging by March 12th. Ravi — HSM key provisioning set up in staging by March 11th. I'll handle the merchant portal feature flag configuration by March 13th. Jordan — run the PCI tokenization dual-read migration test by March 15th. Alex — FX rate cache staleness monitoring dashboard by March 14th. Tyler — load test merchant portal with 50K records by March 18th. Sarah again — secondary FX provider failover integration by March 20th. Ravi — TLS 1.3 certificate rotation across all services by March 19th.

**Levi Garner** [00:06:38]: That's a full sprint. Any blockers right now?

**Sarah Kim** [00:06:42]: Just the FX provider API access — I've got a support ticket open. Should have it by Monday.

**Ravi Krishnan** [00:06:50]: No blockers. The HSM hardware is provisioned, just needs configuration.

**Fatima Malik** [00:06:56]: Clean start. Let's execute. Sprint 30 — Scale & Multi-Currency. We ship in two weeks.

**Levi Garner** [00:07:02]: Let's go.

---

*Synced by InteliG Signal | Transcribed by Fathom | March 8, 2026*
