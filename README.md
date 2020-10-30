# egpu-setup
How to setup an external GPU unit on Mac

## The confusing world of CUDA + macOS
First off, let's just acknowledge it is not the most ideal situation to try to setup a NVIDIA GPU with a Mac. There is a plethora combination of iOS + XCode + CUDA + cuDNN + Pytorch versions that need to be mutually compatible with each other. One is better off to just sign up to a cloud-based GPU or use one of the free GPU services like Google Colab. 

With that rant out of the way, here are the steps we will go through to get an external GPU unit hooked up and working with your Mac. Furthermore we will cover how to install PyTorch to utilize the eGPU.
1. Setup the GPU hardware driver on your Mac system (so that your OS recognizes the hardware)
2. Install the correct version of Xcode on your Mac (so that the corresponding versions of CUDA & cuDNN can be installed)
3. Install PyTorch with CUDA enabled (so all the tensor computations can be sped up)

Sounds simple enough? Let's get started.

## Reference
This setup guide references several scripts, posts and online Q/As. Big thanks to these fine folks for paving the road to make this a feasible task!

- [Accelerated Deep Learning on a MacBook with PyTorch: the eGPU (NVIDIA Titan XP)](https://mc.ai/accelerated-deep-learning-on-a-macbook-with-pytorch-the-egpu-nvidia-titan-xp/) by Janne Spijkervet 
She pretty much covered 90% of the steps so a big shout out to her! There was a couple of steps where I had to do a bit of digging to get around since things may have changed a bit since when this blog post was written.
- [macOS-eGPU.sh](https://github.com/learex/macOS-eGPU/tree/master) by learex
Thank God for this script for making setting up the driver pretty straight foward!

## My Setup
This process/experience was based on my setup so it may or may not work for you, but the overall steps are probably similar enough to reference off of (as long as you get all your software versions compatible with each other which admittedly was a pain).

- iMac 27-inch, Late 2012 
    - High Sierra (Version 10.13.6)
    - 2.9 GHz Intel Core i5
    - 16 GB 1600 MHz DDR3 Memory
    - NVIDIA GeForce GTX 660M 512 MB graphics card
- NVIDIA TITAN Xp External GPU

## Step 1. Setup the GPU Driver
### Instructions
- Reboot your Mac in recovery mode by immediately holding down the Command + R keys until you see an Apple logo or spinning globe
- Open terminal once inside the recovery mode and type the command `csrutil disable` to disable SIP
- Reboot your Mac normally
- Clone the repository: https://github.com/learex/macOS-eGPU.git
- Read the "Requirements", "Important Information", and "Step by Step Guide" to make sure this script fits your situation before you proceed
- Locally, start a terminal and navigate to the local copy of the repo you just cloned and run `bash <(curl -s https://raw.githubusercontent.com/learex/macOS-eGPU/master/macOS-eGPU.sh)` and follow the instruction of the prompt output by the script

### Checkpoint
Once the script installation process has completed, you can check whether your Mac OS now recognizes your eGPU unit via *Spotlight Search -> systeminformation -> Graphics/Displays* and check to see if your eGPU model is now listed.

## Step 2. Install CUDA
The most important part of this step is to correctly identify the version of CUDA you want to install. NVIDIA provides a list here: [CUDA toolkit archive](https://developer.nvidia.com/cuda-toolkit-archive). 
- Find the CUDA version (v9.2 was what I used to correspond with my High Sierra v10.13.6)
- Check *Online Documentation-> Installation Guide Mac OS X* for the chosen CUDA version; there should be a section that highlights tool chain versions required for that version of CUDA. ([Installation Guide for CUDE v9.2](https://docs.nvidia.com/cuda/archive/9.2/cuda-installation-guide-mac-os-x/index.html))
- Check the specified Xcode version under the system requirements section to make sure you have the version of the Xcode installed (install that particular version of Xcode if you do not)
- Follow the instructions provided in the Installation Guide to complete the installation of CUDA Driver, CUDA Toolkit	, and CUDA Samples.

### Checkpoint
Follow instructions under the *Verification* section of the Installation to build and run the  *deviceQuery* and *bandwidthTest* samples. This will verify that your CUDA installation can query and profile your eGPU device.

## Step 3. Install PyTorch compiled with CUDA
Unfortunately the normal PyTorch versions available through `pip install` or `conda install` do not come with CUDA pre-compiled (at least for OSX systems). This requires us to build PyTorch from source and you will need to find the specific release version that works with your installed CUDA version. The specifics can be found in the [**From Source**](https://github.com/pytorch/pytorch#from-source) section of the official PyTorch Github repo page. The installation instruction is under the [On macOS](https://github.com/pytorch/pytorch#install-pytorch) section of the page.

This [eGPU setup blog post](https://mc.ai/accelerated-deep-learning-on-a-macbook-with-pytorch-the-egpu-nvidia-titan-xp/) by Janne Spijkervet provides execellent instructions on installing `miniconda` and setting up a separate `conda env` in preparation for your PyTorch installation and this is highly recommended to insulate the rest of your system from the installation process. I had followed her instructions and checkouted out `v1.0rc1` of PyTorch for installation. Her instructions are generally good and detailed, but when I tried to do it, the process complained about a missing submodule "nervanagpu", which was removed from NervanaSystems' GitHub. [This thread](https://github.com/gpgpu-sim/pytorch-gpgpu-sim/issues/3) provided the solution to get around this issue, which was to simply remove reference to this submodule in your local git repo ([source](https://gist.github.com/myusuf3/7f645819ded92bda6677)):
```
To remove a submodule you need to:

- Delete the relevant section from the .gitmodules file.
- Stage the .gitmodules changes git add .gitmodules
- Delete the relevant section from .git/config.
- Run git rm --cached path_to_submodule (no trailing slash).
- Run rm -rf .git/modules/path_to_submodule (no trailing slash).
- Commit git commit -m "Removed submodule "
- Delete the now untracked submodule files rm -rf path_to_submodule
``` 

The PyTorch installation process building from source takes an awful long time to complete (but the result is worth the wait!). After the installation completes, continue to follow Jane's instructions on **Compile and install torchvision** from source to also install the `torchvision` module.

### Checkpoint
Start your python session and `import torch`. Create a random tensor object and attempt to load it on to your GPU as follows:
```
python
import torch
torch.cuda.is_available()
print(torch.cuda.device_name(0))
d = torch.device("cuda")
x = torch.tensor([1.0, 2.0]).to(d)
```
If this executes without problem, then you're good to go!


