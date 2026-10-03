# Sabi: schema

## Overview

The model is flat and referential, targeting a relational primary database with a separate relational store for payment records. Entities are joined by foreign-key scalars rather than nested objects, and each entity maps cleanly onto tables. All monetary amounts are integer minor units (kobo for NGN), not decimal major units, and each amount field is paired with a `currency` field that gives the currency; integer minor units keep every amount exact and remove floating-point rounding error from a payment. Personal data fields carry an `x-personal-data` annotation so the retention and deletion rules in [nfr](nfr.md) can locate them. One deliberate denormalization exists: the payment record stores the amount, currency, and payee at execution time so a receipt never changes if a catalogue entry is later edited. Payment events record each state change append-only for the case timeline and audit trail, and payment methods store only a masked identifier and an optional processor token, never a full account or card number.

## Entities

### User

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/sabi/user.schema.json",
  "title": "User",
  "description": "A person with an account, whether reached by app or by channel.",
  "type": "object",
  "properties": {
    "id": { "type": "string", "description": "Stable identifier for the user." },
    "displayName": { "type": ["string", "null"], "description": "Name shown in the interface.", "x-personal-data": true },
    "locale": { "type": "string", "enum": ["en", "pcm", "ha", "yo", "ig"], "description": "Preferred language." },
    "preferredChannel": { "type": "string", "enum": ["app", "ussd", "sms", "whatsapp"], "description": "Channel used for reminders and results." },
    "kycStatus": { "type": "string", "enum": ["unverified", "pending", "verified", "rejected"], "description": "Identity verification state that gates regulated payments." },
    "accountStatus": { "type": "string", "enum": ["active", "suspended", "closed"], "description": "Lifecycle state of the account." },
    "createdAt": { "type": "string", "format": "date-time", "description": "Creation time (UTC)." },
    "updatedAt": { "type": "string", "format": "date-time", "description": "Last update time (UTC)." }
  },
  "required": ["id", "locale", "preferredChannel", "kycStatus", "accountStatus", "createdAt", "updatedAt"]
}
```

### AuthCredential

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/sabi/auth-credential.schema.json",
  "title": "AuthCredential",
  "description": "A login credential for a user, either a passkey or a recovery code.",
  "type": "object",
  "properties": {
    "id": { "type": "string", "description": "Stable identifier for the credential." },
    "userId": { "type": "string", "description": "References the User who owns this credential." },
    "type": { "type": "string", "enum": ["passkey", "recovery_code"], "description": "Kind of credential." },
    "credentialId": { "type": ["string", "null"], "description": "Authenticator-issued passkey credential id, required to look up a passkey during login; null for a recovery code. A non-secret identifier, not personal data." },
    "publicKey": { "type": ["string", "null"], "description": "Public half of a passkey; null for a recovery code.", "x-personal-data": true },
    "secretHash": { "type": ["string", "null"], "description": "One-way hash of a recovery code; null for a passkey.", "x-personal-data": true },
    "lookupDigest": { "type": ["string", "null"], "description": "Non-secret, peppered digest of a recovery code used to locate the credential in O(1) before constant-time verification; null for a passkey. Not a secret and not reversible to the code, so not personal data." },
    "signatureCounter": { "type": ["integer", "null"], "description": "Passkey signature counter; null for a recovery code." },
    "status": { "type": "string", "enum": ["active", "revoked", "used"], "description": "Credential lifecycle state." },
    "createdAt": { "type": "string", "format": "date-time", "description": "Creation time (UTC)." }
  },
  "required": ["id", "userId", "type", "status", "createdAt"]
}
```

