# Sabi: model quality

## Overview

This document defines the behavior and quality contract for the model-backed capabilities of Sabi, whose AI-core shape is declared in [overview](overview.md). The capabilities in scope are intent resolution (FEAT-003), service and biller discovery ranking (FEAT-004), interface composition (FEAT-011), and Nigerian-language understanding (FEAT-016). Latency budgets and cost ceilings are owned by [nfr](nfr.md) and referenced here by `NFR-###`, not restated.

## Model-backed capabilities

| ID | Capability | Feature | Output contract | Fallback |
|---|---|---|---|---|
| MQ-001 | Turn a free-form request into a structured payment or discovery intent | FEAT-003 | Structured schema | Ask follow-up questions across up to three turns; then keyword and search resolution with confirmation |
| MQ-002 | Rank candidate agencies and billers for a query | FEAT-004 | Structured ranked list | Present no match and record the query for coverage |
| MQ-003 | Compose a task interface from the client catalogue | FEAT-011 | Declarative UI messages | Render the client fallback form for the same task |
| MQ-004 | Understand Hausa, Yoruba, and Igbo requests | FEAT-016 | Structured schema with detected language | Fall back to English with a note rather than mis-resolve |

## Output and behavior contract

### MQ-001: Intent resolution

**Output:** Structured object with fields: `intent_type` (one of `gov_payment`, `bill_payment`, `discovery`, `status_check`, `receipt_lookup`, `non_command`, `unknown`), `target` (a catalogue service identifier or null), `amount` (integer minor units in `currency`, or null), `currency` (default `NGN`), `reference` (string or null), `constraints` (object), `confidence` (0 to 1), `language`, and `clarifying_question` (string or null). Amounts are integer minor units (kobo for NGN), never decimal major units, and `currency` gives the currency.
**Reproducibility:** Model version and decoding settings pinned for evaluation; structure is fixed and the model receives the earlier turns of the conversation, so phrasing of clarifications may vary across turns.
**Refusal:** If `confidence` is below the cutoff of **0.70**, the model must ask a follow-up question and must not set `intent_type` to a payment type. The conversation is stateful: each turn carries the earlier turns, and the model asks again while the request stays below the cutoff, up to **three** clarifying questions in one conversation. If confidence is still below 0.70 after the third, the model must state that it cannot resolve the request, offer catalogue search and human support, record the request for coverage, and must not initiate a payment. A greeting or other non-command message is classified `non_command`, not `unknown`: the model asks no clarifying question, sets no payment field, and records nothing for coverage, and the client renders a localised conversational reply from that intent rather than from model text. If the target cannot be resolved to a catalogue identifier, it must return `unknown` and must not invent an identifier, amount, or reference.
**Fallback:** When the model is unavailable or returns unusable output, the client resolves the request through keyword and search over the catalogue and requires explicit confirmation before any payment.

### MQ-002: Discovery ranking

**Output:** Ranked list of up to five results, each with `service_id`, `name`, `collecting_body`, `category`, `known_amount` (integer minor units, kobo for NGN, or null), and `confidence`. Includes `no_match` when nothing clears the threshold.
**Reproducibility:** Ranking order may vary between equivalent queries; membership of the result set is pinned by the evaluation.
**Refusal:** The model must not fabricate a service or a collecting body. A serving confidence cutoff of **0.70** applies: below it the model returns `no_match` and records the query for coverage. When the catalogue has no match, it does the same.
**Fallback:** Non-model keyword search over catalogue names and aliases, with the same no-invention rule.

### MQ-003: Interface composition

**Output:** A stream of declarative UI messages that create a surface, define a flat component list with identifier references, and update the data model. Components are limited to the client catalogue.
**Reproducibility:** Variation accepted; any stream that validates against the client catalogue and satisfies the task contract is correct.
**Refusal:** The model must not request a component outside the catalogue and must not embed executable code, scripts, or external resource loads in any message. A request outside the catalogue is refused by the client.
**Fallback:** The client renders its static fallback form for the same task, so the task remains completable even if no valid surface arrives.

### MQ-004: Nigerian-language understanding

**Output:** The same structured schema as MQ-001, plus `detected_language` (one of `en`, `pcm`, `ha`, `yo`, `ig`).
**Reproducibility:** Pinned like MQ-001.
**Refusal:** A serving confidence cutoff of **0.70** applies: below it the model falls back to English with a note and confirms the interpreted intent before any payment. When input mixes languages such that the dominant language is unclear, the model must likewise confirm the interpreted intent before payment and may switch to English with a note.
**Fallback:** English-only understanding, with the user informed that their language is not yet supported for this request.

