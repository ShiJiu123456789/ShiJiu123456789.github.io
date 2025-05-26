# APP打包

![code_of_pack_up_app.png](code_of_pack_up_app.png)

### framework中

app存放所有业务相关的代码. 
其中start.py是提供的给业务的开发打包的文件

config存放云端提供的一些配置

mods存放模型bmodel文件

utils存放算能盒子提供的底层的一些计算工具


### tools中

build_tools提供打包需要的文件

打包app时运行

    sh app_docker_build.sh

app_docker_build.sh文件中的内容为

    cd ../../ #将位置定位到1688_comm

    echo $(pwd) #当前位置
    echo 开始编译 app镜像
    docker-compose -f tools/build_tools/docker_compose.yml build
    echo app镜像 编译完成

docker_compose.yml文件中的内容为

    version: '3' #版本号
    services:
      single_image: #服务名字,打包一个镜像可以使用多个服务
        image: single_image:v1.0 #镜像名字
        privileged: true #设定root特权
        build:
          context: ../../
          dockerfile: tools/build_tools/app.dockerfile

app.dockerfile文件中的内容将某个服务中的镜像镜像融入到新镜像的过程

    FROM ubuntu:20.04

    ENV DEBIAN_FRONTEND=noninteractive
    
    # 设置时区环境变量
    ARG TZ=Asia/Shanghai
    ENV TZ=${TZ}
    
    # 安装基础工具、Python3 和时区数据，并配置时区
    RUN apt-get update \
        && apt-get install -y --no-install-recommends tzdata python3-pip \
        # 配置时区
        && ln -fs /usr/share/zoneinfo/${TZ} /etc/localtime \
        && echo "${TZ}" > /etc/timezone \
        && dpkg-reconfigure -f noninteractive tzdata \
        # 配置 pip 源
        && pip3 config set global.index-url https://pypi.mirrors.ustc.edu.cn/simple \
        && pip3 config set global.trusted-host 192.168.2.100:8002 \
        # 清理无用文件
        && apt-get clean \
        && rm -rf /var/lib/apt/lists/*
    
    RUN pip3 install opencv-python-headless pyyaml requests websocket-client rel
    
    # 复制data和平台库文件
    COPY framework/utils/sophon/libsophon-0.4.10/lib/*.so* /usr/lib/
    COPY framework/utils/sophon/sophon-ffmpeg_1.8.0/lib/*.so* /usr/lib/
    COPY framework/utils/sophon/sophon-opencv_1.8.0/lib/*.so* /usr/lib/
    COPY framework/utils/sophon/sophon-soc-libisp_1.0.0/lib/*.so* /usr/lib/
    
    COPY framework/docs/. /workspace/docs/
    
    # 复制启动脚本
    COPY tools/build_tools/app_start.sh /workspace/start.sh #copy 盒子地址 容器地址
    RUN chmod +x /workspace/start.sh
    
    # 安装SDK和其他py依赖
    COPY framework/libs/. /workspace/libs/
    RUN pip install /workspace/libs/*.whl
    
    # 复制业务代码 和模型
    COPY framework/app/. /workspace/app/
    COPY framework/config/. /workspace/config/
    COPY framework/mods/. /workspace/mods/
    
    
    CMD ["/bin/bash","/workspace/start.sh"]

start.sh文件中的内容

    set -e
    
    python3 -u /workspace/app/start.py

查看当前镜像

    docker images

![pack_up_app_image.png](pack_up_app_image.png)

其中的single_image为刚刚打包出来的镜像,也就是打包好的app

使用镜像,创建一个临时容器,即可执行app

    docker run -it --rm --privileged 8edcac0e1ed1

欢迎收看！

## 遇到问题

    ERROR: Couldn't connect to Docker daemon at http+docker://localhost - is it running?

**解决办法**