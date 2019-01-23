---
layout:     post
title:      Ubuntu 18.04 Install Vivado Cable Driver
subtitle:   
date:       2019-01-23
author:     Earl
header-img: img/post-bg-new-4.jpg
catalog: true
tags:
    - Software Tool
---

> Ubuntu 18.04 安装vivado时默认不安装vivado cable driver，因此需要手动安装一下。

# linux安装Vivado默认无法安装JTAG驱动

注意：以下操作必须在root权限下进行。

1. 找到install_script目录

	/usr/local/app/Xilinx/Vivado/2018.3/data/xicom/cable_drivers/lin64/install_script/install_drivers

2. 安装驱动
	./install_drivers