## Quality bar

Each capability with a judged output carries a measured bar. A threshold applies only with the dataset and judge stated here.

| ID | Capability | Dataset | Metric | Threshold | Judge | Gate |
|---|---|---|---|---|---|---|
| MQ-001 | Intent resolution | 1,000 labelled Nigerian payment utterances in English and Pidgin, including multi-turn clarification sequences, refreshed monthly | Top-1 intent accuracy | 0.92 | Human-labelled golden set | Every release |
| MQ-002 | Discovery ranking | 500 labelled queries to agencies and billers | Correct result within top 3 | 0.90 | Human-verified labels | Every release |
| MQ-003 | Interface composition | 300 task flows across payment types and channels | Schema-valid output; rubric pass | 0.95 schema; 0.90 rubric | Automated catalogue validation plus rubric review | Every release |
| MQ-004 | Nigerian-language understanding | 500 utterances per language, labelled by native speakers | Top-1 intent accuracy per language | 0.85 per language | Native-speaker labelled set per language | Per language, before that language is enabled |

A payment capability is held to a higher bar than a discovery capability because a wrong intent moves money to the wrong place. MQ-001 carries the highest threshold for that reason. Each bar has a stated growth path of one point per quarter until it reaches 0.97, pending dataset expansion.

Separately from the accuracy bars above, a single serving confidence cutoff of **0.70** gates every judged capability that emits a confidence. The cutoff and its per-capability behavior are:

- **MQ-001:** below 0.70 the model asks a follow-up question and does not resolve a payment intent; it may ask up to three clarifying questions in one conversation, each turn carrying the earlier turns; if confidence is still below 0.70 after the third, it states it cannot resolve, offers search and human support, records the request for coverage, and does not initiate a payment (see the MQ-001 Refusal rule). A non-command message is classified `non_command`, is never a clarification turn, and is never recorded for coverage.
- **MQ-002:** below 0.70 the model returns `no_match` and records the query for coverage.
- **MQ-004:** below 0.70 the model falls back to English with a note and confirms the interpreted intent before any payment.

MQ-003 carries no serving cutoff: it is judged by schema validity and rubric, not by a confidence value.

## Grounding, safety, and refusal behavior

Facts about agencies, billers, amounts, references, and receipts must come from the catalogue or the payment source, never from model memory. The model must not emit an amount, a reference number, or a service identifier that is not present in retrieved data. Where an answer depends on retrieved content, the answer cites the source record.

User input and retrieved content are untrusted. Instructions found inside retrieved content never override the payment policy, and no payment is executed without an explicit, signed authorisation bound to the amount and payee (see `NFR-027` in [nfr](nfr.md)). The model must never emit executable code, must never request a component outside the client catalogue, and must never disclose another user's data.

When the model is unsure, it asks or declines. It does not guess a payee, an amount, or a service. This refusal behavior is a product decision and is enforced before execution, not left to the implementer.

## Model version and fallback

**Pinned version:** The primary path is `gemini-3.1-pro-preview`, and high-volume intent resolution uses the lower-cost `gemini-3.1-flash-lite`. Decoding is pinned per capability: temperature `0.0` and top_p `1.0`. The quality bar is measured against this pinned build. The primary is a preview model, so the deprecation policy below applies to it.
**Upgrade policy:** A version change requires a full evaluation run against every bar above; a capability that falls below threshold does not ship until it is retrained, re-prompted, or the version is rolled back.
**Provider failure:** When the provider is unavailable, the product degrades to keyword and search resolution with mandatory confirmation, and payment execution pauses rather than proceeding without grounding.
**Deprecation:** When the pinned model is retired, the replacement is evaluated against the same datasets before cutover, and the fallback path remains available until the new model clears every bar.

## Measurement and verification

| ID | Verified by | When | Owner |
|---|---|---|---|
| MQ-001 | Evaluation run against the 1,000-utterance golden set | Every release | AI Engineering |
| MQ-002 | Evaluation run against the 500-query set | Every release | AI Engineering |
| MQ-003 | Catalogue schema validation plus rubric review of 300 flows | Every release | AI Engineering and Client |
| MQ-004 | Per-language evaluation with native-speaker labels | Per language, before enablement | AI Engineering |
| All | Live quality telemetry and drift review against the bars | Monthly | Operations |
