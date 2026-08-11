---
name: requirement-reverse-engineering
description: >
  Use when requirements lack a PRD (product built code-first), when dealing with legacy/historical
  requirements that need documentation retroactively, or when asked to "reverse engineer requirements"
  from a codebase. Triggers: "反推需求", "从代码生成需求文档", "没有PRD", "反向工程需求",
  "reverse engineer requirements", "generate PRD from code".
---

# 需求文档反推

从 Git 仓库代码自动反推需求文档。用于无原始 PRD 的历史需求，反推后与正常 PRD 完全等价，进入标准评分流程。

## 总流程

```
Step 1: AI 全自动（扫描仓库 → 过滤第三方 → 识别辅助材料 → 反推需求文档）
  → Step 2: 开发者复核需求文档
  → Step 3: 进入标准评分流程（AI 计数 → 规则引擎套公式 → 映射贡献分）
```

Step 1 产出一份需求文档。Step 2 开发者签认后，该文档与正常 PRD 完全等价。

---

## Step 1：AI 全自动反推

### 1.1 输入

- Git 仓库完整文件树（含所有分支的代码文件、文档、配置）

### 1.2 自动排除规则

以下目录/文件不读入、不计入需求文档：

| 排除项 | 判定规则 |
|--------|---------|
| 第三方代码目录 | `_bundle`、`vendor`、`third_party`、`external` |
| npm 依赖 | `node_modules` |
| 构建产物 | `dist`、`build`、`cache`、`out`、`target`、`.next` |
| 版本控制 | `.git`、`.svn` |
| 锁文件 | `*.lock`、`package-lock.json`、`yarn.lock`、`pnpm-lock.yaml` |
| 二进制/媒体文件 | `.png`、`.jpg`、`.gif`、`.ico`、`.ttf`、`.woff`、`.mp4`、`.exe` |
| 不可读文件 | `.pdf`、`.xlsx`、`.docx`、`.pptx`（除非 README 或 devlog 明确提及为需求参考） |
| Python 缓存 | `__pycache__/`、`*.pyc` |

### 1.3 辅助材料识别

按优先级读入：

| 优先级 | 材料 | 识别规则 | 用途 |
|--------|------|---------|------|
| 1 | 开发日志 | 根目录 `devlog*`、`CHANGELOG*`、`dev-log*` | 理解设计意图、区分自研 vs 整合 |
| 2 | 设计文档 | `docs/` 下所有 `.md`（含 `plans/`、`design/`、`spec/`） | 补充设计意图 |
| 3 | 项目概述 | 根目录 `README.md` 或 `README*.md` | 项目定位、高层功能概述 |

### 1.4 自写 vs 胶水代码判定

| 判定 | 标准 | 处理 |
|------|------|------|
| 自写代码 | 不在排除目录中，在根目录或 `src/`/`app/` 等自写目录 | 完整读入，功能细节写入需求文档 |
| 第三方代码 | 在排除目录中 | 不读入，功能不计入需求文档 |
| 胶水代码 | 自写代码，但功能是集成/编排/配置第三方能力 | 读入并写入需求文档，仅描述"集成 XX 第三方能力"，不描述第三方内部功能 |

**胶水代码判定辅助信号**（来自 devlog 或代码注释）：
- devlog 中出现"集成""预置""引入""离线可用"
- 代码注释中出现 "from"/"built from"/"sourced from" + 外部 URL
- 文件内容为配置文件格式（JSON/YAML/TOML）且包含外部仓库/包引用

---

## 反推规则：代码信号 → PRD 章节

### 功能模块

从代码中提取，不以 UI 交互/页面 DOM 推断：

| 代码信号 | 提取为 |
|---------|--------|
| 函数/方法定义 | 功能节点 |
| CLI 命令/子命令（case 分支、argparse subcommand） | 功能节点 |
| 脚本文件（独立 `.sh`/`.bat`/`.py`） | 功能模块 |
| 目录结构（`commands/`、`modules/`、`handlers/`） | 功能分组 |
| Dockerfile 构建阶段（RUN/COPY 语义块） | 功能节点 |
| 事件处理/回调注册（onclick、addEventListener、hook） | 功能节点 |

### 业务实体

从代码中数据结构定义直接提取，不以页面表单/UI 推断：

