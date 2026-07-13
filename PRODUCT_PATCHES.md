> ⚠️ **合规警告**：此文件存在以下无法自动修补的问题：
> - NAS 路径引用
> 原稿已保存为 .orig 文件。

> ⚠️ **合规警告**：此文件存在以下无法自动修补的问题：
> - NAS 路径引用
> 原稿已保存为 .orig 文件。

> ⚠️ **合规警告**：此文件存在以下无法自动修补的问题：
> - NAS 路径引用
> 原稿已保存为 .orig 文件。

> ⚠️ **合规警告**：此文件存在以下无法自动修补的问题：
> - NAS 路径引用
> 原稿已保存为 .orig 文件。

> ⚠️ **合规警告**：此文件存在以下无法自动修补的问题：
> - NAS 路径引用
> 原稿已保存为 .orig 文件。

# 产品代码补丁清单
## 📋 审阅指南
- **重点段落**：[待标注]
- **可跳过**：[待标注]
- **数据来源**：[待标注]
- **假设列表**：[待标注]
- **建议审阅方式**：[待标注]
每次 `hermes update` 后，到这里检查是否需要重新 apply。

apply 命令：
  cd ~/.hermes/hermes-agent && git apply ~/.hermes/PRODUCT_PATCHES.patch

  cd ~/.hermes/hermes-agent && git diff --stat
## TL;DR
> ⚠️ 自动生成，请审阅

主题：产品代码补丁清单 > 📋 审阅指南 > ⚠️ 升级后 apply 免责声明。关键点：重点段落；可跳过；数据来源。含 16 处数据点。


## ⚠️ 升级后 apply 免责声明

**行号不保证匹配。** `.patch` 文件是在当前代码基线（commit d718023a8 + working tree）上生成的。`hermes update` 后上游代码发生变化，patch 中的行号几乎必然偏移。

**三种可能的结果：**
| 情况 | 含义 | 处理 |
|------|------|------|
| 某个 patch 段 apply 成功 | 上下文匹配，改动仍然适用 | ✅ 无需操作 |
| 某个 patch 段 apply 失败（行号偏移） | 上下文变了但相关代码还在 | 手动定位对应函数，重写 patch 段 |
| 某个 patch 段 apply 失败（代码段不存） | 相关代码被上游删除或重写 | 确认新版本是否有等效功能——有则放弃此段，无则在新代码上重新实现 |

**标准流程**（每次 `hermes update` 后）：
1. `cd ~/.hermes/hermes-agent && git apply --check ~/.hermes/PRODUCT_PATCHES.patch`
2. 如果有报错，逐个 patch 段处理——不要盲目 `git apply`
3. 确认所有改动后，重新生成 `.patch` 文件覆盖旧版
4. 更新本文件中各 P 条目的「状态」字段

---

## P001: MEDIA 扩展名白名单新增 .md / .html
**日期**：2026-05-24 → **2026-05-30 已上游化**
**文件**：`gateway/platforms/base.py`
**验证**：N/A（已上游化）
**断言**：已上游化，无需验证
**状态**：🟢 不再需要。新版 `MEDIA_DELIVERY_EXTS`（line 1168）已包含 `.md`，注释中明确提到修复了 `.md` 的 "silent black hole" 问题。

---

## P002: 未识别 MEDIA 扩展名告警日志
**日期**：2026-05-24
**文件**：`gateway/platforms/base.py`（两处：format_message_body + format_message_body_with_media）
**原因**：MEDIA: 指令带了不在白名单的扩展名时，新增 logger.warning 以便排查。
**影响**：仅日志。2026-07-03 升级：上游采用 bare-path fallback 设计，我们学上游 + 叠加一行 debug 日志（决策 B）。本补丁不再需要。

---

## P003: WeChat 大图原生处理注释
**日期**：2026-05-24 **2026-05-30 升级覆盖**
**文件**：`gateway/platforms/weixin.py` → `_outbound_media_builder()`
**验证**：N/A（已废弃）
**断言**：已废弃，无需验证
**状态**：🟢 无需恢复。纯注释，不重新打。

---

## P004: [SILENT] 精确匹配（修复子串误杀）
**日期**：2026-05-31
**文件**：`cron/scheduler.py` line 1948
**原因**：`SILENT_MARKER in` 子串匹配导致日报中「补录任务返回 [SILENT]」被误判为 [SILENT] 信号，跳过投递。改为 `==` 精确匹配——只有整条响应恰好是 `[SILENT]` 才跳过。
**验证**：`grep -c "== SILENT_MARKER" cron/scheduler.py`
**断言**：>= 1
**影响**：cron 投递逻辑。2026-07-03 升级确认：上游已采用 `.strip().upper() == SILENT_MARKER.upper()` 精确匹配，我们的意图已被覆盖。✅ 无需重新 apply。

