---
title: "Spectral Super-resolution from Single RGB Image Using Multi-scale CNN"
date: 2018-06-09
author: yiqi
permalink: /projects/2018-06-09-super-spectral
collection: projects
tag:
- Deep Learning
---

***
[Paper (preprint)](https://arxiv.org/abs/1806.03575)   
[Paper (camera ready)](https://link.springer.com/chapter/10.1007/978-3-030-03335-4_18)   
[Code](https://github.com/SaoYan/Multiscale-Super-Spectral)  
[PRCV 2018 poster](/files/Poster-PRCV-2018.pdf)   

**Bachelor's Thesis**  
[English Version](/files/Bachelor-Thesis.pdf)  
[Chinese Version (Unofficial)](/files/Bachelor-Thesis-CHN.pdf)  

***

Different from traditional hyperspectral super-resolution approaches that focus on improving the spatial resolution, spectral superresolution aims at producing a high-resolution hyperspectral image from the RGB observation with super-resolution in spectral domain. However, it is challenging to accurately reconstruct a high-dimensional continuous spectrum from three discrete intensity values at each pixel, since too much information is lost during the procedure where the latent hyperspectral image is downsampled (e.g., with ×10 scaling factor) in spectral domain to produce an RGB observation. To address this problem, we present a multi-scale deep convolutional neural network (CNN) to explicitly map the input RGB image into a hyperspectral image. Through symmetrically downsampling and upsampling the intermediate feature maps in a cascading paradigm, the local and non-local image information can be jointly encoded for spectral representation, ultimately improving the spectral reconstruction accuracy. Extensive experiments on a large hyperspectral dataset demonstrate the effectiveness of the proposed method.

## Network architecture

<img src="/images/Bachelor-Thesis/1.png"  alt="figure_1"/>  

The network follows the encoder-decoder pattern, and skip connections are used to concatenate the corresponding feature maps of the encoder and decoder. The ***encoder*** can be interpreted as extracting features from RGB images. Through downsampling in a cascade way, the receptive field of the network is constantly increased, which allows the network to “see” more pixels in an increasingly larger field of view. By doing so, both the local and non-local information can be encoded to better represent the latent spectra. The symmetric ***decoder*** procedure is employed to reconstruct the latent hyperspectral images based on these deep and compact features. The skip connections with concatenations are essential for introducing multi-scale information and yielding better estimation of the spectra.

## Quantitative Evaluation  

We compare our method with regular spline interpolation, the sparse recovery method proposed by [Arad et al.](https://link.springer.com/chapter/10.1007/978-3-319-46478-7_2), the [A+ based method](http://openaccess.thecvf.com/content_ICCV_2017_workshops/papers/w9/Aeschbacher_In_Defense_of_ICCV_2017_paper.pdf), and another deep learning method [Galliani et al.](https://arxiv.org/abs/1703.09470).  

<img src="/images/Bachelor-Thesis/2.png"  alt="figure_2_1"/><br>  
<img src="/images/Bachelor-Thesis/3.png"  alt="figure_2_2"/>  

## Visualization  

Reconstruction error map (from left to right: RGB rendition, A+, Galliani et al., and our method)  

<img src="/images/Bachelor-Thesis/4.png"  alt="figure_3_1"/><br>  
<img src="/images/Bachelor-Thesis/5.png"  alt="figure_3_2"/>  

Sample results of spectral reconstruction by our method  

<img src="/images/Bachelor-Thesis/6.png"  alt="figure_4"/>  