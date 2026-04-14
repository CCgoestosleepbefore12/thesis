# 第 6 章实验计划

本文档列出 Chapter 6 需要补做的全部实验、配置与预期结果形式。实验数据在补做后填回本文件对应的"结果"空位，然后再转写到 `content/ch6_experiments.tex`。

**最后更新**：2026-04-14（初版，基于 Ch.3-5 写完后的梳理）

---

## 实验总览

| # | 实验 | 优先级 | 验证的 claim | 训练任务数 | 预估时间 |
|---|---|---|---|---|---|
| 1 | 观测空间消融（31D vs 28D） | 🔴 必做 | Ch.3 Prop 1 vs Prop 2 | 6 | ~6-12 h |
| 2 | 固定 vs 随机偏差训练 | 🔴 必做 | Ch.4.3 核心 | 9 | ~9-18 h |
| 3 | Backup S1 主存活率 | 🔴 必做 | Ch.5 有效性 | 3 | ~3-6 h |
| 4 | Reward V1/V2/V3 演化 | 🔴 必做 | Ch.5.4 reward hacking | 9 | ~9-18 h |
| 5 | Backup 偏差鲁棒性 | 🟡 应做 | Ch.5.2(iv) | 0 | ~30 min |
| 6 | DR on/off 消融 | 🟡 应做 | Ch.5.5 DR 必要性 | 3 | ~3-6 h |
| 7 | Backup S2 场景扩展 | 🟡 应做 | Ch.5.3 扩展性 | 3 | ~3-6 h |
| 8 | 残差 RL 基线对照 | 🟢 可选 | Ch.4 必要性 | 3 + 代码 | ~1-2 d |
| 9 | 集成系统 end-to-end | 🟢 可选 | 联合架构 | 0 + 脚本 | ~0.5-1 d |

**总计**：必做 4 个 / 应做 7 个 / 全做 9 个

**Seeds**：每组 **3 seeds**（最小可接受），成本允许时提到 5

---

## Exp 1：观测空间消融（31D vs 28D）🔴

### 目的
实证 Ch.3 命题 1 与命题 2 的对应——剔除独立外部 TCP $\vect{o}_\text{real}$ 后，偏差不可辨识；带 $\vect{o}_\text{real}$ 时可辨识。

### 配置
| 项 | 值 |
|---|---|
| 环境 | `gym_frrl/PandaPickPlaceSafeBiasJ1RandomKeyboard-v0`（随机偏差） |
| 观测 A（完整） | 31D = robot(18) + block(3) + plate(3) + real_tcp(3) + hand_active(1) + hand_pos(3) |
| 观测 B（消融） | 28D = robot(18) + block(3) + plate(3) + hand_active(1) + hand_pos(3)（剔除 real_tcp） |
| 训练算法 | HIL-SERL（RLPD + HIL） |
| 偏差分布 | $b_i \sim \mathcal{U}[0, 0.25]$ rad，per-joint |
| 训练步数 | 每 seed 100k online steps |
| Seeds | 3 |
| 总训练任务 | 2 × 3 = 6 |

### 评估
- 偏差扫描 $\vect{b}_\text{eval} \in \{0.0, 0.05, 0.10, 0.15, 0.20, 0.25\}$ rad，全关节同值
- 每个偏差值跑 **50 episodes**
- 指标：任务成功率（block 到 plate 的 success rate）

### 结果形式
**图 6.1**：偏差-成功率曲线（两条，31D vs 28D），横轴偏差幅值，纵轴成功率。
每条曲线标注 3 seeds 的 mean ± std（阴影带）。

### 预期
- 31D：全范围 80-100% 稳定
- 28D：偏差增大时显著退化，最差点接近随机水平

### 数据待填
```
| 偏差 (rad) | 31D 成功率 (mean ± std) | 28D 成功率 (mean ± std) |
|-----------|------------------------|------------------------|
| 0.00      |                        |                        |
| 0.05      |                        |                        |
| 0.10      |                        |                        |
| 0.15      |                        |                        |
| 0.20      |                        |                        |
| 0.25      |                        |                        |
```

---

## Exp 2：固定偏差 vs 随机偏差训练 🔴

