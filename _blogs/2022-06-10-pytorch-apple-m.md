---
title: "I wanted to try PyTorch on M1 Max, but I couldn't..."
author: yiqi
permalink: /blog/2022-06-10-pytorch-apple-m
date: 2022-06-10
collection: blog
tag:
- Deep Learning
- PyTorch
---

***
## TLDR

* When installing ```pytorch``` and ```torchvision``` together, ```pytorch``` version is forced to fall back to latest stable release (1.11) which does not have Apple silicon support.
* Same issue happened before. [Conda install from pytorch-nightly channel with cpu-only option delivers 1.3.1 version](https://github.com/pytorch/pytorch/issues/33103)
* New issue submitted. [Conda install from pytorch-nightly channel does not install the expected version on macOS](https://github.com/pytorch/pytorch/issues/79337)

***

PyTorch has announced [support for Apple silicon GPUs](https://pytorch.org/blog/introducing-accelerated-pytorch-training-on-mac/) for sometime. The official release will be in next v1.12, and is already available in nightly build.

When I was thinking about Friday evening activity for today, a thought came to me to play with PyTorch nightly a bit and see how it performs on my new Mac Studio with M1 Max. Sadly I failed because of some wired bug...

I googled a bit and found the currently best solution for installing native Python on Apple silcon is through miniforge

```
brew install miniforge

conda init zsh

conda activate
```

(If you are using other shells instead of zsh, adjust ```conde init``` accordingly.)

Everything looked great and I happily installed pytorch

```
conda install pytorch torchvision torchaudio -c pytorch-nightly
```

Here comes the first problem: ```torchaudio``` is not found

```
PackagesNotFoundError: The following packages are not available from current channels:

  - torchaudio

Current channels:

  - https://conda.anaconda.org/pytorch/osx-arm64
  - https://conda.anaconda.org/pytorch/noarch
  - https://conda.anaconda.org/conda-forge/osx-arm64
  - https://conda.anaconda.org/conda-forge/noarch

To search for alternate channels that may provide the conda package you're
looking for, navigate to

    https://anaconda.org

and use the search bar at the top of the page.
```

There is actually an active issue for this problem: [Unable to install Preview (Nightly) on M1 macOS: "Symbol not found"](https://github.com/pytorch/pytorch/issues/78681).

This is not blcking me since I just need the other two.

```
conda install pytorch torchvision -c pytorch-nightly
```

Unexpectedly, conda does not pull from ```pytorch-nightly``` channel, but the default one ```conda-forge```. PyTorch v1.11 got installed. This is the latest stable release without Apple silicon support!

I tried installing pytorch only without torchvision

```
conda install pytorch -c pytorch-nightly
```

and this time v1.13 was installed. Problem solved? Not really...

When instaling torchvision following pytorch

```
conda install torchvision -c pytorch-nightly
```

pytorch is forced to fall back to v1.11

```
The following packages will be SUPERSEDED by a higher-priority channel:

  pytorch            pytorch-nightly::pytorch-1.13.0.dev20~ --> conda-forge::pytorch-1.11.0-cpu_py39h03f923b_1
```

Now I'm blocked...

Actually the same issue has happened before where torchvision nightly build is bind with earlier version of pytorch: [Conda install from pytorch-nightly channel with cpu-only option delivers 1.3.1 version](https://github.com/pytorch/pytorch/issues/33103).

I submitted a new issue and let's hope this gets resolved or v1.12 get released ASAP. [Conda install from pytorch-nightly channel does not install the expected version on macOS](https://github.com/pytorch/pytorch/issues/79337)