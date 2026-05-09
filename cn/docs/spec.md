# 论文规格说明书 (Thesis Spec)

## 基本信息

- **标题（中）**: 基于分层模仿与强化学习的故障鲁棒机器人操作研究
- **标题（英）**: Hierarchical Imitation and Reinforcement Learning for Fault-Tolerant Manipulation
- **作者**: 程程 (Cheng Cheng)
- **学校**: Universität Stuttgart
- **类型**: Master Thesis
- **语言**: 先写中文版打磨，后翻译为英文/德文提交
- **页数目标**: 50–60 页
- **LaTeX 编译**: 中文版 `xelatex`，英文版 `pdflatex`

---

## 核心叙事线

> 编码器校准偏差把机器人操作问题从 MDP 退化为 POMDP
> → **理论核心（Ch.3）**：POMDP 形式化 + 可观测性命题（基础观测不可辨识 / 增强观测唯一恢复）
> → **方法设计（Ch.4）**：观测增强 + 随机偏差训练 + 共享网络架构（DINOv3-S + 三个网络头）
> → **仿真验证（Ch.5）**：HIL-SERL 训练任务策略（H1–H3 闭环）+ SAC + DR 训练备份策略 + V1→V3b reward 演化
> → **真机部署（Ch.6）**：BC + HG-DAgger 任务策略 + WiLoR 备份感知 + **三任务联合 fault injection 4 列成功率矩阵 = 论文灵魂数据**

**论文按 sim/real 二分章节**（Ch.5 仿真章 + Ch.6 真机章），与新标题"Hierarchical IL+RL"完美对位：
- **Hierarchical** = task ⊕ backup 双策略分层
- **Imitation** = real BC + HG-DAgger（Ch.6 主线）
- **Reinforcement** = sim HIL-SERL + sim backup SAC（Ch.5 主线）
- **Fault-Tolerant** = 编码器偏差容错 + 人机安全

---

## 两个子系统的训练范式对比（核心架构决策）

这是论文的一个独立贡献点——**同一机器人系统对不同子任务、不同部署阶段选用不同的范式**：

| 维度 | 任务策略 sim（§5.2） | 任务策略 real（§6.2） | 备份策略 sim（§5.3） | 备份策略 real（§6.3） |
|---|---|---|---|---|
| **观测模态** | 25D 状态 + 双相机 RGB | 14D 状态 + 双相机 RGB | 纯状态 (28D × 3) | 同（部署不重训）|
| **训练算法** | HIL-SERL (RLPD + SAC + HIL) | BC pretrain + HG-DAgger | 纯在线 SAC + DR | — |
| **训练场景** | 仿真 | 真机 | 纯仿真 | — |
| **核心瓶颈** | — | 视觉 sim2real + 小数据 sparse reward | 低维状态噪声 / 延迟 | 手部检测 self-detection |
| **范式选择理由** | 验证可观测性命题 + 给真机方案提供"成功信号路径"凭据 | HIL-SERL 在 50 demo + sparse reward 真机上 actor 解冻后大幅漂移（论文级 negative result）→ 改 BC + 介入修正 | DR 覆盖低维状态 sim2real，递归安全（避免训 backup 时再需要"backup 的 backup"）| WiLoR YOLO + 双参考点几何过滤 |

**范式选择由观测模态、数据规模、安全约束三者共同决定**——这是论文方法论层的独立贡献。

---

## 章节结构（方案 B：sim/real 二分）

### 第 1 章 绪论（4–5 页）

