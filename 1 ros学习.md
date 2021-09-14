```cpp
【古月居】ros21讲 https://www.bilibili.com/video/BV1zt411G7Vn?from=search&seid=765777793691750397
【代码】https://github.com/tshooting/ros21guyueju/tree/master
安装ubuntu https://www.cnblogs.com/rollupbytian/p/14526163.html
安装ros https://www.cnblogs.com/rollupbytian/p/14528968.html
```

# 1. 基本知识

```cpp
3.1命令
cd pwd mkdir ls ll touch mv cp rm  sudo
ros4 C++/Python极简基础
4.1 安装g++ python
sudo apt-get install g++
sudo apt-get install python
ros5 安装ROS系统
5.1 测试小乌龟
roscore
rosrun turtlesim turtlesim_node
rosrun turtlesim turtle_teleop_key
ros6 ROS是什么
6.1
ros = 通信机制+开发工具+应用功能+生态系统 提高机器人研发中的软件复用率
通信机制 松耦合分布式通信
开发工具 QT Rviz gazebo TF坐标变换
应用功能 Navigation slam movelt
生态系统：
发行版 软件源 ROS wiki 邮件列表 ROS answers blog
```

# 2. ROS核心单元

```cpp
ros7.ROS中的核心概念 
7.1节点 node 执行单元 
执行具体任务的进行、独立运行的可执行文件； 分布式；在系统中的名字唯一 
7.2节点管理器 node master 控制中心 
为节点提供命名和注册服务； 
跟踪和记录话题服务的通信，辅助节点互相查找 建立链接； 
提供参数服务器，节点使用此服务器存储和检索运行时的参数 
7.3 话题 topic 异步通信机制 
节点间用来传输数据的重要总线 使用发布和订阅模式，数据由发布者传输到订阅者 
7.4 消息 message 话题数据 
具有一定类型和数据结构，包括ros提供的标准类型和用户自定义类型 
使用编程语言无关的.msg文件定义，编译过程中生成对应的代码文件 
7.5 服务通信 服务 [话题是异步的，服务是同步的]
service 同步通信机制 使用C/S，请求request和应答response
使用的是.srv文件定义请求和应答数据结构，编译过程中生成对应的代码 
7.6 参数 parameter 全局共享字典 
可以通过网络访问的共享、多变量字典 
节点使用此服务器来存储和检索运行时的参数 
适合存储静态、非二进制的配置参数，不存储动态配置的数据 
7.7 文件系统 
功能包（Package） 
ros软件中的基本单元，包含节点源码、配置文件、数据定义等。 
功能包清单 package manifest 
记录功能包的基本信息，包含作者信息，许可信息，依赖选项、编译标志等 
元功能包 metapackages 组织多个用于同一目的功能包 
```

# 3. ROS命令行工具

```cpp
8.1 以小海龟为例 
roscore 【ros master】 
rosrun turtlesim turtlesim_node 【rosrun 功能包的名字 功能包里面的节点】 
rosrun turtlesim turtle_teleop_key 【rosrun 功能包的名字 功能包里面的节点】 
8.2rqt_graph : 用qt写的，可以看到系统中的运行图 
显示两个节点，仿真器节点和键盘控制节点，和话题 
8.3rosnode list 显示系统中所有的节点，rosout是采集所有节点的日志提交给上面的界面所做显示 
8.4rosnode info /turtlesim 显示节点的信息 
现在这个节点，会发布一些话题，然后订阅一个话题，提供一些服务
8.5rostopic list 显示当前所有的话题 
发布话题消息 -r是循环 10是每秒10次，话题 消息
8.6rostopic pub -r 10 /turtle1/cmd_vel geometry_msgs/Twist 
"linear:
x: 1.0 
y: 0.0 
z: 0.0 
angular: 
x: 0.0 
y: 0.0 
z: 1.0"
8.7rosmsg show geometry_msgs/Twist 
可以查看这个消息的类型 有线速度和角速度 
8.8rosservice list 查看所有的服务
8.9发布请求服务 生成一个新的海龟 
rosservice call /spawn "x: 2.0 y: 4.0 theta: 0.0 name: 'turtle2'" 
会有一个response 发现话题多了turtle2有了
8.10话题记录 rosbag record -a -O ~/Desktop/cmd_record 结束之后按ctrl+s 不要按ctrl+z
先执行上面3个命令，然后再开启这个话题记录终端，然后再使用键盘让小乌龟运动，之后按ctrl+s 然后关闭所有的命令roscore之类的
8.11 话题复现 只打开roscore和仿真器节点，然后运行 rosbag play ~/Desktop/cmd_record.bag
```

# 4. 创建工作空间与功能包

