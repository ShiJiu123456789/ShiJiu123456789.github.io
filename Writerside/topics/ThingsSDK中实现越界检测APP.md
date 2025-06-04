# ThingsSDK中实现越界检测APP

### 主要代码

    def AppMain():
        # config = Configer(useCloud=True)
        config = Configer(useCloud=True, local_config_path='/workspace/YOLOv11/1688_comm/framework/config/app.yml')
        all_vhds = config.video_handler # 获取视频输入源（如摄像头、视频文件）
        all_output = config.output  # 获取输出配置（如日志、API推送）
    
        # 加载模型
        model1 = YoloV8Predict(conf_thresh=0.25, nms_thresh=0.7)
        model1.LoadModel('/workspace/YOLOv11/1688_comm/framework/mods/yolov8_foot.bmodel',
                         '/workspace/YOLOv11/1688_comm/framework/mods/yolov8_foot.txt')
    
        handle = Handle(0)
        bmcv = Bmcv(handle)
    
        while True:
            for vhd in all_vhds.values():
                frame = vhd.get_frame() # 获取当前帧
    
                bmcv.imwrite(f'/workspace/YOLOv11/{frame.frame_name}.jpg', frame.frame) #写出一张图片
    
                result = model1.predict(frame)
                result = [x for x in result if x.classId == 2]  # 筛选出行人
                print(result)
                inside_results = is_inside(result)  # 检测出独人员聚集的结果
                
                logger.info(f'人员越界 检测结果: {inside_results}')
    
                if result: #上传数据
                    for o in all_output.values():
                        o.push('TEXT', result)
                        o.push('BINARY', frame)


### 读取一张jpg图片，转化为frame格式

    img_file = '/workspace/YOLOv11/frame_1749008106662.jpg'
    img_file = '/workspace/YOLOv11/YOLOv8_plus_det/datasets/test/1.png'
    original_img = cv2.imread(img_file)

    yolov8_handle = sail.Handle(0)
    # decode
    decoder = sail.Decoder(img_file, False, 0)
    bmimg = sail.BMImage() #图片解码后的数据
    ret = decoder.read(yolov8_handle, bmimg) #解码过程

    py_image = PYImage()  # 包装一层Image
    py_image.frame_name = 'frame_1749008106662.jpg'
    py_image.camera = '123456'
    py_image.frame = bmimg


### 传入多边形区域

暂时使用config配置文件传入polygon

![add_polygon_to_app_yaml.png](add_polygon_to_app_yaml.png)

### 调试工具

在最下面输入要调试的变量，调试控制台会返回该变量的具体内容。

![debugger_control.png](debugger_control.png)