+++
title = "The Application" 
date = "2025-05-15" 
+++

This section outlines the software stack employed in our project. It includes the tools, frameworks, and libraries we used, as well as the architecture and logic behind our implementation decisions to ensure stability, scalability, and maintainability.

# InferenceController

The __init__ function is the constructor of the class and is executed automatically when a new instance of the class is created. In this case, it sets up both the machine learning model running on the Coral Edge TPU and the Raspberry Pi camera.

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
