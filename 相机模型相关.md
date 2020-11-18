# ROS 初步学习
参考资料：《ROS机器人项目开发11例》。图书馆编号：TP242/A70-2 2018

## 文件系统级（在硬盘上如何组织）

* 综合功能包：保存相关功能包的信息，帮助安装功能包
* 功能包：ROS中软件的主要组织形式，一个功能包的单个模块中包括ROS节点、进程、数据集和配置文件。
* 功能包清单：每个包中有一个名为package.xml的清单文件，其中有包的名称、版本、作者等信息
* 消息(msg)：ROS通过发送ROS消息进行通信，可在具有.msg扩展名的文件中定义消息数据的类型，称为消息文件。保存在our_package/msg/message_files.msg下
* 服务(srv)：服务定义在our_package/srv/service_files.srv下

## 计算图级

* 节点：ROS节点是一个使用ROS API进行通信的进程，机器人使用多个节点来执行计算。可以使用roscpp和rospy等ROS客户端库创建ROS节点。
* 节点管理器：作为一个中间节点，在ROS不同节点间建立连接。节点管理器有ROS环境中所有运行节点的全部细节。它与另一个节点交换细节，建立他们间的连接，交换信息后，两个ROS节点开始通信
* 参数服务器：节点可以在参数服务器中存储变量，并设置其私有属性。如果参数有全局域，则可以由其他所有节点访问。
* 消息：节点以ROS消息的形式发送和接收数据，ROS消息是ROS节点用于交换数据的数据结构。
* 主题：主题是两个ROS节点交换消息的方法之一。每个主题有特定名称，一个节点将数据发布到主题，另一个节点可以通过订阅从主题中读取。
* 服务：不同于主题的另一种通信方式，一个节点作为服务提供者，运行服务例程，客户端节点可以从服务器节点请求服务，服务器将执行服务程序并将结果发送给客户端。
* 消息记录包：用于录制和播放ROS主题。我们有时需要在没有实际硬件的情况下工作。使用rosbag可以记录传感器数据，并可以将消息记录包文件复制到其他计算机，通过回放来检查数据。

## 节点如何使用ROS主题通信
对于两个名为 talker 和 listener 的节点，talker 将"Hello world"的字符串发布到名为 /talker 的主题中，且listener订阅了此主题。可分为如下三个阶段：

(1)在运行任何的ROS节点前，启动ROS节点管理器，启动后，它将等待节点。talker节点（发布者）运行时，它先连接至ROS节点管理器并与主机交换发布主题的详细信息，包括主题名称、消息类型和发布节点的URI。

(2)当启动listener节点（订阅者），它将连接到主节点并交换节点的细节，包括其要订阅的主题、其消息类型和节点URI。

(3)当有相同主题的订阅者和发布者，主节点将交换订阅者和发布者的URI，这将有助于两个节点彼此连接并交换数据。在它们相互连接后，节点管理器没有任何作用，数据不会流经节点管理器，节点是相互直连并交换消息的。


# Launch文件
在ROS应用中，一般涉及多个节点，而每个节点又有很多参数需要设置。为了方便、高效地操作多个节点，可以编写 .launch 文件，然后用roslaunch命令运行。launch文件包含的内容：
* \<node>：要启动的ROS节点，参数包括：
```
    pkg=“mypackage” :节点的功能包
    type="nodetype" :节点的类型
    name="nodename" :节点的名字
    args="arg1 arg2 arg3" (可选):传递给节点的参数
    respawn="true" (可选) :如果节点停止，自动重启节点
    ns="foo" (可选):在"foo"命名空间启动节点
    output="log | screen"  (可选) :如果选择screen，节点的stdout（标准输出）和stderr(标准错误)信息将显示在终端窗口上；如果选择log，stdout和stderr将发送一个log文件,stderr也会显示在终端窗口上
```

实例：
```
<node name="gazebo" pkg="gazebo"  type="gazebo"   args="$(find r2_gazebo)/gazebo /r2_grap.world"    output="screen" respawn="false"/>
```
节点名：`gazebo`；节点功能包：`gazebo`；节点类型：`gazebo`；参数：`r2_grap.world`；节点的stdout和stderr将显示在终端窗口上。如果gazebo节点停止，不能自动重新启动。

=====================================================

* \<param> 用来定义一个设置在Parameter Server(参数服务器)的参数，它可以添加到\<node>中，参数包括：
```
name="namespace/name":参数的名字
value="value"(可选):定义参数的值，如果省略这个参数，则应该指定一个文件（binfile/texfile）或命令
type="str|int|double|boot"(可选):指定参数的类型
texfile="$(find pkg-name)/path/file" (可选)
binfile="$(find pkg-name)/path/file" (可选)
command="(find pkg-name)/exe '$(find pkg-name)/arg.txt'" (可选):exe是可执行文件( .py或 .cpp)，arg.txt是参数文件
```

实例：
```
<param name="robot_description" command="$(find xacro)/xacro.py '$(find r2_gazebo) /robots/r2.sim.urdf.xacro'"/>
```
参数名：`robot_description`；调用的执行文件：`xacro.py`

=====================================================
* \<include> 可以在当前launch文件中调用另一个launch文件
实例：
```
<include file ="$(find pr2_controller_manager)/controller_manager.launch"/>
```

* \<env> 
<env>可以用来设置节点的环境变量，可以在\<launch>、\<include>、\<node>等。参数包括：
```
name="enviroment-variable-name" :设置的环境变量的名字
value="environment-variable-value" :环境变量的值
```
* \<remap>可以将一个参数名映射为另一个名字，它的参数如下：
```
from="original-name"
to="new-name"
```

* \<arg>用来定义一个局部参数，该参数只能在一个launch文件中使用。\<arg>有三种使用方法：
```
<arg name="foo"/> :声明一个参数foo，后面需要给它赋值
<arg name="foo" default="1"/> :声明一个参数foo，它有一个默认值，该值可以被修改。
<arg name="foo" value="bar"/> :声明一个常量foo，它的值不能被修改。
```