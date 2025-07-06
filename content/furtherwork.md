+++
weight = 5
title = "5_Further-Work" 
date = "2025-07-01"
+++

# Autopilot Integration

An autopilot is a system that enables the drone to fly autonomously (e.g. between different waypoints). 
Although the terms are sometimes used interchangeably, an autopilot is usually an extension of a flight controller (FC). It can be seen as an advanced firmware that has more capabilities than a classic FC. 

It utilizes sensors, like the accelerometer, gyroscope, and barometer, to allow the autopilot to orient itself.
If the drone has a GPS module, it can utilize this to travel between predefined waypoints. 

## Prebuilt Solutions

There are many prebuilt autopilot systems available, like this [one](https://www.getfpv.com/electronics/autopilot-systems/holybro-pixhawk-6x-pro-fc-standard-set-w-pm02d-hv.html). 
Many of them are based on the [Pixhawk](https://github.com/pixhawk/Pixhawk-Standards) hardware specification. 
However, these prebuilt systems can be quite expensive. One would usually have to invest several hundred Euros to purchase such a system. 

As one of the objectives of this project is to keep the drones cheap, this direction will be ruled out for the time being. 
Once other options have been exhausted, it could be considered again.

## Betaflight-based Empty Autopilot

[This article](https://medium.com/illumination/fpv-autonomous-operation-with-betaflight-and-raspberry-pi-0caeb4b3ca69) describes the implementation of an "empty" autopilot that simply moves forward, based on Betaflight. 
As this is the FC software of choice and the setups are quite similar, it makes sense to try and integrate this (for more information, look at the [application documentation](/application)).

Going forward, this could be augmented with a targeting system, based on an object tracking mechanism (e.g., with [OpenCV](https://docs.opencv.org/4.x/dc/d6b/group__video__track.html)). Together with a purpose-trained TFLite model, this could be used to build a basic autopilot that follows predefined objects. 

The main challenge here is using a singular camera to approximate the object's position in relation to the drone. This could be done with a [distance sensor](https://www.getfpv.com/hc-sr04-ultrasonic-distance-sensor-module.html) or a LiDAR sensor. However, these sensors add additional weight and cost.

To overcome this, several approaches can be tried out. One of these is [Pix2Pix](https://cir.nii.ac.jp/crid/1360298339716877696), which creates depth images from a monocular camera in a machine learning based approach. The question remains whether the limited hardware setup can cope with these challenging calculations. 

## Ardupilot-based Systems

Ardupilot is an autopilot software project that was started in 2009. It was initially based on the Arduino, hence the name. The Ardupilot software is [compatible](https://ardupilot.org/copter/docs/common-autopilots.html) with the SpeedyBee F405 AIO (the FC used in this project). 

It also provides a ground station software, called [MAVproxy](https://ardupilot.org/mavproxy/index.html). It supports the [MAVlink](https://mavlink.io/en/) protocol for communication with the drone. MAVlink is an XML-based protocol. It uses a publish-subscribe model for data streams and point-to-point connections for its configuration sub-protocols. There are a variety of SDKs provided, which could perhaps be integrated with AI-functionality.

A couple of interesting approaches include [this one](https://sciresol.s3.us-east-2.amazonaws.com/IJST/Articles/2009/Issue-4/Article2.pdf).

## PX4 Autopilot Software

A [PX4](https://docs.px4.io/main/en/index.html) based autopilot is also an interesting route however, the FC used in this project is [not supported](https://docs.px4.io/main/en/flight_controller/). Alternatively it is supported, to run the autopilot on a [Raspberry Pi](https://docs.px4.io/main/en/flight_controller/autopilot_experimental.html) as an external Autopilot. This requires a Raspberry Pi 2/3/4 with Navio2 or PilotPi Shield. PX4 also uses the MAVlink protocol to talk to ground stations and the outside world.

[This](https://scholar.sun.ac.za/bitstreams/a5fc433a-022c-4dfd-b576-3f31dad8c8c5/download) paper introduces a vision-based autonomous unmanned aerial vehicle (UAV) based on PX4. It builds on the ArUco library to estimate position and attitude measurements with the help of augmented reality markers. The vision-based state estimator successfully estimates the position, velocity and attitude of the UAV. This allows for waypoint navigation without using the standard GPS-based state estimator.

# Payload System Integration

The use of drones equipped with a gripper arm opens up a wide range of applications—from inspection and delivery to rescue operations. In this project, the option of mounting a mechanical gripper arm onto an existing drone was investigated, with the goal of picking up and placing objects precisely.
Since there was not enough time for a practical build, this document presents a theoretical plan demonstrating how such a modification could be implemented.

The goal is to develop a concept for a gripper arm that meets the following requirements:
- Take into account the drone’s weight and balance
- Control via Raspberry Pi or with an auxiliray (AUX) switch on the Radiomaster (RC)
- Power supply from the onboard battery via Raspberry Pi or FC

## Mechanical Design and Mounting

Simple [ServoCity Servo-Driven Gripper Kit](https://eu.robotshop.com/products/servocity-servo-driven-gripper-kit-servo-included/):
- Lightweight construction at 101 g, to minimize impact on flight time and stability
- Mounting: Quick-release mechanism or screw attachment on the drones underside
	+ Challenge: A downward-hanging arm shifts the center of gravity
	+ Also: Dampers to reduce vibrations

## Electrical Design (Servo Control and Power Supply)

- Standard servo (PWM-controlled, ~4–6 V supply)
- Connection and control via Raspberry Pi GPIO or FC
- Power supply (4–6 V) via the Raspberry Pi / FC
- Challenge: Secure cable routing to avoid contact with propellers

## Control Concept

- AUX channel for open/close commands via the RC
- Configuration via the “SERVO” tab in Betaflight
- Alternatively: Python script for PWM control of the servo 
	+ This could possibly integrated with object detection
	+ Communication between Raspberry Pi and FC via an MSP connection

## Safety Aspects

- Integrate a Fail-safe mechanism: Servo closes/opens in case of connection loss
- Mechanical lock to prevent unintended opening

## Conclusion
Mounting a gripper arm on the drone is technically feasible and can be realized with existing components (servo, Raspberry Pi, FC). For practical operation, adjustments to weight and center of gravity are required. The presented design provides a basis for future implementation that could go beyond simple object gripping to enable autonomous handling, albeit with reduced overall flight time, due to the additional weight.