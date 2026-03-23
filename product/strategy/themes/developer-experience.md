# Theme: Developer Experience

> Owner: Fatima Malik, Engineering Manager (Platform) | Status: Active | Horizon: Q1–Q4 2026

## Description

Developer Experience is our moat. In a market where payment APIs are converging on feature parity, the team that makes integration painless wins. Our current SDK (v2.4) was built in 2023, predates our webhook system, and requires 14 days median integration time. Competitors are at 3–5 days.

We're rebuilding from the ground up: SDK v3 with TypeScript-first design, a merchant portal that doesn't require a support ticket to navigate, and API improvements that eliminate the top 20 integration friction points identified in our developer survey (n=342, Dec 2025).

The business case is clear: every day of integration time costs us 8% close rate on self-serve signups. Cutting from 14 days to 5 days is worth an estimated $2.1M in recovered pipeline annually.

## Key Initiatives

| Initiative | Status | Quarter | Owner |
|-----------|--------|---------|-------|
| [INI-205](/product/strategy/initiatives/INI-205.md) — Rebuild Merchant Portal | ACTIVE | Q1–Q2 | Fatima Malik |
| SDK v3 (INI-209) | PLANNED | Q2–Q3 | Alex Tran |
| API Versioning & Deprecation (INI-212) | PLANNED | Q2 | Ravi Krishnan |
| Interactive API Explorer (INI-213) | PLANNED | Q3 | Fatima Malik |

## Success Metrics

| Metric | Baseline | Q2 Target | Q4 Target |
|--------|----------|-----------|-----------|
| Developer NPS | 38 | 48 | 55 |
| Median Integration Time | 14 days | 7 days | 5 days |
| Support Tickets per Merchant (monthly) | 4.7 | 2.5 | 1.5 |
| Self-Serve Activation Rate | 34% | 50% | 65% |
| API Documentation Coverage | 72% | 90% | 98% |
| SDK Weekly Downloads | 1,200 | 3,000 | 6,000 |

## Dependencies

- Merchant portal depends on design system v2 (ETA: March 14, 2026)
- SDK v3 requires stable API contracts — blocked until monolith decomposition extracts payment-service
- API explorer requires OpenAPI spec completion (currently 72% coverage, targeting 90% by end of Q1)

## Risks

- **Migration burden**: SDK v2 → v3 migration path must be seamless; 1,847 active merchants can't break
- **Portal scope creep**: Product has 47 feature requests queued; need ruthless prioritization on top 10
- **Competitive timing**: Adyen launched a new developer portal in Jan 2026; market expectations have shifted
