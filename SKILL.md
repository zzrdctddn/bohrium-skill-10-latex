---
name: reproducible-latex-build
description: 把"一份 LaTeX 稿件 + 模板"转成"改得对、交得可用、可复现构建"的可编辑源码工程，并在有编译环境时按预期轮次编译、把报错定位到文件行号与根因。当用户要求"修 LaTeX 编译错误""迁入期刊/会议模板""公式表格改编号改列宽""处理 \cite/\ref 未定义""补齐模板必填项""生成可复现构建清单"时使用。输出含错误诊断树、改动对齐表、多遍编译收敛记录、可复现构建清单与自检结果。
version: 1.0.0
license: MIT
---

# LaTeX 写作与编译（Reproducible LaTeX Build & Diagnose）

> 本 Skill 是**指导 Agent 如何完成 LaTeX 写作与编译任务的方法论与工作流**，不是预先编译好的答案。它规定 Agent 先探测构建环境、再按诊断树定位根因、按规则改源码、按多遍编译收敛验证，最后交付可编辑源码与可复现构建记录。核心纪律：**有环境就真编译、无环境就静态诊断并如实标注，绝不编造编译日志、Exit Code 或"编译通过"**。

## 1. 用途与适用边界

**解决什么**：基础 Agent 处理 LaTeX 的常见失败——停在第一条报错就放弃、把级联错误当独立错误逐个瞎改、公式符号/编号与正文不符、表格靠缩放掩盖溢出、交叉引用与参考文献多遍编译不收敛、交付 PDF 或一段无法编译的片段、记录不出他人可复现的模板/宏包/字体/命令与轮次。本 Skill 用"错误诊断树 + 公式表格规则 + 多遍编译收敛 + 可复现构建清单"消除这些失败。

**触发场景**：修复编译错误与警告；把稿件迁入指定期刊/会议/学位论文模板并补必填项；改公式符号、编号、多行对齐；改表格列宽、数值对齐、三线表、跨页；处理 `\cite`/`\ref`/`\eqref` 未定义与 `.bib` 对应；验证多遍编译收敛；整理可编辑源码结构与构建清单。

**不该用**：任务要求生成正式测试案例答案或 benchmark 分数时（拒绝，作品是方法论非答案）；环境无 TeX 引擎却要求"确认编译通过"时（只能做静态诊断并标 `unverified`/`partial`，不得编造）；用户只要把 PDF 转图片而不要源码时（转用文档处理工具）。

**输入**：`/input/case.json`（含 `case_id`、任务说明、LaTeX 源码/工程、模板与必填项、目标改动点）与 `task.md`。**输出**：`/app/output/result.json`（字段见第 4 节）+ 可编辑的 `.tex` 源码与构建清单 `BUILD_MANIFEST.md`。

## 2. 核心方法（RLBD）

本 Skill 的方法名为 **RLBD = Reproducible LaTeX Build & Diagnose（可复现构建的 LaTeX 诊断）**，四个不可跳过的机制，对应题面四类关键问题：

1. **编译得通 — 编译错误诊断树（Error Diagnosis Tree）**：先把报错按类别归因（缺文件/缺宏包、语法、数学环境、表格、引用、字体、级联），定位到具体**文件+行号+根因**；级联错误归并到首个根因，不逐条瞎改；每改一处即重编译验证。无编译环境时走静态诊断分支（检查括号/环境配对、宏包与命令依赖、文件引用路径），并把验证状态标 `unverified`。
2. **改得对 — 公式与表格规则（Math & Table Rules）**：公式符号的正斜体、编号与 `\label`/`\ref`、多行对齐须与正文叙述一致；表格列宽、对齐、三线表、跨页、数值与来源一致，禁止靠 `\scalebox` 掩盖溢出；版式与模板必填项逐项对照，既合原文意图又合模板要求。
3. **交得可用 — 可编辑源码交付（Editable Source Deliverable）**：交付结构清楚、可继续编辑维护的源码（主文件/子文件、导言区、`.bib`、模板类文件、素材路径），保留改动前后可回溯 diff；**不把 PDF、截图或无法编译的片段当结果**。
4. **复现得了 — 多遍编译收敛 + 可复现构建清单（Multi-pass Convergence & Build Manifest）**：按 `latex → bibtex → latex ×2` 的轮次处理引用与参考文献；收敛判据为无 "undefined references/citations" 警告且 `.aux` 稳定；清单记录 TeX 发行版、模板版本、宏包与字体版本、编译命令与轮次，使他人按记录得到同一产物。

另贯穿铁律：**编译闭环与核对分离**（先诊断改码，再以审阅者身份独立核对，两遍不可合并）；**不编造编译事实**（没有真实运行就写 `static_only`，不写假 Exit Code、假日志、假"通过"）。

## 3. 工作流（SOP）

严格按顺序执行，每步都有检查点；检查点不过则回退该步或转入异常处理。