---

## P005: cron job_id 向后兼容 + 日志级别提升
**日期**：2026-06-01
**文件**：`cron/jobs.py`（`_normalize_job_record`）、`gateway/run.py`（`_start_cron_ticker`）
**原因**：旧版 `jobs.json` 使用 `job_id` 字段而非 `id`，升级后 `KeyError` 静默杀死所有 cron。同时 cron tick 异常仅打 DEBUG 日志，生产环境静默死亡无法发现。
**改动**：
  - `_normalize_job_record`：自动检测 `job_id` → `id` 映射
  - `advance_next_run`：`.get("id")` + `.get("job_id")` 双保险
  - `_start_cron_ticker`：cron tick 异常日志 DEBUG → ERROR
**验证**：`grep -c "job_id.*id" cron/jobs.py`
**断言**：>= 1
**状态**：✅ 2026-07-03 升级确认：`normalize_job_record` 已在上游 v2026.7.1 中存在，无需重新 apply。

---

## P006: api_server /v1/files 端点 + MEDIA 重写
**日期**：2026-06-07（部署）→ 2026-06-07（升级丢失后恢复）
**文件**：`gateway/platforms/api_server.py`
**原因**：Desktop 远程客户端不能使用 MEDIA: 标签（被 base.py 在 api_server 之前消费）。替代方案：`_rewrite_media_paths()` 将回复中的 MEDIA:/path 重写为 HTTP URL，`_handle_files()` 通过 `/v1/files?path=&api_key=` 端点提供文件下载。
**内容**：
  - `_rewrite_media_paths` 方法：将 MEDIA: 替换为 `/v1/files` HTTP URL
  - `_handle_files` 端点：路径安全检查（仅 ~/hermes-media/ + ~/hermes-local/）
  - 三处调用的 final_response 行包裹 `self._rewrite_media_paths(..., request)`
  - 路由注册 `add_get("/v1/files", self._handle_files)`
**验证**：`grep -c "_rewrite_media_paths" gateway/platforms/api_server.py`
**断言**：>= 1
**影响**：Desktop/麒麟/Ubuntu 客户端文件投递。2026-07-03 升级确认：`_rewrite_media_paths` + `_handle_files` + 5 处 final_response 包裹 + 路由注册全部存在于上游 v2026.7.1。✅ 无需重新 apply。

### P006 — Post-Hook 验证机制（2026-06-14）
- **文件**：`cron/scheduler.py`、`cron/jobs.py`（`create_job` 新增参数）
- **改动**：在 `_process_job` 中 `save_job_output` 后插入 post_hook 验证环节
- **功能**：
  - 支持 `jobs.json` 中 `post_hook: {script, args}` 配置
  - 支持 `{output_path}` / `{job_id}` 占位符
  - exit 0=通过，1=验证失败（重试最多 retry_limit 次），≥2=脚本异常（不重试）
  - 超时不重试
  - 失败记录到 `~/.hermes/state/hook_fail_history.json`（24h 滑动窗口）
  - 通知路由调用 `roles.py`（own_cron_fail / own_cron_fail_escalation）
  - 向后兼容：无 post_hook 字段的旧 cron 不做任何验证
- **新增字段**：`post_hook` / `hook_type` / `hook_timeout` / `retry_limit`（均为可选）
- **回退**：`git apply -R` 移除 P006 段

---

## P007: write_file 机械守门（ADR-0006）
**日期**：2026-06-12
**文件**：`tools/file_tools.py`（~360行新增）、`cron/jobs.py`（`save_job_output` 集成）
**原因**：
  - SOUL.md 的 Self-check 规则要求产出含特定标记（🟡假设/📋审阅指南/🔴🟡🟢⚪置信度），但 Agent 经常遗漏——执法者与违规者是同一个 LLM
  - NAS 路径污染：产出中混入 `/mnt/storage/*` 路径，违反 SOUL.md §2
  - 文件名含 emoji/Windows 保留字符，NAS SMB 静默失败
  - 预防式阻断 > 事后检测
**设计原则**（用户明确）：
  - 永不阻断文件写入——原稿永远落盘为 `.orig` 备份
  - 自动修补缺失标记（注入模板占位符）
  - 修补后仍缺 → 发原稿 + 警告头
  - 外部规则配置 `write_guard.yaml`，升级不丢
**改动**：
  - 新增函数：`_clean_filename`、`_check_content_compliance`、`_auto_fix_content`、`_backup_orig_files`、`_load_guard_rules` 等
  - `write_file_tool`：写入前合规检查 + 自动修补 + .orig 备份
  - `patch_tool`：patch 前 .orig 备份 + patch 后合规检查（后置，锁内）
  - `save_job_output`：cron 产出同样走合规检查 + 文件名清理
  - `_check_file_staleness` 等原有函数的 stale warning 合并进合规 warnings
