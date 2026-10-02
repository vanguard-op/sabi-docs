# Sabi: risk log

## Risk assessment scale

### Likelihood

| Score | Label | Definition |
|---|---|---|
| 1 | Rare | Under 5% chance in the project timeline |
| 2 | Unlikely | 5% to 20% chance |
| 3 | Possible | 20% to 50% chance |
| 4 | Likely | 50% to 80% chance |
| 5 | Almost certain | Over 80% chance |

### Impact

| Score | Label | Definition |
|---|---|---|
| 1 | Negligible | Minor inconvenience, easily absorbed |
| 2 | Minor | Some disruption, manageable with existing resources |
| 3 | Moderate | Significant disruption, requires dedicated effort |
| 4 | Major | Threatens timeline or budget materially |
| 5 | Critical | Threatens product viability or reputation |

### Risk score

**Risk score = likelihood x impact**

| Score range | Severity | Action |
|---|---|---|
| 1 to 4 | Low | Monitor |
| 5 to 9 | Medium | Plan mitigation, monitor regularly |
| 10 to 19 | High | Active mitigation required, escalate to stakeholders |
| 20 to 25 | Critical | Immediate action required, may require a product pivot |

## Risk register

### Market risks

| ID | Risk | L | I | Score | Mitigation | Contingency | Owner |
|---|---|---|---|---|---|---|---|
| R-001 | MTN and Ant International launch a MoMo mini-app super app in Nigeria from Q3 2026, and WhatsApp remains the default for communication, so Sabi is squeezed between an incumbent platform and an entrenched habit ([Techmoonshot](https://techmoonshot.com/2026/06/10/mtn-and-ant-international-partner-to-transform-momo-into-a-super-app-nigeria-goes-first/), [Krestel Digital](https://kresteldigital.com/resources/social-media-statistics-in-nigeria-2026-data-report/)) | 4 | 4 | 16 | Compete on the government and receipt wedge that neither platform is built around; make provider registration open so Sabi can add agencies faster than a closed ecosystem; own Nigerian-language quality | Partner with a licensed payment provider that also wants the government channel; reposition as a layer that works over WhatsApp rather than against it | Product Manager |
| R-002 | Users reject conversational payment and prefer existing bill-pay screens and agents | 3 | 5 | 15 | Test the interaction before full build; support a conventional fallback form for every task; measure activation and completion from launch | Fall back to a guided form flow with the agent as an assistant rather than the primary interface | Product Manager |
| R-015 | Fraud, scams, and low trust in digital payments deter adoption, and a single publicised loss damages the brand ([FIJ](https://fij.ng/article/opay-palmpay-keep-publisher-waiting-11-months-after-he-flagged-n480000-taken-by-fraudsters/)) | 4 | 4 | 16 | Signed payment mandates bound to amount and payee; automatic refund path; transparent case status; no custody of balances | Public incident response with restitution, a published post-mortem, and tightened controls | Head of Trust and Safety |
| R-016 | The incumbent OAGF gateway moves into the consumer channel and contests the government-payment wedge it already owns, since Remita launched a super app with government payments and bill pay in September 2026 ([Daily Post](https://dailypost.ng/2026/09/25/remita-launches-super-mobile-app-to-bring-banking-payments-money-management-into-one-place/)) | 3 | 4 | 12 | Compete on the FTeR-native receipt and the conversational agent rather than on incumbency; keep provider registration open so agencies can be added faster than a closed gateway; own Nigerian-language quality | Position as a layer that sits over the gateway's collection channel, and differentiate on verifiable receipts and reach to feature phones | Product Manager |

### Technical risks

| ID | Risk | L | I | Score | Mitigation | Contingency | Owner |
|---|---|---|---|---|---|---|---|
| R-009 | The model mis-resolves an intent and a payment is sent to the wrong payee or amount | 3 | 5 | 15 | Confirm before execution; refuse below the MQ-001 threshold; signed mandate bound to amount and payee; never emit an amount or reference absent from retrieved data | Reverse and refund through the payment partner, notify the user, and preserve the audit trail | AI Engineering |
| R-010 | Nigerian-language coverage stays below a usable bar, so the assistant misunderstands Hausa, Yoruba, or Igbo users ([AfricaNLP 2025](https://sites.google.com/view/africanlp2025/home)) | 4 | 3 | 12 | Build and label a dataset per language; gate each language behind its MQ-004 bar; keep Pidgin at launch scope | Keep those languages disabled and serve English with a note until the bar is met | AI Engineering |
| R-011 | Prompt injection from user input or retrieved content triggers an unsafe action | 2 | 5 | 10 | Treat all input and retrieved content as untrusted; prevent retrieved instructions from overriding payment policy; block executable content in composed interfaces | Disable the affected retrieval source, require manual review of the action, and audit for similar attempts | Security |
| R-012 | Dependency on a single model provider, model deprecation, or silent quality drift degrades the product | 3 | 4 | 12 | Pin the model version; re-evaluate every bar on any upgrade; run live quality and drift monitoring; keep the keyword fallback path maintained | Switch to the evaluated replacement or the fallback path until quality is restored | AI Engineering |

### Operational risks

| ID | Risk | L | I | Score | Mitigation | Contingency | Owner |
|---|---|---|---|---|---|---|---|
| R-003 | No distribution. The app competes with services already installed, so growth is slower than planned | 4 | 4 | 16 | Lead with the government e-receipt wedge and provider relationships; use USSD and WhatsApp to reach users without an install; target diaspora payers who have a concrete need | Acquire through agency and provider partnerships and agent networks rather than paid installs | Product Manager |
| R-008 | Identity data quality or verification failure blocks regulated payments, since NIN records show a gap between enrolments and issued identifiers ([HCN Times](https://hcntimes.com/nigeria-needs-62-5-million-nins-in-six-months-to-meet-target/)) | 3 | 3 | 9 | Accept multiple identity paths (NIN, BVN) where the rail allows; queue verification instead of failing; give a clear correction route | Allow manual verification with supporting documents while the automated path is unavailable | Compliance |
| R-014 | Support and reversal load grows faster than capacity because failed payments and disputes are common in the market | 3 | 4 | 12 | Automate reversal and receipt cases; expose case status in the app; keep a self-service refund path; run the case queues and service windows in [operations](operations.md) | Add support capacity and a dedicated dispute queue before a national push | Operations |

### Legal and compliance risks

| ID | Risk | L | I | Score | Mitigation | Contingency | Owner |
|---|---|---|---|---|---|---|---|
| R-004 | No payment licence is held and no licensed partner is secured, so money cannot move | 3 | 5 | 15 | Start partner and licence discussions before build; ship discovery first so value exists without a licence | Launch discovery and receipt storage only, and defer payments until a partner is signed | Legal |
| R-005 | OAGF or PSSP approval for federal collections is delayed or refused, blocking the wedge | 3 | 4 | 12 | Engage the OAGF early; use the open banking and licensing dialogue to align ([21st Century Chronicle](https://21stcenturychronicle.com/cbn-to-launch-open-banking-roadmap-in-three-months/)) | Serve state and local obligations and everyday bills while federal approval is pending | Legal |
| R-006 | CBN changes licence categories or tightens PSB restrictions, altering the financial surface available ([CBN PSB guidelines](https://usercontent.one/wp/www.acaebin.org/wp-content/uploads/2023/07/APPROVED20REVIEWED20GUIDELINES20FOR20LICENSING20AND20REGULATION20OF20PAY.pdf)) | 3 | 4 | 12 | Keep the financial model modular so it can run under a partner or a different licence; monitor CBN consultations | Re-plan the licensed surface and move affected features behind a partner | Legal |
| R-007 | A personal data breach or non-compliance with the Nigeria Data Protection Act 2023 triggers penalties up to N10,000,000 or 2% of gross revenue ([NDPA 2023](https://ncc.gov.ng/sites/default/files/2024-11/Nigeria_Data_Protection_Act_2023.pdf)) | 2 | 5 | 10 | Apply the controls in [nfr](nfr.md); minimise identity data; rehearse data subject requests and breach response | Notify the NDPC and affected users, contain the breach, and complete a full review | Compliance |

### Financial risks

| ID | Risk | L | I | Score | Mitigation | Contingency | Owner |
|---|---|---|---|---|---|---|---|
| R-013 | Model and payment costs run ahead of revenue, especially if retries and clarification turns are not bounded | 3 | 3 | 9 | Cost ceilings per completed task (NFR-050 to NFR-052); monitor cost per task monthly; prefer cheaper decoding paths for simple intents | Cap retries, reduce model calls, and shift simple intents to the keyword path | Finance and Platform |

## Risk trends

| Date | Change | Risk ID | Description |
|---|---|---|---|
| 2026-09-26 | Added | R-001 to R-015 | Initial register covering the four weaknesses and five threats in [overview](overview.md) and the model risks required for the AI-core shape |
| 2026-10-02 | Added | R-016 | New threat in [overview](overview.md): Remita's incumbent gateway moved into the consumer channel |

## Review cadence

- Weekly: review all High and Critical risks (score 10 and above) in team standup.
- Monthly: full register review with stakeholders.
- Per milestone: reassess all risks before each roadmap phase begins.
