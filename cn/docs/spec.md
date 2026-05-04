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
> → **任务策略（双端）**：仿真侧用 RLPD + HIL 验证可辨识性命题与随机偏差鲁棒训练（H1–H3 闭环）；真机侧采用仿真已验证的"成功信号路径"，用 BC pretrain + HG-DAgger 介入迭代部署
> → **备份策略（纯仿真）**：SAC + 域随机化训练，提供 task 训练 / 部署阶段的人机安全
> → **分层架构**：task ⊕ backup 在编码器故障注入下的联合成功 = 论文灵魂数据（§6.B 三任务 4 列成功率矩阵）

新标题对位：
- **Hierarchical** = task ⊕ backup 双策略分层
- **Imitation** = real BC + HG-DAgger（真机最终方案）
- **Reinforcement** = sim HIL-SERL + sim backup SAC
- **Fault-Tolerant** = 编码器偏差容错 + 人机安全

---

## 两个子系统的训练范式对比（核心架构决策）

这是论文的一个独立贡献点——**同一机器人系统对不同子任务、不同部署阶段选用不同的范式**：

| 维度 | 任务策略 sim（Ch.4 §4.4） | 任务策略 real（Ch.4 §4.5） | 备份策略（Ch.5） |
|---|---|---|---|
| **观测模态** | 25D 状态 + 双相机 RGB | 14D 状态 + 双相机 RGB | 纯状态 (28D × 3) |
| **训练算法** | HIL-SERL (RLPD + SAC + HIL) | BC pretrain + HG-DAgger | 纯在线 SAC |
| **训练场景** | 仿真 | 真机 | 纯仿真 |
| **Sim2Real 主要瓶颈** | — | 视觉域差距 + 小数据 sparse reward | 低维状态噪声/延迟 |
| **为什么这个范式** | 验证理论命题（H1–H3）、给真机方案提供"成功信号路径"凭据 | HIL-SERL 在 50 demo + sparse reward 真机上 actor 解冻后大幅漂移（论文级 negative result）→ 改用 BC + 介入修正 | DR 在低维状态噪声维度上覆盖 sim2real，递归安全（避免训 backup 时再需要"backup 的 backup"） |
| **观测核心挑战** | 编码器偏差 $\vect{b}$ | 视觉 sim2real（偏差暂未进观测，留 future work）| 手部位置噪声 + 延迟 |
| **是否含 hand 通道** | 含（h_active + p_hand）| 不含（切换决策在 supervisor 外做）| 含（避让目标）|

**范式选择由观测模态、数据规模、安全约束三者共同决定**——这正是论文方法论层的独立贡献。

---

## 章节结构（修订后）

### 第 1 章 绪论（4–5 页）

```
1.1 研究背景与动机
    - 工业机器人编码器故障的 5 种来源
    - 现有方案（停机标定、FDI/FTC）的不足
    - 在线适应（IL/RL）的动机

1.2 问题陈述
    - 编码器偏差使 MDP 退化为 POMDP
    - HIL 训练引入人机安全问题
    - 视觉 sim2real gap 与小数据真机场景的算法选型约束

1.3 论文贡献（三点）
    ① POMDP 形式化 + 可观测性分析（理论贡献，Ch.3）
    ② 分层模仿与强化学习的容错操作框架：
       - sim 端 HIL-SERL 验证理论 + real 端 BC+HG-DAgger 部署
       - task ⊕ backup 双策略分层
    ③ 真机三任务在编码器偏差注入下的端到端验证
       （含 HIL-SERL 在小数据真机失败的 negative result）

1.4 论文组织结构
```

### 第 2 章 相关工作（6–8 页）

**写作原则（与方法章分工）：**
Ch.2 = **领域景观、对比替代方案、定位本文的选择**；Ch.4/Ch.5 = **本文具体用了什么、怎么配置**。
- Ch.2 不重复 Ch.4/Ch.5 的技术配置细节；以对比和定位为主；引用方法章做细节指针。
- 重叠的引用 key 允许出现（同一篇论文 Ch.2 和 Ch.4 用途不同：景观位置 vs 本文配置）。

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
    - 状态空间 DR 的成功案例（用作 Ch.5 backup 范式的支撑）

