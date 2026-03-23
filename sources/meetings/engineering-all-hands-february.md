# Engineering All-Hands — February 2026

> Date: February 28, 2026 | Type: All-Hands | Duration: 60 minutes

## Meeting Details

| Field | Value |
|-------|-------|
| **Date** | February 28, 2026 |
| **Time** | 4:00 PM – 5:00 PM EST |
| **Type** | Monthly Engineering All-Hands |
| **Facilitator** | David Park, VP Engineering |
| **Participants** | Full engineering organization (27 engineers), Sarah Kim (VP Product), Marcus Chen (CEO, first 15 min) |

## Summary

Monthly engineering all-hands covering Q1 progress, team celebrations, and forward-looking priorities. Marcus Chen opened with business context: ARR at $18.8M (up from $18.2M at start of quarter), 14 new enterprise prospects in pipeline. David reviewed engineering metrics and initiative status. Celebrated PCI audit milestone (4 of 6 gaps closed), fraud detection model performance, and merchant onboarding automation launch. Priya presented a deep dive on the HSM tokenization roadmap.

## Key Topics

### 1. CEO Update (Marcus Chen)

**Marcus**: "I want to start with context on why the work you're doing matters to the business right now. We're at $18.8M ARR, up from $18.2M at the start of the quarter. We have 14 enterprise prospects in late-stage pipeline, and I need you to know — 9 of them have asked about multi-currency. When we ship that, we're not just building a feature. We're unlocking a segment of the market that's worth more than our entire current ARR."

**Marcus**: "The board meeting is March 20. I'm going to tell them that we have a credible path to $32M by year-end. The credibility of that number rests on what's happening in this room. Fraud detection, monolith decomposition, multi-currency, PCI — these aren't just engineering projects. They're the business plan."

### 2. Engineering Metrics Review (David Park)

| Metric | January | February | Trend |
|--------|---------|----------|-------|
| Deployment frequency | 4.1/week | 5.8/week | Improving |
| Incident count | 3 (1 Sev-2) | 2 (0 Sev-2) | Improving |
| MTTR | 52 min | 38 min | Improving |
| Sprint velocity | 82 pts | 89 pts | Improving |
| PR cycle time | 3.2 days | 2.1 days | Improving |
| Test coverage | 71% | 74% | Improving |

**David**: "Every metric is moving in the right direction. I want to call out PR cycle time specifically — going from 3.2 days to 2.1 days is a big deal. That's the result of the PR size guidelines we introduced in January. Smaller PRs, faster reviews. Keep it up."

### 3. Initiative Status Updates

**INI-201 — Fraud Detection** (Ravi): "Shadow mode has been live for 3 days. We've scored 890,000 transactions. The model is catching things the rules engine misses — 312 transactions flagged by ML that rules passed. We're validating each one manually. So far, 287 of 312 are confirmed fraudulent. That's a 92% precision rate in the wild."

**INI-202 — Monolith Decomposition** (David): "Payment-service is in staging. CDC pipeline is running with 180ms P99 lag. We're passing 247 of 251 integration tests — the 4 failures are refund edge cases that Carlos is fixing this week. On track for 10% production traffic in Sprint 30."

**INI-203 — Merchant Onboarding** (Carlos): "Shipped to production today. First 12 merchants have gone through the automated flow. Average onboarding time: 1.2 business days. Lisa from operations literally sent me a message saying 'you've given me my life back.' That felt good."

**INI-206 — PCI DSS 4.0** (Priya): "Four of six gaps closed. The two remaining are both tokenization-related and both depend on the HSM integration. I'll do a deeper dive on that in a moment. Audit is March 24–28. We're tight but on track."

### 4. Celebrations

- **PCI audit progress**: 4 gaps closed ahead of schedule. Special shout-out to James Liu for the automated log review implementation — what was estimated as a 2-week project was done in 4 days.
- **Fraud detection model**: Ravi's team trained a model that's outperforming a rules engine that took 2 years to build, in just 6 weeks. The feature engineering work (147 features) was exceptional.
- **Merchant onboarding**: Carlos, Aisha Williams, and Jun Park shipped INI-203 three days ahead of schedule. First initiative completed this quarter.

### 5. HSM Tokenization Deep Dive (Priya Sharma)

Priya presented the HSM-backed tokenization architecture:

- **Current state**: Software-based AES-256 encryption, keys stored in AWS KMS
- **Target state**: Thales Luna Network HSM 7, keys never leave the hardware, FIPS 140-2 Level 3 certified
- **Architecture**: Application → PKCS#11 interface → HSM partition → tokenize/detokenize
- **Performance**: HSM can handle 1,500 crypto operations/second; our peak is 340/second (4.4x headroom)
- **Key rotation**: Automated 90-day rotation via HSM API, split knowledge for master key (3 of 5 key custodians required)

**David**: "What's the latency impact?"

**Priya**: "We benchmarked in staging. Software tokenization: 2ms average. HSM tokenization: 8ms average. The 6ms delta is acceptable — it's within our latency budget and PCI auditors will love it."

**Ravi**: "Can we use the HSM for the fraud model's feature encryption too?"

**Priya**: "Yes, but let's not scope-creep. HSM for card tokenization first. We can extend to other use cases in Q2."

### 6. Q&A Highlights

**Engineer (Aisha Williams)**: "Are we hiring for the SRE roles? The on-call rotation is getting thin with the new services."

**David**: "Yes, two senior SRE roles are in final rounds. I expect offers out by mid-March. In the meantime, James is building out the Datadog dashboards so we have better observability before we add services."

**Engineer (Jun Park)**: "What's the plan for the monolith after decomposition? Are we sunsetting it?"

**David**: "Eventually, yes. But 'eventually' means Q3 at the earliest. The monolith will shrink as we extract services, but it'll still run settlement, reporting, and notification until we extract those. Think of it as a graceful retirement, not a sudden death."

**Engineer (Nadia Volkov)**: "Can we get budget for the full Datadog APM tier? The current plan doesn't include distributed tracing, and we're going to need that with microservices."

**David**: "Good timing — I'm putting together the Q2 tooling budget next week. I'll include Datadog APM upgrade. James, send me the pricing comparison by Monday."

## Decisions

1. **Datadog APM upgrade** to be included in Q2 tooling budget request (David to submit by March 7)
2. **HSM tokenization** confirmed as the approach for PCI gap #5 and #6 — no fallback plan, this must ship
3. **Fraud model cutover** approved for Sprint 30, contingent on shadow mode metrics remaining stable
4. **No scope expansion on HSM** — card tokenization only for Q1, other use cases deferred to Q2

## Action Items

| Action | Owner | Due |
|--------|-------|-----|
| Submit Q2 tooling budget (incl. Datadog APM) | David Park | Mar 7 |
| Datadog APM pricing comparison | James Liu | Mar 3 |
| SRE candidate final interviews | David Park | Mar 10 |
| Fraud model cutover plan document | Ravi Krishnan | Mar 7 |
| HSM integration sprint plan for Sprint 30 | Priya Sharma | Mar 7 |
| Share merchant onboarding metrics dashboard with sales | Carlos Mendez | Mar 3 |
