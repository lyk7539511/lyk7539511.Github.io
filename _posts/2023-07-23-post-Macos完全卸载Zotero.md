---
title: "Macos彻底删除Zotero"
categories:
  - Blog
  - Tech
tags:
  - Macos
  - Zotero
---

## 1. 删除Zotero.app

```bash
rm -rf /Applications/Zotero.app
```

## 2. 删除Zotero的配置文件

这一步很重要，如果不删除配置文件，重新安装Zotero时，会出现之前的配置，比如之前的插件。

```bash
rm -rf ~/Library/Application\ Support/Zotero
```