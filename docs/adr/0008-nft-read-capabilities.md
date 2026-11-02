# ADR 0008 — NFT read contracts and bounded metadata

Status: released in 20260908-preview-12 (API 0.6.0).

NFT identity is network + contract + exact decimal token ID. Three independent capability contracts cover detail, ERC-721 owner, and ERC-721/1155 holder quantity. The domain owns types; operations own workspace metering and quotas; routing owns adapter eligibility; the EVM adapter owns ERC-165 and ABI reads; a separate metadata adapter owns untrusted network access.

REST and MCP use the same service and typed envelope. OpenAPI, actual MCP registration, provider declarations and the admin contract center must agree. The app uses independent page components with shared form fields/request lifecycle, not a universal NFT/balance component. Public website and app retain English-default Chinese support and no provider directory.

ERC-6551 is surfaced in separate resolve/inspect pages, with explicit implementation/salt and no transaction execution. Account inspection is not proof of NFT ownership or implementation safety.

Tradeoffs: strict ERC-165 excludes legacy/noncompliant and ambiguous dual-interface collections. RPC cannot enumerate wallet inventories or guarantee complete history; those remain explicitly planned until an indexer is configured. External metadata is optional, bounded, SSRF-protected and displayed as text. No caching is introduced in this preview; shared quotas bound demand. Images and arbitrary attributes are not loaded.

Validation: exact uint256 boundaries, same-block reads across all three networks, owner/quantity semantics, revert errors, metadata URL/size/JSON controls, REST/MCP schema parity, workspace quota/records and authenticated integration tests.
