---
title: ROS2 Installation
date: 2025-06-03
draft: false
tags:
  - ros2
description:
---
Created: 2025-06-03
Follow the [official instructions](https://docs.ros.org/en/humble/Installation.html) for your platform. I have setup a convenience script [todo: here](https://github.com/privvyledge/). The steps installing ROS 2 Humble on an Ubuntu 22.04 system as detailed [here](https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debs.html) are reproduced below for convenience.

```bash
export ROS_DISTRO=humble
# Set locale
locale  # check for UTF-8

sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

locale  # verify settings

# Setup sources
sudo apt install software-properties-common -y
sudo add-apt-repository universe -y
sudo apt update && sudo apt install curl -y
export ROS_APT_SOURCE_VERSION=$(curl -s https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest | grep -F "tag_name" | awk -F\" '{print $4}')
curl -L -o /tmp/ros2-apt-source.deb "https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}.$(. /etc/os-release && echo $VERSION_CODENAME)_all.deb" # If using Ubuntu derivates use $UBUNTU_CODENAME
sudo apt install /tmp/ros2-apt-source.deb

sudo apt update && sudo apt upgrade

# Install ROS Desktop Full and other ROS tools
sudo apt install ros-humble-desktop-full ros-dev-tools python3-pip python3-bloom python3-colcon-mixin python3-colcon-common-extensions python3-rosdep python3-vcstool

# Source the ROS Environment on every terminal launch
echo "source /opt/ros/${ROS_DISTRO}/setup.bash" >> ~/.bashrc
```

