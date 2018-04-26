Gibson Env介绍
==========
很高兴给大家介绍我们入选CVPR2018年的项目Gibson Environment。这是一个主要适用于机器人导航任务的模拟平台。我们在传统的图形学渲染管线基础上进行了创新，使用神经网络作为渲染引擎(Neural Network Rendering Engine)，达到了近乎真实环境的渲染效果。通过和物理引擎的融合，我们可以做到在计算机里1:1地模拟真实环境： 

![](https://raw.githubusercontent.com/StanfordVL/GibsonEnv/master/misc/ui.gif)

效果图：我们动态地模拟了斯坦福计算机系(Gates Building)一楼的真实场景，让虚拟机器人可以在其中进行探索，学习真实世界。我们在Gibson Environment里可以同时“激活”大量类似的机器人。喜欢电影黑客帝国的读者可能对这个概念并不陌生。

通过Gibson Environment，我们可以把真实的场景(例如家庭住宅，学校，办公室)虚拟化，以节约大量在真实环境训练机器人的资源。另一方面，我们可以把虚拟环境中训练出来的机器人部署到真实环境。这为实现真实的强化学习提供了有力的基础。目前Gibson Environment已经完全开源，正在Beta测试阶段。有兴趣的读者可以在网站([gibson.vision](http://gibson.vision))上使用我们的源代码。

[[项目地址]](http://gibson.vision)  [[论文]](http://gibson.vision/Gibson_CVPR2018.pdf) [[Github地址]](https://github.com/StanfordVL/GibsonEnv) [[视频介绍]](https://www.youtube.com/watch?v=KdxuZjemyjc)

## 背景
2016年起，伴随深度强化学习的兴起，计算机视觉领域的研究重心从静态图片开始转向动态的控制。大量的仿真模拟平台涌现而出(例如虚拟驾驶平台Carla, 虚拟无人机平台Airsim)。

![](https://hsto.org/web/8d6/0b2/d87/8d60b2d875ab4206a47bc2f1e19eb53e.gif)

传统机器人领域倾向于将一个复杂的任务分成感知(perception)模块和决策(planning)模块，而强化学习让我们可以端到端地学习到更复杂的控制(End-to-end Control/Sensorimotor Control)，即输入为传感器信息，直接输出控制信号。目前最前沿的强化学习算法已经在很多端到端任务上获得了的成功，例如在游戏中学会解迷宫，在不平的路面上学会行走。在自动驾驶中，从摄像头拍到的画面，我们可以直接预测方向盘的转角和油门刹车。

![](https://storage.googleapis.com/deepmind-live-cms/documents/ezgif.com-resize_8knzk3G.gif)

这样的任务无法在静态的数据集（例如ImageNet）中学习。我们需要在一个可交互式的动态环境种训练智能体。

![test](envs.png) 

这张图涵盖了目前主流的模拟环境，包括游戏类的毁灭战士（VIZdoom)，侠盗猎车（GTA），驾驶类的CARLA，物理类的Roboschool。之前提到的解迷宫、行走智能体就出自这些环境。有了这些成果，我们能不能智能体运用于实际生活中，解决驾驶、机器人行走的问题呢？事实告诉我们，部署到实际中的智能体往往会因为观测到的像素不同而导致结果不理想，甚至失灵。例如在侠盗猎车手中训练的自动驾驶汽车到了真实世界中，看到从没有见过的场景，会不幸成为马路杀手。

针对这个问题，我们设计了Gibson Environment，以解决模拟平台不够真实（photorealisitic)的问题。目前大部分的模拟平台都是基于计算机图形学的方法(例如THOR, House3D，Carla)，而使用这种方法通常很难迁移到真实环境。在我们的工作中，我们使用基于图片的渲染(IBR)方法，接合神经网络，达到了高效和真实的渲染。

Gibson Environment的名字来源于美国认知心理学之父James J. Gibson。他提出认知(perception)和行动(action)具有非常紧密的联系，婴儿需要通过主动玩耍才能学会识别各种物品。对于人工智能也是一样。Gibson Environment的科研价值在于它正是这样一个环境，让智能体可以同时学习认知和行动。


## 方法

为了渲染出的看起来更加真实的画面，计算机图形学领域主要有两条主要的技术线路，一种是通过更仔细的建模和更好的光线追踪算法来实现渲染。这种方法在电影制作中十分常见，通常需要消耗大量的计算资源和资金，不适合用于实时(real time)的模拟环境。另一种方法是直接从真实环境中采集图片，把渲染的问题定义为“视角合成”问题，即给定几个从已有的视角采集的图片，合成一个新的视角。我们采用了这种方法作为我们的渲染方法，这个方法的示意图如下：

![](method.jpg)

方法的输入是环境的3D模型（比较粗糙）和一系列视角采集到的图片。对于要渲染的任意一个视点，我们选取周围的k个视点，将每个视点的每个像素投射到3D模型上，得到一个三维点云。之后，我们对3D点云进行简单的双线性插值，得到一个初步的渲染结果。不同于常见3D模型材质渲染的方法，我们对于不同的视点选取材质的方法是自适应的（更近的视点采样更多）。在此之上为了还原更多微细节（例如植物，无法被实景扫描捕捉），我们使用一个卷积神经网络对渲染进行后处理。具体技术细节可以参考原论文。

我们项目的另一个创新是把像素级别域迁移(Pixel Level Domain Adaptation)的机制嵌入到渲染引擎当中。我们的后处理网络(f)可以让渲染看起来像真实世界中的照片，与此同时我们还训练了另外一个网络(u)，让真实世界中的图片看上去像我们的渲染。这样做简化了机器人在真实世界的部署：只需要在机器人的传感器上接入我们的网络，就像给机器人戴上了一副虚拟的“眼镜”(goggles)。

![](http://gibson.vision/public/img/figure4.jpg)

## 数据集

近年来随着实景扫描技术的进步，有大量的楼房，住宅，真实场所被扫描并保存成了虚拟档案。最初，这样的档案主要被应用于房地产网络销售。

![](http://gibson.vision/public/img/figure1.jpg)

斯坦福视觉实验室（Stanford Vision Lab）是最早将这样的数据应用于科研的实验室。在[Stanford 2D3DS](http://buildingparser.stanford.edu/dataset.html)项目中，研究者将斯坦福大学6栋主要建筑进行了扫描，并取得了一系列突破。在此之后，被应用于科研的实景扫描数据量呈指数式增长。

Gibson Environment可以模拟任何被扫描过的真实环境，这是它的一个巨大优点。你完全可以扫描自己的房子，然后用Gibson Environment为之生成一个虚拟的环境，训练你的扫地机器人。在我们CVPR18的论文中，我们收集并开源了572个建筑物(1440层)的扫描。作为现有最大的数据集，我们比同类数据集(例如matterport3D)大约一个数量级。目前我们已经[在这里](https://github.com/StanfordVL/GibsonEnv)发布了一小部分数据集作为环境Beta测试的一部分，主要的数据集将会在近期发布。

## 讨论

在文中，我们对我们的渲染做了各种测试，包括速度，和真实图像的差距，以及域迁移能否成功实现等，有兴趣的读者可以参考我们的文章。不过由于时间的限制，在CVPR的文章里我们并没有在机器人上做实验，近期我们正在进行这些实验，包括语义导航、语义建图、目标驱动的三维重建等任务。

## ROS demo

由于面向的是机器人的应用，我们集成了Gibson Env和机器人操作系统，ros的用户可以方便地使用Gibson Env作为模拟器，来模拟摄像头或者kinect输入。下图是用Gibson模拟器模拟机器人建图(mapping)的一个简单的demo。

![](https://raw.githubusercontent.com/StanfordVL/GibsonEnv/57e4b8ca08a2363f098d0c742dc35197c0866837/misc/slam.png)
