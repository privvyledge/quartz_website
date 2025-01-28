---
title: Real-Time Trajectory Tracking
date: 2025-01-27
draft: true
tags:
  - autonomous-vehicles
  - mpc
  - optimization
  - controls
  - ros
  - ros2
  - robotics
  - casadi
  - acados
  - python
description: A Tutorial on Real-Time Trajectory Tracking
---
Created: 2025-01-27

# Real-Time Trajectory Tracking
Autonomous vehicles have seen a recent boost in popularity thanks to the increase in awareness spurred by popular Artificial Intelligence models as well as increase in computational power. The autonomous driving stack takes in sensor data (cameras, LIDAR, RADAR, IMU, GPS, etc.), preprocesses these sensor stream, perceives obstacles around the vehicle as well as the vehicles location (localization), then plans a path/trajectory from the current location to a goal location (or waypoints) and finally passes the control actions to lower level controllers to move the actuators.

## Trajectory Tracking/Following
Trajectory tracking involves driving a car through a series of waypoints at specific speeds or with strict time requirements. Probably the most common way of accomplishing this is through Model Predictive Control (MPC). MPC is a powerful optimization method since it allows customization of desired objective functions (trajectory tracking) while enforcing constraints, e.g speed and actuator limits and respecting the dynamics of the vehicle. This article focuses on implementing real-time trajectory trackers on a real F1/10 scale car as well as in a realistic simulator (CARLA) using [[ROS2]]. The corresponding code is available on my github page [here](https://github.com/privvyledge/trajectory_following_ros2).

## Vehicle Model
Many models have been developed for autonomous vehicles [[Trajectory Tracking Models]] with the choice of the model yielding various performance implications beyond the scope of this article. For this article, we will use the Kinematic Bicycle Model in the cartesian frame. However, using my code above, the various models from [[Trajectory Tracking Models]] can be used interchangeable.
### todo: add model and equations here with figures

## Model Predictive Controller
The constrained optimization problem for the trajectory tracking problem is as follows:
### todo: add MPC formulation.

The above is modelled using the Casadi toolbox and the corresponding C code is generated and compiled for increase in performance. See [[Optimization Notes]] for more optimization tips.





