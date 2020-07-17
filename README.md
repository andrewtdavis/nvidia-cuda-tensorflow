# Installing NVIDIA Drivers & CUDA, withTensorFlow support on CentOS / RHEL
This guide helps install the NVIDIA drivers, as well as one or more versions of CUDA specifically for TensorFlow but will work for any CUDA workloads. We'll also help you set up a couple versions TensorFlow to test GPU support.

This process has been tested on:

- TensorFlow GPU 1.14
- TensorFlow GPU 2.0
- TensorFlow GPU 2.1
- TensorFlow GPU 2.2

## Install NVIDIA GPU Drivers

### Disable built-in drivers

If this is the first time installing the NVIDIA driver on a system, be sure to disable the built-in noveau driver.
1. Edit */etc/modprobe.d/blacklist.conf*
2. Add the following line:
   -  `blacklist noveau`
3. Re-build the initramfs:
   1. `mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bak`
   2. `dracut -v /boot/initramfs-$(uname -r).img $(uname -r)`
4. Reboot with driver disabled
>*If this is in response to a CUDA incompatibility, be sure to completely remove the old Nvidia drivers first, and reboot*

### Install the NVIDIA drivers
>WARNING: NVIDIA GPU drivers frequently break on kernel updates. You may need to re-do this for every kerenl update.
1. Find out your GPU
    - `lspci | grep NVIDIA`, and look for a line like: `VGA compatible controller: NVIDIA Corporation GP104GL [Quadro P5000] `
2. Download the driver for your GPU, using Linux 64-bit, and "Linux long-lived" download type
    1. https://www.nvidia.com/download/index.aspx?lang=en-us
    2. When it prompts for a Display driver with the changelog, select **download**
    3. The second download page will give a download button. Right-click on that button and **select copy link address**.
3. Back on the workstation, download the file:
    - `wget *URL of driver*`
4. Enable execution of the .run file downloaded.
    - `chmod u+x NVIDIA-Linux-x86_64-*.run`
5. If GDM is running, disable it and reboot before proceeding:
    - `sudo systemctl stop gdm`  
      `sudo systemctl disable gdm`
6. Install the NVIDIA driver
    - `sudo ./NVIDIA-Linux-x86_64-*.run`
        >The installation process seems to stall, but it takes a while for the progress bar to show up. Watch top if you think it's taking too long.
7. Reboot when complete
8. Ensure that the driver loaded on startup:
    - `grep NVIDIA /var/log/dmesg` , and look for the line NVRM: loading NVIDIA UNIX x86_64 Kernel Module 
9. If GDM was running previously, enable it and re-start
    - `sudo systemctl start gdm`
    - `sudo systemctl enable gdm`

## Install CUDA

### Instll the CUDA Yum repository
1. Go to the CUDA toolkit archive: https://developer.nvidia.com/cuda-toolkit-archive
2. Select the version of CUDA that's compatible with the version of TensorFlow that's going to be used.
    >Find the version of CUDA to install based on TensorFlow listed here: https://www.tensorflow.org/install/source#tested_build_configurations
3. Select OS, Architecture (x86_64), Distribution, Version
4. For Installer Type, select rpm (network)
5. Like the driver, right-click the Download button and s*elect copy link address*.
6. Back on the workstation, download the file
    - `wget URL of CUDA rpm`
7. Install the repo:
    - i.e. `sudo rpm -i cuda-repo-rhel7-10.0.130-1.x86_64.rpm` 
8. Clean yum:
    - `sudo yum clean all`

### Install the CUDA libraries

#### Install One version of CUDA (basic):
- Look for the specific version to install, or else install the latest with `yum install cuda`
- To find a specific version to install, list available CUDA packages and look at the top of the list for packages named with versions, e.g. `cuda-10-0.x86_64`
    - `sudo yum list available | grep cuda-`
    - `sudo yum install cuda  or yum install cuda-10.0`

#### Install Multiple versions of CUDA (advanced):
>This assumes installing CUDA 10.0 - 10.2 to support TensorFlow 1.13.1 - 2.2.0+
1. Pick the versions of CUDA to install, and note which will be the primary
    - cuda-8-0
    - cuda-9-0
    - cuda-10-0
    - cuda-10-1
    - cuda-10-2
    - cuda-11-0
2. Install the versions required 
    - i.e.: `sudo yum install cuda-10.0 cuda-10.1 cuda-10.2` 
3. Make a symlink to the default version as opposed to the last version installed.
    - `cd /usr/local` 
    - `sudo rm cuda` 
    - `sudo ln -s cuda-10.0 cuda` 
