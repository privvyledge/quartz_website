---
title: Build Open3D 0.19.x on Jetson Orin Nano with CUDA support
date: 2025-05-03
draft: false
tags:
  - f1tenth
  - f1/10
  - jetson
  - jetpack
  - robotics
  - autonomous-vehicles
  - cuda
  - nvidia
  - open3d
  - pointclouds
  - rgbd
description: Open3D 0.19.0 setup
---
Created: 2025-05-03

[[Open3D]] is an open-source library for PointCloud and RGB-D localization, mapping, odometry and other operations. Open3D has highly optimized and fast functions and applications for PointClouds and RGB-D data even with CUDA and as of version 0.19.0, support for various GPU architectures using SYCL. However, the prebuilt wheels for ARM devices do not include GPU support and there is currently no official documentation for building with CUDA support even though it is possible. When trying to apply the instructions for setting up CUDA support on x64 architectures on ARM devices, various errors occur. Therefore, this serves as detailed documentation for setting up Open3D with CUDA acceleration for ARM and other devices including tips and notes for speed ups as well as how the numerous errors are fixed.
## (optional) Upgrade CMake from 3.22.0 to >=3.24.0 for automatic CUDA device detection
1. Download the zipped folder for the desired cmake version: `wget https://cmake.org/files/v3.24/cmake-3.24.0-linux-aarch64.tar.gz`
2. Create the CMake install folder: `sudo mkdir -p /opt/cmake-3.24.0`
3. Unzip the downloaded file to the CMake install folder: `sudo tar --strip-components=1 -xzvf cmake-3.24.0-linux-aarch64.tar.gz   -C /opt/cmake-3.24.0`
4. Add the new CMake to PATH. Use ENV for Dockerfiles: `export PATH=/opt/cmake-3.24.0/bin:$PATH`
5. (optional) Add symbolic links to common Cmake paths
	1. `sudo ln -sf /opt/cmake-3.24.0/bin/cmake /usr/local/bin/cmake`
	2. `sudo ln -sf /opt/cmake-3.24.0/bin/ctest /usr/local/bin/ctest`
