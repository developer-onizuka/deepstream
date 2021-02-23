```
https://ngc.nvidia.com/catalog/containers/nvidia:deepstream

$ sudo docker pull nvcr.io/nvidia/deepstream:5.0.1-20.09-triton
$ xhost +
$ sudo docker run --gpus all -it --rm -v /tmp/.X11-unix:/tmp/.X11-unix --device /dev/video0:/dev/video0:mwr -e DISPLAY=$DISPLAY -w /opt/nvidia/deepstream/deepstream-5.0  nvcr.io/nvidia/deepstream:5.0.1-20.09-triton

root@db8c980a463f:/opt/nvidia/deepstream/deepstream-5.0# deepstream-app -c samples/configs/deepstream-app/source1_usb_dec_infer_resnet_int8.txt
```