```cpp
9.1工作空间（workspace）是存放工程开发相关文件的文件夹
src（代码空间），build（编译空间），devel（开发空间），install（安装空间）
9.2创建工作空间
mkdir -p ~/catkin_ws/src【可以只执行这一步】
cd ~/catkin_ws/src
catkin_init_workspace
编译
cd ~/catkin_ws/
catkin_make[进入这个文件夹去编译]
设置环境变量
source devel /setup.bash
检查环境变量
echo $ROS_PACKAGE_PATH
9.3 创建功能包
cd ~/catkin_ws/src
catkin_create_pkg test_pkg std_msgs rospy roscpp
编译功能包
cd ~/catkin_ws
catkin_make
source ~/catkin_ws/devel/setup.bash【不能在fish下面，要在最初的终端下面】
同一个工作空间下，不允许存在同名功能包，到那时不同工作空间下就允许存在同名的功能包了
使用sudo gedit ~/.bashrc 可以查看
```

# 5. 发布者Publisher的编程实现

```cpp
10.1创建一个功能包 然后添加依赖
cd ~/catkin_ws/src
catkin_create_pkg learning_topic roscpp rospy std_msgs geometry_msgs turtlesim
10.2 创建发布者代码
（1）首先创建ros节点
（2）向ros master 注册节点信息，包括 发布的话题名字和话题中的消息类型 还有就是队列长度（只保留最新的10个）
（3）创建消息数据
（4）按照一定频率循环发布消息

调试，在vscode里面写代码发现没有ros/ros.h的路径，执行下面
https://blog.csdn.net/baidu_38634017/article/details/99875321
https://blog.csdn.net/suoniyusanxing/article/details/107325660?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_title-0&spm=1001.2101.3001.4242
扩展程序会根据当前系统环境配置基本信息，因此有可能配置不完整，这时需要通过生成c_cpp_properties.json文件来配置缺少的信息：
ctrl+shift+P打开Command Palette,运行C/Cpp: Edit configurations...生成c_cpp_properties.json：
在includePath中添加"/opt/ros/kinetic/include/*" ，也就是本地安装的ros路径
/**
 * 发布的是turtle1/cmd_vel话题，消息类型geometry_msgs::Twist
 * */
#include<ros/ros.h>
#include<geometry_msgs/Twist.h>
int main(int argc,char ** argv)
{
    // ros node init  the name of publisher node
    ros::init(argc,argv,"velocity_publisher");
    // create node handle node master
    ros::NodeHandle n;
    // create a publisher && topic name && msg type && queue length
    ros::Publisher turtle_vel_pub = n.advertise<geometry_msgs::Twist> ("/turtle1/cmd_vel",10);
    //set cycle frequency 10times/sec
    ros::Rate loop_rate (10);
    int count = 0;
    while(ros::ok()){
        //init msg 
        geometry_msgs::Twist vel_msg;
        vel_msg.linear.x = 0.5;//now y and z is zero
        vel_msg.angular.z = 0.2;
        //publish msg
        turtle_vel_pub.publish(vel_msg);
        //like cout
        ROS_INFO("Publish turtle velocity command [%0.2f m/s, %0.2f rad/s]",vel_msg.linear.x,vel_msg.angular.z);
        //delay according to cycle frequency  
        loop_rate.sleep();
    }
    return 0;
}
10.3 配置发布者代码编译规则
在CMakeLists.txt中编译规则
154行，因为会分区域，build区域，install 区域
add_executable(velocity_publisher src/velocity_publisher.cpp)
target_link_libraries(velocity_publisher ${catkin_LIBRARIES})

10.4 编译并且运行发布者
（1) 先去编译
cd ~/catkin_ws
catkin_make[编译]
source devel/setup.bash[以前设置过这一句话，现在就不用再设置了]
（2）启动命令

roscore [启动 master]
rosrun turtlesim turtlesim_node[启动仿真器节点，仿真器节点会订阅 /turtle1/cmd_vel 话题，我们自己设置的发布者就是发布这个话题的]
rosrun learning_topic velocity_publisher[启动功能包下面的节点]
就会发现小海龟在动起来了，终端learning_topic velocity_publisher下面会打印rosinfo的信息
对于Python 要注意是让这个.py的源代码的属性变为可以编译成可执行文件的
```

# 6. 订阅者Subscriber的编程实现

```cpp
1 /**
 2  * 现在是实现一个订阅者
 3  * 首先初始化ros节点【这个时候还不知道这个节点的角色，一个节点可以是发布者同时也可以是订阅者】
 4  * 创建句柄
 5  * 
 6  * 创建订阅者 让这个节点去订阅话题
 7  * 循环等待话题消息，接收到消息后进入回调函数
 8  * 在回调函数中完成消息的处理
 9  * 
10  * */
11  
12 #include<ros/ros.h>
13 #include"turtlesim/Pose.h"
14 void poseCallback(const turtlesim::Pose::ConstPtr & msg)
15 {
16     ROS_INFO("Turtle Pose : x:%0.6f,y:%0.6f",msg->x,msg->y);
17 }
18 int main( int argc,char ** argv){
19     //初始化ros节点
20     ros::init(argc,argv, "pose_subscriber");
21
22     //创建句柄
23     ros::NodeHandle n;
24 
25     //创建一个订阅者，订阅者的名字和队列长度，注册回调函数
26     ros::Subscriber pose_sub = n.subscribe("/turtle1/pose",10,poseCallback);
27   
28     //循环等待回调函数
29     ros::spin();
30   
31     return 0;
32 }
33 add_executable(pose_subscriber src/pose_subscriber)
34 target_link_libraries(pose_subscriber ${catkin_LIBRARIES})
```

