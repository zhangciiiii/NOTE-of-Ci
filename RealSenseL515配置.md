# RealSense L515 配置记录

## 电脑配置
Ubuntu16, ROS Melodic

# 安装尝试一（最后失败了，仅做记录） 

## Step1:RealSense SDK

### Register the server's public key:
```
sudo apt-key adv --keyserver keys.gnupg.net --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE || sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE
```

### Add the server to the list of repositories:
```
sudo add-apt-repository "deb http://realsense-hw-public.s3.amazonaws.com/Debian/apt-repo xenial main" -u
```

### Install the libraries 
```
sudo apt-get install librealsense2-dkms

sudo apt-get install librealsense2-utils
```

### 可选安装项：
Optionally install the developer and debug packages:
```
sudo apt-get install librealsense2-dev
sudo apt-get install librealsense2-dbg
```
With dev package installed, you can compile an application with librealsense using g++ -std=c++11 filename.cpp -lrealsense2 or an IDE of your choice.

### 卸载
```
dpkg -l | grep "realsense" | cut -d " " -f 3 | xargs sudo dpkg --purge
```

### How to Use:
```
realsense-viewer 
```

## Step2: RealSense ROS

```
export ROS_VER=melodic 
sudo apt-get install ros-$ROS_VER-realsense2-camera
```


### How to use"

#### 发布相机的ROS节点
```
roslaunch realsense2_camera rs_camera.launch
```

#### 查看Topic
```
rostopic list
```


**!!!!! 但是可能会导致一个问题，由于版本不匹配，会导致无法发布IMU的话题！！！因此采用另一种安装方法如下（源码编译安装，更稳妥**


# 安装尝试二

## Step1:RealSense SDK

### 更新
```
sudo apt-get update 
sudo apt-get upgrade 
sudo apt-get dist-upgrade
```

### 下载源码
```
mkdir -p librealsense_install
cd librealsense_install/
git clone -b v2.39.0 https://github.com/IntelRealSense/librealsense.git
```

### 安装依赖
```
sudo apt-get install git libssl-dev libusb-1.0-0-dev pkg-config libgtk-3-dev
sudo apt-get install libglfw3-dev
```

### 安装(过程非常漫长)
```
cd librealsense_install/librealsense
./scripts/setup_udev_rules.sh
./scripts/patch-realsense-ubuntu-lts.sh //这一步很久，但是好像不运行也能编译
mkdir build 
cd build
cmake ../ -DCMAKE_BUILD_TYPE=Release -DBUILD_EXAMPLES=true
sudo make uninstall 
make clean && make -j8
sudo make install
```

在此过程中，可能会出现Ubuntu内核版本过高会导致无法编译（不过好像新版本的RealSense已经兼容高版本内核了，在我自己电脑上配置不满足github 的要求但是还是成功了）
切换内核，参考下列网站：https://blog.csdn.net/qq_42030961/article/details/82740315

查看已有的内核版本：
```
sudo dpkg --get-selections |grep linux-image
```
下载指定内核：
```
sudo apt-get install linux-image-5.0.0-23-generic
```

## Step2:RealSense-ROS
```
mkdir -p ~/realsense_ros/src
cd ~/catkin_ws/src/

git clone -b 2.2.18 https://github.com/IntelRealSense/realsense-ros.git

cd realsense-ros/realsense2_camera
git checkout `git tag | sort -V | grep -P "^\d+\.\d+\.\d+" | tail -1`
sudo apt-get install ros-melodic-ddynamic-reconfigure

cd ~/realsense_ros
catkin_make clean
catkin_make -DCATKIN_ENABLE_TESTING=False -DCMAKE_BUILD_TYPE=Release
catkin_make install

echo "source ~/realsense_ros/devel/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

# 安装完后运行
仅是进行测试，没有对相机进行标定，内参也都是乱的，仅仅是看一下能不能跑通

## ORB_SLAM3 and RealSense(先跑一个有教程的RGBD试一试)
修改 /home/zhang/realsense_ros/src/realsense-ros/realsense2_camera/launch 文件中关于图像分辨率的参数：RGB 1280✖720， Depth 640✖480。
然后修改 src 中 ros_rgbd.cc 关于发布相应话题的命令：image_raw 和 image_rect_raw（可根据摄像头发布的话题名进行修改）
再运行./build_ros.sh 重新编译。

运行：
```
roscore
```
```
rosrun ORB_SLAM3 RGBD /home/zhang/catkin_ws/src/ORB_SLAM3/Vocabulary/ORBvoc.txt /home/zhang/catkin_ws/src/ORB_SLAM3/Examples/ROS/ORB_SLAM3/Asus.yaml
```
```
roslaunch realsense2_camera rs_rgbd.launch
```

## ORB_SLAM3 and RealSense(Mono-IMU)
修改 /home/zhang/realsense_ros/src/realsense-ros/realsense2_camera/launch/rs_camera.launch。注意，此launch文件中默认不输出IMU信息，因此需要修改，否则会不发布IMU的ROS话题。
然后修改 /home/zhang/catkin_ws/src/ORB_SLAM3/Examples/ROS/ORB_SLAM3/src 中 ros_mono_inertial.cc 关于发布相话题的命令：image_raw 和 image_rect_raw（可根据摄像头发布的话题名进行修改）
再运行./build_ros.sh 重新编译。

运行：
```
roscore
```
```
rosrun ORB_SLAM3 Mono_Inertial /home/zhang/catkin_ws/src/ORB_SLAM3/Vocabulary/ORBvoc.txt /home/zhang/catkin_ws/src/ORB_SLAM3/Examples/Monocular-Inertial/L515VI.yaml
```
```
roslaunch realsense2_camera rs_camera.launch
```
