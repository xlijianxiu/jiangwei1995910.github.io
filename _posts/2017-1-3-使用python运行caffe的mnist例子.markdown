---
layout:     post
title:      "ubuntu16.04下编译安装caffe"
subtitle:   "caffe最简单的安装方式"
date:       2017-1-2 11:00:00
author:     "蒋为"
header-img: "img/18.jpg"
catalog: true
tags:
    - Caffe
---
>记录

## 一、数据

数据分成了训练集（60000张共10类）和测试集（共10000张10类），每个类别放在一个单独的文件夹里。并且将所有的图片，都生成了txt列表清单（train.txt和test.txt)


## 二、导入caffe库，并设定文件路径

我是将mnist直接放在根目录下的，所以代码如下：

{% highlight python %}
# -*- coding: utf-8 -*-

import caffe
from caffe import layers as L,params as P,proto,to_proto
#设定文件的保存路径
root='/home/xxx/'                           #根目录
train_list=root+'mnist/train/train.txt'     #训练图片列表
test_list=root+'mnist/test/test.txt'        #测试图片列表
train_proto=root+'mnist/train.prototxt'     #训练配置文件
test_proto=root+'mnist/test.prototxt'       #测试配置文件
solver_proto=root+'mnist/solver.prototxt'   #参数文件


{% endhighlight %}

其中train.txt 和test.txt文件已经有了，其它三个文件，我们需要自己编写。

此处注意：一般caffe程序都是先将图片转换成lmdb文件，但这样做有点麻烦。因此我就不转换了，我直接用原始图片进行操作，所不同的就是直接用图片操作，均值很难计算，因此可以不减均值。

二、生成配置文件

配置文件实际上就是一些txt文档，只是后缀名是prototxt，我们可以直接到编辑器里编写，也可以用代码生成。此处，我用python来生成。

{% highlight python %}
#编写一个函数，生成配置文件prototxt
def Lenet(img_list,batch_size,include_acc=False):
    #第一层，数据输入层，以ImageData格式输入
    data, label = L.ImageData(source=img_list, batch_size=batch_size, ntop=2,root_folder=root,
        transform_param=dict(scale= 0.00390625))
    #第二层：卷积层
    conv1=L.Convolution(data, kernel_size=5, stride=1,num_output=20, pad=0,weight_filler=dict(type='xavier'))
    #池化层
    pool1=L.Pooling(conv1, pool=P.Pooling.MAX, kernel_size=2, stride=2)
    #卷积层
    conv2=L.Convolution(pool1, kernel_size=5, stride=1,num_output=50, pad=0,weight_filler=dict(type='xavier'))
    #池化层
    pool2=L.Pooling(conv2, pool=P.Pooling.MAX, kernel_size=2, stride=2)
    #全连接层
    fc3=L.InnerProduct(pool2, num_output=500,weight_filler=dict(type='xavier'))
    #激活函数层
    relu3=L.ReLU(fc3, in_place=True)
    #全连接层
    fc4 = L.InnerProduct(relu3, num_output=10,weight_filler=dict(type='xavier'))
    #softmax层
    loss = L.SoftmaxWithLoss(fc4, label)
    
    if include_acc:             # test阶段需要有accuracy层
        acc = L.Accuracy(fc4, label)
        return to_proto(loss, acc)
    else:
        return to_proto(loss)
    
def write_net():
    #写入train.prototxt
    with open(train_proto, 'w') as f:
        f.write(str(Lenet(train_list,batch_size=64)))

    #写入test.prototxt    
    with open(test_proto, 'w') as f:
        f.write(str(Lenet(test_list,batch_size=100, include_acc=True)))
{% endhighlight %}
配置文件里面存放的，就是我们所说的network。我这里生成的network，可能和原始的Lenet不太一样，不过影响不大。

三、生成参数文件solver

同样，可以在编辑器里面直接书写，也可以用代码生成。

{% highlight python %}
#编写一个函数，生成参数文件
def gen_solver(solver_file,train_net,test_net):
    s=proto.caffe_pb2.SolverParameter()
    s.train_net =train_net
    s.test_net.append(test_net)
    s.test_interval = 938    #60000/64，测试间隔参数：训练完一次所有的图片，进行一次测试  
    s.test_iter.append(100)  #10000/100 测试迭代次数，需要迭代100次，才完成一次所有数据的测试
    s.max_iter = 9380       #10 epochs , 938*10，最大训练次数
    s.base_lr = 0.01    #基础学习率
    s.momentum = 0.9    #动量
    s.weight_decay = 5e-4  #权值衰减项
    s.lr_policy = 'step'   #学习率变化规则
    s.stepsize=3000         #学习率变化频率
    s.gamma = 0.1          #学习率变化指数
    s.display = 20         #屏幕显示间隔
    s.snapshot = 938       #保存caffemodel的间隔
    s.snapshot_prefix =root+'mnist/lenet'   #caffemodel前缀
    s.type ='SGD'         #优化算法
    s.solver_mode = proto.caffe_pb2.SolverParameter.GPU    #加速
    #写入solver.prototxt
    with open(solver_file, 'w') as f:
        f.write(str(s))
{% endhighlight %}
四、开始训练模型

