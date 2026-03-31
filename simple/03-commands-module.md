# 03 - Commands Module 深度分析

## 1. 目录结构概览

### 1.1 物理布局

```
src/commands/
├── commands.ts                    # 命令注册中心 (758行)
├── types/command.ts               # 命令类型定义 (217行)
│
├── # === 会话管理类 ===
├── clear/                         # 清除对话历史
│   ├── index.ts                   # 命令元数据
│   ├── clear.ts                   # 实现
│   ├── caches.ts                  # 缓存清理
│   └── conversation.ts            # 对话清理
├── session/                       # 远程会话URL/QR码
├── resume/                        # 恢复历史对话
├── rename/                        # 重命名会话
│   ├── index.ts
│   ├── rename.ts
│   └── generateSessionName.ts
├── compact/                       # 压缩上下文(保留摘要)
├── exit/                          # 退出REPL
├── copy/                          # 复制最后消息
├── export/                        # 导出会话
├── rewind/                        # 回退对话
├── tag/                           # 标签管理
├── share/                         # 分享会话
│
├── # === 配置类 ===
├── config/                        # 配置面板
├── model/                         # 切换AI模型
├── theme/                         # 主题切换
├── permissions/                   # 工具权限管理
├── privacy-settings/              # 隐私设置
├── color/                         # Agent颜色
├── output-style/                  # 输出样式
├── keybindings/                   # 快捷键管理
├── advisor/                       # 顾问模型配置
├── sandbox-toggle/                # 沙箱开关
├── effort/                        # 努力程度设置
├── passes/                        # Passes配置
├── rate-limit-options/            # 速率限制选项
│
├── # === 工具/扩展类 ===
├── mcp/                           # MCP服务器管理
│   ├── index.ts
│   ├── mcp.ts
│   ├── addCommand.ts
│   └── xaaIdpCommand.ts
├── plugin/                        # 插件管理
│   ├── index.tsx
│   ├── plugin.tsx
│   └── parseArgs.ts
├── skills/                        # 技能列表
├── hooks/                         # Hook配置
├── reload-plugins/                # 重载插件
├── install-github-app/            # 安装GitHub App
│   ├── index.ts
│   ├── install-github-app.ts
│   └── setupGitHubActions.ts
├── install-slack-app/             # 安装Slack App
├── terminalSetup/                 # 终端安装
├── upgrade/                       # 升级检查
│
├── # === 代码操作类 ===
├── commit.ts                      # Git提交
├── commit-push-pr.ts              # 提交+推送+PR
├── branch/                        # 分支/派生对话
├── diff/                          # 差异查看
├── review.ts                      # PR审查
│   ├── review/
│   │   ├── reviewRemote.ts
│   │   └── ultrareviewEnabled.ts
├── context/                       # 上下文信息
├── files/                         # 文件列表
├── status/                        # 状态查看
├── stats/                         # 统计信息
├── cost/                          # 费用统计
├── usage/                         # 用量信息
│
├── # === 桥接/远程类 ===
├── bridge/                        # 远程控制连接
├── bridge-kick.ts                 # 桥接故障注入(内部)
├── desktop/                       # 桌面端续接
├── ide/                           # IDE集成
├── remote-env/                    # 远程环境配置
├── remote-setup/                  # 远程安装设置
├── teleport/                      # 远程会话传送
├── session/                       # 远程会话信息
│
├── # === 用户交互类 ===
├── help/                          # 帮助
├── feedback/                      # 反馈提交
├── login/                         # 登录
├── logout/                        # 登出
├── install.tsx                    # 安装向导
├── mobile/                        # 移动端QR码
├── stickers/                      # 贴纸
├── doctor/                        # 诊断工具
├── onboarding/                    # 新手引导
│
├── # === 高级功能 ===
├── agents/                        # Agent管理
├── chrome/                        # Chrome集成
├── voice/                         # 语音模式
├── x402/                          # 加密支付(USDC)
├── plan/                          # 计划模式
├── tasks/                         # 任务管理
├── memory/                        # 记忆管理
├── thinkback/                     # 回顾
├── thinkback-play/                # 回顾播放
│
├── # === 初始化类 ===
├── init.ts                        # CLAUDE.md初始化
├── init-verifiers.ts              # 验证器初始化
│
├── # === 内部/调试类 ===
├── version.ts                     # 版本信息
├── heapdump/                      # 堆转储
├── debug-tool-call/               # 调试工具调用
├── mock-limits/                   # 模拟限制
├── reset-limits/                  # 重置限制
├── break-cache/                   # 破坏缓存
├── ant-trace/                     # Ant Trace
├── perf-issue/                    # 性能问题
├── bughunter/                     # Bug猎手
├── security-review.ts             # 安全审查
├── release-notes/                 # 发行说明
├── extra-usage/                   # 额外用量
├── statusline.tsx                 # 状态行
├── vim/                           # Vim模式
├── ctx_viz/                       # 上下文可视化
│
├── # === 特性门控命令 ===
├── proactive/                     # PROACTIVE/KAIROS
├── brief/                         # KAIROS_BRIEF
├── assistant/                     # KAIROS
├── fork/                          # FORK_SUBAGENT
├── buddy/                         # BUDDY
├── workflows/                     # WORKFLOW_SCRIPTS
├── torch/                         # TORCH
├── peers/                         # UDS_INBOX
├── force-snip/                    # HISTORY_SNIP
├── subscribe-pr/                  # KAIROS_GITHUB_WEBHOOKS
│
└── # === 工厂/工具函数 ===
    └── createMovedToPluginCommand.ts  # 迁移到插件的命令工厂
```

### 1.2 统计

| 指标 | 数值 |
|------|------|
| 命令目录数 | 87 个子目录 |
| 根目录命令文件 | 17 个 (.ts/.tsx) |
| 总命令数 (COMMANDS数组) | ~70+ 内置命令 |
| 类型定义文件 | 1 个 (types/command.ts) |
| 注册中心文件 | 1 个 (commands.ts, 758行) |
| 分派执行文件 | 1 个 (processSlashCommand.tsx, 922行) |
| 特性门控命令 | ~13 个 |
| 内部专用命令 (INTERNAL_ONLY_COMMANDS) | ~28 个 |

---

## 2. 命令统一接口与类型系统

### 2.1 核心类型定义

所有命令统一实现 `Command` 联合类型，定义在 `/Users/zaxtyson/Documents/claude-code-analyze/claude-code/src/types/command.ts`：

```typescript
export type Command = CommandBase & (PromptCommand | LocalCommand | LocalJSXCommand)
```

这是一个** discriminated union **(可辨识联合类型)，通过 `type` 字段区分三种实现模式：

### 2.2 三种命令实现模式

