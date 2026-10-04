# Sabi: features

## Overview

This feature set delivers a conversational agent that resolves, pays, and records Nigerian government obligations and everyday bills. The launch outcome is a user who can state a payment need in English or Pidgin, authorise it, and hold a verifiable receipt, on an app, USSD/SMS, or WhatsApp. The platform is extensible: providers register as discoverable agent services, so new billers and agencies are added without an app release.

## Priority definitions

| Priority | Label | Meaning |
|---|---|---|
| **P0** | Must have | Critical for launch. The product cannot ship without these. |
| **P1** | Should have | Important but not launch-blocking. Ship soon after launch. |
| **P2** | Could have | Nice to have. Adds reach or revenue. Ship when capacity allows. |
| **P3** | Won't have now | Acknowledged but deferred. Revisit in a future planning cycle. |

## Foundations: onboarding and identity

### FEAT-001: Passwordless onboarding and identity

**Priority:** P0
**Effort:** M
**Depends On:** None

**User story:**
As a new user, I want to create an account and return to it without a password or a phone number, so that I can use discovery and the assistant immediately.

**Description:**
Creates an account using a passkey as the primary credential, with an account recovery code as a fallback and an optional email. A phone number is not required for the discovery layer. Channel identity (USSD number, WhatsApp number) can link to the same account later. This supports Ngozi, Musa, and Tunde in starting without a password.

**Acceptance criteria:**
- **Given** a user on a passkey-capable device
  **When** the user registers
  **Then** an account is created, a passkey is enrolled, and a recovery code is shown once for saving.
- **Given** a user who has lost their only passkey
  **When** they enter a saved recovery code
  **Then** they regain access and are prompted to enrol a new passkey.
- **Given** an existing account
  **When** the user links a USSD or WhatsApp number
  **Then** the channel is attached to the same account without creating a duplicate.

**Edge cases and failure states:**
- Device without passkey support: fall back to recovery code plus email verification, and state the reduced security.
- Recovery code and passkey both lost: account is not recoverable; state this before enrolment.
- Two users link the same channel number: second link is rejected and the account owner is notified.

### FEAT-002: KYC verification for regulated payments

**Priority:** P0
**Effort:** M
**Depends On:** FEAT-001

**User story:**
As a user making a regulated payment, I want to verify my identity once, so that my payments comply with KYC rules and I am not asked again for every provider.

**Description:**
Verifies identity against the national identity infrastructure (NIN, with BVN where the payment rail requires it) and stores a verification status on the account. Verification gates only the payment features that require it; discovery remains open. This supports Ifeoma and Tunde.

**Acceptance criteria:**
- **Given** a user who has not verified
  **When** they attempt a payment that requires KYC
  **Then** verification is requested, and the payment is held rather than failed.
- **Given** a verified user
  **When** they start another regulated payment
  **Then** no repeat verification is requested.
- **Given** an identity check that returns no match
  **When** the check completes
  **Then** the user sees a plain reason and a route to correct their details.

**Edge cases and failure states:**
- Identity provider unavailable: queue the verification, tell the user it is pending, and do not block discovery.
- Data mismatch between supplied details and the identity record: allow correction once, then require support.
- A verified account is later flagged for fraud: suspend payment features and preserve the audit trail.

## Agent core: intent, discovery, and interface

### FEAT-003: Agent intent understanding in English and Pidgin

**Priority:** P0
**Effort:** XL
**Depends On:** None

**User story:**
As a user, I want to describe what I need to pay in my own words, so that I do not have to learn a menu structure or a portal.

**Description:**
Interprets a free-form request (for example, "pay my land use charge" or "buy five thousand MTN") and turns it into the action the client takes. Handles English and Nigerian Pidgin at launch, including common payment and government vocabulary. The assistant is conversational: the model authors its user-facing replies in natural language (the greeting, the clarifying questions, the refusals, and the non-resolving reply), while structured output carries only the machine-actionable intent the client acts on (the action, target, amount, reference, candidates, and confidence). It keeps the context of the exchange and, when a request does not resolve, asks follow-up questions that build on what the user has already said rather than restarting. The model may phrase its reply freely, but every fact in it (an amount, a reference, a service identifier, a collecting body) comes only from the retrieved context, and it never states one that is not grounded. Fallback: when confidence is below threshold, Sabi asks a follow-up question with the closest candidates. It may ask up to three clarifying questions in one conversation; if the request is still unresolved after the third, it gives a non-resolving reply that states plainly why it cannot proceed, offers catalogue search and human support, and invites the user to continue, rather than guessing. A greeting or other non-command message is answered conversationally, with what Sabi can do and an invitation to state a request, and is never treated as a coverage miss or recorded for coverage. The assistant also resolves a question about a record the user already holds: a lookup such as "show me the receipt for my December NEPA payment" or "did my Airtel top-up go through" is understood as a record lookup, grounded in the user's own records, and answered either by rendering the record inline as a composed surface or by routing the user to the existing screen that shows it, with the choice left to the user and neither route forced. Every non-resolving and conversational reply (the after-bound reply, the greeting reply, and the clarifying question) is honest and specific, names a concrete next step, and is category-neutral: it never assumes or prefers a service category or an identifier type, and it never substitutes a different specific service for the one the user named. Because naming a specific payee asserts ownership, the assistant never answers a request for one named service by proposing another of the same kind ("Airtel airtime" is never answered by offering "MTN airtime"; "Ikeja Electric" is never answered by offering "Eko Electric"). Asking a neutral, category-neutral disambiguating question ("which network?", "which provider?", "the collecting body's name?") is allowed and encouraged, and offering catalogue search is allowed, because both leave the choice to the user rather than asserting a payee. No payment is initiated without explicit confirmation, on any turn.

