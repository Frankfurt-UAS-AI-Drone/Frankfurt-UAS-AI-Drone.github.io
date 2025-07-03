+++
weight = 3
title = "3_Rapsberry-Pi" 
date = "2025-07-01" 
+++

The following section covers all aspects related to the Raspberry Pi used in our project. This includes its hardware setup, software configuration, and its role in running the AI application.

# Raspberry Pi

As mentioned on the drone page, we used the single board computer `Raspberry Pi Zero WH PI3G` for the AI application. For the application to run, it is necessary to make some configurations and install specific packages onto the computer. 

## Installations:
- Install Python (default version of bullseye)
- Install [libedgetpu1-std](https://coral.ai/docs/accelerator/get-started/#runtime-on-linux) 
- Install python3-pycoral (requires python < 3.10)
- Install python3-opencv (requires python < 3.12)
- Install python3-picamera2
- Install pip3 -> MultiWii (requires python > 3.7) and pyserial
- Install Docker Engine for [Debian](https://docs.docker.com/engine/install/debian/)

All these installations can be made by using pip. For more information on how to use pip, please refere to the [official documentation](https://pip.pypa.io/en/stable/installation/).<br/>

## Configurations
### RPi imager
Begin by installing the Raspberry Pi Imager appropriate for your operating system. Launch the application and select the target device as `Raspberry Pi Zero 2 W`. For the operating system, choose `Raspberry Pi OS Lite (64-bit, Bullseye)`. Proceed by selecting the SD card as the storage medium. Click Next to continue, then access the OS customization settings. Complete the configuration form, ensuring that SSH access is enabled—either via password authentication or public key. Finally, apply the configuration to initiate the imaging process.

Before booting the Raspberry Pi for the first time, open the terminal and edit the `/boot/config.txt` file. Within this file, set `camera_auto_detect=0` to disable automatic camera detection, and specify the correct camera overlay by adding `dtoverlay=imx219`, which corresponds to the connected Pi camera module. This step ensures proper initialization of the camera hardware. For additional details and technical specifications, refer to the manufacturer's documentation available [here](https://joy-it.net/de/products/rb-camera-jt-v2-120).

### First Boot
#### (1) Establish Connection to Raspberry Pi
After booting the Raspberry Pi, determine its IP address by checking your router’s connected devices list or using a network scanning tool. Once identified, establish a secure shell (SSH) connection using the following syntax in a terminal: `ssh pi@<IP-ADDRESS>`. Replace `<IP-ADDRESS>`with the acctual IP address of the raspberry.

#### (2) Verify camera functionality
To test the functionality of the connected camera module, use the pre-installed utility `libcamera-hello`. After connecting to the Raspberry Pi via SSH, use `libcamera-hello` to test wether the camera functions properly.