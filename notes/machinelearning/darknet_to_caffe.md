# Darknet to Caffe

# 一、Darknet to Caffe（X86_64）

> caffe 不支持 shortcut、route、upsample、yolo等这些yolov3的层
>
> shortcut 可以用 eltwise 替换
>
> route 可以用 concat 替换
>
> upsample 可以添加一层代码，重新编译caffe
>
> yolo 只能自己写

##### 1、拉取caffe镜像

[Dockerfile](https://hub.docker.com/r/bvlc/caffe/dockerfile)

[git](https://github.com/hlyani/caffe)

```
docker pull bvlc/caffe:cpu
```

##### 2、进入容器

[hlyani/darknet2caffe](https://github.com/hlyani/darknet2caffe)

```
git clone https://github.com/hlyani/darknet2caffe
```

```
docker run -it -v /root/yolov3:/srv/ -w /srv --name yolov3 bvlc/caffe:cpu bash
```

```
apt update && apt install vim

pip install torch future -i https://pypi.tuna.tsinghua.edu.cn/simple
```

##### 3、修改caffe环境

```
cp upsample_layer.hpp /opt/caffe/include/caffe/layers/
cp upsample_layer.cpp upsample_layer.cu /opt/caffe/src/caffe/layers/

vim /opt/caffe/src/caffe/proto/caffe.proto
...
//  optional WindowDataParameter window_data_param = 129;
  ...
  optional UpsampleParameter upsample_param = 149;
//}
...

...
//message PReLUParameter {
...
//}
...
message UpsampleParameter{
  optional int32 scale = 1 [default = 1];
}
```

##### 4、重新编译 caffe

[参考Dockerfile](https://github.com/hlyani/caffe/blob/master/docker/gpu/Dockerfile)

```
cd /opt/caffe/build
cmake -DUSE_CUDNN=1 -DUSE_NCCL=0 ..
make -j"$(nproc)"
```

##### 5、转换

[darknet2caffe](https://github.com/ChenYingpeng/darknet2caffe)

```
python darknet2caffe.py voc.cfg yolov3_tiny.weights yolov3-tiny.prototxt yolov3-tiny.caffemodel
```

##### Todo: yolo 层转换

# 二、.caffemodel to .om

##### 1、安装环境

```
参考：Atlas 300 编译、部署、使用手册-20210826.docx
```

[Atlas 300 编译、部署、使用手册-20210826.docx](https://github.com/hlyani/darknet2caffe_sample/blob/main/doc/Atlas%20300%20%B1%E0%D2%EB%A1%A2%B2%BF%CA%F0%A1%A2%CA%B9%D3%C3%CA%D6%B2%E1-20210826.docx)

[https://github.com/hlyani/detection](https://github.com/hlyani/detection)

##### 2、转换

> 使用上面转换的.caffemodel

```
atc --model=yolov3-tiny.prototxt --weight=yolov3-tiny.caffemodel --framework=0 --output=yolov3-tiny.om --soc_version=Ascend310
```

##### 3、运行 sample

[git@github.com:hlyani/darknet2caffe_sample.git](git@github.com:hlyani/darknet2caffe_sample.git)

```
git clone git@github.com:hlyani/darknet2caffe_sample.git

```

> 拷贝转换的模型，再执行

```
python yolov3_read_avi.py
```