```
1.1 研究背景与动机
    - 工业机器人编码器故障的 5 种来源
    - 现有方案（停机标定、FDI/FTC）的不足
    - 在线适应（IL/RL）的动机

1.2 问题陈述
    - 编码器偏差使 MDP 退化为 POMDP
    - HIL 训练引入人机安全问题
    - 视觉 sim-to-real gap 与小数据真机场景的算法选型约束

1.3 论文贡献（三点）
    ① POMDP 形式化 + 可观测性分析（理论贡献，Ch.3）
    ② 分层模仿与强化学习的容错操作框架：
       - 共享方法层（Ch.4 观测设计 + 随机偏差训练 + 网络架构）
       - sim 端 HIL-SERL 验证（Ch.5）
       - real 端 BC + HG-DAgger 部署（Ch.6）
       - task ⊕ backup 双策略分层
    ③ 真机三任务在编码器偏差注入下的端到端验证
       （含 HIL-SERL 在小数据真机失败的 negative result）

1.4 论文组织结构
```

### 第 2 章 相关工作（6–8 页）

**写作原则**：Ch.2 = 领域景观 + 对比替代方案 + 定位本文选择；Ch.4–6 = 本文具体配置。

```
2.1 机器人容错控制（FDI/FTC）
    - Blanke 2016 FDI/FTC 教材；传统故障模型的局限

2.2 POMDP 求解方法
    - 信念状态方法 (Kaelbling 1998)
    - RNN/GRU 隐式记忆 (Hausknecht 2015 DRQN)
    - 仅作为文献背景，不作对比论断

2.3 机器人强化学习与残差结构
    - 残差 RL (Johannink 2019) 作为 baseline
    - 其他结构化 RL 方法

2.4 鲁棒 RL 与域随机化
    - Tobin 2017 视觉 DR；视觉 sim2real gap 的既定难题
    - 状态空间 DR 的成功案例（用作 §5.3 backup 范式的支撑）

2.5 模仿学习与人在回路（IL 在前 RL 在后，呼应新标题）
    - BC / DAgger (Ross 2011) / HG-DAgger (Kelly 2019, 本文真机采用)
    - HIL-SERL (Luo 2025, Science Robotics) —— 本文 sim 端采用
    - RLPD (Ball 2023) —— HIL-SERL 的底层算法
    - 对比维度：纯在线 SAC / pure BC / DDPGfD / vanilla DAgger / HG-DAgger / HIL-SERL
    - 定位本文双端选择：sim HIL-SERL 验证理论 + real BC + HG-DAgger 部署

2.6 安全强化学习与避障 + Reward hacking 背景
    - Kiemel 2024 非负即时奖励
    - Reward hacking / reward misspecification 概念背景（为 §5.3.3 服务）
```

### 第 3 章 编码器偏差建模与 POMDP 分析（8–10 页）⭐ 理论核心

**完全不动**——理论核心，是论文最硬贡献。已写完 3.1–3.4.2 + 3.5；Fig.3.3 已加。

```
3.1 工业机器人编码器校准偏差
3.2 因果链：一源三效应
3.3 POMDP 形式化
3.4 可观测性分析
    3.4.1 命题 1：基础观测下的不可辨识性
    3.4.2 命题 2：观测增强恢复可辨识性
    3.4.3 可辨识性 ≠ 可求解性
    + Fig.3.3 几何示意（已加）
3.5 仿真实现与等价性分析
```

### 第 4 章 任务策略：方法设计与网络架构（6–8 页）⭐ 减薄

**仅含方法层与共享网络架构**，sim/real 算法实现搬到 Ch.5/Ch.6。