2.5 模仿学习与人在回路
    - BC / DAgger (Ross 2011) / HG-DAgger (Kelly 2019)
    - HIL-SERL (Luo 2025, Science Robotics) —— 本文 sim 端采用
    - RLPD (Ball 2023) —— HIL-SERL 的底层算法
    - 对比维度：纯在线 SAC / pure BC / DDPGfD / vanilla DAgger / HG-DAgger / HIL-SERL
    - 定位本文双端选择：sim HIL-SERL 验证理论 + real BC+HG-DAgger 部署
    - 不重复 §4.4 / §4.5 的技术配置细节

2.6 安全强化学习与避障
    - Kiemel 2024 非负即时奖励
    - Reward hacking / reward misspecification 概念背景
      （为 5.4 奖励演化分析做准备）
```

### 第 3 章 编码器偏差建模与 POMDP 分析（8–10 页）⭐ 理论核心

**基本不动**——理论核心，是论文最硬贡献。已写完 3.1–3.4.2，待写 3.4.3 + 3.5。

```
3.1 工业机器人编码器校准偏差
    - 5 种偏差来源（Tab.1）
    - 本文聚焦：episode 内恒定的偏差

3.2 因果链：一源三效应
    - 3.2.1 理想情况下的控制与观测数据流（Fig.3.1）
    - 3.2.2 效应①：初始位置偏移（PD 控制律稳态推导）
    - 3.2.3 效应②：控制偏差（OSC 一阶线性化）
    - 3.2.4 效应③：感知偏差（q + FK(q) 两个位置类观测共污染）
    - 3.2.5 因果链总图（Fig.3.2）

3.3 POMDP 形式化
    - 3.3.1 无偏差 MDP 基准
    - 3.3.2 有偏差 POMDP 七元组
    - 3.3.3 b 的 episode 级静态性 + 两个推论

3.4 可观测性分析
    - 3.4.1 命题 1：基础观测下的不可辨识性（严格命题+证明）
    - 3.4.2 命题 2：观测增强恢复可辨识性（严格命题+证明）
    - 3.4.3 可辨识性 ≠ 可求解性
          · 偏差物理上仍存在，策略必须以 b 为条件输出补偿
          · 噪声 ε > 0 时的"平均性补偿"
          · 为 Ch.4 §4.4 sim 随机偏差训练铺垫"P(b) 上期望最优"
          · 为 Ch.4 §4.5 真机当前用 tcp_true 直接绕开偏差留 future work 出口

3.5 仿真实现与等价性分析
    - 3.5.1 故障注入的充分条件（与实现无关的两个条件）
    - 3.5.2 三效应的 MuJoCo 实现 + 临时替换 qpos 的等价性
    - 3.5.3 外部传感器仿真（含噪声真实 TCP）
    - 3.5.4 仿真-真实差距分析（8 项对比表）
    - 桥接：真机 B+D 注入方案详见 Ch.7
```

### 第 4 章 任务策略：仿真理论验证与真机模仿部署（10–12 页）

```
4.1 方法设计动机
    基于 Ch.3 推导出三个设计选择：
    (1) 观测增强（来自 3.4 Prop 2）
    (2) 随机偏差训练（来自 3.3.3 和 3.4.3 的条件补偿要求）
    (3) 双端范式：sim 验证理论 + real 部署成功子路径
       - sim 端：HIL-SERL 既验证 H1–H3 又证明方法可行
       - real 端：sparse reward + 50 demo 小数据下 SAC online 漂移，改 BC+HG-DAgger
       - 真机当前观测使用 tcp_true 通道（理论"成功信号路径"），
         bias 鲁棒性闭环留 §8 future work

