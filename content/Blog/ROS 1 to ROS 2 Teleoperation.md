---
title: ROS 1 to ROS 2 Teleoperation
date: 2025-03-21
draft: false
tags: 
description:
---
Created: 2025-03-21
The corresponding code can be found [at my GitHub repository](https://github.com/privvyledge/ros1_to_ros2_communication)
## Installation:
1. Pull the ROS foxy to noetic bridge: `docker pull ros:foxy-ros1-bridge`
2. Pull the Melodic/noetic docker image:  `docker pull ros:noetic-ros-base-focal`
3. Check the noetic outputs in a Melodic container
4. Install docker compose: `sudo apt update && sudo apt-get install -y docker-compose-plugin`
## Usage:
### Option 1: Using Docker Compose
* Setup environment variables: ROS_MASTER, ROS_IP, START_ROSCORE, VEHICLE_NAME, USER.
* `docker compose up` or `docker-compose up` for older versions

### Option 2: Manually run each docker container
1. ROS 2 Container:
	1. Launch necessary topics
2. ROS 1 Containers
	1. `roscore`
	2. Subscribe to the topics
3. ROS Bridge Container (run only one)
	1. Bridge specific topics: `rosparam load /<path to yaml>/bridge.yaml && sr2 && ros2 run ros1_bridge parameter_bridge
	2. To bridge all topics (not recommended as it adds a lot of overhead): `ros2 run ros1_bridge dynamic_bridge --bridge-all-topics`

## Tips
### Starting ROSCore in Windows Subsystem for Linux
1. In WSL
	1. Get the IP address <wsl_ip_address>: `ip addr show eth0` # or `ifconfig`
	2. Set the ROS_MASTER_URI to the IP address: `export ROS_MASTER_URI=http://<wsl_ip_address>:11311`
	3. Set the ROS_HOSTNAME to the IP address of WSL: `export ROS_HOSTNAME=<wsl_ip_address>`
	4. Start the ROS master: `roscore`
2. In a powershell running as administrator:
	1. Enable Firewall passthrough rule by running the following in powershell as administrator. Either run both commands in method 1 or the command in method 2 but not necessarily both:
		1. Method 1:
			1. `Enable-NetFirewallRule -DisplayName 'Virtual Machine Monitoring (Echo Request - ICMPv4-In)'`
			2. `Enable-NetFirewallRule -DisplayName 'Virtual Machine Monitoring (Echo Request - ICMPv6-In)'
		2. Method 2:
			1. `New-NetFirewallRule -DisplayName "WSL" -Direction Inbound  -InterfaceAlias "vEthernet (WSL)"  -Action Allow`
	2. Forward connections from WSL to other computers since otherwise only the windows host will be able to communicate with WSL
		1. Automatically
			1. `netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=11311 connectaddress=(wsl hostname -I) connectport=11311
		2. Manually specifying IP addresses
			1. Get the windows IP address by copying after running <windows_ip_address>: `ipconfig`
			2. Obtain the WSL IP address as shown in 1.1 above <wsl_ip_address>
			3. Run the following to forward all WSL traffic. Replace <windows_ip_address> and <wsl_ip_address> with the appropriate values: `netsh interface portproxy add v4tov4 listenaddress=<windows_ip_address> listenport=11311 connectaddress=<wsl_ip_address> connectport=11311`
3. Connect to the ROS master from other computers by setting ROS_MASTER_URI to <windows_ip_address> not the WSL IP and ROS_HOSTNAME to the IP address of the connecting computer provided both computers can ping each other.
	1. `export ROS_MASTER_URI=http://<windows_ip_address>:11311`
	2. `export ROS_HOSTNAME=<connecting_computer_ip>`
	3. Tip: 
		1. In MATLAB, run the following: `rosinit('http://<windows_ip_address>:11311/')`