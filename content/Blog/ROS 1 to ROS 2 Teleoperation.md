The corresponding code can be found [at my GitHub repository](https://github.com/privvyledge/ros1_to_ros2_communication)
## Installation:
1. Pull the ROS foxy to noetic bridge: `docker pull ros:foxy-ros1-bridge`
2. Pull the Melodic/noetic docker image:  `docker pull ros:noetic-ros-base-focal`
3. Check the noetic outputs in a Melodic container
4. Install docker compose: `sudo apt update && sudo apt-get install -y docker-compose-plugin`
## Usage:
### Option 1: Using Docker Compose


### Option 2: Manually run each docker container
1. ROS 2 Container:
	1. Launch necessary topics
2. ROS 1 Containers
	1. `roscore`
	2. Subscribe to the topics
3. ROS Bridge Container (run only one)
	1. Bridge specific topics: `rosparam load /<path to yaml>/bridge.yaml && sr2 && ros2 run ros1_bridge parameter_bridge
	2. To bridge all topics (not recommended as it adds a lot of overhead): `ros2 run ros1_bridge dynamic_bridge --bridge-all-topics`
