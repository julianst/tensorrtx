# TensorRTx

TensorRTx aims to implement popular deep learning networks with tensorrt network definition APIs. As we know, tensorrt has builtin parsers, including caffeparser, uffparser, onnxparser, etc. But when we use these parsers, we often run into some "unsupported operations or layers" problems, especially some state-of-the-art models are using new type of layers.

So why don't we just skip all parsers? We just use TensorRT network definition APIs to build the whole network, it's not so complicated.

I wrote this project to get familiar with tensorrt API, and also to share and learn from the community.

TensorRTx has a brother project [Pytorchx](https://github.com/wang-xinyu/pytorchx). All the models are implemented in pytorch first, and export a weights file xxx.wts, and then use tensorrt to load weights, define network and do inference.

## Getting Started

There is a guide for quickly getting started, taking lenet5 as a demo. [Getting_Started.](./getting_started)

## Test Environment

1. Jetson TX1 / Ubuntu16.04 / cuda9.0 / cudnn7.1.5 / tensorrt4.0.2 / nvinfer4.1.3

2. GTX1080 / Ubuntu16.04 / cuda10.0 / cudnn7.6.5 / tensorrt7.0.0 / nvinfer7.0.0

Currently, TX1 ans x86 GTX1080 were tested. trt4 api were using, some api are deprecated in trt7, but still can compile successfully.


## Models

Following models are implemented, each one also has a readme inside.

|Name | Description |
|-|-|
|[lenet](./lenet) | the simplest, as a "hello world" of this project |
|[alexnet](./alexnet)| easy to implement, all layers are supported in tensorrt |
|[googlenet](./googlenet)| GoogLeNet (Inception v1) |
|[inception](./inceptionv3)| Inception v3 |
|[mnasnet](./mnasnet)| MNASNet with depth multiplier of 0.5 from the paper |
|[mobilenet](./mobilenetv2)| MobileNet V2, V3-small, V3-large. |
|[resnet](./resnet)| resnet-18, resnet-50 and resnext50-32x4d are implemented |
|[senet](./senet)| se_resnet50 |
|[shufflenet](./shufflenetv2)| ShuffleNetV2 with 0.5x output channels |
|[squeezenet](./squeezenet)| SqueezeNet 1.1 model |
|[vgg](./vgg)| VGG 11-layer model |
|[yolov3](./yolov3)| darknet-53, weights from yolov3 authors |

## Tricky Operations

Some tricky operations encountered in these models, already solved, but might have better solutions.

|Name | Description |
|-|-|
|BatchNorm| Implement by a scale layer, used in resnet, googlenet, mobilenet, etc. |
|MaxPool2d(ceil_mode=True)| use a padding layer before maxpool to solve ceil_mode=True, see googlenet. |
|average pool with padding| use setAverageCountExcludesPadding() when necessary, see inception. |
|relu6| use `Relu6(x) = Relu(x) - Relu(x-6)`, see mobilenet. |
|torch.chunk()| implement the 'chunk(2, dim=C)' by tensorrt plugin, see shufflenet. |
|channel shuffle| use two shuffle layers to implement `channel_shuffle`, see shufflenet. |
|adaptive pool| use fixed input dimension, and use regular average pooling, see shufflenet. |
|leaky relu| I wrote a leaky relu plugin, but PRelu in `NvInferPlugin.h` can be used, see yolov3. |
|yolo layer| yolo layer is implemented as a plugin, see yolov3. |
|upsample| replaced by a deconvolution layer, see yolov3. |
|hsigmoid| hard sigmoid is implemented as a plugin, hsigmoid and hswish are used in mobilenetv3 |

