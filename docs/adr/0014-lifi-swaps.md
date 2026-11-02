# ADR 0014 — LI.FI EVM swap preparation and user-wallet execution

## Context

Token swaps existed only as a planned website capability. The syntra-protocol
reference separates a credential-bearing LI.FI boundary from browser-wallet
execution. 0xbot must preserve its Go capability contracts and module boundaries.

## Decision

Introduce swaps domain types, a LI.FI adapter and read-only routed operations for
catalogs, quotes and status. Register real REST/MCP mappings and generate the app
client from OpenAPI. Six EVM networks form the initial scope. Native assets use
zero-address identity; tokens use exact network+contract. Never guess by ticker.
No source-code imports from syntra-protocol; its proxy/amount/expiry approach is
adapted to the existing Go application.

The app submits through the user's EIP-1193 wallet, after an independent review.
No API write capability, private key, server signing or MCP execution is introduced.
Existing data:read permits preparation, not wallet authorization. Each approval and
swap is a distinct user action. ERC-20 approval uses the exact input amount; an
insufficient nonzero allowance is explicitly reset first. Fresh quotes follow
approval confirmation. Expiry, account, chain and allowance are checked before
submission. Quotes are never cached; catalog cache is bounded by supported networks.

Before eth_sendTransaction, record an unknown receipt in tab-scoped sessionStorage.
Only explicit wallet rejection clears it immediately. Submitted hashes are tracked
without rebroadcast; unknown/no-hash receipts require wallet activity review.
A synchronous lock prevents repeated clicks while a prompt is open, and receipt
state survives navigation/reload. This is not server idempotency, multi-tab locking
or durable execution history. There is no automatic fallback for a write.

## Consequences

Transaction signing/broadcast is wallet-owned; quote operations have ordinary
workspace-isolated query metering. LI.FI errors and incomplete statuses cannot be
reported as completion. Partial/refunded/unknown transfers require manual review.
The backend verifies response identities, exact amounts, slippage, transaction
chain/sender/value and calldata shape. It does not claim independent semantic
verification of arbitrary router bytecode or token safety.

An enabled route and either a server key or explicit `LIFI_PUBLIC_ACCESS=true`
are required for live use. LI.FI documents public access without a key
(https://docs.li.fi/api-reference/rate-limits): 75 quotes/IP per two hours.
Public mode locally caps at 70 rolling quotes per process and honors upstream
reset headers; restarts do not reset the upstream quota. Quote cooldown remains
separate from receipt queries. Non-EVM, WalletConnect,
server-side execution, route comparison across multiple quote providers and
cross-device transaction recovery remain outside this release. Rollback: disable
`lifi`, roll back app/API images; retain additive migration 008 and query history.
No destructive database rollback is needed. Back up using the existing deployment
procedure before release. Real-funds validation is separate from deterministic tests.