```
4.1 方法设计动机
    - 三个设计选择：观测增强 + 随机偏差训练 + sim/real 双端范式
    - 设计选择三压缩为承启级别（详细 sim 实现见 §5.2，real 实现见 §6.2）

4.2 仿真观测空间设计（25D）
    - 25D = robot(18) + real_tcp(3) + hand_active(1) + hand_pos(3)
    - 透明脚注：实验沿用 31D 默认 env，剔除常量信道后等价于 25D
    - 信息通路 Fig.4.1（保留）+ 25D 分辨识相关 21D / 运行时安全 4D 的细分

4.3 随机偏差训练
    - b ~ U[0, b_max] 作为隐变量 DR
    - 与 Tobin 视觉 DR 的本质区别（必须 condition vs 冗余变量）

4.4 网络架构与训练目标 ⭐ 共享 sim/real
    - 共享 encoder（DINOv3-S frozen ViT-S/16, 22M frozen）
    - SpatialLearnedEmbeddings 384×8×8 → 3072 → 256
    - 三个网络头：连续 Actor / Twin Critic / 离散 Gripper Critic
    - SAC 损失（用于 sim HIL-SERL）+ BC NLL 损失（用于 real BC pretrain）
    - 训练超参表 sim/real 双列对照（Tab.4.3）
    - Fig.4.3 网络架构图（共享，sim 用全部头 / real BC 仅训 actor）

4.5 稀疏奖励设计
    - sim：r=1 iff (block-plate distance<5cm) AND (height_gain>20cm)
    - real：keyboard reward 协议（S/Enter/Space/Backspace）
    - 稀疏奖励可行性论证（探索瓶颈 vs Q estimation 准确性两个独立瓶颈）
```

### 第 5 章 仿真：策略训练与可观测性验证（12–15 页）⭐ 重组

```
5.1 章节引言：仿真侧范式与本章组织
    - sim/real 分工总览
    - 仿真侧两条主线：task policy HIL-SERL + backup policy SAC+DR
    - sim 实验组织：H1-H3 task / Backup S1/V3b / Reward V1/V2/V3 演化

5.2 任务策略仿真训练（HIL-SERL）⭐ 搬自旧 §4.4
    5.2.1 HIL-SERL 训练框架
    5.2.2 算法栈：SAC + RLPD + HIL（高 UTD + LayerNorm 设计）
    5.2.3 人在回路干预（DAgger 风格 vs query-by-policy 区别）
    5.2.4 分布式 Actor-Learner 架构（Fig.5.1 = 旧 fig:hilserl_arch）
    5.2.5 仿真训练流程（Demo warmup + 在线混合采样）

5.3 备份策略仿真训练（SAC + DR）⭐ 搬自旧 Ch.5
    5.3.1 问题定义与场景设计（部署场景 + 安全需求）
    5.3.2 训练范式选择（为何不用 HIL：观测低维 + DR 覆盖 + 递归安全）
    5.3.3 场景 S1/S2 与观测向量（28D × 3 帧 = 84D）
    5.3.4 障碍物运动模式（LINEAR / ARC / STOP_GO / PASSING）
    5.3.5 奖励函数设计演化（Reward Hacking 案例研究 V1→V2→V3）
    5.3.6 V3b 几何升级（单球 → 5 球全臂 + obstacle r=0.10 + saturating proximity）⭐ 新加
    5.3.7 面向 Sim-to-Real 的域随机化
    5.3.8 Sim-to-Real 对齐分析

5.4 仿真实验与分析 ⭐ 搬自旧 Ch.6 §6.A
    5.4.1 实验设置（评估协议、seeds、统计口径）
    5.4.2 H1：编码器偏差退化验证（PickCube env）
    5.4.3 H2 + H3：固定 vs 随机偏差训练（三策略对比）
    5.4.4 观测空间消融（PickCube 18 / 21 / 24D）
    5.4.5 Backup S1 V3b 主存活率
    5.4.6 Backup 偏差鲁棒性 zero-shot
    5.4.7 Reward V1 / V2 / V3 演化训练曲线
    5.4.8 Backup S2 多障碍场景扩展

5.5 仿真到真机的桥接
    - 仿真验证的 H1-H3 给真机提供"成功信号路径"凭据
    - V3b 几何升级是真机部署观察到的失效模式倒逼仿真升级的案例
    - 真机部分核心数据见 §6.4 三任务联合成功率矩阵
```

### 第 6 章 真机：模仿部署与端到端验证（14–18 页）⭐ 重组（含原 Ch.7 内容）

