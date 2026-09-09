# 数学建模竞赛 AI 工作流（AGENTS.md）

本文件是项目工作流的权威 policy 源，供 `workflow-orchestrator` 读取，也是 opencode 自动加载的项目指令。竞赛解题时遵循本文件定义的阶段、门控与人机分工。

## 核心哲学

- **AI 负责机械正确性，人负责建模判断。**
- 从目标、对象、约束、数据、输出、变量、关系出发，不从模型名称或偏好算法出发。
- 区分「假设 / 观察 / 推导 / 已验证结论」。
- 不虚构数据、假设、证据、结果或参考文献。

## 两套 Skill 的定位

本工作区有 41 个 skill，分两层：

**主流水线（MathModeling-skills，28 个）**：门控驱动、人机分工、反伪造。每个阶段有通过条件，数字冻结进 `frozen_numbers.json`，方法选择由人决定。**这是竞赛解题的主干。**

**增强层（其余 13 个）**：在对应门控阶段内被调用，提供中文方法论、绘图美化、去 AI 味、代码规范、数学计算。**增强层只注入内容与技巧，绝不越过人机分工边界**——方法选择、数字含义、物理/贡献解释仍由人决定。

| 增强 skill | 归属阶段 | 作用 |
|-----------|---------|------|
| math-modeling-problem-analysis | 阶段 1/2 | 中文 A-F 题型对照、选题、创新三方向 |
| math-modeling-data-preprocessing | 阶段 1 | 中文数据预处理方法论（双语言） |
| math-modeling-solving-validation | 阶段 4 | 检验类型补充、三层结果分析 |
| math-modeling-plotting | 阶段 4 | 国奖级绘图美化（雷达/3D柱状/饼图/置信区间） |
| math-modeling-paper-writing | 阶段 5 | 中文论文结构/摘要/格式模板 |
| math-modeling-ai-trace-removal | 阶段 6 | 数模论文 AI 痕迹识别清单 |
| math-modeling-ai-workflow | 全局参考 | 四阶段万能流程（仅作中文方法论参考，不取代门控） |
| qu-ai-wei | 阶段 6 | 简体中文去 AI 味重写（保留事实/学术语域） |
| article-writing | 阶段 5 | 长文写作、禁套话清单 |
| literature-review | 阶段 1 | 系统文献综述 |
| python-patterns / pytorch-patterns | 阶段 3 | Python/PyTorch 代码规范 |
| math-computation | 阶段 3 | SymPy/NumPy/SciPy 符号与数值计算 |

## 配置

`planning/session_config.json` 两个独立开关：

```json
{ "interaction_mode": "learning", "rigor_profile": "lean" }
```

- `interaction_mode`：`learning`（多问）或 `speed`（少问），只影响提问密度。
- `rigor_profile`：`lean`（探索迭代）或 `submission`（论文交接/定稿），只影响产物与审计密度，**不改变人机分工边界**。
- 新工作区默认 `learning + lean`。

## 工作流阶段与门控

### 阶段 0 · 初始化
- `workflow-orchestrator` 读 `session_config.json`，检查运行时与骨架，报告当前状态，路由单一下一步动作。

### 阶段 1 · 赛题结构化 —— G1 PROBLEM_FRAMED
- `problem-parser`：解析为 goals/objects/data/constraints/outputs/subquestions/dependencies/variables，写 `planning/parse/problem_parse.json`。
- `problem-classifier`：每小问打题型标签，写 `planning/classification/`。
- `related-paper-analyzer`：找文献，不虚构引用。
- `symbol-table-builder`：全局符号表 `planning/symbol_table.md`。
- `model-assumptions-builder`：必要假设 vs 简化假设，`planning/model_assumptions.md`。
- `data-auditor-cleaner`：审计原始数据，产出清洗副本 + 数据画像；`workspace/data_raw/` 只读。
- 增强：`math-modeling-problem-analysis`（A-F 题型、选题）、`literature-review`（综述）、`math-modeling-data-preprocessing`（中文方法论）。
- **通过条件**：parse、classification、data inventory、success criteria、human framing 齐备，无人占位。

### 阶段 2 · 方法筛选 —— G2 METHOD_SCREENED → G2.5 👤 人选择
- `method-selector`：构建 1 主候选 + 1 可用基线 + 最多 1 条件备用，写 `qx_method_card.md` 与 `risk_probe_summary.json`。
- `decision-prompt-builder`：在真正建模判断点出示选择卡。
- `modeler-decision-logger`：把人的答案追加到 `methods/Qx/qx_decisions.jsonl`。
- 增强：`math-modeling-problem-analysis`（创新三方向：算法改进/跨领域迁移/多模型融合）。
- **G2 通过**：method card 定义主候选+基线，基线能完成真实任务，risk probe 覆盖适用检查（含输出退化），主/基线 verdict 为 PASS 或 CONDITIONAL。
- **G2.5 通过（人）**：`qx_decisions.jsonl` 含人的 `DECIDED` 方法选择并引用 probe 证据。代码生成仅当 G2+G2.5 都通过。

### 阶段 3 · 代码生成与审查 —— G3 CODE_AND_EXPERIMENT_REVIEWED
- `model-code-analyzer`：规划 `experiments/roundN/` 布局与 `run_summary.json` 字段。
- `python-model-code-generator` / `matlab-model-code-generator`：生成代码（固定 `SEED=2026`，MATLAB 避开 Live Script/App Designer）。
- `code-reviewer`：识别语言并路由到对应审查器。
- `python-code-reviewer` / `matlab-code-reviewer`：写 `code/Qx/reviews/qx_<lang>_review.json`，含 5 项命名检查（syntax/input_contract/method_alignment/reproducibility/output_contract）。
- 增强：`math-computation`（SymPy/SciPy）、`python-patterns`、`pytorch-patterns`（代码规范）。
- **通过条件**：主方法与基线已执行，`run_summary.json` 完整，语言审查含 5 项命名检查通过。

