# Normal Usage of [`ultralytics`](https://github.com/ultralytics/ultralytics)

## Export TensorRT Engine

### 1. Python script

Usage:

```python

from ultralytics import YOLO

# Load a model
model = YOLO("yolov8n.pt")  # load a pretrained model (recommended for training)
success = model.export(format="engine", device=0)  # export the model to engine format
assert success
```

After executing the above script, you will get an engine named `yolov8n.engine` .

### 2. CLI tools

```shell
yolo export model=yolov8n.pt format=engine device=0
```

After executing the above command, you will get an engine named `yolov8n.engine` too.

## Inference with c++

You can infer with c++ in [`csrc/detect/normal`](../csrc/detect/normal) .

### Build:

Please set you own librarys in [`CMakeLists.txt`](../csrc/detect/normal/CMakeLists.txt) and modify `CLASS_NAMES` and `COLORS` in [`main.cpp`](../csrc/detect/normal/main.cpp).

Besides, you can modify the postprocess parameters such as `num_labels` and `score_thres` and `iou_thres` and `topk` in [`main.cpp`](../csrc/detect/normal/main.cpp).

```c++
int num_labels = 80;
int topk = 100;
float score_thres = 0.25f;
float iou_thres = 0.65f;
```

And build:

``` shell
export root=${PWD}
cd src/detect/normal
mkdir build
cmake ..
make
mv yolov8 ${root}
cd ${root}
```

Usage:

``` shell
# infer image
./yolov8 yolov8s.engine data/bus.jpg
# infer images
./yolov8 yolov8s.engine data
# infer video
./yolov8 yolov8s.engine data/test.mp4 # the video path
```
