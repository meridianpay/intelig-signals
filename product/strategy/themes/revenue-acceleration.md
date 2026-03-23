# Theme: Revenue Acceleration

> Owner: Sarah Kim, VP Product | Status: Active | Horizon: Q1–Q4 2026

## Description

Revenue Acceleration is the growth engine of Meridian Pay's 2026 strategy. We're targeting a jump from $18.2M to $32M ARR by unlocking three capabilities our merchants have been requesting for over a year: multi-currency processing, enterprise-grade billing features, and geographic expansion into the UK and EU markets.

Today, we lose 30–40% of enterprise prospects at the multi-currency question. Our largest churned account (DataWeave, $287K ARR) cited single-currency as the primary reason for switching to Adyen. That stops in Q2.

## Key Initiatives

| Initiative | Status | Quarter | Owner |
|-----------|--------|---------|-------|
| [INI-204](/product/strategy/initiatives/INI-204.md) — Ship Multi-Currency | ACTIVE | Q1–Q2 | Ravi Krishnan |
| [INI-205](/product/strategy/initiatives/INI-205.md) — Rebuild Merchant Portal | ACTIVE | Q1–Q2 | Fatima Malik |
| Enterprise Billing (INI-208) | PLANNED | Q3 | Sarah Kim |
| UK Market Launch (INI-210) | PLANNED | Q3 | Carlos Mendez |

## Success Metrics

| Metric | Baseline | Q2 Target | Q4 Target |
|--------|----------|-----------|-----------|
| ARR | $18.2M | $23M | $32M |
| Multi-Currency TPV | $0 | $120M | $480M |
| Enterprise Accounts (>$100K ARR) | 12 | 18 | 30 |
| Net Revenue Retention | 108% | 115% | 125% |
| Pipeline from Self-Serve | 22% | 30% | 40% |

## Dependencies

- Multi-currency requires monolith decomposition (INI-202) to extract the payment processing service first
- Merchant portal rebuild is blocked on design system v2 (shipping mid-March)
- UK expansion requires FCA registration (legal team lead, timeline: 8–12 weeks)

## Risks

- **Currency volatility**: Need robust FX hedging strategy before launch — treasury team engaged
- **Pricing complexity**: Multi-currency adds billing edge cases; finance team needs 4 weeks for model updates
- **Competitive pressure**: Stripe launched improved multi-currency in Jan 2026; our window is narrowing
