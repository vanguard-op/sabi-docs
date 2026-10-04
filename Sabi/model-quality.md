# Sabi: model quality

## Overview

This document defines the behavior and quality contract for the model-backed capabilities of Sabi, whose AI-core shape is declared in [overview](overview.md). The capabilities in scope are intent resolution (FEAT-003), service and biller discovery ranking (FEAT-004), interface composition (FEAT-011), and Nigerian-language understanding (FEAT-016). Intent resolution and discovery author a user-facing `reply` in natural language alongside the machine-actionable intent, so those capabilities carry both a structured contract and a reply rubric. Latency budgets and cost ceilings are owned by [nfr](nfr.md) and referenced here by `NFR-###`, not restated.

## Model-backed capabilities

| ID | Capability | Feature | Output contract | Fallback |
|---|---|---|---|---|
| MQ-001 | Turn a free-form request into a machine-actionable intent, and author the user-facing reply | FEAT-003 | Model-authored `reply` (rubric) plus a reduced structured intent (schema) | Client renders its prewritten fallback reply; ask follow-up questions across up to three turns; then keyword and search resolution with confirmation |
| MQ-002 | Rank candidate agencies and billers for a query, and author the accompanying reply | FEAT-004 | Model-authored `reply` (rubric) plus a structured ranked list (schema) | Client renders its prewritten fallback reply; present no match and record the query for coverage |
| MQ-003 | Compose a task interface from the client catalogue | FEAT-011 | Declarative UI messages | Render the client fallback form for the same task |
| MQ-004 | Understand Hausa, Yoruba, and Igbo requests | FEAT-016 | Structured schema with detected language | Fall back to English with a note rather than mis-resolve |

## Output and behavior contract

### MQ-001: Intent resolution and reply

**Output:** Two parts. (1) `reply`, the user-facing message the model authors in natural language: the greeting, the clarifying question, the refusal, or the non-resolving reply, in the user's language, nullable only when the client's fallback is used. (2) A reduced, machine-actionable intent the client acts on, with fields: `intent_type` (one of `gov_payment`, `bill_payment`, `discovery`, `status_check`, `receipt_lookup`, `non_command`, `unknown`), `target` (a catalogue service identifier or null), `amount` (integer minor units in `currency`, or null), `currency` (default `NGN`), `reference` (string or null), `constraints` (object), `confidence` (0 to 1), and `detected_language` (one of `en`, `pcm`, `ha`, `yo`, `ig`). The clarifying question and the greeting are part of `reply`, not a separate field. Amounts are integer minor units (kobo for NGN), never decimal major units, and `currency` gives the currency. `detected_language` stays in the structured output: the reply's language is bound into the model context, but the detected or served language is a machine-actionable signal the client and analytics consume and MQ-004 is judged on. The reply wording follows [branding](branding.md)'s Tone of Voice; the structured intent follows this schema.
**Reply boundaries:** The model may phrase the `reply` freely, but every fact in it (an amount, a reference, a service identifier, a collecting body) comes only from the retrieved context, and it never states a fact that is not grounded. The reply contains no executable code and no URLs, and it never discloses another user's data. The reply never substitutes a different specific service for the one the user named: because naming a specific payee asserts ownership, the model must never answer a request for one named service by proposing another of the same kind as if it were that service ("Airtel airtime" is never answered by offering "MTN airtime"; "Ikeja Electric" is never answered by offering "Eko Electric"). Three things are distinguished: (a) substituting a different specific payee is forbidden, because it asserts ownership and risks sending money to the wrong place; (b) a neutral, category-neutral disambiguating question is allowed and encouraged ("which network?", "which provider?", "the collecting body's name?"), because it asks the user to choose rather than asserting a payee; and (c) offering catalogue search is allowed, because the user picks from real catalogue rows.
**Reproducibility:** Reply wording is non-deterministic by design: two equivalent turns may phrase the reply differently and both are correct. The structured intent and the evaluation harness stay pinned. Model version and decoding settings are pinned for evaluation; the structure is fixed and the model receives the earlier turns of the conversation.
**Refusal:** If `confidence` is below the cutoff of **0.70**, the model must ask a follow-up question in `reply` and must not set `intent_type` to a payment type. The conversation is stateful: each turn carries the earlier turns, and the model asks again while the request stays below the cutoff, up to **three** clarifying questions in one conversation. If confidence is still below 0.70 after the third, the model must return `unknown` and author the after-bound reply specified in the non-resolving reply contract below: it states plainly why it cannot proceed, offers catalogue search and human support, invites the user to continue, records the request for coverage, and initiates no payment. A greeting or other non-command message is classified `non_command`, not `unknown`: the model asks no clarifying question, sets no payment field, records nothing for coverage, and authors a conversational reply stating what Sabi can do and inviting a request. If the target cannot be resolved to a catalogue identifier, it must return `unknown` and must not invent an identifier, amount, or reference. The clarifying question itself is category-neutral: it names the general thing the user must supply (an agency or biller name, a reference, or a reference for the obligation), never one category's artefact. It may ask the user to disambiguate among options ("which network?", "which provider?", "the collecting body's name?"), because that asks the user to choose; it must not assert one by proposing a different specific service as a substitute for the one the user named (see the reply-boundary rule above).
**Fallback:** When the model is unavailable or returns unusable output, the client resolves the request through keyword and search over the catalogue and requires explicit confirmation before any payment. A missing or unusable `reply` falls back to the client's prewritten reply for the state, so the user is never shown a blank turn.

