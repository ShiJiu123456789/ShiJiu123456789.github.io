# yolov11从pt转bmodel


## onnx准备工作 {#prepare_for_onnx}
可选择从YOLOv8官方主页下载yolov11s.pt模型，或在导出onnx模型中自动下载模型。 安装如下依赖。

    pip3 install ultralytics
    pip3 install ultralytics==8.3.12 #如果需要导出yolov11，需要新版本ultralytics，且需要python版本>=3.10。

找到这个文件：~/.local/lib/python3.8/site-packages/ultralytics/nn/tasks.py。如果找不到，请通过pip3 show ultralytics查包的安装位置。 找到这个函数：
    
    def _predict_once(self, x, profile=False, visualize=False, embed=None):
        ...
        return x

修改返回值，加一个transpose操作，这样更有利于cpu后处理连续取数。将return x修改为：
    
    return x.permute(0, 2, 1)
    

## pt模型转为onnx {#pt_convert_to_onnx}
    from ultralytics import YOLO
    model = YOLO('Yolov11/model/yolo11s.pt')
    model.export(format='onnx', opset=17, dynamic=True)

## 验证onnx模型是否正确
查看onnx模型结构的地址：https://netron.app/

算能模型yolov11模型
![yolov11s_in_sophon.png](yolov11s_in_sophon.png)

我们转的yolov11s模型
<img src="yolov11s_our_convert.png" alt="yolov11s_our_convert"  border-effect="line"/>


## onnx转为bmodel

### 在linux上创建docker容器

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

    docker-compose up -d

运行容器

    docker start ll_yolov11s

    
进入容器, docker exec -it [容器名，如ll_yolov11s] bash

    docker exec -it ll_yolov11s bash

安装依赖TPU-MLIR

    pip install tpu_mlir-1.11b0-py3-none-any.whl

退出容器

    exit

停止容器

    docker stop ll_yolov11s

### 在容器中转为bmodel {#convert_bmodel}

onnx转为mlir

    model_transform.py \
            --model_name yolov11s \
            --model_def ../models/yolo11s.onnx \
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

创建容器后，容器名称为ll_yolov11s，运行容器，再进入容器

cd 到奇哥盒子里面的地址：/data/face_app/data/libs

安装里面的两个文件sophon_arm-3.8.0-py3-none-any.whl和ThingsSDK-1.0.0-py3-none-any.whl

    pip install sophon_arm-3.8.0-py3-none-any.whl
    pip intsall ThingsSDK-1.0.0-py3-none-any.whl

## bmodel模型在盒子上创建的容器ll_yolov11s上运行
因为yolov11和yolov8模型结构兼容，代码是相同的，直接下载yolov8的代码

下载代码：https://github.com/sophgo/sophon-demo/tree/release/sample/YOLOv8_plus_det

运行代码为：yolov8_bmcv.py
![YOLOv8_puls_det.png](YOLOv8_puls_det.png)

修改yolov8_bmcv.py
    
    if __name__ == "__main__":
        args = argsparser()
        #修改需要测试的图片所在的路径
        args.input="/workspace/YOLOv11/YOLOv8_plus_det/datasets/test"
        #修改bmodel模型所在的路径
        args.bmodel= "/workspace/YOLOv11/YOLOv8_plus_det/models/BM1688/yolov11s_fp32_1b.bmodel"
        main(args)
        print('all done.')