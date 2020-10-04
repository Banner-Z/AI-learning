# Stacked Hourglass模型学习与实践
<-- 注：项目仍未完成，同志正在努力。会持续更新。 -->
## 论文
[ 链接 ] : https://arxiv.org/abs/1603.06937

里面的英语有点难啃，有点地道。仔细分析了模型的设计和这些设计的原因，有内味。

[一篇不错的模型解读] : https://blog.csdn.net/shenxiaolu1984/article/details/51428392
## 概述
模型设计比较复杂，并且用了很多的连跳，很多的上采样、下采样，用于单人2D姿态估计。

模型最小的一个单元是在文章中被称为“Residual Module”的残差块，里面用了Batch Normalization和relu，这里也是连跳的开始。

然后是Hourglass子网络，这个网络分成两路进行，一路经过了下采样之后再上采样，与另一路没有上下采样的相加。这种网络的意义在于，当期进行多层嵌套时，他可以有多次的连续下采样，但每次下采样后的结果总会有一个连跳，与后期对应的上采样时的地方相加，以保留特征。因此，整个网络看起来像个沙漏。

但是这还不止，作者还来了多个Hourglass，上一个Hourglass的输入、输出和预测都被拿来相加，作为下一个Hourglass的输入。这样做的意义在于，模型有了多次预测，也就有了多个loss，这样在反向传播的时候能避免传播到最上面的层时的梯度消失。

Stacked Hourglass模型发表于2016年，据（知乎）说在提出时是强过了当时所有的模型，而且比排名第二的CPM在设计上更清晰，模型更简洁（但还是很大啊...）。
## 实践
[ github pytorch版本参考 ] :https://github.com/bearpaw/pytorch-pose

作者说是没在python3上测试过，有待考究。

搬运步骤：下载GitHub项目->下载数据集到对应的文件夹下（的image文件夹内）：mpii数据集很大（11.3G），而且需要梯子，标注的json文件GitHub项目里已经有了。

->使用ln命令指定数据集位置（相当于为某一个文件在另外一个位置建立一个同步的链接）->

直接调main.py进行训练：可以在原有模型上继续。 或 在experiments下也有对应的.sh文件可以启动main.py。

原代码可能出现的问题：

1.Bar类所在的文件夹progress需要从另一个链接的GitHub仓库里下载，放在原有的progress内。

2.model_zoo不在torchvision.models.resnet下，原文件中某处使用了这个位置调用model_zoo，这显然有问题。将model_zoo导入的路径更改为torch.utils即可（因为model_url会指定所选的model，不需担心模型对应问题）。

3.在transform中进行rotate调用时给angle了一个torch类型的参数，应该用.item()转换到double类型。

4.还有一些库版本不一致造成的问题，比如函数或类不在对应包位置上。

5.原GitHub中似乎只介绍了直接调用main文件开始训练的方法，其实更合理的方式应该为使用experiment中的sh文件。

训练过程：

1.首先测试了1个hourglass的模型，100个epoch后精度趋近80%，完全没有过拟合。

2.然后修改了sh文件，添加了epoch的传参，并使用8个hourglass的堆叠模型，200个epoch后精度趋近83%，基本没有过拟合。

未完待续。。。