# 7. 话题消息的定义与使用

## 7.1 基本用法

```cpp
12.1
（1）首先定义一个msg文件（在learning_topic下面创建一个msg文件夹，在里面创建）
string name
uint8 sex
uint8 age
uint8 unknown = 0
uint8 male = 1
uint8 female = 2
（2）然后在Package.xml[learning_topic下面的Package.xml]里面添加功能包编译依赖和运行依赖
<build_depend>message_generation</build_depend>
<exec_depend>message_runtime</exec_depend>
，最后需要在CMakeLists.txt里面添加编译选项
find_package( ...... message_generation)
add_message_files(FILES Person.msg)
generate_messages(DEPENDENCIES std_msgs)
catkin_package(...... message_runtime)
（3）编译之后会自动生成.h文件，在catkin_ws 下面编译 catkin_make
12.2 然后定义一个发布者和订阅者测试代码
 1 /**
 2  * 
 3  * 现在是创建一个话题的发布者
 4  * 1 初始化ros节点
 5  * 2 创建句柄
 6  * 3 创建发布者 话题名字 队列长队
 7  * 4 设置发布频率
 8  * 5 创建循环，设置消息，发送消息，按照频率进行延迟
 9  * 
10  * */
11 #include<ros/ros.h>
12 #include "learning_topic/Person.h"
13 int main(int argc,char ** argv)
14 {
15     //初始化ros节点
16     ros::init(argc,argv,"person_publisher");
17     //创建句柄
18     ros::NodeHandle n;
19     //创建一个发布者learning_topic::Person消息类型 话题是/person_info
20     ros::Publisher person_info_pub = n.advertise<learning_topic::Person>("/person_info",10);
21     //设置循环频率
22     ros::Rate loop_rate(1);
23     int count = 0;
24     while(ros::ok()){
25         //初始化learning_topic::Person 类型的消息
26         learning_topic::Person person_msg;
27         person_msg.name = "Tom";
28         person_msg.age = 18;
29         person_msg.sex = learning_topic::Person::male;
30         //发布消息
31         person_info_pub.publish(person_msg);
32         ROS_INFO("Publish Person Info : name : %s  age : %d sex : %d",person_msg.name.c_str(),person_msg.age,person_msg.sex);
33         //按照循环频率延时
34         loop_rate.sleep();
35
36     }
37     return 0;
38 }
39 /**
40  * 创建一个订阅者，需要导入消息的类
41  * 定义回调函数
42  * 初始化ros节点
43  * 创建句柄
44  * 创建订阅者 
45  * 循环等待回调函数
46  * */
47 #include<ros/ros.h>
48 #include"learning_topic/Person.h" 
49 void personInfoCallback(const learning_topic ::Person::ConstPtr & msg){// 在这个地方去获得消息 learning_topic 功能包
50     ROS_INFO("Subscribe Person Info : name : %s age : %d sex: %d",msg->name.c_str(),msg->age,msg->male);
51 }
52 int main(int argv,char ** argc){
53     //init node
54     ros::init(argv,argc,"person_subscriber");
55   
56     ros::NodeHandle n;
57 
58     ros::Subscriber person_info_sub = n.subscribe("/person_info",10,personInfoCallback);
59 
60     ros::spin();
61     return 0;
62 }
63 CMakeLists.txt
64 add_executable(person_publisher src/person_publisher.cpp)
65 target_link_libraries(person_publisher ${catkin_LIBRARIES})
66 add_dependencies(person_publisher ${PROJECT_NAME}_generate_messages_cpp)
67 
68  
69 add_executable(person_subscriber src/person_subscriber.cpp)
70 target_link_libraries(person_subscriber ${catkin_LIBRARIES})
71 add_dependencies(person_subscriber ${PROJECT_NAME}_generate_messages_cpp)
72 
73  
写完之后进行编译
cd ~/catkin_ws
catkin_make
roscore
rosrun learning_topic person_publisher
rosrun learning_topic person_subscriber
```

## 7.2 补充用法

```cpp
1.在消息中如果是 vector的形式中间不能有空格
float32[] data(正确)
float32 [] data(错误)
```

# 8. 客户端Client的编程实现