### 目的
暴露"单点条件化 → 零偏差过补偿"（Ch.4.3 核心），实证 Ch.3.4.3 的定量论断。

### 配置
| 项 | 值 |
|---|---|
| 环境 | `PandaPickPlaceSafeKeyboard-v0`（基座）+ 不同偏差配置 |
| 观测 | 31D（Exp 1 胜出配置） |
| 训练算法 | HIL-SERL |
| 训练步数 | 100k online steps |
| Seeds | 3 |

三组配置：
1. **No-bias**：$\vect{b} = \vect{0}$，训练环境：`PandaPickPlaceSafeKeyboard-v0`
2. **Fixed bias**：$b_i = 0.2$ rad（所有关节），训练环境需要新注册（参照 `PandaPickPlaceSafeBiasJ1Random-v0` 改 `bias_mode="fixed", fixed_bias_value=0.2, target_joints=None`）
3. **Random bias**：$b_i \sim \mathcal{U}[0, 0.25]$ rad，训练环境：`PandaPickPlaceSafeBiasJ1RandomKeyboard-v0` 或新注册的 AllJoints 随机版

**总训练任务**：3 × 3 = 9

### 评估
- 偏差扫描 $\{0.0, 0.05, 0.10, 0.15, 0.20, 0.25\}$ rad
- 每个偏差值 50 episodes
- 指标：任务成功率

### 结果形式
**图 6.2**：三条偏差-成功率曲线 overlaid，横轴偏差幅值，纵轴成功率。

### 预期（验证过补偿现象）
- **No-bias**：$b = 0$ 高、$b$ 增大退化
- **Fixed @ 0.2**：$b = 0.2$ 附近高、**$b = 0$ 明显掉**（过补偿的直接实证）
- **Random**：全范围鲁棒

### 数据待填
```
| 偏差 (rad) | No-bias | Fixed (b=0.2) | Random [0, 0.25] |
|-----------|---------|---------------|------------------|
| 0.00      |         |               |                  |
| 0.05      |         |               |                  |
| 0.10      |         |               |                  |
| 0.15      |         |               |                  |
| 0.20      |         |               |                  |
| 0.25      |         |               |                  |
```

**关键数字**（会回填到 Ch.3.4.3 与 Ch.4.3）：Fixed @ 0.2 训练的策略在 $b = 0$ 下的成功率。

---

## Exp 3：Backup S1 主存活率 🔴

### 目的
报告 Ch.5 备份策略在主场景 S1 上的性能（这是 Ch.5 的主结果）。

### 配置
| 项 | 值 |
|---|---|
| 环境 | `gym_frrl/PandaBackupPolicyS1-v0` |
| 观测 | 84D（28 × 3 帧堆叠） |
| 算法 | 纯在线 SAC |
| Reward | V3（非负即时 + 大终止惩罚） |
| UTD | 8 |
| $\gamma$ | 1.0 |
| 训练步数 | 200k |
| Seeds | 3 |
| 总训练任务 | 3 |

### 评估
- 200 episodes × 每 episode 200 步连续存活检查
- 按 4 种运动模式分项统计（LINEAR / ARC / STOP_GO / PASSING）

### 结果形式
**表 6.1**：Backup S1 各运动模式存活率

### 数据待填
```
| 运动模式 | 存活率 (mean ± std) |
|---------|--------------------|
| LINEAR  |                    |
| ARC     |                    |
| STOP_GO |                    |
| PASSING |                    |
| **总体** |                    |
| 碰撞率   |                    |
| 位移超限率 |                  |
```

---

## Exp 4：Reward V1/V2/V3 演化实证 🔴

### 目的
实证 Ch.5.4 的 reward hacking 分析——V1 "跑远"、V2 "快死捷径"、V3 "稳定收敛"。

### 配置
三组训练：
1. **V1**：接近度奖励（proximity reward）
2. **V2**：全负奖励（位移 + 动作 + 平滑）
3. **V3**：非负即时 + 大终止惩罚（当前最终版）

共享配置：
| 项 | 值 |
|---|---|
| 环境 | `PandaBackupPolicyS1-v0`（同 Exp 3） |
| 算法 | SAC, UTD=8 |
| 训练步数 | 200k |
| Seeds | 3 |
| 总训练任务 | 3 × 3 = 9 |

