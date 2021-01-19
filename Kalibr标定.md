参考网址：https://blog.csdn.net/Hanghang_/article/details/103546033

# 1.标定IMU

#### 1.1  安装ceres
https://github.com/ceres-solver/ceres-solver
http://www.ceres-solver.org/

#### 1.2 下载imu-utils
```
mkdir -p ~/imu_catkin_ws/src
cd ~/imu_catkin_ws/src
catkin_init_workspace
cd ~/imu_catkin_ws 
catkin_make
source ~/imu_catkin_ws/devel/setup.bash
```

```
cd ~/imu_catkin_ws/
git clone https://github.com/gaowenliang/code_utils.git
cd ~/imu_catkin_ws/
catkin_make 
```

```
cd ~/imu_catkin_ws/src/
git clone https://github.com/gaowenliang/imu_utils.git
cd ~/imu_catkin_ws
catkin_make 
```
最后编译时会出现头文件位置错误而导致的报错，根据输出复制相应头文件即可。

#### 1.3 录制IMU数据
```
roscore 
```
```
roslaunch realsense2_camera rs_camera.launch
```
（注意这里的launch文件要修改，acc和gyr要改为true使之输出为imu的主题，并将频率改为200hz）
```
cd ~/imu_catkin_ws      //等下录制到这个文件夹上
rosbag record -O imu_calibration /camera/imu 
```
静置相机至少10min以上，ctr+c停止录制，保存在imu_calibration.bag中


#### 1.4 编写标定的launch文件

创建launch文件
```
cd ~/imu_catkin_ws/src/imu_utils/launch
touch L515_imu_calibration.launch
gedit L515_imu_calibration.launch
```

写入内容：
```
<launch>
    <node pkg="imu_utils" type="imu_an" name="imu_an" output="screen">
        <param name="imu_topic" type="string" value= "/camera/imu"/>
        <param name="imu_name" type="string" value= "L515_imu_calibration"/>
        <param name="data_save_path" type="string" value= "$(find imu_utils)/data/"/>
        <param name="max_time_min" type="int" value= "10"/>
        <param name="max_cluster" type="int" value= "100"/>
    </node>
</launch>
```

#### 1.5 标定IMU
```
source ~/imu_catkin_ws/devel/setup.sh 
rosbag imu_utils L515_imu_calibration.launch 
```

```
source ~/imu_catkin_ws/devel/setup.sh 
cd ~/imu_catkin_ws 
rosbag play -r 200 imu_calibration.bag
```

最终输出结果在 cd ~/imu_catkin_ws/src/imu_utils/data 
文件名为：L515_imu_calibration_imu_param.yaml

#### 1.6 输出结果
```
%YAML:1.0
---
type: IMU
name: L515_imu_calibration
Gyr:
   unit: " rad/s"
   avg-axis:
      gyr_n: 2.3263871478726261e-03
      gyr_w: 3.5295863266256249e-05
   x-axis:
      gyr_n: 1.8151725688281854e-03
      gyr_w: 3.4715279878744823e-05
   y-axis:
      gyr_n: 3.5966765690919182e-03
      gyr_w: 4.7288663744504480e-05
   z-axis:
      gyr_n: 1.5673123056977737e-03
      gyr_w: 2.3883646175519442e-05
Acc:
   unit: " m/s^2"
   avg-axis:
      acc_n: 1.3117344344762888e-02
      acc_w: 4.2272862195087869e-04
   x-axis:
      acc_n: 1.4149112092146242e-02
      acc_w: 6.4517545129483560e-04
   y-axis:
      acc_n: 1.2705066085459557e-02
      acc_w: 2.7625856919581563e-04
   z-axis:
      acc_n: 1.2497854856682870e-02
      acc_w: 3.4675184536198496e-04

```

关键数据：
```
name: L515_imu_calibration
Gyr:
   avg-axis:
      gyr_n: 2.3263871478726261e-03
      gyr_w: 3.5295863266256249e-05

Acc:
   avg-axis:
      acc_n: 1.3117344344762888e-02
      acc_w: 4.2272862195087869e-04
```

