+++
weight = 4
title = "4_Application" 
date = "2025-07-03" 
+++

# Overview
The main goal of the project was, to enable the drone to follow an object (i.e. a banana). For this **object detection** (with a targeting logic) and a way to **send commands** from the Raspberry Pi to the flight controller (FC). </br>
A side goal was implementing a way to start the autopilot via the remote control (RC). To achieve this, a way to **receive telemetry** from the FC is required. 

We were not able to enable the drone to follow an object. The autopilot therefore, remains _empty_. It simply increases the pitch and throttle upon activation. 

## Repos (sources)
Parts of this work were based on existing code bases. ["autopilot_bee_ept"](https://github.com/under0tech/autopilot_bee_ept) and ["YAMSPy"](https://github.com/thecognifly/YAMSPy) are used for the communication with the flight controller, via the ["MulitWii Serial Protocol (MSP)"](https://betaflight.com/docs/development/API/MSP-Extensions). ["reefwing"](https://gist.github.com/reefwing/e9ba13aed51e83cb7245bb4e55b84dea) provides an overview of various MSP codes that can be used to get telemetry from the FC and to send commands to it. </br>

# The Code
This section provides an overview of interesting sections of the code and some explanations if necessary.

## main
At the core of the application lies a control loop that polls the FC for the status of the RC. Upon a state change in _AUX 3_, the `inference` component is triggered to run basic object detection either on a still or on a video (implemented as a stream). Upon a state change in _AUX 4_, the empty autopilot is triggered. Going forward, the autopilot is supposed to be based on a targeting mechanism.

The `inference` component handles the communication with the camera and the Coral TPU. FC communication is enabled through the functions in the `msp` component.

```python
 while True:
        msp.send_msp_request(port, msp.Command.MSP_RC)
        code, payload = msp.read_msp_response(port)
        assert code == msp.Command.MSP_RC

        rc_state = msp.process_MSP_RC(payload)
        print(f'RC: {rc_state}')

        aux3_current = rc_state['aux3']
        aux4_current = rc_state['aux4']

        # Stop video stream if AUX4 switches away from 2000 
        if video_thread and aux3_current != 2000:
            print("Stop recording...")
            controller.stop_event.set()
            video_thread.join(timeout=2)
            video_thread = None

        if aux3_previous == 1000 and aux3_current == 1503:
            print('Taking photo...')
            controller.take_and_infer_still()

        if aux3_previous == 1000 and aux3_current == 2000:
            print("Taking video...")
            video_thread = threading.Thread(target=controller.stream_and_infer_video)
            video_thread.start()
           
        if aux4_previous == 1000 and aux4_current == 1503:
            print('Preparing autopilot...')
            prepared = autopilot.prepare()

        elif aux4_previous == 1503 and aux4_current == 2000:
            print('Starting flight...')
            autopilot.go_forward()

        # Update values
        aux3_previous = aux3_current
        aux4_previous = aux4_current
```

## inference
To enable communication with the camera and the Coral TPU, the `InferenceController` class is implemented, with the following methods.

### __init__(self)
`__init__` initializes the controller with two different models[(links in repo)](https://github.com/Frankfurt-UAS-AI-Drone/AI-Drone). `effdet_lite3` (512x512 / 107.6 ms) for stills and `ssd_mobnet2_tf2` (300x300 / 7.6 ms) for streams. The reason for using two different models is performance. While the former model offers a higher accuracy, it is not usable for videos as it only manages around 3 FPS in all calculations. The latter model musters a 10 FPS stream, which is usable. None of these models are useful in production. For good production performance, it is necessary to train a purpose-specific model. 

```python
self.still_interpreter = make_interpreter("models/effdet_lite3/effdet_lite3.tflite")
self.still_interpreter.allocate_tensors()
self.still_size = common.input_size(self.still_interpreter)
self.still_labels = read_label_file("models/effdet_lite3/effdet_lite3.labels")

self.video_interpreter = make_interpreter('models/ssd_mobnet2/ssd_mobnet2_tf2.tflite')
self.video_interpreter.allocate_tensors()
self.video_size = common.input_size(self.video_interpreter)
self.video_labels = read_label_file("models/ssd_mobnet2/ssd_mobnet2_tf2.labels")
```

The camera is configured to use `RGB888` because this is the configuration required by the models. By default, `Picamera2` includes an alpha channel, which would require every frame to be resized, costing performance.

```python
self.video_config = self.camera.create_video_configuration(main= {"size": self.video_size, 'format': 'RGB888'}, controls={"FrameRate": self.recording_fps})
self.still_config = self.camera.create_still_configuration({"size": self.still_size, 'format': 'RGB888'})
```

### process_frame(self, frame, video=False)
The process_frame function handles the object detection process for a single frame and draws the boxes and annotations around any detected objects. The score threshold can be used to modify the required confidence for an object. This is a tradeoff between accuracy and detecting anything at all. Especially for the smaller `ssd_mobnet2_tf2`, this can become quite difficult for anything other than a person.

```python
objects = detect.get_objects(self.still_interpreter, score_threshold=0.7)
labels = self.still_labels

for obj in objects:
    bbox = obj.bbox
    cv2.rectangle(frame, (bbox.xmin, bbox.ymin), (bbox.xmax, bbox.ymax), (0, 255, 0), 2)
    cv2.putText(frame, f'{obj.id} {labels.get(obj.id, obj.id)} {obj.score:.2f}', (bbox.xmin, bbox.ymin - 10), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 255, 0), 2)
    print(f"Detected: {labels.get(obj.id, obj.id)} (ID: {obj.id}, Score: {obj.score:.2f})")
```

### take_and_infer_still(self)
This function is very straight forward. The onlything noteworthy is the line below. It allows to switch between different configurations and optimize the camera for a still instead of a video. 

```python
    def take_and_infer_still(self):
        self.camera.switch_mode(self.still_config)
        ...
```

### stream_and_infer_video(self)
A multi-threaded model is used for video inference. When AUX 3 is set to record a video on the RC, a new thread is started to record the video. The video is implemented as a stream of images, not as a video itself. This implementation only manages 10 FPS. However, it is in real-time, which could be useful when building functionalities on top of it. 

When AUX 3 is switched away from recording mode, the stop event is set, and the controller stops recording. Upon ending the recording, it is very important to execute `out.release()` otherwise, the resulting video file will be corrupted.

```python
def stream_and_infer_video(self):
        self.stop_event.clear()
        self.camera.switch_mode(self.video_config)
        out = cv2.VideoWriter(f"/home/pi/drone/videos/{datetime.today().strftime('%Y-%m-%d_%H-%M-%S')}.avi", cv2.VideoWriter_fourcc(*'XVID'), self.playback_fps, self.video_size)

        try: 
            while not self.stop_event.is_set():
                recorded_frame = self.camera.capture_array()
                processed_frame = self.process_frame(recorded_frame, video=True)
                out.write(processed_frame)
        except Exception as e:
            print(f"Recording error: {e}")
        finally:
            print("Exiting...")
            out.release()
```

## autopilot
Because the autopilot is empty, the `AutopilotController` class only contains a limited amount of functionality. `init` initializes the values that are necessary for overriding the RC.

### override_rc()
This method is a wrapper for the `send_msp_command` and is used to execute MSP_SET_RAW_RC. This only works for the channels that have been enabled by the `set msp_override_channels_mask` command in the FC.

```python
def override_rc(roll, pitch, throttle, yaw, aux1, aux2, aux3, aux4):  
    data = [roll, 
            pitch, 
            throttle, 
            yaw, 
            aux1, aux2, aux3, aux4]
    
    msp.send_msp_command(serial_port, msp.Command.MSP_SET_RAW_RC, data)
    
    msp_command_id, payload = msp.read_msp_response(serial_port)

    if msp_command_id != msp.MSP_SET_RAW_RC:
        return False
    return True
```
### Flying
Currently, the flying functionality consists of two methods: `prepare()` and `go_forward()`. Both of them wrap override_rc, and simply implement that the autopilot flies forward. It is important that the switch that triggers these methods also enables the MSP_OVERRIDE mode in the flight controller.

```python
def go_forward():
    status = self.override_rc(
                self.DEFAULT_ROLL,
                self.DEFAULT_PITCH + 20, 
                self.DEFAULT_THROTTLE + 100, 
                self.DEFAULT_YAW, 
                self.AUX1, self.AUX2, self.AUX3, self.AUX4) 
    return status
```

## msp
The `Command` enum contains the necessary command IDs for the implemented functionalities. `send_msp_request` is used to send telemetry requests, and `send_msp_command` is used to send instructions to the FC. The main difference is that the latter allows for a payload that contains the instructions. This is used to send RC values to the flight controller, to control it from the Pi.

```python
def send_msp_command(serial_port, msp_command_id, data):
    payload = bytearray()
    for value in data:
        payload += struct.pack('<1H', value)

    header = b'$M<'
    length = len(payload)
    checksum = get_checksum(msp_command_id, payload)

    msp_package = header + bytes([length, msp_command_id]) + payload + bytes([checksum])
    serial_port.write(msp_package)
```

`read_msp_response` is used to deconstruct the response of the FC, so its payload can be processed with the command-specific functions (e.g. `process_MSP_STATUS`). With the help of `readbytes`, these functions turn the response into a format that is readable by humans.

```python
def process_MSP_STATUS(data):
    cycleTime = readbytes(data, size=16, unsigned=True)
    i2cError = readbytes(data, size=16, unsigned=True)
    activeSensors = readbytes(data, size=16, unsigned=True)
    mode = readbytes(data, size=32, unsigned=True)
    profile = readbytes(data, size=8, unsigned=True)

    return {
        'cycleTime': cycleTime,
        'i2cError': i2cError,
        'activeSensors': activeSensors,
        'mode': mode,
        'profile': profile,
    }
```