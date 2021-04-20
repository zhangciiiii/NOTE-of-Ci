## 1. 如何利用YoloROS发布的信息
调用YoloV3:
```
source devel/setup.bash 
roslaunch darknet_ros yolo_v3.launch
```

查看数据内容：
网上显示，bounding_boxes ([darknet_ros_msgs::BoundingBoxes])：
Publishes an array of bounding boxes that gives information of the position and size of the bounding box in pixel coordinates.
查看 bounding_boxes 的数据类型
```
rostopic type /darknet_ros/bounding_boxes
```
输出显示：darknet_ros_msgs/BoundingBoxes。查看这个里面是什么数据：

```
source devel/setup.bash 
rosmsg show darknet_ros_msgs/BoundingBoxes
```

结果显示：
```
std_msgs/Header header
  uint32 seq
  time stamp
  string frame_id
std_msgs/Header image_header
  uint32 seq
  time stamp
  string frame_id
darknet_ros_msgs/BoundingBox[] bounding_boxes
  float64 probability
  int64 xmin
  int64 ymin
  int64 xmax
  int64 ymax
  int16 id
  string Class
```

在 rosbag play的过程中查看发布了什么
```
rostopic echo /darknet_ros/bounding_boxes
```
输出显示：
```
---
header: 
  seq: 628
  stamp: 
    secs: 1618816774
    nsecs: 558836864
  frame_id: "detection"
image_header: 
  seq: 4457
  stamp: 
    secs: 1616640527
    nsecs: 228025436
  frame_id: "camera_color_optical_frame"
bounding_boxes: 
  - 
    probability: 0.962878763676
    xmin: 472
    ymin: 245
    xmax: 554
    ymax: 290
    id: 2
    Class: "car"
  - 
    probability: 0.833207130432
    xmin: 530
    ymin: 239
    xmax: 583
    ymax: 260
    id: 2
    Class: "car"
  - 
    probability: 0.668122410774
    xmin: 402
    ymin: 248
    xmax: 420
    ymax: 261
    id: 2
    Class: "car"
  - 
    probability: 0.621477127075
    xmin: 559
    ymin: 242
    xmax: 635
    ymax: 272
    id: 2
    Class: "car"
---
```

### 如何利用名为 /darknet_ros/bounding_boxes 的话题
https://blog.csdn.net/weixin_28930461/article/details/110530853
https://github.com/leggedrobotics/darknet_ros/issues/190

