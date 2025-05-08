## Applications
### Sensors
### Realsense ([RealSense with Open3D - Open3D primary (unknown) documentation](https://www.open3d.org/docs/latest/tutorial/sensor/realsense.html))
**Note: this was tested with a RealSense D435i Camera but should work with other realsense devices.**
#### Record/Capture Live Data
##### List connected RealSense devices and their metadata:
* `<path_to_Open3D>/build/bin/examples/RealSenseRecorder -l`
##### Export the output folder as an environment variable
* `export OUTPUT_FOLDER=../dataset/realsense/`
##### Save a JSON file for the desired camera configuration, e.g copy the below to a file by running: `nano realsense_d435i.json`
```json
{
    "serial": "",
    "color_format": "RS2_FORMAT_RGB8",
    "color_resolution": "640,360",
    "depth_format": "RS2_FORMAT_Z16",
    "depth_resolution": "640,360",
    "fps": "60",
    "visual_preset": ""
 }
```
##### Record a bag by running the command below, press: 
###### Method 1: Use this option to pass configs to the realsense driver, such as width, height, fps, etc.
1. `<path_to_Open3D>/build/bin/examples/RealSenseRecorder --config realsense_d435i.json --align --record ${OUTPUT_FOLDER}/realsense.bag`. Optional: add the `--align` flag to align depth frames to RGB. This could lead to performance issues at higher FPS. Remove the flag if performance issues arise.
2. Press the SPACE key in the visualizer window to start/pause recording.
3. Press the ESC key after some time to save the recording and exit.
###### Method 2
**Note: This method automatically saves the bag to  ${OUTPUT_FOLDER}/realsense.bag**
1. `python3 <path_to_Open3D>/examples/python/reconstruction_system/sensors/realsense_recorder.py  --record_rosbag  --output_folder ${OUTPUT_FOLDER}`
2. Press the ESC (within the GUI window) or CTRL-C (within the terminal window) to stop
#### Replay Recorded Data
###### Method 1
* `<path_to_Open3D>/build/bin/examples/RealSenseBagReader --input ${OUTPUT_FOLDER}/realsense.bag`
###### Method 2
**Note: This method automatically reads the bag from  ${OUTPUT_FOLDER}/realsense.bag**
* `python3 <path_to_Open3D>/examples/python/reconstruction_system/sensors/realsense_recorder.py --playback_rosbag  --output_folder ${OUTPUT_FOLDER}`
#### Run the Reconstruction System
**There are two versions of the reconstruction system, the newer tensor based implementation and the legacy implementation. The tensor based run files can use either the new tensor based implementation or the legacy implementation. While both support GPU acceleration, the newer tensor based method is recommended for online (real-time) odometry and SLAM. Use the legacy version if the tensor-based implementation fails.**
**Tip: if Open3D was built with RealSense and GUI support, the sections below can be done with the following command: `<path_to_Open3D>/build/bin/examples/OnlineSLAMRealSense --align --record ${OUTPUT_FOLDER}/realsense.bag`  # pass in --help to see more options**
##### Record Dataset (if Bag was not recorded)
* `python3 examples/python/reconstruction_system/sensors/realsense_recorder.py  --record_imgs --output_folder ${OUTPUT_FOLDER}`  # could also replace `--record_imgs` with `--record_rosbag`
##### Save the configuration file for the reconstruction system
* `mkdir config`
###### For the New tensor-based reconstruction system
*  Open the following yaml file for editing: `nano config/realsense_reconstruction_t.yaml`
* Paste the lines below. Take note of the following:
	* Set "engine" to legacy for legacy support. Setting "multiprocessing" to true is only valid for the legacy engine.
	* Set "path_intrinsic" to a common intrinsic file. However, if the color and depth cameras have different intrinsic parameters, set "path_intrinsic" and "path_color_intrinsic" to the depth and color intrinsic values respectively.
	* "odometry_method" corresponds to one the following: point2plane, intensity, hybrid, frame2model. frame2model is only available for the tensor engine.
	* "icp_method" corresponds to one of colored, point2point, point2plane, generalized
	* "global_registration_method" corresponds to one of fgr, ransac
