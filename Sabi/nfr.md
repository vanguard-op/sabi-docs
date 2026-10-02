# Sabi: non-functional requirements

## Overview

These requirements cover performance, availability, security, privacy, and model cost for a mobile and USSD/SMS/WhatsApp product serving Nigerian users on low-end devices and intermittent networks. Quality targets are stated as measurable budgets. The model behavior and quality bar are owned by [model-quality](model-quality.md) and referenced here by `MQ-###` rather than restated. Platform-specific choices for meeting these targets are left to a later architecture document.

## Performance and scalability

Targets are stated per operation class. Each row includes the load assumption under which the target holds.

| ID | Operation class | Metric | Target (p50 / p95 / p99) | Load assumption | Verification |
|---|---|---|---|---|---|
| NFR-001 | App cold start to interactive, entry-level Android | Time to interactive | 2.5s / 4.0s / 6.0s | 4G, device with 1 to 2 GB RAM | Device-lab measurement in CI |
| NFR-002 | Intent resolution (FEAT-003) | Server response, inference included | 800ms / 2.0s / 4.0s | 50 rps sustained | Load test with recorded intents |
| NFR-003 | Service and biller discovery (FEAT-004) | Server response | 300ms / 700ms / 1.5s | 50 rps, 1M catalogue records | Load test |
| NFR-004 | Payment execution to rail acknowledgement (FEAT-005) | End-to-end | 3s / 8s / 20s | 20 rps sustained | Synthetic payment probe |
| NFR-005 | Official receipt retrieval (FEAT-008, FTeR) | End-to-end | 2s / 5s / 15s | 20 rps | Synthetic receipt probe |
| NFR-006 | USSD step response (FEAT-010) | Per-step response | 1.0s / 1.5s / 2.5s | 100 rps, session under 120s | Channel monitor |
| NFR-007 | First composed interface surface (FEAT-011) | Time to first rendered surface | 1.5s / 3.0s / 5.0s | 50 rps | Client performance budget |

**Throughput and capacity:**
- **Peak concurrency:** 5,000 concurrent users at launch without degradation.
- **Data volume:** 2 million user records, 1 million catalogue records at launch; 10x growth over 12 months.
- **Scaling trigger:** scale the inference and payment paths when p95 latency exceeds target at 60% of peak concurrency for 15 minutes.

## Availability and reliability

| ID | Availability target | Monthly error budget | Measurement |
|---|---|---|---|
| NFR-010 | 99.5% overall at launch, growth path to 99.9% | 3h 39m | Uptime monitor plus status page |
| NFR-011 | 99.9% payment authorization path | 43m 12s | Synthetic authorization probe |
| NFR-012 | 99% USSD channel | 7h 18m | Channel monitor |

**Recovery objectives:**
- **RTO (Recovery Time Objective):** 1 hour for the payment path, 4 hours for the assistant and discovery.
- **RPO (Recovery Point Objective):** 5 minutes for payment and receipt records, 1 hour for catalogue data.

**Degradation, what happens when a dependency fails:**

| Failure | Expected behavior | User-visible effect |
|---|---|---|
| Payment rail unavailable | Stop new executions, keep discovery and receipt views working | Banner stating payments are paused; no debits attempted |
| Model provider unavailable | Fall back to keyword and search resolution (FEAT-003 fallback) | Assistant answers from search; confirms before any payment |
| Identity provider unavailable | Queue verification, keep discovery open | Verification marked pending; gated payment held, not failed |
| Interface rendering failure | Render the fallback form for the same task (FEAT-011 fallback) | Task remains completable in the fallback form |
| SMS provider unavailable | Queue messages and retry | Tokens and references delayed, never lost |
| USSD channel unavailable | Hold pending sessions for resumption | User is told to try again; pending payment preserved |

Planned maintenance occurs in a published window with at least 48 hours notice, and outside peak evening hours.

## Security

