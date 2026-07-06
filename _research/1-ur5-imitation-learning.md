---
title: "Transformer-Based Imitation Learning for Long-Horizon Robotic Pick-and-Place Manipulation"
date: 2026-06-03
tags: [Robot Learning, Imitation Learning, Transformers, CoppeliaSim]
excerpt: "Can a policy trained offline on expert demonstrations reliably execute sequential robotic manipulation in closed-loop? This project investigates that question by comparing Behavioural Cloning (BC) and Action Chunking Transformers (ACT) on a randomized UR5 pick-and-place task in CoppeliaSim. Through controlled evaluation under identical initial conditions, I examined how covariate shift, temporal consistency, and action chunking influence long-horizon execution stability. Despite achieving very low offline validation error (MSE ≈ 0.001), BC frequently exhibited freezing and drift accumulation during deployment, whereas ACT produced stable trajectories and successfully completed all 20 evaluation episodes within the same workspace distribution used for demonstration collection.<br/>
    <a href='https://github.com/sunday-ikechukwu/ur5-imitation-learning'>
    <img src='/images/bc_act_rollout.gif' style='width:100%; max-width:700px;'></a>
    <br/>
    <a href= 'https://github.com/sunday-ikechukwu/ur5-imitation-learning'>Github</a>"

collection: portfolio
---
## Transformer-Based Imitation Learning for Long-Horizon Robotic Pick-and-Place Manipulation

**A Comparative Study of Behavioral Cloning and Action Chunking Transformers for robotic manipulation**