```yaml
name: Default reconstruction system config
fragment_size: 100
device: CUDA:0
engine: tensor
multiprocessing: false
path_dataset: '../dataset/realsense/'
depth_folder: depth
color_folder: color
path_intrinsic: '../dataset/realsense/camera_intrinsic.json'
path_color_intrinsic: ''
depth_min: 0.1
depth_max: 3.0
depth_scale: 1000.0
odometry_method: hybrid
odometry_loop_interval: 10
odometry_loop_weight: 0.1
odometry_distance_thr: 0.07
icp_method: colored
icp_voxelsize: 0.05
icp_distance_thr: 0.07
global_registration_method: ransac
registration_loop_weight: 0.1
integrate_color: true
voxel_size: 0.0058
trunc_voxel_multiplier: 8.0
block_count: 40000
est_point_count: 6000000
surface_weight_thr: 3.0
```
###### For the legacy reconstruction system
* Open the following json file for editing: `nano config/realsense_reconstruction.json`
* Paste the lines below and modify the values of "path_dataset" and "path_intrinsic" to match your values. Take note of the following:
	* Leave "path_intrinsic" empty or set to any invalid value to get the intrinsic data from the bagfile. Otherwise, set to the a valid realsense intrinsic path, e.g ${OUTPUT_FOLDER}/ as in the example below.
	* Set "path_dataset" to the either a ROSBAG file, e.g "${OUTPUT_FOLDER}/realsense.bag" (if `--record_rosbag` was used) or a directory, e.g ${OUTPUT_FOLDER} (if `--record_imgs` was used).
	* If "path_dataset" corresponds to a ROSBAG, set "python_multi_threading" to false. For "path_dataset", setting "python_multi_threading" to either true of false works. We set it as false here just to cover both cases.
```json
{
    "name": "Realsense bag file",
    "path_dataset": "../dataset/realsense/",
    "path_intrinsic": "../dataset/realsense/camera_intrinsic.json",
    "depth_max": 3.0,
    "voxel_size": 0.05,
    "depth_diff_max": 0.07,
    "preference_loop_closure_odometry": 0.1,
    "preference_loop_closure_registration": 5.0,
    "tsdf_cubic_size": 3.0,
    "icp_method": "color",
    "global_registration": "ransac",
    "python_multi_threading": true
}
```
##### Run the reconstruction system
###### Method1: Only works if open3d was built with gui and python support (all options listed are equivalent, but listed in decreasing order of recommendation)
1. Built C++ binary file (recommended): `<path_to_Open3D>/build/bin/examples/OnlineSLAMRGBD --align --dataset_path ${OUTPUT_FOLDER} --device CUDA:0 --intrinsic_path ${OUTPUT_FOLDER}/camera_intrinsic.json`
2. Python : `python <path_to_Open3D>/examples/python/reconstruction_system/dense_slam_gui.py --config config/realsense_reconstruction_t.yaml`
###### method2: works as long as open3d was built with python support
* New tensor-based reconstruction system:`python <path_to_Open3D>/examples/python/t_reconstruction_system/run_system.py --config config/realsense_reconstruction_t.yaml
* Legacy reconstruction system:`python <path_to_Open3D>/examples/python/reconstruction_system/run_system.py --config config/realsense_reconstruction.json --make --register --refine --integrate --device cuda:0`
Here is a breakdown of what the arguments mean:
* `--make`:                 Step 1) make fragments from RGBD sequence.
* `--register`:            Step 2) register all fragments to detect loop closure.
* `--refine`:             Step 3) refine rough registrations
* `--integrate`:           Step 4) integrate the whole RGBD sequence to make final mesh.
* `--slac`:                Step 5) (optional) run slac optimization for fragments.
* `--slac_integrate`:      Step 6) (optional) integrate fragments using slac to make final pointcloud / mesh.
* `--debug_mode`:          turn on debug mode.
* `--device DEVICE`:       (optional) select processing device for slac and slac_integrate. Example: cpu:0, cuda:0
* `--config CONFIG`:       path to the config file
