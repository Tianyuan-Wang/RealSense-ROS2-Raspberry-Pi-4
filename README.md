# RealSense-ROS2-Raspberry-Pi-4
A step-by-step instruction on how to set up RealSense ROS2 node on Raspberry Pi 4

## Prerequisites

* Ubuntu Server 22.04 LTS: Use [rpi-imager](https://www.raspberrypi.com/software/) to install Ubuntu Server 22.04 LTS 64bit version to the Pi4.

* ROS2 Humble: Follow the [official instructions](https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debians.html) to install ROS2 on the Pi4.

* Dependencies: Install necessary packages based on the instructions given by the [official documentation](https://github.com/IntelRealSense/librealsense/blob/development/doc/installation.md#install-dependencies)

		sudo apt-get install \
			libssl-dev libusb-1.0-0-dev libudev-dev pkg-config libgtk-3-dev \
			git wget cmake build-essential \
			libglfw3-dev libgl1-mesa-dev libglu1-mesa-dev at

* RealSense Camera Firmware version: It is recommended to use the realsense-viewer on a x86 computer to upgrade or downgrade the D435i camera's firmware to version 5.15.0.2 as it is better compatible with the SDK version 2.54.1.

* Disable Pi4's embedded camera devices:

		sudo modprobe -r bcm2835_codec
		sudo modprobe -r bcm2835_isp

## RealSense SDK

In the home folder, download the RealSense SDK:

	cd ~ && git clone -b v2.54.1 https://github.com/IntelRealSense/librealsense.git

Inside the RealSense SDK repository directory, unplug the camera and set up udev rules:

	cd ~/librealsense && ./scripts/setup_udev_rules.sh

Create the build folder:

	mkdir build && cd build

Configure the SDK source code:

	cmake \
    -DBUILD_GRAPHICAL_EXAMPLES=false \
    -DBUILD_PYTHON_BINDINGS=true \
    -DPYTHON_EXECUTABLE=/usr/bin/python3 \
    -DFORCE_RSUSB_BACKEND=true \
    -DCMAKE_BUILD_TYPE=Release ../

Compile and install:

	make -j4 all && sudo make install

## RealSense ROS2

In the home folder, create a ros2 workspace:

	cd ~ && mkdir -p ros2_ws/src && cd ros2_ws/src

Download the RealSense ROS repository:

	git clone https://github.com/IntelRealSense/realsense-ros.git && cd ..

Build the ROS2 package:

	colcon build

## Run the node

Source corresponding environments:

	source ~/ros2_ws/install/setup.bash

Run the node with an acceptable resolution (acceptable for the Pi):

	ros2 launch realsense2_camera rs_launch.py \
	depth_module.profile:=424x240x6 \
	rgb_camera.profile:=424x240x6 \
	pointcloud.enable:=true

## Visualise the point cloud

On the host computer with ROS2 installed, open RVIZ2:

	ros2 run rviz2 rviz2

On the left panel, set the `Fixed Frame` to `camera_link` and add the topic `/camera/camera/depth/color/points`.

## Tips

One can try with higher resolution but it might cause the node to freeze.

There will be a lot of warnings like `(messenger-libusb.cpp:42) control_transfer returned error, index: 768, error: Resource temporarily unavailable, number: 11` even when the node is working.

The CMake flag 'FORCE_RSUSB_BACKEND' must be enabled or the ROS node will report error.