**验证**：`grep -c "def _check_content_compliance" tools/file_tools.py`
**断言**：>= 1
**影响**：所有 write_file/patch/cron 产出。2026-07-03 升级确认：`_check_content_compliance` 在 file_tools.py（13处）+ jobs.py（1处）+ write_guard.yaml 全部存在于上游 v2026.7.1。P010/P016（skip_for_paths 白名单）升级后补回。✅ 核心守门已覆盖。

### session_vectorize.py
**路径**：`~/.hermes/scripts/session_vectorize.py`
**说明**：此文件在 git repo 外（`~/.hermes/scripts/`），`hermes update` 不会覆盖。但全量重建或换机器时需手动迁移。
**加密增量备份**：已覆盖（`~/.hermes/scripts/` 在备份路径内）。
**最后修改**：2026-06-07 —— 新增 `_make_chunk`、`chunk_api_server_sessions`，`incremental_update` 移除早期 return，`full_build` 加入 api_server 处理。

🟡 假设：此产出由Agent自动生成，未经用户手动标注假设。

---

## P008: session_reset 策略关闭（2026-06-16）

**类型**：配置级补丁（非源码修改）

**问题**：`config.yaml` 中 `session_reset.mode: both` + `at_hour: 4` 导致每天凌晨 4 点后第一条消息触发 session 自杀。该配置自 4 月写入但为死代码——5 月 23 日 Hermes 升级（commit `588cdacd4`）激活 `SessionResetPolicy` 后开始生效。高频重启日（如 6 月 16 日）白天 session 反复断裂，上下文持续丢失。

**修复**：
```yaml
# config.yaml
session_reset:
-  mode: both
+  mode: none
  idle_minutes: 1440
  at_hour: 4
```

**生效**：需重启 gateway。后续升级不会被覆盖（config.yaml 在 git repo 外）。

**回滚**：改回 `mode: both` 即可。

**验证**：`grep -c "mode: none" ~/.hermes/config.yaml`
**断言**：== 1
**参见**：`gateway/session.py` `_should_reset()`（行 790-830）、`gateway/config.py` `SessionResetPolicy`（行 238-265）

---

## P009: TimeoutStopSec 修复（2026-06-16）

**类型**：系统级补丁（systemd unit）

**问题**：`hermes-gateway.service` 的 `TimeoutStopSec=30s`，但 gateway drain 需要 180s。systemd 在 30s 时 SIGKILL → pre-drain `mark_resume_pending()` 从未完成 → `resume_pending` 永远为 False。

**修复**：`hermes gateway install --system --force`，重新生成 unit 文件。

**验证**：`systemctl show hermes-gateway -p TimeoutStopUSec | grep -c "3min"`
**断言**：== 1（180s drain + 30s buffer）

---

## P010: write_guard 白名单支持（2026-06-16）

**类型**：源码补丁 + 配置

**问题**：ADR-0006 合规检查对 SOUL.md/MEMORY.md 等政策文件中的**禁止性 NAS 引用**（"不要用 /mnt/storage"）产生误报，每次文件修改都在文件头追加"合规警告"冗余行。Agent 每轮读到"此文件存在无法修补的问题"会削弱对系统提示的信任。

**修复**：
1. `tools/file_tools.py` `_check_content_compliance()`：新增 `skip_paths` 白名单检查（3 行）
2. `write_guard.yaml`：新增 `skip_paths: [~/.hermes/memories/]`

**生效**：需重启 gateway。

**状态**：✅ 2026-07-03 已重新部署。升级后 `_detect_violations()` 中 per-rule `skip_for_paths` 检查缺失，手动补回 3 行。

P010 更新（2026-06-16）：规则级白名单替代文件级白名单。
- 撤销了 `skip_paths`（文件级跳过 → 会漏检 SOUL.md 的其他违规）
- 改为 `nas_path.skip_for_paths`（只跳过 NAS 路径检测 → 其他 6 条规则照跑）
- `_detect_violations()` 中新增 per-rule `skip_for_paths` 检查（3 行）

## P011: P1-6 身份锚前移 — system_prompt.py
- `agent/system_prompt.py`：USER.md 从 volatile 层移至 stable 层，注入到 SOUL.md 之前
- 效果：Agent 先知道"我是谁/在跟谁说话"，再加载行为规则
- 行数：stable 层 +7 行（加载 USER.md），volatile 层 -5 行（移除重复）
- P1-5 L1 场景路由已解决触发时机，此改动解决检索位置优化

---

## P012: CONTEXT_FILE_MAX_CHARS 扩大至 100K（2026-06-18）