### 阶段 4 · 结果 / 鲁棒性 / 冻结 —— G4 👤 人判断
- `result-report-generator`：仅在人判断点或终轮出报告。
- `robustness-checker`：只跑风险相关的敏感性/误差/基线/浓度检查，不凑通用清单。
- `final-method-explainer`：写 `qx_final_method_explanation.md`。
- `figure-table-planner`：图分四类（1 诊断/2 对比/3 论文/4 附录），诊断图不进论文。
- `math-figure-generator`：从证据生成图并 render-check，通过才标为论文图。
- `solution-package-builder`：构建写手包并产出 `frozen_numbers.json`（禁止手改）。
- 增强：`math-modeling-solving-validation`（检验补充）、`math-modeling-plotting`（美化）。
- **G4 通过（人）**：结果/稳定性判定引用计算证据；`submission` 下还需终方法说明、终结果分析、鲁棒性报告、写手包、`frozen_numbers.json` 齐备且当前。

### 阶段 5 · 论文撰写 —— G5 PAPER_SECTION_READY
- `paper-section-writer`：从写手包 + frozen 快照撰写，人拥有的物理解释/贡献来自决策台账。
- `paper-polisher`：检查时态、hedging、过度声明、公式一致性。
- `reference-manager`：生成 BibTeX 并校验引用真实，伪造引用阻塞。
- 增强：`math-modeling-paper-writing`（中文模板）、`article-writing`（长文）。
- **通过条件**：三写作规则、frozen 数字来源、人确认的解释/声明范围、已验证图。

### 阶段 6 · 去 AI 味定稿
- `qu-ai-wei`：简体中文去 AI 味重写，保留事实/证据强度/学术语域，不把学术论文改成口语。
- `math-modeling-ai-trace-removal`：数模论文 AI 痕迹自查（默认字体/浅色调/分点/公式换行）。

### 阶段 7 · 三重审计提交 —— G6 FINAL_AUDIT_PASSED
- `consistency-auditor`：跨文件一致性（数字/文件名/符号 vs `frozen_numbers.json` + 磁盘 + 符号表）。
- `completeness-auditor`：语义证据完整性。
- `quality-assurance-auditor`：工作流完整性 + 三核心规则 + 反伪造，最后签署。
- 三者都必须 PASS，任一不过不得提交。

## 人机决策约定

- 人的决策记录在每小问一个 append-only 台账 `methods/Qx/qx_decisions.jsonl`（全局前期决策用 `planning/framing_decisions.jsonl`）。
- AI 可出示证据与选项，但**不得发起**人的方法选择、理由、置信度、物理/贡献解释或提交授权。
- 选择卡只用于建模判断点，通常每小问两次：①方法筛选前（输出形式/可解释性优先级/不可接受失败/实验预算）；②首次有效实验后（proceed/adjust/fallback）。可选第三次：冻结前声明范围与置信度。
- 不问人可机械核查的事。

## 风险探测契约（替代通用 PoC）

`methods/Qx/probes/risk_probe_summary.json` 必须含：
- `executability`、`data_coverage`、`assumption_checks`、`output_degeneracy`（含浓度/熵/唯一值）、`perturbation_sensitivity`、`scale_check`、`verdict`（PASS/CONDITIONAL/FAIL）。

## 冻结数字约定

- 数字流向：code → results → freeze → paper。
- 绝不手改 `frozen_numbers.json`。
- 改冻结值：解冻 → 改 canonical 源 → 重跑受影响工作 → 重冻结，并记录到 `freeze_change_log.md`。

## 变更影响与审计

| 级别 | 含义 | 动作 |
|------|------|------|
| NONE | scratch/格式/注释 | 无 |
| LOCAL | 冻结前探索代码/方法卡 | 本地测试/审查 |
| CANONICAL | schema/单位/符号/公式/参数/图路径 | 受影响 Qx 定向一致性 |
| FROZEN | 影响冻结值或论文声明 | 解冻记录→重跑→重冻结→定向一致性 |

仅 `submission` 下、终装订前运行完整三重审计一次；不因多文件变更就做全工作区审计。

## 精简 vs 提交

- **lean**：只维护 manifest、method card、决策台账、probe summary、run summary；不要求每轮报告/冻结/论文/终审。
- **submission**：增加终说明、审查、分析、鲁棒性、写手包、冻结、论文、G6 三重审计。

## 目录骨架

```text
project/
├── planning/            # parse/ classification/ manifests/ symbol_table.md model_assumptions.md session_config.json
├── methods/Qx/          # qx_method_card.md qx_decisions.jsonl probes/
├── code/Qx/ + matlab/Qx/  # reviews/qx_<lang>_review.json
├── results/Qx/          # experiments/roundN/ + reports/(写手包 + frozen_numbers.json)
├── robustness/Qx/
├── paper/               # sections/ figures/ audits/ refs.bib main.tex qa_report.md
├── workspace/           # data_raw/(只读) data_clean/ archived/
└── scratch/             # 临时，无需可复现
```

硬规则：`data_raw/` 只读；论文每个数字都在 `frozen_numbers.json`；`[REJECTED]` 方法自动归档；`frozen_numbers.json` 绝不手改。

## 竞赛规则自查

- 本工作流默认按最严格的可信解读设计，可产出 `ai_use_disclosure.md` 记录 AI 起草 vs 人撰写。
- 各竞赛（COMAP / CUMCM / 各省赛）AI 政策逐年不同，最终合规判断由参赛者自负。
