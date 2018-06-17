---
layout: post
title: "关于单片机初始化的注意事项"
tags: 单片机
---



# 正题

为了让单片机完成我们所需的功能，我们每次都会使用到很多的外设，并且在控制系统中，为了保证控制的时效性，有一个专门的定时器来产生中断，进入控制环——controlloop，在主函数的循环中则放入不痛不痒的程序——led闪烁之类的。一个标准的单片机程序流程为：初始化各个外设，进入while(1)循环，定时器定时产生中断，在中断中，完成控制任务，回到while(1)，再进入下一个中断，解算姿态......以此循环往复。（本程序中TIM14产生定时中断）

在外设数量较多，并有可能又要添加新外设的情况下，我们通常使用BSP（board support package ，板级支持包 ）来管理外设。主要的好处是不用每次添加新外设的时候在每个外设的程序前面去包含新的外设头文件。


所以我们的程序一般就变成了这样。

![捕获3](\img\an_bug_about_microcontroller_initalization_img\捕获3.PNG)

bsp_Init中初始化全部外设。

![捕获2](\img\an_bug_about_microcontroller_initalization_img\捕获2.PNG)

这看起来是一个没问题的程序，但是实际运行的时候，却跑不起来。

# BUG在哪里呢？

bug在于初始化的顺序，以上面的程序为例，运行之后发现程序卡住了，进入debug发现程序卡死在这里，这是干什么用的呢？printf的发送程序，等待发送完成的信号。为什么没等到发送完成呢？因为串口都还没初始化啊！

![捕获](\img\an_bug_about_microcontroller_initalization_img\捕获.PNG)

正常的程序，运行流程图应该是这样的：

![捕获4](\img\an_bug_about_microcontroller_initalization_img\捕获4.PNG)

这个错误的程序，运行流程图是这样的：（外设2依然会初始化完成，但在如果在还没初始化完成的时候就进入中断，就会出现问题）

![捕获6](\img\an_bug_about_microcontroller_initalization_img\捕获6.PNG)

问题出现在，控制程序controlloop是肯定要用到外设的——串口，ADC之类的。如果过早进入定时器中断，外设还未初始化完成，就会出现程序轮询一直等相应的情况——等待串口数据发送完成，等待ADC采样完成（程序卡死一般都是因为while(xx)这句等待而导致的）。但是其实根本没法完成，因为这些还没开机初始化呢。

所以，在单片机程序中，产生定时器定时中断的timer必须放在最后初始化。如下图所示：

![捕获7](\img\an_bug_about_microcontroller_initalization_img\捕获7.PNG)