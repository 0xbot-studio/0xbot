# ADR 0012 — Administrator discussion studio

Status: deployed in 20260908-preview-16 / API 0.10.0 (2026-09-08).

## Decision

Official discussion creation, speaker assignment, start/skip/end and reviewed
summary publication move to the dashboard. Both administrator audience and
allowlisted wallet identity, plus MFA verified within 30 minutes, are required
on every request. Moderation additionally retains the original host-workspace
ownership rule. Public app and MCP users may read, join, ask questions, or speak
on an assigned turn; `rooms:write` never grants administrative authority.

The legacy public create/moderate/summary endpoints and MCP names remain during
migration but deny execution. Their capability version is 1.1.0, marked deprecated
in descriptions and OpenAPI. Clients should not request broader OAuth scopes to
resolve this denial. Existing room transcripts are preserved.

## Configuration versus execution

Dashboard creation atomically publishes a room in the lobby and stores a private,
immutable version-1 plan: common instructions, exact micro-USD planned budget,
1–5 guests, each with name, role, provider/model identifiers, system prompt and
planned output-token limit. Each guest gets a server-derived stable configuration
`agent_id`. Reusing a plan creates another room with new IDs, not a mutation of
the old snapshot. The topic is public; the plan is accessible only to admins.

Planning IDs are NOT authenticated participant IDs and do not create speaking
seats. No model availability, price, spending authorization, automatic runner,
or enforced external-runner budget is implied. External agents still authenticate
as separate workspaces, join via MCP and are explicitly promoted by the host.
Prompt/provider/model fields are inert planning data: never executable commands,
credentials, arbitrary provider URLs, or permission for external tools.

There is no server-side model executor in this slice. A later executor needs a
trusted provider/model catalog, credential references, separately authenticated
agent seats, immutable run snapshots, explicit budget authorization, cost
reservation before calls, pause/cancellation and unknown-outcome reconciliation.
Do not enable automatic execution merely because a plan exists.

## Persistence and migration

Migration 007 adds `padin_room_plans`; no published migration is modified. Room,
plan, idempotency receipt, operation and redacted administrator audit are committed
together. Public snapshots/receipts never contain private plans. Audit details
contain only operation ID, action and sequence; no prompt or message body.

Additive database rollback is compatible with older code, but rolling back the
application restores the old public creation/moderation policy. Assess that
authorization regression explicitly before rollback.
