---
layout:     post
title:      "业务敏捷之docker：Android"
subtitle:   "通过docker启动Android APK加快开发与测试"
date:       2015-12-17 18:18:00 +0800
author:     "颜俊标"
header-img: "img/post-bg-2015.jpg"
tags:
    - Docker
    - Agility
    - Android
    - ARC
    - Test
---

Android应用是目前移动应用不可或缺的部分，尤其是在智能设备领域，其开源性以及众多开发者使其成为众多智能设备创业公司的首选：方便定制，众多的可检索
知识，众多的Android开发工程师可以招聘。

所以Android开发的自动化是提升敏捷的重要领域。最近检索了一些资料并实践了下。

我们知道在Web领域，自动化已经非常普及，如著名的[Selenium](http://www.seleniumhq.org/), [WebdriverIO](http://webdriver.io/)。
同时Google有个[ARC-App Runtime for Chrome](https://developer.chrome.com/apps/getstarted_arc), 我们可以利用Chrome的一个插件
[ARC Welder](https://chrome.google.com/webstore/detail/arc-welder/emfinbmielocnlhgmfkkmkngdoccbadn)在Chrome里运行测试
Android应用。

这个安装配置Chrome，ARC Welder比较麻烦，同时ARC Welder一次只允许运行一个App，对开发和测试来说很可能不够，我们需要一个更敏捷，更轻量的方法。

Docker是最佳选择。我们将配置的环境打包成Docker映象，需要时用docker从这个映像启动一个容器即可。启动速度极快，1秒内就可以启动完成。同时，也可以
在一台主机上启动任意个容器来运行不同的App。开发、测试人员可以同时运行不同版本App。

## 最简单的方式：

```
docker run -it \
        --net host \
        --cpuset-cpus 0 \
        --memory 512mb \ # max memory it can use
        -v /tmp/.X11-unix:/tmp/.X11-unix \ # mount the X11 socket
        -e DISPLAY=unix$DISPLAY \
        -v $HOME/Downloads:/root/Downloads \
        --device /dev/snd \ # so we have sound
        --name arcwelder \
        arc-welder:latest
```
注意：我在实践中遇到找不到显示设备的问题，你也可能遇到。我实践的环境是Ubuntu vivid, 15.04。下面是我的解决方法，
```
sudo apt-get install x11-xserver-utils

xhost +
```
这里的原理是，容器使用主机的X11服务，需要授权设置允许所有用户访问X11。

另外，在实践中还碰到一个问题：声音设备找不到。经过检索调查发现是docker版本太低，docker升级的最新版（1.9.1）问题就消失了。

Ubuntu上升级Docker的方法参见[在Ubuntu升级Docker的方法](http://blog.csdn.net/delphiwcdj/article/details/42836423)



我用这个Docker跑了愤怒的小鸟，我自己开发的[ShareBJ](www.hy-cloud.info), 没发现问题。

## 定制Image
如果arc-welder映像无法满足你的需要，你可以可以参照[Docker-ARC-Welder](https://github.com/tommyoshaw/arc-welder)制作自己的映像文件。

## 参考
[Docker-ARC-Welder](https://github.com/tommyoshaw/arc-welder)
[在Ubuntu升级Docker的方法](http://blog.csdn.net/delphiwcdj/article/details/42836423)
[Selenium](http://www.seleniumhq.org/)
[WebdriverIO](http://webdriver.io/)
[ARC-App Runtime for Chrome](https://developer.chrome.com/apps/getstarted_arc)
[ARC Welder](https://chrome.google.com/webstore/detail/arc-welder/emfinbmielocnlhgmfkkmkngdoccbadn)
