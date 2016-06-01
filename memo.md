# 阅读摘要

> 技术领域的信息摘要

## 五个的深度学习框架

DeepMind宣布采用谷歌开源的深度学习框架TensorFlow，不再采用Torch框架。Torch 诞生时间较久，直到去年Facebook 开源了大量Torch的深度学习模块才开始流行起来。

DeepMind是谷歌并购的一家AI公司，今年因AlphaGo以4:1的成绩战胜了韩国围棋大师李世石而名声大噪。除此以外，谷歌还有规模更大的Google Brain团队。

Caffe。源自加州伯克利分校的Caffe，由C++开发。雅虎今年2月份开源的CaffeOnSpark, 就是基于 Caffe，还有能够优化迭代工作负载的数据运算系统 Spark （它是对 Hadoop 的补充，可以在 Hadoop 文件系统中并行运行）。雅虎所做的只是创建了一个可以在Spark集群上运行Caffee的方法。它可以单独在Spark上运行，或者在 Hadoop上。Feng说，除了让AI开发人员更方便的使用相似工具、避免来回移动数据外，CaffeOnSpark还将在众多服务器中分发深度学习进程变得相对容易，而这正是谷歌开源版本的TensorFlow做不到的。

Deeplearning4j。是”for Java”的深度学习框架，由创业公司Skymind于2014 年6月发布，可与Hadoop和Spark集成，即插即用，方便开发者在APP中快速集成深度学习功能。该学习框架成熟度较高，可以直接面向商用。

Brainstorm。来自瑞士人工智能实验室IDSIA 的一个深度学习软件包，Brainstorm 能够处理上百层的超级深度神经网络——Highway Networks。

其他还有Theano、Chainer、Marvin、Neon、ConvNetJS等都出自创业公司、AI个人爱好者或大学项目组，未来被广泛应用的可能性较小。