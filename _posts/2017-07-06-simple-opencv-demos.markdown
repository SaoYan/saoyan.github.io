---
title: "Simple OpenCV Demos"
date: 2017-07-06
author: yiqi
permalink: /projects/2017-07-06-simple-opencv-demos
collection: projects
tag:
- OpenCV
---

I posted several [tutorials for OpenCV beginners](http://mp.weixin.qq.com/mp/homepage?__biz=MzIxOTQ3MTI5NQ==&hid=11&sn=a972be0cf5056ec2e44bc773bb58032c#wechat_redirect) on my WeChat Public Account. I've found that providing simple demos can help readers a lot to understand  the use of OpenCV.  

You can get the code from my [Github](https://github.com/SaoYan/OpenCV_SimpleDemos).  

This post is a brief summary of the demos.

## Calculate image convolution using DFT

[DFT](https://github.com/SaoYan/OpenCV_SimpleDemos/tree/master/DFT)  

Convolution can be efficiently calculated in frequency domain. In fact, all OpenCV Image Filter API perform their calculations this way.  
In this demo I implement this myself. The result may not be exactly the same as what you get when you use Image Filter API directly, because OpenCV has some extra operations.  
You need some basic knowledge of DSP in order to understand the code.  
The result is shown below:   
figure #1: original image; figure #2: convolution result  

<img src="/images/OpenCV_demos/test_DFT.jpg" width="300" height="300" />
<img src="/images/OpenCV_demos/result_DFT.jpg" width="300" height="300" />

## SURF & SIFT Feature Detect

[FeatureDetect](https://github.com/SaoYan/OpenCV_SimpleDemos/tree/master/FeatureDetect)  

SURF feature & SIFT feature detector is included in opencv_contrib modules. Therefore, you may find the implementation quite different from other Opencv project.

1. Remember to use the namespace cv::xfeatures2d rather than cv. Many other modules in opencv_contrib also have their own namespaces. Pay attention to this in other demos.
2. In this implementation, we first define two structure, rather than use the Opencv class & function directly. The reason is that the corresponding classes are defined as abstract classes, thus cannot be used directly.

detection result (figure #1: SURF, figure #2: SIFT):  

<img src="/images/OpenCV_demos/SURF.jpg" width="320" height="240" />
<img src="/images/OpenCV_demos/SIFT.jpg" width="320" height="240" />

## Flood Fill algorithm

[FloodFill](https://github.com/SaoYan/OpenCV_ToyExamples/blob/master/FloodFill)  

Remember the Fill tool in Windows Paint Application? It can paint a connected region in the image with one color. This looks like:

<img src="/images/OpenCV_demos/test_FloodFill.jpg" width="288" height="162" />
<img src="/images/OpenCV_demos/FloodFill.jpg" width="288" height="162" />

Figure #2 is created based on Flood Fill algorithm. The API in OpenCV also returns a mask which indicates the painted region.  

<img src="/images/OpenCV_demos/FloodFillMask.jpg" width="288" height="162" />  

Please refer to the code and OpenCV documents for more details.

## Histogram Calculation

[HistCal](https://github.com/SaoYan/OpenCV_SimpleDemos/tree/master/HistCal)  

This demo calculate & display the histogram of the input image.  
figure #1: input image, figure #2-4: histogram of channel R, G, B, respectively.

<img src="/images/OpenCV_demos/test_HistCal.jpg" />  
<img src="/images/OpenCV_demos/R.jpg" />
<img src="/images/OpenCV_demos/G.jpg" />
<img src="/images/OpenCV_demos/B.jpg" />

## Histogram Back-projection

[HistBackProject V1](https://github.com/SaoYan/OpenCV_SimpleDemos/tree/master/HistBackProject%20V1)  

This demo is a simple use of histogram backprojection to detect an monochromatic (or nealy monochromatic) object.
I provide one test image with the sample patch extracted from it. Be free to use you own test images! You may be wondering how to use you camera to run histogram back-projection online. We'll see how to do this in the following demos (refer to V4).  
You may find out that the outcome is very upsetting. We'll see the reason and try to modify it in demo 'V2'.

[HistBackProject V2](https://github.com/SaoYan/OpenCV_SimpleDemos/tree/master/HistBackProject%20V2)  

In this demo, HSV color space is used to improve the performance in V1.  
The result is shown below.  

<img src="/images/OpenCV_demos/result_HistBackV2.jpg" width="315" height="235" />

[HistBackProject V3](https://github.com/SaoYan/OpenCV_SimpleDemos/tree/master/HistBackProject%20V3)

This demo has the following modifications based on HistBackProject V2.
1. Morphology operation is used to enhance the target object in the binary image.
2. Extra contours and calculate bounding box.  

The result is shown below.  

<img src="/images/OpenCV_demos/result_binary_HistBackV3.jpg" width="380" height="300" />
<img src="/images/OpenCV_demos/result_HistBackV3.jpg" width="380" height="300" />

[HistBackProject V4](https://github.com/SaoYan/OpenCV_SimpleDemos/tree/master/HistBackProject%20V4)  

This demo run Histogram Back-projection online, i.e. capture video frames via the camera on you computer and detect the target.  
Follow the steps:
1. Use your mouse to select the target region.
2. When you are done with step 1, press 'Y' and watch the object detection result.
3. Press 'Q' at any time to quit.

## Pedestrian detection

[HogPedestrianDetection](https://github.com/SaoYan/OpenCV_SimpleDemos/tree/master/HogPedestrianDetection)  

This is a fairly simple demo. The result is shown below.  

<img src="/images/OpenCV_demos/result1_pedestrian.jpg" />
<img src="/images/OpenCV_demos/result2_pedestrian.jpg" />
<img src="/images/OpenCV_demos/result3_pedestrian.jpg" />

## LSD Line Segment Detector

[LSD](https://github.com/SaoYan/OpenCV_SimpleDemos/tree/master/LSD)  

LSD is an algorithm proposed in this [IEEE paper](http://ieeexplore.ieee.org/document/4731268/).  
This algorithm is powerful, but sometimes too 'powerful' to eliminate tiny line elements in an image.  This problem is shown below.  

<img src="/images/OpenCV_demos/result_LSD.jpg" />  

In this demo, I filter the detection result by eliminating lines whose length are less than the threshold. When setting threshold to be 30, the result looks much better.  

<img src="/images/OpenCV_demos/result_filtered_LSD.jpg" />

## Object Tracking

[Tracking](https://github.com/SaoYan/OpenCV_SimpleDemos/tree/master/Tracking)

There are 5 tracking algorithms in OpenCV Tracking API. You can select one via command line parameters.  

**Running this demo needs extra parameters from the command line:**  
```
./opencv_exp trackerName {-vid | -img} { <video filename> | <dir name>
```

The parameter 'trackerName' can be one of the following:
<ul>
<li> Boosting</li>
<li> KCF</li>
<li> MedianFlow</li>
<li> MIL</li>
<li> TLD</li>
</ul>

For example  
**to use KCF tracker & video file:**
```
./opencv_exp KCF -vid test.mp4
```
**to use KCF tracker & image sequence:**
```
./opencv_exp KCF -img test
```