```
6.1 真机系统架构与编码器偏差注入 ⭐ 搬自旧 §7.1-7.3
    6.1.1 双机硬件架构（RT PC + GPU 工作站 + 网络分离）
    6.1.2 B+D 双注入点（C++ 阻抗控制器 + Flask franka_server）
    6.1.3 OSC 与笛卡尔阻抗控制的等价性
    Fig.6.1 = 系统架构图
    Fig.6.2 = B+D 双注入点数据流

6.2 任务策略真机部署 ⭐ 搬自旧 §4.5
    6.2.1 真机 14 维观测
        - 14D = tcp_true(7) + tcp_vel(6) + gripper(1)
        - Fig.6.3 = 旧 fig:obs_real_14d sim 25D vs real 14D 通路对比
        - Tab 14D 维度构成
    6.2.2 BC pretrain
        - L_BC = NLL on tanh-Gaussian + λ·CE_discrete
        - 三个工程修正（encoder_is_shared / σ clamp / DrQ random_shift）
    6.2.3 HG-DAgger 介入迭代
        - 介入状态机（Fig.6.4 = 旧 fig:dagger_state_machine）
        - 迭代协议（Fig.6.5 = 旧 fig:dagger_iteration）
    6.2.4 三任务部署：共享管线与任务级差异
        - pickup / wipe / pickandplace
        - 夹爪锁定模式 + workspace ROI + 联合 backup
    6.2.5 HIL-SERL 真机失败的方法论复盘 ⭐ negative result
        - 主要现象 + 多轮修复尝试 + 主要根因假设 + 替代解释 + 范围限定

6.3 备份策略真机部署
    6.3.1 手部检测选型 MediaPipe → WiLoR ⭐ 重写
        - MediaPipe 初选（CPU 实时、轻量）
        - 部署阶段发现 self-detection（WiLoR 把 Franka gripper 误识别为人手）
        - 切换到 WiLoR YOLO + 双参考点几何过滤
        - flange_radius=0.10 + tcp_radius=0.06 球过滤
    6.3.2 Hierarchical Supervisor + HomingController
        - 三态 FSM（TASK / BACKUP / HOMING）
        - HomingController 6D clipped-P 控制器
    6.3.3 sim2real Route A center-dist 几何对齐
        - 阈值 D_SAFE 0.40m / D_CLEAR 0.45m
        - center-to-center 几何（替代历史 Minkowski 虚点方案）
        - TCP_OFFSET 反推 flange 几何

6.4 真机实验与分析
    6.4.1 三任务联合成功率矩阵 ⭐ 论文核心数据
        4 列 × 3 行：
        | 任务 \ 配置 | 无 bias | 有 bias | + backup 无 bias | + backup 有 bias |
        | pickup |  |  |  |  |
        | wipe |  |  |  |  |
        | pickandplace |  |  |  |  |
    6.4.2 HG-DAgger 介入率下降曲线
    6.4.3 编码器偏差注入端到端验证
        - bias monitor 时序图：q_true vs q_biased 物理扫动
    6.4.4 Backup 真机避让评估
        - WiLoR 检出率 / FSM 触发率 / 避让收敛步数
    6.4.5 HIL-SERL 真机 negative result 复盘
        - 多次实验观察到的 actor 漂移现象
        - 切换到 BC + HG-DAgger 后的对比效果

6.5 操作员安全约定与失效模式
    - 人手不能伸入 flange 10cm 球（self-detection 盲区）
    - 阻抗收尾 ±2-5mm 抖动 + done_consecutive_n=3 streak
    - 真机 deploy 不做 episode 终止，靠 supervisor 切换

6.6 实测精度与失效复盘
    - cam-to-robot 标定精度 ~15-20mm（SVD 主求解）
    - 真机训练 round 4 / round 5 的 P0 修复总览
```

### ~~第 7 章 真机部署与验证~~（删除，内容合并到 Ch.6）

### 第 8 章 总结与展望（2–3 页）

