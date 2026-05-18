# 硕士论文项目

## 作者与论文

程程（Cheng Cheng），Universität Stuttgart 硕士。

论文：**基于在线强化学习的编码器故障鲁棒机器人操作策略研究**
（英: Fault-Resilient Robot Manipulation via Online Reinforcement Learning under Encoder Calibration Bias）

工作模式：
- 先写中文版打磨叙事和逻辑，再翻译英文 / 德文提交。中文是 single source of truth，英文版当前是骨架。
- 中文版 LaTeX 用 `xelatex`（`paper/cn/`），英文版用 `pdflatex`（`paper/en/`）。
- 目标 50–60 页。

## 仓库结构

| 仓库 | 用途 |
|---|---|
| 本仓库 `thesis/` | 写论文，`paper/cn/` = 中文版，`paper/en/` = 英文版 |
| `../FR-RL-lerobot/`（同级目录） | 论文支撑的实验代码（GitHub: CCgoestosleepbefore12/FR-RL-lerobot, main 分支） |

仓库内目录结构：

```
thesis/
├── docs/        项目文档（spec.md 规格书、ch6_experiment_plan.md、todo.md）
├── paper/       论文正文
│   ├── cn/      中文版（xelatex，main.tex）
│   └── en/      英文版（pdflatex，thesis.tex）
├── slides/      答辩 deck（详见下文「答辩 deck」节）
├── reports/     实验报告 PDF
├── videos/      演示视频（被 slides 以相对路径引用）
└── archive/     归档（cs-template/ 旧 CS 模板克隆、ias_latex_template.zip）
```

**论文仓库内权威文档：**
- `docs/spec.md` — 论文规格说明书（叙事线、章节结构、实验数据汇总、写作流程、进度 checklist）。比章节 `.tex` 更新得勤，是 source of truth。
- `docs/ch6_experiment_plan.md` — Ch.6 仿真实验补做计划（9 个实验，必做 / 应做 / 可选分级）
- `docs/todo.md` — 写作 todo
- `paper/cn/content/ch{1..8}_*.tex` — 中文版章节正文（`paper/en/content/` 为英文版）

接到论文相关任务时：先读 `docs/spec.md` 看当前进度（「当前任务」标记 + checklist），再读对应章节 `.tex`；涉及代码事实 / 实验数据时去 `../FR-RL-lerobot/` 拉取，不要凭空发挥。

**写作顺序**（spec.md 定）：3 → 4 → 5 → 6 → 2 → 1 → 7（依赖真机）→ 8。

## FR-RL-lerobot 文档索引

支撑仓库 `../FR-RL-lerobot/` 的关键文档（按使用频率排）：

| 文档 | 内容 | 适合查询 |
|---|---|---|
| `docs/project_progress.md` | 实验结果、待办、当前聚焦、checkpoint 索引、章节关键参数 | 「现在做到哪了」/ 最新数据 |
| `docs/task_policy_training.md` | Pick-place task policy 真机 HIL-SERL 训练全流程；BC vs SAC 算法对比；HG-DAgger 介入迭代 | Ch.4 / Ch.7 任务策略真机 |
| `docs/backup_policy_deployment.md` | Backup policy 真机部署完整流程（ArUco 标定、WiLoR 手部检测、FSM、Route A 几何对齐） | Ch.5 / Ch.7 备份策略真机 |
| `docs/fault_injection_realhw.md` | 真机 B+D 双注入点编码器偏差实现 | Ch.7 故障注入 |
| `docs/fault_injection_architecture.md` | 为什么选 B+D 注入点（设计决策） | Ch.7 设计动机 |
| `docs/fault_simulation_design.md` | 仿真侧 bias 注入设计 | Ch.3.5 仿真注入 |
| `docs/sim_exp_data.md` | sim 实验原始数据与 ablation | Ch.6 实验 |
| `docs/rl_reward.md` | Reward 设计演化（V1/V2/V3） | Ch.5.4 reward hacking |
| `docs/data_flow.md` | 观测 / 动作 / reward 数据流 | Ch.4.2 / Ch.7 观测设计 |
| `docs/real_robot_deployment_plan.md` | 两机（RT PC + GPU）整体部署方案（历史规划） | Ch.7.1 硬件架构 |
| `docs/rt_pc_runbook.md` | RT PC 开机到 server 启动流程 | Ch.7 部署细节 |
| `README.md` | 顶层入口 + 项目结构 + 文档索引 | 总览 |

代码骨架（按论文章节映射）：
- Ch.3/4 仿真 task policy: `frrl/envs/sim/panda_pick_*.py`
- Ch.5 backup: `frrl/envs/sim/panda_backup_policy_env.py`
- Ch.7 真机: `frrl/envs/real.py`、`frrl/robots/franka_real/`、`scripts/real/deploy_*.py`
- 故障注入: `frrl/fault_injection.py`（共通）、`patches/serl_franka_controllers_bias_injection.patch`（C++）
- 训练框架: `frrl/rl/{core,infra,supervisor}/`

⚠️ **关键 caveat**：sim 与 real 已显著 diverge（观测维度、bias 目标关节、旋转 scale 等），引用数据时务必看清 `[sim]` vs `[real]` 标签。

## 答辩 deck

- **在线链接**：https://ccgoestosleepbefore12.github.io/thesis/slides/en/mockup-deck/index.html
- **源文件**：`slides/en/mockup-deck/index.html`（cartesian 模板改造的英文 deck）
- **视频**：`videos/` 下的 webm/mp4，deck 用相对路径 `../../../videos/x.webm` 引用
- **GitHub Pages 设置**：repo `CCgoestosleepbefore12/thesis`，Settings → Pages → Deploy from branch `main` / root
- **更新流程**：改完 `git add slides/ videos/ && git commit && git push`，Pages 1-2 分钟自动重新部署
- **注意**：thesis 仓库必须 public 才能用免费版 GitHub Pages；`git add` 不要用 `-A`（会误加 `slides/templates/` 63MB 的模板 clone，已被 `slides/.gitignore` 排除）
