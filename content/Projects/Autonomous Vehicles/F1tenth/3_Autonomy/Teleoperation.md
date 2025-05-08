1. Disable the desktop: `sudo init 3`
2. Disable RustDesk to reduce CPU utilization: `sudo systemctl disable rustdesk`
3. Start the docker containers: `jetson-containers run -v /dev:/dev --ipc host --env="QT_X11_NO_MITSHM=1" --env VEHICLE_NAME=$VEHICLE_NAME --env USER=$VEHICLE_NAME --env ROS_LOCALHOST_ONLY=0 --env ROS_DOMAIN_ID=0 --env RMW_IMPLEMENTATION=rmw_fastrtps_cpp --env NAMESPACE=${NAMESPACE:-/$VEHICLE_NAME} -v ${HOME}/bolus_ws/ros1_to_ros2_communication/topics_to_bridge.yaml:/topics_to_bridge.yaml:ro --privileged privvyledge/f1tenth:humble-latest /bin/bash -c "ros2 launch f1tenth_launch teleop.launch.py launch_localization:=False launch_vehicle:=True launch_sensors:=False use_f1tenth_namespace:=True"`
4. Run whatever test
5. Enab