# Sabi

**One-liner:** An AI agent that finds, pays for, and stores verifiable receipts for Nigerian government services and everyday bills, through conversation on the phone a user already owns.
**Status:** In Documentation
**Interfaces:** Mobile (Android, iOS), USSD/SMS, WhatsApp
**AI:** Core

## Overview

Sabi is a conversational payment agent. A user says what they need to pay or settle, in English or Nigerian Pidgin, and Sabi identifies the responsible agency or biller, confirms the amount, executes the payment through licensed rails, retrieves the Federal Treasury e-Receipt for federal payments, and keeps the receipt as a retrievable record. The platform is extensible: providers, agencies, and merchants register as independent agent services that Sabi discovers and calls on the user's behalf, and the assistant composes the interface for a task at the moment it is needed instead of forcing every provider into one fixed form.

## The problem

Paying government and bills in Nigeria is fragmented. A user has to know which agency or biller is responsible, find the right portal or USSD code, work out the amount, pay, and then keep separate proof for each channel. Small businesses with recurring statutory obligations such as taxes, levies, fees, and fines carry the same problem with higher stakes. The cost is time, wrong payments, lost receipts, and disputes that are hard to settle.

**How it's solved today:** users move between the OAGF gateway (Remita, being replaced by TMRAS), fintech bill-pay screens such as OPay and PalmPay, bank USSD and mobile apps, and physical agent networks ([Techpoint](https://techpoint.africa/news/nigeria-abandons-remita/), [KPMG and Orange smartphone study](https://assets.kpmg.com/content/dam/kpmgsites/ng/pdf/2026/09/Nigeria%20Smartphone%20Study%20-%20Orange%20Group%20and%20KPMG.pdf.coredownload.inline.pdf), [Techdom Africa](https://techdomafrica.com/super-apps-must-serve-local-realities/)). Discovery is mostly word of mouth, and proof of payment is scattered across SMS, email, and paper.