# 2. 安装Kalibr
https://github.com/ethz-asl/kalibr/wiki/installation
务必注意Eigen版本（3.3.4可以），以及如果安装过程中有任何的package失败，根据输出结果去debug，一般有头文件、CmakeList之类的错误。


# 3. 相机标定

#### 3.1 标定准备
https://github.com/ethz-asl/kalibr/wiki/downloads
下载棋盘格pdf以及对应Yaml配置文件，我下载了
Aprilgrid 6x6 0.5x0.5 m (unscaled)。

打印时缩小到40%的比例可以打印在A4上，然后修改yaml文件
```
target_type: 'aprilgrid' #gridtype
tagCols: 6               #number of apriltags
tagRows: 6               #number of apriltags
tagSize: 0.022           #size of apriltag, edge to edge [m]
tagSpacing: 0.3          #ratio of space between tags to tagSize
```

#### 3.2 录制
```
roscore 
```
```
roslaunch realsense2_camera rs_camera.launch
```
```
rviz
```

修改录制帧率
```
rosrun topic_tools throttle messages /camera/color/image_raw 4.0 /color
```

录制bag(一分钟左右)
```
rosbag record -O CamL515 /color
```

#### 3.3 标定

```
kalibr_calibrate_cameras --target /home/zhang/Kalibr/L515/april_6x6_50x50cm.yaml --bag /home/zhang/Kalibr/L515/CamL515.bag --bag-from-to 10 100 --models pinhole-radtan --topics /color --show-extraction
```

# 4. Camera-IMU 联合标定

#### 4.1 录制bag
```
roscore 
```
```
roslaunch realsense2_camera rs_camera.launch
```
```
rviz
```

相机20Hz，IMU200Hz，并分别以/color和/imu为话题名发布
```
rosrun topic_tools throttle messages /camera/color/image_raw 20.0 /color
```
```
rosrun topic_tools throttle messages /camera/imu 200.0 /imu
```
录制：(各个方向都要有激励)
```
rosbag record -O L515_VI_dynamic /color /imu
```

#### 4.2 编写标定文件

Camera对应的yaml，就是上文第三节中所得的yaml文件。

IMU对应的yaml：创建yaml文件，其内容格式为：
```
#Accelerometers
accelerometer_noise_density: 1.86e-03   #Noise density (continuous-time)
accelerometer_random_walk:   4.33e-04   #Bias random walk

#Gyroscopes
gyroscope_noise_density:     1.87e-04   #Noise density (continuous-time)
gyroscope_random_walk:       2.66e-05   #Bias random walk

rostopic:                    /imu0      #the IMU ROS topic
update_rate:                 200.0      #Hz (for discretization of the values above)
```

而其中数据，替换为第一节中测得的IMU参数：
关键数据：
```
name: L515_imu_calibration
Gyr:
   avg-axis:
      gyr_n: 2.3263871478726261e-03
      gyr_w: 3.5295863266256249e-05

Acc:
   avg-axis:
      acc_n: 1.3117344344762888e-02
      acc_w: 4.2272862195087869e-04
```

替换后：
```
#Accelerometers
accelerometer_noise_density: 1.31e-02   #Noise density (continuous-time)
accelerometer_random_walk:   4.23e-04   #Bias random walk

#Gyroscopes
gyroscope_noise_density:     2.33e-03   #Noise density (continuous-time)
gyroscope_random_walk:       3.53e-05   #Bias random walk

rostopic:                    /imu     #the IMU ROS topic
update_rate:                 200.0      #Hz (for discretization of the values above)
```
#### 4.3 标定

```
kalibr_calibrate_imu_camera --target /home/zhang/Kalibr/L515/april_6x6_50x50cm.yaml --cam /home/zhang/Kalibr/L515/Camera-IMU/CamL515.yaml --imu /home/zhang/Kalibr/L515/Camera-IMU/IMU.yaml --bag L515_VI_dynamic.bag --show-extraction
```

生成 camchain-imucam-L515_VI_dynamic.yaml