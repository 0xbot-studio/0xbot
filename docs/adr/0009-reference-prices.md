# ADR 0009 — Typed reference-price capabilities

Status: deployed in 20260908-preview-13 (2026-09-08).

## Decision

Prioritize BTC/ETH/BNB and exact-contract price queries before wallet NFT indexing. Introduce market.asset_price and market.token_price, with separate MCP tools and REST routes sharing use cases and generated contracts. Keep global asset identity separate from chain-bound contract identity; USD reference prices are not executable swap quotes.

The provider registry depends on identity and network support only. Balance and price calls assert their own narrow capability ports. A price adapter must not implement a fake balance method. Existing eligibility declarations, route settings, health tracking and usage records remain shared. The internal market scope handles global quotes; public asset capabilities do not falsely advertise chain-specific prices.

Start with a dedicated DefiLlama adapter and explicit upstream mappings. Preserve exact decimal strings, source identity, observation/fetch times and cache state. Bound cache size, request rate, response size and freshness; return explicit errors instead of zero or stale prices. See api/docs/prices.md for the operational limits and rollout migration.

## Consequences

New providers can adapt the same contracts without changing balance logic or page components. Separate bilingual Playground pages keep price inputs distinct. Admin contracts derive from the executable registry; no manual frontend-only capabilities.

One initial price source means no cross-source consensus or guaranteed availability. Global cold-fetch limits and cache are process-local; a multi-replica rollout may need a shared upstream budget. Not every contract is indexed. Commercial redistribution terms require review before monetization; no paid account or credentials were provisioned.
