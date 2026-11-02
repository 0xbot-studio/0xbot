# ADR 0010 — PADIN Inbox and wallet-bound message authority

Status: deployed in 20260908-preview-14 (2026-09-08).

## Decision

Ship the address-message closed loop first, with PADIN Inbox (`padin-inbox`) as a provider. The public branding is PADIN Inbox; no unsupported claim of independent operation or end-to-end encryption is added. Keep Go as the main backend, with separate message domain/use cases and a narrow provider port. Do not build a new A2A protocol or require an external XMTP account for this first release.

Expose messaging.send/list/get through REST and MCP with one typed contract. Store messages for Ethereum-format recipients, including unregistered addresses. Read lists/details only as a verified recipient. The identity layer verifies EOA wallet signatures; message bodies cannot assert sender identity or confer permissions.

Public-data permission is insufficient for private inbox data or writes. Add explicit messages:read/messages:send OAuth scopes alongside data:read, and bind each message grant to one explicitly selected, already verified wallet. Preserve existing grants and keys as data-only. Browser sessions use current account ownership; X-only accounts cannot claim addresses.

PADIN storage is PostgreSQL plaintext, not E2EE. Prompt-injection resistance relies on treating bodies as untrusted data and avoiding execution/rendering of embedded markup. Message bodies are excluded from general usage/audit records. The send result means stored, not delivered/read. Send idempotency and usage commit atomically; unknown outcomes must retry the same provider/key/payload. No automatic provider fallback.

## Consequences

This satisfies address send → wallet authentication → recent inbox → message details without a third-party messaging network. It does not provide push notifications, decentralized transport, blocking, automatic expiry or client-side encryption. Retention/backup policy and stronger spam controls remain operational work before wider rollout. See api/docs/messages.md for limits and migration/rollback details.

XMTP remains a future provider. A separate Node SDK adapter is the likely integration boundary, subject to wallet-installation authorization, key custody, recipient reachability and network fee review. Different provider inboxes are not inherently interoperable; the contract must preserve delivery semantics rather than silently forwarding or retrying elsewhere.
