# TrendRadar — 国内热榜聚合器

> 状态：✅ 运行中 | 部署：Zeabur Docker | 更新：2026-03-24

## 项目定位

feedloop 的**国内40+平台热榜**信息源。聚合微博/知乎/抖音/B站等平台实时热榜，支持 MCP 协议。

在 feedloop 架构中的角色：**4 源之一（上游采集层）**

```
TrendRadar → feedloop_hub.py → 统一表「信息中枢」→ 消费端
```

## 工作方式

- 覆盖：40+ 国内主流平台热榜
- 支持：RSS 订阅、MCP 协议、AI 多语言推送
- 输出：HTTP API（feedloop 定时拉取）

## 部署

| 项 | 值 |
|---|---|
| 运行方式 | Zeabur Docker |
| 配置文件 | `config/` 目录 |
| 文档 | README.md / DEPLOY_ZEABUR.md |

## 注意

- 这是开源项目的本地定制版本，上游：sansan0/TrendRadar
- 升级版本前参考 `DEPLOYMENT_CHECKLIST.md`
