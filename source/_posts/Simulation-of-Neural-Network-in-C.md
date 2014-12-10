title: Simulation of Neural Network in C++
date: 2014-12-08 19:03:49
categories:
- Study
tags:
- Matlab
- C++
- Neural Network
---
Matlab神经网络训练及C++模拟
===

神经网络的应用领域很广泛，通过将输出与输入之间及其复杂的关系简化成了所有输入的加权之后的结果（当然不只加权，还有激励函数等），这样就免去了很复杂的数学公式。
它就像一个黑盒一般，将输入丢给它后会返回给你一个正确率极高的输出。在不要求100%正确的应用中简直是法宝-。-

我也是在研究中寻求一个4变量的未知函数苦苦挣扎无果后才转而学习并采用神经网络的。
关于如何训练神经网络并逐步确立节点权值的过程很复杂，但如果只是使用并模拟一个训练好的神经网络还是很容易理解的。

<!-- more -->

在我的研究工作中，有一个需求是将预先采集的数据进行神经网络的训练(training)，然后在实时的计算过程中利用训练完成的参数进行模拟(simulation)。
神经网络的训练Matlab中已经有了很方便的工具了，而对于如何提取参数进行模拟还需花费一些功夫。

由于当时没有找到一个简单的模拟程序，所以用一个Matrix的库写了一个两层隐含层的模拟程序，用于验证训练结果以及C++的实时模拟。
在此将完整的过程记录一下。

为了简化过程，将采用Matlab自带的测试数据集进行演示。nndatasets下有很多，选一个Function Fitting的来演示。

**Matlab版本： R2014a**

**数据集：chemical_dataset**

有两种方式训练：
* 图形化
 
 输入nftool之后会出现图形化的节目来选择输入与输出文件，还会进行自动识别。

* 代码
 
 图形化选择之后也是会生成代码，为了在C++之中进行模拟，此处选用了代码。

参照[StackOverflow上](https://stackoverflow.com/questions/15526112/export-a-neural-network-trained-with-matlab-in-other-programming-languages)的回答，Matlab会做归一化的预处理，所以必须将预处理函数取消，否则C++的模拟部分也得加上归一化。

```Matlab
inputFile = fopen('..\\bpTrainInput.txt', 'rt');inputData = textscan(inputFile, '%f %f %f %f');fclose(inputFile);x = transpose(cell2mat(inputData));targetFile = fopen('..\\bpTrainTarget.txt', 'rt');targetData = textscan(targetFile, '%f');fclose(targetFile);t = transpose(cell2mat(targetData));
% Create a Fitting NetworkhiddenLayerSize = [12, 12];net = fitnet(hiddenLayerSize);% Setup Division of Data for Training, Validation, Testingnet.divideParam.trainRatio = 70/100;net.divideParam.valRatio = 15/100;net.divideParam.testRatio = 15/100;
% Disable pre and post process functionnet.inputs{1}.processFcns = {};net.outputs{3}.processFcns = {};
% Train the Network[net,tr] = train(net,x,t);% Test the Networky = net(x);e = gsubtract(t,y);performance = perform(net,t,y)
```

代码运行之后就会自动拟合了，几个条件满足其中之一就会终止。
演示数据量小，所以一瞬就好了，我自己的跑了几十分钟。

![拟合完成图](http://tecton.qiniudn.com/BPNN_simulation_complete.png-middleSize) 

此时可以打net看看到底有啥内容。对我来说最关键的就是weight and bias values了，就是训练完成的节点权重，只要有了它，就可以实时模拟了！
把它们小心地保存下来= =（万一不小心覆盖了就白训练了）
```
dlmwrite('IW.txt', net.IW{1});dlmwrite('LW1.txt', net.LW{2});dlmwrite('LW2.txt', net.LW{6});dlmwrite('B1.txt', net.B{1});dlmwrite('B2.txt', net.B{2});dlmwrite('B3.txt', net.B{3});
```

照着公式可以很方便地理解它们的作用以及如何拟合。
对于一层隐藏层的神经网络而言，一共就3层：输入层，隐藏层与输出层。

照着nftool里面的那张图就可以看懂了：
隐藏层的输出 `y1 = tansig(IW * input + bias1)`
最终输出 `output = (LW * y1 + bias2)` （没有tansig函数，否则输出都在(-1,1)之间了）

IW和LW都是权重，bias是一个补正的偏移，而tansig是激励函数。
如果没有它的话加权计算再怎么变不都是线性的么，有了它就可以是非线性的了(吧?)。

通过矩阵维数来看的话也是符合的:

变量 | 矩阵大小
----|--------
IW | 10*8
input | 8*1(单个输入)
bias1 | 10*1
所以y1是10*1的矩阵

同样

变量 | 矩阵大小
----|--------
LW | 1*10
bias2 | 1*1

所以output是`1*1`对应单个值的输出。

如此一来只要在c++中模拟这个矩阵运算就行了，tansig函数也是已知的，工程在此。
主要代码如下：
```
//y1 = IW * input + B1
MatrixTimesAddMatrix(IW, *input, B1, y1);

MatrixTansig(y1, y1t);
MatrixTimesAddMatrix(LW, y1t, B2, result);
```

用某个输入值测试一下
输入为matlab数据第100个，x(:,100)
样本为506
程序输出502.273
验证通过。

![C++验证图](http://tecton.qiniudn.com/NN_verification_result.png-middleSize)

总结一下，matlab有很多诸如神经网络的易用工具，即使对于具体算法不了解，我们也可以通过在C++中具体实现一遍模拟过程，对神经网络的具体概念有一个更深的了解，它没有想象的那么复杂和难以理解。