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
Interprets a free-form request (for example, "pay my land use charge" or "buy five thousand MTN") and turns it into a structured intent with the target, the amount if stated, and any constraints. Handles English and Nigerian Pidgin at launch, including common payment and government vocabulary. Output contract: structured JSON with intent type, target candidate list, amount, currency, and confidence; the client renders a confirmation from this structure, not from raw text. Fallback: when confidence is below threshold, Sabi asks one clarifying question with the closest candidates, and if it still cannot resolve, it says so and offers search or human support rather than guessing.

**Acceptance criteria:**
- **Given** a Pidgin request such as "I wan pay my electricity"
  **When** submitted
  **Then** Sabi returns a structured bill-payment intent with the electricity biller category as the target.
- **Given** a request naming a levy Sabi knows
  **When** submitted
  **Then** Sabi returns the responsible agency and asks for confirmation before payment.
- **Given** a request Sabi cannot map
  **When** confidence is below threshold
  **Then** Sabi asks one clarifying question instead of initiating a payment.

**Edge cases and failure states:**
- Ambiguous target matches several agencies: present the top candidates and require a choice.
- Model provider unavailable: fall back to keyword and search-based resolution and label the result as unresolved until confirmed.
- A request that names another person's account: refuse and explain.

### FEAT-004: Service, agency, and biller discovery

**Priority:** P0
**Effort:** L
**Depends On:** FEAT-003

**User story:**
As a user, I want to search for the right agency, levy, or biller and see what I owe, so that I pay the correct place the first time.

**Description:**
Resolves a query to a real agency, obligation, or biller from a maintained catalogue, and shows what is known about the obligation (amount where an agency or biller provides it, reference, deadline). Search accepts text, and results show the collecting body and any charge. This serves Ngozi, Ifeoma, and Tunde. Output contract: a ranked list of structured service records with identifiers, collecting body, and known amount; the client renders selectable results. Fallback: when the catalogue has no match, Sabi says the service is not yet covered and records the request rather than fabricating a result.

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

**Edge cases and failure states:**
- Duplicate or renamed agencies: prefer the current registered name and note the former name.
- No results: record the query for coverage planning and offer human support.
- Stale catalogue data: mark records past their refresh window and warn before payment.

### FEAT-011: Agent-composed interface for task flows

**Priority:** P0
**Effort:** L
**Depends On:** FEAT-003

**User story:**
As a user, I want the assistant to show me exactly the form, confirmation, or receipt I need for a task, so that a new provider works without me updating the app.

**Description:**
Renders task interfaces (confirmation, amount entry, receipt) from the client's approved component catalogue, generated by the agent rather than shipped per provider. The client owns styling and security; the agent may only request components the client exposes. This supports all personas by removing per-provider screens. Output contract: a stream of declarative UI messages (surface creation, components, data model) conforming to the client catalogue; the client renders natively. Fallback: if a surface fails validation or a component is unknown, the client renders a text and form fallback for the same task, and the task remains completable.

**Acceptance criteria:**
- **Given** a service added after the app release
  **When** the user starts a payment to it
  **Then** the confirmation screen is composed from the client catalogue with no app update.
- **Given** an agent UI message requesting an unknown component
  **When** received
  **Then** the client refuses that component and renders the fallback form.
- **Given** a composed receipt surface
  **When** displayed
  **Then** it uses the client's components and styling.

**Edge cases and failure states:**
- Malformed UI message: discard it and render the fallback without losing the task.
- Partial data model: render placeholders and continue when the missing data arrives.
- Component requests a permission the client does not grant: refuse and continue with approved components.

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
Covers federal payments to ministries, departments, and agencies, including taxes, levies, fees, and fines. On success, retrieves the Federal Treasury e-Receipt (FTeR) from the central platform and stores it against the obligation. This serves Ngozi, Ifeoma, and Tunde.

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
- Source receipt cannot be verified: mark the receipt as unverified and state why.
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
| P1 | 5 | FEAT-013, FEAT-014, FEAT-015, FEAT-016, FEAT-017 |
| P2 | 3 | FEAT-018, FEAT-019, FEAT-020 |
| P3 | 2 | FEAT-021, FEAT-022 |

## Persona coverage

Every persona is served by at least one feature, and the mapping runs both ways.

| Persona | Features serving them |
|---|---|
| Ngozi | FEAT-006, FEAT-007, FEAT-013, FEAT-016, FEAT-018 |
| Musa | FEAT-007, FEAT-010, FEAT-016 |
| Ifeoma | FEAT-006, FEAT-008, FEAT-012, FEAT-014, FEAT-017, FEAT-020 |
| Tunde | FEAT-006, FEAT-008, FEAT-013, FEAT-014, FEAT-015 |