| 代码信号 | 提取为 |
|---------|--------|
| TypeScript `interface`/`type` | 业务实体 |
| JavaScript 对象模型（`const state = {...}`、`defaultState()`） | 业务实体 |
| Python `dataclass`/`NamedTuple`/`__init__` 参数 | 业务实体 |
| 数据库 schema（CREATE TABLE、ORM model） | 业务实体 |
| JSON schema / 配置文件结构 | 业务实体 |
| 硬编码常量数组 | 配置实体（标注为非持久化） |

每个实体标注：字段名、字段类型（文本/数字/枚举/对象/日期）、持久化方式、是否为计算字段。

### 校验规则

| 代码信号 | 提取为 |
|---------|--------|
| `if (xxx) throw/return/alert` | 校验规则 |
| `assert`、`validate`、`check` 函数 | 校验规则 |
| 表单/输入验证逻辑 | 校验规则 |
| 范围约束（`min`/`max`/`step`、`>=0 && <=10`） | 约束规则 |
| `case *` 后的 `die()` / `exit 1` | 校验规则 |

### 计算公式

保留原始变量名：

| 代码信号 | 提取为 |
|---------|--------|
| 多步数学运算（reduce、map、算术表达式链） | 计算公式 |
| 算法函数（`calcXxx()`、`computeXxx()`） | 计算公式 |
| SHA1/MD5/哈希计算 | 计算公式 |
| 排序/比较/排名逻辑 | 计算公式 |

### 分支与异常路径

| 代码信号 | 提取为 |
|---------|--------|
| `if-else` 分支 | 业务分支 |
| `try-catch` / `\|\| true` / `2>/dev/null` | 异常路径 |
| `switch-case` 多分支 | 业务分支 |
| API 响应的 `if (!res.ok)` 处理 | 异常路径 |

### 状态机

| 代码信号 | 提取为 |
|---------|--------|
| 枚举类型 / 状态常量 | 状态名称 |
| 状态转移函数（`switchStep()`、`nextStep()`） | 转移路径 |
| 生命周期状态（构建→运行→退出） | 状态节点 |
| 标志位变化（loading/success/error） | 状态转移 |

### 外部系统交互

仅提取业务 API 对接，不提取基础设施依赖：

| 计入（业务对接） | 不计入（基础设施） |
|-----------------|-------------------|
| HTTP API 调用（第三方服务端点） | Docker Engine |
| SDK 集成 | npm/pip/apt 包管理器 |
| WebSocket 长连接 | GitHub（代码托管/插件市场拉取） |
| OAuth/SSO 认证 | 操作系统/文件系统 |
| 消息队列/事件总线 | CI/CD 流水线 |
| Appium Server（WebDriver 协议） | ADB |

---

## 输出格式

按《需求文档模版》（`需求文档模版260614.md`）格式输出。

### 反推元数据块（标题下方、第 0 章上方，必须包含）

```markdown
> **来源**：代码反推（无原始 PRD）
> **代码范围**：<自写文件列表，标明自写/胶水>
> **第三方排除**：<排除的目录/文件及判定理由>
> **辅助材料**：<找到的 devlog/docs/README 文件列表>
> **开发者复核**：□ 待确认
```

### 不确定项处理

遇到无法从代码确定的内容，用以下格式标注：

```markdown
<!-- 待确认：具体问题描述 -->
```

常见不确定场景：
- devlog 和代码对同一功能描述不一致
- 功能存在但 devlog 未提及（可能是实验性/未完成）
- 代码中的注释与实际逻辑矛盾

---

## Step 2：开发者复核

开发者逐项确认，清除所有 `<!-- 待确认 -->` 标记：

| 复核项 | 检查内容 |
|--------|---------|
| 功能范围 | 是否有遗漏？是否有虚增（第三方功能被当成自写）？胶水代码标注是否正确？ |
| 业务实体 | 字段名、类型、持久化方式是否与实际代码结构一致？计算字段是否正确标注？ |
| 校验规则/公式 | 是否准确反映代码逻辑？ |
| 外部系统 | 是否仅含实际对接的 API（不含基础设施）？ |
| 状态机 | 状态节点和转移路径是否反映实际代码？ |
| 不确定项 | 逐条确认或修正，清除所有 `<!-- 待确认 -->` |

复核完成后签认：

```markdown
> **开发者复核**：✅ 已确认（YYYY-MM-DD）
```

---

## Step 3：进入标准评分

开发者签认后的反推文档，与正常流程产出的 PRD 完全等价，进入《需求贡献分评分》skill 的标准评分流程。