> **Contract impact (C-009).** The client contract moves to a `reply` field on the response and a reduced structured intent: `clarifyingQuestion` folds into `reply`, and the intent keeps only the machine-actionable fields. `CONTRACTS.md` is not written here; this note is for the lead to revise C-009.

### MQ-002: Discovery ranking and reply

**Output:** A `reply` the model authors in natural language accompanying the results, plus a ranked list of up to five results, each with `service_id`, `name`, `collecting_body`, `category`, `known_amount` (integer minor units, kobo for NGN, or null), and `confidence`. Includes `no_match` when nothing clears the threshold. The reply obeys the same boundaries as MQ-001: it phrases freely but every fact in it comes only from the retrieved results, it contains no executable code and no URLs, and it never discloses another user's data.
**Reproducibility:** Ranking order and reply wording may vary between equivalent queries; membership of the result set is pinned by the evaluation.
**Refusal:** The model must not fabricate a service or a collecting body. A serving confidence cutoff of **0.70** applies: below it the model returns `no_match` and records the query for coverage. When the catalogue has no match, it does the same. On `no_match` the model authors the non-resolving reply specified in the non-resolving reply contract below, in the register that there is no resolution for that service right now, never a bare statement that the service is unsupported. A missing or unusable reply falls back to the client's prewritten reply for the state.
**Fallback:** Non-model keyword search over catalogue names and aliases, with the same no-invention rule.

### MQ-003: Interface composition

**Output:** A stream of declarative UI messages that create a surface, define a flat component list with identifier references, and update the data model. Components are limited to the client catalogue.
**Reproducibility:** Variation accepted; any stream that validates against the client catalogue and satisfies the task contract is correct.
**Refusal:** The model must not request a component outside the catalogue and must not embed executable code, scripts, or external resource loads in any message. A request outside the catalogue is refused by the client.
**Fallback:** The client renders its static fallback form for the same task, so the task remains completable even if no valid surface arrives.

> **Contract impact (C-012).** The composed surface is the task mechanism for every task and record type, not only task flows. C-012's `task` enum is currently `confirmation|amount_entry|receipt|discovery` and must gain `verification` and `recovery`; and `receipt` must accept a context carrying a resolved record reference from a record lookup, not only a post-payment service payment. `CONTRACTS.md` is not written here; this note is for the lead to revise C-012.

### MQ-004: Nigerian-language understanding

**Output:** The same contract as MQ-001: a model-authored `reply` in the user's language, plus the reduced structured intent including `detected_language` (one of `en`, `pcm`, `ha`, `yo`, `ig`).
**Reproducibility:** Pinned like MQ-001: reply wording varies, the structured intent and the harness stay pinned.
**Refusal:** A serving confidence cutoff of **0.70** applies: below it the model falls back to English with a note and confirms the interpreted intent before any payment. When input mixes languages such that the dominant language is unclear, the model must likewise confirm the interpreted intent before payment and may switch to English with a note.
**Fallback:** English-only understanding, with the user informed that their language is not yet supported for this request.

## Quality bar

Each capability with a judged output carries a measured bar. A threshold applies only with the dataset and judge stated here. A capability that authors a reply carries two: a structured-intent bar and a reply-rubric bar.