### ChannelLink

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/sabi/channel-link.schema.json",
  "title": "ChannelLink",
  "description": "A channel identity attached to one user account.",
  "type": "object",
  "properties": {
    "id": { "type": "string", "description": "Stable identifier for the link." },
    "userId": { "type": "string", "description": "References the User the channel is attached to." },
    "channel": { "type": "string", "enum": ["ussd", "sms", "whatsapp"], "description": "Channel type." },
    "address": { "type": "string", "description": "Channel address, such as a phone number.", "x-personal-data": true },
    "verifiedAt": { "type": ["string", "null"], "format": "date-time", "description": "When the link was confirmed; null if unverified." },
    "createdAt": { "type": "string", "format": "date-time", "description": "Creation time (UTC)." }
  },
  "required": ["id", "userId", "channel", "address", "createdAt"]
}
```

### IdentityVerification

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/sabi/identity-verification.schema.json",
  "title": "IdentityVerification",
  "description": "A record of an identity check for a user.",
  "type": "object",
  "properties": {
    "id": { "type": "string", "description": "Stable identifier for the verification." },
    "userId": { "type": "string", "description": "References the User being verified." },
    "method": { "type": "string", "enum": ["nin", "bvn"], "description": "Identity method used." },
    "identifierMasked": { "type": "string", "description": "Masked identity number, never the full value.", "x-personal-data": true },
    "status": { "type": "string", "enum": ["pending", "verified", "failed", "corrected"], "description": "Outcome of the check." },
    "providerReference": { "type": ["string", "null"], "description": "Reference returned by the identity provider." },
    "verifiedAt": { "type": ["string", "null"], "format": "date-time", "description": "When the check succeeded; null otherwise." },
    "createdAt": { "type": "string", "format": "date-time", "description": "Creation time (UTC)." }
  },
  "required": ["id", "userId", "method", "identifierMasked", "status", "createdAt"]
}
```

### Service

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/sabi/service.schema.json",
  "title": "Service",
  "description": "A payable service: a government obligation, a biller, or a merchant.",
  "type": "object",
  "properties": {
    "id": { "type": "string", "description": "Stable identifier for the service." },
    "type": { "type": "string", "enum": ["government", "biller", "merchant"], "description": "Category of service." },
    "name": { "type": "string", "description": "Current registered name.", "maxLength": 200 },
    "collectingBody": { "type": "string", "description": "Agency, biller, or merchant that collects the money." },
    "category": { "type": "string", "description": "Grouping used in discovery, such as tax, levy, electricity, airtime." },
    "aliases": { "type": "array", "items": { "type": "string" }, "description": "Alternative and former names used in search." },
    "providerId": { "type": ["string", "null"], "description": "References the Provider that fulfils payments for this service." },
    "capabilityId": { "type": ["string", "null"], "description": "References the Capability used to pay this service." },
    "knownAmount": { "type": ["integer", "null"], "minimum": 0, "description": "Amount in minor units (kobo for NGN) when the source provides one; null when confirmed at payment. The currency field gives the currency." },
    "currency": { "type": "string", "description": "Currency code.", "default": "NGN" },
    "status": { "type": "string", "enum": ["active", "unavailable", "retired"], "description": "Availability state." },
    "refreshedAt": { "type": ["string", "null"], "format": "date-time", "description": "When the record was last refreshed from source." },
    "createdAt": { "type": "string", "format": "date-time", "description": "Creation time (UTC)." }
  },
  "required": ["id", "type", "name", "collectingBody", "category", "currency", "status", "createdAt"]
}
```

### Provider

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/sabi/provider.schema.json",
  "title": "Provider",
  "description": "An external organisation that registers one or more payable capabilities.",
  "type": "object",
  "properties": {
    "id": { "type": "string", "description": "Stable identifier for the provider." },
    "name": { "type": "string", "description": "Registered provider name, unique case-insensitively across providers." },
    "type": { "type": "string", "enum": ["agency", "biller", "merchant", "aggregator"], "description": "Kind of provider." },
    "contactEmail": { "type": ["string", "null"], "format": "email", "description": "Operational contact for the provider.", "x-personal-data": true },
    "status": { "type": "string", "enum": ["pending", "active", "suspended", "deregistered"], "description": "Provider lifecycle state." },
    "createdAt": { "type": "string", "format": "date-time", "description": "Creation time (UTC)." }
  },
  "required": ["id", "name", "type", "status", "createdAt"]
}
```

