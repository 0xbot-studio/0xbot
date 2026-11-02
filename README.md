# 0xbot

一个接口，连接 Web3 服务。One API for Web3 services.

0xbot is a Web3 provider aggregation gateway for developers and AI agents: standard capability interfaces, provider adapters and routing, shared across REST and MCP. This repository holds the platform overview, the architecture baseline and the architecture decision records. The applications live in independent repositories:

| Repository | Stack | Domain |
| --- | --- | --- |
| [0xbot-api](https://github.com/0xbot-studio/0xbot-api) | Go / PostgreSQL | api.0xbot.org |
| [0xbot-app](https://github.com/0xbot-studio/0xbot-app) | React / Vite / TypeScript | app.0xbot.org |
| [0xbot-dashboard](https://github.com/0xbot-studio/0xbot-dashboard) | React / Vite / TypeScript | dashboard.0xbot.org |
| [0xbot-website](https://github.com/0xbot-studio/0xbot-website) | Next.js / React (static export) | 0xbot.org |

Each application is an independent repository, build and deployment unit. Cross-project integration uses the versioned OpenAPI contract; there are no runtime source imports between projects.

## Current status

The implemented slice is wallet sign-in → personal workspace → API key → native ETH / BNB / POL and curated USDT / USDC balances across Ethereum, BSC and Polygon PoS → request history and usage. Free public routes use PublicNode, dRPC and BNB Chain endpoints, with network-scoped fallback. API 0.11.0 adds curated tokenized-equity reads (xStocks balances and underlying-stock reference prices; see ADR 0015). PADIN public rooms, inbox, reference prices, NFT reads and read-only LI.FI swap quotes are being delivered incrementally; see each repository's README for the exact scope and safety boundaries. X sign-in requires an X OAuth application. Paid routes are not enabled. See ADR 0004 for token provenance, precision and batch limits.

## Documentation

- [Architecture baseline](docs/architecture.md)
- [ADR index](docs/adr/) — runtime, multichain routes, MCP OAuth, capability contracts, PADIN rooms/inbox, live streaming and LI.FI swaps.

Read the architecture baseline and the ADRs before changing boundaries. Each repository carries its own AGENTS.md engineering policy for standalone use.

## License

Apache-2.0, see [LICENSE](LICENSE). The 0xbot name, logo and 0xbot.org domains are project trademarks and are not licensed for third-party deployments — please rebrand self-hosted forks.
