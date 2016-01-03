---
layout:     post
title:      "Windows上绑定主机目录到容器内卷的问题"
subtitle:   "主机与VM之间的目录共享及-v绑定语法"
date:       2016-01-03 23:00:00 +0800
author:     "颜俊标"
header-img: "img/post-bg-2015.jpg"
tags:
    - Docker
    - Windows
    - Container
    - VirtualBox
---
这几天在我的Windows 10上使用[Jekyll in Docker](https://github.com/jekyll/docker)来维护我的博客时，遇到个问题，花了蛮多时间解决。
在这里和大家分享下,避免走一些弯路。

我的Windows上根据Docker官方网站的说明安装了[Docker Toolbox 1.9.1](https://docs.docker.com/windows/step_one/)。

然后在Terminal里运行Jekyll

```
docker run --rm --label=jekyll --volume=$(pwd):/srv/jekyll \
  -it -p $(docker-machine ip `docker-machine active`):4000:4000 \  
    jekyll/jekyll:pages    
```

出现了第一个问题：

```
invalid value "D:\\HyCloud\\byrne-yan.github.io;C:\\Program Files\\Git\\srv\\jekyll" for flag --volume: bad mode specified: \Program Files\Git\srv\jekyll
See 'C:\Program Files\Docker Toolbox\docker.exe run --help'.
```

在网上搜索了一些资料，发现Docker的官方Issues提到了这方面的问题：[Issue 80](https://github.com/docker/toolbox/issues/80)。里面提供一
个解决方案：Windows上的主机目录要以//开头。

我按照这个方法给了参数：

```
docker run --rm --label=jekyll --volume=/$(pwd):/srv/jekyll \
  -it -p $(docker-machine ip `docker-machine active`):4000:4000 \  
    jekyll/jekyll:pages 
```

前面的问题消失了，但出现了另外一个问题：

```
Configuration file: none
            Source: /srv/jekyll
       Destination: /srv/jekyll/_site
      Generating...
                    done.
 Auto-regeneration: enabled for '/srv/jekyll'
Configuration file: none
jekyll 2.4.0 | Error:  Permission denied @ dir_s_mkdir - /srv/jekyll/_site
ok: down: /etc/startup3.d/nginx: 0s, normally up
sh: can't kill pid 20: No such process
```
从字面上看是没有权限创建_site目录，但是同一个Blog源码工程，在Linux下用Jekyll in Docker跑没有这个问题。可以排除是Jekyll in Docker的配置问题。
在网上查找相关资料，有很多这方面的帖子，包括[Jekyll官方Issue #31](https://github.com/jekyll/docker/issues/31)，
一个[日本的帖子](http://qiita.com/hidekuro/items/aa83583b20db5a6857d8)。我按照这些方法都都没有解决此问题。

花了很多时间查资料，实验各种方法，都没有成功。最后在仔细研读了Docker官方文档关于卷部分的资料后，终于看到了一段话：
> If you are using Docker Machine on Mac or Windows, your Docker daemon has only limited access to your OS X or Windows filesystem. Docker Machine tries to auto-share your /Users (OS X) or C:\Users (Windows) directory. So, you can mount files or directories on OS X using.
  
> `docker run -v /Users/<path>:/<container path> ...`
>  On Windows, mount directories using:
>  
>  `docker run -v /c/Users/<path>:/<container path> ...`
>
> All other paths come from your virtual machine’s filesystem. For example, if you are using VirtualBox some other folder 
  available for sharing, you need to do additional work. In the case of VirtualBox you need to make the host folder 
  available as a shared folder in VirtualBox. Then, you can mount it using the Docker -v flag.

根据文档，Windows下，Docker Toolbox缺省使用VirtualBox作为VM，VM只能有限使用主机的目录。缺省情况下，只有c:/Users目录作为主机和VM间的共享目录。
而我的工程目录在D盘，VM没有权限访问它。

至此，有两种方法解决这个问题：

1. 将工程目录移到c:/Users下
2. 将工程目录做主机和VM间的共享目录，具体方法参见[VirtualBox指南：4.3. Shared folders](http://www.virtualbox.org/manual/ch04.html#sf_mount_manual)
    1. 在Docker Toolbox的Terminal里运行`docker-machine stop default`停止VM
    2. 启动一个Windows命令窗口，进入VirtualBox安装目录并运行`C:\Program Files\Oracle\VirtualBox>VBoxManage.exe sharedfolder add default --name d/HyCloud --hostpath "D:\HyCloud" --automount`
    3. 在Docker Toolbox的Terminal里运行`docker-machine start default`运行VM
    4. 在VirtualBox管理器里查看日志，可以看到以下信息：  
      
        ```
        00:00:01.797856 SharedFolders host service: Adding host mapping
        00:00:01.797863     Host path 'D:\HyCloud', map name 'd/HyCloud', writable, automount=true, create_symlinks=false, missing=false
        ```
        
        说明目录已共享！
    5. 在Docker Toolbox的Terminal里运行`docker-machine ssh default 'sudo mkdir -p /d/HyCloud && sudo mount -t vboxsf d/HyCloud /d/HyCloud'`
    挂载此共享目录（照理应该像c:\Users一样能自动挂载，但我实际使用时发现没有自动挂载，所以每次都需要运行一下挂载命令）
    