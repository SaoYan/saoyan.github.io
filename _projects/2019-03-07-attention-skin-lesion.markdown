---
title: "Melanoma Recognition via Visual Attention"
date: 2019-03-07
author: yiqi
permalink: /projects/2019-03-07-attention-skin-lesion
collection: projects
tag:
- Deep Learning
---

***
[Paper (preprint)](http://www.cs.sfu.ca/~hamarneh/ecopy/ipmi2019.pdf)  
[Paper (camera ready)](https://link.springer.com/chapter/10.1007%2F978-3-030-20351-1_62)  
[IPMI 2019 poster](/files/Poster-IPMI-2019.pdf)   
[Code](https://github.com/SaoYan/IPMI2019-AttnMel)  

***

## Network architecture

<img src="/images/IPMI2019/network.png" alt="network_architecture"/>  

The backbone network is VGG-16 (the yellow and red blocks) without any dense layers. Two attention modules are applied (the gray blocks). The three feature vectors (green blocks) are computed via global average pooling and are concatenated together to form the final feature vector, which serves as the input to the classification layer. The classification layer is not shown here. For more details please refer to the paper.  

## Melanoma Recognition Performance  

We perform ablation study to explore the effectiveness of visual attention. We also compare with previous method. For experimental details, please refer to the paper.  

<img src="/images/IPMI2019/ISIC2016.png" alt="isic2016"/><br>  
<img src="/images/IPMI2019/ISIC2017.png" alt="isic2017"/>

## Qualitative Analysis of Attention Maps  

* The deeper layer (pool-4) exhibits more concentrated attention to valid regions than the shallower layer (pool-3).  
* The models with additional regularization (rows 4-7) produce more refined and semantically meaningful attention maps, which accounts for the accuracy improvement.  

<img src="/images/IPMI2019/visualization.png" alt="visualization"/>  

## Quantitative Analysis of Attention Maps  

We quantify the “quality” of the learned attention map by computing its overlap with the ground truth lesion segmentation. First, we re-normalize each attention map to [0,1] and binarize it using a threshold of 0.5. Then we compute the Jaccard index with respect to the ground truth lesion segmentation.

<img src="/images/IPMI2019/jaccard.png" alt="jaccard"/>  
