---
layout: post
title: "解决Ubuntu下Sublime Text 3无法输入中文"
subtitle: "转载"
＃date: 2014-10-11 18:28:48 +0800
author: "颜俊标"
header-img: "img/post-bg-2015.jpg"
tags:
    - ubuntu
    - sublime text
    - 中文输入
---

在我的ubuntu机器上安装sublime text后发现无法输入中文，检索后找到个解决方法：

使用[sublime-text-imfix](https://github.com/lyfeyaj/sublime-text-imfix)

```
git clone https://github.com/lyfeyaj/sublime-text-imfix.git
cd sublime-text-imfix && ./sublime-imfix
```


## 注意：　

`必须在terminal下运行subl`才有效。

通过桌面启动是无效的，如果想通过桌面启动sublime text依然可以输入中文，需要修改桌面快捷方式，请参考Vaayne的原文：

[解决Ubuntu下Sublime Text 3无法输入中文](http://www.jianshu.com/p/bf05fb3a4709)