# Theme: Platform Resilience

> Owner: David Park, VP Engineering | Status: Active | Horizon: Q1–Q3 2026

## Description

Platform Resilience is about earning the right to scale. Meridian Pay processes $2.1B in annual payment volume through a monolithic Java application that was built for $200M. We've hit the ceiling — deployment frequency has dropped from 12/week to 4/week, incident rate has doubled quarter-over-quarter, and our PCI DSS 4.0 audit deadline is March 31, 2026.

This theme is the highest-priority engineering investment in 2026. We're allocating 40% of engineering capacity (11 of 27 engineers) to resilience work in Q1, tapering to 25% by Q3 as the foundation solidifies.

## Key Initiatives

| Initiative | Status | Quarter | Owner |
|-----------|--------|---------|-------|
| [INI-202](/product/strategy/initiatives/INI-202.md) — Monolith Decomposition | ACTIVE | Q1–Q2 | David Park |
| [INI-206](/product/strategy/initiatives/INI-206.md) — PCI DSS 4.0 Compliance | ACTIVE | Q1 | Priya Sharma |
| Observability Platform (INI-207) | PLANNED | Q2 | James Liu |
| Disaster Recovery Overhaul (INI-211) | PLANNED | Q3 | David Park |

## Success Metrics

| Metric | Baseline | Q2 Target | Q4 Target |
|--------|----------|-----------|-----------|
| Platform Uptime | 99.97% | 99.99% | 99.995% |
| Deployment Frequency | 4/week | 15/week | 25/week |
| Mean Time to Recovery (MTTR) | 47 min | 15 min | 8 min |
| P99 API Latency | 820ms | 200ms | 120ms |
| PCI DSS 4.0 Findings | 6 open | 0 | 0 |
| Services Extracted | 0 | 4 | 8 |

## Dependencies

- Monolith decomposition requires CDC (Change Data Capture) infrastructure — Debezium evaluated and approved in Feb architecture review
- PCI DSS 4.0 requires HSM-backed tokenization — vendor contract signed with Thales (Luna Network HSM 7)
- Observability platform requires budget approval ($14K/month for Datadog upgrade)

## Risks

- **Decomposition velocity**: Service extraction is slower than estimated; payment-service took 3 sprints vs. planned 2
- **Data consistency**: CDC introduces eventual consistency — product team needs to accept async settlement reconciliation
- **Talent**: Need 2 senior SREs; recruiting pipeline has 3 candidates in final rounds
