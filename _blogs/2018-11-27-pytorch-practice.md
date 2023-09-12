---
title: "PyTorch Practice - General Code Template"
author: yiqi
permalink: /blogs/2018-11-27-pytorch-practice
date: 2018-11-27
collection: blogs
tag:
- Deep Learning
- PyTorch
---

Here is a general code template for PyTorch (assuming an image classification task).  

```python
import random
import numpy as np
import torch
import torch.nn as nn
import torch.nn.functional as F
import torch.optim as optim
from torchvision import transforms
from networks import MyModel # defined by your self in another script

os.environ["CUDA_DEVICE_ORDER"] = "PCI_BUS_ID"
os.environ["CUDA_VISIBLE_DEVICES"] = "0,1"

device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
device_ids = [0,1]

torch.backends.cudnn.benchmark = True

if __name__ == "__main__":
    # define model
    model = MyModel()
    model = nn.DataParallel(model, device_ids=device_ids).to(device)

    # loss function
    criterion = xxxx
    criterion.to(device)

    # optimizer
    optimizer = optim.xxxx

    # data loader
    def _worker_init_fn_():
        torch_seed = torch.initial_seed()
        np_seed = torch_seed // 2**32-1
        random.seed(torch_seed)
        np.random.seed(np_seed)
    transform = transforms.Compose([xxxxxx])
    dataset = xxxxxx
    dataloader = torch.utils.data.DataLoader(dataset, batch_size=xxxx,
        shuffle=True, num_workers=8, worker_init_fn=_worker_init_fn_())    

    # training procedure
    num_epochs = xxxx
    for epoch in range(num_epochs):
        torch.cuda.empty_cache()
        for i, data in enumerate(dataloader, 0):
            model.train()
            model.zero_grad()
            optimizer.zero_grad()

            image, label = data
            image, label = image.to(device), label.to(device)

            pred = model(image)
            loss = criterion(pred, label)
            loss.backward()
            optimizer.step()

            if i % 10 == 9:
                model.eval()
                with torch.no_grad():
                    #####some testing#####
                    print("[epoch {}][{}/{}] loss {:.4f} accuracy {:.2f}%".format(epoch, i+1, len(dataloader), loss.item(), xxxx))
        # the end of one epoch
        model.eval()
        checkpoint = {
            'state_dict': model.module.state_dict(),
            'opt_state_dict': optimizer.state_dict(),
            'epoch': epoch
        }
        torch.save(checkpoint, xxPATHxx)
        with torch.no_grad():
            #####some testing#####
            print("xxxxxxx".format(xxxxxxx))
            #####logging (e.g. tensorboard)#####
```

Some notes:  
* Since PyTorch 0.4, always use *device = torch.device("cuda" if torch.cuda.is_available() else "cpu")* and *to(device)*, letting PyTorch choice the proper device itself (use GPU when available, otherwise CPU).  
* *torch.backends.cudnn.benchmark = True* will make training procedure a bit faster.
* The *_worker_init_fn_* in the above code avoids the random seed being copied across multiple workers. For more detail, refer to [PyTorch Official Document](https://pytorch.org/docs/stable/notes/faq.html#my-data-loader-workers-return-identical-random-numbers)   
* Use *torch.cuda.empty_cache()* periodically to free unused GPU memory, in case you forget to free some Tensors yourself.  
* Since dropout and batch normalization act differently during training and testing, always call *model.train()* before gradient descent and call *model.eval()* before testing.  
* Remember  *zero_grad()* before each step of gradient descent, otherwise PyTorch will accumulate gradients.  
* When saving checkpoint, save the state of optimizer and other necessary info in addition to network parameters. This will ensure that you can restore your training procedure (if interrupted for some reason).  
* If you wrap the network using *nn.DataParallel*, note that *.module* returns the actual object you expect.   
* Use *model(image)*; DO NOT use *model.forward(image)*. [See why.](https://discuss.pytorch.org/t/any-different-between-model-input-and-model-forward-input/3690)     
