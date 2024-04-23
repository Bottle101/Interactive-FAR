# Docker Setup Instructions (Ubuntu 20.04 with ROS Noetic)

## Install Docker

### 1) For computers without a Nvidia GPU

Install Docker and grant user permission.
```
curl https://get.docker.com | sh && sudo systemctl --now enable docker
sudo usermod -aG docker ${USER}
```
Make sure to **restart the computer**, then install additional packages.
```
sudo apt update && sudo apt install mesa-utils libgl1-mesa-glx libgl1-mesa-dri
```

### 2) For computers with Nvidia GPUs

Install Docker and grant user permission.
```
curl https://get.docker.com | sh && sudo systemctl --now enable docker
sudo usermod -aG docker ${USER}
```
Make sure to **restart the computer**, then install Nvidia Container Toolkit (Nvidia GPU Driver
should be installed already).

```
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor \
  -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list \
  | sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' \
  | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```
```
sudo apt update && sudo apt install nvidia-container-toolkit
```
Configure Docker runtime and restart Docker daemon.
```
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```
Test if the installation is successful, you should see something like below.
```
docker run --gpus all --rm nvidia/cuda:11.0.3-base-ubuntu20.04 nvidia-smi
```
```
Sat Dec 16 17:27:17 2023       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 525.125.06   Driver Version: 525.125.06   CUDA Version: 12.0     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  Off  | 00000000:01:00.0  On |                  N/A |
| 24%   50C    P0    40W / 200W |    918MiB /  8192MiB |      3%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
+-----------------------------------------------------------------------------+
```

## Try a Prepared Docker Image

Allow remote X connection.
```
xhost +
```
Pull the prepared image.
```
docker pull lukechen01/ifar_demo:v2
```
For computers **without a Nvidia GPU**, start the container.
```
docker run -it --rm --privileged -e DISPLAY -e QT_X11_NO_MITSHM=1 \
  -e XAUTHORITY=/tmp/.docker.xauth -v /tmp/.X11-unix:/tmp/.X11-unix:rw -v /etc/localtime:/etc/localtime:ro \
  -v /dev/input:/dev/input -v /dev/bus/usb:/dev/bus/usb:rw -v /home/$USER:/home/$USER:rw \
  --network=host -p 10000:10000 -p 5005:5005 db908956f17f #[IMAGE_ID]
```
For computers **with Nvidia GPUs**, start the container with '--gpus all' flags.
```
docker run --gpus all -it --rm --privileged -e DISPLAY -e QT_X11_NO_MITSHM=1 \
  -e XAUTHORITY=/tmp/.docker.xauth -v /tmp/.X11-unix:/tmp/.X11-unix:rw -v /etc/localtime:/etc/localtime:ro \
  -v /dev/input:/dev/input -v /dev/bus/usb:/dev/bus/usb:rw -v /home/$USER:/home/$USER:rw \
  --network=host -p 10000:10000 -p 5005:5005 db908956f17f #[IMAGE_ID]
```
Now, try the navigation demo.
```
/home/docker/start_ifar_demo.sh
```
