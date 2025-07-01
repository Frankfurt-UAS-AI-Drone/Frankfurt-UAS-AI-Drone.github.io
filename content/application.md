+++
title = "The Application" 
date = "2025-05-15" 
+++

# Overview
The main goal of the project was enabling the drone to follow an object (i.e. a banana). For this **object detection** (with a targeting logic) and a way to **send commands** from the Raspberry Pi to the flight controller (FC). </br>
A side goal was implementing a way to start the auto pilot via the remote control (RC). To achieve this, a way to **receive telemetry** from the FC is required. 

## Repos (sources)
Parts of this work were based on existing code bases. ["autopilot_bee_ept"](https://github.com/under0tech/autopilot_bee_ept) and ["YAMSPy"](https://github.com/thecognifly/YAMSPy) are used for the communication with the flight controller, via the ["MulitWii Serial Protocol (MSP)"](https://betaflight.com/docs/development/API/MSP-Extensions). ["reefwing"](https://gist.github.com/reefwing/e9ba13aed51e83cb7245bb4e55b84dea) provides an overview of various MSP codes, that can be used to get telemetry from the FC and to send commands to it. </br>

The inference and targeting logic are based on the ["Directional-object-tracking-with-TFLite-and-Edge-TPU"](https://github.com/Tqualizer/Directional-object-tracking-with-TFLite-and-Edge-TPU). 


## Architecture
The architecture of this project is a work in progress. What follows is a rough description of the current state. At the core of the application lies a control loop, that polls the FC for the status of the RC.

Upon a state change in _AUX 3_ the `inference` component is triggered to take and infer one picture or a stream of pictures. Currently this only saves them to a file. In the future this is supposed to be the base for the auto pilot. Upon a state change in _AUX 4_, an empty autopilot is triggered, as the targeting logic has not been finalized. 

The `inference` component handles the communication with the camera and the Coral TPU. FC communication is enabled by the functions in the `msp` component.

# The Code
This section provides an overview over the code and some explanations if necessary.

## inference
To enable communication with the camera and the Coral TPU, the `InferenceController` class is implemented, with the following methods:


__init__ initializes the controller with two different models. `effdet_lite3` (512x512 / 107.6 ms) for stills and `ssd_mobnet2_tf2` (300x300 / 7.6 ms) for streams. The reason for using two different models, is performance. While the former model offers a higher accuracy, it is not usable for videos as it only managaes around 3 FPS in all calculations. The latter model manages a 10 FPS stream, which is usable. However it should be looked at on how this performance can be increased to provide a better base for the auto pilot.
```python
self.interpreter = make_interpreter('models/effdet_lite3.tflite')
self.interpreter.allocate_tensors()
self.size = common.input_size(self.interpreter)
```
First, the Coral TPU is initialized by loading a pre-trained TensorFlow Lite model (effdet_lite3.tflite). The interpreter is created with make_interpreter, and the memory for the model's tensors is allocated using allocate_tensors(). The required input size for the model is then stored in self.size, which will later be used to ensure the camera feeds the model with correctly sized input.

```python
self.camera = Picamera2()
self.preview_config = self.camera.create_preview_configuration({'format': 'RGB888'})
self.video_encoder = H264Encoder()

self.video_config = self.camera.create_video_configuration(main={"size": self.size}, encode="main")
self.video_fps = 30
self.still_config = self.camera.create_still_configuration({"size": self.size})
```
Next, the Raspberry Pi camera is initialized using the Picamera2 library. A preview configuration is created using the RGB888 format, which is suitable for on-screen display or further processing. Additionally, a video encoder (H264Encoder) is initialized for handling video streams. A video configuration is prepared that matches the model's input size, and the desired frame rate is set to 30 frames per second. There's also a configuration for still image capture, again using the model’s input dimensions.

```python
self.camera.configure(self.preview_config)
self.camera.set_controls({"FrameRate": self.video_fps})
self.camera.start()
sleep(2)
```
After these configurations are set up, the camera is configured with the preview settings, the frame rate is applied via camera controls, and the camera is started. A short delay of two seconds is included at the end to give the camera time to properly initialize before use.

```python
def __init__(self):
    self.interpreter = make_interpreter('models/effdet_lite3.tflite')
    self.interpreter.allocate_tensors()
    self.size = common.input_size(self.interpreter)

    self.camera = Picamera2()
    self.preview_config = self.camera.create_preview_configuration({'format': 'RGB888'})
    self.video_encoder = H264Encoder()

    self.video_config = self.camera.create_video_configuration(main={"size": self.size}, encode="main")
    self.video_fps = 30
    self.still_config = self.camera.create_still_configuration({"size": self.size})

    self.camera.configure(self.preview_config)
    self.camera.set_controls({"FrameRate": self.video_fps})
    self.camera.start()
    sleep(2)
```

The process_frame function handles the object detection process for a single video frame.

```python
common.set_input(self.interpreter, frame)
self.interpreter.invoke()
```
First, it prepares the frame as input for the Coral Edge TPU by passing it to the interpreter using common.set_input(). Once the frame is set, the interpreter runs inference with self.interpreter.invoke(), which processes the image using the preloaded TensorFlow Lite model.

```python
objects = detect.get_objects(self.interpreter, score_threshold=0.5)

for obj in objects:
    bbox = obj.bbox
    cv2.rectangle(frame, (bbox.xmin, bbox.ymin), (bbox.xmax, bbox.ymax), (0, 255, 0), 2)
    cv2.putText(frame, f'{obj.id} {obj.score:.2f}', (bbox.xmin, bbox.ymin - 10), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 255, 0), 2)
```
After the inference is complete, the function retrieves the detected objects using detect.get_objects(), filtering out any results with a confidence score below 0.5. For each detected object, it extracts the bounding box (bbox) and draws a green rectangle around the object on the frame using OpenCV. It also overlays a label above the box displaying the object ID and confidence score.

```python
return frame
```
Finally, the annotated frame is returned, now containing visual markers for any detected objects. This frame can then be displayed, recorded, or used for further processing.

```python
def process_frame(self, frame):
    common.set_input(self.interpreter, frame)
    self.interpreter.invoke()

    objects = detect.get_objects(self.interpreter, score_threshold=0.5)

    for obj in objects:
        bbox = obj.bbox
        cv2.rectangle(frame, (bbox.xmin, bbox.ymin), (bbox.xmax, bbox.ymax), (0, 255, 0), 2)
        cv2.putText(frame, f'{obj.id} {obj.score:.2f}', (bbox.xmin, bbox.ymin - 10), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 255, 0), 2)

    return frame
```



```python
```

```python
```
