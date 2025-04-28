# yolov11从pt转bmodel

## pt模型转为onnx

## onnx转为bmodel

### 创建容器

在对应项目目录（\{PWD}）下新建docker-compose.yml（ sophgo/tpuc_dev:v3.2镜像需要翻墙）

    version: "3"
    services:
      tpuc_dev:
        image: sophgo/tpuc_dev:v3.2
        container_name: ll_yolov11s #容器名
        privileged: true
        volumes:
          - ${PWD}/YOLOv11:/workspace #${PWD}/YOLOv11为要映射到容器里面的文件夹地址，/workspace为容器里面的文件夹地址
        tty: true
        stdin_open: true

docker-compose.yml所在目录下运行代码创建docker容器，容器名为ll_yolo11s

    docker compose up -d

运行容器

    docker start ll_yolov11s

    
进入容器, docker exec -it [容器名，如ll_yolov11s] bash

    docker exec -it ll_yolov11s bash

退出容器

    exit

停止容器

    docker stop ll_yolov11s

### 在容器中转为bmodel {#convert_bmodel}

onnx转为mlir

    model_transform.py \
            --model_name yolov11s \
            --model_def ../models/road_signs.onnx \
            --input_shapes [[1,3,640,640]] \
            --mean 0.0,0.0,0.0 \
            --scale 0.0039216,0.0039216,0.0039216 \
            --keep_aspect_ratio \
            --pixel_format rgb  \
            --output_names output0 \
            --mlir yolov11s.mlir 

mlir转为bmodel

    model_deploy.py \
            --mlir yolov11s.mlir \
            --quantize INT8 \
            --chip bm1688 \
            --model yolov11s_fp32.bmodel \
            --num_core 2


## 盒子上创建docker容器
cd 到奇哥盒子里面的地址：/data/face_app

    cd /data/face_app


修改docker-compose.yml内容

    version: "3"
    services:
      test:
        image: ubuntu_2004_py_app #已经在盒子上创建的镜像名称
        user: "root:root"
        container_name: ll_yolov11s
        privileged: true
        environment:
            - TZ=Asia/Shanghai
            - LD_LIBRARY_PATH=/opt/sophon/sophon-sail/lib:/opt/sophon/libsophon-0.4.9/lib:/opt/sophon/sophon-ffmpeg-latest/lib:/opt/sophon/sophon-opencv-latest/lib/:/opt/sophon/sophon-soc-libisp_1.0.0/lib
        volumes:
            - ./data:/workspace:rw
            - /opt/sophon:/opt/sophon
            - /data/LL/YOLOv11:/workspace/YOLOv11:rw
        ports:
            - "2222:22"
        restart: always



## bmodel模型在盒子上运行


 