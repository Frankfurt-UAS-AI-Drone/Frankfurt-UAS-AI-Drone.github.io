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



