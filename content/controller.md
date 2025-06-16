+++
title = "Drone Controller"
date = "2025-06-16"
+++

The following section covers all aspects related to the configuration of the controller used to fly the drone. This includes setting up input channels, assigning switches, and adjusting flight modes to ensure full manual control and reliable communication with the drone. As mentioned in the hardware section, we used the `Radiomaster Boxer ELRS GRYM2` as a controller.

# Controller
## Startup Warnings
When the controller is powered on, it may display the following warnings:

- Throttle Warning: Triggered if the throttle stick (left stick) is not in the lowest position when turned on.
- Switch Warning:   Appears when one or more switches are not in their default positions.
- Alarms Warning:   Shown if the sound mode is set to mute, which may prevent important audio alerts.
- SD Card Warning:  Displayed if the SD card's content version does not match the controller’s firmware version. If you are facing issues regarding the SD Card Warning, please have a look at this [fix](https://oscarliang.com/fix-sd-card-warning-opentx/)

## Channels
### Channel Mappings
Channel mapping of a controller refers to the way specific control inputs (like stick movements or switch toggles) are assigned to output channels that are sent to the flight controller or receiver. Each channel carries a particular function and channel mapping determines which input controls which channel. 
There are two common channel mapping schemes for RC controllers: AETR and TAER. We chose AETR, as this is the default configuration in Betaflight (AETR1234).
AETR stands for Aileron (A), Elevator (E), Throttle (T), and Rudder (R), corresponding to the basic control axes of the drone.

The first four channels are mapped as follows:

- Channel 1: Aileron (Roll – left/right)
- Channel 2: Elevator (Pitch – forward/backward)
- Channel 3: Throttle (speed up/down)
- Channel 4: Rudder (Yaw – rotate left/right)


## Switches
The switches on the controller are used to perform key functions such as arming the drone, changing flight modes, or triggering features like GPS rescue.
It’s recommended to use Channel 5 for arming the drone. To configure this:
1. Navigate to the Mixes page, highlight Channel 5, and press the jog wheel to enter its settings.
2. Scroll to the Source field and flip the switch you want to assign for arming (e.g. SA or SD).
3. Leave the other values at their default and save the configuration.

For flight mode selection, it's recommended to use a 3-position switch (e.g. SC or SD) to allow easy toggling between multiple modes. After selecting your switch:
1. Go to the next free channel (e.g. CH6).
2. Repeat the same setup process as for arming, assigning the chosen switch to this channel.

## Modes
### Setting up Modes
Setting up modes is essential for configuring your controller to trigger specific flight functions like arming, flight modes, or GPS rescue. Before assigning any modes, make sure the corresponding AUX channels are already configured on your transmitter.

Follow these steps to set up modes in the Betaflight Configurator:
1. Open the Modes tab in Betaflight Configurator.
2. Select the desired mode (e.g., ARM) and click “Add Range.”
3. Choose the appropriate AUX channel from the dropdown (e.g., AUX 1, which is typically Channel 5).
4. Drag the range slider to define when the mode should be active, based on your switch type:
    - 2-position switch:
        - UP = ~1000
        - DOWN = ~2000
    - 3-position switch:
        - UP = ~1000
        - MID = ~1500
        - DOWN = ~2000

Repeat this process for each additional mode you'd like to configure, such as ANGLE, HORIZON, ACRO, or GPS RESCUE.

### Important Modes
- ARM
    - Starts the motors and enables flight.
    - Should be disabled immediately after a crash to prevent damage.
    - If Motor Stop is enabled, motors only spin up when throttle is increased.

- ACRO
    - No flight assistance from the flight controller.
    - All movement is controlled manually by the pilot.
    - Ideal for experienced pilots.
    - Also known as the “ultimate flight mode.”

- ACRO TRAINER
    - Similar to ACRO, but with a tilt angle limit to help beginners.
    - Prevents full flips and reduces the risk of losing control.

- ACRO TRAINER
    - Similar to ACRO, but with a tilt angle limit to help beginners.
    - Prevents full flips and reduces the risk of losing control.
    - Requires accelerometer to be enabled.

- HORIZON
    - Like ANGLE mode, but allows flips when the stick passes a certain threshold.
    - Also requires accelerometer.

- GPS RESCUE
    - Commands the drone to fly back to its takeoff point using GPS.
    - Useful in case of signal loss or disorientation.
    - Requires a GPS module

- FAILSAFE
    - Simulates an RC signal loss.
    - Used for testing failsafe behavior during setup.

- ANTI-GRAVITY
    - Smooths out sudden dips caused by rapid throttle changes.
    - Helps maintain altitude during quick stick inputs.
    
- BLACKBOX
    - Toggles Blackbox logging, which records flight data for later analysis.
    - Helpful for tuning and troubleshooting.

- FLIP OVER AFTER CRASH (Turtle Mode)
    - Allows the quad to flip itself upright if it's upside down after a crash.
    - Works by spinning specific motors in reverse.