**日期**：2026-06-18
**文件**：`agent/prompt_builder.py` line 914
**问题**：`CONTEXT_FILE_MAX_CHARS = 20_000` 导致 SOUL.md（86K）被截断为前 14K + 后 4K = 18K，丢失 63% 的行为规则（Rule 3–15、L2-SCAN、Communication Style、Collaboration Modes、Interruption Rules 等）。MEMORY.md（95K）同样被截断至 18K。两者独立上限，互不影响。
**修复**：`20_000 → 100_000`，单行改动。
**影响**：
- SOUL.md 86K 完整注入（前 70K + 后 20K 覆盖全量）
- MEMORY.md 95K 完整注入
- Token 成本增加但可控（SOUL + MEMORY 全量 ≈ 50K tokens）
- 回滚：改回 `20_000` + 重启 gateway
**验证**：`grep -c "CONTEXT_FILE_MAX_CHARS = 100_000" agent/prompt_builder.py`
**断言**：== 1
**备份**：`prompt_builder.py.bak.20260618174154`

---

## P013: read_extract 缺失导致 read_file 全军覆没（2026-06-18）

**日期**：2026-06-18
**文件**：`tools/file_tools.py` line 788
**问题**：`from tools.read_extract import ...` 在 read_file_tool 中无条件执行，但 `tools/read_extract.py` 文件不存在。无 try/except 保护——所有 read_file 调用（不管文件类型）都报 `No module named 'tools.read_extract'`。
**修复**：import 包裹 try/except ImportError，缺失时 `is_extractable_document = lambda p: False`（优雅降级，跳过文档提取）。
**副作用**：.docx/.xlsx 的结构化文本提取暂时不可用——回退到二进制文件检测。
**回滚**：恢复原 import 行 + 重启 gateway。
**验证**：`grep -c "except ImportError" tools/file_tools.py`
**断言**：>= 1
**发现者**：Agent（笔记本），2026-06-18

---

## P016: write_guard 路径白名单——系统目录跳过模板注入（2026-06-19）

**日期**：2026-06-19
**文件**：`tools/file_tools.py` `_check_content_compliance()`、`~/.hermes/write_guard.yaml`
**问题**：ADR-0006 模板注入对**所有** .md 文件生效——包括 cron 产出、缓存、日志等系统文件。导致：
  - cron 产出被塞入 `[待标注]` 占位符
  - 合规审计误判 cron 产出为违规（"审阅指南缺失"）
  - 每次写给用户的文档需要手动清理占位符
**修复**：
  1. `write_guard.yaml` 新增 `skip_for_paths` 列表（排除 `~/.hermes/cron/`、`cache/`、`logs/`、`state/`、`tmp/`）
  2. `_check_content_compliance()` 在扩展名过滤**前**检查路径——命中 `skip_for_paths` 则跳过全部注入
**影响面**：cron 产出洁净 ✅、用户交付文件（hermes-local/、kb/）仍自动标注 ✅。2026-07-03 升级后 `_check_content_compliance()` 中顶层 `skip_for_paths` 检查缺失，手动补回 3 行。
**回滚**：删除 `skip_for_paths` + 移除 `_check_content_compliance` 中新增的路径检查块 + 重启 gateway
**验证**：`grep -c "skip_for_paths" ~/.hermes/write_guard.yaml`
**断言**：>= 1
**发现者**：用户："ADR-0006 模板经常注入影响我们"

## P017: Desktop 交互审批链路（is_rich 回调优先级）
**日期**：2026-06-18
**文件**：`tools/approval.py`、`tui_gateway/server.py`
**问题**：Desktop (WebSocket) 审批回调可能被非 rich callback 覆盖——`register_gateway_notify` 不区分回调类型，后注册的覆盖先注册的。
**修复**：
  - `tools/approval.py`：新增 `_gateway_notify_rich` set + `is_rich` 参数——rich 回调注册后，同 key 的非 rich 注册被静默丢弃
  - `tui_gateway/server.py`：Desktop 审批回调注册时传 `is_rich=True`
**验证**：`grep -c "is_rich" tools/approval.py`
**断言**：>= 1
**影响**：2026-07-03 升级确认：`register_gateway_notify` 签名仍含 `is_rich` 参数，approval.py 3处 + tui_gateway 1处全部存在。✅ 无需重新 apply。
**回滚**：`git checkout tools/approval.py tui_gateway/server.py`

---

## P018: slash_commands truncated diff 提示优化
**日期**：2026-06-18
**文件**：`gateway/slash_commands.py`
**问题**：truncated skill diff 输出只提示 `~/.hermes/pending/skills/{id}.json`，未提示 CLI 路径。
**修复**：追加提示 `/skills diff {id}` 在 CLI 上可用。
**验证**：`grep -c "skills diff" gateway/slash_commands.py`
**断言**：>= 1
**影响**：纯 UX，升级后丢失不影响功能。
**回滚**：`git checkout gateway/slash_commands.py`

