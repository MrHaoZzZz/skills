# cumcm-template/ —— 国赛 CUMCM 完整 LaTeX 模板

> 来源：从 [Mrite](https://github.com/i3by4t3oyt/Mrite)（MIT 开源）数模智能体的生产模板提取，包含可直接编译的国赛 `format.cls`。与上层 [`论文模板.tex`](../论文模板.tex)（`ctexart` 精简版）二选一使用。

## 文件

| 文件 | 用途 |
|------|------|
| `format.cls` | 国赛标准文档类（`cumcmthesis` 风格），提供封面、章节、图表字体等格式 |
| `论文.tex` | 主文件，`\input` 各章节；`【】`处填竞赛名称/标题 |
| `0.摘要.tex` ~ `10.附录.tex` | 各章节模板，逐节填写 |
| `5.1.问题1的建立求解.tex` / `5.1.1` / `5.1.2` | 单问拆分为「分析与准备 + 建模与求解」两个子文件 |
| `求解计划.md` | 求解阶段的结构化计划模板 |

## 编译

```bash
xelatex -interaction=nonstopmode -halt-on-error 论文.tex
xelatex -interaction=nonstopmode -halt-on-error 论文.tex
```

必须用 **XeLaTeX**（`format.cls` 内 `\RequireXeTeX`，pdflatex 直接报错）。编译两次以处理交叉引用。

## 字体依赖（重要）

`format.cls` 硬编码了以下字体，缺字体会编译失败：

| 字体 | 位置 | 说明 |
|------|------|------|
| Source Han Serif CN（思源宋体） | `./fonts/SourceHanSerifCN-Regular.otf`、`SourceHanSerifCN-Bold.otf` | **需自行下载放入本目录 `fonts/`** |
| Times New Roman | 系统 | 正文西文 |
| Arial | 系统 | 无衬线 |
| Consolas | 系统 | 等宽（macOS 无，可改 `Menlo`） |
| simkai.ttf（楷体） | 系统 | 中文楷体（macOS/Linux 无，可改系统楷体或删除该行） |

**字体不便时的替代方案**：直接用上层 [`论文模板.tex`](../论文模板.tex)（基于 `ctexart`，用系统 CJK 字体，开箱即用），内容组织仍遵循 [`论文排版规范.md`](../../references/论文排版规范.md)。

> 若要让本模板跨平台开箱即用，可编辑 `format.cls`：将 `\setmainfont{Times New Roman}`、`\setmonofont{Consolas}`、`\setCJKfamilyfont{kai}...{simkai.ttf}` 与 `./fonts/` 的思源宋体改为系统已有字体（如 `Songti SC`/`STSong`、`Menlo`、`Kaiti SC`）。