From 1 January 2026 the workarounds narrow. The federal government banned cash collection for all federal revenues and made the Federal Treasury e-Receipt (FTeR) the only legally recognised proof of payment, generated centrally on the Revenue Optimisation (RevOp) platform that consolidates TSA, GIFMIS, CBN, NIBSS, and FIRS systems ([Businessday](https://businessday.ng/news/article/fg-rolls-out-unified-revenue-platform-in-biggest-overhaul-since-tsa/), [Arise News](https://www.arise.tv/nigeria-bans-cash-payments-for-all-government-revenues-to-boost-transparency/)).

## The solution

Sabi replaces the search, the portal, the amount lookup, and the receipt chase with one conversation. The user states an intent; Sabi resolves it to a real agency or biller, shows the amount and any charges, takes authorisation, pays through licensed infrastructure, and returns a stored, verifiable receipt. For federal payments the receipt is the FTeR, which Sabi retrieves rather than re-creates.

Two choices separate this from a bill-pay screen. First, the interface is generated per task: the agent composes the confirmation, form, or receipt view it needs from the client's trusted components, so a new provider does not require a new screen. Second, providers are agents, not integrations: each registers a discoverable capability, and Sabi delegates to it and collects the result. Adding a state agency, a university, or a new biller is a registration, not a release.

## Market & competition

**Alternatives today:**
- OAGF payment gateway (Remita, transitioning to TMRAS) for federal revenue ([Techpoint](https://techpoint.africa/news/nigeria-abandons-remita/))
- Remita's consumer super app, launched at Nigeria Fintech Week 2026, which brings bank accounts, bill pay, government payments, and money management into one app ([Daily Post](https://dailypost.ng/2026/09/25/remita-launches-super-mobile-app-to-bring-banking-payments-money-management-into-one-place/))
- Fintech bill-pay inside OPay, PalmPay, and Moniepoint, adopted by up to 88% of surveyed Nigerian smartphone users for banking and fintech apps ([KPMG and Orange](https://assets.kpmg.com/content/dam/kpmgsites/ng/pdf/2026/09/Nigeria%20Smartphone%20Study%20-%20Orange%20Group%20and%20KPMG.pdf.coredownload.inline.pdf))
- Bank USSD strings and mobile banking apps
- Physical agent and PoS networks, which Opay and others use for cash-in and cash-out ([Techdom Africa](https://techdomafrica.com/super-apps-must-serve-local-realities/))
- WhatsApp commerce and ordering agents already in market, such as Queek ([Queek](https://queek.com.ng/))

**Demand signal:** Real-time payment volume is large and established. NIP processed about 1.82 billion transactions worth N320.76 trillion in Q1 2026 alone ([TV360 Nigeria, citing CBN](https://www.tv360nigeria.com/electronic-payments-hit-n1-05-quadrillion-in-q1-2026-despite-9-2-drop-in-volume/)). The FTeR mandate creates a forced migration for every payer to a federal agency ([Businessday](https://businessday.ng/news/article/fg-rolls-out-unified-revenue-platform-in-biggest-overhaul-since-tsa/)).

**Platform/regulatory exposure:** Payment services require CBN licensing or a licensed partner; PSB licences bar lending, foreign-currency deposits, and FX dealing ([CBN PSB guidelines](https://usercontent.one/wp/www.acaebin.org/wp-content/uploads/2023/07/APPROVED20REVIEWED20GUIDELINES20FOR20LICENSING20AND20REGULATION20OF20PAY.pdf)). Federal collection requires an OAGF-regularised PSSP channel ([Arise News](https://www.arise.tv/nigeria-bans-cash-payments-for-all-government-revenues-to-boost-transparency/)). Personal data handling falls under the Nigeria Data Protection Act 2023, enforced by NDPC with penalties up to N10,000,000 or 2% of annual gross revenue ([NDPA 2023](https://ncc.gov.ng/sites/default/files/2024-11/Nigeria_Data_Protection_Act_2023.pdf)). Identity is increasingly foundational: the NIMC Act 2026 makes the NIN the national credential under a one person, one identity policy ([The Eastern Updates](https://theeasternupdates.com/2026/07/13/nimc-given-2026-deadline-to-enrol-all-nigerians-dg-says/)).

**Verdict:** Reshape. The "WeChat for Africa" framing is not defensible: prior attempts failed structurally and MTN partnered with Ant International to launch a MoMo mini-app super app in Nigeria from Q3 2026 ([Techmoonshot](https://techmoonshot.com/2026/06/10/mtn-and-ant-international-partner-to-transform-momo-into-a-super-app-nigeria-goes-first/)). The government and bills agent is a narrower, time-boxed opening. Proceed on that wedge.

## Target audience

**Primary:** Nigerian adults and small businesses that owe recurring government payments (taxes, levies, fees, fines) and recurring bills (electricity, airtime, data, television, water), who use a low-end to mid-range Android phone or a feature phone, and who currently pay through agents, USSD, or fintech apps.
**Secondary:** diaspora Nigerians paying bills and statutory obligations for family at home, and informal agents who pay on behalf of others.
**Not for:** users looking for a WhatsApp replacement or a general social network, borrowers seeking credit, and users seeking crypto custody or trading.

These users are chosen because the payment obligation is non-optional and recurring, the current path costs time and produces weak proof, and the channel mix (Android, iOS, USSD/SMS, WhatsApp) matches how the audience actually connects.

## Success metrics & KPIs

| Metric | Definition | Baseline | Target | Timeframe | Source |
|---|---|---|---|---|---|
| Activation | Share of new users who complete one successful payment within 7 days | None (new product) | Above 35% | 90 days post-launch | Product telemetry (planned) |
| Payment completion | Successful payments divided by payments started | None | Above 90% | 90 days | Product telemetry (planned) |
| Repeat use | Share of activated users who transact again in month 2 | None | Above 40% | 6 months | Product telemetry (planned) |
| Receipt coverage | Share of government payments with a stored, verifiable e-receipt | None | 100% | At launch | Product telemetry (planned) |
| Agent resolution | Share of requests resolved without human escalation | None | Above 80% | 6 months | Product telemetry (planned) |

## SWOT analysis

### Strengths
- The agent and interface layer is designed on current open standards (A2A v1.0, A2UI, AP2), so third parties can register a capability instead of waiting for a bespoke integration ([Linux Foundation](https://www.linuxfoundation.org/press/a2a-protocol-surpasses-150-organizations-lands-in-major-cloud-platforms-and-sees-enterprise-production-use-in-first-year), [A2UI](https://a2ui.org/), [AP2](https://ap2-protocol.org/)).
- The FTeR and RevOp mandate gives a dated, government-driven reason for payers to change behaviour, which incumbents built around the old gateway are not positioned for ([Businessday](https://businessday.ng/news/article/fg-rolls-out-unified-revenue-platform-in-biggest-overhaul-since-tsa/)).
- A multi-channel design (Android, iOS, USSD/SMS, WhatsApp) reaches users that an app-only competitor cannot, on devices where affordability, not data cost, is the binding constraint ([GSMA smartphone adoption](https://www.gsma.com/about-us/regions/africa/wp-content/uploads/2025/11/GSMA-SmartPhone_Adoption_Report_final.pdf)).
- Passwordless onboarding lowers signup friction for the social and discovery layer.

### Weaknesses
- No payments licence, no banking partner, and no agent network exist at concept stage; the product cannot move money on its own.
- No distribution. The app competes with services already installed and with habits held on WhatsApp.
- Nigerian-language model quality is weak today: GPT-4o scores above 90% on English exam questions but below 40% on Yoruba, Igbo, and Hausa, so agent comprehension must be built and evaluated rather than assumed ([AfricaNLP 2025](https://sites.google.com/view/africanlp2025/home)).
- Federal receipt and collection integrations depend on OAGF approval and PSSP regularisation, which the team does not control.

### Opportunities
- The CBN is issuing an open banking implementation roadmap in 2026, which lowers the cost of aggregating accounts ([21st Century Chronicle](https://21stcenturychronicle.com/cbn-to-launch-open-banking-roadmap-in-three-months/)).
- NIN enrolment and the NIMC root certificate authority give a growth path to verified identity for higher-value services ([The Eastern Updates](https://theeasternupdates.com/2026/07/13/nimc-given-2026-deadline-to-enrol-all-nigerians-dg-says/)).
- The cashless federal mandate creates a one-time migration window for a product built around the e-receipt ([Arise News](https://www.arise.tv/nigeria-bans-cash-payments-for-all-government-revenues-to-boost-transparency/)).
- The gap in African-language model quality is a defensible asset if Sabi owns it rather than renting it ([NaijaNLP](https://arxiv.org/html/2502.19784v2)).
- Diaspora bill payment for family is a paying segment reachable without local device constraints.

### Threats
- MTN and Ant International are launching a MoMo mini-app super app in Nigeria from Q3 2026, with telecom distribution and an existing wallet base ([Techmoonshot](https://techmoonshot.com/2026/06/10/mtn-and-ant-international-partner-to-transform-momo-into-a-super-app-nigeria-goes-first/)).
- The incumbent federal gateway is moving into the consumer channel: Remita launched a super app in September 2026, so the gateway that already owns federal payments now competes for the same users ([Daily Post](https://dailypost.ng/2026/09/25/remita-launches-super-mobile-app-to-bring-banking-payments-money-management-into-one-place/)).
- WhatsApp holds roughly 51 million Nigerian users and near-universal reach among internet users, so any social or conversational layer competes with an entrenched default ([Krestel Digital](https://kresteldigital.com/resources/social-media-statistics-in-nigeria-2026-data-report/)).
- CBN licensing categories and PSB restrictions can change the financial surface available under a single entity ([CBN PSB guidelines](https://usercontent.one/wp/www.acaebin.org/wp-content/uploads/2023/07/APPROVED20REVIEWED20GUIDELINES20FOR20LICENSING20AND20REGULATION20OF20PAY.pdf)).
- Identity data quality is uneven: as of mid-2026 there was a gap between enrolments and issued NINs, and the World Bank flagged accuracy and authentication risks ([HCN Times](https://hcntimes.com/nigeria-needs-62-5-million-nins-in-six-months-to-meet-target/)).
- Fraud and low trust in digital payments raise the cost of every money movement.

## Scope

**In scope:**
- Conversational discovery, payment, and receipt handling for federal government payments and everyday bills
- A passwordless identity and onboarding path that does not require a phone number for the discovery layer
- An extensible agent and service registry so agencies, billers, and merchants can be added without an app release
- Verifiable, retrievable receipts as a first-class record

**Out of scope (for now):**
- Social feed, general messaging, and community features, deferred until transaction use creates a reason to gather
- Merchant storefronts and business operations tooling
- State-level government payments beyond federal agencies
- Lending, savings, insurance, FX, and crypto

**Explicit non-goals:**
- Replacing WhatsApp as a chat product
- Holding customer balances as a bank or becoming a deposit-taking institution
- Offering credit or underwriting
- Custodying or trading crypto assets

## Assumptions & dependencies

**Assumptions** - believed true; if wrong, the plan changes:

| # | Assumption | Risk if false | Validated? |
|---|---|---|---|
| A1 | Users will authorise payments through a conversational agent rather than a form | The core interaction is rejected and the product reverts to a conventional bill-pay UI | No |
| A2 | A licensed partner will provide the payment rail while Sabi owns the agent layer | Movement of money is blocked and only discovery ships | No |
| A3 | Nigerian-language comprehension can be raised to a usable bar with targeted data and evaluation | The agent misunderstands intents and loses trust | No |
| A4 | The FTeR and RevOp mandate persists on its current timeline | The forced-migration window narrows and urgency drops | Partly - the mandate is dated and published ([Businessday](https://businessday.ng/news/article/fg-rolls-out-unified-revenue-platform-in-biggest-overhaul-since-tsa/)) |

**Dependencies** - external and blocking, not under the team's control:

| # | Dependency | Type | Blocks | Status |
|---|---|---|---|---|
| D1 | CBN-licensed payment partner or licence | Legal | All payment features | Not started |
| D2 | OAGF-approved PSSP channel for federal collections | Legal and platform | Government payment features | Not started |
| D3 | NIN or BVN verification path for regulated payments | Platform | KYC for payments | Researching |
| D4 | Agent-to-agent and agent-to-UI standards remain stable | Technology | Extensibility and generated interfaces | Available at v1.0 and v0.9.1 ([A2A](https://www.linuxfoundation.org/press/a2a-protocol-surpasses-150-organizations-lands-in-major-cloud-platforms-and-sees-enterprise-production-use-in-first-year), [A2UI](https://a2ui.org/)) |

## Documentation

| Document | Purpose |
|---|---|
| [user-personas](user-personas.md) | Who this product is for |
| [branding](branding.md) | Brand identity, design character, and the design system |
| [features](features.md) | What gets built, prioritized, with acceptance criteria |
| [nfr](nfr.md) | The quality targets the system must meet |
| [roadmap](roadmap.md) | When it gets built, in phases |
| [architecture](architecture.md) | How the system is structured |
| [risk-log](risk-log.md) | What could go wrong and the plan for it |
| [user-flow](user-flow.md) | How users move through the product |
| [screens](screens.md) + `screens/` | Every screen, its states, and its components |
| [schema](schema.md) | The data model |
| [model-quality](model-quality.md) | The model behavior and quality contract |
