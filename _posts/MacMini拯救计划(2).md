---
title : 'Mac Mini 拯救计划(2): AI'
permalink: /posts/2024/07/mac-2/
date : 2024-07-15
tags:
    - Misc
---
## 0x0. 前传
自从我将MacMini改造为寝室里面的旁路由之后，就一直在想着开发MacMini的新功能，毕竟一个M1芯片在哪，不能只让它干一些网络分流的杂活，于是我突然想到了开源AI绘画的佼佼者 Stable Diffusion，为了色图说干就干。

## 0x1. 安装依赖
使用如下命令安装依赖
```
brew install cmake protobuf rust python@3.10 git wget
```

## 0x2. 下载StableDiffusion
由于这个服务的目的不是在MacOS上进行使用，而是暴露给局域网中其他的设备使用，所以这里选择[StableDiffusionWebui](https://github.com/AUTOMATIC1111/stable-diffusion-webui)进行部署，使用如下指令进行部署。
```
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui
cd stable-diffusion-webui
./webui.sh
```
运行完之后，命令行中会输出一个一条url表示webui的地址，在浏览器中打开即可进行绘画。
> 这里可能会报错无法导入httpx库，只需修改webui.sh，在其中添加一行pip install socksio即可

## 0x3. 下载模型
但是仅仅使用stable-diffusion自带的模型当然是不满足我们的需求的，这时就需要去网络上找别人训练好的模型，这里推荐一个模型[ghostmix](https://civitai.com/models/36520/ghostmix)，这个模型用来画人像非常好用。

为了让stable-diffusion加载这个模型，首先需要将该模型下载到本地
```
wget 'https://civitai.com/api/download/models/76907?type=Model&format=SafeTensor&size=pruned&fp=fp16'
## or use curl
curl -O ghostmix.safetensors 'https://civitai.com/api/download/models/76907?type=Model&format=SafeTensor&size=pruned&fp=fp16'
```
然后将模型移动到stable-diffusion的目录下
```
mv ghostmix.safetensors /path/to/stable-diffusion/models/Stable-diffusion/
```
这样stable-diffusion启动时就会将该模型加载进去，在webui的左上角也可以选择该模型了。

## 0x4. 修改配置
之前说过这个服务最终是为连接路由器的其他设备提供的，所以不能使用原始的命令行参数来启动，我们只需要修改根目录下的`webui-user.sh`即可。
将其中的`COMMANDLINE_ARGS`一行取消注释，然后修改参数如下：
```
# --listen 用于暴露给局域网
# --skip-torch-cuda-test 由于mac没有显卡也没有cuda核心，所以需要加上这个参数
# --port 指定端口
# --medvram 内存不足的加上的参数
export COMMANDLINE_ARGS="--listen --skip-torch-cuda-test --port 7891 --medvram"
```
> 注意，这里的参数仅仅是我个人情况配置的参数，完整的参数列表请参加[官方WIKI](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Command-Line-Arguments-and-Settings)

## 0x5. 添加开机自启动
由于是个服务，所以万一MacMini断电自启，我们并不想手动的再去开启该服务，这里可以将其包装为一个开机自启动的服务。
在stable-diffusion根目录下新建一个`start.command`文件，然后使用编辑该文件，键入如下内容
```
#! /bin/bash

cd /path/to/stable-diffusion
nohup ./webui.sh &
```
熟悉*nix命令行的应该都知道这些语句的含义，我也不再赘述，不懂的照做就行。
然后执行
```
chmod 777 start.command
```
在设置->通用->登录项中点击+号，然后选中刚刚创建的`start.command`文件即可。

## 0x6. 大功告成
到这一步基本完成了Stable-Diffusion的部署和配置，现在可以在局域网中的其他设备上测试是否可以正常使用。如下图所示，我在我的PC上可以访问到该服务并且成功的绘制出了一张图，并且通过多次的测试，一张图的生成速度大概在20-30s左右，还是相当不错的。

![](/img/MacMini拯救计划-2-Stable-Diffusion/1.png)

## 0x7. Ref
[手把手教你安装运行AI绘图Stable-Diffution-Webui（Mac OS篇](https://zhuanlan.zhihu.com/p/609577723)