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

Going forward, this could be augmented with a targeting system, based on an object tracking mechanism (e.g. with OpenCV). To be continued.

## PX4 Autopilot software