| 模式 | type 值 | 签名 | 用途 | 典型代表 |
|------|---------|------|------|----------|
| **Prompt** | `'prompt'` | `getPromptForCommand(args, context) => ContentBlockParam[]` | 生成提示词注入到对话 | `/commit`, `/review`, `/init` |
| **Local** | `'local'` | `call(args, context) => Promise<LocalCommandResult>` | 纯逻辑执行，返回文本 | `/clear`, `/version`, `/advisor` |
| **Local-JSX** | `'local-jsx'` | `call(onDone, context, args) => Promise<ReactNode>` | 渲染交互式 Ink UI | `/config`, `/model`, `/help` |

### 2.3 CommandBase 公共属性

```typescript
export type CommandBase = {
  // === 身份标识 ===
  name: string                          // 命令名称 (如 "config")
  aliases?: string[]                    // 别名 (如 ["settings"])
  description: string                   // 简短描述
  argumentHint?: string                 // 参数提示 (如 "[model]")
  whenToUse?: string                    // 详细使用场景

  // === 可见性控制 ===
  isEnabled?: () => boolean             // 动态启用检查
  isHidden?: boolean | (() => boolean)  // 是否隐藏
  availability?: CommandAvailability[]  // 认证要求 ['claude-ai' | 'console']

  // === 安全控制 ===
  disableModelInvocation?: boolean      // 禁止模型调用
  userInvocable?: boolean               // 用户可调用
  isSensitive?: boolean                 // 参数脱敏

  // === 元数据 ===
  loadedFrom?: 'skills' | 'plugin' | 'managed' | 'bundled' | 'mcp'
  kind?: 'workflow'                     // 工作流标记
  immediate?: boolean                   // 立即执行(跳过队列)
  version?: string                      // 版本号
  userFacingName?: () => string         // 用户可见名称
}
```

### 2.4 三种模式的具体接口

**PromptCommand** - 提示词注入型：
```typescript
type PromptCommand = {
  type: 'prompt'
  progressMessage: string              // 进度提示
  contentLength: number                // 内容长度(用于token估算)
  source: SettingSource | 'builtin' | 'mcp' | 'plugin' | 'bundled'
  allowedTools?: string[]              // 额外工具权限
  model?: string                       // 指定模型
  effort?: EffortValue                 // 努力程度
  hooks?: HooksSettings                // 技能钩子
  context?: 'inline' | 'fork'          // 执行上下文
  agent?: string                       // fork时的agent类型
  paths?: string[]                     // 文件路径过滤
  pluginInfo?: { pluginManifest, repository }
  getPromptForCommand(args, context): Promise<ContentBlockParam[]>
}
```

**LocalCommand** - 纯逻辑型：
```typescript
type LocalCommand = {
  type: 'local'
  supportsNonInteractive: boolean
  load: () => Promise<{ call: LocalCommandCall }>
}
// LocalCommandCall: (args, context) => Promise<LocalCommandResult>
// LocalCommandResult: { type: 'text', value: string } | { type: 'compact', ... } | { type: 'skip' }
```

**LocalJSXCommand** - 交互UI型：
```typescript
type LocalJSXCommand = {
  type: 'local-jsx'
  load: () => Promise<{ call: LocalJSXCommandCall }>
}
// LocalJSXCommandCall: (onDone, context, args) => Promise<ReactNode>
// onDone: (result?, options?) => void  // 完成回调
```

---

## 3. 命令注册与调度机制

### 3.1 注册中心架构

注册中心位于 `/Users/zaxtyson/Documents/claude-code-analyze/claude-code/src/commands.ts`，采用**分层注册**策略：

```
                    ┌─────────────────────────────────────────┐
                    │            getCommands(cwd)             │
                    │    (主入口, 每次调用重新过滤)             │
                    └──────────────┬──────────────────────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              │                    │                    │
              ▼                    ▼                    ▼
    ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
    │  loadAllCommands│  │ meetsAvailability│  │ isCommandEnabled│
    │  (memoized)     │  │ (实时检查)        │  │ (实时检查)       │
    └────────┬────────┘  └────────┬────────┘  └────────┬────────┘
             │                    │                    │
    ┌────────┴────────┐           │                    │
    │  COMMANDS()     │           │                    │
    │  (memoized)     │           │                    │
    └────────┬────────┘           │                    │
             │                    │                    │
    ┌────────┴────────────────────┴────────────────────┘
    │
    ▼
┌───────────────────────────────────────────────────────────────┐
│                    命令来源层次                                 │
│                                                               │
│  1. bundledSkills      - 内置捆绑技能 (同步加载)                │
│  2. builtinPluginSkills - 内置插件技能                          │
│  3. skillDirCommands   - .claude/skills/ 目录扫描               │
│  4. workflowCommands   - 工作流脚本 (特性门控)                   │
│  5. pluginCommands     - 已安装插件                             │
│  6. pluginSkills       - 插件提供的技能                          │
│  7. COMMANDS()         - 内置命令 (~70个)                       │
│  8. dynamicSkills      - 运行时发现的动态技能                    │
└───────────────────────────────────────────────────────────────┘
```

### 3.2 注册流程

```
┌──────────────────────────────────────────────────────────────────────┐
│                        命令注册流程                                    │
│                                                                      │
│  模块加载时 (静态)                                                     │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │ 1. import 所有内置命令模块                                        │
│  │    - 每个模块 export default 一个 Command 对象                   │
│  │    - 只包含元数据, 不加载实现                                     │
│  │                                                                │  │
│  │ 2. 特性门控命令使用 require() 延迟加载                             │
│  │    const bridge = feature('BRIDGE_MODE')                        │  │
│  │      ? require('./commands/bridge/index.js').default            │  │
│  │      : null                                                     │  │
│  │                                                                │  │
│  │ 3. INTERNAL_ONLY_COMMANDS 数组 (仅 USER_TYPE=ant 可见)            │
│  │    - bughunter, mockLimits, version, teleport, etc.             │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  首次调用 getCommands(cwd) 时 (懒加载)                                 │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │ 1. COMMANDS() 被 memoize 缓存 (按 cwd)                           │
│  │ 2. loadAllCommands(cwd) 并行加载:                                │
│  │    - getSkills(cwd)     -> 技能目录 + 插件技能 + 捆绑技能          │
│  │    - getPluginCommands() -> 已安装插件命令                        │
│  │    - getWorkflowCommands -> 工作流命令                           │
│  │ 3. 合并所有来源, 按优先级排序                                      │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  每次调用 getCommands(cwd) 时 (实时过滤)                               │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │ 1. meetsAvailabilityRequirement(cmd)                            │
│  │    - 检查认证类型 (claude-ai / console)                          │
│  │    - 不 memoize, 因为登录状态可能变化                              │
│  │                                                                │  │
│  │ 2. isCommandEnabled(cmd)                                        │
│  │    - 调用 cmd.isEnabled?.() ?? true                             │
│  │    - 检查特性开关、环境变量、平台兼容性                             │
│  │                                                                │  │
│  │ 3. 插入 dynamicSkills (去重后)                                    │
│  └────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────┘
```