```
8.1 工作总结
    - POMDP 形式化与可观测性分析（Ch.3）
    - 共享方法层（Ch.4 观测设计 + 网络架构）
    - 双端范式：sim HIL-SERL 验证（Ch.5）+ real BC + HG-DAgger 部署（Ch.6）
    - 分层 task ⊕ backup 在故障注入下的端到端验证
    - HIL-SERL 真机失败的方法论级 negative result

8.2 局限性
    - 真机当前用 tcp_true privileged 通道，bias 鲁棒性闭环未在真机验证
    - environment_state 6D 当前为 placeholder（物体检测未接入）
    - 单关节 / 固定类型偏差
    - 三任务规模 + 单一硬件平台

8.3 未来工作
    - **真机 bias 鲁棒性两步消融**：
      (i) 真机 14D 的 tcp_true → tcp_biased 应失败
      (ii) 在 (i) 基础上补 q+b → 应恢复成功（与 sim 25D 路径同构）
    - 显式偏差估计器（在线最小二乘 from Jacobian / RMA 风格 GRU）
    - 多关节 / 时变偏差
    - 多故障类型扩展
    - 物体检测接入 → environment_state 真值化
```

---

## 已确定的设计决策

| 决策 | 选择 | 原因 |
|---|---|---|
| 论文标题 | Hierarchical IL + RL for Fault-Tolerant Manipulation | 准确覆盖 sim RL + real IL 双端范式 + 分层 task⊕backup + 故障容错 |
| **章节组织** | **方案 B：按 sim/real 二分**（Ch.5 仿真章 / Ch.6 真机章 / Ch.7 删除合并） | 与新标题 IL+RL 二分对位；真机权重显著提升（Ch.6 14-18 页 vs Ch.5 12-15 页） |
| **Ch.4 定位** | 仅含方法层 + 共享网络架构 | sim/real 算法实现搬到 Ch.5/Ch.6，Ch.4 减薄到 6-8 页 |
| **共享网络图位置** | 保留在 Ch.4 §4.4 | sim 与 real 共享同一架构，放在共通方法章合理 |
| **过渡叙事** | 保留章节引言 + 桥接段（如 §5.5） | 让读者在 sim/real 切换时有方向感 |
| 理论框架 | 标准 POMDP（不引入 HiP-MDP） | 术语通用，审稿人熟悉 |
| 可观测性分析严格度 | 严格命题 + 构造性证明 + Fig.3.3 几何示意 | 硕士论文需要理论深度 |
| sim 任务策略观测 | 25D（描述层面，常量信道剔除等价性脚注） | 删 plate/block 与真机对齐 |
| real 任务策略观测 | 14D = tcp_true 7 + tcp_vel 6 + gripper 1 | 对齐 hil-serl ram_insertion proprio 子集 |
| sim/real 拆分 | sim HIL-SERL（§5.2）+ real BC + HG-DAgger（§6.2）| sim 端 HIL-SERL 仿真稳定；真机 sparse + 小数据下 actor 漂移 |
| HIL-SERL 真机 negative result | §6.2.5 + §6.4.5 诚实写明 | 是论文方法论级贡献而非缺陷 |
| 手部检测选型 | MediaPipe → WiLoR + 双参考点过滤（§6.3.1）| 部署阶段 self-detection 问题倒逼切换 |
| Backup 几何 | V2 单球 → V3b 5 球全臂 + obstacle r=0.10 | sim V2 真机部署观察到肘/前臂仍会撞 |
| Backup FSM 阈值 | D_SAFE 0.30 → 0.40m, D_CLEAR 0.35 → 0.45m, center-to-center | Route A 与 sim 训练几何 1:1 对齐 |
| §6.4.1 真机表为论文核心数据 | 三任务 × {无bias / 有bias / + backup × 2} 4 列矩阵 | task ⊕ backup 在故障注入下联合成功 = 论文灵魂 |
| 不补做新 sim 实验 | 全部沿用历史已跑数据 | 实验路径档案见 `ch6_experiment_plan.md`（已废弃） |
| Reward hacking | Ch.2.6 加背景 + §5.3.5 用作分析框架 | V1/V2 是典型 reward hacking 案例 |
| 引用规范 | 每个引用必须有可核验的真实来源 | 已核实关键引用清单（见 git history） |
| 写作语言 | 先中文打磨 → 翻译英文/德文提交 | 母语写作效率高 |

