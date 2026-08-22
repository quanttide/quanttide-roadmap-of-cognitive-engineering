# provider scope

## 范围

思维处理的服务端——AI 加工接口（清晰度判断、澄清、结构化），支撑 cli 与 studio。

## 当前状态

- Python（FastAPI + openai + httpx，uv 管理）
- 与 qtcloud-agent provider（Go）等其他产品云 provider 模式不一致

## 待办：重构为 Go

- **对齐产品云 provider 模式**（qtcloud-agent/qtcloud-health 的 Go 实现：cmd/server + internal 分层）
- 存储抽象（local/OSS）与多端一致
- 4D 认知过程的服务化（感知 → 理解 → 预测 → 决策/行动 的 API 面）