### 3.3 命令分派执行

分派逻辑在 `/Users/zaxtyson/Documents/claude-code-analyze/claude-code/src/utils/processUserInput/processSlashCommand.tsx`：

```
用户输入 /command args
        │
        ▼
┌───────────────────┐
│  parseSlashCommand │  解析命令名和参数
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│  hasCommand()      │  检查命令是否存在
└────────┬──────────┘
         │
    ┌────┴────┐
    │  不存在   │  → 返回 "Unknown skill" 错误
    └─────────┘
         │
         ▼ (存在)
┌───────────────────────────────────────────────────────┐
│              getMessagesForSlashCommand()              │
│                                                       │
│  1. 检查 userInvocable === false → 拒绝用户直接调用     │
│                                                       │
│  2. switch (command.type) {                            │
│                                                       │
│     case 'local-jsx':                                  │
│       ┌─────────────────────────────────────────────┐  │
│       │ 返回 Promise, 等待 onDone 回调               │  │
│       │ 1. command.load() → 懒加载实现模块            │  │
│       │ 2. mod.call(onDone, context, args) → 渲染UI  │  │
│       │ 3. setToolJSX({ jsx, isLocalJSXCommand })    │  │
│       │ 4. onDone(result, options) 触发完成           │  │
│       └─────────────────────────────────────────────┘  │
│                                                       │
│     case 'local':                                      │
│       ┌─────────────────────────────────────────────┐  │
│       │ 同步执行, 返回文本结果                         │  │
│       │ 1. command.load() → 懒加载                    │  │
│       │ 2. mod.call(args, context) → 执行逻辑         │  │
│       │ 3. 返回 { type: 'text', value }              │  │
│       │    或 { type: 'compact', compactionResult }   │  │
│       │    或 { type: 'skip' }                        │  │
│       └─────────────────────────────────────────────┘  │
│                                                       │
│     case 'prompt':                                     │
│       ┌─────────────────────────────────────────────┐  │
│       │ 提示词注入, 发送到模型                         │  │
│       │ 1. 如果 context === 'fork' → 子Agent执行      │  │
│       │ 2. 否则 → getPromptForCommand(args, context) │  │
│       │ 3. 生成 ContentBlockParam[] 注入对话           │  │
│       │ 4. shouldQuery: true → 触发模型响应            │  │
│       └─────────────────────────────────────────────┘  │
│  }                                                     │
└───────────────────────────────────────────────────────┘
```

---

## 4. 命令分类详解

### 4.1 完整命令分类表

| 分类 | 命令数 | 主要类型 | 代表命令 | 特征 |
|------|--------|----------|----------|------|
| **会话管理** | 12 | local, local-jsx | clear, resume, rename, compact, exit | 操作会话状态 |
| **配置管理** | 14 | local-jsx, local | config, model, theme, permissions | 修改运行时设置 |
| **工具/扩展** | 9 | local-jsx | mcp, plugin, skills, hooks | 管理扩展生态 |
| **代码操作** | 10 | prompt, local-jsx | commit, diff, branch, review | 操作代码/PR |
| **桥接/远程** | 7 | local-jsx | bridge, desktop, ide, remote-env | 跨设备/环境 |
| **用户交互** | 9 | local-jsx | help, feedback, login, logout | 用户服务 |
| **高级功能** | 8 | local-jsx, local | agents, chrome, voice, x402 | 实验性/高级 |
| **安装部署** | 4 | local-jsx, local | install, install-github-app | 安装向导 |
| **内部调试** | 15 | local | version, heapdump, mock-limits | 仅 ant 用户 |
| **特性门控** | 13 | 混合 | proactive, fork, buddy | 按 feature flag |
| **初始化** | 2 | prompt | init, init-verifiers | 项目设置 |

### 4.2 各类别详细分析

#### 4.2.1 会话管理类

```
/clear      → type: 'local'   → 清除对话历史, 支持别名 ['reset', 'new']
/session    → type: 'local-jsx' → 显示远程会话URL和QR码, 仅远程模式可见
/resume     → type: 'local-jsx' → 恢复历史对话, 别名 ['continue']
/rename     → type: 'local-jsx' → 重命名当前会话, immediate: true
/compact    → type: 'local'   → 压缩上下文(保留摘要), 支持自定义摘要指令
/exit       → type: 'local-jsx' → 退出REPL, 别名 ['quit'], immediate: true
/copy       → type: 'local-jsx' → 复制最后一条消息
/export     → type: 'local-jsx' → 导出会话数据
/rewind     → type: 'local-jsx' → 回退到之前的对话点
/tag        → type: 'local-jsx' → 会话标签管理
/share      → type: 'local'   → 分享会话链接 (内部命令)
/summary    → type: 'local'   → 生成会话摘要 (内部命令)
```

**设计特点**：
- `/clear` 采用**懒加载模式**：index.ts 只包含元数据，实现从 clear.ts 动态导入
- `/rename` 标记 `immediate: true`，跳过命令队列立即执行
- `/session` 使用 `isEnabled` + `isHidden` 双重门控，仅在远程模式显示

#### 4.2.2 配置管理类

```
/config           → type: 'local-jsx' → 打开配置面板, 别名 ['settings']
/model            → type: 'local-jsx' → 切换AI模型, 动态描述(显示当前模型)
/theme            → type: 'local-jsx' → 切换主题
/permissions      → type: 'local-jsx' → 管理工具允许/拒绝规则, 别名 ['allowed-tools']
/privacy-settings → type: 'local-jsx' → 隐私设置, 仅消费者订阅可用
/color            → type: 'local-jsx' → 设置Agent颜色
/output-style     → type: 'local-jsx' → 输出样式配置
/keybindings      → type: 'local-jsx' → 快捷键管理
/advisor          → type: 'local'     → 配置顾问模型, 支持 on/off
/sandbox-toggle   → type: 'local-jsx' → 沙箱模式开关
/effort           → type: 'local-jsx' → 努力程度设置
/passes           → type: 'local-jsx' → Passes配置
/rate-limit-options → type: 'local-jsx' → 速率限制选项
```

**设计特点**：
- `/model` 使用 **getter 属性** 实现动态描述：`get description() { return \`当前 ${renderModelName(...)}\` }`
- `/model` 支持 `immediate` 属性，根据推理配置决定是否立即执行
- `/privacy-settings` 使用 `isEnabled: () => isConsumerSubscriber()` 进行订阅检查

#### 4.2.3 工具/扩展类

