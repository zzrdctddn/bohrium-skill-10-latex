# LaTeX 工程骨架与构建清单（可编辑模板）

> 用后删除本说明。以下为示意结构，占位以 `[...]` 标出；实际内容以案例材料与模板为准，未知项写"待确认"，不得编造版本号或编译结果。

## 目录结构（可编辑）
```
project/
  main.tex            % 主文件：文档类、导言区、include
  sections/
    intro.tex
    method.tex
  refs.bib            % 文献条目
  template.cls        % 模板类文件（若为外部提供，记录来源）
  figs/               % 图片素材
  BUILD_MANIFEST.md   % 可复现构建清单
```

## main.tex（导言区示意）
```latex
\documentclass[10pt,a4paper]{[模板类名]}
\usepackage{amsmath,amssymb,bm}      % 数学
\usepackage{booktabs}                 % 三线表
\usepackage{siunitx}                  % 数值对齐（如可用）
\usepackage[numbers]{natbib}          % 按模板要求选用
\title{[待确认]}
\author{[待确认]}
\date{[待确认]}
\begin{document}
\maketitle
\include{sections/intro}
\include{sections/method}
\bibliographystyle{[模板指定样式]}
\bibliography{refs}
\end{document}
```

## 公式（示意：编号、标签、对齐）
```latex
\begin{align}
  y &= f(x; \theta) \label{eq:main} \\
  \hat{y} &= \arg\max_y p(y\mid x) \label{eq:decide}
\end{align}
% 正文引用：如式~\eqref{eq:main} 所示；符号含义与正文叙述一致。
```

## 表格（示意：三线表、列宽、数值对齐）
```latex
\begin{table}[htbp]
  \centering
  \caption{[表题，按模板要求]}
  \label{tab:result}
  \begin{tabular}{l S S}
    \toprule
    方法 & {指标A} & {指标B} \\
    \midrule
    基线 & 1.23 & 4.56 \\
    本文 & 1.30 & 4.80 \\
    \bottomrule
  \end{tabular}
\end{table}
% 禁止用 \scalebox/\resizebox 掩盖溢出；数值来自材料，未给则占位追问。
```

## BUILD_MANIFEST.md（构建清单示意）
```
TeX 发行版与版本：[待确认]（来源：环境探测/材料）
模板类文件与版本：[待确认]
宏包与版本：[待确认]
字体与版本：[待确认]
编译命令：pdflatex main.tex（引擎按模板）
轮次：latex → bibtex → latex × 2（模板另有规定从其规定）
收敛判据：无 undefined references/citations 且 .aux 稳定
素材路径：figs/, refs.bib, template.cls
```
