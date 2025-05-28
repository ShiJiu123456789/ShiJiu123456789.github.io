# Docker学习教程

## 使用Dockerfile创建一个docker镜像

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


创建一个容器



