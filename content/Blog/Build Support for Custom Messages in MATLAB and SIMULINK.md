---
title: Build Support for Custom Messages in MATLAB and SIMULINK
date: 2025-05-20
draft: false
tags: 
description:
---
Created: 2025-05-20
## Links:
* [ros2genmsg](https://www.mathworks.com/help/ros/ref/ros2genmsg.html)
* [ROS Toolbox System Requirements](https://www.mathworks.com/help/ros/gs/ros-system-requirements.html)
## Steps
1. Download the required message folder, e.g for ackermann_msgs: either clone the repository (`git clone https://github.com/ros-drivers/ackermann_msgs.git -b ros2`) or download the corresponding zip file (`wget https://github.com/ros-drivers/ackermann_msgs/archive/refs/heads/ros2.zip`).
2. Create a "customMsgs" directory for MATLAB then move the cloned/unzipped folder to the "customMsgs" directory.
3. In MATLAB:
    1. Specify the path to the "customMsgs" directory: `folderPath = fullfile(pwd, "customMsgs");`
    2. Build support for the custom messages: `ros2genmsg(folderPath)`
    3. Check if the custom messages are built: `ros2 msg list`