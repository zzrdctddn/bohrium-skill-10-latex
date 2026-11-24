# LaTeX 写作与编译 Skill（reproducible-latex-build）

一个指导 Agent 完成"可复现构建 + 错误诊断"的 LaTeX Skill。核心：编译错误诊断树 + 公式与表格规则 + 多遍编译收敛 + 可复现构建清单 + 不编造编译事实。

## 目录结构
- `SKILL.md` —— Skill 本体（含 YAML frontmatter、RLBD 工作流、输出契约）
- `references/` —— 方法细则、自检清单、字段字典、LaTeX 规则、构建清单规则
  - `method.md` —— RLBD 四机制细则与反例库
  - `checklist.md` —— 覆盖"编译得通/改得对/交得可用/复现得了"的分组自检清单
  - `schema.json` —— result.json 的 JSON Schema（draft-07）
  - `latex_rules.md` —— 错误分类树、公式/表格规则、引用与模板适配规则
  - `build_manifest_rules.md` —— 清单字段字典、多遍收敛程序、静态诊断协议、诚实纪律
- `assets/latex_project_template.md` —— 可编辑 LaTeX 工程骨架与清单模板
- `examples/` —— 最小调用示例（已标注为示例，非真实案例）
- `GITHUB_URL.txt` —— 提交前填 GitHub 公开链接

## 使用方式
1. 将本目录作为 Skill 安装到 Agent（平台约定路径 `/skills/weekly-skill`）。
2. 输入 `/input/case.json`（源码/工程、模板与必填项、目标改动点）与 `task.md`，自然语言下指令如"修复编译错误并迁入模板"。
3. 按 SKILL.md 的 SOP（Step 0~7）执行，输出到 `/app/output/result.json`、可编辑 `.tex` 源码工程与 `BUILD_MANIFEST.md`。

## 依赖
本 Skill 本体仅需文本处理能力。**编译验证依赖运行环境提供 TeX 发行版**（如 TeX Live / MiKTeX，含 `pdflatex`/`xelatex`/`lualatex` 与 `bibtex`）；Step 2 先探测，若不可用则走静态诊断分支，如实标 `available=false`、`converged=unknown`、`compiles_verified=false`，不编造编译日志或 Exit Code。宏包/字体版本以案例材料给出的为准，未知写"待确认"。

## 适用与限制
- 适用：LaTeX 编译错误定位与修复、模板适配与必填项补齐、公式/表格/交叉引用改动、可复现构建清单编制。
- 限制：不提交预制答案或 benchmark 分数；无编译环境时只能静态诊断，不宣称编译通过；模板类文件、字体、版本号缺失时标 `partial`/`blocked` 并追问；不替代作者确认只有本人知道的模板必填项（作者、基金、日期等）。

## 与评分维度的对应
- 实际效果(40)：SOP Step 3~6 落实错误定位、按规则改码、多遍收敛与可编辑交付；日志片段与改动对照提供复核（见第 3、4 节）。
- 使用增益(25)：诊断树与收敛程序减少查文档、反复编译和盲改成本；增益对照由平台白板给出，本作品不自带分数（见第 2、7 节）。
- 创新增益(15)：错误分类树、级联归并、多遍收敛判据、可复现构建清单为新增能力（见第 2 节与 `references/method.md`）。
- 可靠体验(10)：异常处理覆盖无引擎、缺模板/字体、报错级联、引用不收敛等情形（见第 6 节）。
- 生态贡献(10)：规则文件、清单规则、工程模板、示例与反例库可供复用（见第 8 节与 `references/`）。

## Runtime contract
平台加载 Skill 到 `/skills/weekly-skill`，输入在 `/input/case.json` 与 `task.md`，输出到 `/app/output/result.json`（含全部输出契约字段与 `case_id`）。资源最多 50 个 UTF-8 文本文件、单文件 ≤500KB、总计 ≤2MB、无软链。执行预算 64 次模型请求 / 900 秒 / 每次 16384 token。编译环境由平台提供或在 `environment_probe` 中如实记录为不可用。
