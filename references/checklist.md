# LaTeX 构建自检清单（逐项打钩）

## 编译得通（诊断）
- [ ] 先探测并记录构建环境（`available` 与版本/依据一致）
- [ ] 无环境时走静态诊断，未编造日志/Exit Code/通过结论
- [ ] 每条报错有类别、文件、行号、根因、修复建议
- [ ] 级联错误归并到首个根因，未停在第一条
- [ ] 日志原文片段可复核（有环境时）

## 改得对（公式/表格/引用/版式）
- [ ] 公式符号正斜体、编号、`\label`/`\ref` 与正文一致
- [ ] 多行公式对齐符合关系符
- [ ] 表格列宽/数值对齐/三线表/跨页处理正确，无缩放掩盖溢出
- [ ] `\cite`/`\ref`/`\eqref` 与 `.bib`、`\label` 对应
- [ ] 模板必填项逐项对照，既合原文意图又合模板；未知标"待确认"

## 交得可用（交付）
- [ ] 交付可编辑 `.tex` 源码工程（主/子文件、导言区、`.bib`、模板类文件、素材）
- [ ] 非 PDF/截图/不可编译片段
- [ ] 改动前后可回溯，`change_log` 齐全

## 复现得了（收敛与清单）
- [ ] 多遍编译轮次明确（`latex → bibtex → latex ×2` 或模板规定）
- [ ] 收敛判据（无 undefined refs/cites 且 `.aux` 稳定）已检查
- [ ] 无环境时 `converged=unknown`，未声称收敛
- [ ] 清单记录发行版、模板/宏包/字体版本、命令与轮次，未知标"待确认"

## 结构与边界
- [ ] `result.json` 含 case_id/status/artifact/证据映射/uncertainties/human_review_items/change_log/self_check
- [ ] 无编造编译日志、Exit Code、版本号、benchmark 分数
- [ ] `status` 诚实（success/partial/blocked），partial/blocked 写明原因
- [ ] 诊断与核对分离，改动均有 `change_log`
- [ ] 示例标注为示意、非真实案例