| ID | Capability | Dataset | Metric | Threshold | Judge | Gate |
|---|---|---|---|---|---|---|
| MQ-001 | Intent resolution | 1,000 labelled Nigerian payment utterances in English and Pidgin, including multi-turn clarification sequences, refreshed monthly | Top-1 intent accuracy | 0.92 | Human-labelled golden set | Every release |
| MQ-001R | Intent-resolution reply | The same 1,000-utterance set, scored on the model's authored replies | Reply rubric pass rate, across the dimensions below | 0.90 | Human rubric review on a sampled subset | Every release |
| MQ-002 | Discovery ranking | 500 labelled queries to agencies and billers | Correct result within top 3 | 0.90 | Human-verified labels | Every release |
| MQ-002R | Discovery reply | The same 500-query set, scored on the model's authored replies | Reply rubric pass rate, across the dimensions below | 0.90 | Human rubric review on a sampled subset | Every release |
| MQ-003 | Interface composition | 300 task flows across payment types and channels | Schema-valid output; rubric pass | 0.95 schema; 0.90 rubric | Automated catalogue validation plus rubric review | Every release |
| MQ-004 | Nigerian-language understanding | 500 utterances per language, labelled by native speakers | Top-1 intent accuracy per language | 0.85 per language | Native-speaker labelled set per language | Per language, before that language is enabled |

A payment capability is held to a higher bar than a discovery capability because a wrong intent moves money to the wrong place. MQ-001 carries the highest threshold for that reason. Each bar has a stated growth path of one point per quarter until it reaches 0.97, pending dataset expansion.

**Reply rubric.** A model-authored reply is judged by rubric, not by exact match, because the wording is non-deterministic by design. A reply passes when it satisfies every dimension:

- **Grounded.** Every fact in the reply (an amount, a reference, a service identifier, or a collecting body) is present in the retrieved context, and the reply states none that is not grounded.
- **Tone.** The register follows [branding](branding.md): plain, human, calm, in the user's language; specific, warm, and forward-looking; never apologetic to the point of blame and never framed as a limit of Sabi.
- **Neutral.** The reply and any clarifying question are category-neutral: they never assume or prefer a service category, a biller, an agency, or an identifier type. The reply never substitutes a different specific service for the one the user named: it does not answer a request for one named service by proposing another of the same kind ("Airtel airtime" is never answered by offering "MTN airtime"; "Ikeja Electric" is never answered by offering "Eko Electric"). A neutral disambiguating question ("which network?", "which provider?", "the collecting body's name?") and catalogue search are allowed, because both leave the choice to the user rather than asserting a payee.
- **Actionable.** Where the request did not resolve, the reply states the honest reason, names a concrete next step, and invites the user to continue; it is not a full stop.
- **Safe.** The reply contains no executable code and no URLs, and it never discloses another user's data.

The rubric pass rate is measured on a human-reviewed sample of the dataset's replies, alongside the structured-intent bar on the same set.

Separately from the accuracy bars above, a single serving confidence cutoff of **0.70** gates every judged capability that emits a confidence. The cutoff and its per-capability behavior are:

- **MQ-001:** below 0.70 the model asks a follow-up question and does not resolve a payment intent; it may ask up to three clarifying questions in one conversation, each turn carrying the earlier turns; if confidence is still below 0.70 after the third, it states it cannot resolve, offers search and human support, records the request for coverage, and does not initiate a payment (see the MQ-001 Refusal rule). A non-command message is classified `non_command`, is never a clarification turn, and is never recorded for coverage.
- **MQ-002:** below 0.70 the model returns `no_match` and records the query for coverage.
- **MQ-004:** below 0.70 the model falls back to English with a note and confirms the interpreted intent before any payment.

MQ-003 carries no serving cutoff: it is judged by schema validity and rubric, not by a confidence value.

## Grounding, safety, and refusal behavior

Facts about agencies, billers, amounts, references, and receipts must come from the catalogue or the payment source, never from model memory. The model must not emit an amount, a reference number, or a service identifier that is not present in retrieved data. This rule binds the model-authored `reply` as much as the structured intent: the model may phrase the reply freely, but every fact it states (an amount, a reference, a service identifier, a collecting body) comes only from the retrieved context, and it states none that is not grounded. Where an answer depends on retrieved content, the answer cites the source record.

The reply is additionally constrained as free text: it carries no executable code and no URLs, it follows the tone set in [branding](branding.md), it stays category-neutral and never substitutes a different specific service for the one the user named (see the reply-boundary rule in MQ-001), and it never discloses another user's data.

Account grounding (FEAT-023) and durable memory (FEAT-024) are retrieved content and obey the same rule. The assistant's account context (the user's display name, verification status, receipts, and obligations) is read from the source records and cited, never recalled; naming the user is a read of `User.displayName` when present, and no name is invented when it is absent. Persisted preferences and corrections are advisory only: memory may narrow or pre-fill a resolution, but it never supplies a fact and never overrides the catalogue, a receipt, or the confirmation gate. A payment field resolved from memory must be re-confirmed against the catalogue before any payment, and a preference that conflicts with the catalogue loses to the catalogue. Memory is untrusted like all retrieved content: it cannot override the payment policy, and it is subject to the same consent, retention, and deletion rules as the rest of the personal data (NFR-040). These are grounding constraints on the existing capabilities, not a new judged capability, so no new quality bar is added.

