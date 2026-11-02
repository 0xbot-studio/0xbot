# ADR 0006 — 能力契约标准 v1 与只读契约中心

日期：2026-09-07。状态：已随 20260907-preview-08 上线，通过自动化及公网验证。

## 决定

0xbot 的契约是供应商无关的 API 能力标准，不是链上智能合约。每个可执行能力具有稳定 ID、独立语义版本、双语说明、输入输出 Schema、执行限制以及 REST/MCP 映射。Provider 接入由工程实现适配器，不把原始 RPC/供应商响应直接作为平台契约。

`internal/capabilities` 统一维护已发布能力身份、版本及映射。OpenAPI 维护 REST Schema；Go 输入/输出类型维护 MCP Schema，契约测试核对双方字段及必填项，避免复制后漂移。MCP 后台视图与真实工具注册共用构建路径，不经 HTTP 自调用，不执行工具。Provider 能力取自注册适配器的 CapabilityIDs 声明，而不是给所有 Provider 硬编码相同列表。

管理端按「能力契约」「MCP 工具」「供应商适配」拆成三个页面，显示实际定义和运行配置映射；GET /v1/admin/contracts 必须经过管理员身份与 TOTP 校验。只读，不支持在线编辑代码、Schema、RPC 地址或凭证。注册支持不等于可用：另显示 enabled、priority 和熔断观测状态，不伪造健康探测或 CI 历史。

## 统一输出

所有路由读取能力共享 `data.QueryResult[T]`，保持现有 JSON 不变：

| 字段 | 标准 |
| --- | --- |
| id | 平台操作 ID，关联计量与调用记录 |
| provider | 实际命中的平台路由 ID，不是原始端点 URL |
| status | 成功结果为 succeeded；失败不返回伪造成功对象 |
| latency_ms | 平台记录的执行耗时，整数毫秒 |
| fee_usd | 十进制字符串；当前预览为 "0" |
| data | 能力专属载荷；单币为 Balance，批量为 Balance[] |

Balance 必须携带 address、network、asset、symbol、decimals、amount、block_number、observed_at。amount 为最小单位非负整数字符串，不使用浮点数；block_number 为十进制整数字符串。时间为 RFC3339 观测时间，不冒充区块时间或最终性证明。代币额外携带 contract、name、variant，保留 Binance-Peg、USDT0 等发行差异。零余额是成功。批量固定同一 Provider、同一区块的三种资产，全部成功或整体失败。

REST 失败保持 `{error:{code,request_id}}` 与 HTTP 状态码：invalid_request/unsupported_provider 为 400，authentication_required 为 401，权限或来源拒绝为 403，rate_limited 为 429，provider_unavailable 为 503，internal_error 为 500。MCP 失败保持协议定义的 isError 和安全文本；认证及协议错误独立处理。两者共享业务语义，不强行伪装成相同 wire envelope。get_supported_tokens 是静态注册表辅助工具，不产生路由查询操作，不套用 QueryResult。

## Provider 适配义务

1. 明确支持网络和能力。上游链 ID 与请求不匹配则拒绝；不得跨网络兜底。
2. 将上游十六进制数量转换为精确十进制整数；核验响应格式、数量范围和 decimals。
3. 将平台资产注册表身份、观测区块与时间填入 Balance；不按 USDT/USDC 名称推断合约。
4. 正确传播取消与超时。只读调用可以有限故障切换，显式指定 Provider 不静默换线。
5. 将上游错误映射为平台安全错误；禁止返回上游凭证或原始认证头。
6. 适配器不创建计量 ID、不自行收费；应用用例统一记录操作和费用。
7. 新适配器须通过固定样本契约测试（字段、单位、错误、超时、错误链和畸形数据），再加入真实 RPC 冒烟。页面不以注册声明替代测试证据。

## 版本与边界

本轮三项余额契约标记为 1.0.0，但服务仍是无 SLA 的 public preview。新增可选字段可兼容演进；更改字段语义、单位、必填参数或删除字段需新主版本及迁移。现有 REST 路径、四个 MCP 工具名称与授权 scope 不变；无数据库迁移。写操作、NFT、receipt 尚未实现，不纳入可执行契约。动态草稿、描述发布、回滚、独立适配器版本及 CI 运行历史后续单独设计。

## 验证

Go 全量测试（含专用数据库权限集成）、race、vet 和构建通过；测试逐字段检查 REST/MCP 类型、必填项及嵌套结构，验证后台元数据与 HTTP tools/list Schema 一致，覆盖未登录、普通用户、管理员未 MFA 拒绝与 MFA 后读取成功。发现并修正 MCP 批量数组可为 null 的旧声明，成功结果现明确为三项数组。管理端双语页面服务端渲染与关联示例测试、类型/格式检查及构建通过。

preview-08 公网验证：管理员及 TOTP 权限边界、完整契约快照（3 能力、4 工具、6 Provider）、新后台静态资源、原有 REST/MCP 三网络查询和 OAuth 全流程通过。首轮 dRPC 批量查询 503，完整复测通过。备份和 preview-07 回滚点保留。未做浏览器视觉/交互复核；自动化验证不等于外部安全审计。