```cpp
（1）创建一个节点
（2）发现某个服务之后，创建一个客户端， （请求的数据类型也要知道）
（3）初始化请求数据
 (4）请求服务调用，会有响应显示调用的结果
catkin_create_pkg learning_service roscpp rospy std_msgs geometry_msgs turtlesim
 1 /**
 2  * 创建一个客户端，请求/spawn，服务的数据类型是 turtlesim::Spawn
 3  * 初始化ros节点
 4  * 发现服务创建一个客户端实例
 5  * 设置请求服务数据，发布请求
 6  * 等待server处理之后的应答结果
 7  * */
 8 #include<ros/ros.h>
 9 #include<turtlesim/Spawn.h>
10 int main(int argc, char ** argv)
11 {
12     //初始化节点 创建句柄
13     ros::init(argc,argv,"turtle_spawn");
14     ros::NodeHandle node;
15     //发现服务 创建客户端
16     ros::service::waitForService("/spawn");
17     ros::ServiceClient add_turtle = node.serviceClient<turtlesim::Spawn> ("/spawn");
18     //初始化turtlesim::Spwn 的请求数据
19     turtlesim::Spawn srv;
20     srv.request.x = 2.0;
21     srv.request.y = 2.0;
22     srv.request.name = "turtle2";
23     //请求服务调用
24     ROS_INFO("Call service to spawn turtle [x : %0.6f,y:%0.6f ,name :%s]",srv.request.x,srv.request.y,srv.request.name);
25     add_turtle.call(srv);
26 
27     //显示服务调用结果
28     ROS_INFO("Spawn turtle successfully [name :%s]",srv.response.name.c_str());
29 
30     return 0;
31 }
32 add_executable(turtle_spawn src/turtle_spawn.cpp)
33 target_link_libraries(turtle_spawn ${catkin_LIBRARIES})
 rosrun turtlesim turtlesim_node

 rosrun learning_service turtle_spawn
```

# 9. 服务端Server的编程实现

```cpp
（1）创建一个节点【可以再节点里面注册发布者，注册服务】
（2）创建server实例
（3）循环等待服务请求，进入回调函数
（4）在回调函数中完成服务功能的处理，并反馈应答数据
 1 /**
 2  * 现在要实现的就是一个服务器
 3  * 初始化ros节点
 4  * 创建服务端
 5  * 循环等待服务请求，进入回调函数响应请求
 6  * 在回调函数中完成服务功能的处理并反馈给客户端应答数据
 7  * 
 8  * */
 9 
10 #include<ros/ros.h>
11 #include<geometry_msgs/Twist.h>
12 #include<std_srvs/Trigger.h>
13 ros::Publisher turtle_vel_pub;
14 bool pubCommand = false;
15 bool commandCallback(std_srvs::TriggerRequest & req, std_srvs::TriggerResponse & res){
16 
17     pubCommand =  !pubCommand;
18     //显示请求数据
19     ROS_INFO ("publish turtle velocity command [%s]", pubCommand == true ? "Yes":"No");
20     //设置反馈数据
21     res.success = true;
22     res.message = "Change turtle command state!";
23 
24 }
25 int main(int argc,char ** argv )
26 {
27     //初始化创建句柄
28     ros::init(argc,argv,"turtle_command_server");
29     ros::NodeHandle n;
30     //创建一个服务 注册回调函数
31     ros::ServiceServer command_service = n.advertiseService("/turtle_command",commandCallback);
32     //创建一个发布者 设置循环发布消息
33     turtle_vel_pub = n.advertise<geometry_msgs::Twist>("/turtle1/cmd_vel",10);
34     //循环等待回调函数 因为这是一个服务段，要一直等待响应客户端的请求
35     ROS_INFO("Ready to receive turtle command.");
36     //设置循环频率
37     ros::Rate loop_rate(10);
38     while(ros::ok()){
39         //查看一次回调函数队列
40         ros::spinOnce();
41         //如果标志为true，则发布速度指令
42         if(pubCommand){
43             geometry_msgs::Twist vel_msg;
44             vel_msg.linear.x = 0.5;
45             vel_msg.angular.z = 0.2;
46             turtle_vel_pub.publish(vel_msg);
47         }
48         //按照循环频率延时
49         loop_rate.sleep();
50     }
51 
52     return 0;
53 }
54 add_executable(turtle_command_server src/turtle_command_server.cpp)
55 target_link_libraries(turtle_command_server ${catkin_LIBRARIES})
rosrun turtlesim turtlesim_node
rosrun learning_service turtle_command_server
rosservice call /turtle_command "{}"
```

# 10. 服务数据的定义与使用