**Acceptance criteria:**
- **Given** a Pidgin request such as "I wan pay my electricity"
  **When** submitted
  **Then** Sabi returns a structured bill-payment intent with the electricity biller category as the target.
- **Given** a request naming a levy Sabi knows
  **When** submitted
  **Then** Sabi returns the responsible agency and asks for confirmation before payment.
- **Given** a request Sabi cannot map
  **When** confidence is below threshold
  **Then** Sabi asks a follow-up question with the closest candidates instead of initiating a payment, and keeps the conversation open for an answer.
- **Given** an answer that still does not resolve the request
  **When** fewer than three clarifying questions have been asked
  **Then** Sabi asks another, narrowed by the answers already given.
- **Given** a request still unresolved after three clarifying questions
  **When** the conversation reaches the bound
  **Then** Sabi gives a specific reply that states plainly why it cannot proceed and what the user can do next, offers catalogue search and human support, records the request for coverage, initiates no payment, and invites the user to continue in the conversation.
- **Given** a greeting such as "Hello"
  **When** submitted
  **Then** Sabi responds conversationally with what it can do and an invitation to state a request, and does not record the message as a coverage gap.
- **Given** a record lookup such as "show me the receipt for my December NEPA payment"
  **When** submitted
  **Then** Sabi resolves it as a record lookup, grounds the answer in the user's own records, and either renders the matching record inline as a composed surface or routes the user to the existing screen that shows it, leaving the choice to the user.
- **Given** a record lookup such as "did my Airtel top-up go through"
  **When** submitted
  **Then** Sabi answers from the source record (state, amount, reference, date) and never from model recall.
- **Given** a record lookup that matches no record
  **When** it completes
  **Then** Sabi says plainly that no matching record was found, composes no record surface, and does not fabricate one.