### 记录的训练诊断
- **V1**：平均 TCP 累计位移（应该逼近 15 cm 位移上限）、平均存活步数
- **V2**：平均 episode 奖励（应该从 $\sim -0.88$ 持续恶化到 $\sim -1.00$）、平均 episode 长度（应该逐渐缩短）
- **V3**：平均 episode 奖励（应该稳定收敛到 $\sim +9$ 到 $+10$）、存活率（应该上升到 $\sim 99\%$）

### 结果形式
**图 6.3**：三条训练曲线对比（可用 subplot）
- 子图 A：平均 episode 奖励 vs 训练步数
- 子图 B：平均 episode 长度 / 存活步数 vs 训练步数
- 子图 C：（若有）平均 TCP 位移 vs 训练步数（主要展示 V1 跑远）

### 数据待填
```
V1 最终指标（200k 步时）：
- 平均 episode 位移：___ cm（预期逼近 15 cm 上限）
- 平均存活步数：___

V2 最终指标：
- 平均 episode 奖励：___（预期约 -1.00）
- 平均 episode 长度：___（预期显著小于 10 步）

V3 最终指标：
- 平均 episode 奖励：___（预期约 +9 到 +10）
- 最终存活率：___%（预期约 99%）
```

---

## Exp 5：Backup 偏差鲁棒性 🟡

### 目的
实证 Ch.5.2(iv)——backup 对 encoder bias 天然鲁棒、无需 bias-aware 训练。

### 配置
- 用 **Exp 3 的 S1 no-bias checkpoint**（不需要重训）
- 在两种评估环境分别跑：
  1. `PandaBackupPolicyS1-v0`（无偏差）
  2. `PandaBackupPolicyS1BiasJ1-v0`（joint 1 随机偏差 $b_1 \sim \mathcal{U}[0, 0.25]$ rad）
- 200 episodes × 每种 env × 3 seeds

### 总训练任务
**0**（只评估）

### 结果形式
**表 6.2**：存活率对比

### 数据待填
```
| 评估环境                 | 存活率 (mean ± std) |
|------------------------|--------------------|
| No-bias                |                    |
| Joint 1 random bias    |                    |
| Δ（两者差）              |                    |
```

**预期**：$\Delta \leq 2\%$（基本一致）

---

## Exp 6：DR on/off 消融 🟡

### 目的
实证 Ch.5.5 的 DR 必要性——关闭 DR 训练后，策略在带感知噪声的 env 上会严重退化。

### 配置
额外训练一组：
- **No-DR**：`PandaBackupPolicyS1NoDR-v0`（已注册，DR 关闭）
- 其他同 Exp 3：200k 步、UTD=8、V3 reward、3 seeds

### 评估矩阵
| 训练配置 | 评估环境 | 备注 |
|---|---|---|
| DR-on（Exp 3 的 checkpoint） | DR-on env（S1-v0） | training ≈ eval 分布 |
| DR-on | DR-off env（S1NoDR-v0） | 干净评估，上界 |
| DR-off | DR-on env | 噪声下测试（关键） |
| DR-off | DR-off env | 干净评估 |

### 总训练任务
**3**（只多训 DR-off 一组）

### 结果形式
**表 6.3**：$2 \times 2$ 存活率矩阵

### 数据待填
```
| 训练 ↓ / 评估 → | DR-on | DR-off |
|----------------|-------|--------|
| DR-on          |       |        |
| DR-off         |       |        |
```

**预期**：DR-off 训练 + DR-on 评估 应显著退化；DR-on 训练在两种评估下都稳定。

---

## Exp 7：Backup S2 场景扩展 🟡

### 目的
证明 Ch.5 架构可扩展到更复杂的 S2 场景（移动 + 静止障碍）。

### 配置
| 项 | 值 |
|---|---|
| 环境 | `gym_frrl/PandaBackupPolicyS2-v0` |
| 观测 | 114D（38 × 3 帧堆叠） |
| UTD | **4**（S2 代码里是 4，非 S1 的 8） |
| 其他 | 同 Exp 3 |
| Seeds | 3 |
| 总训练任务 | 3 |

### 评估
- 200 episodes × 4 运动模式分项

### 结果形式
**表 6.1 扩展**：加一列 S2