### Capability

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/sabi/capability.schema.json",
  "title": "Capability",
  "description": "A discoverable ability a provider exposes, called by the agent on a user's behalf.",
  "type": "object",
  "properties": {
    "id": { "type": "string", "description": "Stable identifier for the capability." },
    "providerId": { "type": "string", "description": "References the Provider that exposes this capability." },
    "serviceId": { "type": ["string", "null"], "description": "References the Service this capability pays, when it maps to one." },
    "name": { "type": "string", "description": "Capability name." },
    "inputSchemaRef": { "type": "string", "description": "Reference to the schema describing required inputs." },
    "amountSource": { "type": "string", "enum": ["fixed", "user_entered", "provider_query"], "description": "How the amount is determined." },
    "billable": { "type": "boolean", "description": "Whether the provider charges for using this capability." },
    "priceAmount": { "type": ["integer", "null"], "minimum": 0, "description": "Charge per use in minor units (kobo for NGN) when billable; null otherwise. The priceCurrency field gives the currency." },
    "priceCurrency": { "type": ["string", "null"], "description": "Currency of the charge." },
    "schemaVersion": { "type": "string", "description": "Version of the capability input schema." },
    "healthStatus": { "type": "string", "enum": ["healthy", "degraded", "unavailable"], "description": "Latest health result." },
    "status": { "type": "string", "enum": ["pending", "active", "quarantined", "withdrawn"], "description": "Capability lifecycle state." },
    "createdAt": { "type": "string", "format": "date-time", "description": "Creation time (UTC)." }
  },
  "required": ["id", "providerId", "name", "inputSchemaRef", "amountSource", "schemaVersion", "healthStatus", "status", "createdAt"]
}
```

### CapabilityReview

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/sabi/capability-review.schema.json",
  "title": "CapabilityReview",
  "description": "A recorded provider or capability review decision, so approvals and quarantine are queryable rather than audit-only.",
  "type": "object",
  "properties": {
    "id": { "type": "string", "description": "Stable identifier for the review." },
    "providerId": { "type": "string", "description": "References the Provider under review." },
    "capabilityId": { "type": ["string", "null"], "description": "References the Capability under review; null for a provider-level decision." },
    "reviewerRole": { "type": "string", "description": "Role responsible for the decision." },
    "decision": { "type": "string", "enum": ["approve", "reject", "quarantine", "deregister"], "description": "Outcome of the review." },
    "reason": { "type": "string", "description": "Stated reason for the decision." },
    "schemaVersion": { "type": "string", "description": "Version of the capability input schema at review time." },
    "createdAt": { "type": "string", "format": "date-time", "description": "When the decision was recorded (UTC)." }
  },
  "required": ["id", "providerId", "reviewerRole", "decision", "reason", "schemaVersion", "createdAt"]
}
```

### PaymentMandate

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/sabi/payment-mandate.schema.json",
  "title": "PaymentMandate",
  "description": "A signed authorisation binding a user to a specific amount and payee.",
  "type": "object",
  "properties": {
    "id": { "type": "string", "description": "Stable identifier for the mandate." },
    "userId": { "type": "string", "description": "References the User who authorised the payment." },
    "capabilityId": { "type": ["string", "null"], "description": "References the Capability being paid, when used." },
    "amount": { "type": "integer", "description": "Authorised amount in minor units (kobo for NGN). The currency field gives the currency.", "minimum": 0 },
    "currency": { "type": "string", "description": "Currency code.", "default": "NGN" },
    "payee": { "type": "string", "description": "Authorised payee." },
    "payeeReference": { "type": ["string", "null"], "description": "Payee reference, such as an obligation number." },
    "nonce": { "type": "string", "description": "Single-use value preventing replay." },
    "signature": { "type": "string", "description": "Signature binding the user, amount, and payee." },
    "status": { "type": "string", "enum": ["issued", "consumed", "expired", "revoked"], "description": "Mandate lifecycle state." },
    "expiresAt": { "type": "string", "format": "date-time", "description": "Expiry time (UTC)." },
    "createdAt": { "type": "string", "format": "date-time", "description": "Creation time (UTC)." }
  },
  "required": ["id", "userId", "amount", "currency", "payee", "nonce", "signature", "status", "expiresAt", "createdAt"]
}
```

### PaymentMethod

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/sabi/payment-method.schema.json",
  "title": "PaymentMethod",
  "description": "A funding source a user can authorise a payment from.",
  "type": "object",
  "properties": {
    "id": { "type": "string", "description": "Stable identifier for the method." },
    "userId": { "type": "string", "description": "References the User who owns the method." },
    "type": { "type": "string", "enum": ["bank_account", "wallet", "card"], "description": "Kind of funding source." },
    "label": { "type": "string", "description": "Display name, such as a bank or card nickname." },
    "maskedIdentifier": { "type": "string", "description": "Masked account or card identifier for display; never the full value.", "x-personal-data": true },
    "processorToken": { "type": ["string", "null"], "description": "Token from the payment processor; null for account and wallet methods. No card number is ever stored.", "x-personal-data": true },
    "isDefault": { "type": "boolean", "description": "Whether this is the user's default method.", "default": false },
    "status": { "type": "string", "enum": ["active", "expired", "revoked"], "description": "Method lifecycle state." },
    "createdAt": { "type": "string", "format": "date-time", "description": "Creation time (UTC)." }
  },
  "required": ["id", "userId", "type", "label", "maskedIdentifier", "isDefault", "status", "createdAt"]
}
```

