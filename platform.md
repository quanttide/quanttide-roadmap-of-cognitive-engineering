# 纯 CLI 管线路线图

## 设计原则

- 每个子命令是一个独立 filter：stdin → 处理 → stdout，不做持久化
- 数据格式统一为 JSON/YAML，命令之间通过 Unix pipe 组合
- 无 REPL、无 TUI、无 SQLite、无状态
- 用户自行决定输出重定向和版本管理
- Project 11 是基座，qtcloud-think 的功能作为新子命令加入

## 数据流

```
原始想法（stdin 文本）
  │ pipe
  ▼
collect       # 原始文本 → 结构化想法 JSON（id/title/description/tags）
  │ pipe
  ▼
clarify       # 结构化想法 → 扩充为 Situation + Intention JSON
  │ pipe
  ▼
distill       # Situation/Intention JSON → gallery 格式 YAML（stdout）
  │ tee
  ▼
本地 YAML 文件 → 人工审核 → git commit → gallery

---

qtthink report 2026-W24                    # 输出 Markdown 到 stdout
qtthink relate 2026-W24                    # 输出 JSON 关系到 stdout
qtthink tension 2026-W24                   # 输出 JSON 张量到 stdout
qtthink diff 2026-W24 2026-W25             # 输出 Markdown diff 到 stdout
```

所有输出命令不写文件，由用户 `>` 重定向。

## 阶段 0：重构 Project 11 为纯 CLI

| 子命令 | 行为 | 依赖 |
|--------|------|------|
| `report <week>` | 输出 Markdown 到 stdout，不写 reports/ 目录 | 现有 report.rs 去掉文件写入 |
| `relate <week>` | 输出 JSON 关系到 stdout | 现有 relate_llm 去掉文件缓存 |
| `tension <week>` | 输出 JSON 张量到 stdout | 现有 tension |
| `diff <a> <b>` | 输出 Markdown 到 stdout | 现有 diff |
| `schemas <week>` | 输出 JSON 图式到 stdout | 现有 list_schemas |
| `weeks` | 输出可用周列表到 stdout | 现有 |
| -v / --version | 输出版本信息 | 新增 |

gallery 路径通过 `--gallery` 参数或 `GALLERY_PATH` 环境变量传入，不再支持内置默认值。

**子命令统一接口**：成功时出口码 0，正常结果输出到 stdout；错误和日志信息输出到 stderr（`eprintln!`），不影响 pipe。

## 阶段 1：增加 collect → clarify → distill 管线

| 子命令 | 行为 | 输入 | 输出 |
|--------|------|------|------|
| `collect` | 读 stdin 原始文本，输出结构化想法 JSON（id/title/description） | 文本 | JSON |
| `clarify` | 读 JSON，调用 LLM 扩充为 Situation + Intention + Schema | JSON | JSON |
| `distill` | 读 JSON，输出 gallery 格式 YAML | JSON | YAML |

使用方式：

```bash
echo "今天和客户 battle 了一下午，取舍真的很重要" \
  | qtthink collect \
  | qtthink clarify \
  | qtthink distill \
  > staging/2026-W24/business.yaml
```

`clarify` 是唯一需要 LLM 的子命令（它负责 agenda/ecology/frame/dynamics 的生成）。`collect` 和 `distill` 不需要 LLM，纯数据处理。

## 阶段 2：整理管线

| 子命令 | 行为 |
|--------|------|
| `organize` | 读 stdin JSON + 已有的 YAML 目录，输出跨领域关系（复用 report.rs 的关键词+实体匹配逻辑） |

```bash
cat staging/2026-W24/*.yaml | qtthink organize --gallery gallery/
```

## 阶段 3：删除项

| 删除内容 | 原因 |
|---------|------|
| `repl.rs` | 纯 CLI 不再需要交互式循环 |
| `reports/` 目录写入逻辑 | 输出全部走 stdout |
| `relate_llm` / `ri` 的文件缓存 | 用户自己的 shell 做缓存（`tee`） |
| `rusqlite` 依赖 | 无状态 |
| `ratatui` / `crossterm` 依赖 | 无 TUI |
| 内置 gallery 默认路径 | 必须显式指定 |

## 子模块定位

合并后 gallery 成为纯数据仓库——系统通过 `distill` 输出 YAML 到 stdout，用户自己写到本地 staging 目录，人工审核后 git push。系统不再直接操作仓库。九宫格中 data/insight、data/intention、data/report 的内容由系统通过 `collect-clarify-distill` 管线自动生成并输出，用户决定是否和如何归档。

## 时间线

- 阶段 0：移除 REPL、改为 clap 子命令、统一 stdout/stderr 规范。可并行进行，1-2 天
- 阶段 1：collect/clarify/distill 三个子命令。clarify 需要 LLM 调用（复用现有 llm.rs），纯数据逻辑 1-2 天
- 阶段 2：organize 子命令，复用现有关系计算逻辑，半天
- 阶段 3：删除多余依赖和基础设施，随时可做