```
/mcp            → type: 'local-jsx' → 管理MCP服务器, immediate: true
/plugin         → type: 'local-jsx' → 管理插件, 别名 ['plugins', 'marketplace']
/skills         → type: 'local-jsx' → 列出可用技能
/hooks          → type: 'local-jsx' → 查看Hook配置, immediate: true
/reload-plugins → type: 'local-jsx' → 重新加载插件
/install-github-app → type: 'local-jsx' → 安装GitHub Actions
/install-slack-app  → type: 'local'     → 安装Slack应用
/terminalSetup  → type: 'local-jsx' → 终端安装向导
/upgrade        → type: 'local-jsx' → 检查升级
```

**插件参数解析器** (`plugin/parseArgs.ts`) 展示了复杂的子命令解析：
```typescript
type ParsedCommand =
  | { type: 'menu' }
  | { type: 'install'; marketplace?: string; plugin?: string }
  | { type: 'manage' }
  | { type: 'uninstall'; plugin?: string }
  | { type: 'enable' | 'disable'; plugin?: string }
  | { type: 'validate'; path?: string }
  | { type: 'marketplace'; action?: 'add' | 'remove' | 'update' | 'list'; target?: string }
```

#### 4.2.4 代码操作类

```
/commit         → type: 'prompt'  → 创建git提交, 注入安全协议提示词
/commit-push-pr → type: 'prompt'  → 提交+推送+创建PR
/branch         → type: 'local-jsx' → 创建对话分支, 别名 ['fork']
/diff           → type: 'local-jsx' → 查看未提交变更和逐轮差异
/review         → type: 'prompt'  → PR代码审查
/ultrareview    → type: 'local-jsx' → 远程Bug审查 (仅当 ultrareview 启用)
/context        → type: 'local-jsx' → 显示上下文信息
/files          → type: 'local-jsx' → 列出跟踪的文件
/status         → type: 'local-jsx' → 显示状态
/stats          → type: 'local-jsx' → 统计信息
/cost           → type: 'local-jsx' → 会话费用统计
/usage          → type: 'local-jsx' → 用量信息
```

**`/commit` 命令分析** - 典型的 Prompt 类型命令：
```typescript
// 1. 声明允许的工具白名单
const ALLOWED_TOOLS = ['Bash(git add:*)', 'Bash(git status:*)', 'Bash(git commit:*)']

// 2. 构建包含安全协议的提示词
function getPromptContent(): string {
  return `## Git Safety Protocol
  - NEVER update the git config
  - NEVER skip hooks (--no-verify, --no-gpg-sign)
  - ALWAYS create NEW commits. NEVER use git commit --amend
  ...`
}

// 3. 执行时动态注入权限并运行shell命令
async getPromptForCommand(_args, context) {
  const finalContent = await executeShellCommandsInPrompt(
    promptContent,
    { ...context, getAppState() { /* 临时提升权限 */ } },
    '/commit',
  )
  return [{ type: 'text', text: finalContent }]
}
```

#### 4.2.5 桥接/远程类

```
/bridge (remote-control) → type: 'local-jsx' → 远程控制连接, 别名 ['rc']
/bridge-kick             → type: 'local'     → 注入桥接故障(内部测试)
/desktop                 → type: 'local-jsx' → 在Claude Desktop续接会话
/ide                     → type: 'local-jsx' → 管理IDE集成
/remote-env              → type: 'local-jsx' → 配置默认远程环境
/remote-setup (web-setup)→ type: 'local-jsx' → 设置Claude Code on the Web
/teleport                → type: 'local'     → 远程会话传送 (内部)
```

**桥接安全机制** (`commands.ts` 中的桥接安全命令白名单)：
```typescript
// 远程模式安全命令 (仅影响本地TUI状态)
export const REMOTE_SAFE_COMMANDS: Set<Command> = new Set([
  session, exit, clear, help, theme, color, vim, cost, usage,
  copy, btw, feedback, plan, keybindings, statusline, stickers, mobile,
])

// 桥接安全命令 (可从移动端执行)
export const BRIDGE_SAFE_COMMANDS: Set<Command> = new Set([
  compact, clear, cost, summary, releaseNotes, files,
])

// 桥接安全检查函数
export function isBridgeSafeCommand(cmd: Command): boolean {
  if (cmd.type === 'local-jsx') return false   // JSX命令总是被阻止
  if (cmd.type === 'prompt') return true       // prompt命令总是安全
  return BRIDGE_SAFE_COMMANDS.has(cmd)         // local命令查白名单
}
```

#### 4.2.6 用户交互类

```
/help       → type: 'local-jsx' → 显示帮助和可用命令
/feedback   → type: 'local-jsx' → 提交反馈, 别名 ['bug']
/login      → type: 'local-jsx' → 登录Anthropic账户
/logout     → type: 'local-jsx' → 登出
/install    → type: 'local-jsx' → 安装向导 (39KB, 大型JSX组件)
/mobile     → type: 'local-jsx' → 显示移动端下载QR码, 别名 ['ios', 'android']
/stickers   → type: 'local-jsx' → 贴纸功能
/doctor     → type: 'local-jsx' → 诊断工具
/onboarding → type: 'local'     → 新手引导 (内部)
```

**`/feedback` 命令的复杂启用条件**：
```typescript
isEnabled: () => !(
  isEnvTruthy(process.env.CLAUDE_CODE_USE_BEDROCK) ||   // 排除Bedrock
  isEnvTruthy(process.env.CLAUDE_CODE_USE_VERTEX) ||    // 排除Vertex
  isEnvTruthy(process.env.CLAUDE_CODE_USE_FOUNDRY) ||   // 排除Foundry
  isEnvTruthy(process.env.DISABLE_FEEDBACK_COMMAND) ||
  isEnvTruthy(process.env.DISABLE_BUG_COMMAND) ||
  isEssentialTrafficOnly() ||                           // 隐私级别
  process.env.USER_TYPE === 'ant' ||                    // 排除内部用户
  !isPolicyAllowed('allow_product_feedback')            // 策略检查
)
```

#### 4.2.7 高级功能类

```
/agents   → type: 'local-jsx' → 管理Agent配置
/chrome   → type: 'local-jsx' → Chrome集成设置, 仅 claude-ai
/mobile   → type: 'local-jsx' → 移动端QR码
/voice    → type: 'local'     → 语音模式开关, 仅 claude-ai
/x402     → type: 'local'     → 加密支付(USDC on Base), 别名 ['wallet', 'pay']
/plan     → type: 'local-jsx' → 计划模式
/tasks    → type: 'local-jsx' → 任务管理
/memory   → type: 'local-jsx' → 记忆管理
/thinkback→ type: 'local-jsx' → 回顾功能
```

#### 4.2.8 特性门控命令

这些命令使用 `feature()` 函数进行编译时和运行时双重门控：