---

## P019: gateway 重启死循环——残留 cron_tick() + one-shot cron
**日期**：2026-06-20
**文件**：`gateway/run.py`
**问题**：两个 bug 叠加导致 gateway 重启死循环（~4小时）：
  1. 上游 cron 调度重构后，`_start_gateway_housekeeping()` 残留 `cron_tick()` 调用但未 import → NameError 致 cron scheduler 异常
  2. 一个延迟的 one-shot cron job（`8de18bd47e37`）在 gateway 启动后触发 `systemctl restart` → SIGTERM → systemd Restart=always → 再启动 → 再触发 → 死循环
**修复**：
  1. 删除 `gateway/run.py` 第 17028-17031 行（残留的 cron_tick try-except 块）
  2. 从 `jobs.json` 删除 job `8de18bd47e37`
**验证**：`grep -c "cron_tick(verbose" gateway/run.py`
**断言**：== 0
**影响**：2026-07-03 升级确认：gateway/run.py 无残留 cron_tick() 调用（grep 结果 = 0），jobs.json 无死循环 job。✅ 无需重新 apply。
**发现者**：家属的 Hermes（deepseek-v4-pro），2026-06-20
**回滚**：恢复删除的 4 行即可

---

## P020: L0 中断系统日志精简（2026-06-21）

**日期**：2026-06-21
**文件**：`plugins/l0-safety-guard/__init__.py`
**问题**：全链路 23 条 `_log_scan` 日志节点，每轮消息都写。user_msg[:40] 预览引入隐私风险（明文进普通日志文件）。
**修复**：精简为 6 条——4 异常（p1_fail/p1_retry_fail/timeout/p2_fail）+ 2 触发（triggered/inject）。移除 user_msg[:40] 预览。决策依据：已验证注入链路通，让助手通过日志确认注入成功的需求不存在。"生产级最小日志"。
**验证**：`grep -c "_log_scan" plugins/l0-safety-guard/__init__.py`
**断言**：== 7（1 定义 + 6 调用）
**回滚**：恢复 `__init__.py.bak.20260621180516`

---

## P021: MEDIA 扩展名白名单补全——run.py 侧 .md/.html（2026-06-29）

**日期**：2026-06-29
**文件**：`gateway/run.py`（两处 `_TOOL_MEDIA_RE` 正则：line 1033 + line 15589）
**问题**：`base.py` 的 `MEDIA_DELIVERY_EXTS` 已含 `.md`（上游 P001 已修），但 `run.py` 用自己的 `_TOOL_MEDIA_RE` 正则做 MEDIA: 标签提取——两个白名单不同步。Agent 回复中 `MEDIA:/path/file.md` 从未被正则匹配到，导致 .md/.html 文件发送静默失败（Agent 声称"已发送"，用户实际未收到）。
**修复**：正则扩展名列表 `txt|csv|apk|ipa` → `txt|csv|md|html|apk|ipa`（两处）。
**验证**：发送 .md 文件，用户确认收到。
**影响**：升级后上游可能同步此改动（与 base.py 对齐），届时 P021 可标记为 🟢 已上游化。
**回滚**：`git checkout gateway/run.py` 或改回 `apk|ipa`。

### ⚠️ 事后纠正（2026-06-29 同会话）

P021 的诊断是错误的。`base.py` 的 `extract_media()` 用 `MEDIA_DELIVERY_EXTS`（含 `.md`），正则直接测试能匹配 `.md` 文件。真正的问题是：
1. **时序问题**：Agent 写 MEDIA: 标签时文件还未落盘到 `hermes-media/`，`validate_media_delivery_path` 用 `strict=True` 解析路径，文件不存在静默返回 None。
2. **静默丢弃无日志**：`filter_media_delivery_paths` 过滤掉无效路径但不打 warning。
3. **`hermes send` CLI 走不同管线**，不受此影响。

**2026-06-29 已撤销 P021 源码改动**，改为行为层修复：SOUL Rule 16 新增文件存在性验证步骤。

---

## P022: MEDIA 全链路诊断日志 + 用户可见投递失败兜底（2026-06-30）