[[GitHub]](https://github.com/sunday-ikechukwu/ur5-imitation-learning)

---

### Overview

This project investigates long-horizon execution stability in robotic pick-and-place manipulation by comparing Behavioral Cloning (BC) and Action Chunking Transformers (ACT) under identical randomized initial condition using a UR5 arm in CoppeliaSim. 100 expert demonstrations were collected from a Lua IK-based expert policy across a randomised workspace, then used to train the two policies which where later evaluated in closed-loop across 20 identical test episodes.

The project reveals a stark performance gap between the two methods and provides concrete, experimentally-grounded insight into why frame-wise imitation learning fails on long-horizon manipulation tasks.

---

### Task

The robot must pick a cube from a randomised position on a pick table and place it on a fixed target table. The cube start position is uniformly sampled across a validated reachable workspace at the start of each episode.

- **Robot:** UR5 6-DOF arm with BarrettHand gripper
- **Simulator:** CoppeliaSim (ZMQ Remote API)
- **Demonstrations:** 100 episodes, ~5s each, ~3 Hz effective sampling rate
- **Observation:** joint angles (6) + EE pose (7) + cube position (3) + gripper state (1) = 17-dim
- **Action:** target joint angles (6-dim)

---

### Methods

**Behaviour Cloning (BC)** — A 3-layer MLP (256 hidden units, LayerNorm, Dropout) trained via supervised regression to map the current observation to the next joint configuration. Trained for 200 epochs on Google Colab (T4 GPU). Val loss: 0.001, MAE: 0.29°.

**ACT (Action Chunking with Transformers)** — An encoder-decoder transformer trained to predict a chunk of 10 future actions from the current observation, with action smoothness regularisation. Action chunking breaks the per-step feedback loop that causes BC to stagnate. Trained for 200 epochs. Val loss: 0.030, MAE: 0.43°.

---

### Results

Despite BC achieving lower per-step MAE during offline evaluation, ACT dramatically outperformed it in closed-loop deployment:

| Metric | BC | ACT |
|---|---|---|
| **Success Rate** | 0% | **100%** |
| Grasp Rate | 10% | 100% |
| Avg Steps to Grasp | 198.5 | **40.85** |
| Avg Placement Error | N/A | **4.66 cm** |
| Primary Failure Mode | Trajectory stagnation | — |

### BC/ACT Full Rollout
<table>
  <tr>
    <td align="center"><b>BC Policy - 0% Success</b></td>
    <td align="center"><b>ACT Policy - 100% Success</b></td>
  </tr>
  <tr>
    <td><img src="/images/bc_rollout.gif" width="100%"></td>
    <td><img src="/images/act_rollout.gif" width="100%"></td>
  </tr>
</table>

>BC failed on all 20 episodes due to covariate shift, once the arm deviated slightly from expert trajectories, the policy entered states unseen during training, producing low-magnitude actions that caused the arm to stall. ACT completed every episode in under 90 steps with consistent sub-5cm placement accuracy.

---

### Key Insight

> Low offline validation loss does not guarantee successful closed-loop robotic control.

BC achieved a validation MSE of 0.001, well-optimised by supervised learning standards, yet failed entirely during deployment. This is a direct manifestation of the **teacher forcing gap**: during training, the model observes correct expert states; during inference, it must act on its own imperfect predictions, causing errors to compound over time.

ACT's action chunking directly addresses this by committing to 10-step trajectories before re-querying, reducing the frequency of potentially error-inducing policy queries and improving temporal consistency.

---

### Visualisations

**Joint trajectory comparison — same cube start position:**

The plot below shows how BC and ACT execute the task from identical starting conditions.
ACT executes a decisive multi-phase trajectory, completing grasp at step 42 and release at step 88.
BC drifts slowly, triggers a late grasp at step 185, then immediately stalls with no further progress.

<img src="/images/bc_vs_act_joint_trajectory.png" style="width:100%; max-width:700px;">

**ACT placement accuracy across 20 episodes:**

<img src="/images/act_placement_accuracy.png" style="width:100%; max-width:700px;">

**Quantitative comparison:**

<img src="/images/act_vs_bc_comparison_plots.png" style="width:100%; max-width:700px;">

---

### Technical Contributions

- Implemented a state-based ACT policy for sequential robotic pick-and-place manipulation and evaluated its effect on long-horizon execution stability
- Automated expert demonstration pipeline using CoppeliaSim ZMQ Remote API
- Custom HDF5 dataset with full schema documentation ([data/README.md](https://github.com/sunday-ikechukwu/ur5-imitation-learning/blob/main/data/README.md))
- Position-based grasp emulation mirroring Lua expert controller logic
- In-memory ACT dataset with action chunking for efficient Colab training
- I built an end-to-end imitation learning pipeline, investigated why Behavioral Cloning fails in long-horizon manipulation, instrumented rollout evaluation, and empirically demonstrated how temporal action chunking improves closed-loop robotic control.
- Comprehensive BC failure mode taxonomy with root cause analysis

---

### Limitations and Future Work

This project implemented a <b>state-based Action Chunking Transformer (ACT)</b> policy, where the transformer received structured robot state observations (joint positions, end-effector pose, cube position, and gripper state) rather than raw visual inputs.

This formulation enabled controlled investigation of long-horizon execution stability, temporal consistency, and action chunking in robotic manipulation. However, it does not fully capture the perception challenges encountered in real-world robotic systems, where policies must infer task-relevant information directly from visual observations.

In addition, the study was conducted entirely in simulation using a deterministic grasping mechanism based on object attachment (`setObjectParent`) rather than contact-rich physical grasping. While this design isolated trajectory learning from grasp physics and enabled reproducible evaluation, it simplifies important aspects of real robotic manipulation such as contact dynamics, grasp uncertainty, and object slippage.

Evaluation was also performed within the same workspace distribution used during demonstration collection. Although randomized cube positions prevented simple memorization, the experiments did not explicitly evaluate out-of-distribution generalization to substantially different object placements or workspace configurations

Future work will focus on extending the framework toward a <b>visuomotor ACT implementation</b>, incorporating camera observations and image-based scene understanding. This would enable investigation of:

- visual feature learning for manipulation,
- robustness to perception uncertainty,
- end-to-end visuomotor imitation learning,
- and deployment in more realistic robotic environments.

Additional future directions include:

- increasing demonstration diversity and dataset scale,
- higher-frequency synchronized trajectory recording,
- evaluation under larger workspace randomization,
- explicit out-of-distribution generalization testing,
- comparison with diffusion-based manipulation policies,
- contact-rich grasping and physics-based interaction,
- and sim-to-real transfer studies on physical robotic platforms.

>Overall, the project serves as a controlled investigation into the strengths and limitations of imitation learning for sequential robotic manipulation, while providing a foundation for future research on visuomotor learning and robust long-horizon control.
---

### References

- Zhao et al. (2023). *Learning Fine-Grained Bimanual Manipulation with Low-Cost Hardware.* RSS 2023.
- Hussein et al. (2017). *Imitation Learning: A Survey of Learning Methods.* ACM CSUR.