```typescript
// 编译时: 特性不存在时, 模块不会被打包
const proactive = feature('PROACTIVE') || feature('KAIROS')
  ? require('./commands/proactive.js').default
  : null

// 运行时: 特性存在但运行时关闭, 命令不加入列表
const bridge = feature('BRIDGE_MODE')
  ? require('./commands/bridge/index.js').default
  : null

// 动态添加到 COMMANDS 数组
...(proactive ? [proactive] : []),
...(bridge ? [bridge] : []),
...(voiceCommand ? [voiceCommand] : []),
...(forkCmd ? [forkCmd] : []),
```

| 特性标志 | 命令 | 说明 |
|----------|------|------|
| `PROACTIVE` / `KAIROS` | proactive | 主动式任务 |
| `KAIROS_BRIEF` | brief | 简报功能 |
| `KAIROS` | assistant | 助手模式 |
| `BRIDGE_MODE` | bridge, remoteControlServer | 远程控制 |
| `VOICE_MODE` | voice | 语音模式 |
| `FORK_SUBAGENT` | fork | 子Agent派生 |
| `BUDDY` | buddy | 伙伴系统 |
| `WORKFLOW_SCRIPTS` | workflows | 工作流脚本 |
| `TORCH` | torch | Torch功能 |
| `UDS_INBOX` | peers | UDS收件箱 |
| `HISTORY_SNIP` | force-snip | 历史裁剪 |
| `KAIROS_GITHUB_WEBHOOKS` | subscribe-pr | PR订阅 |
| `CCR_REMOTE_SETUP` | web-setup | Web远程设置 |
| `EXPERIMENTAL_SKILL_SEARCH` | (技能搜索缓存) | 实验性技能搜索 |

---

## 5. 命令生命周期

### 5.1 完整生命周期图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          命令生命周期                                        │
│                                                                             │
│  阶段1: 模块加载 (应用启动)                                                   │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  src/commands.ts 被导入                                                │  │
│  │  ├── 静态 import 所有内置命令模块                                       │  │
│  │  │   └── 每个模块导出 Command 元数据对象                                 │  │
│  │  │       └── 不执行 load(), 不加载实现                                   │  │
│  │  ├── 特性门控命令: require() 条件加载                                    │  │
│  │  └── INTERNAL_ONLY_COMMANDS 数组 (仅 ant 用户)                          │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  阶段2: 命令解析 (用户输入 /command)                                          │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  parseSlashCommand(inputString)                                        │  │
│  │  ├── 提取 commandName 和 args                                          │  │
│  │  ├── 检测 MCP 命令 (mcp:prefix format)                                 │  │
│  │  └── 返回 { commandName, args, isMcp }                                 │  │
│  │                                                                        │  │
│  │  hasCommand(commandName, commands)                                     │  │
│  │  ├── 检查 command.name === commandName                                 │  │
│  │  ├── 检查 command.aliases?.includes(commandName)                       │  │
│  │  └── 检查 command.userFacingName?.() === commandName                   │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  阶段3: 权限检查 (执行前)                                                     │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  1. meetsAvailabilityRequirement(cmd)                                  │  │
│  │     ├── 无 availability → 通过 (通用命令)                               │  │
│  │     ├── 'claude-ai' → 检查 isClaudeAISubscriber()                      │  │
│  │     └── 'console' → 检查非3P + 1P API                                  │  │
│  │                                                                        │  │
│  │  2. isCommandEnabled(cmd)                                              │  │
│  │     └── cmd.isEnabled?.() ?? true                                      │  │
│  │                                                                        │  │
│  │  3. command.userInvocable 检查                                          │  │
│  │     └── false → 拒绝用户直接调用 (仅模型可调用)                           │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  阶段4: 命令执行 (按类型分派)                                                 │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  switch (command.type) {                                               │  │
│  │                                                                        │  │
│  │  case 'local-jsx':                                                     │  │
│  │    ┌─────────────────────────────────────────────────────────────┐    │  │
│  │    │ 1. command.load() → Promise<LocalJSXCommandModule>          │    │  │
│  │    │    └── import('./command.js')  // 动态导入实现                │    │  │
│  │    │                                                             │    │  │
│  │    │ 2. mod.call(onDone, context, args) → Promise<ReactNode>     │    │  │
│  │    │    └── 渲染 Ink UI 组件                                      │    │  │
│  │    │                                                             │    │  │
│  │    │ 3. setToolJSX({ jsx, isLocalJSXCommand: true })             │    │  │
│  │    │    └── 将 JSX 组件设置到 TUI 显示层                           │    │  │
│  │    │                                                             │    │  │
│  │    │ 4. 用户交互完成后: onDone(result, options)                   │    │  │
│  │    │    ├── display: 'skip' → 不添加消息                          │    │  │
│  │    │    ├── display: 'system' → 系统消息                          │    │  │
│  │    │    └── display: 'user' → 用户消息 (默认)                     │    │  │
│  │    │        ├── shouldQuery: true → 触发模型响应                   │    │  │
│  │    │        ├── metaMessages → 隐藏元数据消息                      │    │  │
│  │    │        ├── nextInput → 预填输入                               │    │  │
│  │    │        └── submitNextInput → 自动提交                        │    │  │
│  │    └─────────────────────────────────────────────────────────────┘    │  │
│  │                                                                        │  │
│  │  case 'local':                                                         │  │
│  │    ┌─────────────────────────────────────────────────────────────┐    │  │
│  │    │ 1. command.load() → Promise<LocalCommandModule>             │    │  │
│  │    │    └── import('./command.js') 或 Promise.resolve({ call })  │    │  │
│  │    │                                                             │    │  │
│  │    │ 2. mod.call(args, context) → Promise<LocalCommandResult>    │    │  │
│  │    │    ├── { type: 'text', value: string }                      │    │  │
│  │    │    ├── { type: 'compact', compactionResult, displayText }   │    │  │
│  │    │    └── { type: 'skip' }                                     │    │  │
│  │    │                                                             │    │  │
│  │    │ 3. 结果包装为消息:                                            │    │  │
│  │    │    └── <local-command-stdout>${result.value}</...>          │    │  │
│  │    └─────────────────────────────────────────────────────────────┘    │  │
│  │                                                                        │  │
│  │  case 'prompt':                                                        │  │
│  │    ┌─────────────────────────────────────────────────────────────┐    │  │
│  │    │ 1. 如果 command.context === 'fork':                          │    │  │
│  │    │    └── executeForkedSlashCommand() → 子Agent执行              │    │  │
│  │    │        ├── prepareForkedCommandContext()                     │    │  │
│  │    │        ├── runAgent({ agentDefinition, promptMessages })    │    │  │
│  │    │        └── extractResultText(agentMessages)                  │    │  │
│  │    │                                                             │    │  │
│  │    │ 2. 否则 (inline 模式):                                        │    │  │
│  │    │    ┌─────────────────────────────────────────────────────┐  │    │  │
│  │    │    │ a. command.getPromptForCommand(args, context)       │  │    │  │
│  │    │    │    → Promise<ContentBlockParam[]>                   │  │    │  │
│  │    │    │                                                     │  │    │  │
│  │    │    │ b. 注册技能钩子 (command.hooks)                      │  │    │  │
│  │    │    │    └── registerSkillHooks(...)                      │  │    │  │
│  │    │    │                                                     │  │    │  │
│  │    │    │ c. 记录技能调用 (addInvokedSkill)                    │  │    │  │
│  │    │    │                                                     │  │    │  │
│  │    │    │ d. 解析额外工具权限 (command.allowedTools)            │  │    │  │
│  │    │    │                                                     │  │    │  │
│  │    │    │ e. 构建消息:                                         │  │    │  │
│  │    │    │    ├── 用户消息 (命令输入元数据)                       │  │    │  │
│  │    │    │    ├── 元消息 (提示词内容, isMeta: true)              │  │    │  │
│  │    │    │    ├── 附件消息 (@-mentions 解析)                     │  │    │  │
│  │    │    │    └── 权限消息 (allowedTools, model)                │  │    │  │
│  │    │    │                                                     │  │    │  │
│  │    │    │ f. shouldQuery: true → 发送到模型                     │  │    │  │
│  │    │    └─────────────────────────────────────────────────────┘  │    │  │
│  │    └─────────────────────────────────────────────────────────────┘    │  │
│  │  }                                                                    │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  阶段5: 执行完成 (后处理)                                                     │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  1. 遥测记录: logEvent('tengu_input_command', {...})                   │  │
│  │     ├── command_name, invocation_trigger                               │  │
│  │     ├── plugin_name, marketplace_name (如果是插件命令)                  │  │
│  │     └── skill_name, skill_source, skill_loaded_from                    │  │
│  │                                                                        │  │
│  │  2. 消息处理:                                                           │  │
│  │     ├── shouldQuery: true → 发送到模型, 等待响应                        │  │
│  │     ├── shouldQuery: false → 不触发模型响应                             │  │
│  │     └── nextInput / submitNextInput → 预填/自动提交                     │  │
│  │                                                                        │  │
│  │  3. UI 清理:                                                            │  │
│  │     └── setToolJSX({ clearLocalJSX: true }) // 清理命令UI              │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 5.2 懒加载机制

