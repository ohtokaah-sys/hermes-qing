# Hermes Agent（小青）
## 📋 审阅指南
- **重点段落**：[待标注]
- **可跳过**：[待标注]
- **数据来源**：[待标注]
- **假设列表**：[待标注]
- **建议审阅方式**：[待标注]
> **97.5% KV Cache 命中率。不是编程场景，是跨业务场景——架构设计、管理辅导、财务分析、药品零售渠道策略、招投标评审、PPT 生成、视频分析。月均 86 亿 input tokens，¥1,004/月。没有后端优化，纯上下文工程。**
这是一个生产级 AI 助手系统的架构佐证材料。含行为宪法、插件代码、运维脚本、方法论——证明系统设计和工程实现能力。
[English version](#hermes-changqing-evergreen-agent)
---
## TL;DR
主题：Hermes Agent（小青） > 📋 审阅指南 > 数字。关键点：重点段落；可跳过；数据来源。含 34 处数据点。
## 数字
| 指标 | 数据 |
|:--|:--|
| KV Cache 命中率 | **97.5%**（2026-06 全月均值） |
| 月吞吐 | **86 亿** input tokens |
| 月 API 调用 | **68,147** 次 |
| 月成本 | **¥1,004** |
| 主模型 | DeepSeek V4 Pro（非编程专用，通用场景） |
不是代码补全刷出来的命中率。68,147 次调用覆盖了：IT 架构方案设计、资深技术专家入职策略、家庭十年账单分析、药品零售渠道汇报、能源投资 PPT 重做、视频深度分析、招投标评审——一个真实架构师的日常工作流。
没做后端 KV 优化。这 97.5% 是靠：分层记忆（L1 即时→L1.5 活跃上下文→L2 可检索归档）、精准上下文注入（每轮只加载该加载的）、规则引擎压缩（233 行 SOUL 替代 1,351 行）。
---
## 两件事让它不一样
### 1. 上下文工程效率
分层记忆架构 + 精准注入。每轮只加载该加载的，不灌全量。结果：97.5% 的 token 都在缓存里，API 只计增量。
### 2. 确定性执法：脚本守门，不靠 LLM 自觉
LLM 自己检查自己 = 不可靠。核心设计原则是**执法者外移**：
- **L0 机械插件**：网关层 pre/post hook，正则匹配，零 token，零幻觉
- **外部脚本执法**：cron 驱动验证链——LLM 产出后独立脚本检查是否落盘
- **7 道机械门控**：回复发送前逐条扫描——缺标签不发、缺来源不发、缺验证不发
- **at-job 自愈回滚**：修改系统配置前自动部署定时回退，Agent 断线也能恢复
---
## 仓库内容
| 内容 | 路径 | 说明 |
|:--|:--|:--|
| **行为宪法** | `SOUL.md` | 17 条 Iron Laws + 4 种协作模式 + 29 条场景路由 |
| **能力清单** | `小青特性注册表.md` | 完整能力目录 |
| **设计方法论** | `methodology/Hermes-Skill设计参考指南.md` | Skill 设计模式与陷阱 |
| **插件系统** | `plugins/`（12 个） | 产出合规检查、注入防御、中断扫描、上下文注入 |
| **运维脚本** | `scripts/` `scripts-b6/` | 文件验证、依赖检查、双模审计、进程日志 |
| **配置模板** | `config-templates/` | systemd unit、安全规则、cron 清单模板 |
| **开源补丁** | `PRODUCT_PATCHES.patch` | 对上游 Hermes Agent 的 20+ 个定制补丁 |
| **模板** | `templates/` | 用户画像和记忆模板 |
---
## 你可以怎么用
1. **理解架构**——看一个生产级 Agent 如何组织规则引擎、插件、脚本、验证链
2. **学习方法论**——Skill 设计指南和 plan 模板不依赖任何代码
3. **参考代码**——插件和脚本直接展示了「Agent 怎么自己管自己」
4. **借鉴 SOUL.md**——行为宪法的分层设计可用于任何 Agent 系统
---
## 你需要自己准备的
- Hermes Agent 运行环境
- DeepSeek / 百炼 / 火山 等模型的 API key
- （可选）Milvus/FAISS 向量库
- （可选）PPT Master——如需 PPT 生成链
---
## 为什么不是「开箱即用」
这个仓库来自一个运行两个月的生产系统。它曾经连接着备份存储、管理着 VPN、同步着多用户画像、推送着消息通知。那些能力涉及个人数据和特定基础设施，无法开源。
公开发布的不是完整系统，是**架构骨架**——足够证明设计能力和工程实现，但需要你自己填肉。
---
## 许可证
MIT
---
# Hermes Changqing (Evergreen Agent)
> **97.5% KV Cache hit rate. Not code completion. Cross-business scenarios—architecture design, management coaching, financial analysis, pharmaceutical retail strategy, bid evaluation, PPT generation, video analysis. 8.6 billion input tokens/month, ¥140/month. No backend optimization. Pure context engineering.**
Production-grade AI agent system architecture portfolio. Contains behavioral constitution, plugin code, ops scripts, and methodology—demonstrating system design and engineering capability.
---
## The Numbers
| Metric | Value |
|:--|:--|
| KV Cache Hit Rate | **97.5%** (June 2026 monthly average) |
| Monthly Throughput | **8.6 billion** input tokens |
| Monthly API Calls | **68,147** |
| Monthly Cost | **~¥140** (~$20 USD) |
| Primary Model | DeepSeek V4 Pro (general-purpose, not coding-specific) |
This isn't a coding benchmark. Those 68,147 calls spanned: IT architecture design, onboarding strategy for a 45-year-old Huawei Principal Engineer, decade-long household financial analysis, pharmaceutical retail channel presentations, oilfield investment deck rebuilds, deep video analysis, bid evaluation—the actual daily workflow of an enterprise architect.
No backend KV optimization was done. The 97.5% comes from: hierarchical memory (L1 instant→L1.5 active context→L2 searchable archive), precise context injection (only load what's needed per turn), and rule engine compression (233-line SOUL replacing 1,351-line original).
---
## Two Things That Set It Apart
### 1. Context Engineering Efficiency
Hierarchical memory + precise injection. Every turn loads only what's relevant. Result: 97.5% of tokens live in cache, API only bills the delta.
### 2. Deterministic Enforcement: Scripts Guard, Not LLM Self-Check
LLMs auditing themselves is unreliable. Core principle: **move enforcement outside the model**.
- **L0 Mechanical Plugins**: Gateway pre/post hooks, regex matching, zero tokens, zero hallucination
- **External Script Enforcement**: Cron-driven verification chains—independent scripts check LLM output after generation
- **7 Mechanical Gates**: Pre-delivery scanning—missing label → no send, missing source → no send, missing verification → no send
- **at-job Self-Healing Rollback**: Auto-scheduled rollback before any system config change, recovers even if Agent disconnects
---
## Repo Contents
| Content | Path | Description |
|:--|:--|:--|
| **Behavioral Constitution** | `SOUL.md` | 17 Iron Laws + 4 collaboration modes + 29 scenario routes |
| **Capability Registry** | `小青特性注册表.md` | Complete capability catalog |
| **Design Methodology** | `methodology/Hermes-Skill设计参考指南.md` | Skill design patterns and pitfalls |
| **Plugin System** | `plugins/` (12) | Output compliance, injection defense, interrupt scanning, context injection |
| **Ops Scripts** | `scripts/` `scripts-b6/` | File verification, dependency checks, dual-model auditing, process logging |
| **Config Templates** | `config-templates/` | systemd units, security rules, cron manifest template |
| **Upstream Patches** | `PRODUCT_PATCHES.patch` | 20+ custom patches for upstream Hermes Agent |
| **Templates** | `templates/` | Profile and memory templates |
---
## How to Use This Repo
1. **Study the architecture**—see how a production Agent organizes rule engines, plugins, scripts, and verification chains
2. **Learn the methodology**—the Skill design guide and plan templates are framework-agnostic
3. **Reference the code**—plugins and scripts demonstrate "how an Agent manages itself"
4. **Steal SOUL.md**—the layered behavioral constitution design works for any Agent system
---
## What You'll Need
- Hermes Agent runtime
- API keys for DeepSeek / Bailian / Volcengine models
- (Optional) Milvus/FAISS vector database
- (Optional) PPT Master—for PPT generation pipelines
---
## Why Not "Ready to Run"
This repo comes from a two-month production system. It once connected to backup storage, managed VPNs, synced multi-user profiles, and pushed message notifications. Those capabilities involve personal data and specific infrastructure that can't be open-sourced.
What's published isn't the complete system—it's the **architectural skeleton**. Enough to prove design and engineering capability, but you'll need to add the muscle.
---
## License
MIT