- **Given** a clarifying question, a greeting reply, or an after-bound reply
  **When** the model authors it
  **Then** it names the general thing the user must supply (a collecting body's name, a reference, or a corrected detail) and never assumes one service category or identifier type as the default.
- **Given** a request naming a specific service ("Airtel airtime", "Ikeja Electric")
  **When** it does not resolve to that service
  **Then** the reply never proposes a different specific service of the same kind as a substitute, and instead asks a neutral disambiguating question or offers catalogue search.
- **Given** a model-authored reply that names an amount, a reference, a service identifier, or a collecting body
  **When** it is produced
  **Then** every such fact is present in the retrieved context and none is stated that is not grounded.

**Edge cases and failure states:**
- Ambiguous target matches several agencies: present the top candidates and require a choice.
- Model provider unavailable: fall back to keyword and search-based resolution and label the result as unresolved until confirmed.
- A request that names another person's account: refuse and explain, with the same honest reason and concrete next step.
- Non-command message (a greeting, thanks, or small talk): answer conversationally; never count it as a clarification turn, record it for coverage, or start a task.
- The user stops answering mid-clarification: keep the request on screen, start no task, and record nothing for coverage.
- A clarifying question or non-resolving reply that would name a category's artefact (for example a meter number): the model replaces it with the general thing the user must supply, so no single category is assumed.
- The user names a specific service that does not resolve ("Airtel airtime" when only "MTN airtime" matches, "Ikeja Electric" when only "Eko Electric" matches): the reply must never offer the different specific service as a substitute, because that asserts ownership and risks sending money to the wrong place. It asks a neutral disambiguating question ("which network?", "which provider?") or offers catalogue search, and the user chooses from real catalogue rows or states the correct payee.
- A model-authored reply that is missing or unusable, or a model that is unavailable: the client renders its prewritten fallback reply for the state, so the user is never shown a blank turn.
- A record lookup that matches several records: present the candidates and let the user choose which one to open, rather than guessing.
- A record lookup raised while a payment task is in progress: answer it without abandoning the open task, and keep the task's place in the conversation.
- A record the user is not entitled to (another account's reference): never disclose it; return nothing for the mismatched reference.

### FEAT-004: Service, agency, and biller discovery

**Priority:** P0
**Effort:** L
**Depends On:** FEAT-003

**User story:**
As a user, I want to search for the right agency, levy, or biller and see what I owe, so that I pay the correct place the first time.

**Description:**
Resolves a query to a real agency, obligation, or biller from a maintained catalogue, and shows what is known about the obligation (amount where an agency or biller provides it, reference, deadline). Search accepts text, and results show the collecting body and any charge. This serves Ngozi, Ifeoma, and Tunde. Output contract: a ranked list of structured service records with identifiers, collecting body, and known amount in minor units; the client renders selectable results, and the model authors the reply that accompanies them. Fallback: when the catalogue has no match, Sabi gives a non-resolving reply in the register that there is no resolution for that service right now (true of the request and forward-looking), offers catalogue search and human support, and invites the user to name the collecting body or another detail, and records the request rather than fabricating a result. It never substitutes a different specific service for the one the user named: because naming a specific payee asserts ownership, a query for one named service is never answered by offering another of the same kind ("Airtel airtime" is never answered by offering "MTN airtime"; "Ikeja Electric" is never answered by offering "Eko Electric"). A neutral disambiguating question ("which network?", "which provider?", "the collecting body's name?") and catalogue search are allowed, because both leave the choice to the user rather than asserting a payee.

**Acceptance criteria:**
- **Given** a query "council levy Onitsha"
  **When** searched
  **Then** the results list the matching collecting body with its identifier.
- **Given** a selected obligation with a known amount
  **When** opened
  **Then** the amount, reference, and deadline are shown before payment.
- **Given** an obligation with no known amount
  **When** opened
  **Then** the user is told the amount will be confirmed at payment.
- **Given** a query naming a specific service that does not resolve ("Airtel airtime", "Ikeja Electric")
  **When** searched
  **Then** no different specific service of the same kind is offered as a substitute, and a neutral disambiguating question or catalogue search is given instead.

**Edge cases and failure states:**
- Duplicate or renamed agencies: prefer the current registered name and note the former name.
- A query names a specific service that does not resolve ("Airtel airtime" when only "MTN airtime" matches, "Ikeja Electric" when only "Eko Electric" matches): no different specific service of the same kind is offered as a substitute, because that asserts ownership and risks sending money to the wrong place. A neutral disambiguating question ("which network?", "which provider?") or catalogue search is given instead, and the user chooses from real catalogue rows.
- No results: record the query for coverage planning, offer human support, and give the non-resolving reply in the register that there is no resolution for that service right now: the honest reason, a concrete next step, and an invitation to continue, with no claim that the service is unsupported.
- Stale catalogue data: mark records past their refresh window and warn before payment.

### FEAT-011: Agent-composed surface as the task mechanism

**Priority:** P0
**Effort:** L
**Depends On:** FEAT-003

**User story:**
As a user, I want the assistant to show me exactly the form, confirmation, receipt, or status I need for any task or record, so that I can get everything done in the conversation and a new provider or record type works without me updating the app.

**Description:**
The composed surface is the task mechanism: every task and every record type is rendered from the client's published Sabi catalog, generated by the agent rather than shipped per provider or per record kind. It covers every task-shaped surface the product has: service payment (confirmation, amount entry, provider choice), identity verification, failed-payment or case recovery, and the composed rendering of a resolved record such as a receipt or a payment status the user asked about.

The client publishes a Sabi **catalog**: a JSON Schema conforming to A2UI's Catalog schema, with a `catalogId` URI, that mirrors the Sabi design system's composed surfaces (the confirmation summary, amount entry, provider picker, service result row, receipt card, status pill, verification, and recovery). The catalog is the single source of truth the agent composes against and both sides validate against; the shell (composer, navigation, static chrome, fixed buttons and labels) is not in the catalog and is never composed. A single published catalog removes the drift that a server-side hand copy of the client's component names introduces when the two fall out of step.

The surface is expressed as A2UI messages and rendered through the official Flutter **GenUI SDK** renderer against the catalog, not a bespoke parser. The lifecycle follows A2UI: the client advertises its `supportedCatalogIds` and the agent selects from them when it creates a surface; the agent validates the payload against the catalog before sending and the client validates it again on receipt; an unknown component degrades gracefully to the fallback for the same task; and a client-side validation failure is reported back to the agent as `VALIDATION_FAILED`. The catalog is versioned per A2UI's additive-only rules (deprecate rather than delete, open enums), with a major version carried in the `catalogId` URI. The client owns styling and security; the agent may only request components the catalog exposes. This supports all personas by removing per-provider and per-record screens.

**Renderer maturity.** A2UI v0.9.1 is the current stable specification, but the Flutter `genui` package that renders it is alpha and is expected to change ([A2UI renderers](https://a2ui.org/reference/renderers/), [GenUI SDK for Flutter](https://docs.flutter.dev/ai/genui)). The design therefore keeps task completion independent of the SDK: the SDK renders the agent-composed surface when it can, and the client's prewritten fallback form for the same task carries the task when the SDK is unavailable, fails to render, or the surface does not validate.

Output contract: an A2UI message stream (surface creation, components, data model) conforming to the published catalog; the client renders natively. Fallback: if a surface fails validation, a component is unknown, or the GenUI SDK is unavailable or fails to render the surface, the client renders its prewritten text-and-form fallback for the same task, and the task remains completable.

**Acceptance criteria:**
- **Given** the published Sabi catalog
  **When** the agent creates a surface
  **Then** it names the catalog's `catalogId` from the client's `supportedCatalogIds` and composes only components the catalog defines.
- **Given** a service added after the app release
  **When** the user starts a payment to it
  **Then** the confirmation surface is composed against the published catalog with no app update.
- **Given** a payment that needs identity verification
  **When** the user must verify before paying
  **Then** the verification surface is composed in the conversation and the payment resumes there once verification succeeds.
- **Given** a failed or reversed payment the user must resolve
  **When** recovery is needed
  **Then** the recovery surface is composed in the conversation, showing the current state and the next step.
- **Given** a resolved record the user asked about
  **When** the record is rendered
  **Then** the record surface is composed against the published catalog with the record's own fields, not only a post-payment receipt.
- **Given** the prewritten shell
  **When** any surface is composed
  **Then** the composer, the navigation shell, the static chrome, and the fixed buttons and labels are not in the catalog and remain prewritten client code that the agent does not compose.
- **Given** a composed surface
  **When** displayed
  **Then** the client renders it through the official Flutter GenUI SDK renderer against the catalog, and the surface uses the client's components and styling.
- **Given** an agent payload that fails the client's validation against the catalog
  **When** received
  **Then** the client reports it to the agent as `VALIDATION_FAILED` and renders the fallback form for the same task.
- **Given** an agent UI message requesting an unknown component
  **When** received
  **Then** the client degrades gracefully: it skips or substitutes that component and renders the fallback for the same task, and the task stays completable.

**Edge cases and failure states:**
- Malformed UI message: discard it and render the fallback without losing the task.
- Partial data model: render placeholders and continue when the missing data arrives.
- Component requests a permission the client does not grant: refuse and continue with approved components.
- Agent sends no surface because no compatible catalog was found in the client's `supportedCatalogIds`: render the fallback form for the same task, so no task becomes unreachable.
- The GenUI SDK is unavailable or fails to render a surface: render the prewritten fallback form for the same task; task completion never depends on the alpha SDK.
- A task the catalog has no surface type for: fall back to the prewritten form or the existing screen, so no task becomes unreachable.
- A record lookup that resolves to no record: compose no record surface and give the conversational reply, rather than a blank or fabricated card.

## Payments and receipts

### FEAT-005: Payment authorization and execution

**Priority:** P0
**Effort:** L
**Depends On:** FEAT-002, FEAT-004

**User story:**
As a user, I want to authorise a payment from my chosen method and have it executed, so that the obligation is settled without leaving the conversation.

**Description:**
Collects explicit authorisation for a specific amount and payee, executes the payment through a licensed rail, and returns a transaction reference. Supports authorised-from-account and authorised-from-wallet, and applies AP2-style signed mandates so a payment is provably tied to the user's instruction. This serves all personas.

**Acceptance criteria:**
- **Given** a resolved obligation with amount and payee
  **When** the user authorises payment
  **Then** a single payment is executed and a transaction reference is returned.
- **Given** an execution that fails at the rail
  **When** the failure is returned
  **Then** the payment is not marked complete and the user sees a reason and next step.
- **Given** an authorisation for an amount above the user's set limit
  **When** submitted
  **Then** the payment requires a second confirmation.

**Edge cases and failure states:**
- Duplicate authorisation (double tap, retried session): execute once and return the same reference.
- Insufficient funds or limit: fail cleanly with no partial debit.
- Rail timeout with unknown outcome: mark as pending, do not retry immediately, and reconcile before informing the user.

### FEAT-006: Federal government payment flows

**Priority:** P0
**Effort:** L
**Depends On:** FEAT-005

**User story:**
As a payer to a federal agency, I want to settle taxes, levies, fees, and fines and receive the official receipt, so that I have proof the government recognises.

**Description:**
Covers federal payments to ministries, departments, and agencies, including taxes, levies, fees, and fines. The agency-side collection facts the shared catalogue does not carry, the collection account, whether a payer reference is mandatory and how to obtain one, and whether the account has changed, are held on `FederalCollection` in [schema](schema.md). On success, retrieves the Federal Treasury e-Receipt (FTeR) from the central platform and stores it against the obligation. This serves Ngozi, Ifeoma, and Tunde.

**Acceptance criteria:**
- **Given** a federal obligation with a valid reference
  **When** the payment succeeds
  **Then** the FTeR is retrieved and stored, and the user can open it.
- **Given** a payment that succeeds at the rail but has no FTeR yet
  **When** the user opens the receipt
  **Then** Sabi shows the payment as awaiting the official receipt and keeps checking.
- **Given** an obligation that requires a reference number
  **When** the user does not have one
  **Then** Sabi explains how to obtain it rather than paying without it.

**Edge cases and failure states:**
- Agency has switched collection accounts: detect the mismatch and route to the current channel.
- Rail confirms but the agency record does not reflect it: open a reconciliation case with the reference.
- Cash-only counter remains the only option: tell the user, since Sabi cannot pay cash.

### FEAT-007: Everyday bill payment flows

**Priority:** P0
**Effort:** L
**Depends On:** FEAT-005

**User story:**
As a user, I want to buy electricity, airtime, data, and television or water service in one place, so that daily payments are not spread across separate apps.

**Description:**
Covers electricity tokens, airtime, data, television subscriptions, and water where a biller is integrated. Returns the token or confirmation plus a receipt. This serves Ngozi and Musa.

**Acceptance criteria:**
- **Given** a prepaid electricity meter number
  **When** the user pays
  **Then** the token is returned and stored with the receipt.
- **Given** an airtime purchase
  **When** it succeeds
  **Then** the recipient number and amount are recorded and a receipt is available.
- **Given** a biller that is offline
  **When** the user tries to pay
  **Then** payment is prevented and the user is told to try later.

**Edge cases and failure states:**
- Payment succeeds but the token does not return: hold the transaction open and retry retrieval before marking complete.
- Wrong meter number entered: allow cancellation only before execution.
- Biller rejects after rail debit: trigger the refund path in FEAT-009.

### FEAT-008: Verifiable receipt capture and retrieval

**Priority:** P0
**Effort:** M
**Depends On:** FEAT-005

**User story:**
As a user who must account for payments, I want every receipt stored and retrievable, so that I can prove a payment months later.

**Description:**
Stores a receipt for every completed payment with its reference, amount, payee, timestamp, and, where applicable, the official receipt identifier such as the FTeR. Receipts are searchable and exportable. This serves Ifeoma and Tunde in particular.

**Acceptance criteria:**
- **Given** a completed payment
  **When** it completes
  **Then** a receipt is stored and linked to the obligation.
- **Given** a user searching a past payment
  **When** they search by reference, agency, or date
  **Then** the matching receipt is returned.
- **Given** an official receipt identifier exists
  **When** the receipt is opened
  **Then** the identifier is shown and marked as verified against the source.

**Edge cases and failure states:**
- Source receipt cannot be verified: mark the receipt as unverified, set `Receipt.unverifiedReason` in [schema](schema.md) to the plain reason, and state why.
- Receipt storage failure: queue for retry and warn the user not to discard the transaction reference.
- A receipt is disputed: preserve the original and record the dispute without altering it.

### FEAT-009: Payment status, failure, and refund handling

**Priority:** P0
**Effort:** L
**Depends On:** FEAT-005

**User story:**
As a user whose payment failed or lost money, I want a clear status and an automatic path to a refund, so that I am not left chasing a reversal.

**Description:**
Tracks each payment through its states, detects failure and reversal conditions, initiates refunds with the rail or biller, and reports progress. This addresses the frustration every persona shares.

**Acceptance criteria:**
- **Given** a pending payment
  **When** the user checks it
  **Then** the current state and the last update time are shown.
- **Given** a payment debited but not delivered
  **When** the condition is detected
  **Then** a refund is initiated automatically and a case is opened.
- **Given** a case awaiting the biller
  **When** the biller does not respond within the stated window
  **Then** the case escalates and the user is told.

**Edge cases and failure states:**
- Rail reports success and biller reports failure: reconcile against both sources before refunding.
- Duplicate refunds: detect by reference and block the second.
- Case exceeds the service window: escalate to the payment partner and notify the user.

## Reach channels

### FEAT-010: USSD and SMS channel

**Priority:** P0
**Effort:** L
**Depends On:** FEAT-001, FEAT-005

**User story:**
As a feature-phone user, I want to pay and get a receipt by USSD and SMS, so that I do not need a smartphone or data.

**Description:**
Provides a short USSD flow for the highest-frequency payments and receipts, plus SMS delivery of tokens and references. Sessions stay short because session time is billed to airtime, and prompts are localised. This serves Musa.

**Acceptance criteria:**
- **Given** a feature-phone user with no data
  **When** they dial the Sabi code
  **Then** they can start and complete a supported payment and receive the result by SMS.
- **Given** a USSD session that times out mid-payment
  **When** the user starts a new session
  **Then** the pending payment is resumed rather than restarted.
- **Given** a completed payment
  **When** it succeeds
  **Then** the reference and token arrive by SMS.

**Edge cases and failure states:**
- Session timeout before authorisation: no payment is executed.
- Session timeout after authorisation: mark pending and resolve, resuming in the next session.
- SMS delivery fails: retain the message for retry and show the status on the next session.

### FEAT-013: WhatsApp channel

**Priority:** P1
**Effort:** M
**Depends On:** FEAT-005

**User story:**
As a user who already works on WhatsApp, I want to reach Sabi there, so that I do not have to switch apps to pay a bill.

**Description:**
Exposes the assistant through a WhatsApp business channel for discovery, payment, and receipts, using the same account and rails as the app. This serves Ngozi and Tunde.

**Acceptance criteria:**
- **Given** a linked WhatsApp number
  **When** the user sends a payment request
  **Then** Sabi resolves and completes it and returns the receipt in the chat.
- **Given** an unlinked number
  **When** the user messages Sabi
  **Then** linking is offered without requiring a new identity.

**Edge cases and failure states:**
- Channel provider outage: queue inbound messages and notify users of a delay.
- Duplicate request over app and WhatsApp: deduplicate by intent and reference.

## Extensibility and platform

### FEAT-012: Provider service registry for third parties

**Priority:** P0
**Effort:** L
**Depends On:** FEAT-003

**User story:**
As an agency or biller, I want to register a payable service once, so that Sabi can discover and pay it without a bespoke integration.

**Description:**
Lets a provider publish a discoverable capability (what it collects, required inputs, amount source, and result), which Sabi calls on the user's behalf. Registration is reviewed before activation. This supports Ifeoma and keeps adding providers a registration, not a release.

**Acceptance criteria:**
- **Given** a provider capability description
  **When** it is submitted and approved
  **Then** Sabi can discover it and route payments to it.
- **Given** a registered capability
  **When** its schema changes
  **Then** incompatible changes are rejected until reviewed.
- **Given** a capability that fails health checks
  **When** discovered
  **Then** it is marked unavailable and not offered to users.

**Edge cases and failure states:**
- A capability returns a forged or missing result: quarantine it and open a review.
- A provider is deregistered mid-payment: complete or refund the in-flight payment and stop offering it.
- Two providers claim the same service: resolve ownership before activation.

### FEAT-020: Agent marketplace and third-party monetization

**Priority:** P2
**Effort:** L
**Depends On:** FEAT-012

**User story:**
As a service provider, I want to offer a billable capability inside Sabi, so that I can reach users and earn from the agent channel.

**Description:**
Extends the registry with billing for provider capabilities and discovery placement. Deferred until the core payment loop is proven.

**Acceptance criteria:**
- **Given** an approved provider
  **When** it enables a billed capability
  **Then** charges are shown to the user before use.
- **Given** a billable capability
  **When** used
  **Then** the charge is settled to the provider and recorded.

**Edge cases and failure states:**
- Charge dispute: hold the settlement and open a case.
- Provider withdraws: existing subscriptions are honoured until expiry.

## Assistance and compliance

### FEAT-014: Obligation calendar and reminders

**Priority:** P1
**Effort:** M
**Depends On:** FEAT-008

**User story:**
As a user with recurring obligations, I want to be reminded before a deadline, so that I avoid penalties.

**Description:**
Tracks known obligations and deadlines and reminds the user ahead of time, with a one-step route to pay. This serves Ifeoma and Tunde.

**Acceptance criteria:**
- **Given** an obligation with a deadline
  **When** the reminder window is reached
  **Then** the user is reminded on their preferred channel.
- **Given** a reminder
  **When** the user acts
  **Then** the payment starts with the obligation pre-filled.

**Edge cases and failure states:**
- Deadline changes at the agency: update and re-notify.
- Payment already made: suppress the reminder and show the receipt.

### FEAT-023: Account grounding in the assistant

**Priority:** P1
**Effort:** M
**Depends On:** FEAT-003, FEAT-008

**User story:**
As a user, I want the assistant to already know my receipts, my open obligations, and my verification status when I talk to it, so that I do not re-state what Sabi already holds and can be addressed by name.

**Description:**
Each turn, the orchestrator retrieves a bounded account context for the asking user, their display name, verification status, recent receipts, and active obligations, and grounds the answer in those records rather than in model memory. The assistant uses `User.displayName` to address the user familiarly whenever it is present; a user may set it before identity verification, so the name is never tied to KYC. When no display name is held, the assistant addresses the user generically and does not invent one. The context is read-only: it is cited, never used to move money, and every payment still requires explicit confirmed authorisation. Output contract: the orchestrator passes a structured `account_context` object (identity summary, verification status, receipts, obligations) to the model as retrieved, citable content; the answer cites the source record. Fallback: when a source is unavailable, the assistant states it cannot see that part of the account rather than answering from memory.

**Acceptance criteria:**
- **Given** a user with a stored display name, whether or not they are verified
  **When** they send a request
  **Then** the assistant may address them by name and the name comes from the user record.
- **Given** a user with no display name
  **When** they send a request
  **Then** the assistant addresses them generically and does not invent a name.
- **Given** a user asking "did I pay my land use charge?"
  **When** a matching receipt exists
  **Then** the answer cites the receipt record (reference, amount, date) rather than the model's recall.
- **Given** a user with an obligation approaching its deadline
  **When** they ask what is due
  **Then** the assistant lists the obligation from the source record with its deadline.
- **Given** a payment the user has not made
  **When** the assistant is asked whether it was paid
  **Then** the assistant says no matching record was found and does not fabricate one.

**Edge cases and failure states:**
- Account context unavailable: state the part that cannot be seen and continue; never substitute a remembered amount or reference.
- A receipt is disputed or unverified: surface `Receipt.verificationStatus` and the plain reason, not a bare claim of payment.
- Context belongs to another account: never disclose another user's records; return nothing for the mismatched reference.
- A name is stated in the moment by a user who has none on record: use it for that conversation only and do not persist it (persisting it is FEAT-024).

### FEAT-024: Assistant long-term memory

**Priority:** P2
**Effort:** M
**Depends On:** FEAT-003

**User story:**
As a returning user, I want the assistant to remember my corrections and my stated preferences across conversations, so that I do not repeat them every time ("always Ikeja Electric for me").

**Description:**
Persists a bounded set of durable memory per user across sessions: explicit corrections (for example, the right meter number) and stated preferences (for example, a default biller or a preferred channel). Memory is captured only from an explicit user statement or a user-confirmed correction, never inferred silently, and is consent-gated and user-manageable under [nfr](nfr.md) (NFR-040). Memory is advisory: it may pre-fill or narrow a resolution, but it never supplies a fact and never overrides a source record or the confirmation gate. Output contract: the orchestrator passes retrieved `user_preferences` and `memory_items` as untrusted, cited context; a payment field resolved from memory is re-confirmed against the catalogue before use. Fallback: when memory is disabled, ambiguous, or withdrawn, the assistant behaves as a first-time assistant for that user.

**Acceptance criteria:**
- **Given** a user who says "always use Ikeja Electric for my electricity"
  **When** they next request an electricity payment
  **Then** Ikeja Electric is proposed first and the user confirms before payment.
- **Given** a user who corrects a stored detail
  **When** they confirm the correction
  **Then** the memory is updated and the prior value is not used.
- **Given** a user who has not consented to durable memory
  **When** they use the assistant
  **Then** nothing is persisted across sessions and behaviour is first-time.
- **Given** a user who withdraws consent or deletes a memory item
  **When** the request is processed
  **Then** the stored item is hard-deleted under the NFR-040 lifecycle and no longer affects answers.
- **Given** a stored preference that conflicts with the current catalogue
  **When** it is applied
  **Then** the catalogue wins and the assistant says so.

**Edge cases and failure states:**
- Ambiguous correction: do not store; ask which value is right.
- Memory points to a retired or unavailable service: ignore it and offer current results.
- Conflicting preferences: prefer the most recent explicitly stated value and surface the conflict.
- Memory store unavailable: answer first-time; never guess a remembered fact.

### FEAT-017: Receipt verification for third parties

**Priority:** P1
**Effort:** M
**Depends On:** FEAT-008

**User story:**
As a business that must verify payment, I want to confirm that a receipt is genuine, so that I can accept it without a phone call.

**Description:**
Lets an authorised third party verify a receipt or reference against the source of truth, returning a verified or unverified result. This serves Ifeoma.

**Acceptance criteria:**
- **Given** a genuine receipt reference
  **When** verified
  **Then** the result confirms validity, amount, and payee.
- **Given** a revoked or unknown receipt
  **When** verified
  **Then** the result is unverified with a reason.

**Edge cases and failure states:**
- Source unavailable: return pending, not a false negative.
- Rate abuse: throttle and require authentication for bulk checks.

## Diaspora and language

### FEAT-015: Diaspora payer mode

**Priority:** P1
**Effort:** M
**Depends On:** FEAT-005

**User story:**
As a diaspora payer, I want to pay a bill or obligation in Nigeria directly and share the receipt, so that I know it was settled.

**Description:**
Supports paying Nigerian obligations from abroad, including funding from a foreign card or transfer and sharing the receipt with the beneficiary. This serves Tunde.

**Acceptance criteria:**
- **Given** a diaspora user
  **When** they pay a Nigerian bill
  **Then** the biller is settled in Naira and the receipt is available to sender and beneficiary.
- **Given** a completed payment
  **When** the user shares it
  **Then** a link or record is delivered to the beneficiary.

**Edge cases and failure states:**
- FX or settlement delay: show the true status and expected timing.
- Card declined: offer an alternative method without losing the obligation.

### FEAT-016: Nigerian language expansion

**Priority:** P1
**Effort:** XL
**Depends On:** FEAT-003

**User story:**
As a user who prefers Hausa, Yoruba, or Igbo, I want to use Sabi in that language, so that I am not forced to work in English.

**Description:**
Extends intent understanding and responses to Hausa, Yoruba, and Igbo, with accuracy measured per language because these are low-resource for current models. This serves Musa and Ngozi.

**Acceptance criteria:**
- **Given** a request in Hausa
  **When** submitted
  **Then** Sabi resolves the intent and responds in Hausa.
- **Given** a language evaluation set
  **When** a release is tested
  **Then** each language meets the accuracy bar before it is enabled.

**Edge cases and failure states:**
- Dialect variation: fall back to English with a note rather than mis-resolving.
- Mixed-language input: detect the dominant language and confirm the intent.

## Growth and future

### FEAT-018: Shared and group payment pools

**Priority:** P2
**Effort:** M
**Depends On:** FEAT-005

**User story:**
As a member of a group, I want to collect contributions and pay one obligation, so that we settle a shared fee together.

**Description:**
Creates a collection for a known obligation, tracks contributions, and pays on completion. This serves Ngozi.

**Acceptance criteria:**
- **Given** a group obligation
  **When** a pool is created
  **Then** contributors can pay their share and see progress.
- **Given** a fully funded pool
  **When** the target is met
  **Then** the obligation is paid and all contributors see the receipt.

**Edge cases and failure states:**
- Target not met by the deadline: refund contributions.
- A contribution is reversed: reopen the gap and notify the group.

### FEAT-019: Merchant and storefront discovery

**Priority:** P2
**Effort:** L
**Depends On:** FEAT-004

**User story:**
As a user, I want to find and pay a trusted merchant through Sabi, so that everyday commerce is not separate from my payments.

**Description:**
Extends discovery from agencies and billers to merchants and storefronts. Deferred until the payment and receipt loop is proven.

**Acceptance criteria:**
- **Given** a merchant query
  **When** searched
  **Then** matching merchants are listed with a payment route.

**Edge cases and failure states:**
- Fraudulent listing: suspend and review before removal.

### FEAT-021: Community and social layer

**Priority:** P3
**Effort:** XL
**Depends On:** FEAT-012

**User story:**
As a user, I want a place to coordinate payments with people I trust, so that shared money matters are not scattered across chats.

**Description:**
Adds a light community layer around transaction contexts. Deferred; the product does not compete with WhatsApp for general chat.

**Acceptance criteria:**
- **Given** a group with a shared obligation
  **When** members join
  **Then** they see the obligation and its status.

**Edge cases and failure states:**
- Abuse or spam: moderation and reporting apply.

### FEAT-022: Savings and credit products

**Priority:** P3
**Effort:** XL
**Depends On:** FEAT-002

**User story:**
As a verified user, I want to save or access credit, so that I can cover an obligation I cannot pay at once.

**Description:**
Deferred. Lending and deposit-taking require a licence Sabi does not hold, and PSB rules bar lending under that licence.

**Acceptance criteria:**
- **Given** a verified user
  **When** the feature is enabled under an appropriate licence
  **Then** eligibility, terms, and costs are disclosed before acceptance.

**Edge cases and failure states:**
- No licence held: the feature stays disabled.

## Priority summary

| Priority | Count | Feature IDs |
|---|---|---|
| P0 | 12 | FEAT-001, FEAT-002, FEAT-003, FEAT-004, FEAT-005, FEAT-006, FEAT-007, FEAT-008, FEAT-009, FEAT-010, FEAT-011, FEAT-012 |
| P1 | 6 | FEAT-013, FEAT-014, FEAT-015, FEAT-016, FEAT-017, FEAT-023 |
| P2 | 4 | FEAT-018, FEAT-019, FEAT-020, FEAT-024 |
| P3 | 2 | FEAT-021, FEAT-022 |

## Persona coverage

Every persona is served by at least one feature, and the mapping runs both ways.

| Persona | Features serving them |
|---|---|
| Ngozi | FEAT-006, FEAT-007, FEAT-013, FEAT-016, FEAT-018, FEAT-023, FEAT-024 |
| Musa | FEAT-007, FEAT-010, FEAT-016, FEAT-023, FEAT-024 |
| Ifeoma | FEAT-006, FEAT-008, FEAT-012, FEAT-014, FEAT-017, FEAT-020, FEAT-023, FEAT-024 |
| Tunde | FEAT-006, FEAT-008, FEAT-013, FEAT-014, FEAT-015, FEAT-023, FEAT-024 |
