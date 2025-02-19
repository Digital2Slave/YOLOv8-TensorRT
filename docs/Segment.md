# YOLOv8-seg Model with TensorRT

Instance segmentation models are currently experimental.

Our conversion route is :
    YOLOv8 PyTorch model -> ONNX -> TensorRT Engine

***Notice !!!*** We don't support TensorRT API building !!!

# Export Your Own ONNX model

You can export your onnx model by `ultralytics` API.

``` shell
python3 export_seg.py \
--weights yolov8s-seg.pt \
--opset 11 \
--sim \
--input-shape 1 3 640 640 \
--device cuda:0
```

#### Description of all arguments

- `--weights` : The PyTorch model you trained.
- `--opset` : ONNX opset version, default is 11.
- `--sim` : Whether to simplify your onnx model.
- `--input-shape` : Input shape for you model, should be 4 dimensions.
- `--device` : The CUDA deivce you export engine .

You will get an onnx model whose prefix is the same as input weights.

This onnx model doesn't contain postprocessing.

# Export Engine by TensorRT Python api

You can export TensorRT engine from ONNX by [`build.py` ](../build.py).

Usage:

``` shell
python3 build.py \
--weights yolov8s-seg.onnx \
--fp16  \
--device cuda:0 \
--seg
```

#### Description of all arguments

- `--weights` : The ONNX model you download.
- `--fp16` : Whether to export half-precision engine.
- `--device` : The CUDA deivce you export engine.
- `--seg` : Whether to export seg engine.

# Export Engine by Trtexec Tools

You can export TensorRT engine by [`trtexec`](https://github.com/NVIDIA/TensorRT/tree/main/samples/trtexec) tools.

Usage:

``` shell
/usr/src/tensorrt/bin/trtexec \
--onnx=yolov8s-seg.onnx \
--saveEngine=yolov8s-seg.engine \
--fp16
```

# Inference

## Infer with python script

You can infer images with the engine by [`infer.py`](../infer.py) .

Usage:

``` shell
python3 infer.py \
--engine yolov8s-seg.engine \
--imgs data \
--show \
--out-dir outputs \
--device cuda:0 \
--seg
```

#### Description of all arguments

- `--engine` : The Engine you export.
- `--imgs` : The images path you want to detect.
- `--show` : Whether to show detection results.
- `--out-dir` : Where to save detection results images. It will not work when use `--show` flag.
- `--device` : The CUDA deivce you use.
- `--profile` : Profile the TensorRT engine.
- `--seg` : Infer with seg model.

## Infer with C++

You can infer segment engine with c++ in [`csrc/segment`](../csrc/segment) .

### Build:

Please set you own librarys in [`CMakeLists.txt`](../csrc/segment/CMakeLists.txt) and modify you own config in [`main.cpp`](../csrc/segment/main.cpp) such as `CLASS_NAMES`, `COLORS`, `MASK_COLORS` and postprocess parameters .

```c++
int topk = 100;
int seg_h = 160; // yolov8 model proto height
int seg_w = 160; // yolov8 model proto width
int seg_channels = 32; // yolov8 model proto channels
float score_thres = 0.25f;
float iou_thres = 0.65f;
```

``` shell
export root=${PWD}
cd src/segment
mkdir build
cmake ..
make
mv yolov8-seg ${root}
cd ${root}
```

***Notice !!!***

If you have build OpenCV(>=4.7.0) by yourself, it provides a new api [`cv::dnn::NMSBoxesBatched`](https://docs.opencv.org/4.x/d6/d0f/group__dnn.html#ga977aae09fbf7c804e003cfea1d4e928c) .
It is a gread api about efficient in-class nms . It will be used by default!

***!!!***

Usage:

``` shell
# infer image
./yolov8-seg yolov8s-seg.engine data/bus.jpg
# infer images
./yolov8-seg yolov8s-seg.engine data
# infer video
./yolov8-seg yolov8s-seg.engine data/test.mp4 # the video path
```
