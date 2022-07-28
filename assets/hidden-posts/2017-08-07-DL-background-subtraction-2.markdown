---
title: "Background Subtraction Using Deep Learning -- Part II"
date: 2017-08-07
author: yiqi
permalink: /posts/2017-08-07-dl-background-subtraction-2
tag:
- Deep Learning
- Tensorflow
- Mitacs Internship 2017
---

***This post summarizes my work during week 5-6 of my summer internship.***  

I finished the training of the model mentioned in the [last report](https://saoyan.github.io/posts/2017/07/27). The result was disappointing. Then I modified the model and got a fairly good result. For convenience, I will mention the model mentioned in the last report as Model I, and the modified model as Model II and Model III.

All the training curves and visualization images are created with [Tensorboard Toolkit](https://github.com/tensorflow/tensorboard) ([1]).

You can get a [PDF version of this post from here](/files/Mitacs_Internship_Summary_Report_3.pdf).

The source code of *DL Background Subtraction* is [available on my Github](https://github.com/SaoYan/VehicleCounting_DL).  


<h1>Contents</h1>

* [Part I First experiment](#Part1)
    * [1.1 Hardware configuration](#1.1)
    * [1.2 Hyperparameters](#1.2)
    * [1.3 Experiment result](#1.3)
        * [1.3.1 Training curve](#1.3.1)
        * [1.3.2 Image visualization](#1.3.2)
* [Part II Modification of the model](#Part2)
* [Part III experiment result of the modified models](#Part3)
    * [3.1 Training curve](#3.1)
    * [3.2 Checkboard artifacts](#3.2)
    * [3.3 Comparison with classical methods](#3.3)

<h1 id="Part1">Part I First experiment</h1>

The first experiment is based on Model I. Refer to figure 1.1 as a review of the model.

<center>
<img src="/images/bgsCNN_2/figure_1_1.png"  alt="figure_1_1"/><br>
<b>Figure 1.1</b> the model proposed in the last report
</center>

<h2 id="1.1">1.1 Hardware configuration</h2>

As is mentioned in the last report, I use cloud server to run the code. The hardware information is shown in the following table.

<center>
<img src="/images/bgsCNN_2/table_1_1.png"  alt="table_1_1"/><br>
<b>Table 1.1</b> Hardware configuration
</center>

<h2 id="1.2">1.2 Hyperparameters</h2>

In the last report, I mentioned the value of some hyper-parameters, but I modify some of them in my actual experiment. The following results are all based on the new set of parameters. Refer to table 1.2.

<center>
<img src="/images/bgsCNN_2/table_1_2.png"  alt="table_1_2"/><br>
<b>Table 1.2</b> value of hyperparameters
</center>

<h2 id="1.3">1.3 Experiment result</h2>

<h3 id="1.3.1">1.3.1 Training curve</h3>

Figure 1.2 shows part of the training curve (from step 5000 to the end).

<center>
<img src="/images/bgsCNN_2/figure_1_2.png"  alt="figure_1_2"/><br>
<b>Figure 1.2</b> training curve of Model I (from step 5000 to the end)
</center>

It is clear that the cross-entropy loss on training set keeps vibrating within a relatively large range. This result is far from satisfactory. And it is not surprising that the loss on the test set is as high as 0.167.

<h3 id="1.3.2">1.3.2 Image visualization</h3>

I select one frame from test set and visualize the output feature map of each layer. Refer to figure 1.3.

In the output feature map of sigmoid activation, the activated region is approximately the same as ground truth, which means that the training does work. But after 10000 steps of iteration, the crossentropy loss is still vibrating on the training set and remains pretty high on the test set. Based on these analyses, we can come to the conclusion that the training samples are too few for the loss function to converge to the global minimum. Therefore, parameter-reduction is needed.

There is another interesting characteristic. Each feature map in figure 1.3 has checkboard artifacts. This is due to deconvolutional operations. (In deconvolutional layer, when stride is not 1, zeros will be filled into feature map. Refer to [2]) This will also be improved in the new models.

<center>
<img src="/images/bgsCNN_2/figure_1_3.png"  alt="figure_1_3"/><br>
<b>Figure 1.3</b> (Model I) visualization result of one frame in test set; for each feature map, only the first channel is shown. <i><b>Top:</b></i> original image and ground truth; <i><b>Bottom, from left to right:</b></i> the output feature of three deconvolutional layers, one convolution layer, and the final sigmoid activation
</center>

<h1 id="Part2">Part II Modification of the model</h1>

For the convenience of comparison, the architecture of Model I is recorded in detail. Refer to table 1.3. (Pay attention to the horrible amount of parameters). Table 1.4 and 1.5 shows the architecture details of Model II and III respectively. The number of parameters is reduced to a large degree in both models.

* Model II differs from Model I in the following two aspects.

1. Use 3D average pooling to reduce the output feature map of ResNet.
2. Use additional max pooling layers.
3. Use smaller deconvolutional filters to reduce the number of parameters.
* Model III does not have additional max pooling layers. Instead, I use larger deconvolutional filters.

<center>
<img src="/images/bgsCNN_2/table_2_1.png"  alt="table_2_1"/><br>
<b>Table 2.1</b> architecture details of Model I
</center><br>

<center>
<img src="/images/bgsCNN_2/table_2_2.png"  alt="table_2_2"/><br>
<b>Table 2.2</b> architecture details of Model II
</center><br>

<center>
<img src="/images/bgsCNN_2/table_2_3.png"  alt="table_2_3"/><br>
<b>Table 2.3</b> architecture details of Model III
</center><br>

It is worth noting that Model II and III use different methods to improve checkboard artifacts. In Model II, additional max pooling layers eliminate extra zeros in the deconvolutional feature map. In Model III, larger deconvolutional filters can cover more non-zero elements. I will compare them in the following sections.

<h1 id="Part3">Part III experiment result of the modified models</h1>

<h2 id="3.1">3.1 Training curve</h2>

It took about 2 days 10 hours to train each model. Figure 3.1 and 3.2 show training curve of Model II and Model III respectively. Compared to Model I, both II and III converge fairly well.

<center>
<img src="/images/bgsCNN_2/figure_3_1.png"  alt="figure_3_1"/><br>
<b>Figure 3.1</b> Training curve of Model II<br>
<i><b>Top:</b></i> training curve; <i><b>Bottom:</b></i> zoom in
</center><br>

<center>
<img src="/images/bgsCNN_2/figure_3_2.png"  alt="figure_3_2"/><br>
<b>Figure 3.2</b> Training curve of Model III<br>
<i><b>Top:</b></i> training curve; <i><b>Bottom:</b></i> zoom in
</center>

In figure 3.3, training curves of II and III are plotted in the same graph for comparison. Model II and III have pretty similar performance with respect to cross-entropy loss. The loss on the test set is about 0.23 for both models.

<center>
<img src="/images/bgsCNN_2/figure_3_3.png"  alt="figure_3_3"/><br>
<b>Figure 3.3</b> Comparison of Model II and Model III<br>
<i><b>Top:</b></i> training set; <i><b>Bottom:</b></i> test set
</center>

<h2 id="3.2">3.2 Checkboard artifacts</h2>

Visualization analysis is also performed for Model II and III. Refer to figure 3.4 and 3.5 respectively.

According to the visualization result, Model II out-performs Model III a lot with respect to improving checkboard artifacts. It is worth noting that each additional max pooling operation reduces checkboard artifacts in the corresponding deconvolutional feature map.

<center>
<img src="/images/bgsCNN_2/figure_3_4.png"  alt="figure_3_4"/><br>
<b>Figure 3.4</b> (Model II) visualization result of one frame in test set; for each feature map, only the first channel is shown. <i><b>Top:</b></i> original image and ground truth; <i><b>Bottom, from left to right:</b></i> the output feature of four pairs of deconvolutional+pooling layers, and the final sigmoid activation
</center><br>

<center>
<img src="/images/bgsCNN_2/figure_3_5.png"  alt="figure_3_5"/><br>
<b>Figure 3.5</b> (Model III) visualization result of one frame in test set; for each feature map, only the first
channel is shown. <i><b>Top:</b></i> original image and ground truth; <i><b>Bottom, from left to right:</b></i> the output feature of three deconvolutional layers, two convolutional layers, and the final sigmoid activation
</center>

<h2 id="3.3">3.3 Comparison with classical methods</h2>

Now that we’ve got get a satisfactory result on the test, we may still want to know how well does deep learning method generalize? I download a video from the internet, run both SuBSENSE and deep learning method (Model II), and compare the result. Two of the frames are shown in figure 3.6.

<center>
<img src="/images/bgsCNN_2/figure_3_6_1.png"  alt="figure_3_6_1"/><br>
<img src="/images/bgsCNN_2/figure_3_6_2.png"  alt="figure_3_6_2"/><br>
<b>Figure 3.6</b> background subtraction test on surveillance video<br>
<i><b>Top:</b></i> original frame<br>
<i><b>Bottom left:</b></i> foreground mask created by SuBSENSE<br>
<i><b>Bottom right:</b></i> foreground mask created by Model II
</center>

First, let’s focus on the objects highlighted by red rectangles. They are small objects at a relatively longer distance from the camera. With respect to these objects, SuBSENSE out-performs deep learning method. CNN model seems not able to distinguish between small objects.

As for nearer, bigger objects, as is noted by blue rectangles, deep learning method shows its advantage. Foreground masks created by SuBSENSE is often ‘broken’ into parts, and CNN improves this to a large degree.

But why is my model unable to detect small objects? The reason still remains to be figured out.

<h2>Reference</h2>
[1] [Tensorboard: TensorFlow's Visualization Toolkit](https://github.com/tensorflow/tensorboard)  
[2] [Theano document: Convolution arithmetic tutorial](http://deeplearning.net/software/theano/tutorial/conv_arithmetic.html)