---

## 关键图表清单

> 注：LaTeX 自动按文件出现顺序编号；下表的 Fig 编号是按新章节归属的预期值，重组后实际编号以编译为准。

### 必需的图

| 编号（预期）| 内容 | label | 所在章节 |
|---|---|---|---|
| Fig.3.1 | 理想情况下的控制与观测数据流 | fig:pipeline_ideal | §3.2 |
| Fig.3.2 | 编码器偏差因果链总图 | fig:causal_chain | §3.2.4 |
| **Fig.3.3** | **可辨识性几何示意（Prop 1 平移对称 vs Prop 2 唯一交点）** | **fig:identifiability** | **§3.4.2** |
| Fig.4.1 | 25 维观测空间的信息通路 | fig:obs_dataflow | §4.2 |
| Fig.4.3 | 共享网络架构（DINOv3-S + 三头）| fig:network_arch | §4.4 |
| Fig.5.1 | sim HIL-SERL Actor-Learner 分布式架构 | fig:hilserl_arch | §5.2 |
| Fig.5.2 | Backup Policy 部署场景示意（S1/S2） | fig:deployment_scene | §5.3 |
| Fig.5.3 | Reward V1→V2→V3 训练曲线对比 | （新加）| §5.4.7 |
| Fig.5.4 | H1 退化曲线（成功率 vs 偏差）| （新加）| §5.4.2 |
| Fig.5.5 | H2/H3 三策略对比（no-bias / fixed / random）| （新加）| §5.4.3 |
| Fig.6.1 | 真机系统架构图 | （新加）| §6.1.1 |
| Fig.6.2 | B+D 双注入点数据流 | （新加）| §6.1.2 |
| Fig.6.3 | sim 25D / real 14D 观测空间对比 | fig:obs_real_14d | §6.2.1 |
| Fig.6.4 | HG-DAgger 介入状态机 | fig:dagger_state_machine | §6.2.3 |
| Fig.6.5 | BC + HG-DAgger 迭代工作流 | fig:dagger_iteration | §6.2.3 |
| Fig.6.6 | WiLoR 双参考点 self-detection 过滤几何 | （新加）| §6.3.1 |
| Fig.6.7 | 三任务 4 列成功率柱状图 ⭐ | （新加，等数据）| §6.4.1 |
| Fig.6.8 | HG-DAgger 介入率下降曲线 | （新加，等数据）| §6.4.2 |
| Fig.6.9 | HIL-SERL 真机 actor 漂移时序 | （新加，等数据）| §6.4.5 |

### 必需的表

| 编号（预期）| 内容 | label | 所在章节 |
|---|---|---|---|
| Tab.3.1 | 编码器偏差来源分类 | — | §3.1 |
| Tab.3.2 | 仿真-真实差距 8 项对比 | — | §3.5 |
| Tab.4.1 | sim 25D 观测空间各维度说明 | tab:obs_25d | §4.2 |
| Tab.4.2 | 设计选择映射 | tab:method_mapping | §4.1 |
| Tab.4.3 | 任务策略训练超参（sim HIL-SERL / real BC + HG-DAgger 双列） | tab:hyperparams | §4.4 |
| Tab.5.1 | 障碍物运动模式与参数 | — | §5.3.4 |
| Tab.5.2 | DR 参数表 | — | §5.3.7 |
| Tab.5.3 | H1 退化数据 | — | §5.4.2 |
| Tab.5.4 | H2/H3 三策略对比表 | — | §5.4.3 |
| Tab.5.5 | 观测消融 vs 理论预测对应表 | — | §5.4.4 |
| Tab.5.6 | Backup S1 V3b 各运动模式存活率 | — | §5.4.5 |
| Tab.6.1 | 真机 14D 观测构成 | tab:obs_real_14d | §6.2.1 |
| Tab.6.2 | 手部检测选型对比（MediaPipe vs WiLoR） | （新加）| §6.3.1 |
| **Tab.6.3** | **三任务 4 列成功率矩阵 ⭐ 论文核心数据** | （新加，等数据）| §6.4.1 |
| Tab.6.4 | HG-DAgger 迭代统计 | （新加，等数据）| §6.4.2 |

