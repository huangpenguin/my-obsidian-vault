---
title: "how to understand codes by others"
publish: false
tags: ["机器学习"]
---
# how to understand codes by others

如果是看一个陌生的大项目，我会尝试从目标导向，画流程图的方式帮助自己理解代码。这里的流程图，就是类似我们看论文时的“overview of the framework”, 包含了input, output，以及中间的一系列operator（主要是训练时的forward）。

第一步先列一个粗糙的骨架： raw data ——> preprocessing ——> input ——> network ——> output。

第二步，把代码文件对应上去, 看各个.py分别实现了哪一步。如果这个代码的命名比较规范，这一步应该不难。比如，dataloader.py 对应的是 raw data——>input的这个部分，model.py 是network的部分, train.py或者main.py 是input——>network ——> output。 (一般我们先找train.py或者main.py)

第三步，读每个.py，进一步细化这个框架。以 dataloader和 preprocessing为例：

- 先弄明白这一步的input和output是什么，比如，“把机器采集的信号（raw data/ input）——>处理成质量好的图像输出（output），作为下一步[神经网络](https://zhida.zhihu.com/search?content_id=216441132&content_type=Article&match_order=1&q=%E7%A5%9E%E7%BB%8F%E7%BD%91%E7%BB%9C&zhida_source=entity)的输入”。
- 那么，知道了输入输出，这一步的目标就知道了，根据具体任务，把“质量好”再明确一下，比如低噪声、高对比度之类的。那么这些中间步骤，一定都是为了这个目标而服务的。
- 然后，看具体的代码，了解各个代码块大概是实现什么功能（这个时候不要纠结于读懂每一行是啥意思，看每个调用的函数是啥功能就行了）。然后画出预处理这个步骤的流程图，比如：raw data——> 读取目录里的文件 ——> 数据格式转换 ——> 降噪 / 增强 / 旋转/ 融合 / 其它神奇预处理balabala——> image 。遇到搞不懂的大段代码，继续重复“输入输出-目标-拆解”，从“这个代码块目的可能是实现什么功能”的角度来帮助理解。
- 最后，重复这个步骤，直到把整个代码框架填完。最终到“明白每个函数对应framework里的哪个功能”就够了。
