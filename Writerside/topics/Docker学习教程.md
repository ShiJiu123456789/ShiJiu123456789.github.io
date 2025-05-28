# Docker学习教程

## Dockerfile
    #使用miniconda作为基础镜像
    FROM continuumio/miniconda3:4.11.0
    
    #禁止交互
    ENV DEBIAN_FRONTEND=noninteractive
    
    #使用清华镜像
    RUN sed -i -e "s/deb.debian.org/mirrors.tuna.tsinghua.edu.cn/" /etc/apt/sources.list && sed -i -e "s/security.debian.org/mirrors.tuna.tsinghua.edu.cn/" /etc/apt/sources.list
    
    #apt更新，并下载常用软件
    RUN apt update && \
        apt install -y libgl1-mesa-glx && \
        apt install -y vim && \
        apt autoremove -y
    
    #更新pip源
    RUN pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
    
    #下载pytorch
    RUN conda install -y pytorch
    
    #RUN pip3 install --upgrade torch

## 命令

    #这个是根据Dockerfile构建镜像
    docker build -t image_name:0.1 .
    
    #镜像构建成功了可以使用下面命令启动
    docker run --rm -it --name abcd --gpus all -v `pwd`/data:/data image_name:0.1 bash

**对于docker build命令：**

-t :表示tag的意思，即给镜像命名，同时带上版本号

**对于docker run命令：**

--rm :表示容器退出即销毁，一般可以不用

-it :表示i开启交互式模式,，让容器能够持续接收来自标准输入（stdin）的数据。即使你未主动与容器交互，容器的标准输入依然保持开启状态。 t为容器分配一个伪终端,这会让容器的输出看起来如同在终端中一样，能进行格式化和显示颜色等操作。

--name :表示定义容器名，不加则会系统随机一个容器名

--gpus :表示挂载显卡到容器中，一般参数有: all 0 0,2 (其中数字中间加逗号，第一个数字指起始显卡编号，第二个数字指几个显卡)

-v :本地目录和容器目录映射关系(一般用来把模型文件和图像数据等文件映射进去)

bash :进入bash终端？？？