User input and retrieved content are untrusted. Instructions found inside retrieved content never override the payment policy, and no payment is executed without an explicit, signed authorisation bound to the amount and payee (see `NFR-027` in [nfr](nfr.md)). The model must never emit executable code, must never request a component outside the client catalogue, and must never disclose another user's data.

When the model is unsure, it asks or declines. It does not guess a payee, an amount, or a service. This refusal behavior is a product decision and is enforced before execution, not left to the implementer.

### Non-resolving reply contract

Three states reach the user without a task to carry: the request stays unresolved after the clarification bound (MQ-001), the catalogue has no match (MQ-002, FEAT-004), and the message is a greeting or other non-command (MQ-001, FEAT-003). The model authors the reply for each of these states in natural language, and every such reply obeys one contract. The client holds a prewritten reply for each state as the fallback when no usable model reply arrives; it is the safety net, not the source of truth.

**Shape.** A non-resolving reply states, in one or two plain sentences, the honest reason the request did not move forward, then a concrete next step the user can act on now, then an invitation to continue in the same conversation. It names what the user can supply next (the collecting body's name, a reference, a corrected detail) so the next turn can succeed. It is not a full stop.

**Register.** The reason is given about the request, not about the platform: the register is that there is no resolution for that service right now, true of the request and forward-looking, rather than a fixed "I searched and found no match". Wording that frames the omission as a limit of Sabi, such as "not supported", "not available", or "coming soon", is not used.

**Tone.** The reply is specific, warm, and forward-looking. It is never apologetic to the point of blame. The register follows [branding](branding.md): plain, human, calm, in the user's language.

**Neutrality.** The reply and any clarifying question are category-neutral. They never assume or prefer a service category, a biller, an agency, or an identifier type, and they never present one category's artefact as the default thing to supply. The user supplies a reference, an account number, or the collecting body's name as the case requires; electricity and a meter number are one case among many, not the template. The reply never substitutes a different specific service for the one the user named: it does not answer a request for one named service by proposing another of the same kind ("Airtel airtime" is never answered by offering "MTN airtime"; "Ikeja Electric" is never answered by offering "Eko Electric"), because naming a specific payee asserts ownership and risks sending money to the wrong place. Asking the user to disambiguate among options ("which network?", "which provider?", "the collecting body's name?") is allowed and encouraged, and offering catalogue search is allowed: both leave the choice to the user rather than asserting a payee.

**Grounding.** Every fact in the reply comes only from the retrieved context, and the reply states none that is not grounded (see the grounding rule above).

The client's prewritten fallback copy for each state, and the tone and neutrality rules the model's reply must satisfy, are set in [Home](screens/03-home.md); the reply wording itself is authored by the model, not fixed as a client string.

## Model version and fallback

**Pinned version:** The primary path is `gemini-3.1-pro-preview`, and high-volume intent resolution uses the lower-cost `gemini-3.1-flash-lite`. Decoding is pinned per capability for the structured intent and the evaluation harness: temperature `0.0` and top_p `1.0`. Reply wording is non-deterministic by design and is judged by rubric rather than pinned by decoding. The quality bar is measured against this pinned build. The primary is a preview model, so the deprecation policy below applies to it.
**Upgrade policy:** A version change requires a full evaluation run against every bar above; a capability that falls below threshold does not ship until it is retrained, re-prompted, or the version is rolled back.
**Provider failure:** When the provider is unavailable, the product degrades to keyword and search resolution with mandatory confirmation, and payment execution pauses rather than proceeding without grounding.
**Deprecation:** When the pinned model is retired, the replacement is evaluated against the same datasets before cutover, and the fallback path remains available until the new model clears every bar.

## Measurement and verification

| ID | Verified by | When | Owner |
|---|---|---|---|
| MQ-001 | Evaluation run against the 1,000-utterance golden set | Every release | AI Engineering |
| MQ-001R | Reply rubric review on a sampled subset of the same set | Every release | AI Engineering |
| MQ-002 | Evaluation run against the 500-query set | Every release | AI Engineering |
| MQ-002R | Reply rubric review on a sampled subset of the same set | Every release | AI Engineering |
| MQ-003 | Catalogue schema validation plus rubric review of 300 flows | Every release | AI Engineering and Client |
| MQ-004 | Per-language evaluation with native-speaker labels | Per language, before enablement | AI Engineering |
| All | Live quality telemetry and drift review against the bars | Monthly | Operations |
