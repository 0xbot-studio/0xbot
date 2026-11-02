# ADR 0007: Optional ERC-6551 read capabilities

Status: deployed in 20260908-preview-09 (API only). Decision date: 2026-09-07; deployment date: 2026-09-08 Asia/Shanghai.

## Context

0xbot aggregates Web3 capabilities for AI. Some callers represent an agent with an NFT-bound account. Supporting those accounts must not require all users to adopt NFTs, change login, or authorize transactions.

## Decision

Add two versioned read capabilities (resolve/get) behind REST and MCP, with transport-independent request/result definitions in `internal/tokenbound` and canonical ERC-6551 bytecode/CREATE2 handling in the EVM RPC adapter. Route and operation execution use typed generic payloads to reuse existing eligibility, timeouts, circuit state and metering without encoding account data as balances. Provider capability declarations are hard routing filters.

The first version is canonical-registry, same-chain Ethereum/BSC/Polygon only. Resolve requires explicit implementation, bytes32 salt and decimal uint256 token ID. Get reads the immutable proxy binding and validates its canonical CREATE2 address. Returned verification concerns proxy structure/address only, not NFT existence/ownership, authorization, implementation behavior or safety. No arbitrary eth_call target/method interface is exposed. Existing balance capabilities accept the returned address unchanged.

Resolve/get share a 20/minute/workspace limit and existing data:read authorization. Unsupported proxy observations are semantic errors, not provider faults; upstream errors may fail over only in auto mode. Code reads are pinned to one block/provider; no partial success is returned. No migrations, deployment, credentials changes or expanded transaction scopes are involved.

## Consequences

- MCP discovery, REST OpenAPI 0.5.0 and the read-only admin contract center expose the same five business capabilities / six tools. Frontend API types are regenerated. No new Playground page is part of this backend increment.
- A counterfactual address is useful for lookup but is not proof of deployability or safe custody. All descriptions and documentation state this boundary.
- Custom registries, cross-chain bindings, enumeration, owner lookup, creation and execution need separate future decisions and capability contracts.
- Provider RPC access alone suffices for these reads; no NFT indexer dependency is introduced. The platform does not become an NFT issuer or wallet custodian.

See `api/docs/token-bound-accounts.md` for inputs, outputs, errors and verification; protocol source: https://eips.ethereum.org/EIPS/eip-6551.
