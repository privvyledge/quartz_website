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
4. (optional: if SDKManager method was used) Install Docker: `./scripts/install_nvidia_docker.sh`
5. Add our user to the docker group to run without sudo: `sudo systemctl restart docker && sudo usermod -aG docker $USER && newgrp docker`
6. Reboot to enable JTOP: `sudo reboot`
7. Modify the OS for efficient ROS communication: See [[iii. Remote PC Communication]] for details.
8. (optional) Setup RustDesk for remote communication and visualization. See [[RustDesk Setup on Jetson Orin Nano]] for details.
9. Setup the ROS Workspace
	1. Development mode
		1. `./scripts/build_f1tenth_docker.sh`
	2. Production
		1. Pull and run the latest container: `xhost +local:root && jetson-containers run -v /dev:/dev -v ${HOME}/shared_dir:/mnt/shared_dir -v ${HOME}/data:/mnt/data --ipc host --env="QT_X11_NO_MITSHM=1" --privileged privvyledge/f1tenth:humble-latest /bin/bash && xhost -local:root`
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

#### e. (optional: if using SD Card): Mount SSD and migrate docker to SSD
1. Unplug the Jetson from the power supply.
2. Install the M.2 SSD and secure with the screw.
3. Connect the power supply and boot up the Jetson.
4. Verify that the system recognizes the SSD and memory controller by running: `lspci`
5. Find the device name by running: `lsblk`
6. Format the SSD for compatibility with the Jetson. Assuming your SSD name from the above is 'nvme0n1': `sudo mkfs.ext4 /dev/nvme0n1`
7. Create a mount point for the SSD: `sudo mkdir -p /mnt/f1tenth_ssd`
8. Mount the SSD: `sudo mount /dev/nvme0n1 /mnt/f1tenth_ssd`
9. Setup automatic mounting on each boot:
	1. Find the SSD UUID by running: `lsblk -f`
	2. Add a new entry to the FSTAB file: `sudo apt update && sudo apt install -y nano && sudo nano /etc/fstab`
	3. Insert the following line to the fstab file replacing <DEVICE_UUID> with the UUID identified when runnin `lsblk -f`: `UUID=<DEVICE_UUID> /mnt/f1tenth_ssd/ ext4 defaults 0 2`
	4. Change the ownership of the SSD to the user instead of root: `sudo chown ${USER}:${USER} /mnt/f1tenth_ssd`
10. Migrate docker to the SSD
	1. Move the existing docker folder from the SD Card to the SSD: `sudo du -csh /var/lib/docker/ && sudo mkdir /mnt/f1tenth_ssd/docker && sudo rsync -axPS /var/lib/docker/ /mnt/f1tenth_ssd/docker/ && sudo du -csh  /mnt/f1tenth_ssd/docker/`
	2. Edit the docker configuration using a text editor, e.g nano or gedit: `sudo nano /etc/docker/daemon.json`
	   Insert "data-root" like below:
> 		{
> 		    "runtimes": {
> 		        "nvidia": {
> 		            "path": "nvidia-container-runtime",
> 		            "runtimeArgs": []
> 		        }
> 		    },
> 		    "default-runtime": "nvidia",
> 		    "data-root": "/mnt/f1tenth_ssd/docker"
> 		}
11. Rename the old docker directory: `sudo mv /var/lib/docker /var/lib/docker.old`
12. Restart the docker daemon: `sudo systemctl daemon-reload && sudo systemctl restart docker && sudo journalctl -u docker`

