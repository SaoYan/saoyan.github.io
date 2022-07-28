---
title: "Obstacle Avoiding Wheeled Robot"
date: 2017-04-23
author: yiqi
permalink: /projects/2017-04-23-obstacle-avoiding-wheeled-robot
collection: projects
tag:
- OpenCV
---

## I. Brief Description

The task for this program was to build a automatic robot which can navigate through several randomly placed obstacles and stop exactly at the finishing line. I was responsible for building the vision system as well as the controlling module. For user convenience, I developed this GUI based on OpenCV and Qt Creator.  

<img src="/images/Obstacle-Avoiding-Wheeled-Robot/GUI.jpg">  

### Vision

I convert each frame captured by the camera into HSV color space for robust histogram matching. What is more, in order to reduce the influence of illumination variance, I mask out pixels whose saturation (the 'S' element in HSV space) is too high or too low. A tracking method called cam-shift is used as an assistance to histogram matching.

<img src="/images/Obstacle-Avoiding-Wheeled-Robot/vision1.png"> <br>  
<img src="/images/Obstacle-Avoiding-Wheeled-Robot/vision2.png">

## Controlling

I utilize finite state machine for the control of the robot. First, I define several implicit variables, and different values of these variables illustrate different state of the robot. Then I set a timer which will detect the robot's 'state' every 10 ms, and the robot will decide what to do next based on the current state. It is important that these implicit variables should be updated independently, in another parallel thread, based on vision signals and commands from the user.

## II. Source Code

[Source code available here](https://github.com/SaoYan/obstacle-avoiding-wheeled-robot)

### Requirements

* [Qt Creator](https://www.qt.io/ide/)
* [OpenCV](http://opencv.org/)
* Either Window or Linux operating system is OK (Qt Creator is a  cross-platform IDE)

### Main features

* Object tracking using histogram matching.
* Robot controlling through serial port using finite state machine software implementation.

## III. Summary Reports

* [Report on vision system (in Chinese)](/files/wheeled-robot-vision-report.pdf)
* [Report on the whole program (in Chinese)](/files/wheeled-robot-project-report.pdf)

## IV. Videos

### Demo: Object detection and tracking based on histogram matching

<iframe width="560" height="315" src="https://www.youtube.com/embed/VLcuBOuiDPg" frameborder="0" allowfullscreen></iframe>  

### 18th National Robot Championship

<iframe width="560" height="315" src="https://www.youtube.com/embed/sFd6vTo6jhg" frameborder="0" allowfullscreen></iframe>  
