+++
title = "Application" 
date = "2025-07-01" 
+++

# Overview
The main goal of the project was enabling the drone to follow an object (i.e. a banana). For this **object detection** (with a targeting logic) and a way to **send commands** from the Raspberry Pi to the flight controller (FC). </br>
A side goal was implementing a way to start the auto pilot via the remote control (RC). To achieve this, a way to **receive telemetry** from the FC is required. 

We were not able to enable the drone to follow an object. The autopilot therefore remains _empty_. It simply increases the pitch and throttle, upon being activated. 

## Repos (sources)
Parts of this work were based on existing code bases. ["autopilot_bee_ept"](https://github.com/under0tech/autopilot_bee_ept) and ["YAMSPy"](https://github.com/thecognifly/YAMSPy) are used for the communication with the flight controller, via the ["MulitWii Serial Protocol (MSP)"](https://betaflight.com/docs/development/API/MSP-Extensions). ["reefwing"](https://gist.github.com/reefwing/e9ba13aed51e83cb7245bb4e55b84dea) provides an overview of various MSP codes, that can be used to get telemetry from the FC and to send commands to it. </br>

## Architecture
The architecture of this project is a work in progress. What follows is a rough description of the current state. At the core of the application lies a control loop, that polls the FC for the status of the RC.

Upon a state change in _AUX 3_ the `inference` component is triggered to run basic object detection either on a still or on a video (implemented as a stream). Upon a state change in _AUX 4_, the empty autopilot is triggered. Going forward the autopilot is supposed to be based on a targeting mechanism to.

The `inference` component handles the communication with the camera and the Coral TPU. FC communication is enabled by the functions in the `msp` component.

# The Code
This section provides an overview over interesting sections of the code and some explanations if necessary.

## inference
To enable communication with the camera and the Coral TPU, the `InferenceController` class is implemented, with the following methods:

### __init__(self)
`__init__` initializes the controller with two different models. `effdet_lite3` (512x512 / 107.6 ms) for stills and `ssd_mobnet2_tf2` (300x300 / 7.6 ms) for streams. The reason for using two different models, is performance. While the former model offers a higher accuracy, it is not usable for videos as it only managaes around 3 FPS in all calculations. The latter model manages a 10 FPS stream, which is usable. Increasing the performance of the video stream remains a task for future semesters.

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

The camera is configured to use `RGB888` because this is the configuration required by the models. By default `Picamera2` includes an alpha channel, which would require every frame to be resized, costing performance.

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
A multi threaded model is used for video inference. When AUX 3 is set to record a video on the RC, a new thread is started to record the video. The video is implemented as a stream of images, not as a video itself. This implementation only manages 10 FPS. However it is in realtime, which could be useful, when building functionalities on top of it. 

When AUX 3 is switched away from recording mode, the stop event is set and the controller stops recording. Upon ending the recording, it is very important to execute `out.release()` otherwise the resulting video file will be corrupted.

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

## msp

