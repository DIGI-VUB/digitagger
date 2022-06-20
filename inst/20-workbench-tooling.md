########################################################################################################
## General tooling on machine
## 
########################################################################################################

- Workbench has 
    - Tesla T4
    - NVIDIA-SMI 470.103.01, Driver Version: 470.103.01, CUDA Version: 11.4
    - Python 3.8.10
    - vGPU 13.2 (https://docs.nvidia.com/grid/index.html), installed with `nvidia-vgpu-ubuntu-470/now 470.103.01 amd64`
    - gcc version 9.4.0 (gcc --version)
    - glibc version 2.31 (ldd --version)
    - kernel version 5.4.0-117 (cat /proc/version or uname -r)
        - sudo apt-get install linux-headers-$(uname -r) (kernel headers and development packages for the currently running kernel can be installed with)


- WARNING: DO NOT UPGRADE linux-headers/linux-libc/linux-virtual to avoid compatibility issues with pre-installed Nvidia-drivers

```{bash}
sudo apt update
sudo apt list --upgradable
sudo apt -y autoremove
sudo reboot
```

####################################################
## NIVDIA GPU
## 

```{bash}
$ lspci | grep -i nvidia
00:07.0 3D controller: NVIDIA Corporation TU104GL [Tesla T4] (rev a1)
nvidia-smi
$ nvidia-smi
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 470.103.01   Driver Version: 470.103.01   CUDA Version: 11.4     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  GRID T4-4C          On   | 00000000:00:07.0 Off |                    0 |
| N/A   N/A    P8    N/A /  N/A |    560MiB /  4096MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```

####################################################
## NIVDIA GPU Tooling:
##   - cudart, cublas, cufft, curand, cusparse, cusolver
##   - nvrtc, nvcc
##   - cudnn8, cutensor, nccl, cusparselt
## 

```
sudo apt-key del 7fa2af80
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-keyring_1.0-1_all.deb
sudo dpkg -i cuda-keyring_1.0-1_all.deb
rm cuda-keyring_1.0-1_all.deb
sudo apt update
sudo apt install -y cuda-cudart-11-4 cuda-cudart-dev-11-4 
sudo apt install -y libcublas-11-4 libcublas-dev-11-4 libcufft-11-4 libcufft-dev-11-4 
sudo apt install -y libcurand-11-4 libcurand-dev-11-4 libcusparse-11-4 libcusparse-dev-11-4 libcusolver-11-4 libcusolver-dev-11-4 
sudo apt install -y cuda-nvrtc-11-4 cuda-nvrtc-dev-11-4 cuda-nvcc-11-4
sudo apt install -y libcudnn8 libcudnn8-dev libcutensor1 libcutensor-dev libnccl2 libnccl-dev libcusparselt0 libcusparselt-dev
sudo apt -y autoremove
## Make sure LD_LIBRARY_PATH is set correctly for the cuda tools
## driver in /usr/lib/x86_64-linux-gnu and /usr/lib32
## extra libraries are in /usr/local/cuda/lib64
cat << EOF | sudo tee -a /etc/environment
LD_LIBRARY_PATH="/usr/lib32:/usr/lib/x86_64-linux-gnu:/usr/local/cuda/lib64"
EOF
cat /etc/environment
``

#sudo apt-get --purge remove "*cuda*" "*cublas*" "*cufft*" "*cufile*" "*curand*" "*cusolver*" "*cusparse*" "*gds-tools*" "*npp*" "*nvjpeg*" "nsight*" 
`

####################################################
## Docker
## 

```
sudo apt install -y docker docker.io
```

####################################################
## Utils
## 

```
sudo apt install --yes --no-install-recommends wget
```