```cpp
自己创建一个服务端一个客户端，自己编写服务数据类型
15.1自定义服务数据
(1)创建服务数据文件
Person.srv
string name
uint8 age
unit8 sex
uint8 unknown = 0
unit8 male = 1
uint8 female = 2
---
string result
（2）在Package.xml 里面添加功能包依赖
<build_depend>message_generation</build_depend>
<exec_depend>message_runtime</exec_depend>
（3）在CMakeLists.tst 添加编译选项
find_package( ...... message_generation)
add_service_files(FILES Person.srv)
generate_messages(DEPENDENCIES std_msgs)
catkin_package(
# INCLUDE_DIRS include
# LIBRARIES learning_service
 CATKIN_DEPENDS geometry_msgs roscpp rospy std_msgs turtlesim message_runtime
# DEPENDS system_lib
)
现在就去编译 catkin_make
就会发现这个文件下面 有了Person.h了
/home/tian/catkin_ws/devel/include/learning_service
15.2创建服务端和客户端
问题：在vscode里面，会出现找不到.h的情况这个时候在c_cpp_properties.json 设置
"includePath": [
 "${workspaceFolder}/**",
 "/opt/ros/kinetic/include/*",
 "/home/tian/catkin_ws/devel/include/*"
 ],
15.2.1实现一个服务端
初始化节点
创建服务端，注册回调函数
循环等待回调函数
客户端
创建节点
发现服务 然后创建客户端
设置数据
调用服务
查看返回结果
 1 /**
 2  * 创建一个服务端 执行/show_person 服务，参数类型是learning_service::Person
 3  * 初始化节点
 4  * 创建服务端，注册回调函数 
 5  * 循环等待回调函数
 6  * 
 7  * */
 8 #include<ros/ros.h>
 9 #include "learning_service/Person.h"
10 bool personCallback(learning_service::Person::Request & req,
11     learning_service::Person::Response & res)
12 {
13     ROS_INFO("Person : name : %s age : %d sex : %d",req.name.c_str(),req.age,req.male);
14     //设置反馈数据
15     res.result = "OK";
16 
17 }
18 int main(int argc,char ** argv)
19 {
20     ros::init(argc,argv,"person_server");
21     ros::NodeHandle n;
22     ros::ServiceServer person_service = n.advertiseService("/show_person",personCallback);
23     //循环等待回调函数
24     ROS_INFO("Ready to show person information.");
25     ros::spin();
26     return 0;
27 }
28 /**
29  * 现在是客户端代码
30  * 初始化节点
31  * 发现服务然后创建客户端
32  * 初始化数据 发送请求
33  * 请求响应的结果
34  * 
35  * */
36 #include<ros/ros.h>
37 #include "learning_service/Person.h"
38 int main(int argc,char ** argv)
39 {
40     //初始化
41     ros::init(argc,argv,"person_client");
42     ros::NodeHandle node;
43     //等待服务 创建客户端
44     ros::service::waitForService("/show_person");
45     ros::ServiceClient person_client = node.serviceClient<learning_service::Person>("/show_person");
46     //初始化learning_service::Person
47     learning_service::Person srv;
48     srv.request.name = "Tom";
49     srv.request.age = 20;
50     srv.request.sex = learning_service::Person::Request::male;
51     //请求服务调用
52     ROS_INFO("Call service to show person [name :%s,age : %d,sex:%d]",
53         srv.request.name.c_str(),srv.request.age,srv.request.sex
54     );
55     //调用服务
56     person_client.call(srv);
57     //显示服务调用结果
58     ROS_INFO("show person result : %s ",srv.response.result.c_str());
59 
60 
61     return 0;
62 }
63 add_executable(person_publisher src/person_publisher.cpp)
64 target_link_libraries(person_publisher ${catkin_LIBRARIES})
65 add_dependencies(person_publisher ${PROJECT_NAME}_generate_messages_cpp)
66 
67  
68 add_executable(person_subscriber src/person_subscriber.cpp)
69 target_link_libraries(person_subscriber ${catkin_LIBRARIES})
70 add_dependencies(person_subscriber ${PROJECT_NAME}_generate_messages_cpp)
rosrun learning_service person_server
rosrun learning_service person_clien
```

# 11. 参数的使用与编程方法

```cpp
16.1 ros master 里面有一个parameter server【全局字典】 里面有各种参数以及对应的值
启动roscore
启动 rosrun turtlesim turtlesim_node
(1)rosparam list [列出当前参数]
/background_b
/background_g
/background_r
/rosdistro
/roslaunch/uris/host_tian_nl__39943
/rosversion
/run_id
(2)显示某一个参数
rosparam get /background_b
（3）设置某个参数值
osparam set /background_b 0
为了显示已经改变了参数的值，可以刷新一下显示小乌龟的背景
rosservice call /clear "{}"
（4）保存参数到文件 文件后缀是.yaml
rosparam dump now_param.yaml
（5）从文件读取参数
rosparam load now_param.yaml
rosservice call /clear "{}"
（6） 删除参数
rosparam delete param_key
16.2 参数编程
初始化ros节点
get 函数获取参数
set函数设置参数
 1 /**
 2  * 设置海龟历程中的参数
 3  * */
 4 #include<string>
 5 #include<ros/ros.h>
 6 #include<std_srvs/Empty.h>
 7 int main(int argc,char ** argv)
 8 {
 9     int red,green , blue;
10     //ros 初始化和创建句柄
11     ros::init(argc,argv,"parameter_config");
12     ros::NodeHandle node;
13     //读取背景颜色
14     ros::param::get("/background_r",red);
15     ros::param::get("/background_g",green);
16     ros::param::get("/background_b",blue);
17     //输出读取的背景颜色
18     ROS_INFO("Get Background Color [%d ,%d,%d]",red,green,blue);
19     //设置背景颜色参数
20     ros::param::set("/background_r",255);   
21     ros::param::set("/background_g",255);   
22     ros::param::set("/background_b",255);   
23     ROS_INFO("Get Background Color [255,255,255]");
24     //因为已经被修改了 再读取背景颜色
25     ros::param::get("/background_r",red);
26     ros::param::get("/background_g",green);
27     ros::param::get("/background_b",blue);
28     ROS_INFO("Re-get Background Color [%d ,%d,%d]",red,green,blue);
29     //调用服务，刷新背景颜色　等待服务　服务找到之后就设置参数　调用服务
30     ros::service::waitForService("/clear");
31     ros::ServiceClient clear_background = node.serviceClient<std_srvs::Empty>("/clear");
32     std_srvs::Empty srv;
33     clear_background.call(srv);
34     sleep(1);
35     return 0;
36 }
37 add_executable(parameter_config src/parameter_config.cpp)
38 target_link_libraries(parameter_config ${catkin_LIBRARIES})
启动roscore
启动 rosrun turtlesim turtlesim_node
启动　rosrun learning_parameter parameter_config
```

