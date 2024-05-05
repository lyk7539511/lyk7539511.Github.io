---
title: "MotionBERT部署"
categories:
  - Blog
  - Tech
tags:
  - MotionBERT
---

# 3D骨架

# 1 硬件與軟件環境：

CPU: i5 10400F

Memory: 16G

SSD: 512G

GPU: NVIDIA 3080Ti 12G

OS: Windows 10 21H2

NVIDIA Driver: 531.79

# 2 環境配置過程：

## 2.1 Conda

[https://repo.anaconda.com/miniconda/Miniconda3-py310_23.5.2-0-Windows-x86_64.exe](https://repo.anaconda.com/miniconda/Miniconda3-py310_23.5.2-0-Windows-x86_64.exe)

下載之後按照步驟安裝就好

在Start菜單打開Prompt，分別運行以下兩條命令：

```bash
conda create -n alphapose python=3.7
conda create -n motionbert python=3.7 anaconda
```

## 2.2 安裝AlphaPose

克隆倉庫

[https://github.com/MVIG-SJTU/AlphaPose](https://github.com/MVIG-SJTU/AlphaPose#quick-start)

下載安裝Visual Studio（2015+）

[https://visualstudio.microsoft.com/downloads/](https://visualstudio.microsoft.com/downloads/)

在Prompt裏面運行下面的命令：

```bash
conda activate alphapose
pip install --upgrade pip
pip install torch==1.12.0+cu113 torchvision==0.13.0+cu113 torchaudio==0.12.0 --extra-index-url https://download.pytorch.org/whl/cu113
pip install cython
# 切換至AlphaPose目錄，運行
python setup.py build develop --user
```

安裝完成後將/blob/master/setup.py文件124行的False修改為True

```python
force_compile = True
```

下載模型文件

1、[https://drive.google.com/open?id=1D47msNOOiJKvPOXlnpyzdKA3k6E97NTC](https://drive.google.com/open?id=1D47msNOOiJKvPOXlnpyzdKA3k6E97NTC)

將該文件放在detector/yolo/data目錄

2、https://github.com/Megvii-BaseDetection/YOLOX/releases/download/0.1.1rc0/yolox_l.pth

將該文件放在detector/yolox/data目錄

3、https://drive.google.com/file/d/1S-ROA28de-1zvLv-hVfPFJ5tFBYOSITb/view?usp=sharing

將該文件放在pretrained_models目錄

4、https://drive.google.com/file/d/1myNKfr2cXqiHZVXaaG8ZAq_U2UpeOLfG/view?usp=share_link

將該文件放在trackers/weights目錄

移動文件

將scripts/demo_inference.py文件移動至倉庫根目錄

## 2.3 安裝MotionBERT

克隆倉庫

[https://github.com/Walter0807/MotionBERT](https://github.com/Walter0807/MotionBERT)

在Prompt裏面運行以下命令

```python
conda activate motionbert
conda install pytorch torchvision torchaudio pytorch-cuda=11.6 -c pytorch -c nvidia
# 切換到MotionBERT目錄，運行
pip install -r requirements.txt
```

修改infer_wild.py中41行的testloader_params定義，原代碼中的定義適用於多GPU的情況，需要修改為單GPU的配置：

```python
testloader_params = {
          'batch_size': 1,
          'shuffle': False,
          'num_workers': 0,
          'pin_memory': True,
          'persistent_workers': False,
          'drop_last': False
}
```

下載模型文件https://1drv.ms/f/s!AvAdh0LSjEOlgT67igq_cIoYvO2y?e=bfEc73，將該文件放在checkpoint/pose3d/FT_MB_lite_MB_ft_h36m_global_lite/目錄中

# 3 推理運行

## 3.1 文件準備

錄製視頻，mp4文件經過測試可以運行。（僅限單人！）

假設文件放置在demo/demo.mp4，AlphaPose、MotionBERT與demo目錄同級

demo目錄下建立2D、3D文件夾

打開Prompt，進入demo、AlphaPose、MotionBERT根目錄。

## 3.2 2D骨架

```python
conda activate alphapose
cd AlphaPose
python demo_inference.py --cfg configs/halpe_26/resnet/256x192_res50_lr1e-3_1x.yaml --checkpoint pretrained_models/halpe26_fast_res50_256x192.pth --video ../demo/demo.mp4 --outdir ../demo/2D --save_video --gpus 0
```

## 3.3 3D骨架

```python
conda activate motionbert
cd MotionBERT
python infer_wild.py --vid_path ../demo/demo.mp4 --json_path ../demo/2D/alphapose-results.json --out_path ../demo/3D
```

## 3.4 推理結果

[![Screenshot_2023-07-14_at_6.39.31_PM.png](https://img2.imgtp.com/2024/05/05/0FNoYNyp.png)](https://img2.imgtp.com/2024/05/05/0FNoYNyp.png)