# 人员聚集算法设计_deepseek

## 目标检测模块

### 模型选择：

YOLOv11s（平衡速度与精度，COCO预训练模型，输入尺寸640×640）。

对于一张密集人群的图片，用YOLO的各个版本进行检测



### 过滤干扰：

通过置信度阈值（如conf=0.6）和类别筛选（只保留person类）。

## 人群密度分析

### 聚集判定逻辑：

间距法：计算检测框中心点欧氏距离，若80%以上人员间距<1.5米则触发聚集。

重叠率法：检测框IoU（交并比）>0.1时视为潜在聚集。

热力图辅助：

        # OpenCV密度聚类示例
        points = np.array([[x1,y1], [x2,y2], ...])  # 所有检测框中心点
        criteria = (cv2.TERM_CRITERIA_EPS + cv2.TERM_CRITERIA_MAX_ITER, 10, 1.0)
        _, _, centers = cv2.kmeans(points, K=3, bestLabels=None, criteria=criteria, attempts=10, flags=cv2.KMEANS_RANDOM_CENTERS)

### 动态阈值调整：

根据场景人流量自动调整聚集判定阈值（如商场高峰时段放宽至2米）。

## 行为分析（可选）

### 轨迹追踪：

使用ByteTrack或FairMOT关联多帧检测框，过滤短暂误判。

聚集持续判定：同一区域人员停留时间>30秒（需时间戳记录）。

### 异常行为扩展：

检测快速移动人群（防踩踏）或倒地行为（SOTA模型如ST-GCN）。