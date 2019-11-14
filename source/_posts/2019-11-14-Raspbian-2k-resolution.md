---
title: 树莓派外接2560x1440 2k屏设置
date: 2019-11-14 09:55:51
category: learn
tags:
    - Raspbian
---

> [官网的Raspbian系统 config.txt 配置详解]( https://www.raspberrypi.org/documentation/configuration/config-txt/ )

烧录完系统，将tf卡插入到树莓派中，再启动设备。接入2k屏幕，显示的一直是黑屏，需要将系统的配置文件修改下。



<!-- more -->

[Raspbian 显示模式设置]( https://www.raspberrypi.org/documentation/configuration/config-txt/video.md )

![video config](2.png)

在 `/boot/config.txt` 中添加2k分辨率的配置。

```text
start_x=0
hdmi_drive=1
hdmi_group=2
hdmi_mode=87
hdmi_cvt=2560 1440 60

// 下面可加不加
framebuffer_width=2560
framebuffer_height=1440
max_framebuffer_width=2560
max_framebuffer_height=1440
```



![hdmi_cvt配置](1.png)



将卡插回到树莓派中，然后再启动树莓派，就可以显示在2k屏幕上了。

