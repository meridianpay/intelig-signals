# PCI Tokenization Deep Dive — HSM Architecture

> Date: March 4, 2026 | Type: Working Session | Duration: 90 minutes

## Meeting Details

| Field | Value |
|-------|-------|
| **Date** | March 4, 2026 |
| **Time** | 10:00 AM – 11:30 AM EST |
| **Type** | Technical Working Session |
| **Facilitator** | Priya Sharma, Security Lead |
| **Participants** | Priya Sharma (Security Lead), Ravi Krishnan (Sr Backend), James Liu (DevOps/SRE), Mei Chen (Staff Engineer), David Park (VP Eng, first 30 min) |

## Summary

Deep-dive working session on the HSM-backed tokenization architecture required for PCI DSS 4.0 compliance (gaps #5 and #6 from INI-206). The team worked through the Thales Luna HSM integration design, key management lifecycle, and automated rotation mechanism. Resolved a critical firmware compatibility issue discovered during the Sprint 29 integration spike. Produced a detailed implementation plan for Sprint 30 with daily milestones.

## Key Topics

### 1. HSM Integration Spike Results

Priya presented findings from the Sprint 29 integration spike (Feb 24 – Mar 5):

**What worked:**
- PKCS#11 interface is well-documented and stable
- Key generation benchmarks: 1,500 ops/sec (4.4x our peak requirement)
- Tokenization latency: 8ms average (within our latency budget)
- Network connectivity from EKS pods to HSM via dedicated VPC endpoint is stable

**What didn't work:**
- Firmware version mismatch: HSM shipped with firmware 7.7.0, but the PKCS#11 Java wrapper (LunaProvider 10.4) requires firmware 7.8.1
- Firmware update took 2 days (required Thales support involvement due to FIPS compliance)
- The Java wrapper's session management has a connection pool leak under high concurrency — discovered during load testing

**Priya**: "The firmware issue cost us 2 days and was frustrating, but it's resolved. The connection pool leak is more concerning. Under sustained load (> 200 concurrent tokenization requests), the PKCS#11 sessions aren't being returned to the pool. We see a gradual increase in active sessions until we hit the HSM's 500-session limit."

**Mei**: "Have you filed a bug with Thales?"

**Priya**: "Yes, but we can't wait for their fix. I've written a wrapper that implements explicit session cleanup with a try-with-resources pattern. It's not elegant, but it works — load test ran for 6 hours with stable session count."

### 2. Tokenization Architecture

The team whiteboarded the tokenization flow:

```
Merchant API Request (card data)
    → API Gateway (Kong)
    → Payment Service
    → Tokenization Service
        → PKCS#11 Interface
        → Thales Luna HSM (tokenize)
        → Return token + masked PAN
    → Store token in payments DB
    → Return token to merchant
```

**Key design decisions:**

**Token format**: `tok_` prefix + 24-character alphanumeric string (e.g., `tok_a1b2c3d4e5f6g7h8i9j0k1l2`). Format-preserving encryption (FPE) considered but rejected — tokens don't need to look like card numbers.

**Token vault**: Separate PostgreSQL table (`token_vault`) in the CDE schema. Columns: `token`, `encrypted_pan`, `key_version`, `created_at`, `last_used_at`. Encrypted PAN is HSM-encrypted, meaning even database access doesn't expose card data.

**Detokenization**: Only the settlement service needs to detokenize (for processor submission). Detokenization requires HSM access + service-level mTLS certificate. No other service can detokenize.

**Mei**: "What about the existing 4.2M tokens in the old format?"

**Priya**: "Migration plan: batch re-tokenize using HSM during off-peak hours (2 AM – 6 AM EST). At 1,500 ops/sec, we can process 4.2M tokens in under 47 minutes. We'll run old and new formats in parallel for 30 days, then decommission the software-based tokens."

### 3. Key Management Lifecycle

Ravi presented the key management design:

**Key hierarchy:**
- **Master Key (MK)**: Generated on HSM, never exported. Split knowledge: 3 of 5 key custodians required for recovery. Key custodians: David Park, Priya Sharma, Marcus Chen, Mei Chen, Ravi Krishnan.
- **Key Encryption Key (KEK)**: Derived from MK, used to wrap Data Encryption Keys. Rotated annually.
- **Data Encryption Key (DEK)**: Used for actual tokenization operations. Rotated every 90 days. Multiple DEKs active simultaneously (one per quarter).

**Automated rotation flow:**
1. CronJob triggers 7 days before DEK expiration
2. New DEK generated on HSM
3. New DEK version registered in key metadata table
4. All new tokenization uses new DEK
5. Existing tokens remain valid (old DEK retained for detokenization)
6. After 12 months, old DEKs archived (tokens re-encrypted with current DEK in background)

**James**: "What's the blast radius if a DEK rotation fails?"

**Ravi**: "Zero. The old DEK continues to work. The rotation CronJob has a 7-day window — if it fails, it retries daily. We get alerted on first failure. Even if rotation is delayed, there's no service impact."

**David**: "This is the right design. The QSA will love the split knowledge and automated rotation. Make sure we document the key ceremony procedures — they'll want to see those."

### 4. Implementation Plan for Sprint 30

Daily milestones for the 2-week sprint:

| Day | Date | Milestone | Owner |
|-----|------|-----------|-------|
| 1 | Mar 7 | Tokenization service scaffold + PKCS#11 wrapper with session fix | Ravi |
| 2 | Mar 8 | Token vault schema, migration scripts | Mei |
| 3-4 | Mar 10-11 | Tokenize/detokenize API implementation | Ravi |
| 5 | Mar 12 | Integration with payment service (new tokens) | Ravi + Carlos |
| 6 | Mar 13 | Key rotation CronJob implementation | Mei |
| 7 | Mar 14 | End-to-end testing in staging | Priya |
| 8 | Mar 15 | Load testing (sustained 400 ops/sec for 4 hours) | James |
| 9 | Mar 17 | Token migration script (4.2M existing tokens) | Mei |
| 10 | Mar 18 | Production deployment + migration run | All |
| 11-12 | Mar 19-20 | Monitoring, validation, pre-audit documentation | Priya |

**Priya**: "This is aggressive but achievable. The spike work de-risked the hard parts — PKCS#11 integration and session management. Sprint 30 is assembly and testing."

### 5. Audit Preparation

Priya outlined what Coalfire will evaluate during the March 24–28 audit:

- HSM physical security documentation (Thales provides)
- Key management procedures (split knowledge, rotation)
- Tokenization implementation review (code + architecture)
- Penetration test of tokenization service (NCC Group, scheduled Mar 20)
- Access controls audit (who can access HSM, CDE, token vault)
- Key ceremony observation (Coalfire will watch a simulated DEK rotation)

**Priya**: "I need everyone in this room available March 24–28 for QSA questions. Block your calendars. Coalfire's lead assessor is Rachel Wong — she's thorough but fair. If we have our documentation in order, we'll be fine."

## Decisions

1. **Token format**: `tok_` prefix + 24-char alphanumeric. FPE rejected.
2. **PKCS#11 session management**: Custom wrapper with explicit cleanup (Priya's try-with-resources pattern). Not waiting for Thales fix.
3. **Token migration**: Batch re-tokenization during off-peak hours. 30-day parallel operation of old and new formats.
4. **Key rotation**: Automated 90-day DEK rotation via CronJob. 7-day pre-expiration window with daily retry.
5. **Key custodians**: David Park, Priya Sharma, Marcus Chen, Mei Chen, Ravi Krishnan (3 of 5 required for MK recovery).

## Action Items

| Action | Owner | Due |
|--------|-------|-----|
| Tokenization service repository setup | Ravi Krishnan | Mar 7 |
| Token vault schema PR | Mei Chen | Mar 8 |
| Key ceremony procedure document | Priya Sharma | Mar 12 |
| Schedule NCC Group penetration test for tokenization service | Priya Sharma | Mar 7 |
| Block calendars for March 24–28 audit | All participants | Mar 5 |
| Order YubiKeys for key custodians (backup MFA) | James Liu | Mar 7 |
| Document HSM firmware upgrade procedure (for audit evidence) | Priya Sharma | Mar 10 |
| Load test environment provisioning (matching prod HSM throughput) | James Liu | Mar 13 |