### 数据待填
```
| 运动模式 | S1 存活率 | S2 存活率 |
|---------|---------|---------|
| LINEAR  |         |         |
| ARC     |         |         |
| STOP_GO |         |         |
| PASSING |         |         |
| **总体** |         |         |
```

---

## Exp 8：残差 RL 基线对照 🟢

### 目的
可选对照——证明 HIL-SERL 在稀疏奖励 + 偏差的任务下优于 Johannink 2019 的残差 RL。

### 配置
- Base controller：手写线性趋近（TCP → block → plate）
- Residual：SAC 学的修正项
- 训练环境同 Exp 2 的 Random bias 组
- 3 seeds × 100k 步

### 评估
- 偏差扫描 $\{0.0, 0.05, 0.10, 0.15, 0.20, 0.25\}$
- 50 episodes per bias

### 总训练任务
**3 + 写代码时间**

### 判断
- 若时间紧张：**跳过**，Ch.2 相关工作的"残差 RL 对比"改成理论分析或未来工作
- 若有时间：完整做，预期结果是残差 RL 在稀疏奖励下不收敛或大幅不及 HIL-SERL

### 数据待填
```
| 偏差 (rad) | 残差 RL 成功率 | HIL-SERL 成功率（from Exp 2 Random） |
|-----------|---------------|-----------------------------------|
| 0.00      |               |                                   |
| ...       |               |                                   |
```

---

## Exp 9：集成系统 end-to-end 🟢

### 目的
证明任务策略 + 备份策略的联合部署在带 hand 干扰的 pick-and-place 上能完成任务。

### 配置
- 加载 Exp 2 的 **Random bias 任务策略** checkpoint
- 加载 Exp 3 的 **Backup S1** checkpoint
- 运行时切换脚本：hand_dist < 8 cm → backup 接管 ≤ 10 步 → 切回 task
- 环境：`PandaPickPlaceSafeKeyboard-v0`（有 hand 间歇出现）
- 评估：100 完整 episodes × 3 seeds

### 对照组
- (a) 仅任务策略（无 backup）：碰撞率预期高
- (b) 任务策略 + backup（本文方案）
- （可选 c）任务策略 + 硬编码冻结：碰撞率应比 (a) 好、比 (b) 差

### 总训练任务
**0**（只评估 + 写切换脚本）

### 结果形式
**表 6.4**：集成系统评估

### 数据待填
```
| 配置                    | 任务完成率 | 人机碰撞率 | 切换次数/episode |
|-----------------------|----------|----------|----------------|
| 仅任务策略              |          | 高        | —              |
| 任务 + 硬编码冻结       |          | 中        | —              |
| 任务 + Backup（本文）   |          | 低        |                |
```

---

## 后续写作流程

1. **按此表跑实验**，把数据填进各个"数据待填"空位
2. **更新此文件**，保留为实验日志
3. **转写到 `content/ch6_experiments.tex`**：每个 Exp 对应一个小节，数据进表/图
4. **回填 Ch.3 / Ch.4 / Ch.5 里之前标记 "待 Ch.6 给出具体数值" 的位置**：
   - Ch.3.4.3 "单点条件化 $\to$ 零偏差过补偿"的具体数字（来自 Exp 2）
   - Ch.4.1 / Ch.4.3 任何提到"实验预告"的成功率数字（来自 Exp 1, 2）
   - Ch.5.2 Backup 偏差鲁棒性的"基本一致"具体数字（来自 Exp 5）
   - Ch.5.4 V2 训练日志 "-0.88 → -1.00"（来自 Exp 4，若有完整曲线则用曲线替代）

---

## 待确认的问题

- [ ] Exp 1 消融环境：需要新注册 28D 观测变体，还是直接改 env 代码？
- [ ] Exp 2 Fixed bias 环境：需要新注册 "AllJoints Fixed 0.2" 变体（目前只有 J4 Fixed 0.3 和 Random 变体）
- [ ] Exp 4 V1 / V2 的 env 与 reward：需要恢复历史 reward 函数（或保留分支）
- [ ] Exp 8 残差 RL 是否跑，决定后要同步调整 spec.md 和 Ch.2 计划
- [ ] Exp 9 切换脚本是否已存在？若无需要先写
