---
layout:     post
title:      FaceSwap深度学习AI实现视频换脸详解
subtitle:   FaceSwap 实战详解
date:       2019-10-30
author:     FormWeb
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - FaceSwap
    - AI
    - 
--- 

给大家介绍最近超级火的黑科技应用deepfake，这是一个实现图片和视频换脸的app。前段时间神奇女侠加尔盖朵的脸被换到了爱情动作片上，2233。我们这里将会从github项目faceswap开始一步一步实现一个视频换脸的教程。

##### 注意：

本技术存在一定的使用风险，本教程仅做技术交流，请不要用在其他不应该被使用的地方。

# 技术分析

faceswap 项目是一个学习重建脸部特征的深度学习算法。你给它一堆的图片，它学习几个小时后，通过分辨哪些是合成的图片最终。但是它并不是仅仅把相似的图片替换，而是通过对不同的表情进行学习分析，最后可以对给出的图片进行模型替换达到以假乱真的效果。详细分析可参考Reddit。

假设你可以连续12小时盯着一个人看，观察他的表情并记在大脑里面。然后他跟你说画个我微笑的样子，悲伤的样子，你瞬间就可以完成这个要求并且能够达到高清的图片效果。这就是深度学习的威力。

 

换脸针对训练的是脸部特征，因此对于输入的图片应该也是只有脸部的图片，所以如果是非脸部的图片需要提取脸部再来进行替换，这也是后面一开始出错的原因。

 

# AI换脸详细步骤

拷贝项目到本地并开始训练模型

首先需要安装git，使用git clone拷贝faceswap项目到本地。从本站上传的文件下载测试数据，faceswap项目地址https://github.com/deepfakes/faceswap

 
```
git clone https://github.com/deepfakes/faceswap
cd faceswap
pip install -r requirements.txt
wget https://www.bobobk.com/wp-content/uploads/2019/01/input.zip
unzip input.zip
```
 

解压后可以看到目录下有了input_A和input_B训练数据文件夹。

由于训练耗时较长，如果GPU不强的话会更久，建议使用screen运行，本教程训练使用GTX1080ti训练。代码如下：
```
screen -S faceswap
python faceswap.py train
模型将会保存在model文件夹下。这里我运行到损失函数小于0.03就没有继续训练了。
```
 

# 输入文件准备

模型训练好了就是大家喜闻乐见的换脸环节了，由于我们的测试数据是川普和尼古拉斯凯奇，所以首先我们找一张最新的一小段视频。视频来源https://www.youtube.com/watch?v=S73swRzxs8Y，由于模型只是用来进行图片替换的，这里需要首先将视频转换为图片，项目文件中toos.py可以进行此项工作，用法：

 
```
mkdir input 
mkdir output

python tools.py effmpeg  -i cage.mp4 -o input -s 00:0148 -e 00:0156

####视频1分48秒可以到1分56秒结束。
```
 

运行失败，为什么，因为没有安装FFmpeg，其中一个功能就是将视频里面的图片提取出来，图片合成会视频。安装可看https://cloud.tencent.com/developer/article/1027379，这里直接给出centos7安装代码

```
yum install -y vim
yum install -y epel-release
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
yum repolist
rpm --import http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro
rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-1.el7.nux.noarch.rpm
yum repolist
yum update -y
yum install -y ffmpeg
ffmpeg -version
```
　

安装后就可以了,切割后的图片存在于input文件夹中，接下来就是转换了。

运行报错　　

```
python faceswap.py convert -i original -o modified
```

原因是没有比对的文件，想想也是，我们的输入是一张人脸图片，给的确实一整个电影的一帧，比对就是先把脸部给提取出来以用来替换，所以这里需要先做脸部比对。这在项目里就是使用extract命令（一开始我以为extract是解压文件呢，233333）

把前面从视频中提取的图片文件进行脸部提取，使用方法

```
python faceswap.py extract -i original -o input
```

运行训练好的模型换脸

继续运行convert命令
```
python faceswap.py convert -i input -o output
```
可以看到已经在逐步输出换脸后的图片，想想还有点小激动呢。



 

 

将换脸后的图片重新合成回视频

代码：
```
python tools.py effmpeg  -i output -r input/cage.mp4 -o modified/cage.mp4 -a gen-vid
```
生成的mp4在modified文件夹下，下载播放即可。播放时还真的是乐趣多多。

# 应用

该技术自开发一来就有各种声音，恐惧其威力，想想别人使用你的样子发表一段根本不是你自己录制的视频是什么情景。

个人认为这是一个非常好玩的东西，你可以把自己换到自己喜欢的角色身上，比如哈利波特什么的啊。想想就很有意思。像这篇文章作者把妻子换到节目主持人身上，他妻子得多激动，啊哈哈哈。

# 总结

本文一步一步地从模型训练，到视频文件的处理，再到图片换脸并重新根据原始视频合成生成的视频。是一个完整的使用faceswap 深度学习AI实现视频换脸教程，理论上可以运用于任何的视频和想替换的人脸，只需要给与足够的目标模型和训练模型的图片即可，这也是很多明星被换到小电影上的原因。

这项换脸技术相信会在未来产生大量好玩的视频，应用，敬请期待！！！

另：由于使用的是深度学习，在训练模型上需要耗费大量的计算资源，因此最好使用GU或者TPU训练较好。

 

[原文链接](https://www.bobobk.com/258.html)


