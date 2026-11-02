# Skills for AI-Native PM (Full-Stack)

给有研发和代码落地能力的产品经理用的技能包。

面向使用 coding agent 推进产品落地的 AI-native PM，让 AI 帮你把任务跑完、把产品行为看清楚。

- **适用工具**：Claude Code、Codex 等 coding agent。
- **使用方式**：每个 skill 都有独立指令和按需读取的资源，可单独安装，也可按项目约定调整。

## Skills：

- **TaskDeck**：执行可恢复的长任务，从启动时就提供 HTML 监控页。
- **PostHog Instrumentation**：为已经实现的功能补充、调整和清理埋点。

## 快速安装

通过 [skills CLI](https://github.com/vercel-labs/skills) 安装，按提示选择需要的 skill 和目标 agent：

```bash
npx skills@latest add sleep9ull/skills
```

也可以只安装其中一个。

### Claude Code

```bash
npx skills@latest add sleep9ull/skills --skill task-deck -a claude-code
```

```bash
npx skills@latest add sleep9ull/skills --skill posthog-instrumentation -a claude-code
```

### Codex

```bash
npx skills@latest add sleep9ull/skills --skill task-deck -a codex
```

```bash
npx skills@latest add sleep9ull/skills --skill posthog-instrumentation -a codex
```

### 调用技能

- **Claude Code**：使用 `/task-deck` 或 `/posthog-instrumentation`。
- **Codex**：使用 `$task-deck` 或 `$posthog-instrumentation`。

### 检查更新

```bash
npx skills@latest check
```

```bash
npx skills@latest update
```

<details>
<summary>本地安装</summary>

适合需要直接维护 skill 文件的情况。先克隆仓库：

```bash
git clone https://github.com/sleep9ull/skills.git
```

在仓库根目录执行以下命令：

```bash
mkdir -p "$HOME/.claude/skills"
```

```bash
ln -s "$PWD/skills/task-deck" "$HOME/.claude/skills/task-deck"
```

```bash
ln -s "$PWD/skills/posthog-instrumentation" "$HOME/.claude/skills/posthog-instrumentation"
```

使用时注意：

- 目标链接须尚不存在。若已安装同名 skill，先确认当前安装位置，避免重复安装。
- Codex 安装时，将目标目录 `.claude/skills` 替换为 `.codex/skills`。
- 保留本地仓库目录；移动后需要更新链接。
- 本地文件修改会通过链接生效，远端更新需自行同步到本地仓库。

</details>

## 有哪些技能

### TaskDeck：跑长任务

[`task-deck`](skills/task-deck/SKILL.md) 把长任务组织为有计划、有检查点、可核验的后台作业。

- **启动即看进度**：交付 HTML 监控页，按实际执行情况更新。
- **中断后可恢复**：记录批次、结果、异常和进程句柄，恢复前核验旧 worker 与检查点。
- **结束时有交代**：对照验收条件交付结果，标明完成、失败和待人工处理的部分。

```text
/task-deck 按目录分批处理这批文件，并发 3，保留原文件。
后台执行，启动时给我 HTML 监控页，支持中断后恢复。
```

使用边界：

- 附带的渲染器使用 Python 标准库，生成可离线打开的 HTML。
- 渲染器只负责展示，worker、进程监管和状态更新需由 agent 根据实际任务接入。
- 任务和监控能否在会话结束后继续运行，取决于实际执行环境及后台载体。
- 仅在用户明确要求时使用，不因任务耗时就自动接管。
- 调用 skill 本身不增加外部写入、通知或付费授权。

### PostHog：补业务埋点

[`posthog-instrumentation`](skills/posthog-instrumentation/SKILL.md) 从已经实现的功能代码、协议和真实调用链出发，补充与维护 PostHog events/properties。

- **首次补埋点**：盘点已实现的行为，定义事件与属性契约，补充采集。
- **跟随功能调整**：同步事件和属性，清理失效的旧事件。
- **检查采集质量**：排查漏报与重复上报，核对事件含义和触发时机。
- **验证改动**：按目标仓库的技术栈、架构和测试约定执行测试，并逐段自审 diff。

```text
/posthog-instrumentation 检查这个页面已实现的行为，补齐 PostHog 事件和属性，
清理失效或重复的采集。保持功能实现和业务接口原样，说明无法准确观测的缺口。
```

**最高原则：只调整 PostHog 相关内容，不修改功能实现或业务接口。**

- 不为拿到属性扩业务回调。
- 不把点击或请求发起当成业务成功。
- 不为完善漏斗改业务状态机。
- 现有信号不足时，保留明确的观测缺口。
- 协议声明需要结合实际实现核实。
- 数据查询、报表和 dashboard 配置不属于此 skill 的职责。

## 资源索引

| Skill | 使用场景 | 主要资源 |
|---|---|---|
| [TaskDeck](skills/task-deck/SKILL.md) | 显式指定的批量处理、采集、迁移、生成和分析长任务 | [状态契约](skills/task-deck/references/status-contract.md)、[执行与恢复](skills/task-deck/references/execution.md)、[界面规范](skills/task-deck/references/interface.md)、[HTML 渲染器](skills/task-deck/scripts/render_dashboard.py) |
| [PostHog Instrumentation](skills/posthog-instrumentation/SKILL.md) | 对已实现代码进行 PostHog 埋点新增、调整、审计和清理 | [改动边界](skills/posthog-instrumentation/references/change-boundary.md)、[统计语义](skills/posthog-instrumentation/references/measurement-patterns.md) |

## 本地验证

### TaskDeck 测试

在仓库根目录运行渲染器测试，无需安装第三方 Python 依赖：

```bash
python3 -I -B -m unittest discover -s skills/task-deck/tests -v
```

### PostHog 评测

- **评测输入**：[evaluation-cases.json](skills/posthog-instrumentation/references/evaluation-cases.json)。
- **结果与局限**：[evaluation-notes.md](skills/posthog-instrumentation/references/evaluation-notes.md)。
- **评测范围**：这些是隔离补丁提案的评测，不能代替实际项目构建、业务测试或 PostHog 接收验收。

### 仓库约定

以下内容不写入本仓库：

- 任务运行数据
- 凭据
- 浏览器记录
- 缓存
- 生成的监控页面
