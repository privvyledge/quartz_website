---
title: How to Read Velodyne PCAP Packets in ROS2
date: 2025-05-27
draft: false
tags:
  - ros2
  - autonomous-vehicles
  - object-detection
  - open3d
  - pointclouds
  - lidar
  - velodyne
  - intelligent-transportation-systems
  - smart-city
  - intelligent-traffic
description: Tutorial on Setting Up Velodyne LIDAR with ROS2
---
Created: 2025-05-27

For the past couple of years (from 2023), I have been working on the US Department of Transportations [Intersection Safety Challenge](https://www.its.dot.gov/research-aread/Intersection-Safety-Challenge) where our team at the FAMU-FSU College of Engineering's Resilience and Autonomous System (RASLAB) group have advanced in all stages ([[US Department of Transportations Intersection Safety Challenge]]) with our [[Predictive Intersection Safety System (PREDISS)]]. We wrote all the code from scratch in python with calls to C extensions for real-time speed without ROS. However, I am working on implementing our pipeline with ROS nodes for modularity and more flexibility. To that end, I would like to start by extracting the PointClouds obtained from Velodyne LIDARs in PCAP format and publish them to the ROS network. For this tutorial, we will be using ROS2 humble. Later, we will setup PointCloud object detection nodes, publish images (from videos, cameras and even IP cameras), detect objects in the images, perform sensor fusion, predict trajectories and collisions. 

Without further ado, here are the steps needed to read PCAP files and publish corresponding PointClouds in ROS2:

## 1. Obtain PCAP files
The first step is to download LIDAR PCAP files. For the purpose of this tutorial and project, we will download the Sample 1 dataset from the [[US Department of Transportations Intersection Safety Challenge]] found [here]([Intersection Safety Challenge Stage 1B Sample | Department of Transportation - Data Portal](https://data.transportation.gov/Roadways-and-Bridges/Intersection-Safety-Challenge-Stage-1B-Sample/vq7s-mv3v/about_data)). However, any PCAP file generated from Velodyne LIDARs should suffice.
## 2. Install ROS 2 Humble
Follow the instructions detailed in [[ROS2 Installation]].

## Publish the PCAP PointClouds as a stream of ROS PointCloud2 messages.
### Method 1: Using my custom node (Recommended)

### Method 2: Using the Velodyne ROS 2 Driver
#### 1. Install the Velodyne ROS 2 Driver
You can either install the Velodyne packages and dependencies from prebuilt binaries or  build from sources for more advanced/customizable use cases. For most users, the binary installation is faster and recommended.
##### Option 1: Binary Installation (Recommended)
1. Install the latest binaries: `sudo apt update && sudo apt install -y ros-${ROS_DISTRO}-velodyne`
2. Source the default ROS 2 workspace to reflect the latest download: `source /opt/ros/${ROS_DISTRO}/setup.bash`
##### Option 2: Source Build Installation
**Skip this section if already installed from binaries.**
1. Add a variable for the ros2 workspace. For the purpose of this tutorial, we will assume this is the `~/ros2_ws` directory: `export ROS2_WS=${HOME}/ros2_ws`
2. Navigate to your ros2 workspace: `mkdir -p ${ROS2_WS}/src && cd ${ROS2_WS}/src`
3. Download the Velodyne ROS 2 Driver and related packages: `git clone https://github.com/ros-drivers/velodyne -b humble-devel`
4. Install dependencies: `cd ${ROS2_WS} && source /opt/ros/${ROS_DISTRO}/setup.bash &&  sudo apt update && sudo apt install ros-${ROS_DISTRO}-diagnostics -y && && rosdep update && rosdep install --from-paths src --ignore-src -r -y -q`
5. Build the Velodyne driver: `colcon build --symlink-install --cmake-args ' -DCMAKE_BUILD_TYPE=Release'`
6. Source the ROS 2 workspace: `source ${ROS2_WS}/install/setup.bash`
#### 2. Publish the PointCloud from the PCAP file
First, the raw packets need to be streamed either from a live Velodyne LIDAR or a PCAP file using the 'velodyne_driver' node. Then the packets need to be converted to a ROS PointCloud2 and optionally a LaserScan message with the 'velodyne_pointcloud' and 'velodyne_laserscan' nodes respectively. To accomplish this, we can either use my provided launch file to accomplish this or run both nodes on separate terminals. I recommend using my provided launch file as it is more configurable and can be further included in other launch files. Regardless of the method chosen, make sure to use parameters/files for your specific LIDAR device. For the [[US Department of Transportations Intersection Safety Challenge]], the Velodyne LIDAR used is the Velodyne VLP32C a.k.a Velodyne Ultra Puck (VLP-32C).

* (optional) Export the namespace/frame_id for each lidar, e.g: `export LIDAR_NS=velodyne_front`
* Replace <path_to_pcap_file> with the path to the file, usually ending with '.pcap'.
* See the official GitHub documentation for:
	* [Velodyne Driver node](https://github.com/ros-drivers/velodyne/tree/humble-devel/velodyne_driver)
	* [Velodyne PointCloud node](https://github.com/ros-drivers/velodyne/tree/humble-devel/velodyne_pointcloud)
	* [Velodyne LaserScan node](https://github.com/ros-drivers/velodyne/tree/humble-devel/velodyne_laserscan)
##### Option 1: Using the provided combined launch file
**Todo: set this up**
* Run all nodes as composable nodes in a shared container: `ros2 launch velodyne velodyne-all-nodes-VLP32C-composed-launch.py`
##### Option 2: Launching both nodes separately
1. Start the velodyne driver: `ros2 run velodyne_driver velodyne_driver_node --ros-args -r __ns:=/${LIDAR_NS} -p frame_id:=${LIDAR_NS} -p model:='32C' -p pcap:=<path_to_pcap_file> -p enabled:=true -p read_once:=true -p repeat_delay:=0.0 -p read_fast:=false -p time_offset:=0.0 -p gps_time:=false -p rpm:=600.0 -p timestamp_first_packet:=false  # -p device_ip:='192.168.1.201' -p port:=2368`
2. Start the pointcloud converter node (use the velodyne_transform_node to transform the PointCloud to another frame or velodyne_convert_node to just publish the PointCloud in the default frame): `ros2 run velodyne_pointcloud velodyne_transform_node --ros-args -r __ns:=/${LIDAR_NS} -p model:='32C' -p min_range:=0.9 -p max_range:=200.0 -p view_direction:=0.0 -p organize_cloud:=true -p calibration:='/opt/ros/humble/share/velodyne_pointcloud/params/VeloView-VLP-32C.yaml'  # -p target_frame:=${LIDAR_NS} -p fixed_frame:=base_link` 
3. (optional) Start the laserscan node: `ros2 run velodyne_laserscan velodyne_laserscan_node --ros-args -r __ns:=/${LIDAR_NS} -p ring:=-1 -p resolution:=0.007`

## 5. Visualize the published PointCloud
### RViz view of the "Run_2" PointCloud of "LIDAR_1" from the "Sample_1" dataset running at 10Hz.
![[sample1_run2_lidar1_pointcloud.gif]]### RViz view of the corresponding LaserScan from the same LIDAR PointCloud above.
![[sample1_run2_lidar1_laserscan.gif]]# 6. Velodyne Node Graph
![[velodyne_rosgraph.png]]