### Payment

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/sabi/payment.schema.json",
  "title": "Payment",
  "description": "A single executed or attempted money movement for a service.",
  "type": "object",
  "properties": {
    "id": { "type": "string", "description": "Stable identifier for the payment." },
    "userId": { "type": "string", "description": "References the User who paid." },
    "serviceId": { "type": "string", "description": "References the Service being paid." },
    "mandateId": { "type": "string", "description": "References the PaymentMandate authorising this payment." },
    "paymentMethodId": { "type": ["string", "null"], "description": "References the PaymentMethod that funded the payment; null when settled from a channel without a stored method." },
    "type": { "type": "string", "enum": ["government", "bill"], "description": "Payment category." },
    "amount": { "type": "integer", "description": "Amount paid in minor units (kobo for NGN), stored at execution time. The currency field gives the currency.", "minimum": 0 },
    "currency": { "type": "string", "description": "Currency code.", "default": "NGN" },
    "payee": { "type": "string", "description": "Payee as authorised, stored at execution time so a receipt never changes if a catalogue entry is later edited." },
    "channel": { "type": "string", "enum": ["app", "ussd", "sms", "whatsapp"], "description": "Channel the payment was made from." },
    "status": { "type": "string", "enum": ["initiated", "authorized", "pending", "succeeded", "failed", "reversed", "refunded"], "description": "Payment state." },
    "railReference": { "type": ["string", "null"], "description": "Reference returned by the payment rail." },
    "failureReason": { "type": ["string", "null"], "description": "Reason when the payment fails." },
    "idempotencyKey": { "type": "string", "description": "Key that prevents a duplicate execution." },
    "createdAt": { "type": "string", "format": "date-time", "description": "Creation time (UTC)." },
    "updatedAt": { "type": "string", "format": "date-time", "description": "Last state change (UTC)." },
    "completedAt": { "type": ["string", "null"], "format": "date-time", "description": "Completion time (UTC); null while open." }
  },
  "required": ["id", "userId", "serviceId", "mandateId", "type", "amount", "currency", "payee", "channel", "status", "idempotencyKey", "createdAt", "updatedAt"]
}
```

### PaymentEvent

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/sabi/payment-event.schema.json",
  "title": "PaymentEvent",
  "description": "An append-only record of a state change on a payment.",
  "type": "object",
  "properties": {
    "id": { "type": "string", "description": "Stable identifier for the event." },
    "paymentId": { "type": "string", "description": "References the Payment the event belongs to." },
    "status": { "type": "string", "enum": ["initiated", "authorized", "pending", "succeeded", "failed", "reversed", "refunded"], "description": "Payment state after this event." },
    "actor": { "type": "string", "enum": ["user", "system", "rail", "provider"], "description": "What caused the state change." },
    "detail": { "type": ["string", "null"], "description": "Short reason or note attached to the change." },
    "occurredAt": { "type": "string", "format": "date-time", "description": "When the change occurred (UTC)." },
    "createdAt": { "type": "string", "format": "date-time", "description": "Creation time (UTC)." }
  },
  "required": ["id", "paymentId", "status", "actor", "occurredAt", "createdAt"]
}
```

