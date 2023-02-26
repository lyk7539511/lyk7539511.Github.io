---
title: "解决Discord无法检测到OBS虚拟摄像机"
categories:
  - Blog
  - Tech
tags:
  - Discord
  - OBS
  - Virtual Camera
---

Discord无法检测到OBS虚拟摄像机的原因在于签名问题，需要解除当前签名并重新对Discord签名即可。

```shell
# 解除当前签名
sudo codesign --remove-signature "/Applications/Discord.app/Contents/Frameworks/Discord Helper (Renderer).app/"

# 重新签名
sudo codesign --sign - "/Applications/Discord.app/Contents/Frameworks/Discord Helper (Renderer).app"
```
