# Defense Speech Script (EN)

Target: 15–18 min. Per-slide timing in (parentheses). Stage directions in [brackets].
Deck: `slides/en/mockup-deck/index.html`

---

## Opening — Slide 1 · Cover  (~30 s)

Good afternoon, everyone. My name is Cheng Cheng. Thank you all for being here.

Today I will present my master's thesis — *Hierarchical Imitation and Reinforcement Learning for Fault-Tolerant Manipulation*.

This work was carried out at the Institute of Industrial Automation and Software Engineering, under the supervision of Professor Andrey Morozov.

---

## §1 Motivation  (~2 min)

### Slide 2 · Agenda  (~20 s)

Here is the outline of the talk. I will start with the motivation, then formalize the problem as a POMDP. After that I present the method, followed by the simulation and real-robot results, and I close with the conclusions and future work.

### Slide 3 · § 1 divider  (~5 s)

Let's begin with the motivation.

### Slide 4 · Motivation hook  (~45 s)

Industrial robots rely on joint encoders to know where they are. Every controller and every observer reads the encoder.

But in real deployment, encoders develop a *systematic bias* — after a collision, after a part is replaced without recalibration, or simply from thermal drift.

And the magnitude matters. A calibration offset of just 0.1 radian — about six degrees — shifts the end-effector by roughly six and a half centimeters. Under more severe bias, up to nineteen centimeters. That is far beyond the tolerance of any precision manipulation task.

Today the standard fix is to stop the production line and recalibrate. That costs throughput. Our goal is different — we want the robot to *adapt online*, without halting.

### Slide 5 · Fault-injection demo  (~30 s)

This is what encoder bias actually looks like on the real robot. [play video, ~8 s]

Here we inject a random bias at joint one. You can see the end-effector drift away from where the controller *thinks* it is. The controller is confident — but it is confidently wrong.

This drift is the problem the rest of the talk is about.

---

## §2 Problem Formulation  (~2.5 min)

### Slide 6 · § 2 divider  (~5 s)

To design a solution, we first need to formalize what kind of problem this is.

### Slide 7 · MDP → POMDP  (~75 s)

In a standard control setting, we have a Markov Decision Process — an MDP. The agent observes the full state, and the optimal policy is simply a function of that observable state.

Encoder bias breaks this. The encoder reads *q-measured*, which equals the true joint angle *q-true* plus a hidden bias *b*. The agent never sees *q-true* directly — it only ever sees the contaminated reading.

A hidden variable that the agent cannot observe — that is exactly the definition of a Partially Observable MDP, a POMDP. Formally, we extend the tuple with an observation space and an observation function.

So the first message is simple: control under encoder bias is *not* an MDP. It is a POMDP, and *b* is the hidden variable.

### Slide 8 · Causal chain  (~65 s)

Now, how does this one bias actually propagate? This is the causal chain — one source, three effects.

The single bias *b* enters at three independent points. First, the operational-space controller computes a wrong Jacobian and wrong forward kinematics. Second, the joint PD loop settles at an offset position. And third, the observation module returns biased joint readings and a biased end-effector pose.

Here is the key point: all three channels read *q plus b*. There is no clean channel anywhere. The agent never observes *b* directly — and that is precisely why this is a POMDP. The whole method that follows is about dealing with this one hidden variable.

---

## §3 Method  (~5 min)

### Slide 9 · § 3 divider  (~5 s)

Now the method. How do we make a policy robust to this hidden bias?

### Slide 10 · Mechanism 1 · Observation Augmentation  (~45 s)

The method has two mechanisms. The first is observation augmentation.

This table is our 25-dimensional observation, split by whether each channel is contaminated by the bias. The joint angles and the forward-kinematics TCP are contaminated — they all carry *q plus b*. But there is one channel that is *not*: the Real TCP, measured by an external sensor, independent of the encoder.

Why does this one channel matter? Without it, the agent cannot tell the true configuration apart from a shifted one — it can only learn an *average* compensation. With it, the difference between the biased and the real TCP is, approximately, a function of *b*. That difference is what lets the policy infer the bias implicitly.

### Slide 11 · Mechanism 2 · Random-Bias Training  (~45 s)

The second mechanism is random-bias training.

At every episode we sample a fresh bias — uniform, up to 0.25 radian, larger than typical industrial drift. The bias is never given to the policy as an input; the policy has to perceive it implicitly, through the 25-dimensional observation.