命令实现采用**懒加载模式**，通过 `load()` 函数延迟导入：

```typescript
// 元数据注册时 (轻量, 不加载实现)
const config = {
  type: 'local-jsx',
  name: 'config',
  description: 'Open config panel',
  load: () => import('./config.js'),  // 动态 import, 不立即执行
} satisfies Command

// 执行时 (按需加载)
const mod = await command.load()  // 此时才加载 config.js
const result = await mod.call(onDone, context, args)
```

**懒加载的优势**：
1. **启动加速** - 应用启动时只加载元数据，不加载 ~100 个命令的实现
2. **内存优化** - 未使用的命令不占用内存
3. **依赖隔离** - 每个命令的依赖项只在需要时加载
4. **错误隔离** - 单个命令加载失败不影响其他命令

**两种懒加载模式**：
```typescript
// 模式A: 动态 import (重型命令)
load: () => import('./heavy-command.js')

// 模式B: Promise.resolve (轻量命令, 实现已在内存中)
load: () => Promise.resolve({ call })
```

---

## 6. 权限控制机制

### 6.1 多层权限模型

```
┌─────────────────────────────────────────────────────────────────────┐
│                        权限控制层次结构                               │
│                                                                     │
│  Layer 1: 编译时过滤 (打包阶段)                                       │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  feature('XXX') 检查                                           │  │
│  │  ├── 特性不存在 → 代码被 tree-shaking 消除                      │  │
│  │  └── 特性存在 → 代码保留在包中                                   │  │
│  │                                                               │  │
│  │  USER_TYPE === 'ant' 检查                                      │  │
│  │  ├── 非 ant 用户 → INTERNAL_ONLY_COMMANDS 不包含                 │  │
│  │  └── ant 用户 → 包含内部命令                                     │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  Layer 2: 可用性过滤 (认证/提供者)                                    │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  cmd.availability 检查                                         │  │
│  │  ├── undefined → 通用命令, 所有用户可用                          │  │
│  │  ├── ['claude-ai'] → 仅 claude.ai 订阅者                        │  │
│  │  ├── ['console'] → 仅 Console API 密钥用户                      │  │
│  │  └── ['claude-ai', 'console'] → 两者都可用                      │  │
│  │                                                               │  │
│  │  meetsAvailabilityRequirement(cmd):                            │  │
│  │  ├── isClaudeAISubscriber() → OAuth 订阅检查                    │  │
│  │  ├── isUsing3PServices() → 排除 Bedrock/Vertex/Foundry          │  │
│  │  └── isFirstPartyAnthropicBaseUrl() → 1P API 检查               │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  Layer 3: 启用状态过滤 (运行时)                                       │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  isCommandEnabled(cmd) = cmd.isEnabled?.() ?? true             │  │
│  │                                                               │  │
│  │  常见检查模式:                                                  │  │
│  │  ├── 环境变量: !isEnvTruthy(process.env.DISABLE_XXX_COMMAND)   │  │
│  │  ├── 特性开关: isVoiceGrowthBookEnabled()                      │  │
│  │  ├── 订阅状态: isConsumerSubscriber()                          │  │
│  │  ├── 策略检查: isPolicyAllowed('allow_remote_sessions')        │  │
│  │  ├── 平台检查: process.platform === 'darwin'                   │  │
│  │  └── 模式检查: !getIsNonInteractiveSession()                   │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  Layer 4: 可见性控制 (UI层)                                          │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  cmd.isHidden 检查                                             │  │
│  │  ├── true → 从类型ahead和帮助中隐藏                              │  │
│  │  ├── () => boolean → 动态计算                                   │  │
│  │  └── undefined → 默认显示                                       │  │
│  │                                                               │  │
│  │  典型模式: isHidden 与 isEnabled 使用相同条件                    │  │
│  │  get isHidden() { return !isEnabled() }                        │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  Layer 5: 调用权限 (执行时)                                           │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  cmd.userInvocable 检查                                        │  │
│  │  ├── undefined / true → 用户可通过 /command 调用                 │  │
│  │  └── false → 仅模型可通过 SkillTool 调用                        │  │
│  │                                                               │  │
│  │  cmd.disableModelInvocation 检查                                │  │
│  │  ├── true → 模型不能调用 (仅用户)                                │  │
│  │  └── false / undefined → 模型可调用                              │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  Layer 6: 工具权限 (Prompt命令)                                      │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  cmd.allowedTools → 为命令执行额外授予工具权限                    │  │
│  │  ├── 格式: ['Bash(git add:*)', 'Read', 'Write']               │  │
│  │  └── 解析: parseToolListFromCLI(command.allowedTools)          │  │
│  │                                                               │  │
│  │  cmd.isSensitive → 参数脱敏                                     │  │
│  │  └── true → args 显示为 '***'                                  │  │
│  └───────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

### 6.2 权限检查示例

```typescript
// 示例1: 多层权限组合 - /voice
const voice = {
  type: 'local',
  name: 'voice',
  availability: ['claude-ai'],           // Layer 2: 仅订阅者
  isEnabled: () => isVoiceGrowthBookEnabled(),  // Layer 3: 特性开关
  get isHidden() {                       // Layer 4: 动态隐藏
    return !isVoiceModeEnabled()
  },
  supportsNonInteractive: false,
  load: () => import('./voice.js'),
}