| ID | Control | Target or standard | Notes |
|---|---|---|---|
| NFR-020 | Authentication | OWASP ASVS Level 2; passkey primary; MFA required for admin roles | Recovery code is single-use and regenerable |
| NFR-021 | Credential storage | Recovery secrets stored with Argon2id, never reversible | Passkey private keys never leave the device |
| NFR-022 | Transport encryption | TLS 1.3, 1.2 minimum; HSTS | No mixed content |
| NFR-023 | Data at rest | AES-256 for all persistent stores and backups | Includes receipt images and references |
| NFR-024 | Secrets management | Managed secret store; no secrets in code or logs; rotation policy stated | Applied to rail and identity credentials |
| NFR-025 | Input handling | Parameterized queries; context-aware output encoding; injection filtering on retrieved and user content | OWASP Top 10 mitigated; covers prompt injection at the input boundary |
| NFR-026 | Session management | HttpOnly, Secure, SameSite cookies; absolute and idle timeout | Channel sessions time-bound separately |
| NFR-027 | Payment authorization integrity | Signed mandate bound to amount, payee, and user, verified before execution | Prevents a changed amount or payee during authorisation |
| NFR-028 | Audit logging | Authentication, authorization, and privileged actions logged tamper-evidently | Retained per the data lifecycle below |
| NFR-029 | Supply chain | No known critical CVEs; SBOM maintained; versions pinned | Applies to client and backend |
| NFR-030 | Accessibility | WCAG 2.2 Level AA for the mobile client, as the target level owned by [branding](branding.md) | Visual implementation (contrast, focus, touch targets) is set in [branding](branding.md); reduced-motion is honoured |

## Model quality and cost

The model behavior and quality bar are owned by [model-quality](model-quality.md) and referenced here by `MQ-###`. This section states only the cost ceilings, and the latency targets above are the operation budgets those capabilities run under.

| ID | Capability | Feature | Cost ceiling per request | Linked bar |
|---|---|---|---|---|
| NFR-050 | Intent resolution | FEAT-003 | 0.02 USD | MQ-001 |
| NFR-051 | Service and biller discovery | FEAT-004 | 0.01 USD | MQ-002 |
| NFR-052 | Interface composition | FEAT-011 | 0.02 USD | MQ-003 |

Cost is measured per completed user task, not per model call, so retries and clarification turns count against the ceiling.

## Privacy and compliance

| ID | Regulation or standard | Applies because | Obligation | Evidence |
|---|---|---|---|---|
| NFR-040 | Nigeria Data Protection Act 2023 | Nigerian users' personal data is processed | Lawful basis, consent capture and withdrawal, data subject access and deletion, breach handling | Privacy notice, data subject request workflow, NDPC compliance return |
| NFR-041 | CBN AML/CFT and KYC rules | Regulated payments are processed | Customer due diligence, transaction monitoring, record keeping | KYC records, monitoring reports |
| NFR-042 | PCI-DSS | Card rails are used for some payments | Scoped assessment; card data never stored | Processor attestation |
| NFR-043 | NIMC identity rules | NIN is used for verification | One person, one identity; restrict identity data use to verification | Verification logs, access controls |
| NFR-044 | Consumer protection and dispute rules | Payment disputes arise | Documented complaint and refund timelines | Case records, refund reports |

Sources: the Nigeria Data Protection Act 2023 sets penalties up to N10,000,000 or 2% of annual gross revenue ([NDPA 2023](https://ncc.gov.ng/sites/default/files/2024-11/Nigeria_Data_Protection_Act_2023.pdf)); the NIMC Act 2026 makes the NIN the foundational credential under a one person, one identity policy ([The Eastern Updates](https://theeasternupdates.com/2026/07/13/nimc-given-2026-deadline-to-enrol-all-nigerians-dg-says/)).

**Data lifecycle:**

| Data class | Retention | Residency | Deletion |
|---|---|---|---|
| Account and profile | Life of account plus 30 days | Nigeria | Hard-delete within 30 days of a valid request |
| Payment and receipt records | 7 years (statutory) | Nigeria | Retained by law, access-restricted, not sold |
| Identity verification data | As required by KYC rules | Nigeria | Deleted when the retention obligation ends |
| Usage and analytics events | 24 months, then aggregated | Nigeria | Anonymised on expiry |
| Support cases | 24 months | Nigeria | Deleted on request where no statutory hold applies |

Access, correction, export, and deletion requests are acknowledged within 72 hours and completed within 30 days.

## Measurement and verification

| ID | Verified by | When | Owner |
|---|---|---|---|
| NFR-001 | Device-lab performance budget | Every release | Mobile |
| NFR-002 | Load test on recorded intents | Pre-launch, then monthly | Platform |
| NFR-004 | Synthetic payment probe | Continuous | Payments |
| NFR-006 | USSD channel monitor | Continuous | Platform |
| NFR-010 | Uptime monitor and monthly SLO review | Continuous and monthly | Platform |
| NFR-011 | Synthetic authorization probe | Continuous | Payments |
| NFR-020 | Penetration test and security review | Pre-launch, annually | Security |
| NFR-027 | Mandate verification test suite | Every release | Payments |
| NFR-030 | Accessibility audit against WCAG 2.2 AA | Pre-launch, then every release | Client |
| NFR-040 | Privacy review and data subject request rehearsal | Pre-launch, annually | Compliance |
| NFR-050 | Cost telemetry per completed task | Monthly | Platform |
