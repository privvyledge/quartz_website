Links:
* [RealSense Calibration]([Overview](https://dev.intelrealsense.com/docs/calibration)) for Depth, IMU, etc.
* [Self-Calibration documentation](https://dev.intelrealsense.com/docs/self-calibration-for-depth-cameras)
* [Calibration flow/order]([Intel® RealSense™ Self-Calibration for D400 Series Depth Cameras](https://dev.intelrealsense.com/docs/self-calibration-for-depth-cameras#appendix-b-details-on-self-calibration-flow-and-c-api))
	* On-Chip Calibration
	* Focal Length Calibration
	* Ground Truth Calibration
	* Tare Calibration
* [Depth camera tuning documentation](https://dev.intelrealsense.com/docs/tuning-depth-cameras-for-best-performance)
* [Tuning tips]([Best Known Methods for Optimal Camera Performance over Lifetime – Intel® RealSense™ Depth and Tracking Cameras](https://www.intelrealsense.com/best-known-methods-for-optimal-camera-performance-over-lifetime/?_ga=2.97371103.1386622930.1747234790-18085914.1745184454#3.3))
* [IMU Calibration Tool]([librealsense/tools/rs-imu-calibration at development · IntelRealSense/librealsense · GitHub](https://github.com/IntelRealSense/librealsense/tree/development/tools/rs-imu-calibration#rs-imu-calibration-tool))
## Calibration
### Depth Image Calibration a.k.a On-Chip Calibration
1. Download the textured calibration pattern from the [Depth camera tuning documentation subsection]([Tuning depth cameras for best performance](https://dev.intelrealsense.com/docs/tuning-depth-cameras-for-best-performance#verify-performance-regularly-on-a-flat-wall-or-target)). [Here](https://www.intelrealsense.com/wp-content/uploads/2019/02/texture-pattern-set.zip?_ga=2.126280781.1386622930.1747234790-18085914.1745184454) is a direct link to download the zip file containing images of the textured calibration pattern at different resolutions.
### Depth Accuracy:
1. Download one of the calibration patterns based on paper size from [here]([Intel® RealSense™ Self-Calibration for D400 Series Depth Cameras](https://dev.intelrealsense.com/docs/self-calibration-for-depth-cameras#ground-truth-targets-and-printing--mounting-instructions)). Direct links for the following sizes are provided: [100x175_11x17-A3](https://www.intelrealsense.com/wp-content/uploads/2023/01/GaussianBlur30x100x175back-11x17-A3.pdf?_ga=2.92579805.1386622930.1747234790-18085914.1745184454), [100x175_8.5x11](https://www.intelrealsense.com/wp-content/uploads/2023/01/GaussianBlur30x100x175back-8.5x11.pdf?_ga=2.54428904.1386622930.1747234790-18085914.1745184454), [100x100_8.5x11](https://www.intelrealsense.com/wp-content/uploads/2023/01/GaussianBlur30x100x100back-8.5x11.pdf?_ga=2.54428904.1386622930.1747234790-18085914.1745184454).
### IMU Calibration
1. Download the PDF with instructions from [here]([IMU Calibration Tool for Intel® RealSense™ Depth Camera](https://dev.intelrealsense.com/docs/imu-calibration-tool-for-intel-realsense-depth-camera)). Direct pdf download [link](https://www.intelrealsense.com/download/18539/?_ga=2.95716444.1386622930.1747234790-18085914.1745184454).
### (optional: If the above fail) Stereo + Depth Calibration Using the Dynamic Calibration Tool