### Receipt

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/sabi/receipt.schema.json",
  "title": "Receipt",
  "description": "A stored proof of a completed payment.",
  "type": "object",
  "properties": {
    "id": { "type": "string", "description": "Stable identifier for the receipt." },
    "paymentId": { "type": "string", "description": "References the Payment this receipt proves." },
    "userId": { "type": "string", "description": "References the User who owns the receipt." },
    "kind": { "type": "string", "enum": ["official_e_receipt", "biller_confirmation", "token"], "description": "Kind of proof." },
    "receiptNumber": { "type": ["string", "null"], "description": "Official receipt identifier, such as the FTeR number." },
    "verificationStatus": { "type": "string", "enum": ["verified", "unverified", "pending"], "description": "Whether the receipt was confirmed against its source." },
    "unverifiedReason": { "type": ["string", "null"], "description": "Plain reason the receipt is unverified, such as source unavailable or identifier not found; null when verified or pending." },
    "documentRef": { "type": "string", "description": "Reference to the stored document." },
    "issuedAt": { "type": ["string", "null"], "format": "date-time", "description": "When the source issued the receipt." },
    "createdAt": { "type": "string", "format": "date-time", "description": "Creation time (UTC)." }
  },
  "required": ["id", "paymentId", "userId", "kind", "verificationStatus", "documentRef", "createdAt"]
}
```

### PaymentCase

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/sabi/payment-case.schema.json",
  "title": "PaymentCase",
  "description": "A case tracking a failed payment, dispute, or missing receipt.",
  "type": "object",
  "properties": {
    "id": { "type": "string", "description": "Stable identifier for the case." },
    "paymentId": { "type": "string", "description": "References the Payment under investigation." },
    "userId": { "type": "string", "description": "References the User affected." },
    "type": { "type": "string", "enum": ["failed_no_delivery", "disputed", "refund", "receipt_missing"], "description": "Reason the case was opened." },
    "status": { "type": "string", "enum": ["open", "awaiting_provider", "resolved", "escalated", "closed"], "description": "Case state." },
    "assigneeRole": { "type": "string", "description": "Role responsible for the case." },
    "openedAt": { "type": "string", "format": "date-time", "description": "When the case was opened (UTC)." },
    "updatedAt": { "type": "string", "format": "date-time", "description": "Last update (UTC)." },
    "resolvedAt": { "type": ["string", "null"], "format": "date-time", "description": "When the case was resolved; null while open." }
  },
  "required": ["id", "paymentId", "userId", "type", "status", "assigneeRole", "openedAt", "updatedAt"]
}
```

### Reminder

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/sabi/reminder.schema.json",
  "title": "Reminder",
  "description": "A scheduled prompt to pay an obligation before its deadline.",
  "type": "object",
  "properties": {
    "id": { "type": "string", "description": "Stable identifier for the reminder." },
    "userId": { "type": "string", "description": "References the User to remind." },
    "serviceId": { "type": "string", "description": "References the Service or obligation that is due." },
    "dueAt": { "type": "string", "format": "date-time", "description": "Deadline (UTC)." },
    "leadTimeHours": { "type": "integer", "description": "How many hours before the deadline to remind.", "minimum": 1 },
    "channel": { "type": "string", "enum": ["app", "ussd", "sms", "whatsapp"], "description": "Channel to remind on." },
    "status": { "type": "string", "enum": ["scheduled", "sent", "cancelled", "paid"], "description": "Reminder state." },
    "createdAt": { "type": "string", "format": "date-time", "description": "Creation time (UTC)." }
  },
  "required": ["id", "userId", "serviceId", "dueAt", "leadTimeHours", "channel", "status", "createdAt"]
}
```

### Pool

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/sabi/pool.schema.json",
  "title": "Pool",
  "description": "A collection toward a single shared obligation.",
  "type": "object",
  "properties": {
    "id": { "type": "string", "description": "Stable identifier for the pool." },
    "ownerId": { "type": "string", "description": "References the User who created the pool." },
    "serviceId": { "type": "string", "description": "References the Service the pool pays." },
    "targetAmount": { "type": "integer", "description": "Amount needed to pay the obligation, in minor units (kobo for NGN). The currency field gives the currency.", "minimum": 0 },
    "currency": { "type": "string", "description": "Currency code.", "default": "NGN" },
    "status": { "type": "string", "enum": ["open", "funded", "paid", "refunded", "closed"], "description": "Pool state." },
    "closesAt": { "type": ["string", "null"], "format": "date-time", "description": "Deadline for contributions; null if open-ended." },
    "createdAt": { "type": "string", "format": "date-time", "description": "Creation time (UTC)." }
  },
  "required": ["id", "ownerId", "serviceId", "targetAmount", "currency", "status", "createdAt"]
}
```