This sounds like domain randomization, but the information role is the opposite. Domain randomization randomizes nuisance variables the policy should *ignore*. We randomize a hidden variable the policy *must condition on* — it has to output a different compensation for a different bias.

So: augment the observation, and randomize the bias. Those are the two mechanisms.

### Slide 12 · Network Architecture  (~25 s)

All policies share one network — a frozen DINOv3 vision backbone, and three heads: a continuous actor, a twin critic, and a discrete gripper critic. The same architecture is used in simulation and on the real robot, and by both the task and the backup policy.

### Slide 13 · Task Policy · Simulation (HIL-SERL)  (~35 s)

So how is the task policy trained? There are two paths.

In simulation we use HIL-SERL — an off-policy RL framework with a human in the loop. We warm up with behavior cloning, then run SAC online, with a human intervening by keyboard to correct the policy.

Simulation allows this because rollouts are cheap, throughput is high, and the human interventions compensate for the sparse reward. Online RL converges.

### Slide 14 · Task Policy · Real Robot (BC + HG-DAgger)  (~35 s)

On the real robot the path is different — behavior cloning plus HG-DAgger.

We collect human demonstrations, train a behavior-cloning policy, and then iterate: the policy rolls out, the human takes over whenever needed, those corrections go into the buffer, and we retrain. Three iterations.

Why not online RL on the real robot? That is exactly the negative result I will show in the next section.

### Slide 15 · Backup Policy · Sim-only Training  (~40 s)

That was the task policy. The backup policy — the collision-avoidance policy — is trained differently again.

It is trained purely in simulation, with SAC and domain randomization, through a reward-and-geometry evolution we call V1 to V3b.

And importantly, it is never fine-tuned on the real robot. The task policy needs real-robot adaptation because its perception is vision-heavy — the sim-to-real gap is large. The backup policy reasons only over joint and Cartesian distances — the gap is small. So it transfers zero-shot.

### Slide 16 · Backup Policy · Real-Robot Deployment  (~25 s)

Here is the backup policy on the real robot — the V3b checkpoint loaded directly, no retraining. [play video, ~6 s] The geometry alignment matches the simulation exactly; the gap that needed closing was geometric, not perceptual.

### Slide 17 · Hierarchical Supervisor FSM  (~45 s)

Finally, how do the two policies coordinate at runtime? Through a Hierarchical Supervisor — a three-state finite-state machine.

In the TASK state, the task policy runs, and we monitor the distance to the human hand. If it gets too close, we switch to BACKUP. In BACKUP, the backup policy takes over until the hand is clear again — with a hysteresis margin to avoid chattering. Then we enter HOMING.

HOMING is the important one. A clipped-P controller resets the end-effector to the pose where backup was triggered, and hands control back to the task policy — same episode.

This means the backup maneuver does *not* contaminate the task trajectory. The backup contributes essentially zero to task success — and that is why, in the results, the success matrix has no backup column.

---

## §4 Simulation Results  (~3 min)

### Slide 18 · § 4 divider  (~5 s)

Let's look at the simulation results.

### Slide 19 · H1 / H2 / H3  (~50 s)

These are the three core simulation experiments, on the PickCube task. The chart shows success rate against encoder bias, for three policies.

H1, the red line — a policy trained without bias. It is perfect at zero bias, but it collapses as the bias grows: by 0.25 radian it is almost at zero. This is the degradation Proposition 1 predicts.

H2 — a policy trained at a single fixed bias. It is robust around that training point, but it *over-compensates* at zero bias — only 83 percent. Single-point conditioning overshoots.

H3 — our random-bias policy, the black line. It stays between 92 and 100 percent across the whole range, and it even extrapolates beyond the 0.25 radian training ceiling. Random-bias training works.

### Slide 20 · Observation Ablation  (~40 s)

Next, the observation ablation — this directly tests the propositions.

With the 18-dimensional observation — no Real TCP — the policy averages 42 percent. Not identifiable, exactly as Proposition 1 predicts. Adding only the object position, 21 dimensions, barely helps — 46 percent. The decisive jump is the Real TCP: at 24 dimensions, success reaches 92 to 100 percent.

So the propositions are not just information-theoretic statements — they predict which observation sets a policy can actually learn from.

### Slide 21 · Backup Policy in Simulation (V3 vs V3b)  (~35 s)

