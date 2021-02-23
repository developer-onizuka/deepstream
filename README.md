# 1. docker install
```
 $ sudo apt install docker.io
```

# 2. run hello world in docker
```
 $ sudo docker pull hello-world
 $ sudo docker images
 REPOSITORY                           TAG                  IMAGE ID            CREATED             SIZE
 hello-world                          latest               bf756fb1ae65        13 months ago       13.3kB
 $ sudo docker run hello-world:latest
 Hello from Docker!
 This message shows that your installation appears to be working correctly.

 To generate this message, Docker took the following steps:
  1. The Docker client contacted the Docker daemon.
  2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
     (amd64)
  3. The Docker daemon created a new container from that image which runs the
     executable that produces the output you are currently reading.
  4. The Docker daemon streamed that output to the Docker client, which sent it
     to your terminal.

 To try something more ambitious, you can run an Ubuntu container with:
  $ docker run -it ubuntu bash
 
 Share images, automate workflows, and more with a free Docker ID:
  https://hub.docker.com/
 
 For more examples and ideas, visit:
  https://docs.docker.com/get-started/
```
# 3. install nvidia-docker2
```
 $ curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey |   sudo apt-key add -
 $ distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
 $ curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list |   sudo tee /etc/apt/sources.list.d/nvidia-docker.list
 $ sudo apt-get update
 $ sudo apt-get install -y nvidia-docker2
 $ sudo systemctl restart docker
```
 
# 4. deepstream
 see also https://ngc.nvidia.com/catalog/containers/nvidia:deepstream.
```
 $ sudo docker pull nvcr.io/nvidia/deepstream:5.0.1-20.09-triton
 $ xhost +
 $ sudo docker run --gpus all -it --rm -v /tmp/.X11-unix:/tmp/.X11-unix --device /dev/video0:/dev/video0:mwr -e DISPLAY=$DISPLAY -w /opt/nvidia/deepstream/deepstream-5.0  nvcr.io/nvidia/deepstream:5.0.1-20.09-triton

 ---in-container---
 root@db8c980a463f:/opt/nvidia/deepstream/deepstream-5.0# deepstream-app -c samples/configs/deepstream-app/source1_usb_dec_infer_resnet_int8.txt
```

 You can see yourself through Web camera running at /dev/video0 and !!!
