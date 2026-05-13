# PPT 大纲与实施 spec

## 0. Meta

- 时长：15–20 min（讲 ~16 min + Q&A ~4 min）
- 张数：18（每张 45–60 s）
- 模板：Signal（HTML, MIT），navy + 暗金 + 衬线 + Noto SC
- 输出：HTML deck，浏览器播放，提前导出 PDF 兜底
- 当前状态：S1 / S6 / S13 mockup 完成，待用户验收模板质感

## 1. Slide 大纲（v0.2，含 demo 锚点）

切换到全英文 cartesian 模板；S1 后插入 Agenda；删原 S3"三贡献页"开头版（合并入末尾 Conclusion）。共 16 张。

| # | 标题 | 主视觉 | 一句口播要点 | demo |
|---|---|---|---|---|
| **1** | Cover | display 衬线大字 + Uni logo 左上 + IAS 右下 + 元数据 | 自我介绍 + 标题朗读 | — |
| **2** | Agenda | 2×3 grid 6 高层 section（I-VI）+ 副标题 | 概览全场结构 | — |
| **3** | Motivation | 左侧 hook + 数字 + claim · 右侧 SVG drift schematic | 0.1 rad → 8-10 cm drift · 工厂停机痛点 · 在线适应 | — |
| **4** | POMDP causal chain | 论文 fig_3_2 英文版（One-Source Three-Effect） | 单 b 三效应 → MDP 退化为 POMDP | — |
| **5** | Identifiability propositions | 命题 1（不可辨识）vs 命题 2（加外部观测可辨识） | 可辨识 ≠ 可求解 ⇒ 必须随机偏差训练 | — |
| **6** | Framework overview ★ | 2×2 矩阵：task/backup × sim/real + Supervisor | 全场最重要的图，后面回指 | — |
| **7** | Shared method layer | 观测通道 + 偏差训练 + backbone | sim/real 共用观测 / 训练 / 网络 | — |
| **8** | Pipelines | 上下两条 pipeline 对比 HIL-SERL vs BC+HG-DAgger | 范式三轴决策 | — |
| **9** | Backup + Supervisor FSM | 左 backup 训练 / 右 三态 FSM + HOMING | backup 通过 HOMING 与 task 解耦 | — |
| **10** | H1/H2/H3 sim results | 三栏曲线 | H1 收敛 / H2 崩溃 / H3 恢复 | H1, H2, H3 各 1 段（5-8 s）|
| **11** | HIL-SERL negative result | Q 表面平坦 + actor 漂移曲线 | N=1 + 10 Hz + 稀疏奖励 → actor 解冻后漂移 | actor 漂移 1 段（8-12 s）|
| **12** | Real setup + 3 tasks | Franka + 三任务 + B+D 注入点 | 平台 + 任务 + 偏差注入 + N=10 探索性 | pickup / wipe / p&p 各 5 s |
| **13** | ★ 4-col matrix | 双层表头 + 6 视频缩略图 + 4 条观察 | 必要 / 充分 / 无副作用 / 残余 gap | col 2 崩溃 vs col 4 鲁棒，每任务 2 段 = 6 段 |
| **14** | HG-DAgger + Backup | 左 干预率曲线 / 右 backup 事件统计 | iter 0/1/2 收敛 + backup 走事件级 | 干预 1 + backup 触发 1（8-10 s）|
| **15** | Conclusion | 三贡献回顾（I 理论 / II 方法 / III 实证）+ 4 条 limitations | 闭环讲完贡献 + 诚实陈述边界 | — |
| **16** | Future Work + Q&A | 4 段 future（两步消融最直接）+ Thanks + Q&A | 下一步明确 | — |

## 2. 必保高光页（评委会记住的 3 张）

1. **S6 框架总图** — task⊕backup × sim/real
2. **S13 4 列矩阵** — 训练 × 测试 bias factorial
3. **S11 HIL-SERL negative result** — 方法论级 contribution

## 3. demo 素材清单

15 段，全有（用户已确认）。命名规范：`s{slide}_{scene}.mp4`

| Slide | 文件名 | 时长 |
|---|---|---|
| S10 | `s10_h1_baseline.mp4` / `s10_h2_bias_collapse.mp4` / `s10_h3_external_recover.mp4` | 5–8 s |
| S11 | `s11_hilserl_actor_drift.mp4` | 8–12 s |
| S12 | `s12_pickup_normal.mp4` / `s12_wipe_normal.mp4` / `s12_pap_normal.mp4` | 5 s |
| S13 | `s13_{task}_{col2_collapse|col4_robust}.mp4` × 6 | 5–8 s |
| S14 | `s14_hgdagger_intervene.mp4` / `s14_backup_trigger_homing.mp4` | 8–10 s |

## 4. Mockup 阶段（当前）

已交付 `cn/mockup-deck/index.html`，覆盖：
- **S1 封面**：论文标题 + 副标题（中英双语）+ 元数据（验证 cover layout 质感）
- **S6 框架图**：2×2 frame-grid + supervisor 横条 + 三轴注脚（验证自定义 layout 是否能融入 signal 设计语言）
- **S13 矩阵页**：multirow 表头 + 6 视频占位缩略图 + 4 观察 bullet（验证最高信息密度页）

待用户验收：
- 模板质感是否撑得住答辩
- 自定义 layout（frame-grid, matrix-table, demo-thumb）是否与 signal 设计语言一致
- 视频缩略图的占位是否需要调整布局

## 5. 下一步（用户验收后）

- 路径 A（继续 HTML）：填 S2–S5 / S7–S12 / S14–S16 共 13 张
- 路径 B（折中）：HTML 封面/章节过渡 + Beamer 主体（如果嫌 HTML 排版重型图表麻烦）
- 路径 C（换模板）：mockup 不满意则换 monochrome / cartesian / vellum 重做
