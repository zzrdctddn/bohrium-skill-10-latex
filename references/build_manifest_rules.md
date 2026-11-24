# 构建清单、收敛程序与静态诊断协议

## 一、可复现构建清单字段字典
| 字段 | 含义 | 可接受来源 | 缺失处理 |
|---|---|---|---|
| tex_distribution | TeX 发行版与版本（如 TeX Live 2023） | 环境探测、材料声明 | "待确认"，不编造年份 |
| template_version | 模板类文件版本/获取地址 | 模板文件头、材料说明 | "待确认" |
| packages | 宏包名与版本 | 导言区、`.log`、材料 | 未知版本写"待确认" |
| fonts | 字体族与版本 | 导言区、材料 | 未知写"待确认" |
| commands | 完整编译命令（含引擎参数） | 模板说明、材料 | 按引擎给默认命令并注明来源 |
| pass_count | 编译轮次与顺序 | 收敛程序 | 模板另有规定从其规定 |

清单以"他人按记录可复现同一产物"为验收标准；每一字段注明 `source`（材料依据或探测结果）。

## 二、多遍编译收敛程序
1. `latex <main>`（生成 `.aux`、首次解析引用）
2. `bibtex <main>`（生成参考文献，若使用 `.bib`）
3. `latex <main>`（解析引用与文献编号）
4. `latex <main>`（确认 `.aux` 稳定、交叉引用收敛）
5. 检查：日志中无 `undefined references` / `undefined citations`；`.aux` 连续两轮无变化。
6. 未收敛回到诊断/改码步骤；不得在未收敛时把 `converged` 写成 `true`。

模板若指定不同轮次（如 `latexmk`），按模板规定并在清单中记录实际命令与轮次。

## 三、静态诊断协议（无编译环境时）
- 触发条件：`environment_probe.available=false`。
- 可做的静态检查：环境 `begin/end` 配对、括号配对、宏包与命令依赖、`\label`/`\ref`/`\cite` 与 `.bib` key 对应、文件与素材路径、模板必填项、表格列数一致性。
- 必须标注：`error_diagnosis.status=static_only`、`compile_passes.source=static`、`convergence.converged=unknown`、`compiles_verified=false`、`status=partial`。
- 禁止：编造日志片段、Exit Code、警告列表、收敛结论、"编译成功"字样。

## 四、诚实纪律（贯穿）
- 只有真实运行的日志才能写入 `raw_message`/`log_excerpt`；静态检查项写明"静态检查"。
- 版本号、宏包版本、字体版本未知一律"待确认"，不猜。
- 不虚构 benchmark 分数或软件状态；平台记录的模型/_case 版本以平台为准。
- 示例输出必须标注"示意、非真实案例"。
