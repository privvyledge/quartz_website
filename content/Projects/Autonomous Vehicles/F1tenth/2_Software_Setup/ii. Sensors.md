## RealSense D435i
### Upgrade/downgrade the firmware to version 5.15.1.55 (Support for more recent versions coming soon)
1. Download the zip file for [Version-5_15_1_55](https://www.intelrealsense.com/download/24238/?tmstv=1725546235) from the [firmware page](https://dev.intelrealsense.com/docs/firmware-releases-d400)
2. Unzip the file: `unzip d400_series_fw_5_15_1_55-1.zip` 
3. Navigate to the path of the bin file: `cd d400_series_fw_5_15_1_55-1`
4. Update by running the following command: `rs-fw-update -f Signed_Image_UVC_5_15_1_55.bin`
### Install the RealSense Drivers:
#### Install dependencies
1. Add Universe repository and update: `sudo apt-add-repository universe -y && sudo apt update`
2. Install dependencies: `sudo apt update && sudo apt-get install libssl-dev libusb-1.0-0-dev pkg-config -y`
3. Install graphics libraries: `sudo apt-get install libgtk-3-dev libglfw3-dev libgl1-mesa-dev libglu1-mesa-dev -y`
4. (optional) Install QTcreator: `sudo apt-get install qtcreator -y`
5. Install python3 for python support: `sudo apt-get install -y python3 python3-dev`
#### Build the Realsense Drivers
1. Select the librealsense version: `export LIBREALSENSE_BRANCH="v2.55.1"`
2. Export other variables for CMake visibility: `export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda/lib64:/usr/local/cuda/targets/aarch64-linux/lib/stubs:/opt/ros/${ROS_DISTRO}/install/lib && export CUDACXX=/usr/local/cuda/bin/nvcc && export PATH=${PATH}:/usr/local/cuda/bin`
3. Download the librealsense version: `mkdir ${HOME}/sdks && cd ${HOME}/sdks && git clone --branch ${LIBREALSENSE_BRANCH} --depth=1 https://github.com/IntelRealSense/librealsense`
4. Navigate to the downloaded: `cd librealsense`
5. Create and navigate to the build directory: `mkdir build && cd build`
6. Configure the build with CMake: `cmake -DBUILD_EXAMPLES=true -DFORCE_RSUSB_BACKEND=true -DBUILD_WITH_CUDA=true -DBUILD_WITH_OPENMP=true -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local  -DBUILD_PYTHON_BINDINGS:bool=true -DPYTHON_EXECUTABLE=/usr/bin/python3.10 -DBUILD_GRAPHICAL_EXAMPLES=true -DPYTHON_INSTALL_DIR=$(python3 -c 'import sys; print(f"/usr/lib/python{sys.version_info.major}.{sys.version_info.minor}/dist-packages")') -DPYTHON_LIBRARY=/usr/lib/$(uname -m)-linux-gnu -DPYTHON_INCLUDE_DIR=/usr/include/python3.10 ../`
7. Build librealsense: `make -j$(nproc)`
8. Install the built drivers system-wide: `sudo make install`
9. (optional) Depending on the version of librealsense downloaded, the python shared libraries are occasionally not correctly copied for system-wide use. This is a known issue with the pyrealsens2 installation and there over 30+ issues currently open [pyrealsense2 issues]([Issues · IntelRealSense/librealsense](https://github.com/IntelRealSense/librealsense/issues?q=is%3Aissue%20state%3Aopen%20pyrealsense2)). To address this, we can manually copy the shared libraries: `sudo cp Release/pyrealsense2.cpython-310-$(uname -m)-linux-gnu.so* /usr/local/lib/`
10. Setup UDEV rules so connected RealSense devices are found: `sudo apt install -y v4l-utils && cd ../ && sudo sh scripts/setup_udev_rules.sh`
11. (optional) Add the built python modules to the PYTHONPATH if multiple versions of python are installed: `export PYTHONPATH=${PYTHONPATH}:/usr/local/lib`
12. (optional) Delete the build folder to save space: `rm -rf ${HOME}/sdks/librealsense/build`


