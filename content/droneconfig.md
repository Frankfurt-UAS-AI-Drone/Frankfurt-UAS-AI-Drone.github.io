+++
weight = 2
title = "2_Configuration"
date = "2025-06-16"
+++

The following section covers all aspects related to the configuration of the controller used to fly the drone. This includes setting up input channels, assigning switches, and adjusting flight modes to ensure full manual control and reliable communication with the drone. As mentioned in the hardware section, we used the `Radiomaster Boxer ELRS GRYM2` as a controller.

# Different Settings
- Board and Sensor Alignment: The yaw angle needed to be set to 0° instead of the default 45° to ensure correct orientation of the flight controller.
- GPS Activation: GPS functionality had to be enabled manually under the "Other Features" menu.
- Throttle Configuration: In the "PID Tuning" section, under "Rateprofile Settings", we set the Throttle Limit to Scale at 70% to reduce overall throttle responsiveness.
- Motor Direction Setup: In the "Motors" tab, the correct motor spin direction was configured—this determines whether the motors spin inward or outward relative to the center of the drone.

# Controller
## Startup Warnings
When the controller is powered on, it may display the following warnings:

- Throttle Warning: Triggered if the throttle stick (left stick) is not in the lowest position when turned on.
- Switch Warning:   Appears when one or more switches are not in their default positions.
- Alarms Warning:   Shown if the sound mode is set to mute, which may prevent important audio alerts.
- SD Card Warning:  Displayed if the SD card's content version does not match the controller’s firmware version. If you are facing issues regarding the SD Card Warning, please have a look at this [fix](https://oscarliang.com/fix-sd-card-warning-opentx/).

## Channels
### Channel Mappings
Channel mapping of a controller refers to the way specific control inputs (like stick movements or switch toggles) are assigned to output channels that are sent to the flight controller or receiver. Each channel carries a particular function and channel mapping determines which input controls which channel. 
There are two common channel mapping schemes for RC controllers: AETR and TAER. We chose AETR, as this is the default configuration in Betaflight (AETR1234).
AETR stands for Aileron (A), Elevator (E), Throttle (T), and Rudder (R), corresponding to the basic control axes of the drone.

1. From the home screen, press the Model button. 
2. Navigate to Mixes tab by pressing the Page button until the heading shows up.
3. The following maps the channels according to the AETR (Betaflight default):
    1. Channel 1 Aileron (Roll – left/right)
        - Highlight CH1 using the jog wheel and press to enter the setup.
        - Set Source to *aileron* stick. -> Optionally, adjust Weight (how much stick movement passed to output) or apply Expo for smoother control.
    2. Channel 2 Elevator (Pitch – forward/backward) -> Highlight CH2 and set the Source to the elevator stick.
    3. Channel 3 Throttle (speed up/down) -> Highlight CH3 and set the Source to the throttle stick.
    4. Channel 4 Rudder (Yaw – rotate left/right) -> Highlight CH4 and set the Source to the rudder stick.

### Default Channels
- To change the ***default channel*** order in EdgeTX do the following:
	1. Press the **SYS** button and navigate to the *Settings* tab with the **PAGE** button.
	2. Scroll down to find the *Default Channel Order* setting and set it to **AETR**
	3. Press return to save the changes.

### Channel testing
- Testing the channel mapping setup:
	1. Connect your drone and open the Receiver tab in Betaflight Configurator
	2. Move the sticks and see if the corresponding channels respond correctly.
		1. CH 5 corresponds to AUX 1, CH 6 corresponds to AUX 2 and so on...

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

### Used Modes
We used two key flight modes in our setup: Arm and Angle.

The Arm mode is required to activate the motors and make the drone ready for flight. When enabled, the motors spin at idle speed, allowing the drone to respond to throttle input. Without arming, the drone cannot take off or be controlled via the transmitter. To ensure intentional activation, arming is configured to require two switches on the controller.</br>
The Angle mode is a stabilized flight mode that automatically levels the drone horizontally when the sticks are released. It limits the maximum tilt angle, making it easier to control and preventing the drone from flipping. This is especially useful for beginner pilots or situations where stable flight is important.

Additionally, many other flight modes can be configured, such as Acro, Acro Trainer, Horizon, and others. For a complete overview of all available modes, please refer to the list [here](https://betaflight.com/docs/development/Modes).

### Mode Configuration
In the Modes tab, we configured Arm on AUX1 and AUX2, Angle Mode on AUX5, and MSP Override Mode on AUX4. The overwrite mode is active in the range from 1775 to 2100 and is controlled via a three-position switch, which is used for activating the autopilot. More details about this feature are provided on the [Application Page](/application).

## Port Configurations
- UART3: Configured with the VTX (IRC Tramp) protocol under the Peripherals section to enable control of the video transmitter.
- UART4: Enabled MSP for Bluetooth Low Energy (BLE) communication.
- UART5: Assigned to GPS under Sensor Input, with a baud rate of 57600.
- UART6: Configured with Serial RX to receive input from the radio receiver.

## MSP Override Mask