**Step 0 · 输入解析与边界确认**
- 读 `case.json` 与 `task.md`，提取：目标模板与版本、任务改动点（报错/公式/表格/引用/版式/必填项）、交付形式、明确禁止事项。
- 检查点：列出改动清单；明确"这是方法论指导任务，不提交预制答案"；任何必需材料缺失登记 `uncertainties`。

**Step 1 · 工程结构盘点（Inventory）**
- 清点并快照：主文件、子文件、导言区、宏包加载顺序、`.bib`、`.cls/.sty`、图片与字体素材；建立文件路径与行号索引。
- 检查点：结构可回溯；缺文件/路径错误先记录，不假设内容。

**Step 2 · 构建环境探测（Environment Probe）**
- 探测 TeX 引擎是否可用及发行版/版本（`pdflatex`/`xelatex`/`lualatex`、TeX Live/MiKTeX 版本）；记录探测依据；**不可用则明确标记 `available=false`，后续只做静态诊断**。
- 检查点：`environment_probe.available` 与记录一致；绝不把"未探测到"写成"编译通过"。

**Step 3 · 编译/静态诊断与错误定位（Diagnosis Tree）**
- 有引擎：按预期轮次编译，读日志把每条报错分类、定位文件/行号/根因，级联错误归并到首因，写 `error_diagnosis`；日志原文保留片段供复核。
- 无引擎：按 `references/latex_rules.md` 做静态检查（环境配对、宏包依赖、命令定义、文件路径、必填项），结论标 `static_only`，`uncertainties` 写明"未实际编译"。
- 检查点：每条问题有类别、位置、根因、修复建议；不停在第一条错误；不编造日志。

**Step 4 · 按规则改源码（Edit & Verify）**
- 按公式/表格/引用/版式规则实施改动：公式符号与编号对齐正文、表格列宽与数值对齐、引用与 `.bib` 对应、模板必填项补齐；每条改动记录意图、前后对照、是否合模板。
- 检查点：改动既合原文意图又合模板；无靠缩放掩盖溢出；改动写入 `edit_verification` 与 `change_log`。

**Step 5 · 多遍编译收敛（Multi-pass Convergence）**
- 有引擎：执行 `latex → bibtex → latex ×2`（或模板指定轮次），逐轮记录命令、警告与日志片段；检查未定义引用/文献是否清零、`.aux` 是否稳定；未收敛则回到 Step 3/4。
- 无引擎：基于源码给出"预期轮次与收敛检查项"，`convergence.converged` 只能标 `unknown`，并在 `uncertainties` 说明需在有环境处复核。
- 检查点：有环境时给出真实收敛结论；无环境时绝不声称收敛。

**Step 6 · 可编辑交付与构建清单（Deliverable & Manifest）**
- 交付可编辑源码工程与 `BUILD_MANIFEST.md`，逐项记录发行版、模板版本、宏包/字体版本、命令与轮次、素材路径；每项注明来源，未知的写"待确认"而非编造版本号。
- 检查点：交付物为可编辑 `.tex` 源码（非 PDF/截图）；清单字段齐全或明确标注未知。

**Step 7 · 独立核对与输出**
- 以审阅者身份逐项走 `references/checklist.md`；生成 `result.json` 与源码工程；所有改动写 `change_log`，未通过项写清原因与 `status`。
- 检查点：自检表全过或每项未过有 `change_log`；`status` 诚实（有环境且收敛=success；静态诊断未验证=partial；材料缺失=blocked）；示例明确为示意。

## 4. 输出契约（result.json）

```json
{
  "case_id": "<原样带回>",
  "status": "success | partial | blocked",
  "note": "示意文本请标注，非真实案例",
  "artifact": {
    "main_file": "<主 .tex 路径>",
    "tex_sources": ["<可编辑源码路径列表>"],
    "bib_file": "<.bib 路径>",
    "build_manifest": "<BUILD_MANIFEST.md 路径>",
    "editable": true,
    "delivery_note": "<非 PDF/截图的说明>"
  },
  "environment_probe": {
    "tex_engine": "<pdflatex|xelatex|lualatex|none>",
    "distribution": "<TeX Live|MiKTeX|unknown>",
    "version": "<版本或 未知>",
    "available": false,
    "basis": "<探测依据；不可用时写明>"
  },
  "error_diagnosis": [
    {"error_id":"E1","raw_message":"<日志原文片段或 静态检查项>","category":"missing_package|file_not_found|syntax|math|table|reference|font|cascade","file":"main.tex","line":42,"root_cause":"...","fix":"...","cascade":false,"status":"root_caused|static_only"}
  ],
  "edit_verification": [
    {"edit_id":"X1","target":"公式(3)","intent":"符号与正文一致","before":"...","after":"...","template_compliant":true,"source_aligned":true}
  ],
  "compile_passes": [
    {"pass_no":1,"command":"pdflatex main.tex","exit_status":"<实际或 static_unverified>","warnings":["..."],"log_excerpt":"...","source":"actual|static"}
  ],
  "convergence": {
    "converged": "true|false|unknown",
    "criterion": "无 undefined references/citations 且 .aux 稳定",
    "undefined_refs": ["..."],
    "undefined_citations": ["..."],
    "basis": "<实际日志或 未编译说明>"
  },
  "build_manifest": {
    "tex_distribution": "<版本或 待确认>",
    "template_version": "<版本或 待确认>",
    "packages": [{"name":"<宏包>","version":"<版本或 待确认>"}],
    "fonts": ["<字体或 待确认>"],
    "commands": ["<编译命令>"],
    "pass_count": "<轮次>",
    "source": "<材料依据>"
  },
  "fact_source_map": [{"item":"<模板必填项/宏包版本>", "value":"...", "source_location":"..."}],
  "uncertainties": ["..."],
  "human_review_items": ["..."],
  "change_log": [{"step":4,"action":"修正表格列宽","reason":"原表溢出被 scalebox 掩盖"}],
  "self_check": {
    "compiles_verified": false,
    "errors_root_caused": true,
    "edits_intent_compliant": true,
    "editable_source_delivered": true,
    "build_record_complete": true,
    "no_fabrication": true
  }
}
```