// 示例2: 平台+认证组合 - /desktop
const desktop = {
  type: 'local-jsx',
  name: 'desktop',
  availability: ['claude-ai'],           // Layer 2: 仅订阅者
  isEnabled: isSupportedPlatform,        // Layer 3: 平台检查 (macOS/Win x64)
  get isHidden() {                       // Layer 4: 不支持平台时隐藏
    return !isSupportedPlatform()
  },
  load: () => import('./desktop.js'),
}

// 示例3: 策略+订阅组合 - /remote-env
const remoteEnv = {
  type: 'local-jsx',
  name: 'remote-env',
  isEnabled: () =>
    isClaudeAISubscriber() &&            // Layer 3: 订阅检查
    isPolicyAllowed('allow_remote_sessions'),  // Layer 3: 策略检查
  get isHidden() {                       // Layer 4: 任一条件不满足时隐藏
    return !isClaudeAISubscriber() || !isPolicyAllowed('allow_remote_sessions')
  },
  load: () => import('./remote-env.js'),
}
```

---

## 7. 命令与桥接层的交互

### 7.1 桥接架构

```
┌─────────────────────────────────────────────────────────────────────┐
│                        桥接命令流                                    │
│                                                                     │
│  移动端/Web客户端                                                    │
│       │                                                             │
│       ▼                                                             │
│  ┌─────────────┐                                                    │
│  │ Bridge Client│  发送命令输入                                      │
│  └──────┬──────┘                                                    │
│         │ WebSocket                                                 │
│         ▼                                                           │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    Bridge Server                             │    │
│  │  (Remote Control / CCR)                                      │    │
│  └──────────────┬──────────────────────────────────────────────┘    │
│                 │                                                   │
│                 ▼                                                   │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │              isBridgeSafeCommand(cmd)                        │    │
│  │                                                              │    │
│  │  if (cmd.type === 'local-jsx') return false  // 总是阻止      │    │
│  │  if (cmd.type === 'prompt') return true      // 总是允许      │    │
│  │  return BRIDGE_SAFE_COMMANDS.has(cmd)        // 白名单检查    │    │
│  └──────────────┬──────────────────────────────────────────────┘    │
│                 │                                                   │
│         ┌───────┴───────┐                                           │
│         │               │                                           │
│    允许执行        阻止执行                                          │
│         │               │                                           │
│         ▼               ▼                                           │
│  正常执行命令      返回错误: "Command not allowed from bridge"       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 7.2 桥接安全命令分类

| 分类 | 命令 | 桥接安全 | 原因 |
|------|------|----------|------|
| **远程安全** | session, exit, clear, help, theme, color, vim, cost, usage, copy, btw, feedback, plan, keybindings, statusline, stickers, mobile | 是 | 仅影响本地TUI状态 |
| **桥接安全** | compact, clear, cost, summary, releaseNotes, files | 是 | 可从移动端安全执行 |
| **Prompt命令** | 所有 type='prompt' 的命令 | 是 | 展开为文本发送，安全 |
| **JSX命令** | 所有 type='local-jsx' 的命令 | 否 | 渲染终端UI，移动端无法显示 |
| **其他Local** | 未在白名单的 type='local' 命令 | 否 | 默认阻止，需显式添加 |

### 7.3 远程控制命令

```typescript
// /bridge (remote-control) - 连接远程控制
const bridge = {
  type: 'local-jsx',
  name: 'remote-control',
  aliases: ['rc'],
  description: 'Connect this terminal for remote-control sessions',
  isEnabled: () => feature('BRIDGE_MODE') && isBridgeEnabled(),
  get isHidden() { return !isEnabled() },
  immediate: true,
  load: () => import('./bridge.js'),
}

// /bridge-kick - 注入桥接故障 (内部测试)
const bridgeKick = {
  type: 'local',
  name: 'bridge-kick',
  description: 'Inject bridge failure states for manual recovery testing',
  isEnabled: () => process.env.USER_TYPE === 'ant',  // 仅内部用户
  supportsNonInteractive: false,
  load: () => Promise.resolve({ call }),
}
```

---

## 8. 设计模式总结

### 8.1 使用的核心设计模式

| 模式 | 应用位置 | 说明 |
|------|----------|------|
| **Discriminated Union** | Command 类型 | 通过 type 字段区分三种命令实现模式 |
| **Lazy Loading** | load() 函数 | 延迟加载命令实现，优化启动性能 |
| **Factory Method** | createMovedToPluginCommand | 创建迁移到插件的命令包装器 |
| **Strategy Pattern** | 三种执行策略 (prompt/local/local-jsx) | 不同类型采用不同执行策略 |
| **Chain of Responsibility** | 权限检查链 | 多层权限依次过滤 |
| **Memoization** | COMMANDS(), loadAllCommands() | 缓存命令列表，避免重复加载 |
| **Observer Pattern** | onDone 回调 | JSX命令通过回调通知完成状态 |
| **Template Method** | getPromptForCommand | Prompt命令的提示词生成模板 |
| **Feature Toggle** | feature() 门控 | 编译时和运行时特性开关 |
| **Command Pattern** | 命令对象本身 | 封装请求为对象，参数化调用 |

### 8.2 数据流图

