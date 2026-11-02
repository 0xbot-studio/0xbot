# ADR 0004 — Curated token and common-asset balances

Date: 2026-09-07. Status: accepted.

## Decision

Expose `data.token_balance` and `data.balances` through shared Go use cases, REST and MCP. Keep provider-specific JSON-RPC encoding in the EVM adapter. The token registry belongs to the data domain, not the transport or UI. Frontends consume GET `/v1/tokens` and the generated OpenAPI types.

Only six curated contracts are supported initially. Identity is network + exact contract, never symbol. Ethereum USDT/USDC use issuer-native contracts. BSC USDT/USDC are Binance-Peg assets (18 decimals). Polygon uses USDT0 at the upgraded USDT contract and issuer-native USDC (6 decimals); USDC.e is deliberately not merged or queried. Ethereum tokens use 6 decimals.

Contract provenance, checked 2026-09-07:

- Ethereum USDT: https://tether.to/en/supported-protocols/
- Ethereum and Polygon USDC: https://developers.circle.com/stablecoins/usdc-contract-addresses
- BSC contracts: https://www.binance.com/en/skills/detail/binance-web3/binance-agentic-wallet
- Polygon USDT0: https://usdt0.to/ecosystem/polygon and https://polygon.technology/blog/native-usdt0-comes-to-polygon-for-lower-fees-and-deeper-liquidity

The adapter validates chain ID, pins a block, verifies each token's onchain `decimals()` against the registry, and reads `balanceOf(address)` using `eth_call`. ABI uint256 responses require an exact 32-byte word; quantities remain integer strings end to end. No prices, approvals, transactions or arbitrary contracts are exposed.

Common-asset queries return native asset plus the two curated tokens for one network. All three are read at the same block from the same provider. A failure discards the entire batch; automatic routing may retry the entire batch on another eligible provider. One successful or failed batch produces one operation record, including after fallback. There is no partial-success response and no synthetic zero on error.

Batch routes have an 8-second per-provider timeout and a 20-second use-case deadline. A shared workspace limiter allows 20 batch requests per minute across REST and MCP, in addition to existing HTTP limits. Native and single-token queries retain existing routing limits. Preview fee is zero; public upstream limits and lack of SLA still apply.

## Tradeoffs

The curated registry avoids counterfeit/same-symbol ambiguity but excludes arbitrary ERC-20 assets and USDC.e. Runtime decimals verification costs one extra RPC call per token and fails closed if metadata changes. Sequential same-provider batches favor simple consistent behavior over maximum throughput. Future providers may implement the narrow batch interface without changing transport or accounting rules.
