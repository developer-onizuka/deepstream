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
 
# 4. deepstream test if it works well
 see also https://ngc.nvidia.com/catalog/containers/nvidia:deepstream.
```
 $ sudo docker pull nvcr.io/nvidia/deepstream:5.0.1-20.09-triton
 $ xhost +
 $ sudo docker run --gpus all -it --rm -v /tmp/.X11-unix:/tmp/.X11-unix --device /dev/video0:/dev/video0:mwr -e DISPLAY=$DISPLAY -w /opt/nvidia/deepstream/deepstream-5.0  nvcr.io/nvidia/deepstream:5.0.1-20.09-triton

 ---in-container---
 root@db8c980a463f:/opt/nvidia/deepstream/deepstream-5.0# deepstream-app -c samples/configs/deepstream-app/source1_usb_dec_infer_resnet_int8.txt
```

 You can see yourself through Web camera running at /dev/video0 !!!
  
 # 5. deepstream with pre-trained model
  see also https://docs.nvidia.com/metropolis/TLT/tlt-getting-started-guide/text/deploying_to_deepstream.html#generating-an-engine-using-tlt-converter.

 (1) download pruned model in container of nvcr.io/nvidia/tlt-streamanalytics:v3.0-dp-py3
```
  $ sudo docker run --gpus all -it -v /mnt/docker/tlt-experiments:/workspace/tlt-experiments -p 8888:8888 nvcr.io/nvidia/tlt-streamanalytics:v3.0-dp-py3 /bin/bash
  
 ---in-container---
  root@a65e47c7859e:/workspace/tlt-experiments# pwd
  /workspace/tlt-experiments

  root@a65e47c7859e:/workspace/tlt-experiments# ls
  data  etc  input  kitti  model  output  sample  tlt-convert

  root@5b2a2185c851:/workspace/tlt-experiments# ngc registry model download-version nvidia/tlt_facedetectir:pruned_v1.0 --dest ./model
  Downloaded 9.09 MB in 1m 9s, Download speed: 134.76 KB/s               
  ----------------------------------------------------
  Transfer id: tlt_facedetectir_vpruned_v1.0 Download status: Completed.
  Downloaded local path: /workspace/tlt-experiments/model/tlt_facedetectir_vpruned_v1.0
  Total files downloaded: 3 
  Total downloaded size: 9.09 MB
  Started at: 2021-02-25 08:10:57.848917
  Completed at: 2021-02-25 08:12:06.925858
  Duration taken: 1m 9s
  ----------------------------------------------------
``` 
 (2) tlt-converter from etlt-file to engine-file
 ```
 root@a65e47c7859e:/workspace# pwd
 /workspace

 root@a65e47c7859e:/workspace# export ENGINE_PATH=tlt-experiments/model/tlt_facedetectir_vpruned_v1.0/resnet18_facedetectir_pruned.engine
 root@a65e47c7859e:/workspace# export MODEL_PATH=tlt-experiments/model/tlt_facedetectir_vpruned_v1.0/resnet18_facedetectir_pruned.etlt

 root@a65e47c7859e:/workspace# tlt-converter -k tlt_encode -i nchw -d 3,544,960 -o output_bbox/BiasAdd,output_cov/Sigmoid -e $ENGINE_PATH -m 1 $MODEL_PATH
 [INFO] Detected 1 inputs and 2 output network tensors.

 root@a65e47c7859e:/workspace# cd tlt-experiments/model/tlt_facedetectir_vpruned_v1.0/

 root@a65e47c7859e:/workspace/tlt-experiments/model/tlt_facedetectir_vpruned_v1.0# ls -trl
 total 31792
 -rw------- 1 root root        4 Feb 25 01:54 labels.txt
 -rw------- 1 root root     4144 Feb 25 01:54 facedetectir_int8.txt
 -rw------- 1 root root  9528386 Feb 25 01:54 resnet18_facedetectir_pruned.etlt
 -rw-r--r-- 1 root root 23009028 Feb 25 06:05 resnet18_facedetectir_pruned.engine
```
 (3) copy pre-trained model to the container of deepstream
 ```
 $ xhost +
 $ sudo docker run --gpus all -it --rm -v /mnt/docker/tlt-experiments:/workspace/tlt-experiments -v /tmp/.X11-unix:/tmp/.X11-unix --device /dev/video0:/dev/video0:mwr -e DISPLAY=$DISPLAY -w /opt/nvidia/deepstream/deepstream-5.0  nvcr.io/nvidia/deepstream:5.0.1-20.09-triton

 ---in-container---
 root@2de88e2a99a3:/opt/nvidia/deepstream/deepstream-5.0# cp -pr /workspace/tlt-experiments/model/tlt_facedetectir_vpruned_v1.0 samples/models/
 ```
 (4) edit some files in the container of deepstream
 ```
 root@2de88e2a99a3:/opt/nvidia/deepstream/deepstream-5.0# cd samples/configs/tlt_pretrained_models/
 ```
 Edit 2 files below like attached files:
 ```
   config_infer_primary_facedetectir.txt
   deepstream_app_source1_facedetectir.txt
 ```
 (5) run deepstream-app
```
 root@2de88e2a99a3:/opt/nvidia/deepstream/deepstream-5.0# deepstream-app -c samples/configs/tlt_pretrained_models/deepstream_app_source1_facedetectir.txt
```
