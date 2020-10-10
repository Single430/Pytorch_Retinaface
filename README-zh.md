
# 人脸检测-RetinaFace in Pytorch

用 [PyTorch](https://pytorch.org/) 实现论文 [RetinaFace: Single-stage Dense Face Localisation in the Wild](https://arxiv.org/abs/1905.00641)。模型大小仅仅1.7M，**Retinaface**使用**Mobilenet0.25**作为主干网络。我们也提供了使用**Resnet50**作为主干网络的版本并获得了更好的结果。这个官方的代码在[MxNet](https://github.com/deepinsight/insightface/tree/master/RetinaFace)中可以查看。

## 移动端 or Windows or Linux or 嵌入式设备部署
我们提供使用python 训练并使用C++推理的[代码](https://github.com/biubug6/Face-Detector-1MB-with-landmark)，可以在edge device设备中运行面部检测。

## 使用Resnet50作为骨干网时，在单尺寸的WiderFace 验证集上性能对比
| Style | easy | medium | hard |
|:-|:-:|:-:|:-:|
| Pytorch (same parameter with Mxnet) | 94.82 % | 93.84% | 89.60% |
| Pytorch (original image scale) | 95.48% | 94.04% | 84.43% |
| Mxnet | 94.86% | 93.87% | 88.33% |
| Mxnet(original image scale) | 94.97% | 93.89% | 82.27% |

## 使用Mobilenet0.25作为骨干网时，在单尺寸的WiderFace 验证集上性能对比
| Style | easy | medium | hard |
|:-|:-:|:-:|:-:|
| Pytorch (same parameter with Mxnet) | 88.67% | 87.09% | 80.99% |
| Pytorch (original image scale) | 90.70% | 88.16% | 73.82% |
| Mxnet | 88.72% | 86.97% | 79.19% |
| Mxnet(original image scale) | 89.58% | 87.11% | 69.12% |
<p align="center"><img src="curve/Widerface.jpg" width="640"\></p>

## FDDB(人脸检测算法评价标准) 性能.
| FDDB(pytorch) | performance |
|:-|:-:|
| Mobilenet0.25 | 98.64% |
| Resnet50 | 99.22% |
<p align="center"><img src="curve/FDDB.png" width="640"\></p>

### 内容
- [安装](#installation)
- [训练](#training)
- [评价](#evaluation)
- [TensorRT加速](#tensorrt)
- [参考文献](#references)

## 安装
##### 克隆代码并安装
1. ``git clone https://github.com/biubug6/Pytorch_Retinaface.git``

2. Pytorch version 1.1.0+ and torchvision 0.3.0+ are needed.

3. 代码基于 Python3

##### 数据1
1. 下载 [WIDERFACE](http://shuoyang1213.me/WIDERFACE/WiderFace_Results.html) 数据集.

2. 从 [baidu cloud](https://pan.baidu.com/s/1Laby0EctfuJGgGMgRRgykA) or [dropbox](https://www.dropbox.com/s/7j70r3eeepe4r2g/retinaface_gt_v1.1.zip?dl=0) 下载标注数据（人脸标注框和5个面部关键点）

3. 数据目录结构如下:

```Shell
  ./data/widerface/
    train/
      images/
      label.txt
    val/
      images/
      wider_val.txt
```
ps: wider_val.txt 只包括验证集的文件名不包括标注信息.

##### 数据2
我们也提供官方的数据集，目录结构同上

链接: from [谷歌盘](https://drive.google.com/open?id=11UGV3nbVv1x9IC--_tK3Uxf7hA6rlbsS) or [百度盘](https://pan.baidu.com/s/1jIp9t30oYivrAvrgUgIoLQ) **密码: ruck**

## 训练
我们使用**restnet50**和**mobilenet0.25**作为主干网络来训练模型。
使用**Mobilenet0.25**作为主干网络，并在**ImageNet数据集**上训练 and get 46.58% in Top 1(意思应该是: ImageNet数据集有上千类，我们只看概率最高的那个结果正确性，准确率为46.58%)，如果你不希望重新训练模型，我们也提供了训练模型。预训练模型和微调模型都可以从[谷歌盘](https://drive.google.com/open?id=1oZRSG0ZegbVkVwUd8wUIQx8W7yfZ_ki1)和[百度盘](https://pan.baidu.com/s/12h97Fy1RYuqMMIV-RpzdPg)下载，密码：**fstq**。

下载文件的目录结构如下：
```Shell
  ./weights/
      mobilenet0.25_Final.pth
      mobilenetV1X0.25_pretrain.tar
      Resnet50_Final.pth
```
1. 训练前，可以检查配置文件 (e.g. batch_size, min_sizes and steps etc..) in ``data/config.py and train.py``.

2. 使用**WIDER FACE**数据集训练模型:
  ```Shell
  CUDA_VISIBLE_DEVICES=0,1,2,3 python train.py --network resnet50 or
  CUDA_VISIBLE_DEVICES=0 python train.py --network mobile0.25
  ```


## 评价
### **widerface**数据集做评价
1. 生成txt文件
```Shell
python test_widerface.py --trained_model weight_file --network mobile0.25 or resnet50
```
2. 评估结果。[例子可以看这里](https://github.com/wondervictor/WiderFace-Evaluation)
```Shell
cd ./widerface_evaluate
python setup.py build_ext --inplace
python evaluation.py
```
3. [官方]You can also use widerface official Matlab evaluate demo in [Here](http://mmlab.ie.cuhk.edu.hk/projects/WIDERFace/WiderFace_Results.html)
### 使用**FDDB**评估

1. 下载图片集 [FDDB](https://drive.google.com/open?id=17t4WULUDgZgiSy5kpCax4aooyPaz3GQH) to:
```Shell
./data/FDDB/images/
```

2. 使用下面代码评估训练模型:
```Shell
python test_fddb.py --trained_model weight_file --network mobile0.25 or resnet50
```

3. 下载 [eval_tool](https://bitbucket.org/marcopede/face-eval) 工具来评估性能。

<p align="center"><img src="curve/1.jpg" width="640"\></p>

## TensorRT加速
-[TensorRT](https://github.com/wang-xinyu/tensorrtx/tree/master/retinaface)

## 参考文献
- [FaceBoxes](https://github.com/zisianw/FaceBoxes.PyTorch)
- [Retinaface (mxnet)](https://github.com/deepinsight/insightface/tree/master/RetinaFace)
```
@inproceedings{deng2019retinaface,
title={RetinaFace: Single-stage Dense Face Localisation in the Wild},
author={Deng, Jiankang and Guo, Jia and Yuxiang, Zhou and Jinke Yu and Irene Kotsia and Zafeiriou, Stefanos},
booktitle={arxiv},
year={2019}
```
