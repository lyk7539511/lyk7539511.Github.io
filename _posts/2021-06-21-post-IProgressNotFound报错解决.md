---
title: "ImportError: IProgress not found. 报错解决"
categories:
  - Blog
  - Tech
tags:
  - Conda
  - Python
  - Pytorch
  - YOLOv5
  - Jupyter NoteBook
  - error fix

---

使用 github 开源 [YOLOv5](https://github.com/ultralytics/yolov5)，安装完成，跑测试，出现以下错误

```python
ImportError: IProgress not found. Please update jupyter and ipywidgets. 
See https://ipywidgets.readthedocs.io/en/stable/user_install.html
```

## 出错原因

安装 Jupyter Notebook 的时候没有安装 [ipywidgets](https://pypi.org/project/ipywidgets/) 包，手动使用pip安装即可，安装完成之后重启Notebook。

## 解决

```bash
pip install ipywidgets
```

## 根本原因是 [tqdm](https://pypi.org/project/tqdm/)(用来生成进度条) 依赖 ipywidgets 包