**日期**：2026-06-30
**文件**：`gateway/platforms/base.py`（validate_media_delivery_path + extract_media）、`gateway/platforms/weixin.py`（send_message）
**问题**：MEDIA 投递静默失败，gateway 日志零条记录。P021 纠正中指出的"静默丢弃无日志"未修复——`validate_media_delivery_path` 有 7 个 return None 点全部静默，extract_media 入口/出口无日志，投递异常被吞且用户不可见。
**修复**（三层）：
1. **base.py validate_media_delivery_path**：7 个 return None 点全部加 `logger.warning`（含路径脱敏 + 具体原因：空路径/expanduser失败/非绝对路径/resolve失败/非普通文件/denylist拒绝/strict模式拒绝）
2. **base.py extract_media**：入口统计原始 MEDIA 行数 + 出口记录提取结果/零提取告警
3. **weixin.py send_message**：extract 后 + filter 后日志，投递失败收集 → 追加 `⚠️ 以下文件发送失败：{文件名}: {原因}` 到回复文本
**影响**：2026-07-03 升级确认：base.py 7处 logger.warning + extract_media 入口/出口日志 + weixin.py 投递失败兜底消息全部存在。✅ 无需重新 apply。
**回滚**：`git checkout gateway/platforms/base.py gateway/platforms/weixin.py`

---
## P023 — weixin.py MEDIA 投递静默失败修复

**日期**：2026-07-03
**文件**：`gateway/platforms/weixin.py`
**备份**：`weixin.py.bak.20260703075530`

**改动点**：
1. `send()` 开头：新增 debug 日志——extract_media 找到几个、每个路径、filter 接受几个
2. `_deliver_media()`：检查 `SendResult.success`，失败 `raise RuntimeError`
3. 投递循环：失败后等 2s 重试一次；两次都失败记 `logger.error`

**影响**：2026-07-03 升级确认：weixin.py 中 SendResult.success 检查 + 2s 重试 + logger.error 全部存在。✅ 无需重新 apply。
**回滚**：见 ADR `kb/决策记录/ADR_2026-07-03_MEDIA静默失败.md`

---

## P024 — media-miss-reshoot 插件：Agent 漏写媒体标签自动补发（2026-07-03）

**日期**：2026-07-03
**文件**：`plugins/media-miss-reshoot/__init__.py`、`plugin.yaml`
**版本**：v1.2

**问题**：Agent 在大量工具调用后偶发空回复（DeepSeek V4 已知问题），或回复中遗漏 MEDIA: 标签——文件已通过 send-media-safe.sh 拷入 hermes-media/ 但从未投递。P023（weixin.py 修复）只解决 gateway 侧静默失败，不解决 Agent 侧漏写标签。

**修复**：双 hook 机械执法插件。
1. `post_tool_call`：拦截 send-media-safe.sh 调用 → 提取 media_path → 写 /tmp/media-miss-reshoot/{session_id}
2. `transform_llm_output`：读 pending → 检查 assistant_response 是否含 `MEDIA:/` → 缺失则调用 send_weixin_direct 直发文件 → 清 pending

