# ADR 0003: Network-scoped public RPC routes

Ethereum, BSC and Polygon PoS share the native-balance capability, not a single interchangeable endpoint. Each route is an independently enabled and monitored provider adapter with exactly one network. Existing Ethereum IDs `publicnode` and `drpc` remain unchanged. New IDs are `publicnode-bsc`, `bnbchain-bsc`, `publicnode-polygon`, `drpc-polygon`.

The router filters by network before priority or health checks. Explicit route selection on a different network is rejected. Adapters validate the upstream chain ID before reading a concrete block and querying its balance. Asset symbol and identifier derive from a shared network registry; amounts remain base-unit strings with 18 decimals. Polygon PoS uses POL, not a hard-coded ETH or MATIC label.

The additive 002 migration inserts new settings without changing existing ones. Old code ignores unknown route settings, so code rollback does not require deleting these rows. Existing operator settings and credentials remain untouched.

All six routes use free public endpoints. Catalog responses expose `pricing_model`, `requires_api_key` (upstream only), `upstream_fee_usd` and `platform_fee_usd`. Platform authentication is still required. No paid credentials, subscriptions or automatic paid fallback are added. Public access is subject to limits and is not an SLA. BSC dRPC is excluded because the research probe returned HTTP 429. BNB's public endpoint is used only for the implemented balance capability, not log indexing.

PublicNode endpoints are documented at https://bsc.publicnode.com/ and https://polygon.publicnode.com/. Additional references: https://docs.bnbchain.org/bnb-smart-chain/developers/json_rpc/json-rpc-endpoint/ and https://docs.polygon.technology/pos/reference/rpc-endpoints. SLIP-44 identifiers follow https://github.com/satoshilabs/slips/blob/master/slip-0044.md (Polygon registry entry retains its historical MATIC name).

Commercial providers remain a follow-up requiring credentials, budget authorization, confirmed aggregation terms and method-aware cost accounting. This release does not implement a generic arbitrary JSON-RPC proxy.