4.2 仿真观测空间设计（25D）
    - 25D = robot(18) + real_tcp(3) + hand_active(1) + hand_pos(3)
    - 每个分量的信息论角色 + 是否受偏差污染
    - 透明脚注：仿真实现层面环境接口为 31D 默认，其中 block_pos / plate_pos
      在本文实验中按 episode reset 至确定性位置、对策略恒为常量信道；
      为避免方法论误解和与真机简化对齐，本文以剔除常量信道后的 25D 形式描述。
      定量结果等价于在 25D 下重跑（常量维度梯度恒为 0）。

4.3 随机偏差训练
    - b ~ U[0, b_max] 作为对隐变量的 DR
    - 与固定偏差对比：过补偿问题（接 §6.A.2 实验）
    - 与 Tobin 视觉 DR 的同源性 + 区别

4.4 仿真训练框架：HIL-SERL（RLPD + SAC + HIL）
    - 离线 demo buffer + 在线 buffer + 50/50 混合采样
    - 键盘干预转移加入 offline buffer
    - 分布式 Actor-Learner gRPC 架构（Fig.4.2）
    - SAC 损失 + 网络架构 + 关键超参数（Tab.4.1）
    - 稀疏奖励的可行性论证

4.5 真机部署框架：BC pretrain + HG-DAgger 介入迭代 ⭐ 新增
    - 真机观测简化（14D = tcp_true 7 + tcp_vel 6 + gripper 1）
      · 编码器路径暂不进观测，统一用外部 tcp_true（"成功信号路径"）
      · hand 通道交由 supervisor 外部消费
      · environment_state 6D 当前为 placeholder（待物体检测器接入）
    - BC pretrain：纯监督学习，从 demo pickle 学动作映射
    - HG-DAgger 介入迭代：BC 部署 → 操作员 SpaceMouse 介入 → 介入帧合并重训
    - 三任务 deploy（pickup / wipe / pickandplace），独立 config + 任务级夹爪锁定
    - HIL-SERL 在真机失败的诚实复盘：
      · actor 解冻后 critic Q 平坦 → policy 漂移
      · 50–200 demo 小数据 + sparse reward 不足以让 critic calibrate
      · → 改用 BC + DAgger 后稳定收敛
    - 这是论文的方法论级 negative result（Ch.6 §6.B 实证）

4.6 网络架构与训练流程
    - 共享 encoder（DINOv3-S frozen ViT-S/16, 22M frozen，2026-04-26 起；之前 ResNet10）
    - State / Image encoder → Actor / Critic（Fig.4.3）
    - sim 端 SAC 损失；real 端 BC NLL（actor only）
    - 关键超参数（Tab.4.2 sim / Tab.4.3 real）

4.7 稀疏奖励设计
    - sim：r=1 iff (block-plate distance<5cm) AND (height_gain>20cm)
    - real：键盘 reward（S/Enter/Space/Backspace 协议），人工裁决
    - 为什么稀疏奖励可行（HIL bootstrap + demo 引导）
```

### 第 5 章 纯仿真与域随机化的备份避障策略（7–8 页）

```
5.1 问题定义与训练范式选择
    - Ch.5 的角色：配套 Ch.4 task 训练 / 部署的安全子系统
    - 为什么不用 HIL：观测低维 + DR 可覆盖 + 递归安全
    - 与 Ch.4 双端范式的对比（明确方法论对照）

5.2 场景设计与观测空间
    - S1 单移动障碍物（28D × 3 帧 = 84D）
    - S2 移动 + 静止障碍物（38D × 3 帧 = 114D）
    - 帧堆叠 + 障碍物速度估计

5.3 障碍物运动模式
    - LINEAR / ARC / STOP_GO / PASSING

