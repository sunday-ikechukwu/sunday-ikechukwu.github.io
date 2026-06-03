---
title: "Reactive Navigation under Uncertainty: Bug2 Implementation and Performance Analysis in ROS2"
date: 2026-05-05
excerpt: "ROS2/Gazebo implementation and evaluation of the Bug2 reactive navigation algorithm, including trajectory logging and multi-trial performance analysis to study limitations of purely reactive planning.<br/>
    <a href='https://github.com/sunday-ikechukwu/Bug2-Maze-Navigation-ROS2'>
    <img src='/images/bug2_demo.gif'>
    </a><br/>
    <a href= 'https://github.com/sunday-ikechukwu/Bug2-Maze-Navigation-ROS2/tree/main/src/bug2_navigation'>Code</a>"

collection: portfolio
---

### Overview

This project investigates reactive navigation using the Bug2 algorithm implemented in ROS 2 and evaluated in Gazebo. The objective is to study the behavior and limitations of purely reactive planning under structured and semi-structured environments, serving as a baseline for learning-based navigation methods.

### Methodology

The Bug2 algorithm was implemented from scratch using LiDAR and odometry data only, without access to global maps or planners. The robot switches between:

- Goal-seeking behavior along the M-line  
- Wall-following behavior upon obstacle detection  

### Experiment

The system was deployed in simulated custom maze environments using TurtleBot3 in Gazebo.

Multiple independent trials were conducted and the following metrics were recorded:

- Path length efficiency  
- Time-to-goal  
- Number of state transitions  

**A Path Efficiency** of **0.787** was recorded, indicating that the robot traveled approximately **21%** more than the straight-line distance, representing the detour cost of navigating around three obstacles. 

Trajectory data was logged and visualized using RViz.

**Trajectory visualization:**

<img src="/images/bug2_trajectory_plot.png" style="width:100%; max-width:700px;">

The trajectory plot visualizes the robot’s path, with blue segments indicating Move-to-Goal behavior and red segments indicating Wall-Following behavior, overlaid on the theoretical M-line (dashed). Three distinct wall-following cycles are observed, each triggered by encounters with internal obstacles. In each case, the robot successfully re-encounters the M-line and resumes goal-directed motion, demonstrating correct Bug2 state transitions and consistent recovery behavior in structured environments

### Dynamic Obstacle Intervention Test

To evaluate the robustness of the reactive navigation policy, a dynamic obstacle was introduced during execution while the robot was in the Move-to-Goal state. The obstacle was placed to obstruct the M-line, forcing an immediate behavioral transition.

The system’s response was analyzed in terms of:

- State transition latency (Move-to-Goal → Wall-Follow)  
- Recovery behavior after obstruction  
- Ability to re-establish the M-line and resume goal-directed motion  

This test simulates unexpected environmental changes and evaluates the responsiveness of the Bug2 algorithm under runtime perturbations.

<img src="/images/dynamic_obstacle_test_demo.gif">

<!-- [View Dynamic Test Video](/images/dynamic_obstacle_test_demo.gif) -->

### Key Insights

While Bug2 reliably reaches the goal in simple environments, performance degrades in cluttered scenarios due to repeated boundary tracking and inefficient re-alignment with the M-line. This highlights a key limitation of purely reactive methods: lack of global awareness and poor scalability in complex environments.

### Limitations

The system assumes accurate localization and does not account for sensor noise or dynamic obstacles. Additionally, simulation results may not fully reflect real-world physical uncertainties, motivating future work in robust and learning-based navigation.

### Future Works

This work serves as a foundation for for an already on ongoing:
- Nav2-based global planning integration  
- Learning-based local navigation (reinforcement learning)  and eventually 
- Robust sim-to-real transfer under uncertainty 

### 🔗 Links

[View on GitHub](https://github.com/sunday-ikechukwu/Bug2-Maze-Navigation-ROS2)  