**开发中发现的四个 bug**：
| # | Bug | 修复 |
|---|-----|------|
| ① | `register()` 未调用 `ctx.register_hook()`——三个工作日零触发 | 加两个 register_hook 调用 |
| ② | `post_llm_call` hook 名不存在 | → `transform_llm_output` |
| ③ | 正则双重转义 `\\\\s` → 匹配字面 `\` 和 `s` | → `\\s` |
| ④ | `"MEDIA:" in response_text` 假阳性——回复中提及字面就被跳过 | → `"MEDIA:/" in response_text` |

**端到端测试**（2次全链路）：
1. `echo "test" > /tmp/x.txt` → `send-media-safe.sh` → post_tool_call 写 pending ✓
2. Agent 回复不含 MEDIA:/ → transform_llm_output 检测 → send_weixin_direct 补发 ✓
3. 用户确认收到文件 ✓

**影响**：纯插件，不修改 gateway 源码。升级后不需要重新 apply。
**回滚**：删 `plugins/media-miss-reshoot/` 目录。

---

## P025 — SearXNG Granian worker 权限拒绝死循环修复（2026-07-03）

**日期**：2026-07-03
**文件**：`/home/USER/searxng/docker-compose.yml`、`/home/USER/searxng/searxng-config/entrypoint-wrapper.sh`
**类型**：Docker 配置修改（非 gateway 源码）

**问题**：SearXNG Granian worker 启动时读 bind mount 的 settings.yml 遇 `Permission denied` → worker 崩溃 → entrypoint 无延迟立刻重启 → 又崩溃 → 死循环 6-11 次。每次循环中所有进行中的搜索被杀死，86 个正常引擎被误标记 unresponsive → disable。

**根因**：Granian 使用 Python `multiprocessing` 的 `forkserver` 模式，worker fork 瞬间 Docker bind mount 存在纳秒级窗口不可用。与 Docker 网络、gateway 重启完全无关。

**修复**：新增 `/etc/searxng/entrypoint-wrapper.sh`（POSIX sh），Granian 退出后 sleep 3s 再重试（最多 3 次），打破死循环。docker-compose.yml 新增 `entrypoint` 指向 wrapper。

**验证**：容器一次启动成功，0 次 crash，86 引擎正常，搜索 9 条结果。

**影响**：纯 Docker 配置，升级 gateway 不受影响。wrapper 文件在 bind mount 目录中，Docker image 更新后不丢失。
**回滚**：docker-compose.yml 删除 `entrypoint` 行 → `docker compose down && docker compose up -d`

---

## P026 — memory_tool.py 自动日期注入（2026-07-06）

**日期**：2026-07-06
**文件**：`tools/memory_tool.py`
**版本**：v1.0

**问题**：`memory add` 和 `apply_batch` 操作的条目缺日期，导致 TTL 执法脚本 `memory_ttl_enforce.py` 无法计算过期时间。78 条缺日期条目被跳过，Agent 自觉写日期不可靠。

**修复**：两处注入点。
1. `add()` 方法：content 写入前自动注入 `（YYYY-MM-DD）` 前缀（UTC+8），已有日期则跳过
2. `apply_batch()` 方法的 `act="add"` 分支：同样的注入逻辑

**验证**：重启 gateway 后所有 `memory add` 操作自动带日期。已存在的 208 条条目中带日期的比例将随新条目增加逐步提升。

**影响**：每次 gateway 升级需重新 apply。两处注入均为 4 行代码，apply 风险极低。
**回滚**：删除两处 `from datetime import ...` 块。


---

## P027 — iLink 限流全局冷却门（2026-07-06）

**日期**：2026-07-06
**文件**：`gateway/platforms/weixin.py`
**版本**：v1.0

**问题**：iLink 限流时断路器只拦截单条消息的重试，不拦截新 `send()`。Agent 连续回复→每条新消息独立触发 `send()`→撞上仍开着的断路器→被拒→重置冷却。反馈循环导致 4 分钟内 24 条消息全部失败（2026-07-06 01:24-01:28 实测）。

**修复**（三处）：
1. `send()` 入口：全局冷却门——断路器开着时 `await asyncio.sleep(cooldown_remaining)`，不等冷却结束不发任何消息。最多等 2 轮。
2. `send_video()`、`send_voice()`：文件投递失败时也调用 `_record_rate_limit_event()`（与 `send_document` 已有的逻辑对齐）。
3. `send_document()` 已有 P027 代码（断路器开着时记录限流事件）。

**影响**：每次 gateway 升级需重新 apply。用户体感：限流期间消息延迟最多 30s，但不丢。
**回滚**：恢复 `weixin.py.bak.*`。


## P028: patch 溯源注释 (2026-07-06)
- **文件**: `tools/file_tools.py`
- **改动**: `patch_tool` 成功后自动在文件末尾追加注释 `@hermes:patch YYYY-MM-DD | session:ID`
- **链**: `_handle_patch` 新增 `session_id` 传递 → `patch_tool` 新增 `session_id` 参数 → 注入逻辑
- **格式**: 支持 20+ 种文件扩展名的注释前缀 (# / // / -- / /*)
- **目的**: 6 个月后通过 `session_search` 找回原始对话上下文


## P029: memory(action='read') 按需拉取 (2026-07-08)
- **文件**: `tools/memory_tool.py`
- **改动**: `memory_tool()` 新增 `elif action == "read"` 分支——调用 `store._entries_for(target)` 返回全部条目，纯内存操作。gate 跳过（`if action != "read"` 不送写入审批）。docstring 更新为 `add, replace, remove, read`。
- **代码量**: +50/-5 行（含 TTL 日期注入、MEMORY_SUMMARY.md 注入、read action）
- **背景**: Hermes 记忆架构原只有写路径（add/replace/remove），读由 system prompt 注入完成。5K 摘要渐进式披露打破了此假设——需要按需拉取全量的路径。
- **关联**: T46（摘要注入）+ T47（按需拉取）= 渐进式披露完整链路
- **ADR**: `kb/决策记录/ADR_2026-07-08_memory_read_渐进式披露工具链.md`
- **回滚**: 恢复 `memory_tool.py.bak.20260708085216`

## P029 — l0-safety-guard 引用确认 + pre_llm_call hook（2026-07-10）

- **文件**: `~/.hermes/plugins/l0-safety-guard/__init__.py`, `plugin.yaml`
- **改动**:
  1. `plugin.yaml` hooks 列表新增 `pre_llm_call` 和 `transform_llm_output`（原仅 `pre_tool_call`）
  2. 新增 `_inject_quote_reminder()` 函数（15行）——pre_llm_call hook，检测用户消息含 `[引用:` → 注入提醒「输出📌已读引用」
  3. `_compliance_scan()` 新增 ⚡QR 检查——用户消息含引用但回复无 `📌 已读引用` → 违规
  4. `_append_rule19_events()` 的 RULE_MAP 新增 QR
  5. 删除旧的 ⚡Q 事后检查（事后警告→提前提醒）
- **代码量**: +50/-10 行
- **背景**: 用户发现长青多次忽略引用内容。事后 ⚡Q 无效——错话已发出。改 pre_llm_call 在 LLM 生成回复前注入提醒。plugin.yaml hooks 声明是根因——未声明 pre_llm_call 导致 hook 静默失败。
- **关联**: R17（WeChat 引用缓存）+ R18（上下文压缩硬分离）
- **session**: 20260710_213000

## P030 — 子Agent产出天生落盘（2026-07-11）

- **文件**: `~/.hermes/plugins/l0-safety-guard/__init__.py`, `plugin.yaml`
- **改动**:
  1. 新增 `post_tool_call` hook + `_post_tool_call` 函数（35行）——拦截 delegate_task，存 delegation_id → goal 映射，同步 fallback 直落盘
  2. `_inject_quote_reminder` 新增 `_save_subagent_result_if_match` 调用——检测 `[ASYNC DELEGATION BATCH COMPLETE` 前缀 → 查映射 → 落盘 → 删条目
  3. 辅助函数：`_deleg_map_read/write/delete/cleanup` + `_safe_slug` + `_write_output`（~130行）
  4. `plugin.yaml` hooks 新增 `post_tool_call`
  5. 移除探针代码（`pre_llm_full_probe.jsonl` / `post_tool_full_probe.jsonl`）
  6. `⚡QR` 违规文字缩短为 `⚡QR 缺失引用确认`
- **代码量**: +150/-55 行
- **背景**: 用户要求子Agent产出自动落盘，可回溯到聊天。四轮方案被否后，探针实测确认 `post_tool_call` + `pre_llm_call` 双hook可行——`post` 存映射，`pre` 前缀匹配 → 落盘。实测通过（deleg_b43df161 产出已自动保存）。
- **关联**: P029（引用确认——同一插件，同一 pre_llm_call hook）
- **session**: 20260711_013000



## P053 — Phase 5C 置信度评分引擎替换关键词 TTL（2026-07-13）

- **文件**: `~/.hermes/scripts/confidence_engine.py`（新增）, `~/.hermes/config/confidence_profiles.yaml`（新增）, `~/.hermes/scripts/memory_ttl_enforce.py`（修改）
- **改动**:
  1. `confidence_engine.py`（180行）——四维置信度引擎：source_weight × recency_decay + bonus + verification。含 CONSTANT_SAFELIST（10条正则防恒定事实衰减）、preprocess 集中维度提取、assess_batch 批量接口、validate_profile YAML 校验。
  2. `confidence_profiles.yaml`——memory_entry 维度配置（base_weights/recency/bonuses/thresholds/ttl_map）
  3. `memory_ttl_enforce.py`——替换 `classify_entry` 关键词分类为 `assess()` 置信度引擎。删除死代码：TTL_DAYS/TYPE_KEYWORDS/DEFAULT_TTL_DAYS/classify_entry/PERMANENT_OVERRIDES（-60行）。新增 confidence_engine import + 新 calculate_ttl（+25行）
- **代码量**: +180/+25/-60 行
- **背景**: 5B source-graded meta 建立后，TTL 仍用 6 类关键词分类（无法区分"用户亲口说"vs"LLM推理"、无时间衰减加权）。5C 构建独立置信度引擎——不止用于 memory TTL，后续可接入画像门控、深思筛选、告警评级等场景。fact_type 三分类（constant/mutable/transient）直接对应 arxiv 2604.11364 四层持久化语义。方案经四家第十人审计（内部+DS+Qwen+火山），25+3 全自动测试通过，dry-run 145条全保留。
- **关联**: P052（5B source-graded meta）、governance-decay-guardian
- **session**: 20260713_133000

## P056 — Git Push 前隐私扫描强制门控（2026-07-13）

- **文件**: `pre_push_privacy_check.sh`, `.git/hooks/pre-push`（两个仓库）, `clean_public_repo.sh`
- **改动**:
  1. pre_push_privacy_check.sh（45行）——39条正则规则覆盖凭据/人名/主机名/组织/路径/邮箱/手机号
  2. 两个仓库安装 git pre-push hook → git push 自动触发扫描，PASS 放行，FAIL 阻断
  3. clean_public_repo.sh（39条 sed 替换规则）
  4. 完整清洗公开仓库：HOST_MAIN, 协作者A, 用户, GROUP, 某外企, 某科技企业 等
- **代码量**: +100/-90 行
- **背景**: 特性注册表未清洗即推送。策略：线上公开仓库→无隐私，本地生产版本→完整。防线：①SOUL.md Rule 18 ②git pre-push hook ③事后 cron 审计（待建）。
- **关联**: agent-privacy-release-audit skill, P055
- **session**: `20260713_174500`
