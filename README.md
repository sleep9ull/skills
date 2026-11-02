# Skills for Observable Work

把任务进度和用户行为变成可核验的事实。

这是一个面向 Claude Code、Codex 等 coding agents 的 skills 集合。目前包含两个独立的工作流：用 **TaskDeck** 执行可恢复的长任务，并从启动时提供 HTML 监控页；用 **PostHog Instrumentation** 为已经实现的功能补充、调整和清理埋点。

每个 skill 都有独立的指令和按需读取的资源。可以单独安装，也可以根据项目约定调整使用。

## 快速安装

通过 [skills CLI](https://github.com/vercel-labs/skills) 安装，按提示选择需要的 skill 和目标 agent：

```bash
npx skills@latest add sleep9ull/skills
```

也可以只安装其中一个。

**Claude Code：**

```bash
npx skills@latest add sleep9ull/skills --skill task-deck -a claude-code
```

```bash
npx skills@latest add sleep9ull/skills --skill posthog-instrumentation -a claude-code
```

**Codex：**

```bash
npx skills@latest add sleep9ull/skills --skill task-deck -a codex
```

```bash
npx skills@latest add sleep9ull/skills --skill posthog-instrumentation -a codex
```

安装后，在 Claude Code 中使用 `/task-deck` 或 `/posthog-instrumentation`；在 Codex 中使用 `$task-deck` 或 `$posthog-instrumentation`。

检查和安装更新：

```bash
npx skills@latest check
```

```bash
npx skills@latest update
```

<details>
<summary>本地 checkout：使用符号链接安装</summary>

适合需要直接维护 skill 文件的情况。先克隆仓库：

```bash
git clone https://github.com/sleep9ull/skills.git
```

在仓库根目录执行以下命令。目标链接须尚不存在；若已安装同名 skill，先确认当前安装位置，避免重复安装。

```bash
mkdir -p "$HOME/.claude/skills"
```

```bash
ln -s "$PWD/skills/task-deck" "$HOME/.claude/skills/task-deck"
```

```bash
ln -s "$PWD/skills/posthog-instrumentation" "$HOME/.claude/skills/posthog-instrumentation"
```

Codex 安装时，将目标目录 `.claude/skills` 替换为 `.codex/skills`。保留 checkout；移动它后需要更新链接。本地文件修改会通过链接生效，获取远端更新需自行同步 checkout。

</details>

## 这两个 skills 解决什么问题？

### 长任务开始了，但不知道进度，也不知道能否恢复

[`task-deck`](skills/task-deck/SKILL.md) 把长任务组织为有计划、有检查点、可核验的后台作业。

它要求在启动时就交付监控 HTML，再按实际执行更新进度；记录批次、结果、异常和进程句柄，恢复前核验旧 worker 与检查点。结束时对照验收条件交付结果，并区分完成、失败和待人工处理。

```text
/task-deck 按目录分批处理这批文件，并发 3，保留原文件。
后台执行，启动时给我 HTML 监控页，支持中断后恢复。
```

附带的渲染器使用 Python 标准库，生成可离线打开的 HTML。它只负责展示；worker、进程监管和状态更新需要由 agent 根据实际任务接入。任务和监控能否在会话结束后继续运行，取决于实际执行环境及后台载体。

TaskDeck 用于用户明确要求的长任务，不因为任务耗时就自动接管。调用 skill 本身不增加外部写入、通知或付费授权。

### 埋点补上了，但事件含义不准，甚至改动了业务逻辑

[`posthog-instrumentation`](skills/posthog-instrumentation/SKILL.md) 从已经实现的功能代码、协议和真实调用链出发，补充与维护 PostHog events/properties。

它覆盖首次补埋点、功能调整后的埋点同步、漏报与重复上报检查，以及旧事件清理。流程包括行为盘点、事件与属性契约、测试、最小埋点实现及逐段 diff 自审。

```text
/posthog-instrumentation 检查这个页面已实现的行为，补齐 PostHog 事件和属性，
清理失效或重复的采集。保持功能实现和业务接口原样，说明无法准确观测的缺口。
```

**最高原则：只调整 PostHog 相关内容，不修改功能实现或业务接口。** 不为拿到属性扩业务回调，不把点击或请求发起当成成功，也不为完善漏斗改业务状态机。现有信号不足时，保留明确的观测缺口。

该 skill 按目标仓库的技术栈、架构和测试约定执行。协议声明需要结合实际实现核实；数据查询、报表和 dashboard 配置不属于此 skill 的职责。

## 技能索引

| Skill | 使用场景 | 主要资源 |
|---|---|---|
| [task-deck](skills/task-deck/SKILL.md) | 显式指定的批量处理、采集、迁移、生成和分析长任务 | [状态契约](skills/task-deck/references/status-contract.md)、[执行与恢复](skills/task-deck/references/execution.md)、[界面规范](skills/task-deck/references/interface.md)、[HTML 渲染器](skills/task-deck/scripts/render_dashboard.py) |
| [posthog-instrumentation](skills/posthog-instrumentation/SKILL.md) | 对已实现代码进行 PostHog 埋点新增、调整、审计和清理 | [改动边界](skills/posthog-instrumentation/references/change-boundary.md)、[统计语义](skills/posthog-instrumentation/references/measurement-patterns.md) |

## 本地验证

在仓库根目录运行 TaskDeck 渲染器测试，无需安装第三方 Python 依赖：

```bash
python3 -I -B -m unittest discover -s skills/task-deck/tests -v
```

PostHog skill 的评测输入见 [evaluation-cases.json](skills/posthog-instrumentation/references/evaluation-cases.json)，结果与局限见 [evaluation-notes.md](skills/posthog-instrumentation/references/evaluation-notes.md)。这些是隔离补丁提案的评测，不能代替实际项目构建、业务测试或 PostHog 接收验收。

任务运行数据、凭据、浏览器记录、缓存及生成的监控页面不写入本仓库。