5.4 奖励函数设计演化（Reward Hacking 案例研究）
    - V1: proximity reward → 策略逃跑（典型 reward hacking）
    - V2: 全负奖励 → "快速死亡优于缓慢死亡"
    - V3: 非负即时 + 大终止惩罚（Kiemel 2024）
    - V3 → V3b 几何升级：
      · 单球 → 5 球全臂（link3/4/5/6/hand）
      · obstacle r=0.10（对齐真机 hand bbox）
      · saturating proximity bonus（饱和 0.20 @ clearance 0.10）

5.5 真机部署的手部检测选型 ⭐ 改写
    - 设计演化：MediaPipe → WiLoR
    - MediaPipe 初选理由（CPU 实时、轻量）
    - 部署阶段发现的问题：
      · WiLoR YOLO 把 Franka gripper 误识别为人手（self-detection）
      · MediaPipe 在工作台俯视角下手部姿态多变检出不稳
    - 切换到 WiLoR YOLO + 双参考点几何过滤：
      · flange_radius=0.10 球（手掌+腕部）
      · tcp_radius=0.06 球（finger 末端）
      · 任一命中视为 self-detection 丢弃
    - 部署精度：~2-4cm 累计误差 vs 8cm 阈值，留有 margin

5.6 面向 Sim-to-Real 的域随机化
    - 位置噪声 σ=3cm、速度噪声 σ=1cm/s、观测延迟 U(0, 2) 步
    - 仅作用于观测层，碰撞检测用真实位置
    - 为什么 DR 对此问题足够：低维状态 + 粗粒度任务

5.7 FSM 切换与几何对齐（Route A）
    - Hierarchical Supervisor 三态 FSM（TASK / BACKUP / HOMING）
    - V2 → V3b 阈值升级：D_SAFE 0.30 → 0.40m, D_CLEAR 0.35 → 0.45m
    - center-to-center 几何（替代历史 Minkowski 虚点方案）
    - HomingController 6D clipped-P 控制器
    - 工作空间硬 clamp（部署侧）vs 训练侧位移 penalty 软引导
```

### 第 6 章 实验与分析（10–12 页）

实验拆 sim / real 两半。**所有 sim 数据都是历史已跑实验**，不补做新实验。
（历史实验补做规划见已废弃的 `ch6_experiment_plan.md`，仅保留为研究路径档案。）

```
6.1 实验设置（共通）
    - 评估协议、seeds、统计口径

— §6.A 仿真实验（理论验证 + 算法验证）—

6.A.1 H1：编码器偏差退化验证（PickCube env）
    - 无偏差基线 → 不同 bias 下性能退化曲线
    - 对应 Ch.3.4 Prop 1 的实证

6.A.2 H2 + H3：固定 vs 随机偏差训练（PickCube env）
    - 三策略对比（No-bias / Fixed b=0.2 / Random U[0, 0.25]）
    - 验证：随机偏差 92–100% 全范围鲁棒、固定偏差零点过补偿
    - 对应 Ch.3.4.3 + Ch.4.3 的实证

6.A.3 观测空间消融（PickCube 18 / 21 / 24D）
    - 直接对应 Ch.3.4 Prop 1 / Prop 2 的命题
    - 18D 不可辨识 / 21D 知目标不知己 / 24D 近似可辨识
    - 注：消融在 PickCube 24D 上做以隔离编码器偏差因素；
      Ch.4 §4.2 的部署 25D 是其 + hand 通道的等价扩展

6.A.4 Backup S1 主存活率
    - V3b 95% 全模式存活率（4 种运动模式分项）
    - 含 hand_collision / excessive_displacement 失败模式分布

6.A.5 Backup 偏差鲁棒性 zero-shot
    - 无偏差训练的 backup 直接在 BiasJ1 env 评估
    - 印证 backup 对编码器偏差天然鲁棒

6.A.6 Reward V1 / V2 / V3 演化
    - 训练曲线对比 + reward hacking 实证
    - V1 跑远 / V2 快死 / V3 稳定收敛

6.A.7 Backup S2 多障碍场景扩展
    - 单移 + 单静的 ~85% 存活率，论证架构可扩展性