#### e. Install Nvidia-Jetpack and other Nvidia Packages if not installed via SDK Manager or SDCard method
`sudo apt update && sudo apt install -y nvidia-jetpack`
#### f. Increase Power Settings to MAXN Super (Fastest)
1. `sudo nvpmodel -m 2`
2. `sudo reboot`
#### g. Mount SWAP
**NOTE: IF using the SD Card + SSD setup, mount SWAP on the SSD since the rapid continuous read/write updates by RAM will degrade the lifespan of the SD card. Replace '/mnt/16GB.swap' below with '/mnt/f1tenth_ssd/16GB.swap'**
When running multiple nodes, mounting swap is recommended for extra memory. If an SSD is available, it should be used as most consumer grade SD Cards have limited read/write cycles and swap could degrade the SD card faster.
```bash
export SWAP_DIRECTORY=/mnt/16GB.swap  # /mnt/f1tenth_ssd/16GB.swap for SD Card + SSD setup
sudo systemctl disable nvzramconfig
sudo fallocate -l 16G ${SWAP_DIRECTORY} 
sudo chmod 600 ${SWAP_DIRECTORY}
sudo mkswap ${SWAP_DIRECTORY}  
sudo swapon ${SWAP_DIRECTORY} 
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

#### o. (optional) Install Jetson Containers: 
`cd ~/Downloads && git clone https://github.com/dusty-nv/jetson-containers && bash jetson-containers/install.sh`
#### p. (optional: if using Logitech F710 controller)
The default kernel of the Jetson Orin Nano Jetpack 6.x does not have support for the Logitech F710 controller and other XBOX like controllers. The most common solution is to rebuild the kernel with support for them ([detailed kernel customization tutorial](https://github.com/muellerbernd/l4t), [forum link](https://forums.developer.nvidia.com/t/jetpack-6-0-issue-with-xbox-gamepad/276392), [Jetson troubleshooting](https://elinux.org/Jetson/L4T/r36.3.x_patches)). However, a more straightforward solution is to install the Xpad package ([xpad solution using dkms](https://github.com/jetsonhacks/logitech-f710-module/issues/7), [xpad solution without dkms](https://github.com/woawo1213/jetpack6-joy)). Follow the instructions below: 
1. Install dkms: `sudo apt-get install dkms`
2. Remove any previous xpad installation: `sudo dkms remove -m xpad -v 0.4 --all` Proceed if 'Error! The module/version combo: xpad-0.4 is not located in the DKMS tree.' is encountered which means no xpad was not installed.
3. Install xpad 0.4: `sudo git clone https://github.com/paroj/xpad.git /usr/src/xpad-0.4 && sudo dkms install -m xpad -v 0.4`
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
See [[iii. Remote PC Communication]] for details.
## 5. (optional) Setup RustDesk for remote communication and visualization
See [[RustDesk Setup on Jetson Orin Nano]] for details.
## 5. Teleoperate
1. Add robot name as an environment variable `echo "export VEHICLE_NAME=<INSERT_NAME>" >> ~/.bashrc && source ~/.bashrc`. Replace <INSERT_NAME> with the desired robot name.
2. Export the motor control type: `echo "export MOTOR_CONTROL_TYPE=0000" >> ~/.bashrc && source ~/.bashrc`. 'VESC Knockoff' and 'Velineon VXL-3S' do not have an IMU onboard (so disable vehicle IMU filter for all but 0000 and 0003). Non-VESC SDKs paired with e.g the Velineon VXL-3S only provides RPS/RPM/ERPM/PWM/DutyCycle control but no current control or feedback.
	1. 0000 = ESC (VESC Original) + Motor + SDK (VESC)
	2. 0001 = ESC (VESC Knockoff) + Motor + SDK (VESC)
	3. 0002 = ESC (Velineon VXL-3S) + Motor + SDK (Arduino + Pyserial)
	4. 0003 = ESC (VESC Original) + Motor + SDK (Arduino + Pyserial)
	5.  0004 = ESC (VESC Knockoff) + Motor + SDK (Arduino + Pyserial)
	6.  0005 = ESC (SparkMax ESC) + Motor + SDK (pySparkMax)
	7.  0006 = ESC (SparkMax ESCl) + Motor + SDK (Arduino + Pyserial)
3. Connect the Joystick to the Jetson
	1. If using a Logitech F710, switch the toggle at the top to 'X' instead of 'D'
4. **NOTE: if running on an x86 PC, change the owner of the workspace directory to enable building and avoiding permission errors by running `sudo chown -R ${USERNAME} /workspaces`**
## 6. (optional) Visualization
Due to the limited CPU and RAM on the Jetson Orin Nano's, it is recommended to visualize the outputs on a remote computer. However, the following can still be done on the Jetson but will slow down the overall performance of nodes
### FoxGlove Setup
1. Download Foxglove Studio from https://foxglove.dev/download on the remote/visualization computer and launch it then Create a free account or Sign in.
2. 

