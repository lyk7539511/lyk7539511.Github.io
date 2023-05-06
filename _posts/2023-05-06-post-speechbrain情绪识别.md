---
title: "利用speechbrain和openai分别实现语音和文字的情绪识别"
categories:
  - Blog
  - Tech
tags:
  - SpeechBrain
  - Emotion
  - OpenAI
---

# 环境需求

## 硬件
Nvidia GPU
Drive 50G
Memory 16G

## 系统
Ubuntu 20.04 LTS Server
CUDA 11.4

请确认已安装好CUDA驱动，可以用下面这个命令
```bash
ubuntu@VM-0-12-ubuntu:~$ nvidia-smi
Sat May  6 09:13:09 2023       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 470.82.01    Driver Version: 470.82.01    CUDA Version: 11.4     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Tesla T4            On   | 00000000:00:08.0 Off |                  Off |
| N/A   27C    P8     9W /  70W |      0MiB / 16127MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```
如果没有出现以上结果请去NVIDIA官网下载并安装对应版本的驱动，并按照指引安装。注意，尽量不要使用带有GUI的ubuntu，有一定几率会出现驱动冲突从而无法正常显示。
CUDA网址：[https://developer.nvidia.com/cuda-toolkit-archive](https://developer.nvidia.com/cuda-toolkit-archive)

# 配置过程

## 1、安装Conda
由于是服务端，所以安装轻量版无GUI的MiniConda
```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh

Do you accept the license terms? [yes|no]
[no] >>> yes

Do you wish the installer to initialize Miniconda3
by running conda init? [yes|no]
[no] >>> yes

#  其他选项默认，不用sudo
```

安装完成后
```bash
ubuntu@VM-0-12-ubuntu:~$ source .bashrc 
(base) ubuntu@VM-0-12-ubuntu:~$
```

出现（base）代表安装成功

## 2、准备代码与模型文件，准备conda环境
下载 speechbrain-restful.tar 文件，并上传至～/目录

解压 speechbrain-restful.tar 到 ～/
```bash
(base) ubuntu@VM-0-12-ubuntu:~$ tar -xvf speechbrain-restful.tar
(base) ubuntu@VM-0-12-ubuntu:~$ ls speechbrain/
audio_cache  docs            InferText.py  lint-requirements.txt  pretrained_models  pyproject.toml  README.md  requirements.txt  results   speechbrain  tests
conftest.py  InferSpeech.py  LICENSE       pretrained             __pycache__        pytest.ini      recipes    restfulTest.py    setup.py  templates    tools
```

## 3、准备Conda环境

下载 emotion_conda23.3.1.tar.gz 文件，并上传至～/目录

解压 emotion_conda23.3.1.tar.gz
```bash
(base) ubuntu@VM-0-12-ubuntu:~$ mkdir emotion
(base) ubuntu@VM-0-12-ubuntu:~$ mv emotion_conda23.3.1.tar.gz emotion
(base) ubuntu@VM-0-12-ubuntu:~$ cd emotion
(base) ubuntu@VM-0-12-ubuntu:~/emotion$ tar -zxvf emotion_conda23.3.1.tar.gz
```
将 ～/emotion 移动到 ～/miniconda3/envs/
```bash
(base) ubuntu@VM-0-12-ubuntu:~$ mv emotion miniconda3/envs/
(base) ubuntu@VM-0-12-ubuntu:~$ cd miniconda3/envs/
(base) ubuntu@VM-0-12-ubuntu:~/miniconda3/envs$ ls
emotion
```
激活环境，确认环境迁移正确
```bash
(base) ubuntu@VM-0-12-ubuntu:~$ source miniconda3/envs/emotion/bin/activate 
(emotion) ubuntu@VM-0-12-ubuntu:~$ conda-unpack
(emotion) ubuntu@VM-0-12-ubuntu:~$ conda deactivate
(base) ubuntu@VM-0-12-ubuntu:~$ conda env list
# conda environments:
#
base                  *  /home/ubuntu/miniconda3
emotion                  /home/ubuntu/miniconda3/envs/emotion
```
至此，所有内容都已准备完成

# 4、测试

要记得修改InferText.py中的url地址，这个url是使用 [https://github.com/cookeem/chatgpt-service](https://github.com/cookeem/chatgpt-service) 搭建的openai服务，参考这个仓库的教程搭建好服务，修改ip地址即可。

```bash
(base) ubuntu@VM-0-12-ubuntu:~$ conda activate emotion
(emotion) ubuntu@VM-0-12-ubuntu:~$ cd speechbrain/
(emotion) ubuntu@VM-0-12-ubuntu:~/speechbrain$ ls
audio_cache  docs            InferText.py  lint-requirements.txt  pretrained_models  pyproject.toml  README.md  requirements.txt  results   speechbrain  tests
conftest.py  InferSpeech.py  LICENSE       pretrained             __pycache__        pytest.ini      recipes    restfulTest.py    setup.py  templates    tools
(emotion) ubuntu@VM-0-12-ubuntu:~/speechbrain$ python restfulTest.py 
 * Serving Flask app 'restfulTest'
 * Debug mode: on
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:5000
 * Running on http://172.26.0.12:5000
Press CTRL+C to quit
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 106-610-369
```
使用postman测试均正常
![Screenshot 2023-05-06 at 10.43.56.png](https://img1.imgtp.com/2023/05/06/OamjJOTi.png)
![Screenshot 2023-05-06 at 10.44.15.png](https://img1.imgtp.com/2023/05/06/XfxUnfv5.png)