— §6.B 真机实验（论文灵魂数据，等数据填）—

6.B.1 三任务 4 列成功率矩阵 ⭐ 论文核心
    | 任务 \ 配置 | 无 bias | 有 bias | + backup 无 bias | + backup 有 bias |
    | pickup |  |  |  |  |
    | wipe |  |  |  |  |
    | pickandplace |  |  |  |  |
    （等真机数据 2026-05-07~08 到位后填）

6.B.2 HG-DAgger 介入迭代曲线
    - iter 0 BC → iter N BC 的介入率下降趋势
    - zero-intvn episode 占比演化

6.B.3 编码器偏差注入的端到端验证
    - bias monitor 时序图：q_true vs q_biased 物理扫动
    - Ch.7 §7.2 的 B+D 注入实测数据

6.B.4 Backup 真机避让评估（含 task 触发）
    - WiLoR 检出率 / FSM 触发率 / 避让收敛步数
    - HOMING 收敛精度（POS_TOL 2cm / ROT_TOL 0.05rad）

6.B.5 HIL-SERL 真机 negative result 复盘
    - 多次实验观察到的 actor 漂移现象（数据 + 时间线）
    - 切换到 BC+HG-DAgger 后的对比效果
    - 论文方法论级负面结论
```

### 第 7 章 真机部署与验证（6–8 页）

```
7.1 硬件架构
    - RT PC（PREEMPT_RT + libfranka 0.9.1）+ GPU 工作站双机
    - 网络分离：直连 Franka + 间接 GPU
    - franka_server (Flask) + serl_franka_controllers (C++ 阻抗)

7.2 编码器偏差真机注入：B+D 双注入点
    - 为什么 /joint_states 注入无效（控制器读 FrankaStateHandle，不读话题）
    - 注入点 B（C++ 阻抗控制器）：RealtimeBuffer<bias> + biased FK/Jacobian
    - 注入点 D（Flask franka_server）：HTTP 路由 + biased state topic
    - 2026-04-15 端到端验证：Joint 1 0.1 rad → 物理扫动 ~7cm
    - 实测时序图（接 §6.B.3）

7.3 OSC 与笛卡尔阻抗控制的等价性
    - 偏差通过相同路径影响两者：q → J(q+b)
    - Λ 任务空间惯性矩阵的差异（仿真有 / 真机无）
    - 真机偏差幅度可能不同但方向一致 → 仿真是保守估计

7.4 任务策略真机部署：三任务统一 pipeline
    - pickup / wipe / pickandplace 的差异（夹爪锁定模式 / reset_pose / workspace ROI）
    - BC checkpoint + HG-DAgger 介入 + KeyboardRewardListener
    - workspace ROI 鼠标框选标定（替代 ArUco 4 角投影）

7.5 备份策略真机部署
    - WiLoR YOLO + D455 深度 + 双参考点 self-detection 过滤
    - HierarchicalSupervisor + HomingController（参数承接 §5.7）
    - sim2real 几何对齐（Route A center-dist + TCP_OFFSET 反推 flange）

7.6 操作员安全约定与失效模式
    - 人手不能伸入 flange 10cm 球（self-detection 盲区）
    - 阻抗收尾 ±2-5mm 抖动 + done_consecutive_n=3 streak
    - 真机 deploy 不做 episode 终止，靠 supervisor 切换

7.7 实测精度与失效复盘
    - cam-to-robot 标定精度 ~15-20mm（SVD 主求解）
    - 真机训练 round 4 / round 5 的 P0 修复总览（HTTP 重试、夹爪锁定、温度防坍塌等）
```

### 第 8 章 总结与展望（2–3 页）

```
8.1 工作总结
    - POMDP 形式化与可观测性分析（Ch.3）
    - 双端任务策略：sim HIL-SERL 验证 + real BC+HG-DAgger 部署
    - 纯仿真 backup + WiLoR 真机感知
    - 分层 task ⊕ backup 在故障注入下的端到端验证
    - HIL-SERL 真机失败的方法论级 negative result