---

## 实验数据汇总

> 注：以下 sim 数据为历史 PickCube 24D env 实测；论文 §4.2 描述为 25D 部署观测（24D + hand 通道），消融在 PickCube 24D 上做以隔离编码器偏差因素。

### Task Policy 仿真实验（PickCube 24D）

**无偏差基线（18D）**：100% 成功率，13.4 ± 0.6 步

**H1 退化曲线**（PickCube 18D，无偏差训练）：

| Bias (rad) | 成功率 | 平均步数 | TCP 偏移 |
|---|---|---|---|
| 0.00 | 100% | 13.5 | 0 cm |
| 0.05 | 100% | 13.5 | ~3 cm |
| 0.10 | 80% | 55.9 | ~6.5 cm |
| 0.15 | 62% | 62.5 | ~10 cm |
| 0.20 | 51% | 99.8 | ~13 cm |
| 0.25 | 5% | 184.5 | ~16 cm |
| 0.30 | 0% | 191.1 | ~19 cm |

**H2 + H3 三策略对比**（PickCube 24D）：

| Bias | No-Bias | Fixed(0.2) | Random[0,0.25] |
|---|---|---|---|
| 0.00 | 100% | 83% | 92% |
| 0.05 | 100% | 87% | 100% |
| 0.10 | 80% | 99% | 99% |
| 0.15 | 62% | 100% | 96% |
| 0.20 | 51% | 100% | 97% |
| 0.25 | 5% | 100% | 96% |
| 0.30 | 0% | 99% | 99% |

**观测空间消融**（PickCube）：

| 观测 | 维度 | 缺少信号 | 结果 | 理论预测 |
|---|---|---|---|---|
| 18D | 18 | real_tcp + block | 42% avg | 不可辨识（Prop 1） |
| 21D | 21 | real_tcp | 训练失败 | 知目标不知己 |
| 24D | 24 | （完整） | 92–100% | 近似可辨识（Prop 2） |

### Backup Policy 仿真实验

**S1 V3b（V3 Path A 训完版，max_disp=0.50, 300k）**：
- 满存活率 95.0%（vs V3 71.5% / V2 100% 但真机存在肘撞）
- hand_collision 3.0%（vs V3 17.5%），excessive_displacement 5.5%
- 4 种运动模式（LINEAR / ARC / STOP_GO / PASSING）分项 ≥ 90%

**S1 偏差鲁棒性 zero-shot**（无偏差训练 → BiasJ1 评估）：
- baseline 99.0% → BiasJ1 98.5%（差距 0.5% ≈ 抽样噪声）

**S2 多障碍**（移动 + 静止）：83-91% 存活率（两次 eval 方差较大）

### Task Policy 真机实验（待数据填）

**§6.4.1 三任务联合成功率矩阵**：

| 任务 \ 配置 | 无 bias | 有 bias | + backup 无 bias | + backup 有 bias |
|---|---|---|---|---|
| pickup | TBD | TBD | TBD | TBD |
| wipe | TBD | TBD | TBD | TBD |
| pickandplace | TBD | TBD | TBD | TBD |