# 12. ROS中的坐标系管理系统

```cpp
17.1坐标变化　就是　P2 = R(P1)+t
17.2 TF 功能包能干什么
５秒钟以前，机器人头部的坐标系相对于全局坐标系的关系是怎么样的
机器人夹取的物体相对于机器人中心坐标系的位置在哪里
机器人中心坐标相对于全局坐标的位置在哪里
TF坐标变换是如何实现的
广播ＴＦ变换
监听　ＴＦ变换
17.3　演示
sudo apt-get install ros-melodic-turtle-tf
roslaunch turtle_tf turtle_tf_demo.launch [乌龟２会追着乌龟１　，不要求他们的头指向同一方向，但是身子要重叠]
rosrun turtlesim turtle_teleop_key
rosrun tf view_frames【会在当前目录下生成　frames.pdf】
rosrun tf  tf_echo turtle1 turtle2[可以看到两个小乌龟的位姿]
rosrun rviz rviz -d `rospack find turtle_tf` /rviz/turtle_rviz.rviz　【在Fixed Frame 变成world  ｜｜　Add-> TF】
```

# 13. tf坐标系广播与监听的编程实现

```cpp
roscore
rosrun turtlesim turtlesim_node
rosrun learning_tf turtle_tf_broadcaster __name:=turtle1_tf_broadcaster /turtle1【turtle1_tf_broadcaster　会替换掉代码里面设置的节点名字　arg[1] = trutle1】
rosrun learning_tf turtle_tf_broadcaster __name:=turtle２_tf_broadcaster /turtle２
rosrun learning_tf turtle_tf_listener
rosrun turtlesim turtle_teleop_key rostopic list
rosnode list

  1 roscore
  2 
  3 rosrun turtlesim turtlesim_node
  4 
  5 rosrun learning_tf turtle_tf_broadcaster __name:=turtle1_tf_broadcaster /turtle1【turtle1_tf_broadcaster　会替换掉代码里面设置的节点名字　arg[1] = trutle1】
  6 rosrun learning_tf turtle_tf_broadcaster __name:=turtle２_tf_broadcaster /turtle２
  7 
  8 rosrun learning_tf turtle_tf_listener
  9 
 10 rosrun turtlesim turtle_teleop_key rostopic list
 11 
 12 rosnode list
 13 
 14 
 15 /**
 16  * 现在是实现一个tf广播器
 17  * 创建坐标变换值
 18  * 发布坐标变换
 19  * 
 20  * */
 21 #include<ros/ros.h>
 22 #include<tf/transform_broadcaster.h>
 23 #include<turtlesim/Pose.h>
 24 std::string turtle_name;
 25 void poseCallback(const turtlesim::PoseConstPtr & msg){
 26     //创建ｔｆ的广播器
 27     static tf::TransformBroadcaster br;
 28     //初始化ｔｆ的数据
 29     tf::Transform transform;
 30     //平移
 31     transform.setOrigin(tf::Vector3(msg->x,msg->y,0));
 32     //四元数
 33     tf::Quaternion q;
 34     q.setRPY(0,0,msg->theta);
 35     transform.setRotation(q);
 36   
 37     //广播ｗｏｒｌｄ与海龟坐标系之间的ｔｆ数据
 38     br.sendTransform(
 39         tf::StampedTransform(
 40             transform,
 41             ros::Time::now(),
 42             "world",
 43             turtle_name
 44         )
 45     );
 46 
 47 }
 48 int main(int argc,char ** argv){
 49     ros::init(argc,argv,"my_tf_broadcaster");
 50     if(argc!=2){
 51         ROS_ERROR("need turtle name as argument");
 52         return -1;
 53     }
 54     turtle_name = argv[1];
 55     //订阅海龟的位子话题
 56     ros::NodeHandle node;
 57     ros::Subscriber sub = node.subscribe(turtle_name+"/pose",10,&poseCallback);
 58     //循环等待回调函数
 59     ros::spin();
 60     return 0;
 61 }
 62 /**
 63  * 定义ｔｆ监听器
 64  * 查找坐标变换
 65  * */
 66 #include<ros/ros.h>
 67 #include<tf/transform_listener.h>
 68 #include<geometry_msgs/Twist.h>
 69 #include<turtlesim/Spawn.h>
 70 int main(int argc,char ** argv){
 71     //初始化ｒｏｓ节点和句柄　请求产生乌龟
 72     ros::init(argc,argv,"my_tf_listener");
 73     ros::NodeHandle node;
 74     ros::service::waitForService("/spawn");
 75     ros::ServiceClient add_turtle = node.serviceClient<turtlesim::Spawn>("/spawn");
 76     turtlesim::Spawn srv;
 77     add_turtle.call(srv);
 78     //创建发布ｔｕｒｔｌｅ速度控制指令的发布者
 79     ros::Publisher turtle_vel = node.advertise<geometry_msgs::Twist>("/turtle2/cmd_vel",10);
 80     //创建ｔｆ的监听器
 81     tf::TransformListener listener;
 82     ros::Rate rate(10.0);
 83     while (ros::ok())
 84     {
 85         //获取两个海龟坐标系之间的ｔｆ数据
 86         tf::StampedTransform transform;
 87         try
 88         {
 89             listener.waitForTransform("/turtle2","/turtle1",ros::Time(0),ros::Duration(3.0));
 90             listener.lookupTransform("/turtle2","/turtle1",ros::Time(0),transform);
 91 
 92         }
 93         catch(tf::TransformException &ex)
 94         {
 95             ROS_ERROR("%s",ex.what());
 96             // ros::Duration(1.0)这里代表时长
 97             ros::Duration(1.0).sleep();
 98             continue;
 99         }
100         //根据两个乌龟之间的位置关系，发布ｔｕｒｔｌｅ的速度控制指令
101         geometry_msgs::Twist vel_msg;
102         //arctan(y/x);
103         vel_msg.angular.z = 4.0 * atan2(
104             transform.getOrigin().y(),
105             transform.getOrigin().x()
106         );
107         vel_msg.linear.x = 0.5* sqrt(
108             pow(transform.getOrigin().x(),2) +
109             pow(transform.getOrigin().y(),2)
110         );
111 
112         turtle_vel.publish(vel_msg);
113     }
114   
115     return 0;
116 }
117 add_executable(turtle_tf_broadcaster src/turtle_tf_broadcaster.cpp)
118 target_link_libraries(turtle_tf_broadcaster ${catkin_LIBRARIES})
119 
120 add_executable(turtle_tf_listener src/turtle_tf_listener.cpp)
121 target_link_libraries(turtle_tf_listener ${catkin_LIBRARIES})
 ros19.launch启动文件[xml]的使用方法
```

