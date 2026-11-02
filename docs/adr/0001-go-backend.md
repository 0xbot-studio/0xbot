# ADR 0001：采用 Go 后端

日期：2026-09-07

状态：已接受，用户明确确认。

## 背景

0xbot 聚合多个 Web3 Provider，需要清晰的适配器边界、并发请求、超时与取消、路由和执行状态管理。原计划暂定 TypeScript / Fastify，尚未初始化代码。

## 决定

api 项目使用 Go，替代原 TypeScript / Fastify 候选。三个前端继续使用 TypeScript。初期采用模块化单体，通过接口隔离 Provider 和基础设施。REST 与 MCP 共享应用用例；OpenAPI 定义跨项目契约，生成前端客户端。

计划配套 net/http + chi、PostgreSQL + pgx + sqlc、goose 和 slog；具体版本及 MCP SDK 在实施时验证。Redis 和独立 worker 按实际需求引入。

## 替代方案与代价

TypeScript 可与前端共享语言，并直接使用 JS 生态的 Provider SDK；本项目选择 Go 后，通过契约生成客户端保证跨语言一致性。供应商优先接 HTTP API，仅当无法避免 JS SDK 时引入独立适配服务，并承担其额外部署维护成本。

本次仅改变规划与工程规范，无现有代码或数据迁移。
