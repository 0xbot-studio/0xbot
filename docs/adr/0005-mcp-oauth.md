# ADR 0005 — MCP authorization and onboarding

Date: 2026-09-07. Status: deployed in 20260907-preview-06; not independently security-audited.

## Boundaries

The public website owns a reusable bilingual connection component shared by the homepage and `/en/connect/`, `/zh/connect/`. Client commands, configuration and agent-assisted instructions come from one client catalog. No credentials are embedded in snippets. Existing Next.js static hosting and the user-specified VPS are preserved; no Sites migration.

`internal/mcpoauth` owns MCP client registration, browser-bound requests, grants, codes and tokens. It is separate from the X OAuth login adapter in `identity`. `httpapi` owns protocol parsing, cookies, Origin checks and discovery. The user app owns consent and connected-app revocation. REST and MCP query use cases remain unchanged.

## Protocol and limits

- Protected resource: configured `API_ORIGIN + /mcp`; issuer: configured API origin, never inferred from proxy Host headers.
- RFC 9728 protected-resource metadata, RFC 8414 authorization-server metadata, 401 WWW-Authenticate discovery, authorization response `iss`.
- Public clients using dynamic registration and `token_endpoint_auth_method=none`. No client secrets or confidential-client grants. No CIMD URL fetching in this release: it is explicitly advertised as unsupported, avoiding a new SSRF surface. Pre-registration/CIMD can be added with separate trust and security rules.
- Authorization code only, PKCE S256 required, exact audience/resource at authorization, exchange and refresh. Only `data:read` is granted. Client-provided names are unverified and visibly labeled on consent.
- Registered redirects must be HTTPS or loopback HTTP, without credentials, fragments, wildcards or reserved OAuth response parameters. Matching is exact except loopback port variation permitted by RFC 8252. Callback query and path may not change during authorization. Code exchange requires the exact actual authorized redirect, including port.
- Random browser-bound request: 10 minutes. Secure deployments use a host-prefixed, Secure, HttpOnly, SameSite=Lax cookie. A later flow in the same browser supersedes the previous pending cookie.
- User signs in using existing wallet/X flow, then explicitly approves or denies. Login never automatically grants an MCP connection. Consent POST requires app Origin plus a valid session and matching flow cookie. X callbacks resume only a valid same-browser request on the configured app origin. X itself still requires application credentials.
- Code: single use, 60 seconds. Access token: at most 15 minutes. Refresh token: rotating, bounded by a 30-day grant lifetime. Random secrets stored only as SHA-256 hashes. Code and refresh consumption are transactional; replay with valid bindings revokes the entire grant. Concurrent refresh attempts must be serialized by clients; there is no replay grace period.
- OAuth tokens are accepted only at `/mcp`; REST, key creation, identity and admin endpoints do not accept them. API keys continue to work as before. Future MCP write tools need a separate scope and explicit authorization gate; they cannot inherit this read-only scope.
- Connected applications are listed/revoked by workspace. Maximum 100 active grants per workspace; approval count is serialized. Revocation affects every access/refresh token in the grant immediately. Website logout does not revoke grants.
- Existing IP auth limit (30/minute) applies to protocol endpoints. Existing workspace query limits apply to OAuth-authenticated MCP requests. Request and token bodies have bounded size. Protocol errors do not expose secrets or storage errors.
- Cross-origin protocol metadata/register/token/revoke requests allow non-credentialed CORS. Consent/account APIs retain restrictive app/admin Origin policy. Cross-origin MCP cannot use ambient session cookies; it must send a bearer credential.

## Storage, rollout and rollback

Migration 003 is additive and uses distinct `mcp_oauth_*` tables. It does not alter wallet login, X identity tables, API keys or query operations. Hourly cleanup removes expired requests, grants expired more than a day ago and stale clients with no grants/requests. Spent refresh tokens are retained until grant cleanup for replay detection. Stale client registrations may need re-registration.

Coordinate API, user app and website deployment; otherwise new OAuth instructions could point to a server without discovery/consent support. Verify production TLS origins, no-referrer and frame-ancestors protections (existing host Nginx config), disabled API access logging, backup and migration readiness. Preserve the previous release. Older API code ignores the additive tables but cannot authenticate OAuth tokens; API keys remain the fallback after rollback. No new signing secret or X credential is automatically provisioned.

## Verification and remaining gaps

Tests cover standard PKCE vector, redirect validation, variable loopback ports, browser binding, explicit consent/denial, expired code/request/access, wrong client/audience/redirect/verifier, code and refresh concurrency/replay, workspace isolation, grant revocation, OAuth discovery/error shape, CORS, Origin/CSRF, MCP initialize/tool call and rejection at REST/admin endpoints. Existing wallet/X/API key tests remain part of the suite.

Website and app compile and build; request examples follow official client docs. A browser connection was unavailable, so visual/interaction QA and actual Claude Code/Cursor/Codex interactive OAuth login remain unverified. Automated HTTP integration uses an isolated test identity and real authorization store, not an actual browser login. Do not advertise full client certification or an external security audit.

Local real-upstream smoke also passed: a generated unfunded EOA completed wallet signature login, API key queries across six routes, token/common-balance queries and MCP; then completed DCR, browser-cookie-bound HTTP consent, PKCE exchange and refresh rotation, called MCP tools using the resulting access token, revoked the grant and observed HTTP 401. Test API keys, OAuth grants and sessions were revoked. Test accounts and operations remain as audit evidence. This is a local-server test against real upstream RPC services, not a production deployment or third-party browser UI test.

Production rollout verification: the same real-upstream smoke passed against https://api.0xbot.org with https://app.0xbot.org as the consent origin, including explicit HTTP consent, PKCE exchange, refresh rotation, MCP queries and revocation (401 afterward). Public discovery origins, consent-page security headers, bilingual connection pages, API-key compatibility and administrator TOTP regression also passed. Backup: `/opt/0xbot/backups/oxbot-20260907T144716Z.dump`; preview-05 retained for rollback. Actual third-party client browser login and visual QA remain unverified.

## Sources checked

- MCP authorization: https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization
- Claude Code MCP: https://code.claude.com/docs/en/mcp
- Cursor MCP: https://cursor.com/docs/mcp
- Codex MCP: https://learn.chatgpt.com/docs/extend/mcp?surface=cli (checked via OpenAI Docs skill)