8.2 局限性
    - 真机当前用 tcp_true privileged 通道，bias 鲁棒性闭环未在真机验证
    - environment_state 6D 当前为 placeholder（物体检测未接入）
    - 单关节 / 固定类型偏差
    - 三任务规模 + 单一硬件平台

8.3 未来工作
    - **真机 bias 鲁棒性两步消融**（与 Ch.4 §4.5 对应）：
      (i) 真机 14D 的 tcp_true → tcp_biased 应失败（验证编码器路径单独不够）
      (ii) 在 (i) 基础上补 q+b 信号 → 应恢复成功（与 sim 25D 路径同构）
    - 显式偏差估计器（在线最小二乘 from Jacobian / RMA 风格 GRU）
    - 多关节 / 时变偏差
    - 多故障类型扩展（编码器噪声 / 关节卡死 / 执行器退化 / 漂移偏差）
    - 物体检测接入 → environment_state 真值化
```

---

## 已确定的设计决策

| 决策 | 选择 | 原因 |
|---|---|---|
| 论文标题 | Hierarchical IL + RL for Fault-Tolerant Manipulation | 准确覆盖 sim RL + real IL 双端范式 + 分层 task⊕backup + 故障容错 |
| 理论框架 | 标准 POMDP（不引入 HiP-MDP） | 术语通用，审稿人熟悉 |
| 可观测性分析严格度 | 严格命题 + 构造性证明 | 硕士论文需要理论深度 |
| 仿真注入实现 | 放第 3.5 节 | 属于建模层面 |
| 真机注入实现 | 放第 7.2 节 | 属于部署层面 |
| 偏差类型 | 固定/随机（episode 内恒定） | 恒定隐变量是清晰 POMDP 结构 |
| 可辨识性 ≠ 可解性 | 3.4.3 专门小节澄清 | 回应读者质疑 + 为 §4.5 真机简化留出口 |
| **sim 任务策略观测** | **25D**（描述层面） | 删常量信道 plate/block，与真机对齐；实验沿用 31D 默认 env、加透明脚注说明 |
| **real 任务策略观测** | **14D = tcp_true 7 + tcp_vel 6 + gripper 1** | 对齐 hil-serl ram_insertion proprio 子集 |
| **§4.4 / §4.5 拆分** | sim HIL-SERL 验证理论 + real BC+HG-DAgger 部署 | sim 端 HIL-SERL 在仿真稳定，真机 sparse + 小数据下 actor 漂移 |
| **HIL-SERL 真机 negative result** | 在 §4.5 末段 + §6.B.5 诚实写明 | 是论文方法论级贡献而非缺陷 |
| **手部检测选型** | MediaPipe → WiLoR + 双参考点过滤 | 部署阶段 self-detection 问题倒逼切换 |
| **Backup 几何** | V2 单球 → V3b 5 球全臂 + obstacle r=0.10 | sim V2 真机部署观察到肘/前臂仍会撞 |
| **Backup FSM 阈值** | D_SAFE 0.30 → 0.40m, D_CLEAR 0.35 → 0.45m, center-to-center | Route A 与 sim 训练几何 1:1 对齐 |
| **§6.B 真机表为论文核心数据** | 三任务 × {无bias / 有bias / + backup × 2} 4 列矩阵 | task ⊕ backup 在故障注入下联合成功 = 论文灵魂 |
| **不补做新 sim 实验** | 全部沿用历史已跑数据 | 实验路径档案见 `ch6_experiment_plan.md`（已废弃） |
| Ch.5 POMDP 处理 | 轻提一句手部观测部分可观测，不做严格形式化 | DR 已覆盖观测退化 |
| Reward hacking | Ch.2.6 加背景 + Ch.5.4 用作分析框架 | V1/V2 是典型 reward hacking 案例 |
| 引用规范 | 每个引用必须有可核验的真实来源 | 已核实关键引用清单（见 git history） |
| 写作语言 | 先中文打磨 → 翻译英文/德文提交 | 母语写作效率高 |

---

## 关键图表清单

### 必需的图

| 编号 | 内容 | 所在章节 |
|---|---|---|
| Fig.3.1 | 理想情况下的控制与观测数据流 | 3.2.1 |
| Fig.3.2 | 编码器偏差因果链总图（一源三效应） | 3.2.5 |
| Fig.3.3 | 观测增强前后的可辨识性示意 | 3.4 |
| Fig.3.4 | 仿真注入流程（临时替换 qpos） | 3.5 |
| **Fig.4.1** | **sim 25D / real 14D 观测空间对比** | 4.2 + 4.5 |
| Fig.4.2 | sim HIL-SERL Actor-Learner gRPC 架构 | 4.4 |
| **Fig.4.3** | **真机 BC + HG-DAgger 介入迭代示意** | 4.5 |
| Fig.4.4 | 共享网络架构（state encoder + actor + critic） | 4.6 |
| Fig.5.1 | Backup Policy 场景示意（S1 / S2） | 5.2 |
| Fig.5.2 | 奖励 V1→V2→V3 训练曲线对比 | 5.4 |
| **Fig.5.3** | **WiLoR 双参考点 self-detection 过滤几何** | 5.5 |
| Fig.6.A.1 | H1 退化曲线（成功率 vs 偏差） | 6.A.1 |
| Fig.6.A.2 | 三策略对比图（no-bias / fixed / random） | 6.A.2 |
| Fig.6.A.3 | 观测维度消融条形图（18/21/24） | 6.A.3 |
| Fig.6.A.6 | Reward V1/V2/V3 训练曲线 | 6.A.6 |
| **Fig.6.B.1** | **三任务 4 列成功率柱状图** | 6.B.1 |
| **Fig.6.B.2** | **HG-DAgger 介入率下降曲线** | 6.B.2 |
| **Fig.6.B.5** | **HIL-SERL 真机 actor 漂移时序** | 6.B.5 |
| Fig.7.1 | 真机系统架构图 | 7.1 |
| Fig.7.2 | B+D 双注入点数据流 | 7.2 |
| **Fig.7.3** | **三任务 deploy 流程对比** | 7.4 |

### 必需的表

| 编号 | 内容 | 所在章节 |
|---|---|---|
| Tab.3.1 | 编码器偏差来源分类 | 3.1 |
| Tab.3.2 | 仿真-真实差距 8 项对比 | 3.5 |
| **Tab.4.1** | **sim 25D 观测空间各维度说明（含 31D → 25D 透明脚注）** | 4.2 |
| **Tab.4.2** | **real 14D 观测空间各维度说明** | 4.5 |
| Tab.4.3 | sim HIL-SERL 超参数 | 4.4 |
| Tab.4.4 | real BC + HG-DAgger 超参数 | 4.5 |
| Tab.5.1 | 障碍物运动模式与参数 | 5.3 |
| Tab.5.2 | DR 参数表 | 5.6 |
| **Tab.5.3** | **手部检测选型对比（MediaPipe vs WiLoR）** | 5.5 |
| Tab.6.A.1 | H1 退化数据 | 6.A.1 |
| Tab.6.A.2 | 三策略对比表 | 6.A.2 |
| Tab.6.A.3 | 观测消融 vs 理论预测对应表 | 6.A.3 |
| Tab.6.A.4 | Backup S1 / V3b 各运动模式存活率 | 6.A.4 |
| **Tab.6.B.1** | **三任务 4 列成功率矩阵 ⭐** | 6.B.1 |
| Tab.6.B.2 | HG-DAgger 迭代统计 | 6.B.2 |

---

## 实验数据汇总

> 注：以下 sim 数据为历史 PickCube 24D env 实测；论文 Ch.4 §4.2 描述为 25D 部署观测（24D + hand 通道），消融在 PickCube 24D 上做以隔离编码器偏差因素，结论可类比扩展。

### Task Policy 仿真实验（PickCube 24D）

**无偏差基线（18D）**：100% 成功率，13.4±0.6 步

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

### Task Policy 真机实验（待数据填，~2026-05-07~08）

**§6.B.1 三任务成功率矩阵**：

| 任务 \ 配置 | 无 bias | 有 bias | + backup 无 bias | + backup 有 bias |
|---|---|---|---|---|
| pickup | TBD | TBD | TBD | TBD |
| wipe | TBD | TBD | TBD | TBD |
| pickandplace | TBD | TBD | TBD | TBD |

**§6.B.2 HG-DAgger 介入迭代**（pickup iter 1，30 episodes 实测）：
- total transitions: 1819
- intervention frames: 288 (15.8%)
- zero-intvn episodes: 18/30 (60%)
- 有介入 episodes 平均长度 67-97 vs 无介入 36-60

### 训练超参数

**sim HIL-SERL（PickCube task policy）**：

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

**real BC pretrain（task policy）**：

| 参数 | 值 |
|---|---|
| BC steps | 20,000（per HG-DAgger iter） |
| batch_size | 256 |
| actor_lr | 3e-4 |
| std_min / std_max | (-5, 2) |
| image aug | random crop + color jitter |
| HG-DAgger tail_k | 10 帧 |
| 介入触发阈值 | sm enter=0.05 / exit=0.02 / persist=3 |

---

## 写作流程

每章写作遵循项目 CLAUDE.md 的工程流程：

```
读 spec → /plan → 用户确认 → 写作 → review → 修改 → /commit → 下一章
```

### 写作顺序（按依赖）

1. ✅ **本 spec 重写**（本次）—— 后续所有章节修订对照源
2. **Ch.4 §4.1–4.7 重写**（含 §4.4 sim / §4.5 real 拆分、25D 描述、HIL-SERL negative result）
3. **Ch.5 §5.1 / §5.5 / §5.7 修订**（WiLoR 选型 + V3b 几何 + Route A center-dist）
4. **Ch.6 §6.A 重写 + §6.B 占位**（等真机数据 ~2026-05-07~08）
5. **Ch.7 重写**（硬件 + B+D 实测验证 + 三任务 deploy + WiLoR 安全）
6. **Ch.6 §6.B 数据填入**（数据到位后）
7. **Ch.2 重写**（含 IL 家族 + HG-DAgger）
8. **Ch.1 写作**（按新三贡献框架）
9. **Ch.8 写作**（含 future work 两步消融）
10. 图表制作 + 参考文献整理 + 全文 review + 翻译

### 当前进度

- [x] 论文骨架搭建（中文版 + 英文版）
- [x] 章节提纲（已按新方案重排）
- [x] 摘要初稿（待按新标题重写）
- [x] 第 3.1 节（编码器偏差来源）
- [x] 第 3.2 节（因果链）
- [x] 第 3.3 节（POMDP 形式化）
- [x] 第 3.4.1–3.4.2 节（命题 1/2 的严格证明）
- [ ] 第 3.4.3 节（可辨识性 ≠ 可解性）
- [ ] 第 3.5 仿真实现（原 3.6 重命名）
- [-] 第 4 章（旧版已写，**待按新方案 §4.4/§4.5 拆分重写**）
- [-] 第 5 章（旧版已写，**待按 WiLoR + V3b + Route A 修订**）
- [ ] 第 6 章 §6.A 正文（数据已有，待写）
- [ ] 第 6 章 §6.B 真机表（等数据 ~2026-05-07~08）
- [ ] 第 7 章正文（按新方案重写）
- [ ] 第 2 章正文
- [ ] 第 1 章正文（按新三贡献写）
- [ ] 第 8 章正文（含两步消融 future work）
- [ ] 图表制作
- [ ] 参考文献整理
- [ ] 全文 review + 翻译
