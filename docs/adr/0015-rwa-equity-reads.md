# ADR 0015: Curated tokenized-equity read capabilities

Status: implemented for API 0.11.0; not yet deployed. Decision date: 2026-11-02; curated contracts verified on-chain via PublicNode on 2026-11-03.

## Context

Tokenized equities (xStocks, Ondo Stocks, exchange offerings) crossed $3B onchain market cap in 2026 and are a growing share of RWA activity. 0xbot aggregates read capabilities for developers and AI agents; balances and reference prices of tokenized stocks fit the existing capability/provider architecture, while issuance, trading and custody stay out of scope for legal exposure.

The stablecoin registry, `get_token_balance` and the common three-asset snapshot are contractual (ADR 0004). Price routing treats `price_not_found` as a semantic error that stops provider iteration (ADR 0009 discipline), so a second price source covering a disjoint contract set cannot share `market.token_price` without changing global routing semantics.

## Decision

- Extend the curated token registry with `asset_class` (`stablecoin` | `tokenized_equity`) plus `underlying` and `issuer` metadata. Equity entries are Backed xStocks ERC-20s on Ethereum: AAPLx, TSLAX, AMATx (18 decimals). Addresses were cross-checked between Etherscan and Ethplorer and limited to entries with an unambiguous provenance; INTCx was excluded because two distinct upstream equity feeds exist with no authoritative tiebreaker. `symbol()/name()/decimals()` and deployed code were verified read-only on PublicNode (2026-11-03); on-chain `name()` values are mirrored in the registry.
- Add two versioned capabilities behind REST and MCP sharing the existing envelope, routing, circuit state and metering: `data.rwa.token_balance` (same-block decimals-verified `balanceOf`) and `market.equity_price` (reference price of the underlying stock, exact contract identity only, no ticker guessing).
- Keep the existing contracts intact: `get_common_balances` filters to `asset_class=stablecoin` and remains a fixed three assets; `get_token_balance` still rejects equity contracts; `market.token_price` remains DefiLlama-only.
- Equity prices are served by a new credential-free `pyth-prices` provider (Pyth Hermes pull API) declaring only `market.equity_price`, with a provider-internal curated contract→feed mapping as eligibility. Exact decimal strings are scaled from Pyth integer price/exponent with bounds; `confidence` reports price/(price+conf) as a 0..1 score; observations older than 10 minutes are `price_stale` (market-closed prices are never returned as last-known values); upstream 401/429/5xx map to explicit errors — the provider fails closed and can be disabled by dashboard route settings.
- `data.rwa.token_balance` gets a dedicated 20/minute/workspace bucket; `market.equity_price` shares the existing price budget.
- A balance is an on-chain observation, not eligibility or a compliance verdict; the issuer's non-US availability restriction still applies. Prices are reference values, not executable quotes or NAV.

## Consequences

- OpenAPI 0.11.0 and the MCP tool registry gain two paths/tools (29 tools total); `GET /v1/tokens` and `get_supported_tokens` expose `asset_class`/`underlying`/`issuer`. Frontend clients regenerate from the published contract.
- Adding tokens or networks (BSC xStocks, Base, Ondo's permissioned tokens) requires registry entries plus verified addresses, and for permissioned models an explicit position on what a restricted transfer means for read semantics.
- During development the Hermes price endpoint returned 401 from the development network while the catalog endpoint worked; production reachability is a deployment-stage check. The provider stays disabled/failing-closed until verified.
- Eligibility checks, corporate actions, NAV, portfolio composition and ERC-6551 holdings snapshots remain future capability contracts; the token-bound account reads (ADR 0007) compose with these capabilities unchanged.
- One price source means no cross-source consensus; equity observations are single-upstream and market-hours-bound by design.

See api/docs/rwa-token-balances.md for inputs, outputs, errors and verification.