# 14. launch启动文件[xml]的使用方法

```cpp
19.1 launch 文件语法
 1 <launch>
 2     <node pkg = "learning_topic" type = "person_subscriber" name="talker" output="screen"/>
 3     <node pkg = "learning_topic" type = "person_publisher" name="listener" output="screen"/>
 4   
 5 </launch>
 6 <!-- 
 7     node:启动节点
 8     pkg:节点所在的功能包名称
 9     type:　节点的可执行文件名称
10     name:节点运行时的名称　【相当于我虽然写了一个cpp　里面我也确实初始化了一个节点
11     并且这个节点也有名字，但是现在我用name来代替我一开始写的节点的名字
12     这样我多几个ｎａｍｅ就多了几个节点但是只写了一份代码
13     】
14     output:　打印在哪里
15     创建一个功能包　不添加任何依赖　learning_launch 在下面添加launch文件夹　把.xml　launch文件 放入到文件下面
16     roslaunch learning_launch simple.launch 
17     rostopic list
18 -->
 

 1 <launch>
 2     <param name = "/turtle_number" value = "2"/>
 3     <node pkg="turtlesim" type="turtlesim_node" name="turtlesim_node">
 4         <param name = "turtle_name1" value = "Tom"/>
 5         <param name = "turtle_name2" value = "Jerry"/>
 6         <rosparam 
 7             file = "$(find learning_launch)/config/param.yaml"
 8             command = "load"
 9         />
10     </node>
11 
12     <node 
13         pkg="turtlesim" 
14         type="turtle_teleop_key"
15         name = "turtle_teleop_key" 
16         output = "screen"
17   
18     />
19 </launch>
20 <!-- 
21     设置ｒｏｓ系统中运行的参数，存储在参数服务器中
22   
23     param:
24         name:
25         value:
26     加载参数文件中的多个参数
27     rosparam
28         file
29         command
30         ns="params"
31     roslaunch learning_launch turtlesim_parameter_config.launch
32     rosparam list
33  -->
 

 1 <launch>
 2     <node 
 3         pkg="turtlesim" 
 4         type="turtlesim_node"
 5         name = "sim" 
 6     />
 7     <node 
 8         pkg="turtlesim" 
 9         type="turtle_teleop_key"
10         name = "teleop" 
11         output = "screen"
12     />
13     <node 
14         pkg="learning_tf" 
15         type="turtle_tf_broadcaster"
16         name = "turtle1_tf_broadcaster" 
17         args="/turtle1"
18     />
19     <node 
20         pkg="learning_tf" 
21         type="turtle_tf_broadcaster"
22         name = "turtle2_tf_broadcaster" 
23         args="/turtle2"
24     />
25     <node 
26         pkg="learning_tf" 
27         type="turtle_tf_listener"
28         name = "listener" 
29  
30     /> 
31   
32   
33 </launch>
34 <!-- 
35     roslaunch learning_launch start_tf_demo_c++.launch
36  -->
 

 1 <launch>
 2   
 3     <include file="$(find learning_launch)/launch/simple.launch"/>
 4     <node pkg="turtlesim" type = "turtlesim_node" name="turtlesim_node">
 5         <remap from="/turtle1/cmd_vel" to="/cmd_vel"/>
 6     </node>
 7 </launch>
 8 <!-- 
 9     remap 重映射　ｒｏｓ计算图资源的命名
10     roslaunch learning_launch turtlesim_remap.launch 
11     rostopic list
12     rostopic pub /cmd_vel geometry_msgs/Twist "linear:
13     x: 5.0
14     y: 0.0
15     z: 0.0
16     angular:
17     x: 0.0
18     y: 0.0
19     z: 2.0" 
20  -->
21
```