```
┌─────────────────────────────────────────────────────────────────────┐
│                          完整数据流                                  │
│                                                                     │
│  用户输入                                                            │
│     │                                                               │
│     ▼                                                               │
│  ┌─────────────┐    ┌──────────────┐    ┌─────────────────────┐    │
│  │ REPL/TextInput│───▶│parseSlashCmd │───▶│ getCommands(cwd)    │    │
│  └─────────────┘    └──────────────┘    └──────────┬──────────┘    │
│                                                    │               │
│                                       ┌────────────┴────────────┐  │
│                                       │     命令来源合并          │  │
│                                       │                         │  │
│                                       │  bundledSkills          │  │
│                                       │  builtinPluginSkills    │  │
│                                       │  skillDirCommands       │  │
│                                       │  workflowCommands       │  │
│                                       │  pluginCommands         │  │
│                                       │  pluginSkills           │  │
│                                       │  COMMANDS()             │  │
│                                       │  dynamicSkills          │  │
│                                       └────────────┬────────────┘  │
│                                                    │               │
│                                       ┌────────────┴────────────┐  │
│                                       │     实时过滤              │  │
│                                       │                         │  │
│                                       │  meetsAvailability()    │  │
│                                       │  isCommandEnabled()     │  │
│                                       │  dedupe dynamicSkills   │  │
│                                       └────────────┬────────────┘  │
│                                                    │               │
│                                                    ▼               │
│                                       ┌─────────────────────┐    │
│                                       │  findCommand()      │    │
│                                       │  (name/aliases)     │    │
│                                       └──────────┬──────────┘    │
│                                                  │               │
│                                                  ▼               │
│                                       ┌─────────────────────┐    │
│                                       │ getMessagesForCmd() │    │
│                                       │                     │    │
│                                       │  switch(type)       │    │
│                                       │    local-jsx → UI   │    │
│                                       │    local → text     │    │
│                                       │    prompt → model   │    │
│                                       └──────────┬──────────┘    │
│                                                  │               │
│                                    ┌─────────────┼─────────────┐  │
│                                    ▼             ▼             ▼  │
│                               ┌────────┐  ┌────────┐  ┌────────┐ │
│                               │ JSX UI │  │ Text   │  │ Model  │ │
│                               │ 渲染   │  │ 结果   │  │ 查询   │ │
│                               └───┬────┘  └───┬────┘  └───┬────┘ │
│                                   │           │           │      │
│                                   ▼           ▼           ▼      │
│                               ┌────────────────────────────────┐  │
│                               │     消息队列 (Message Queue)    │  │
│                               └────────────────────────────────┘  │
│                                                  │               │
│                                                  ▼               │
│                                       ┌─────────────────────┐    │
│                                       │  遥测记录             │    │
│                                       │  logEvent()         │    │
│                                       └─────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 9. 关键实现细节

### 9.1 Fork 子Agent 执行

Prompt 命令支持 `context: 'fork'` 模式，在独立子Agent中执行：

```typescript
// 触发条件
if (command.context === 'fork') {
  return await executeForkedSlashCommand(command, args, context, ...)
}

// 执行流程
1. prepareForkedCommandContext() → 准备子Agent上下文
2. runAgent({
     agentDefinition,           // Agent定义
     promptMessages,            // 提示词消息
     toolUseContext,            // 工具上下文
     canUseTool,                // 权限检查函数
     isAsync: false,            // 同步执行
     querySource: 'agent:custom',
     model: command.model,      // 可选的指定模型
     availableTools,            // 可用工具
   })
3. extractResultText(agentMessages) → 提取结果文本
4. 包装为用户消息返回
```

**Assistant 模式优化** (KAIROS 特性)：
```typescript
if (feature('KAIROS') && kairosEnabled) {
  // 后台异步执行, 不阻塞用户输入
  void (async () => {
    // 等待 MCP 服务器就绪
    await waitForMcpSettle()
    // 运行子Agent
    for await (const message of runAgent(...)) { ... }
    // 结果重新入队
    enqueuePendingNotification({ value: result, isMeta: true })
  })()
  return { messages: [], shouldQuery: false }  // 立即返回
}
```

### 9.2 技能钩子注册

Prompt 命令支持声明钩子，在命令执行时自动注册：

```typescript
// 命令声明
const mySkill = {
  type: 'prompt',
  name: 'my-skill',
  hooks: {
    PreToolUse: [{ matcher: 'Write', command: 'echo "file written"' }],
    Stop: [{ command: 'echo "turn ended"' }],
  },
  skillRoot: '/path/to/skill',  // 钩子命令的根目录
  // ...
}

// 执行时注册
if (command.hooks && hooksAllowedForThisSkill) {
  registerSkillHooks(
    context.setAppState,
    sessionId,
    command.hooks,
    command.name,
    command.skillRoot
  )
}
```

### 9.3 命令迁移工厂

`createMovedToPluginCommand` 用于将内置命令迁移到插件：

```typescript
export function createMovedToPluginCommand({
  name, description, progressMessage,
  pluginName, pluginCommand,
  getPromptWhileMarketplaceIsPrivate,
}): Command {
  return {
    type: 'prompt',
    name,
    source: 'builtin',
    async getPromptForCommand(args, context) {
      if (process.env.USER_TYPE === 'ant') {
        // 内部用户: 提示安装插件
        return [{ type: 'text', text: `
          This command has been moved to a plugin.
          1. Run: claude plugin install ${pluginName}@claude-code-marketplace
          2. Use: /${pluginName}:${pluginCommand}
        ` }]
      }
      // 外部用户: 使用原始提示 (市场公开前)
      return getPromptWhileMarketplaceIsPrivate(args, context)
    },
  }
}
```

---

## 10. 架构优势与特点

### 10.1 优势

1. **统一的命令接口** - 所有命令实现相同的元数据结构，便于管理和扩展
2. **懒加载优化** - 启动时只加载元数据，实现按需加载
3. **多层权限控制** - 编译时、认证、运行时、UI层、调用时五层防护
4. **特性门控** - 支持编译时 tree-shaking 和运行时动态开关
5. **可扩展性** - 支持插件、技能、工作流等多种命令来源
6. **错误隔离** - 单个命令加载失败不影响系统其他部分
7. **桥接安全** - 明确的命令安全分类，防止移动端误操作

### 10.2 设计亮点

- **Discriminated Union 类型** - TypeScript 类型安全地支持三种执行模式
- **Memoization 缓存** - 命令列表按 cwd 缓存，避免重复磁盘 I/O
- **动态技能发现** - 运行时扫描 `.claude/skills/` 目录自动注册
- **即时执行标记** - `immediate: true` 跳过命令队列，用于紧急操作
- **参数脱敏** - `isSensitive: true` 自动隐藏敏感参数

---

## 11. 文件索引

| 文件路径 | 行数 | 职责 |
|----------|------|------|
| `src/commands.ts` | 758 | 命令注册中心、来源合并、权限过滤 |
| `src/types/command.ts` | 217 | 命令类型定义 |
| `src/utils/processUserInput/processSlashCommand.tsx` | 922 | 命令解析、分派执行 |
| `src/utils/plugins/loadPluginCommands.ts` | - | 插件命令加载 |
| `src/skills/loadSkillsDir.js` | - | 技能目录扫描 |
| `src/skills/bundledSkills.js` | - | 捆绑技能注册 |
| `src/commands/createMovedToPluginCommand.ts` | 66 | 迁移命令工厂 |
| `src/commands/plugin/parseArgs.ts` | 105 | 插件参数解析 |