**§6.4.2 HG-DAgger 介入迭代**（pickup iter 1，30 episodes 实测）：
- total transitions: 1819
- intervention frames: 288 (15.8%)
- zero-intvn episodes: 18/30 (60%)
- 有介入 episodes 平均长度 67-97 vs 无介入 36-60

### 训练超参数

**sim HIL-SERL**：

| 参数 | 值 |
|---|---|
| batch_size | 256 |
| utd_ratio | 2 |
| discount | 0.97 |
| temperature_init | 0.01 |
| critic_lr / actor_lr | 3e-4 |
| critic_target_update | 0.005 |
| num_critics | 2 |
| online_steps | 100,000 |
| online_buffer | 100,000 |
| offline_buffer | 100,000 (50 demos) |
| warmup_steps | 500 |
| vision encoder | DINOv3-S frozen ViT-S/16, 22M frozen |

**real BC pretrain**：

| 参数 | 值 |
|---|---|
| BC steps | 20,000（per HG-DAgger iter） |
| batch_size | 256 |
| actor_lr | 3e-4 |
| std clamp | $[10^{-5}, 10]$ |
| BC 离散权重 λ | 0.5 |
| image aug | DrQ random_shift, pad=4 |
| HG-DAgger tail_k | 10 帧 |
| 介入触发阈值 | sm enter=0.05 / exit=0.02 / persist=3 |
| 每轮迭代 episode 数 | 30 |

---

## 写作流程

每章写作遵循项目 CLAUDE.md 的工程流程：

```
读 spec → /plan → 用户确认 → 写作 → review → 修改 → /commit → 下一章
```

### 写作顺序（按方案 B 重组后）

1. ✅ 本 spec 重写（C1）
2. **Ch.4 减薄（C2）** —— 删 §4.4 sim + §4.5 real，重编号 §4.6→§4.4 / §4.7→§4.5
3. **Ch.5 重组（C3）** —— 合并旧 §4.4 sim HIL-SERL + 旧 Ch.5 全部 + 旧 §6.A 实验 + V3b 几何升级新加段
4. **Ch.6 重组（C4）** —— 合并旧 §4.5 real BC+HG-DAgger + 旧 Ch.5 真机 §5.5 + §5.7 + 旧 §6.B 实验 + 旧 Ch.7 全部，删 main.tex 里 \input{ch7}
5. **Ch.6 真机数据填入（待数据 ~2026-05-09 后）** —— 三任务 4 列成功率矩阵
6. **Ch.2 重写** —— 含 IL 家族（前）+ RL 家族（后）
7. **Ch.1 写作** —— 按新三贡献框架
8. **Ch.8 写作** —— 含 future work 两步消融
9. 图表制作 + 参考文献整理 + 全文 review + 翻译

### 当前进度

- [x] 论文骨架搭建（中文版 + 英文版）
- [x] 章节提纲（已按方案 B 重排）
- [x] 摘要初稿（待按新标题重写）
- [x] 第 3.1 节（编码器偏差来源）
- [x] 第 3.2 节（因果链）
- [x] 第 3.3 节（POMDP 形式化）
- [x] 第 3.4.1–3.4.2 节（命题 1/2 的严格证明）
- [x] 第 3.4.3 节（可辨识性 ≠ 可解性）
- [x] 第 3.5 仿真实现
- [x] 第 3.4 Fig.3.3 可辨识性几何示意
- [ ] 第 4 章减薄（C2）—— 待执行
- [ ] 第 5 章重组（C3）—— 待执行
- [ ] 第 6 章重组（C4）—— 待执行（含原 Ch.7 内容）
- [ ] 第 6 章 §6.4.1 真机表（等数据）
- [ ] 第 2 章正文
- [ ] 第 1 章正文
- [ ] 第 8 章正文（含两步消融 future work）
- [ ] 图表制作（部分已有）
- [ ] 参考文献整理
- [ ] 全文 review + 翻译