# 15. 常用可视化工具的使用

```cpp
20.1rqt_console　【　日志输出工具　对日志信息进行筛选　】
rqt_graph [计算图可视化工具]
rqt_plot[　数据绘图工具　把信息绘制成图　在topic那里选择订阅的话题]
rqt_image_view [图形渲染工具　链接图像]
rqt 集成了ros qt的所有的工具，可以把很多的工具就行堆叠
20.2
rosrun rviz rviz
３Ｄ视图区　工具栏　显示项列表　视觉设置区　时间显示区
如果先看什么就点击ａｄｄ　但是都需要订阅对应的话题
gazebo
roslaunch gazebo_ros willowgarage_world.launch
```

# 16. 定时器

## 16.1 定时器的说明

```cpp
1.基本说明
roscpp定时器允许用户安排一个回调发生周期性。
Timers能让你以一定的频率来执行
他们是比ros::Rate更加灵活和有用的形式
注意：定时器不是实时的线程/内核替换，也不能保证它们的准确度，因为系统负载/功能会有很大的变化1。
2. 使用
创建定时器象创建订阅一样,通过ros::NodeHandle::createTimer()方法创建：
ros::Timer timer = n.createTimer(ros::Duration(0.1), timerCallback);
ros::Timer = ros::NodeHandle::createTimer(ros::Duration period, <callback>, bool oneshot = false);
period ，这是调用定时器回调函数时间间隔。例如，ros::Duration(0.1)，即每十分之一秒执行一次，回调函数，可以是函数，类方法，函数对象。
oneshot ，表示是否只执行一次，如果已经执行过，还可以通过stop()、setPeriod(ros::Duration)和start()来规划再执行一次。
3. 回调函数
回调函数：
void timerCallback(const ros::TimerEvent& e);
ros::TimerEvent结构体作为参数传入，它提供时间的相关信息，对于调试和配置非常有用
ros::TimerEvent结构体说明：
ros::Time last_expected 上次回调期望发生的时间
ros::Time last_real 上次回调实际发生的时间
ros::Time current_expected 本次回调期待发生的时间
ros::Time current_real 本次回调实际发生的时间
ros::WallTime profile.last_duration 上次回调的时间间隔（结束时间-开始时间），是wall-clock时间。

```

## 16.2遇到的例子

```cpp
1. 首先定义变量和回调函数
ros::Timer node_status_timer_;
/// node status
void sendNodeStatus(const ros::TimerEvent &timer_event = {});

void SceneMappingWrapper::sendNodeStatus(const ros::TimerEvent &timer_event) {
    static ros::Publisher node_status_publisher_ =
        nh_.advertise<ad_monitor_msgs::NodeStateInfo>("node_state", 10);
    ad_monitor_msgs::NodeStateInfo msg;
    msg.name = "scene_mapping";
    msg.status = cur_node_status_.load();
    msg.is_nodelet = false;
    msg.pid = getpid();
    msg.wall_time_ns = ros::WallTime::now().toNSec();
    node_status_publisher_.publish(msg);
}
2. 使用
node_status_timer_ = nh_.createTimer(
        ros::Rate(1), &SceneMappingWrapper::sendNodeStatus, this);

```

## 16.3 ros::Rate的形式

```cpp
1. ros::Rate方式来发送
ros::Rate loop_rate(10);
/*
解释：
		指定发布消息的频率，这里指10Hz，也即每秒10次
		通过 Rate::sleep()来处理睡眠的时间来控制对应的发布频率。
*/

loop_rate.sleep();
/*
解释：
		根据之前ros::Rate loop_rate(10)的定义来控制发布话题的频率。定义10即为每秒10次
*/
 
```

# 17. 打印发送的内容

```cpp
rostopic echo /local_map[topic的名字]
```

# 18. 查看消息

```cpp
rosmsg show geometry_msgs/Vector3
```

# 19. 录制消息

```cpp
rosbag record -O localmap_rs_li[指定的输出的名字] /local_map/road_structure /local_map/localization_info
rosbag info localmap_rs_li.bag [查看消息的配置]
```