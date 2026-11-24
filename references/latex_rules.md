# LaTeX 规则：错误分类树、公式、表格、引用与模板

## 一、错误分类树（category → 典型迹象 → 根因 → 修复方向）
| 类别 | 典型迹象 | 常见根因 | 修复方向 |
|---|---|---|---|
| `missing_package` | `undefined control sequence` 指某命令 | 使用命令的宏包未加载 | 确认宏包并加 `\usepackage`，注意加载顺序 |
| `file_not_found` | `File ... not found` | 图片/子文件/`.cls`/`.sty` 路径错 | 核对路径与文件名，补素材或说明缺失 |
| `syntax` | `Missing }`/`Runaway argument` | 括号/环境不配对 | 定位未闭合处，按行号修正 |
| `math` | `Missing $ inserted`/对齐报错 | 数学环境或符号用法错 | 检查 `$...$`/环境、上下标、正斜体 |
| `table` | `Misplaced \omit`/`Extra alignment tab` | 列数不匹配、`\multicolumn` 错 | 对齐列格式与单元格数，改列宽 |
| `reference` | `Citation ... undefined`/`Reference ... undefined` | `.bib` 缺项、`\label` 缺失或多遍未收敛 | 补条目/`\label`，按多遍程序编译 |
| `font` | `Font ... not found`/缺字 | 字体未装或引擎不匹配 | 换引擎/字体或说明依赖，标 `uncertainty` |
| `cascade` | 首条错误后大量衍生报错 | 由首个根因引发 | 归并到首因，先修根因再重编译 |

## 二、公式规则
- 变量用斜体、函数名/算子/单位/多字母缩写用正体（如 `\mathrm`, `\operatorname`）。
- 编号环境（`equation`/`align`）`\label` 置于环境内编号之后；`\eqref` 引用得编号。
- 多行按关系符（`=`, `≤`, `≈`）对齐，用 `&` 标记对齐点；避免用空格硬对齐。
- 上下标、希腊字母、求和上下限符合原意；单位与数值间留适当间距。
- 正文叙述提到的符号、编号与公式显示必须一致，不一致以原文意图为准并写 `change_log`。

## 三、表格规则
- 列格式（`l/c/r/p{宽}`）与内容匹配；数值按小数点对齐（可用 `siunitx` 的 `S` 列，前提是宏包可用）。
- 三线表用 `booktabs`，不用竖线与多余横线；表题位置符合模板。
- 宽表跨页用长表环境；禁止靠 `\scalebox`/`\resizebox` 缩小掩盖溢出。
- 表中数值与来源材料一致；材料未给的数值保留占位并追问，不臆造。

## 四、引用与参考文献规则
- `\cite{key}` 的 key 须在 `.bib` 中存在；正文 `\label` 与 `\ref`/`\eqref` 一一对应。
- 多遍编译：`latex → bibtex → latex ×2`（模板另有规定从其规定）；收敛判据为无未定义引用/文献且 `.aux` 稳定。
- 出现 `??` 不直接改正文写死编号，先查 `\label` 位置与编译轮次。
- 文献样式与排序按模板要求，不擅自更换样式文件。

## 五、模板适配规则
- 逐项列出模板必填项（文档类、标题、作者、日期、基金、分类、摘要格式等）并对照材料。
- 未知必填项写"待确认"进 `human_review_items`，不编造作者、基金、日期。
- 迁入时保留原文意图：改动前后对照写入 `edit_verification`，确认合模板且未改变原意。