This slide is the backup policy in simulation. The interesting part is the V3-to-V3b comparison.

V3 reached only 66.5 percent survival. We changed *one* parameter — the displacement budget, from 0.40 to 0.50 meters. That single change fixed two failure modes at once: hand collisions dropped five-fold, and survival rose to 91 percent. One parameter, both failure modes.

### Slide 22 · HIL-SERL Negative Result  (~50 s)

And now the negative result — HIL-SERL on the real robot.

We tried to run the same online RL on the real robot. It failed. The moment the actor unfreezes, the policy diverges — the action norm jumps from 0.05 to 0.6, and the end-effector flies to the workspace edge.

Our leading hypothesis: under 50 demonstrations, a sparse reward, and short episodes, the critic's value surface is too flat — the gradient direction becomes essentially arbitrary. HIL-SERL needs high real-robot throughput to calibrate that surface; at 10 hertz, we don't have it.

We report this as a methodological negative result. It is the direct reason the real-robot path uses behavior cloning and HG-DAgger, not online RL.

---

## §5 Real-Robot Results  (~3 min)

### Slide 23 · § 5 divider  (~5 s)

Now the real-robot results — the core of the thesis.

### Slide 24 · Real-Robot System Setup  (~35 s)

This is the real-robot system. It is a distributed architecture: a real-time PC runs the 1-kilohertz control loop, and a GPU workstation runs the policies, the vision, and the supervisor. Control and computation are decoupled.

The encoder bias is injected at two points — B, in the control law, and D, in the HTTP interface — so the injection faithfully reproduces the causal chain.

### Slide 25 · HG-DAgger Convergence  (~35 s)

Here is the HG-DAgger convergence. The chart shows the human intervention rate over three iterations, for the three tasks.

All three start needing frequent intervention — around 40 to 50 percent — and all three converge by iteration two, below 5 percent. The matrix on the next slide uses these converged checkpoints.

### Slide 26 · ★ 4-Column Bias Matrix  (~75 s)  [核心 — 讲慢]

This is the central result of the thesis — the train-by-test bias matrix, on all three real-robot tasks. Let me read it row by row.

First row: trained without bias, tested without bias — everything works, near 100 percent.

Second row: trained without bias, tested *with* bias — this is the collapse. Success drops by more than half. A clean-trained policy simply cannot handle the bias.

Third row: trained *with* bias, tested without bias — and success is unchanged from the first row. This matters: bias-aware training has *no side-effect*. It does not hurt clean performance.

Fourth row: trained with bias, tested with bias — and success recovers, back into the usable range. This is the payoff. Random-bias training is necessary, it is sufficient, and it is side-effect-free.

There is still a residual gap — the fourth row sits below the third. Closing that gap is the most direct piece of future work. The three videos are one successful example per task, under bias.

### Slide 27 · Full System in Operation  (~25 s)

And here is the full system running. [play video, ~8 s] On the left, the monitor view; on the right, the real robot.

The task policy performs pickup under random bias; when a hand approaches, the supervisor switches to backup; HOMING resets the pose, and the task resumes — same episode, no manual intervention.

---

## §6 Conclusion  (~1.5 min)

### Slide 28 · § 6 divider  (~5 s)

To conclude.

### Slide 29 · Conclusion — Three Contributions  (~50 s)

Three contributions.

First, *theory* — I formalized encoder bias as a hidden variable in a POMDP, with the one-source-three-effect causal chain and two identifiability propositions.

Second, *method* — the task-plus-backup, sim-and-real dual-path framework, built on the two mechanisms: observation augmentation and random-bias training.

Third, *empirics* — the 4-column bias matrix on three real-robot tasks, plus the methodological negative result on HIL-SERL.

The main limitations: the real-robot loop still uses a privileged TCP channel; the bias is single-joint and episode-constant; and it is one platform with three tasks.

### Slide 30 · Future Work  (~30 s)

Three directions for future work. The most direct is a two-step ablation on the real robot, to close the identifiability loop. Second, extending to multi-joint and time-varying bias. And third, using a vision-language model to chain the three short-horizon tasks into one long-horizon task.

### Slide 31 · Thank You  (~5 s)

That concludes my presentation. Thank you for your attention — I am happy to take your questions.

---

*Total ≈ 17 min. Buffer for Q&A and pacing.*

