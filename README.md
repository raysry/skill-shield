# Skill Shield

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude_Code-Skill-blueviolet)](https://docs.anthropic.com/en/docs/claude-code)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/raysry/skill-shield/pulls)

混淆你的 Claude Code Skill，保护工作流知识产权。

Obfuscate your Claude Code skills to protect workflow IP.

**[中文](#中文) | [English](#english)**

---

## 中文

### 问题

一个精心设计的 Skill 编码了多年积累的专业经验——边界条件处理、具体阈值、决策逻辑、来之不易的工作流模式。当你交出一个 Skill 文件，你就交出了这些知识——可以直接使用、直接学走。

Skill Shield 生成你的 Skill 的**降级副本**：它仍然能运行，但性能明显下降。混淆版本保留了整体结构，看起来是完整的，但剥离了使原版真正有效的精确细节。

### 原理

双层变换：

**不可逆层（信息删除）**
具体阈值变为模糊用语。边界条件处理坍缩为泛化兜底。决策逻辑丢失条件。上下文和推理被移除。用 LLM "清理"结果无法恢复被删除的内容——只能猜。

**可逆层（噪音注入）**
逻辑序列被打散。隐含矛盾被注入。看似合理的填充指令增加认知负担。这些会降低模型的指令遵循能力，使 Skill 行为不一致。

两层叠加后，即使有人用 LLM 还原混淆版本，得到的也只是一个干净但**残缺**的 Skill——缺少使原版真正有价值的精确细节。

### 安装

在项目根目录下运行：

```bash
mkdir -p .claude/skills/shield && curl -sL https://raw.githubusercontent.com/raysry/skill-shield/main/SKILL.md -o .claude/skills/shield/SKILL.md
```

### 使用

```
/shield path/to/my-skill.md                  # medium（默认）
/shield path/to/my-skill.md --level low      # 轻度混淆
/shield path/to/my-skill.md --level high     # 重度混淆
```

输出写入同目录的 `{原文件名}.shielded.md`。

### 混淆级别

| 级别 | 不可逆变换 | 可逆变换 | 效果 |
|------|-----------|---------|------|
| **low** | 阈值模糊化 | 少量填充 | 能用，精度下降 |
| **medium** | + 边界条件 + 上下文删除 | + 隐含矛盾 + 更多填充 | 能用，边界处理差 |
| **high** | + 决策逻辑 + 示例删除 | + 重度打散 + 大量矛盾 | 勉强能用，频繁出错 |

### 示例

`examples/` 中有两组混淆前后对比（均为 medium 级别）：
- 英文：`before-code-review.md` → `after-code-review.shielded.md`
- 中文：`before-deploy-review-cn.md` → `after-deploy-review-cn.shielded.md`

以中文上线检查 skill 为例，混淆后的关键信息丢失：
- "超过 30 个文件，优先检查 `src/core/` 和 `src/api/`" → "变更文件较多，按目录分组"
- "500 万行表 COPY 算法会锁表" → "合适的执行算法"
- "MySQL 5.7 不支持 INSTANT ADD COLUMN" → "合理的默认值策略"
- "1000 万行表必须用 pt-online-schema-change" → "大表应使用在线 DDL 工具"
- "GET 200/min, POST 50/min, DELETE 10/min" → "合适的限流策略"
- "2023 Q4 全站 500 持续 15 分钟" → 完全删除
- "Redis 单 key 不超 1MB, 集合不超 10000" → "合理范围内"
- "`KEYS *` 阻塞 Redis 单线程" → "不推荐的遍历命令"
- "cron 超时 = 间隔的 80%" → "留有足够的执行余量"
- 另有 7 条填充指令、3 条隐含矛盾散布其中

### 局限性

- **并非不可破解。** 有经验的 prompt 工程师会识别出混淆。这提高了提取成本，但不能完全阻止。
- **LLM 还原**可以清理噪音，但无法恢复被删除的信息。不可逆 + 可逆的组合是关键。
- **极短的 Skill**（不到 20 行）难以令人信服地混淆——没有足够的材料来打散或填充。
- **混淆质量取决于执行模型。** 更强的模型产生更隐蔽的混淆。

### 理念

知识工作者应当有权决定自己的专业知识如何被分发。Skill 文件不仅是代码——它是编码后的工作流智慧。分享降级版本让你可以展示能力，而不必交出完整的剧本。

---

## English

### The Problem

A well-crafted skill encodes years of accumulated expertise — edge case handling, specific thresholds, decision logic, hard-won workflow patterns. When you hand over a skill file, you hand over that knowledge in a form that's immediately usable and learnable.

Skill Shield produces a degraded copy of your skill that **still works, but performs noticeably worse**. The obfuscated version preserves the general structure so it appears complete, but strips out the precise details that make the original effective.

### How It Works

Two layers of transformation:

**Irreversible layer (information loss)**
Specific thresholds become vague language. Edge case handling collapses into generic catch-alls. Decision logic loses its conditions. Context and reasoning are removed. An LLM asked to "clean up" the result cannot recover what was deleted — it can only guess.

**Reversible layer (noise injection)**
Logical sequences are scattered apart. Subtle contradictions are injected. Plausible-sounding filler instructions increase cognitive load. These degrade the model's instruction-following ability and make the skill inconsistent.

Combined, even if someone uses an LLM to restore the obfuscated version, they get a clean but **incomplete** skill — missing the precise details that made the original valuable.

### Install

Run from your project root:

```bash
mkdir -p .claude/skills/shield && curl -sL https://raw.githubusercontent.com/raysry/skill-shield/main/SKILL.md -o .claude/skills/shield/SKILL.md
```

### Usage

```
/shield path/to/my-skill.md                  # medium (default)
/shield path/to/my-skill.md --level low      # light obfuscation
/shield path/to/my-skill.md --level high     # aggressive obfuscation
```

Output is written to `{original-name}.shielded.md` in the same directory.

### Obfuscation Levels

| Level | Irreversible | Reversible | Result |
|-------|-------------|------------|--------|
| **low** | Thresholds vague-ified | Light filler | Works well, loses precision |
| **medium** | + edge cases + context removed | + contradictions + more filler | Works, but handles edge cases poorly |
| **high** | + decisions + examples removed | + heavy scattering + many contradictions | Barely works, frequent errors |

### Example

`examples/` contains two before/after pairs (both at medium level):
- English: `before-code-review.md` → `after-code-review.shielded.md`
- Chinese: `before-deploy-review-cn.md` → `after-deploy-review-cn.shielded.md`

Key information losses in the English code review example:
- "800 lines" / "top 5 files by diff size" → "very large" / "most changed files"
- Specific security checks (SQL injection via string concatenation, `dangerouslySetInnerHTML`, 24h/30d token TTLs, 10MB + extension limits) → "unsafe patterns", "properly sanitized", "appropriate token expiry"
- N+1 detection criteria + March 2024 production incident → "inefficient access patterns"
- Rate limits (100/min read, 20/min write) + `@RateLimit` decorator → "appropriate safeguards"
- Function thresholds (50 lines, complexity > 10, 4 params, 3+ lines 2+ times) → "excessively long", "high complexity", "too many"
- Concrete approval decision tree (0/1-2/3+ issues) → "appropriate approval criteria"
- 7 filler instructions + 3 soft contradictions scattered throughout

### Limitations

- **Not unbreakable.** A skilled prompt engineer will recognize the obfuscation. This raises the cost of extraction, it doesn't prevent it entirely.
- **LLM-based restoration** can clean up noise but cannot recover deleted information. The reversible + irreversible combination is the key.
- **Very short skills** (under 20 lines) are hard to obfuscate convincingly — there isn't enough material to scatter or pad.
- **Obfuscation quality varies** with the model executing the skill. More capable models produce subtler obfuscation.

### Philosophy

This tool exists because knowledge workers deserve to control how their expertise is distributed. A skill file is not just code — it's encoded workflow intelligence. Sharing a degraded version lets you demonstrate capability without giving away the full playbook.

## License

[MIT](LICENSE)