4. Setup the LD_LIBRARY_PATH  to reference all installed versions of cuda:
    - `sudo sh -c 'echo export LD_LIBRARY_PATH=/usr/local/cuda/lib64:/usr/local/cuda-10.0/lib64:/usr/local/cuda-10.1/lib64L:/usr/local/cuda-10.2/lib64:\$LD_LIBRARY_PATH > /etc/profile.d/cuda.sh'`   
        >Modify this line for the versions that are installed. `/usr/local/cuda-*/lib64` should be added for each installed version of CUDA.
>Multiple versions of CUDA based on: https://medium.com/@peterjussi/multicuda-multiple-versions-of-cuda-on-one-machine-4b6ccda6faae
## Install cuDNN library
>libcuDNN is required for TensorFlow, and not installed as part of the default installation.  
>See TensorFlow issue: https://github.com/tensorflow/tensorflow/issues/20271#issuecomment-413914721

>*NOTE:* Downloading the cuDNN library requires a free NVIDIA developer account. If you don't have one, sign up when attempting to download.

### For single versions of CUDA
1. Download the latest version of cuDNN RPMs for the version of CUDA installed.
   https://developer.nvidia.com/rdp/cudnn-download  
   >*For CentOS/RHEL:* The RPM is listed under *Library for Red Hat (x86_64 & Power architecture)*
   >
   >Be sure to grab both the runtimes and developer library packages for TensorFlow
   >
   > *NOTE:* These files must be downloaded to your computer and copied to the workstation, any attempts to wget  them on the workstation will result in 403 FORBIDDEN 
2. Install both RPMs:
    - `sudo yum install libcudnn7-*`
### For multiple versions of CUDA:
1. Download the latest version of cuDNN tgz for the version of CUDA installed.  
https://developer.nvidia.com/rdp/cudnn-download  
> The .tgz files are distrobution-agnostic, and listed under *Library for Windows, Mac, Linux, Ubuntu and RedHat/Centos(x86_64architecture)*
>
>The developer library is packaged into the main .targz
>
> *NOTE:* These files must be downloaded to your computer and copied to the workstation, any attempts to wget  them on the workstation will result in 403 FORBIDDEN
>
>
For each version (do this one tgz at a time, as they all extract to cuda/):
>replace 10.0 in each line with the version being installed. 
1. Extract the tgz
    - `tar -xzvf cudnn-10.0-linux-x64-v7.6.5.32.tgz` 
2. Copy the includes
    - `sudo cp cuda/include/cudnn.h /usr/local/cuda-10.0/include` 
3. Copy the libs
    - `sudo cp cuda/lib64/libcudnn* /usr/local/cuda-10.0/lib64` 
4. Make everything readable
    - `sudo chmod a+r /usr/local/cuda-10.0/include/cudnn.h` 
    - `sudo chmod a+r /usr/local/cuda-10.0/lib64/libcudnn*` 

## Test Tensorflow with GPU support
>Ensure Python3 and PIP for Python3 are installed, and install them if not.
>
>`python3 --version`  
>`pip3 --version`
>
>
### TensorFlow 1.14 
1. Set up a virtual environment for a specific TensorFlow version
    - `virtualenv -p python3 tensorflow1.14` 
2. Activate the virtual environment
    - `source tensorflow2.0/bin/activate` 
3. Install TensorFlow GPU 1.14 in this environment
    - `pip install tensorflow-gpu==1.14`
4. Setup a test python script (example below), and see if a GPU is found.
### TensorFlow 2.0
1. Setup a virtual environment for a specific tensorflow version
    - `virtualenv -p python3 tensorflow2.0`
2. Activate the virtual environment
    - `source tensorflow2.0/bin/activate` 
3. Install TensorFlow GPU 2.0 in this environment
    - `pip install tensorflow-gpu==2.0`

### TensorFlow test harness
1. Edit a new file named something like test-gpu.py
2. Paste the following code in it:
    ``` python
    import tensorflow
    import os
    os.environ['TF_CPP_MIN_LOG_LEVEL'] = '3'
    print('TENSORFLOW VERSION: {}'.format(tensorflow.__version__))
    print('GPU DEVICE NAME: {}'.format(tensorflow.test.gpu_device_name()))
    print('IS GPU AVAILABLE: {}'.format(tensorflow.test.is_gpu_available()))
    ```
3. Run the code to show if it sees a GPU.
    - `python test-gpu.py`

You should see /dev/GPU:0 if the GPU is available.
