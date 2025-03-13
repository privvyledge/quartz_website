---
title: Software Setup
date: 2025-03-09
draft: false
tags:
  - autonomous-vehicles
  - jetson
  - f1/10
  - f1tenth
  - autodriver
  - jetpack
  - docker
  - sdkmanager
  - ros
  - ros2
description: F1/10 Autodriver software setup tutorial
---
Created: 2025-03-09

The rest of your content lives here. You can use **Markdown** here :)
## 1. Flash Jetpack (6.2)
**Note: The SDKManager Method is recommended for either SD cards or SSDs. However, the SD Card method can be faster or can be used if experiencing occasional bugs with the SDKManager.**
### SSD Method (or SDKManager Method)
### SD Card Method

## 2. Setup Software and Dependencies
### i. Quickstart
Use the convenience script from the F1/10 Autodriver repository:
1. Create a ROS2 workspace and navigate to the `src` directory: `mkdir -p ~/f1tenth_ws/src && cd ~/f1tenth_ws/src`
2. Clone the repository: `git clone https://github.com/privvyledge/autodriver.f1tenth.git`
3. Launch the script: `cd autodriver.f1tenth && chmod +x scripts/* && ./scripts/install_software_jetson.sh`
4. Add our user to the docker group to run without sudo: `sudo systemctl restart docker && sudo usermod -aG docker $USER && newgrp docker`
### ii. Manual Setup
Alternatively, run the desired commands listed below
#### a. Setup SSH
##### Install SSH server (should be installed by default on most version)
`sudo apt-get update && sudo apt-get install openssh-server`
##### Connect to the jetson via SSH
1. Connect the Jetson to a monitor
2. Find the IP address of the Jetson by running: `ifconfig`
3. Connect to the Jetson from a remote computer on the same network by running: `ssh <username>@<ip_address>` and replace username and password with that of the Jetson.
#### b. Install common dependencies
`sudo apt update && sudo apt install -y python3-dev python3-pip python3-setuptools python3-venv build-essential git curl nano cmake
#####  Install VCS tool (if not installing ROS on the host)
```bash
curl -s https://packagecloud.io/install/repositories/dirk-thomas/vcstool/script.deb.sh | sudo bash
sudo apt-get update
sudo apt-get install python3-vcstool
```
#### c. Install JTop
1. sudo pip3 install -U jetson-stats
2. sudo reboot
#### d. Install Docker
**Note: this step is not necessary if Jetpack was installed using the SD Card Method.**
##### For Jetpack 6.2 (SSD or installed via SDK Manager)
###### Installation
1. `sudo apt update && sudo apt install -y nvidia-container curl`
2. `sudo apt update && wget https://get.docker.com -O docker_install.sh && chmod +x docker_install.sh && sudo ./docker_install.sh`
**Note: Due to a current issue with Docker 28.0.0 and Jetpack 6.x, older versions of Docker must be installed. Big thanks to Jetsonhacks (https://github.com/jetsonhacks/install-docker and https://jetsonhacks.com/2025/02/24/docker-setup-on-jetpack-6-jetson-orin/)**
```bash
sudo apt-get install -y docker-ce=5:27.5.1-1~ubuntu.22.04~jammy --allow-downgrades
sudo apt-get install -y docker-ce-cli=5:27.5.1-1~ubuntu.22.04~jammy --allow-downgrades
sudo apt-get install -y docker-compose-plugin=2.32.4-1~ubuntu.22.04~jammy --allow-downgrades
sudo apt-get install -y docker-buildx-plugin=0.20.0-1~ubuntu.22.04~jammy --allow-downgrades
sudo apt-get install -y docker-ce-rootless-extras=5:27.5.1-1~ubuntu.22.04~jammy --allow-downgrades
```

```bash
sudo apt-mark hold docker-ce=5:27.5.1-1~ubuntu.22.04~jammy && sudo apt-mark hold docker-ce-cli=5:27.5.1-1~ubuntu.22.04~jammy && sudo apt-mark hold docker-compose-plugin=2.32.4-1~ubuntu.22.04~jammy && sudo apt-mark hold docker-buildx-plugin=0.20.0-1~ubuntu.22.04~jammy && sudo apt-mark hold docker-ce-rootless-extras=5:27.5.1-1~ubuntu.22.04~jammy
```

`rm docker_install.sh && sudo systemctl --now enable docker && sudo nvidia-ctk cdi generate --mode=csv --output=/etc/cdi/nvidia.yaml && sudo nvidia-ctk runtime configure --runtime=docker --cdi.enabled=true && sudo systemctl restart docker && sudo apt update && sudo apt-get install -y pva-allow-2`
#### Add our user to the docker group to use without sudo
1. `sudo systemctl restart docker`
2. `sudo usermod -aG docker $USER`
3. `newgrp docker`
#### Set Nvidia as the default runtime to automatically build and run images/containers with GPU support
```bash
sudo apt install -y jq && sudo jq '. + {"default-runtime": "nvidia"}' /etc/docker/daemon.json | \
  sudo tee /etc/docker/daemon.json.tmp && \
  sudo mv /etc/docker/daemon.json.tmp /etc/docker/daemon.json && sudo systemctl daemon-reload && sudo systemctl restart docker 
```

#### e. Install Nvidia-Jetpack and other Nvidia Packages if not installed via SDK Manager or SDCard method
`sudo apt update && sudo apt install -y nvidia-jetpack`
#### f. Increase Power Settings to MAXN Super (Fastest)
1. `sudo nvpmodel -m 2`
2. `sudo reboot`
#### g. Mount SWAP
When running multiple nodes, mounting swap is recommended for extra memory. If an SSD is available, it should be used as most consumer grade SD Cards have limited read/write cycles and swap could degrade the SD card faster.
```bash
sudo systemctl disable nvzramconfig
sudo fallocate -l 16G /mnt/16GB.swap  
sudo chmod 600 /mnt/16GB.swap  
sudo mkswap /mnt/16GB.swap  
sudo swapon /mnt/16GB.swap  
sudo cp /etc/fstab /etc/fstab.bak  
echo '/mnt/16GB.swap none swap sw 0 0' | sudo tee -a /etc/fstab  
echo "Allocated 16GB Swap memory."
```

#### h. Disable Misc Services
`sudo systemctl disable nvargus-daemon.service`
#### i. Disable Desktop GUI
If using via SSH, disable the Desktop GUI to free up some RAM (about 800 MB for GNOME/Unity)
On boot (permanent) `sudo systemctl set-default multi-user.target` or temporarily (will be reenabled upon restart) `sudo init 3`
To reenable later
`sudo systemctl set-default graphical.target`  or `sudo init 5` to temporarily enable
#### j. Add joystick support
`sudo apt-get install -y joystick jstest-gtk`
#### k. Setup UDEV rules to allocate namespaces for USB devices to avoid conflicts when starting up USB devices, e.g LIDAR, VESC, RealSense
##### YDLIDAR
```bash
cd /tmp && git clone https://github.com/YDLIDAR/ydlidar_ros2_driver.git -b humble
chmod 0777 ydlidar_ros2_driver/startup/*
sudo sh ydlidar_ros2_driver/startup/initenv.sh
```
##### Realsense
```bash
sudo apt install -y v4l-utils
cd /tmp && git clone https://github.com/IntelRealSense/librealsense.git && cd librealsense
sudo sh scripts/setup_udev_rules.sh
```

##### VESC
```bash
echo 'KERNEL=="ttyACM[0-9]*", ACTION=="add", ATTRS{idVendor}=="0483", ATTRS{idProduct}=="5740", MODE="0666", GROUP="dialout", SYMLINK+="sensors/vesc"' | sudo tee -a /etc/udev/rules.d/99-vesc.rules # echo 'KERNEL=="ttyACM[0-9]*", ACTION=="add", ATTRS{idVendor}=="0483", ATTRS{idProduct}=="5740", MODE="0666", GROUP="dialout", SYMLINK+="sensors/vesc"' > /tmp/99-vesc.rules
sudo mv /tmp/99-vesc.rules /etc/udev/rules.d/99-vesc.rules
```
##### Logitech f710 Joystick
```bash
echo 'KERNEL=="js[0-9]*", ACTION=="add", ATTRS{idVendor}=="046d", ATTRS{idProduct}=="c219", SYMLINK+="input/joypad-f710"' | sudo tee -a /etc/udev/rules.d/99-joypad-f710.rules # echo 'KERNEL=="js[0-9]*", ACTION=="add", ATTRS{idVendor}=="046d", ATTRS{idProduct}=="c219", SYMLINK+="input/joypad-f710"' > /etc/udev/rules.d/99-joypad-f710.rules
sudo mv /tmp/99-joypad-f710.rules /etc/udev/rules.d/99-joypad-f710.rules
```

`sudo udevadm control --reload-rules && sudo udevadm trigger`

#### l. Reboot
`sudo reboot`
#### m. (optional) Enable fan on boot using JTop
1. `jtop`
2. Press "6" to go to CTRL or click
3. In "Profiles", click "cool"
4. (optional) 
5. Press "s| enable"
6. 2. Press "e" to enable jetson clocks on boot or click "on boot"
#### n. Increase default settings for ROS DDS
## 3. Setup F1/10 Workspace for Autonomous Driving
It is recommended to pull the latest pre-built docker image. 
### Method 1: Using Pre-Built Docker Image (About 10 minutes depending on internet strength)

### Method 2: Building and modifying docker image manually (About 3 Hours)
#### Automated setup
It is recommend to use the script included in this repository by running the following: 
1. `cd autodriver.f1tenth/scripts && ./build_f1tenth_docker.sh`
2. Deploy the built image:
	1. Login to docker: `docker login -u <USERNAME>`
	2. Run the following command ``
```bash
${ISAAC_ROS_WS}/src/isaac_ros_common/scripts/docker_deploy.sh \
   --base_image_key "aarch64.ros2_humble.realsense.f1tenth" \
   --ros_ws /workspaces/isaac_ros-dev/ros_ws  \
   --name "privvyledge/f1tenth_main"
```

Otherwise run the commands below for manual setup
1. Install git lfs: `sudo apt-get install git-lfs`
## 4. Modify the OS for efficient ROS communication
1. CYCLONE
## 5. Teleoperate
1. Add robot name as an environment variable `echo "export VEHICLE_NAME=<INSERT_NAME>" >> ~/.bashrc && source ~/.bashrc`. Replace <INSERT_NAME> with the desired robot name.
2. Export the motor control type: `echo "export MOTOR_CONTROL_TYPE=0000" >> ~/.bashrc && source ~/.bashrc`. 'VESC Knockoff' and 'Velineon VXL-3S' do not have an IMU onboard. Non-VESC SDKs paired with e.g the Velineon VXL-3S only provides RPS/RPM/ERPM/PWM/DutyCycle control but no current control or feedback.
	1. 0000 = ESC (VESC Original) + Motor + SDK (VESC)
	2. 0001 = ESC (VESC Knockoff) + Motor + SDK (VESC)
	3. 0002 = ESC (Velineon VXL-3S) + Motor + SDK (Arduino + Pyserial)
	4. 0003 = ESC (VESC Original) + Motor + SDK (Arduino + Pyserial)
	5.  0004 = ESC (VESC Knockoff) + Motor + SDK (Arduino + Pyserial)
	6.  0005 = ESC (SparkMax ESC) + Motor + SDK (pySparkMax)
	7.  0006 = ESC (SparkMax ESCl) + Motor + SDK (Arduino + Pyserial)
3. Connect the Joystick to the Jetson
## 6. (optional) Visualization
Due to the limited CPU and RAM on the Jetson Orin Nano's, it is recommended to visualize the outputs on a remote computer. However, the following can still be done on the Jetson but will slow down the overall performance of nodes
### FoxGlove Setup
1. Download Foxglove Studio from https://foxglove.dev/download on the remote/visualization computer and launch it then Create a free account or Sign in.
2. 

