# ADR 0016: Agent smart-account read capabilities

Status: Draft — proposed, not implemented. Decision date: 2026-11-03.

## Context

The EqualFi Labs agent-wallet-core suite deploys ERC-6551-surfaced, ERC-6900-modular smart accounts for AI agents, with an ERC-8128 policy registry (session windows, budgets, revocation), SIWA gateway validation and ERC-8004 identity adapters. These accounts are deployed outside the canonical ERC-6551 registry, so the existing `data.token_bound_account.get` capability (ADR 0007, canonical CREATE2 verification only) intentionally rejects them. 0xbot serves AI agents over MCP; letting an agent inspect its own on-chain account type, binding, session policies and identity mapping is a natural read capability and fills the documented ADR 0007 blind spot without touching its execution boundaries.

Upstream maturity: ERC-8128 and the SIWA design are drafts; there is no production deployment manifest yet. Interface surfaces are stable view functions.

## Decision

- Add a versioned read capability family behind REST and MCP, sharing the existing envelope, routing, circuit state and metering, scoped to explicit curated deployments (factory, policy registry, identity adapter addresses per network, provenance per ADR 0004/0015 discipline):
  - `agent.account.inspect` — given an account address on a supported network: ERC-165 interface detection (IERC6551Account, IERC6900Account, IERC1271) and, when detected, read the standard views `token()`, `owner()`, `nonce()`, `accountId()`. One provider, one block. Module enumeration is not part of the contract: ERC-6900 manifests exist only at install time and no standard list view exists, so this capability never claims a module inventory.
  - `agent.account.policy` — read `ERC8128PolicyRegistry` views (`getPolicy`, `getEpoch`, `isGuardian`, `isPolicyActive`) for a given account/entityId/sessionKey.
  - `agent.account.identity` — read the ERC-8004 adapter mapping views (`getAgentId(account)`, `getAccount(agentId)`).
- Interface-level detection is classification, not safety: ERC-165 matches do not certify module behavior, signer authority, implementation security or identity correctness. Policy reads are observations, not authorization decisions by 0xbot.
- All three use `data:read`, the existing auth paths and a dedicated per-workspace rate bucket. No signing, execution, module installation, or ERC-1271 validation is performed — those remain outside every current capability contract.
- Until a verified deployment manifest exists, capabilities may ship with routes disabled (fail-closed, pyth-prices pattern) rather than with placeholder addresses.

## Consequences

- 0xbot becomes readable-agnostic across canonical and non-canonical ERC-6551 deployments while ADR 0007's registry guarantees stay unchanged for the canonical path.
- Follow-on options staged for separate decisions: SIWA login via server-side read-only ERC-1271 verification (a third sign-in path beside SIWE and X), and an ERC-8128-native gateway where agent call budgets map to on-chain policies. Both touch identity/auth architecture and draft-stage standards and require their own ADRs.
- The curated deployment list must track upstream contract upgrades; drift is an operational review item for the admin contract center.
- Unverified draft semantics (ERC-8128 shapes) are read as raw typed observations with upstream field names preserved; 0xbot does not interpret policy meaning beyond activity/window status.

See docs/agent-account-reads.md (workspace research note) for interface inventory and the staged integration layers.
