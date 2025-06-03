## Apps/Frameworks written in Python with ROS2 Wrappers. Interactive with sliders, checkboxes and text inputs (for now start with rqt_reconfigure).
### Computer Vision (GPU accelerated with support for batch processing using Kornia). Could render using ReRun, StreamLit but start with RViz/FoxGlove
**Dependecies**
* PyTorch
* Kornia
* (optional) Kornia-rs
* (optional) Kornia-moon
#### 1. GPU Accelerated Image Pre-Processing and Operations  using Kornia
* Undistortion
	* Image and points (https://kornia.readthedocs.io/en/latest/geometry.calibration.html#kornia.geometry.calibration.undistort_image)
* Resizing, anti-aliasing and upscaling
* ColorSpace conversion
* Feature detection, matching and tracking
* Homography and warping
* Stereo rectification
* Disparity->Depth->PointCloud (with optional filtering and smoothing)
* Image Alignment, e.g Depth<-->RGB
* Image Registration
* Image Stitching
* Video frame interpolation a.k.a high speed object deblurring using DeFMO
* Video Stabilization using optical flow
* DepthAnythingV2 or DepthPro. Test with PREDISS or TATMS
* FoundationStereo: Test with F1/10 then feed to RTABMap
#### 2. GPU Accelerated Visual Odometry/SLAM (Monocular, Stereo or RGB-D) with IMU preintegration or odometry
##### Features
##### Steps
1. Image Sequence: Get images for n (usually 2) frames and store. Optinally, also get timestamps/dt or use FPS .
2. Image correction: distortion removal, etc.
	1. Undistortion for image or points: https://kornia.readthedocs.io/en/latest/geometry.calibration.html#kornia.geometry.calibration.undistort_image
3. Feature Detection or Direct Method
4. (optional) Feature filtering using RANSAC
5. Feature Matching/Tracking
	1. Kornia image registration
	2. Metaresearch CoTracker: https://co-tracker.github.io/
	3. Optical Flow: 
		1. Pytorch: 
			1. [RAFT](https://docs.pytorch.org/vision/0.22/auto_examples/others/plot_optical_flow.html#sphx-glr-auto-examples-others-plot-optical-flow-py)
			2. [Multiple models](https://github.com/hmorimitsu/ptlflow). PTLFlow docs found [here](https://ptlflow.readthedocs.io/en/latest/starting/inference.html)
		2. 
6. (optional) Feature Tracking and prediction: Used to interpolate or in case feature matching fails
7. 3D PointCloud generation
8. Inlier Detection
	1. PnP: https://kornia.readthedocs.io/en/latest/geometry.calibration.html#kornia.geometry.calibration.solve_pnp_dlt
9. Motion Estimation
10. (optional) Refinement and Optimization
11. Add PointCloud to map

#### 3. Lines/Lane Segmentation with Object Segmentation/Detection and tracking. Test with PREDISS or TATMS
#### 4. Face Blurring for Privacy
#### 5. Camera/Sensor Calibration
#### 6. 2D Object Detection and Tracking without Ultralytics
#### 6. 3D Object Detection and tracking: Detect in 2D then project using depth or pointcloud

### PointClouds (GPU accelerated with support for batch processing using Open3D)
#### 1. PointCloud Pre-processing:
* PointCloud fusion: fuse multiple pointclouds into one
* Cropping
* Duplicate, NaN and Inf removal
* Voxel filtering and other filters
* Plane Segmentation
#### 2. 3D Detection and Tracking (Clustering or NN based) with optional projection to an Image. Todo: test with PREDISS or TATMS