字段原则：`status` 诚实反映完成情况；`environment_probe.available=false` 时 `compile_passes.source=static`、`convergence.converged=unknown`、`compiles_verified=false`，绝不可写成通过；`partial`/`blocked` 必须写清 `uncertainties`；不得编造编译日志、Exit Code、宏包版本或 benchmark 分数；示例一律标注为示意。

## 5. 质量自检清单

执行前完整读取 `references/checklist.md`，逐项打钩。关键硬项：

- [ ] 先探测构建环境并如实记录；无环境时只做静态诊断并标 `unverified`
- [ ] 报错定位到文件/行号/根因，级联错误归并，未停在第一条
- [ ] 公式符号/编号、表格数值/列宽、引用、版式必填项合原文意图又合模板
- [ ] 交付可编辑 `.tex` 源码工程与清单，非 PDF/截图/不可编译片段
- [ ] 多遍编译轮次与收敛判据明确；无环境时 `converged=unknown`
- [ ] 构建清单记录发行版、模板/宏包/字体版本、命令与轮次，未知标"待确认"
- [ ] 无编造日志、Exit Code、版本号、benchmark
- [ ] `status` 诚实；改动有 `change_log`；诊断与核对分离

## 6. 异常处理与边界

| 情况 | 处理 |
|---|---|
| 无 TeX 引擎/无法编译 | `available=false`，静态诊断，`status=partial`，`converged=unknown`，不编造日志 |
| 缺 `.cls/.sty`/模板类文件 | 记录缺文件路径，`status=partial`，说明需提供模板，不假设内容 |
| 缺字体导致编译失败 | 定位到字体设置行，给替换或安装建议，标 `uncertainty` |
| 报错级联（一条引发多条） | 归并到首个根因，先修根因再重编译，避免逐条瞎改 |
| 表格溢出 | 改列宽/列格式或拆表，禁止用 `\scalebox` 掩盖 |
| 交叉引用不收敛 | 检查 `\label` 位置与多遍轮次，按 Step 5 处理，未收敛不宣称完成 |
| 模板必填项未知 | 标"待确认"并进 `human_review_items`，不臆造作者/基金/日期 |
| 权限不足/文件损坏 | 说明受阻项，`status=blocked`，给出补充后可继续的条件 |

## 7. 最小调用示例（示意，非真实案例）

**输入**（`case.json` 片段）：
```json
{"case_id":"demo-010","task":"修复 main.tex 编译错误并迁入会议模板，补齐必填项","params":{"template":"conf-template.cls","engine":"pdflatex"},"materials":["main.tex","sections/","refs.bib","conf-template.cls"]}
```
**调用**：加载本 Skill 后，用户说"用 reproducible-latex-build 修编译并迁入模板"。

**示意输出**（节选，假设环境不可用）：
```json
{"case_id":"demo-010","status":"partial","environment_probe":{"available":false,"basis":"未检测到 TeX 引擎，按静态诊断处理"},"error_diagnosis":[{"error_id":"E1","category":"missing_package","file":"main.tex","line":12,"root_cause":"静态检查：使用了未加载宏包命令","status":"static_only"}],"convergence":{"converged":"unknown","basis":"未实际编译"},"self_check":{"compiles_verified":false,"no_fabrication":true}}
```
> 上述为演示接口用的虚构示例，不代表任何真实案例结果；`static_only`/`unknown` 体现不编造。

## 8. 资源清单

- `references/method.md` —— RLBD 四机制细则与反例库
- `references/checklist.md` —— 覆盖四类关键问题的逐项自检清单
- `references/schema.json` —— 输出字段字典（JSON Schema）
- `references/latex_rules.md` —— 错误分类树、公式/表格规则、引用与模板适配规则
- `references/build_manifest_rules.md` —— 清单字段字典、多遍收敛程序、静态诊断协议、诚实纪律
- `assets/latex_project_template.md` —— 可编辑 LaTeX 工程骨架与清单模板
- `examples/` —— 最小调用示例（标注为示例，非真实案例）