### Contribution

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/sabi/contribution.schema.json",
  "title": "Contribution",
  "description": "One member's payment into a pool.",
  "type": "object",
  "properties": {
    "id": { "type": "string", "description": "Stable identifier for the contribution." },
    "poolId": { "type": "string", "description": "References the Pool being funded." },
    "contributorId": { "type": "string", "description": "References the User who contributed." },
    "amount": { "type": "integer", "description": "Amount contributed in minor units (kobo for NGN). The currency field gives the currency.", "minimum": 0 },
    "currency": { "type": "string", "description": "Currency code.", "default": "NGN" },
    "status": { "type": "string", "enum": ["pledged", "paid", "reversed"], "description": "Contribution state." },
    "paymentId": { "type": ["string", "null"], "description": "References the Payment that settled this contribution, when paid." },
    "createdAt": { "type": "string", "format": "date-time", "description": "Creation time (UTC)." }
  },
  "required": ["id", "poolId", "contributorId", "amount", "currency", "status", "createdAt"]
}
```

### Notification

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/sabi/notification.schema.json",
  "title": "Notification",
  "description": "A message sent or queued to a user over a channel.",
  "type": "object",
  "properties": {
    "id": { "type": "string", "description": "Stable identifier for the notification." },
    "userId": { "type": "string", "description": "References the User the message is for." },
    "channel": { "type": "string", "enum": ["sms", "push", "whatsapp"], "description": "Delivery channel." },
    "template": { "type": "string", "description": "Template identifier used for the message." },
    "status": { "type": "string", "enum": ["queued", "sent", "delivered", "failed"], "description": "Delivery state." },
    "payloadRef": { "type": ["string", "null"], "description": "Reference to message content that must not be stored inline." },
    "createdAt": { "type": "string", "format": "date-time", "description": "Creation time (UTC)." },
    "sentAt": { "type": ["string", "null"], "format": "date-time", "description": "When the message was sent; null while queued." }
  },
  "required": ["id", "userId", "channel", "template", "status", "createdAt"]
}
```

## Relationships

- **AuthCredential** `userId` references **User**, each credential belongs to exactly one user.
- **ChannelLink** `userId` references **User**, each channel link belongs to exactly one user.
- **IdentityVerification** `userId` references **User**, each verification belongs to exactly one user.
- **Service** `providerId` references **Provider**, each service is fulfilled by at most one provider.
- **Service** `capabilityId` references **Capability**, each service maps to at most one capability.
- **Capability** `providerId` references **Provider**, each capability is exposed by exactly one provider.
- **Capability** `serviceId` references **Service**, a capability maps to at most one service.
- **CapabilityReview** `providerId` references **Provider**, each review concerns exactly one provider.
- **CapabilityReview** `capabilityId` references **Capability**, a review concerns at most one capability.
- **PaymentMandate** `userId` references **User**, each mandate is authorised by exactly one user.
- **PaymentMandate** `capabilityId` references **Capability**, each mandate may name one capability.
- **PaymentMethod** `userId` references **User**, each method belongs to exactly one user.
- **Payment** `userId` references **User**, each payment is made by exactly one user.
- **Payment** `serviceId` references **Service**, each payment settles exactly one service.
- **Payment** `mandateId` references **PaymentMandate**, each payment is authorised by exactly one mandate.
- **Payment** `paymentMethodId` references **PaymentMethod**, each payment is funded by at most one stored method.
- **PaymentEvent** `paymentId` references **Payment**, each event belongs to exactly one payment.
- **Receipt** `paymentId` references **Payment**, each receipt proves exactly one payment.
- **Receipt** `userId` references **User**, each receipt belongs to exactly one user.
- **PaymentCase** `paymentId` references **Payment**, each case concerns exactly one payment.
- **PaymentCase** `userId` references **User**, each case concerns exactly one user.
- **Reminder** `userId` references **User**, each reminder targets exactly one user.
- **Reminder** `serviceId` references **Service**, each reminder concerns exactly one service.
- **Pool** `ownerId` references **User**, each pool has exactly one owner.
- **Pool** `serviceId` references **Service**, each pool pays exactly one service.
- **Contribution** `poolId` references **Pool**, each contribution belongs to exactly one pool.
- **Contribution** `contributorId` references **User**, each contribution is made by exactly one user.
- **Contribution** `paymentId` references **Payment**, a contribution is settled by at most one payment.
- **Notification** `userId` references **User**, each notification targets exactly one user.

## Personal data

Fields annotated `x-personal-data: true` are subject to the retention, residency, and deletion rules in [nfr](nfr.md): `User.displayName`, `AuthCredential.publicKey`, `AuthCredential.secretHash`, `ChannelLink.address`, `IdentityVerification.identifierMasked`, `PaymentMethod.maskedIdentifier`, `PaymentMethod.processorToken`, and `Provider.contactEmail`. `AuthCredential.credentialId` and `AuthCredential.lookupDigest` are non-secret identifiers and carry no `x-personal-data` annotation. Analytics events reference `User.id` only and never these fields.
