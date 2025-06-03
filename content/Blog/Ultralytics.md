---
title: Ultralytics
date: 2025-05-18
draft: false
tags: 
description:
---
Created: 2025-05-18
## Performance Tips
### Export models to optimize for your hardware
The ultralytics package includes scripts for benchmarking the performance of models (YOLO, RTDETR, etc.) against available hardware. One way to speed up performance is to first run the benchmark then export the model to the best performing format.
1. Install dependencies: `python3 -m pip install ultralytics "onnx>=1.12.0,<1.18.0" "onnxslim>=0.1.53" "onnxruntime" "tensorflow>=2.0.0" ncnn "openvino>=2024.0.0" openvino-dev`
2. Run the benchmark script and feel free to specify a device: `yolo benchmark model=yolo11x.pt data='coco8.yaml' imgsz=640 half=true`. For example, on a server with an Intel Xeon CPU and and Nvidia RTX 5090 GPU, here is the output: "todo: put figure here."
3. Note the best performing format. Here are some recommendations based on optimizations from device manufacturers:
	1. Intel CPUs, GPUs, FPGAs, etc: OpenVINO with `half=true`
	2. Nvidia GPUs: TensorRT with `half=true`
	3. Apple Silicon: CoreML
	4. Generic: Onnx with `half=true` and PyTorch with `half=false`
#### Export to the desired format
* Intel (the model is exported to a folder named after the model, e.g "yolo11x_openvino_model"): 
	* Export: `yolo export model=yolo11x.pt format=openvino dynamic=true nms=true half=true imgsz=[640,480]` 
	* Test: `yolo detect predict model=yolo11x_openvino_model source='https://ultralytics.com/images/bus.jpg'`
* 
