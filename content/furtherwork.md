+++
weight = 5
title = "5_Further-Work" 
date = "2025-07-01"
+++

# Payload System Integration

# Autopilot integration
An autopilot is a system that enables the drone to fly autonomously (e.g. between different waypoints). 
Although the terms are sometimes used interchangably, an autopilot is not a flight controller (FC). An autopilot usually contains an FC or is built on one. 

It utilizes the sensors of an FC, like the accelerometer, gyroscope and barometer allow the autopilot to orient itself. 
If the drone has a GPS module, it can utilize this to travel between predefined waypoints. 

## Prebuilt solutions

There are many prebuilt autopilot systems available, like this [one](https://www.getfpv.com/electronics/autopilot-systems/holybro-pixhawk-6x-pro-fc-standard-set-w-pm02d-hv.html). 
Many of them are based on the [Pixhawk](https://github.com/pixhawk/Pixhawk-Standards) hardware specification. 
However these prebuilt systems can be quite expensive. One would usually have to invest several hundred Euros to purchase such a system. 

As one of the objectives of this project is to keep the drones cheap, this direction will be ruled out for the time being. 
Once other options have been exhausted, it could be considered again.

## Betaflight-based Empty Autopilot

[This article](https://medium.com/illumination/fpv-autonomous-operation-with-betaflight-and-raspberry-pi-0caeb4b3ca69) describes the implementation of an "empty" autopilot, that simply moves forward, based on Betaflight. 
As this is the FC software of choice and the setups are quite similar, it makes sense to try and integrate this (for more information look at the application documentation).

Going forward, this could be augmented with a targeting system, based on an object tracking mechanism (e.g. with [OpenCV](https://docs.opencv.org/4.x/dc/d6b/group__video__track.html)). Together with a purpose-trained TFLite model, this could be used to build a basic autopilot that follows predefined objects. 

The main challenge here is using a singular camera to approximate the objects position in relation to the drone. This could be done with a [distance sensor](https://www.getfpv.com/hc-sr04-ultrasonic-distance-sensor-module.html) or a LiDAR sensor. However these sensors add additional weight and cost.

To overcome this, several approaches can be tried out. One of these is [Pix2Pix](https://cir.nii.ac.jp/crid/1360298339716877696), which creates depth images from a monocular cameras, which is a machine learning based approach. The question remains, whether the limited hardware setup can cope with these challenging calculations. 

## Ardupilot based systems

Ardupilot is an autopilot software project, that was started in 2009. It was initially based on the Arduino, hence the name. The Ardupilot software is [compatible](https://ardupilot.org/copter/docs/common-autopilots.html) with the SpeedyBee F405 AIO (the FC used in this project). 

It also provides a ground station software, called [MAVproxy](https://ardupilot.org/mavproxy/index.html). It supports the [MAVlink](https://mavlink.io/en/) protocol, for communication with the drone. MAVlink is an XML-based protocol. It uses a publish-subscribe model for data streams and point-to-point connections for its configuration sub-protocols. There are a variety of SDKs provided, which could perhaps be integrated with AI-functinoality.

A couple of interesting approaches include [this one](https://sciresol.s3.us-east-2.amazonaws.com/IJST/Articles/2009/Issue-4/Article2.pdf).

## PX4 Autopilot software

## Theoretical Part: Concept for a Drone with a Gripper Arm

### Introduction
The use of drones equipped with a gripper arm opens up a wide range of applications—from inspection and delivery to rescue operations. In this project, the option of mounting a mechanical gripper arm onto an existing drone was investigated, with the goal of picking up and placing objects precisely.
Since there was not enough time for a practical build, this document presents a theoretical plan demonstrating how such a modification could be implemented.

### Objective
The goal is to develop a modular concept for a gripper arm that meets the following requirements:
Take into account the drone’s weight and balance
Remote control via Raspberry Pi / flight controller and Radiomaster (AUX)
Power supply from the onboard battery via Raspberry Pi or flight controller
Control either via Raspberry Pi or directly through the flight controller and Radiomaster

### Mechanical Design
#### 3.1. Gripper Arm Design

Simple [ServoCity Servo-Driven Gripper Kit](https://eu.robotshop.com/products/servocity-servo-driven-gripper-kit-servo-included/)

Lightweight construction at 101 g, to minimize impact on flight time and stability
Mounting system with quick-release mechanism or screw attachment on the underside of the drone

#### 3.2. Mounting

Center-of-gravity adjustment (a downward-hanging arm shifts the center of gravity)
Dampers to reduce vibrations

### Electrical Design

#### 4.1. Servo Control

Standard servo (PWM-controlled, ~4–6 V supply)
Connection and control via Raspberry Pi GPIO or flight controller

#### 4.2. Power Supply

Power from 4–6 V via the Raspberry Pi / flight controller
Secure cable routing to avoid contact with propellers

### Control Concept
#### 5.1. Via Flight Controller (Betaflight)

Use of the “SERVO” tab in Betaflight
Configuration of an AUX channel for open/close commands (Radiomaster transmitter)

#### 5.2. Via Raspberry Pi

Python script for PWM control of the servo
Communication between Raspberry Pi and flight controller via MSP/Betaflight

### Software Aspect

Raspberry Pi can act as the control center for autonomous opening/closing
Connection to the flight controller via MSP protocol for status data
Camera integration for object detection
Weight and Flight Characteristics
Simulated calculation: gripper arm + servo + cables ≈ 100–150 g
Reduced flight time (~10–20% less)
Center-of-gravity adjustment may require shifting the battery position

### Safety Aspects

Fail-safe mechanism: servo closes or opens in case of connection loss
Mechanical lock to prevent unintended opening

### Conclusion and Outlook
Mounting a gripper arm on the drone is technically feasible and can be realized with existing components (servo, Raspberry Pi, flight controller). For practical operation, adjustments to weight and center of gravity are required. The presented design provides a basis for future implementation that could go beyond simple object gripping to enable autonomous handling.


PX4 -> https://docs.px4.io/main/en/index.html

[This](https://scholar.sun.ac.za/bitstreams/a5fc433a-022c-4dfd-b576-3f31dad8c8c5/download) paper introduces a vision-based autonomous UAV. 