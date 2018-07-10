---
layout:     post
title:      "反编译PyInstaller的可执行文件"
subtitle:   "使用pyinstxtractor.py和uncompyle反编译python可执行文件"
date:       2018-07-10 14:18:00
author:     "ly"
header-img: "img/post-bg-python.jpg"
tags:
    - Python
---

# **准备工作**

* 环境要求：与可执行文件对应的python版本

* 安装uncompyle2：[Github地址](https://github.com/gstarnberger/uncompyle)

* 下载pyinstxtractor.py：[下载地址](https://sourceforge.net/projects/pyinstallerextractor/)
[Github地址](https://github.com/countercept/python-exe-unpacker)

# **执行命令**

* python pyinstxtractor.py xxx.exe

* 找到目录out00-PYZ.pyz_extracted

* python uncompyle2 -o "输出目录"  out00-PYZ.pyz_extracted