训练过程中，也在不停的测试。


{% highlight python %}
#开始训练
def training(solver_proto):
    caffe.set_device(0)
    caffe.set_mode_gpu()
    solver = caffe.SGDSolver(solver_proto)
    solver.solve()
#最后，调用以上的函数就可以了。

if __name__ == '__main__':
    write_net()
    gen_solver(solver_proto,train_proto,test_proto) 
    training(solver_proto)
	
	
	{% endhighlight %}
五、完成的python文件

mnist.py

{% highlight python %}
 
 # -*- coding: utf-8 -*-

import caffe
from caffe import layers as L,params as P,proto,to_proto
#设定文件的保存路径
root='/home/xxx/'                           #根目录
train_list=root+'mnist/train/train.txt'     #训练图片列表
test_list=root+'mnist/test/test.txt'        #测试图片列表
train_proto=root+'mnist/train.prototxt'     #训练配置文件
test_proto=root+'mnist/test.prototxt'       #测试配置文件
solver_proto=root+'mnist/solver.prototxt'   #参数文件

#编写一个函数，生成配置文件prototxt
def Lenet(img_list,batch_size,include_acc=False):
    #第一层，数据输入层，以ImageData格式输入
    data, label = L.ImageData(source=img_list, batch_size=batch_size, ntop=2,root_folder=root,
        transform_param=dict(scale= 0.00390625))
    #第二层：卷积层
    conv1=L.Convolution(data, kernel_size=5, stride=1,num_output=20, pad=0,weight_filler=dict(type='xavier'))
    #池化层
    pool1=L.Pooling(conv1, pool=P.Pooling.MAX, kernel_size=2, stride=2)
    #卷积层
    conv2=L.Convolution(pool1, kernel_size=5, stride=1,num_output=50, pad=0,weight_filler=dict(type='xavier'))
    #池化层
    pool2=L.Pooling(conv2, pool=P.Pooling.MAX, kernel_size=2, stride=2)
    #全连接层
    fc3=L.InnerProduct(pool2, num_output=500,weight_filler=dict(type='xavier'))
    #激活函数层
    relu3=L.ReLU(fc3, in_place=True)
    #全连接层
    fc4 = L.InnerProduct(relu3, num_output=10,weight_filler=dict(type='xavier'))
    #softmax层
    loss = L.SoftmaxWithLoss(fc4, label)
    
    if include_acc:             # test阶段需要有accuracy层
        acc = L.Accuracy(fc4, label)
        return to_proto(loss, acc)
    else:
        return to_proto(loss)
    
def write_net():
    #写入train.prototxt
    with open(train_proto, 'w') as f:
        f.write(str(Lenet(train_list,batch_size=64)))

    #写入test.prototxt    
    with open(test_proto, 'w') as f:
        f.write(str(Lenet(test_list,batch_size=100, include_acc=True)))

#编写一个函数，生成参数文件
def gen_solver(solver_file,train_net,test_net):
    s=proto.caffe_pb2.SolverParameter()
    s.train_net =train_net
    s.test_net.append(test_net)
    s.test_interval = 938    #60000/64，测试间隔参数：训练完一次所有的图片，进行一次测试  
    s.test_iter.append(500)  #50000/100 测试迭代次数，需要迭代500次，才完成一次所有数据的测试
    s.max_iter = 9380       #最大迭代次数
    s.base_lr = 0.01    #基础学习率
    s.momentum = 0.9    #动量
    s.weight_decay = 5e-4  #权值衰减项
    s.lr_policy = 'step'   #学习率变化规则
    s.stepsize=3000         #学习率变化频率
    s.gamma = 0.1          #学习率变化指数
    s.display = 20         #屏幕显示间隔
    s.snapshot = 938       #保存caffemodel的间隔
    s.snapshot_prefix = root+'mnist/lenet'   #caffemodel前缀
    s.type ='SGD'         #优化算法
    s.solver_mode = proto.caffe_pb2.SolverParameter.GPU    #加速
    #写入solver.prototxt
    with open(solver_file, 'w') as f:
        f.write(str(s))
  
#开始训练
def training(solver_proto):
    caffe.set_device(0)
    caffe.set_mode_gpu()
    solver = caffe.SGDSolver(solver_proto)
    solver.solve()
#
if __name__ == '__main__':
    write_net()
    gen_solver(solver_proto,train_proto,test_proto) 
    training(solver_proto)
 
 
{% endhighlight %}
我将此文件放在根目录下的mnist文件夹下，因此可用以下代码执行

sudo python mnist/mnist.py

在训练过程中，会保存一些caffemodel。多久保存一次，保存多少次，都可以在solver参数文件里进行设置。

另外，如果caffe是only cpu模式，需要修改生产的solve参数文件设置为cpu模式，另外，需要将运行代码中的
    caffe.set_device(0)
    caffe.set_mode_gpu()
注释掉


我设置为训练10 epoch，9000多次，测试精度可以达到99%

