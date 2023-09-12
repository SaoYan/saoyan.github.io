---
title: "Background Subtraction Using Deep Learning -- Part III"
date: 2017-11-18
author: yiqi
permalink: /posts/2017-11-18-dl-background-subtraction-3
tag:
- Deep Learning
- Tensorflow
- Mitacs Internship 2017
---
If you've seen the [DL Background Subtraction project on my Github](https://github.com/SaoYan/bgsCNN), you may find that there are five different models. The summary reports finished during my internship didn't include model IV and V. Since I've been contacted by several people asking about these two models, I decide to briefly describe them.  

Forget about model I~III?  
[summary report I](https://saoyan.github.io/posts/2017-07-27-dl-background-subtraction-1)  
[summary report II](https://saoyan.github.io/posts/2017-08-07-dl-background-subtraction-2)


<h1>Contents</h1>

* [Part I Inspiration](#Part1)
* [Part II Model IV](#Part2)
* [Part III Model V](#Part3)

<h1 id="Part1">Part I Inspiration</h1>

I was inspired by an ICCV paper: [Learning deconvolution network for semantic segmentation, ICCV 2015](https://arxiv.org/abs/1505.04366). The original model is shown in the following figure.

<center>
<img src="/images/bgsCNN_3/figure_1_1.png"  alt="figure_1_1"/><br>
<b>Figure 1.1</b> H. Noh, S. Hong, and B. Han, <br>
“Learning deconvolution network for semantic segmentation”, in ICCV, 2015.
</center>

<h1 id="Part2">Part II Model IV</h1>

The encoder-decoder part follows the ICCV paper:
* Encoder: VGG-16 (convolutional and pooling layers)
* Decoder: deconvolutional layers and unpooling layers
* All the convolutional and deconvolutional layers use same padding, down-sampling and up-sampling are performed by pooling and unpooling respectively.

The differences are:
* The input data has 6 channels, so a convolutional layer before VGG-net is required to map the input to a 3-channel "image".
* After the decoder, the feature map goes through two extra convolutional layes and is transformed to a 1-channel feature. A sigmoid function is then used to get the final probability map.

<h1 id="Part3">Part III Model V</h1>

Until model IV, the output is a 1-channel probability map created by using sigmoid function. This means that ReLU cannot be used.

Why not just treat this task as segmentation? Here we have only two categories of pixels: foreground and background. This idea leads to model V.

* Add ReLU after convolutional and deconvolutional layers.
* The output is a 2-channel feature map (rather than 1-channel), which is then fed to a softmax function.

<center>
<img src="/images/bgsCNN_3/figure_3_1.png"  alt="figure_3_1"/><br>
<b>Figure 3.1</b> results of model V
</center>

<h2>Reference</h2>
[1] [H. Noh, S. Hong, and B. Han, Learning deconvolution network for semantic segmentation, in ICCV, 2015](https://arxiv.org/abs/1505.04366)