6. Remove the zipped folder: `rm cmake-3.24.0-linux-aarch64.tar.gz`
7. Confirm that CMake version is the same as the one just installed: `cmake --version`
## Setup the Open3D repository
**Note: due to CUDA compatibility issues on Ubuntu 22.04, we install from an alternative repository where this is fixed **
1. Move to the desired download directory, here we use the "sdks" folder in the users home directory: `mkdir -p ${HOME}/sdks && cd ${HOME}/sdks` 
2. Clone the repository from github: `git clone https://github.com/johnnynunez/Open3D.git`  # or `git clone -b 0.19.0-patch https://github.com/privvyledge/Open3D.git` 
3. Navigate to the cloned directory: `cd Open3D`
## (Optional: if upgrading CMake step was skipped) Modify the minimum CMake requirement in Open3D's CMakeLists.txt to point to 3.22
* `sed -i 's/cmake_minimum_required(VERSION 3\.24)/cmake_minimum_required(VERSION 3.22)/' CMakeLists.txt`
## Install Dependencies
### Necessary dependencies
1. `bash ./util/install_deps_ubuntu.sh assume-yes`
2. `sudo apt update && sudo apt install -y libglu1-mesa-dev libsdl2-dev python3-dev clang autoconf ninja-build libtool libusb-1.0-0-dev libc++abi-dev libtbb-dev xorg-dev libxcb-shm0 libc++-dev libudev-dev gfortran ccache libosmesa6-dev libxi-dev git`
3. (optional: if GUI support is needed): Note that this has been done in my Open3D repository but not in the upstream version. **This is for version [1.25.3 ](https://github.com/google/filament/releases/tag/v1.25.3)**
	1. Pull Filament v1.25.3: `wget https://github.com/google/filament/archive/refs/tags/v1.25.3.tar.gz`
	2. Get the checksum: `sha256sum v1.25.3.tar.gz | awk '{print $1}'`
	3. Replace with Open3D Filament archive in "<path_to_Open3D>/3rdparty/filament/filament_build.cmake"
	4. Repeat the above for the linux and windows releases and modify "<path_to_Open3D>/3rdparty/filament/filament_download.cmake"
4. (option: if GUI support is needed): Build Filament from source
	1. `git clone https://github.com/google/filament.git`
	2. `cd filament`
	3. `git checkout tags/v1.29.0 -b build_v1.29.0`
	4. `mkdir -p out/cmake-release && cd out/cmake-release`
	5. Install dependencies: `sudo apt update && sudo apt install -y vulkan-tools clang-14 libc++-dev libc++abi-dev     cmake ninja-build git wget unzip     libglu1-mesa-dev libxi-dev libxrandr-dev libxcursor-dev libxxf86vm-dev`
	6. `CC=/usr/bin/clang-14 CXX=/usr/bin/clang++-14 CXXFLAGS="-stdlib=libc++" cmake -G Ninja   -DCMAKE_BUILD_TYPE=Release   -DCMAKE_INSTALL_PREFIX=../release/filament   -DFILAMENT_SKIP_SAMPLES=ON -DFILAMENT_SUPPORTS_VULKAN=ON   ../..`
	7. `ninja`
	8. `ninja install`
	9. `sudo cmake --install out/cmake-release --prefix /usr/local`
### Optional but recommended dependencies.
**Note: if these dependencies are skipped then Open3D will build each of them from source which will increase the build time by a lot**
1. `sudo apt update && sudo apt install -y libeigen3-dev libglew-dev libtbb-dev libfmt-dev libjsoncpp-dev libjpeg-dev liblzf-dev libassimp-dev libblas-dev libqhull-dev libzmq3-dev  libvtk9-dev`
2. Export the following flags to inform Open3D's CMake do not rebuild installed packages: `export USE_SYSTEM_BUILDS="-DUSE_SYSTEM_VTK=ON -DUSE_SYSTEM_TBB=ON -DUSE_SYSTEM_EIGEN3=ON -DUSE_SYSTEM_GLEW=ON -DUSE_SYSTEM_TBB=ON -DUSE_SYSTEM_FMT=ON -DUSE_SYSTEM_JSONCPP=ON -DUSE_SYSTEM_JPEG=ON -DUSE_SYSTEM_LIBLZF=ON -DUSE_SYSTEM_ASSIMP=ON -DUSE_SYSTEM_BLAS=ON -DUSE_SYSTEM_QHULLCPP=ON -DUSE_SYSTEM_ZEROMQ=ON"`

## (optional) Build realsense with CUDA support and a desired version
**Note: if this step is skipped and realsense support is needed, Open3D will automatically build version 2.44.0 of the library which is quite outdated but fully functional.**
## Build Open3D
Navigate to the Open3D directory if not already there: `cd Open3D`
```bash
mkdir build \
    && cd build \
    && cmake  \
      -DBUILD_SHARED_LIBS=ON \
      -DBUILD_CUDA_MODULE=ON \
      -DCMAKE_CUDA_ARCHITECTURES=87  \
      -DBUILD_COMMON_CUDA_ARCHS=ON \
      -DCMAKE_BUILD_TYPE=Release \
      -DBUILD_GUI=OFF \
      -DBUILD_LIBREALSENSE=ON \
      -DUSE_SYSTEM_LIBREALSENSE=ON \
      -DBUILD_AZURE_KINECT=ON \
      -DBUILD_VTK_FROM_SOURCE=OFF \
      -DBUILD_COMMON_CUDA_ARCHS=ON \
      -DCMAKE_INSTALL_PREFIX=/usr/local \
      -DBUILD_PYTHON_MODULE=ON  \
      -DPython3_EXECUTABLE=/usr/bin/python3.10 ${USE_SYSTEM_BUILDS} \
      -DCMAKE_CXX_FLAGS="-Wl,--allow-shlib-undefined"  --no-warn-unused-cli \
      .. && \
 	make -j$(nproc) && \
    sudo apt remove python3-blinker -y && \
    make install-pip-package && sudo make install && \
    echo 'export LD_PRELOAD=/sdks/open3d_install/lib/libOpen3D.so' >> /etc/bash.bashrc
```
## Optional: Delete all items in the build folder except the bin directory to save space (about 3.0 GB)
* `find . -maxdepth 1 ! -name . ! -name bin -exec rm -rf {} +`

