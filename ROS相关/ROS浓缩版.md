## _前言_

        做机器人的第一步是学习ROS。下面我将ROS的知识点以及在学习过程中遇到的问题及解决方法进行汇总，希望能帮助更多的人学习ROS。其中我会对重点与所遇到的问题进行详细解析，而相对明确与易操作的部分只会做一个概括或者跳过，若读者想了解详细内容，可以参考以下链接进行学习：[Introduction · Autolabor-ROS机器人入门课程《ROS理论与实践》零基础教程](http://www.autolabor.com.cn/book/ROSTutorials/ "Introduction · Autolabor-ROS机器人入门课程《ROS理论与实践》零基础教程")（我主要是参考autolabor的教程学习的，当然也可以直接参考官网教程：[cn/ROS/Tutorials - ROS Wiki](http://wiki.ros.org/cn/ROS/Tutorials "cn/ROS/Tutorials - ROS Wiki")）

## 一. ROS安装

        ROS[操作系统安装](https://so.csdn.net/so/search?q=%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E5%AE%89%E8%A3%85&spm=1001.2101.3001.7020)总体步骤:

                a.配置ubuntu软件和更新

                b.设置安装源 sources.list

                c.设置密钥

                d.安装

                e.配置环境变量

        这里教程中还是比较明确的，跟着教程一步步走就行，这里就不做赘述了。

## 二.创建ROS软件包

###         1. 创建catkin工作空间

                先在vscode “workspace\_folder” 文件下新建终端，保证路径正确，然后输入以下代码：

```typescript
mkdir -p ~/catkin_ws/src

cd ~/catkin_ws/

catkin_make
```

                接下来首先source一下新生成的setup.\*sh文件：

```cobol
source ./devel/setup.bash
```

   注意：每次创建了新的工作空间或进入工作空间都需要source一下

                要保证工作区被安装脚本正确覆盖，需确定ROS\_PACKAGE\_PATH环境变量包含你当前的工作空间目录：

```php
echo $ROS_PACKAGE_PATH
```

                显示/home/<username>/catkin\_ws/src:/opt/ros/<distro>/share即成功

###         2.创建catkin软件包

                首先切换到刚才创建的空白catkin工作空间中的源文件空间目录：

```typescript
cd ~/catkin_ws/src
```

                现在使用catkin\_create\_pkg命令创建一个名为beginner\_tutorials的新软件包，这个软件包依赖于std\_msgs、roscpp和rospy：

```undefined
catkin_create_pkg beginner_tutorials std_msgs rospy roscpp
```

###         3. 构建一个catkin工作区并生效配置文件

                在catkin工作区中构建软件包：

```bash
cd ~/catkin_ws

catkin_make
```

                要将这个工作空间添加到ROS环境中，你需要source一下生成的配置文件：

```cobol
. ~/catkin_ws/devel/setup.bash
```

###         4.自定义软件包

                根据要求自定义即可。

###         **5.进入 src 创建 ros 包并添加依赖**

```scss
cd src

catkin_create_pkg 自定义ROS包名 roscpp rospy std_msgs
```

## 三.ROS快速实现（C++）

        C++源码实现中出现的问题：

                1. #include "ros/ros.h"报错，报错显示无法打开源文件“ros/ros.h”

                  A: 先不用管，可以正常运行。

                2. 进入工作空间目录并编译时，catkin\_make报错：

```csharp
Command 'catkin_make' not found, but can be installed with:

sudo apt install catkin
```

                以上报错说明catkin未安装成功，且sudo apt install catkin或sudo apt-get python3-catkin-pkg等语句都没有用，最后输入以下语句可初步解决问题：

```csharp
sudo apt-get install ros-noetic-catkin
```

                此时再输入catkin\_make，若产生新的报错如下：

```csharp
sudo apt-get install ros-noetic-catkin
```

                可能是ros里的一些东西在中间被删除了，返回到前面的代码再跑一下：

```bash
sudo apt install ros-noetic-desktop-full

echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc

source ~/.bashrc
```

                然后再运行catkin\_make不会出现原来的报错，但有可能出现以下报错：

```csharp
Invoking "make cmake_check_build_system" failed
```

到这里要检查一下工作空间中两个CMakeList.txt文件，以防两个文件内容相同导致冲突从而跑不起来。src源码文件下的cmake文件是自动生成的，不要改！功能包下的cmake文件要手改，如下：

```cobol
add_executable(步骤3的源文件名

src/步骤3的源文件名.cpp

)

target_link_libraries(步骤3的源文件名

${catkin_LIBRARIES}

)
```

                以上全部改完后再输入catkin\_make基本就可以跑通了。

                3.执行程序的两步应是在不同终端里执行的。

                  先在第一个终端里执行：

```undefined
roscore
```

                   再新建终端，在第二个终端里执行：

```cobol
cd 工作空间

source ./devel/setup.bash（下面4.完成后以后都可省略这句）

rosrun 包名 C++节点
```

                  即可成功输出HelloWorld!

                4. 简化以后的工作量，解决每次创建了新的工作空间或进入工作空间都需要source一下的问题。

                   可以输入以下代码，将source ~/工作空间/devel/setup.bash添加进.bashrc文件：

```bash
echo "source ~/工作空间/devel/setup.bash" >> ~/.bashrc
```

         注意：如果工作空间就在主目录的一级子目录中，那么“/工作空间”这部分只需要写工作空间名称即可；而如果工作空间非主目录的一级子目录，那么“/工作空间”应写工作空间与工作空间的上级目录（不包括主目录，“～”就代表了主目录）。若填错路径，该命令就是无效命令，会在终端中产生类似如下错误提示：

```cobol
bash: /home/zhenghao/rm2023_auto_sentry_ws/devel/setup.bash: 没有那个文件或目录

bash: /home/zhenghao/开源/RM2023华南农业大学Taurus机器人战队哨兵定位导航算法开源/rm2023_auto_sentry_ws/devel/setup.bash: 没有那个文件或目录
```

## 四.ROS集成开发环境搭建

###         1.安装Terminator终端

                1）安装：sudo apt install terminator

                2）在应用程序里搜索Terminator，然后添加到收藏夹

                3）Terminator常用快捷键

                        教程中有做详细列举，这里不再赘述，今后有用到时再查阅即可。

                        a. 同一标签内的操作

                        b. 各个标签之间的操作

###         2.使用launch文件，一次性启动多个ROS节点

                a. 在功能包下添加launch文件夹。

        功能包是放置ROS源码的最小单元，所有源码都必须放在功能包中。在这里是beginner\_tutorials。（细节在ROS架构中）

                b. 添加launch文件

                c. 编辑launch文件内容如下：

```cobol
<launch>

<node pkg="helloworld" type="demo_hello" name="hello" output="screen" />

<node pkg="turtlesim" type="turtlesim_node" name="t1"/>

<node pkg="turtlesim" type="turtle_teleop_key" name="key1" />

</launch>
```

                   其中，node ---> 包含的某个节点

                              pkg -----> 功能包

                              type ----> 被运行的节点文件

                              name --> 为节点命名

                              output-> 设置日志的输出目标

                   d. 运行launch文件，即可一次性启动多个节点

```scss
roslaunch 功能包名 launch文件名(xxx.launch)
```

## 五.ROS架构

###         1. ROS文件系统导览

                ROS系统架构大致为：

WorkSpace --- 自定义的工作空间

        |--- build:编译空间，用于存放CMake和catkin的缓存信息、配置信息和其他中间文件。

        |--- devel:开发空间，用于存放编译后生成的目标文件，包括头文件、动态&静态链接库、可执行文件等。

        |--- src: 源码

                  |-- package：功能包(ROS基本单元)包含多个节点、库与配置文件，包名所有字母小写，只能由字母、数字与下划线组成

                             |-- CMakeLists.txt 配置编译规则，比如源文件、依赖项、目标文件

                    |-- package.xml 包信息，比如:包名、版本、作者、依赖项...(以前版本是 manifest.xml)

                             |-- scripts 存储python文件

                             |-- src 存储C++源文件

                             |-- include 头文件

                             |-- msg 消息通信格式文件

                             |-- srv 服务通信格式文件

                             |-- action 动作格式文件

                             |-- launch 可一次性运行多个节点

                             |-- config 配置信息

                   |-- CMakeLists.txt: 编译的基本配置

###         2. ROS文件系统相关命令

####                 1) 查

#####                         a. 使用rospack

                        用find参数选项返回软件包所在路径：

                        用法：rospack find \[package\_name\]

                        例如：输入 rospack find roscpp

                                   输出/opt/ros/<distro>/share/roscpp

#####                         b. 使用roscd

                        a) 用roscd直接切换目录至某个软件或软件包中

                            用法： roscd \[locationname\[/subdir\]\]

                            例如：切换至roscpp包  roscd roscpp（只能定位，不能输出）

                        b) 用pwd输出工作目录

                            用法： roscd roscpp（切换定位）

                                        pwd（输出）

                            输出：YOUR\_INSTALL\_PATH/share/roscpp

                        c) 查看ROS\_PACKAGE\_PATH中包含的路径

                            用法： echo $ROS\_PACKAGE\_PATH

                            输出：/opt/ros/noetic/share

                        d) 切换至软件包或软件包集的子目录

                            用法如：roscd roscpp/cmake （切换定位）

                                           pwd（输出）

                            输出：YOUR\_INSTALL\_PATH/share/roscpp/cmake

                        e) 用log进入存储ROS日志文件的目录

                            只有在执行过ROS文件时可用，否则会报错。

                            用法：roscd log

#####                         c. 使用rosls

                            用rosls命令显示指定工作目录下的内容：

                            用法：rosls \[locationname\[/subdir\]\]

                            例如：输入rosls roscpp\_tutorials

                                       输出cmake launch package.xml  srv

#####                         d. Tab补全

                                a) roscd roscpp\_tut<<<按TAB键>>>

                                   自动补充：roscd roscpp\_tutorials/

                                b) roscd tur<<<按TAB键>>>

                                   自动补充： roscd turtle

在这种情况下有许多软件包都以turtle开头。当再次按**TAB**键后会列出所有以turtle开头的ROS软件包：

                                                turtle\_actionlib/          turtlesim/         turtle\_tf/

                                c) roscd turtles<<<按TAB键>>>

                                    自动补充：roscd turtlesim/

                                d）查看当前安装的所有软件包的列表：

                                                rosls <<<双击TAB键>>>

####                 2）增

                        创建新的ROS功能包：

```undefined
catkin_create_pkg 自定义包名 依赖包
```

                        安装ROS功能包：

```undefined
sudo apt install xxx
```

####                 3）删

                        删除某个功能包：

```undefined
sudo apt purge xxx
```

####                 4）改（用得不多，不做赘述）

####                 5）执行（按照以下顺序输入命令行）

                        a. roscore 或 roscore -p xxxx

                        b. rosrun

                        运行指定的ROS节点：

```undefined
rosrun 包名 可执行文件名
```

                        c. roslaunch

                        执行某个包下的 launch 文件：

```undefined
roslaunch 包名 launch文件名
```

###         3.ROS计算图

####                 1）计算图安装

```undefined
sudo apt install ros-noetic-rqt

sudo apt install ros-noetic-rqt-common-plugins
```

## 六. ROS通信机制

###         1. 话题通信

####                 1)  话题通信C++实现

                需求： 编写发布订阅实现，要求发布方以10HZ(每秒10次)的频率发布文本消息，订阅方订阅消息并将消息内容打印输出。

                分析： 在模型实现中，ROS master 不需要实现，而连接的建立也已经被封装了，需要关注的关键点有三个，即发布方/接收方/数据。

                流程：     a.编写发布方实现

                                b.编写订阅方实现；

                                c.编辑配置文件；

                                d.编译并执行。

                问题与注意：

                a. 不要把发布者（publisher）和订阅者（subscriber）的代码放在一个cpp文件中！！两个节点的交互应在两个独立的源文件之间产生（pub.cpp / sub.cpp）。

                b. 同时需要重新配置CMakeList.txt，将两个节点的源文件分别声明为可执行。

                c. 最后启动roscore时，可能会有如下报错：

```cobol
RLException: roscore cannot run as another roscore/master is already running.

Please kill other roscore/master processes before relaunching.

The ROS_MASTER_URI is http://zhenghao-KLVF-XX:11311/

The traceback for the exception was written to the log file
```

                说明已经有ros master，不需要再启动roscore，否则两者冲突，必须结束一个。

                d. 启动发布者与订阅者节点最好用launch文件同时启动，但写launch文件时output要写成screen或log形式(一般写output=“screen”)，若不符合规则，则会有如下报错：

```cobol
RLException: Invalid <node> tag: output must be one of 'log', 'screen'.

Node xml is <node pkg="beginner_tutorials" type="pub" name="publish" output="pub_screen"/>

The traceback for the exception was written to the log file
```

                e.订阅时，第一条数据丢失。

                原因: 发送第一条数据时， publisher 还未在 roscore 注册完毕。

                解决: 注册后，加入休眠 ros::Duration(3.0).sleep(); 延迟第一条数据的发送。

####                 2）自定义msg

                        a. 流程:

                                a）按照固定格式创建 msg 文件

                                b）编辑配置文件

                                c）编译生成可以被 Python 或 C++ 调用的中间文件

                        b. 定义msg文件

                        功能包下新建 msg 目录，添加文件 Person.msg，输入以下内容：

```go
string name

uint16 age

float64 height
```

                        c. 编辑配置文件

                                a）package.xml中添加编译依赖与执行依赖

                                b）CMakeLists.txt编辑msg相关配置

                        d. 编译

####                 3）自定义msg的调用

                注意：

                        a. 为了方便代码提示以及避免误抛异常，需要先配置 vscode，将前面生成的 head 文件路径配置进 c\_cpp\_properties.json 的 includepath属性:

```cobol
"includePath": [

"/opt/ros/noetic/include/**",

"/usr/include/**",

//这里配置head文件的路径

"/xxx/yyy工作空间/devel/include/**"

],
```

                        b. 需要重新配置CMakeLists.txt:

                        需要添加 add\_dependencies 用以设置所依赖的消息相关的中间文件。

```scss
add_dependencies(person_talker ${PROJECT_NAME}_generate_messages_cpp)

add_dependencies(person_listener ${PROJECT_NAME}_generate_messages_cpp)
```

                        c. 写头文件调用msg时注意头文件格式：

```cpp
#include “功能包名/msg文件名.h”
```

                            例如：#include “beginner\_tutorials/Person.h”

###         2.服务通信

####                 1）自定义srv

#####                         a. 需求

服务通信中，客户端提交两个整数至服务端，服务端求和并响应结果到客户端，请创建服务器与客户端通信的数据载体。

#####                         b. 流程

srv 文件内的可用数据类型与 msg 文件一致，且定义 srv 实现流程与自定义 msg 实现流程类似:

                                a）按照固定格式创建srv文件

                                b）编辑配置文件

                                c）编译生成中间文件

#####                         c. 定义srv文件

                        服务通信中，数据分成两部分，请求与响应，在 srv 文件中请求和响应使用---分割，具体实现如下:

                        功能包下新建 srv 目录，添加 xxx.srv 文件，内容:

```cobol
# 客户端请求时发送的两个数字

int32 num1

int32 num2

---

# 服务器响应发送的数据

int32 sum
```

#####                         d. 编辑配置文件

                        只需在CMakeLists.txt中加一个

```scss
add_service_files(

FILES

AddInts.srv

)
```

                        其他配msg文件时都配过，无需重复。

#####                         e. 编译

```undefined
catkin_make
```

####                 2）自定义srv的调用

                        注意：

                        a. 同样，写头文件调用srv时注意头文件格式：

```cpp
#include “功能包名/srv文件名.h”
```

                        例如：#include “beginner\_tutorials/AddInts.h”

                        b. 程序的执行与msg调用不同：

                        流程:

                                需要先启动服务:  

```undefined
rosrun 包名 服务
```

                                然后新建终端，再调用客户端:

```cobol
rosrun 包名 客户端 参数1 参数2
```

                                结果:会根据提交的数据响应相加后的结果。

                        c. 不要在launch文件里补充！这里只能手动启动节点以及在客户端给出数字。用launch服务端会默认客户端给的数字是0 0，直接给出结果0。

                        d. 这里节点的启动是有顺序的，要先启动服务端，再调用客户端。如果先启动客户端，那么会导致运行失败。

                        优化：

                        在客户端发送请求前添加:  

```scss
client.waitForExistence()
```

                        或者：

```cpp
ros::service::waitForService("AddInts");
```

                        这是一个阻塞式函数，保证只有服务启动成功后才会继续执行。

###         3.参数服务器

                参数操作：（需求：实现参数服务器参数的增删改查操作。）

                        1）参数服务器新增/修改参数（增/改）

                        2）参数服务器获取参数（查）

                        3）参数服务器删除参数（删）

                注意：

                        1）以上操作主要基于对字典（set）的运用。

                        roscpp中提供了两套API，均可实现：

                                ros::NodeHandle nh; nh.setParam("键",值);

                                ros::param::set("键",值);

                        2）ros中的输出函数ROS\_INFO();更像是基于C语言的语法，相当于printf();

###         4. 常用命令

####                 1）rosnode

                rosnode:获取节点信息

                        rosnode ping    测试到节点的连接状态

                        rosnode list    列出活动节点

                        rosnode info    打印节点信息

                        rosnode machine    列出指定设备上节点

                        rosnode kill    杀死某个节点

                        rosnode cleanup    清除不可连接的节点

####                 2）rostopic

                rostopic:显示有关ROS 主题的调试信息，包括发布者，订阅者，发布频率和ROS消息

                        rostopic bw :    显示主题使用(消息发布)的带宽

                        rostopic delay : 显示带有 header 的主题延迟(列出消息头信息)

                        rostopic echo :  打印消息到屏幕(获取指定话题当前发布的消息)

                        rostopic find 消息类型 :    根据类型查找主题

                        rostopic hz :    显示主题(消息)的发布频率

                        rostopic info :  显示主题相关信息(包括消息类型/发布者信息/订阅者信息)

                        rostopic list :  显示所有活动状态下的主题名称(直接调用即可，控制台将打印当前运行状态下的主题名称)

                        rostopic list -v : 获取话题详情(比如列出：发布者和订阅者个数...)

                        rostopic pub /主题名称 消息类型 消息内容:   将数据发布到主题

                                                (例如：rostopic pub /chatter std\_msgs gagaxixi)

                        rostopic type :   打印主题类型(列出话题的消息类型)

####                 3）rosmsg

                rosmsg:显示有关 ROS消息类型的信息

                        rosmsg show    显示消息描述

                        rosmsg info    显示消息信息

                        rosmsg list    列出所有消息

                        rosmsg md5    显示 md5 加密后的消息

                        rosmsg package    显示某个功能包下的所有消息

                        rosmsg packages    列出包含消息的功能包

####                 4）rosservice

                 rosservice:包含用于列出和查询ROS[Services](http://wiki.ros.org/Services "Services")的rosservice命令行工具

                        rosservice args  打印服务参数

                        rosservice call    使用提供的参数调用服务

                        rosservice find    按照服务类型查找服务

                        rosservice info    打印有关服务的信息

                        rosservice list    列出所有活动的服务

                        rosservice type    打印服务类型

                        rosservice uri     打印服务的 ROSRPC uri

####                 5）rossrv

                rossrv:显示有关ros服务类型，与rosmsg语法类似

                        rossrv show    显示服务消息详情

                        rossrv info    显示服务消息相关信息

                        rossrv list    列出所有服务信息

                        rossrv md5    显示 md5 加密后的服务消息

                        rossrv package    显示某个包下所有服务消息

                        rossrv packages    显示包含服务消息的所有包

####                  6）rosparam

                rosparam:用于使用YAML编码文件在参数服务器上获取和设置ROS参数

                        rosparam set    设置参数

                        rosparam get    获取参数

                        rosparam load    从外部文件加载参数

                        rosparam dump    将参数写出到外部文件

                        rosparam delete    删除参数

                        rosparam list    列出所有参数

##  七.ROS通信机制进阶

###           1. 常用API

                Q：什么是API？

                A：API，Application Programming Interface，应用程序接口，是一些预先定义的函数，相当于一个被封装好的类，编程人员无需访问源码或理解内部工作机制的细节就能对其进行运用。如上面参数服务器中用到的ros::NodeHandle和ros::param。

                下面我只对常用API展开列举，其他不赘述，详情参考教程。

                        a. 初始化函数

                        b. 话题与服务相关对象(一般由NodeHandle创建)

                                a）发布对象(对象获取/消息发布函数)

                                b）订阅对象(对象获取)

                                c）服务对象(对象获取)

                                d）客户端对象(对象获取/请求发送函数/等待服务函数)

                        c. 回旋函数(ros::spin()和ros::spinOnce()，可用于处理回调函数)

                            spin()与spinOnce()的不同：

                            spin():循环处理回调。即会一直去缓冲区查看是否有回调函数，有则处理，没有则继续查看并等待。因此会一直卡在spin()，spin()后的语句不会被执行到。

                            spinOnce():只处理一次回调。只会往前查看一次回调，有则处理且只处理一次，没有则继续往下执行。因此spinOnce()后面的语句是能被执行到的。

                        d. 时间

                                a）时刻(获取时刻/设置指定时刻)

                                b）持续时间(设置时间间隔)

                                c）持续时间与时刻运算(时刻与时刻(time)之间不可运算，只有时间间隔之间(duration)可以运算)

                                d）设置运行频率

                                e）定时器(包括定时器回调函数)

                        e. 其他函数

                                a）节点状态判断

                                b）节点关闭函数

                                c）日志函数

###         2. ROS中的头文件与源文件

####                 1）自定义头文件调用

                        需求:设计头文件，可执行文件本身作为源文件。

                        流程:

                                a. 编写头文件；

                                b. 编写可执行文件(同时也是源文件)；

                                c. 编辑配置文件并执行。

                        注意：

                        a. 要在功能包下的 include/功能包名 目录下新建头文件。文件名需命名为：xxx.h

                        b. 需要配置.vscode 下 c\_cpp\_properties.json 的 includepath属性:

```csharp
"/home/用户/工作空间/src/功能包/include/**"
```

                        c. 可执行文件(源文件)中的头文件需要添加：

```cpp
#include “功能包名/自定义头文件名.h”
```

                        例如：#include “beginner\_tutorials/hello.h”

                        d. 执行时可能会出现报错：

```cobol
CMake Error at beginner_tutorials/CMakeLists.txt:144 (add_dependencies): Cannot add target-level dependencies to non-existent target "self_def_header".

Invoking "make cmake_check_build_system" failed
```

错误原因： add\_dependencies()的位置错了，注意[CMakeList](https://so.csdn.net/so/search?q=CMakeList&spm=1001.2101.3001.7020 "CMakeList").txt中add\_exacutable()上下都存在add\_dependencies()，需要在add\_exacutable()的下方添加add\_dependencies()。修改后即可成功编译运行。

                        e. 这里的源文件和可执行文件是一体的

####                 2）自定义源文件调用

                        需求:设计头文件与源文件，在可执行文件中包含头文件。

                        流程:

                                a. 编写头文件；

                                b. 编写源文件；

                                c. 编写可执行文件；

                                d. 编辑配置文件并执行。

                        注意：

                                a. 这里的源文件和可执行文件是两个不同的cpp文件。

                                b. 源文件（self\_def\_src.cpp）没有设置节点(ros::init())，因此不需要写入launch中，不需要被启动。

## 八.ROS运行管理

###         1. ROS元功能包

                Q：什么是元功能包？

                A：将不同的功能包打包成一个功能包，当安装其中的某个功能模块时，直接调用打包后的功能包即可，这打包后的“一个功能包”称为元功能包。

                实现：

                        1）新建一个功能包（在工作空间路径下输入以下命令创建新包）      

```undefined
catkin_create_pkg 新功能包名 std_msgs rospy roscpp
```

                        2）修改package.xml，如下：

```cobol
<exec_depend>被集成的功能包</exec_depend>

.....

<export>

<metapackage />

</export>
```

注意：“被集成的功能包”不能与新建的元功能包重名，否则在后面的编译运行中可能导致ros崩溃。

                        3）修改CMakeLists.txt，如下：

```scss
cmake_minimum_required(VERSION 3.0.2)

project(demo)

find_package(catkin REQUIRED)

catkin_metapackage()
```

                        注意：CMakeLists.txt中不可以有换行

###         2. ROS节点管理launch文件

                1）launch文件标签之launch

                        <launch>标签是所有 launch 文件的根标签，充当其他标签的容器

                2）launch文件标签之node

                        <node>标签用于指定 ROS 节点

                        注意：roslaunch 命令不能保证按照 node 的声明顺序来启动节点，节点的启动是多进程的。

                3）launch文件标签之include

                        include标签用于将另一个 xml 格式的 launch 文件导入到当前文件

                4）launch文件标签之remap

                        用于话题重命名

                5）launch文件标签之param

                        <param>标签主要用于在参数服务器上设置参数，参数源可以在标签中通过 value 指定，在<node>标签中视为私有。

                6）launch文件标签之rosparam

                        <rosparam>标签可以从 YAML 文件导入参数，或将参数导出到 YAML 文件，也可以用来删除参数，<rosparam>标签在<node>标签中视为私有。

                7）launch文件标签之group

                        <group>标签可以对节点分组，具有 ns(namespace命名空间) 属性，可以让节点归属某个命名空间。

                8）launch文件标签之arg

                        <arg>标签是用于动态传参，类似于函数的参数，可以增强launch文件的灵活性。

                        传参示例：

                        launch文件传参语法实现：设launch文件名为 yyy.launch

```cobol
<launch>

<arg name="xxx" />

<param name="param" value="$(arg xxx)" />

</launch>
```

                        命令行调用launch传参：

```scss
roslaunch 功能包名 yyy.launch(launch文件名) xxx:=待传值
```

                        执行后“param”的值就变为待传值了。

###         3. ROS工作空间覆盖

          所谓工作空间覆盖，是指不同工作空间中，存在重名的功能包的情形。

          执行顺序：功能包重名时，会按照 ROS\_PACKAGE\_PATH 查找，配置在前的会优先执行。

###         4. ROS节点名称重名

           节点名称重名问题解决方法有多种，如下：

                1）rosrun设置命名空间(使重名的两个节点归属于不同命名空间下)

                语法：rosrun 包名 节点名 \_\_ns:=新名称

```cobol
rosrun turtlesim turtlesim_node __ns:=/xxx

rosrun turtlesim turtlesim_node __ns:=/yyy
```

                查看结果：用rosnode list查看节点信息，结果如下

```cobol
/xxx/turtlesim

/yyy/turtlesim
```

                2）rosrun名称重映射(为节点起别名)

                语法：rosrun 包名 节点名 \_\_name:=新名称

```cobol
rosrun turtlesim turtlesim_node __name:=t1

rosrun turtlesim turtlesim_node __name:=t2
```

                查看结果：用rosnode list查看节点信息，结果如下

```cobol
/t1

/t2
```

                3）rosrun命名空间与名称重映射叠加(设置命名空间同时名称重映射)

                语法：rosrun 包名 节点名 \_\_ns:=新名称 \_\_name:=新名称

```cobol
rosrun turtlesim turtlesim_node __ns:=/xxx __name:=tn
```

                查看结果：用rosnode list查看节点信息，结果如下

```cobol
/xxx/tn
```

                4）launch文件设置命名空间与重映射

               node标签中的name和ns属性分别用于实现名称重映射和命名空间设置。使用方式如下：

```cobol
<launch>

<node pkg="turtlesim" type="turtlesim_node" name="t1" />

<node pkg="turtlesim" type="turtlesim_node" name="t2" />

<node pkg="turtlesim" type="turtlesim_node" name="t1" ns="hello"/>

</launch>
```

                查看结果：用rosnode list查看节点信息，结果如下

```cobol
/t1

/t2

/t1/hello
```

                5）编码设置命名空间与重映射

                自定义节点，可更灵活地设置命名空间/实现重映射。

                重映射：

```swift
ros::init(argc,argv,"zhangsan",ros::init_options::AnonymousName);
```

                命名空间设置：

```cobol
std::map<std::string, std::string> map;

map["__ns"] = "xxxx";

ros::init(map,"wangqiang");
```

###         5. ROS话题名称设置

                话题名称重名解决方法有多种，如下：

                1）rosrun设置话题重映射

                语法：

```go
rorun 包名 节点名 话题名:=新话题名称
```

                2）launch文件设置话题重映射

                语法：

```cobol
<node pkg="xxx" type="xxx" name="xxx">

<remap from="原话题" to="新话题" />

</node>
```

                例如：<remap from="/cmd\_vel" to="/turtle1/cmd\_vel" />

                3）编码设置话题名称

                        a. 全局名称

                        格式:以/开头的名称，及参考的是 / ，与命名空间和节点名称无关

                        示例：ros::Publisher pub = nh.advertise<std\_msgs::String>("/chatter",1000);

                        结果：/chatter

                        b. 相对名称

                        格式：非 / 开头，与节点名称平级，参考的是 /命名空间

                        示例：ros::Publisher pub = nh.advertise<std\_msgs::String>("chatter",1000);

                        结果：xxx/chatter

                        c. 私有名称

                        格式：以~开头的名称，是节点名称的下一级，参考的是 /命名空间/节点名称

                        示例：ros::NodeHandle nh("~");

                                   ros::Publisher pub = nh.advertise<std\_msgs::String>("chatter",1000);

                        结果：/xxx/hello/chatter

注意：当使用~,而话题名称有/开头时，话题名称是绝对的。即该话题前面没有其他前缀名称。示例如下：

                        ros::NodeHandle nh("~");

                        ros::Publisher pub = nh.advertise<std\_msgs::String>("/chatter/money",1000);

                        结果：/chatter/money

###         6. ROS参数名称设置

                参数名称重名解决方法有多种，如下：

                1）rosrun设置参数

                语法：

```go
rosrun 包名 节点名称 _参数名:=参数值
```

                注意：rosrun执行设置参数参数名使用的是私有模式

                2）launch文件设置参数

                注意：在 node 标签外设置的参数是全局性质的，参考的是 / ，在 node 标签中设置的参数是私有性质的，参考的是 /命名空间/节点名称。

                示例：

```cobol
<launch>

<param name="p1" value="100" />

<node pkg="turtlesim" type="turtlesim_node" name="t1">

<param name="p2" value="100" />

</node>

</launch>
```

                结果：rosparam list查看节点信息,显示结果:

```cobol
/p1

/t1/p1
```

                3）编码设置参数

                可用 ros::param 或者 ros::NodeHandle 来设置参数。

                        a. ros::param设置参数

```cobol
ros::param::set("/set_A",100); //全局,和命名空间以及节点名称无关

ros::param::set("set_B",100); //相对,参考 /命名空间

ros::param::set("~set_C",100); //私有,参考 /命名空间/节点名称
```

                        设namespace为xxx，节点名称为yyy，则使用rosparam list查看的结果为:

```cobol
/set_A

/xxx/set_B

/xxx/yyy/set_C
```

                        b. ros::NodeHandle设置参数

```cobol
ros::NodeHandle nh;

nh.setParam("/nh_A",100); //全局,和命名空间以及节点名称无关

nh.setParam("nh_B",100); //相对,参考 /命名空间

ros::NodeHandle nh_private("~");

nh_private.setParam("nh_C",100);//私有,参考 /命名空间/节点名称
```

                        设namespace为xxx，节点名称为yyy，则使用rosparam list查看的结果为:

```cobol
/nh_A

/xxx/nh_B

/xxx/yyy/nh_C
```

###         7. ROS分布式通信

                由于分布式通信涉及多台计算机之间的交互，而日后机器人导航过程中多台计算机分布式通信的应用并不多，因此这里不作介绍，读者若有兴趣请参考教程。

## 九.ROS常用组件

###         1. TF坐标变换

####                 1）坐标msg消息

                        a. geometry\_msgs/TransformStamped

                        命令行键入: 

```cobol
rosmsg info geometry_msgs/TransformStamped
```

                        会显示：

```cobol
std_msgs/Header header #头信息

uint32 seq #|-- 序列号

time stamp #|-- 时间戳

string frame_id #|-- 坐标 ID

string child_frame_id #子坐标系的 id

geometry_msgs/Transform transform #坐标信息

geometry_msgs/Vector3 translation #偏移量

float64 x #|-- X 方向的偏移量

float64 y #|-- Y 方向的偏移量

float64 z #|-- Z 方向的偏移量

geometry_msgs/Quaternion rotation #四元数

float64 x

float64 y

float64 z

float64 w
```

                           其中四元数用于表示坐标的相对姿态。

                        b. rosmsg info geometry\_msgs/PointStamped

                        命令行键入:

```cobol
rosmsg info geometry_msgs/PointStamped
```

                        会显示：

```cobol
std_msgs/Header header #头

uint32 seq #|-- 序号

time stamp #|-- 时间戳

string frame_id #|-- 所属坐标系的 id

geometry_msgs/Point point #点坐标

float64 x #|-- x y z 坐标

float64 y

float64 z
```

####                 2）静态坐标变换

                静态坐标变换，指两个坐标系之间的相对位置固定。

                实现分析:

                        a. 坐标系相对关系，可以通过发布方发布

                       b. 订阅方，订阅到发布的坐标系相对关系，再传入坐标点信息(可以写死)，然后借助于 tf 实现坐标变换，并将结果输出

                实现流程:

                        a. 新建功能包，添加依赖

                        b. 编写发布方实现

                        c. 编写订阅方实现

                        d. 执行并查看结果

                注意：

                        a. 补充：当坐标系之间的相对位置固定时，那么所需参数也是固定的: 父系坐标名称、子级坐标系名称、x偏移量、y偏移量、z偏移量、x 翻滚角度、y俯仰角度、z偏航角度，实现逻辑相同，参数不同，那么 [ROS 系统](https://so.csdn.net/so/search?q=ROS%20%E7%B3%BB%E7%BB%9F&spm=1001.2101.3001.7020)就已经封装好了专门的节点，使用方式如下，建议通过以下方式直接实现静态坐标系相对信息发布:

```cobol
rosrun tf2_ros static_transform_publisher x偏移量 y偏移量 z偏移量 z偏航角度 y俯仰角度 x翻滚角度 父级坐标系 子级坐标系
```

这个方式直接将publisher代码的功能全部打包，使用起来非常方便(但还要写subscriber的代码)。使用时可以手动用rosrun输入示例如下：

```cobol
rosrun tf2_ros static_transform_publisher 0.2 0 0.5 0 0 0 /baselink /laser
```

也可以直接写入launch文件，示例如下：

```cobol
<launch>

<node pkg="tf2_ros" type="static_transform_publisher" name="publish" args="0.2 0 0.5 0 0 0 /baselink /laser" output="screen" />

</launch>
```

                        b. 输入以上示例发布静态坐标系相对信息时可能出现以下报错：

```cobol
[ERROR] [1705501590.799704519]: [registerPublisher] Failed to contact master at [localhost:11311]. Retrying...
```

说明未连接管理者ROS Master，需要先走一下roscore再rosrun发布静态坐标系相对信息。

只要有一次启动过管理者(master或roscore)下面就无需再启动。

                        c. 可以借助于rviz显示坐标系关系，具体操作:

                                a）新建窗口输入命令:rviz;

                                b）在启动的 rviz 中设置Fixed Frame 为 base\_link

                                c）点击左下的 add 按钮，在弹出的窗口中选择 TF 组件，即可显示坐标关系。

                        d. 最后的输出中若有某个量输不出来，检查一下ROS\_INFO里前面的格式符写了没(%d,%f,%s等)。语法问题！

####                 3）动态坐标变换

                 动态坐标变换，指两个坐标系之间的相对位置变化。

                 但这里涉及小乌龟模型的操作，由于安装小乌龟包可能导致ROS中一些包的缺失，我没有安装小乌龟，因此凡是涉及小乌龟模型的操作都跳过。若读者想尝试，请参考教程。

####                 4）多坐标变换

                需求描述:现有坐标系统，父级坐标系统 world,下有两子级系统 son1，son2。son1 相对于 world，以及 son2 相对于 world 的关系是已知的，求 son1原点在 son2中的坐标(即求两坐标的相对关系)，又已知在 son1中一点的坐标，要求求出该点在 son2 中的坐标。

                实现分析:

                      a. 首先，需要发布 son1 相对于 world，以及 son2 相对于 world 的坐标消息。

                      b. 然后，需要订阅坐标发布消息，并取出订阅的消息，借助于 tf2 实现 son1 和 son2 的转换。

                      c. 最后，还要实现坐标点的转换。

                实现流程:

                        a. 新建功能包，添加依赖

                        b. 创建坐标相对关系发布方(需要发布两个坐标相对关系)

                        c. 创建坐标相对关系订阅方

                        d. 执行

                注意：

                        a. 首先需要发布两个子级坐标系相对信息，为了方便，这里使用静态坐标变换发布。直接使用ROS封装好的专门节点用rosrun或写入launch的方式发布。(这样就不用写publisher的代码,但还要写subscriber的代码)

                        b. 执行时可能会出现以下报错：

```cobol
ERROR: cannot launch node of type [coordinate_transform/static_transform_publisher]: Cannot locate node of type [static_transform_publisher] in package [coordinate_transform]. Make sure file exists in package path and permission is set to executable (chmod +x)
```

                        该报错说明系统在coordinate\_transform包里定位不到static\_transform\_publisher，即pkg写错了！这里的pkg不是写自己定义的功能包，而是写ROS系统里已经封装好的static\_transform\_publisher(静态坐标系相对信息发布代码)所在的功能包，即pkg=“tf2\_ros”。修改后即可成功输出结果。

                        也可以换一种方式，即用rosrun手动输入两个子级坐标系相对信息，再rosrun启动multi\_subscriber节点，输入顺序如下：

```cobol
rosrun tf2_ros static_transform_publisher 0.2 0.8 0.3 0 0 0 /world /son1

(新建终端)

rosrun tf2_ros static_transform_publisher 0.5 0 0 0 0 0 /world /son2

(新建终端)

rosrun coordinate_transform multi_subscriber
```

                        最后执行，即可成功输出结果如下：

```cobol
[ INFO] [1705546432.775514367]: Son1 相对于 Son2 的坐标关系:父坐标系ID=son2

[ INFO] [1705546432.775619478]: Son1 相对于 Son2 的坐标关系:子坐标系ID=son1

[ INFO] [1705546432.775658852]: Son1 相对于 Son2 的坐标关系:x=-0.30,y=0.80,z=0.30

[ INFO] [1705546432.775747989]: 在 Son2 中的坐标:x=0.70,y=2.80,z=3.30
```

                        c. 输入以上步骤时可能会出现以下报错：

```cobol
[ERROR] [1705547297.785113647]: static_transform_publisher exited due to not having the right number of arguments
```

                        或一直输出数据异常，原因是坐标系相对信息的参数输入不正确，父级坐标系(/world)与子级坐标系(/son1或/son2)之间一定要输入一个空格！这个空格是必要的分隔符，空格在哪里决定了是哪个坐标系到哪个坐标系的相对关系。

                        例如：

                        输入/world/son1，输出会出现上面的报错，表示无法找到坐标系之间的相对关系。

                        输入/ world/son1，输出为Spinning until killed publishing / to world/son2，表示坐标系world/son1到/的相对关系，显然不正确，后面运行也会异常。

                        输入/world /son1，输出为Spinning until killed publishing /world to /son2，表示坐标系/son1到/world的相对关系，这是正确的。

####                 5）坐标系关系查看

                        a. 准备

                        首先调用rospack find tf2\_tools查看是否包含该功能包，如果没有，请使用如下命令安装:

```cobol
sudo apt install ros-noetic-tf2-tools
```

                        b. 使用

                                a）生成pdf文件

                                启动坐标系广播程序之后，运行如下命令:

```cobol
rosrun tf2_tools view_frames.py
```

注意：所谓“启动”，就是要将有关坐标系的节点(son1/son2)全部启动(roslaunch或rosrun)，未启动的节点不会出现在坐标系关系图中。(multi\_subscriber无需启动，否则整个程序一旦启动就不能输入下一个命令了)

                                会产生类似于下面的日志信息:

```cobol
[INFO] [1592920556.827549]: Listening to tf data during 5 seconds...

[INFO] [1592920561.841536]: Generating graph in frames.pdf file...
```

                                查看当前目录会生成一个 frames.pdf 文件

                                b）查看文件

可以直接进入目录打开文件，或者调用命令查看文件:evince frames.pdf，即会出现坐标系关系图。

####                 6）TF2与TF

                启动TF2与TF两个版本的静态坐标变换：

                TF2：

```cobol
rosrun tf2_ros static_transform_publisher 0 0 0 0 0 0 /base_link /laser
```

                TF：

```cobol
rosrun tf static_transform_publisher 0 0 0 0 0 0 /base_link /laser 100
```

                可见TF 版本的启动中最后多一个参数，该参数是指定发布频率。

                运行结果比对：

                打印tf发布的话题消息：rostopic echo /tf 会循环输出当前坐标系信息

                打印tf2发布的话题消息：rostopic echo /tf\_static 输出坐标系信息只有一次

###         2. rosbag

                rosbag是ros中用以实现数据的留存与读取的专门工具。

####                 1）rosbag使用\_命令行

                        准备工作：创建目录保存录制的文件夹(一定要先建文件夹，不可忽略！否则后面的执行会出问题)

```bash
mkdir ./xxx

cd xxx
```

                        录制：

```scss
rosbag record -a -O(大写) 目标文件
```

                        (这里传入“-O 目标文件”参数相当于给bag文件命名）

                        结束录制使用 ctrl + c，在目录中会生成bag文件

                        查看文件信息：

```undefined
rosbag info 文件名
```

                        回放文件：

```undefined
rosbag play 文件名
```

                        注意：在查看文件与回放文件之前要先输入如下指令重建索引：

```undefined
rosbag reindex 文件名
```

                        否则查看与回放指令执行不了。

####                 2）rosbag使用\_编码实现

                命令实现不够灵活，可以使用编码的方式，增强录制与回放的灵活性。

                        a. 写bag

                        b. 读bag

###         3. rqt工具箱

####                 1）rqt安装启动与基本使用

                        a. 安装

                        一般只要你安装的是desktop-full版本就会自带工具箱

                        如果需要安装可以以如下方式安装：

```csharp
sudo apt-get install ros-noetic-rqt

sudo apt-get install ros-noetic-rqt-common-plugins
```

                        b. 启动

                        方式1：

```undefined
rqt
```

                        方式2：

```undefined
rosrun rqt_gui rqt_gui
```

                        c. 基本使用

                        启动 rqt 之后，可以通过 plugins 添加所需的插件

####                 2）rqt常用插件：rqt\_graph

                        简介：可视化显示计算图

                        启动：在终端中输入rqt\_graph启动

####                 3）rqt常用插件：rqt\_console

                        简介：rqt\_console 是 ROS 中用于显示和过滤日志的图形化插件

                        准备：编写 Node 节点输出各个级别的日志信息

                        启动： 操作顺序如下：

                                      catkin\_make

                                  -->roslaunch启动节点

                                  -->会看到终端输出各级别日志信息

                                  -->新建终端

                                  -->输入rqt\_console启动插件即可看到日志信息控制台，在第二栏也可选择要过滤掉的日志信息

####                 4）rqt常用插件：rqt\_plot

                        简介：图形绘制插件，可以以 2D 绘图的方式绘制发布在 topic 上的数据。

                        启动：在终端中输入rqt\_plot启动

####                 5）rqt常用插件：rqt\_bag

                        简介：录制和重放 bag 文件的图形化插件

                        启动：在终端中输入rqt\_bag启动

## 十.机器人系统仿真

##         1.概述

                1）以“ros- \[ROS\_DISTRO\] -desktop-full”命令安装ROS时，rviz会默认被安装。若没有安装，调用如下命令安装：

```less
sudo apt install ros-[ROS_DISTRO]-rviz
```

                rviz运行使用命令：

```undefined
rviz
```

                或者：

```undefined
rosrun rviz rviz
```

                2）以“ros- \[ROS\_DISTRO\] -desktop-full”命令安装ROS时，gazebo会默认被安装。

                gazebo运行使用命令:

```undefined
gazebo
```

                或者：

```undefined
rosrun gazebo_ros gazebo
```

                注意：gazebo启动异常以及解决

                        问题1:

```vbnet
VMware: vmw_ioctl_command error Invalid argument(无效的参数)
```

                        解决:

```bash
echo "export SVGA_VGPU10=0" >> ~/.bashrc

source .bashrc
```

                        问题2:

```scss
[Err] [REST.cc:205] Error in REST request
```

                        解决:

```cobol
sudo gedit ~/.ignition/fuel/config.yaml
```

                        然后将url : https://api.ignitionfuel.org用 # 注释掉

                        再添加url: [https://api.ignitionrobotics.org](https://api.ignitionrobotics.org/ "https://api.ignitionrobotics.org")

                        问题3:

                        启动时抛出异常:

```cobol
[gazebo-2] process has died [pid xxx, exit code 255, cmd.....
```

                        解决:

```undefined
killall gzserver
```

```undefined
killall gzclient
```

                3）URDF 不能单独使用，需要结合 Rviz 或 Gazebo，URDF 只是一个文件，需要在 Rviz 或 Gazebo 中渲染成图形化的机器人模型。

###         2. URDF集成Rviz基本流程

                需求：在Rviz中显示一个盒状机器人

                流程：

                        1）准备:新建功能包，导入依赖

                        2）核心:编写 urdf 文件

                        3）核心:在 launch 文件集成 URDF 与 Rviz

                        4）在 Rviz 中显示机器人模型

                优化：重复启动launch文件时，Rviz 之前的组件配置信息不会自动保存，需要重复执行步骤4的操作，为了方便使用，可以使用如下方式优化:

                点击Save Config As将配置保存在功能包的config目录中，然后，launch文件中 Rviz 的启动配置添加参数:args,值设置为-d 配置文件路径 ，如下：

```cobol
<launch>

<param name="robot_description" textfile="$(find 包名)urdf文件路径" />

<node pkg="rviz" type="rviz" name="rviz" args="-d $(find 包名)rviz配置文件路径" />

</launch>
```

                注意：代码中的参数/文件路径/包名等等一定要填对！！否则会有一堆麻烦的报错。

###         3. URDF语法详解

                1）robot

                根标签，在该标签内可以通过 name 属性设置机器人模型的名称

                属性：name ---> 指定机器人模型的名称

                2）link

                link 标签用于描述机器人某个部件的外观和物理属性。在 link 标签内，可以设计该部件的形状、尺寸、颜色、惯性矩阵、碰撞参数等一系列属性。

                属性：name ---> 为连杆命名

                子标签：

                        visual ---> 描述外观(对应的数据是可视的)

                                geometry 设置连杆的形状

                                        标签1: box(盒状)

                                                   属性:size=长(x) 宽(y) 高(z)

                                        标签2: cylinder(圆柱)

                                                   属性:radius=半径 length=高度

                                        标签3: sphere(球体)

                                                   属性:radius=半径

                                        标签4: mesh(为连杆添加皮肤)

                                                 属性: filename=资源路径(格式:package://<packagename>/<path>/文件)

                                origin 设置偏移量与倾斜弧度

                                        属性1: xyz=x偏移 y便宜 z偏移

                                        属性2: rpy=x翻滚 y俯仰 z偏航 (单位是弧度)

                                metrial 设置材料属性(颜色)

                                        属性: name

                                        标签: color

                                                属性: rgba=红绿蓝权重值与透明度 (每个权重值以及透明度取值\[0,1\])

                        collision ---> 连杆的碰撞属性

                        Inertial ---> 连杆的惯性矩阵

                3）joint

                用于描述机器人关节的运动学和动力学属性。不同的关节有不同的运动形式: 旋转、滑动、固定、旋转速度、旋转角度限制等等。

                属性：

                       name ---> 为关节命名

                       type ---> 关节运动形式

                                continuous: 旋转关节，可以绕单轴无限旋转

                                revolute: 旋转关节，类似于 continuous,但是有旋转角度限制

                                prismatic: 滑动关节，沿某一轴线移动的关节，有位置极限

                                planer: 平面关节，允许在平面正交方向上平移或旋转

                                floating: 浮动关节，允许进行平移、旋转运动

                                fixed: 固定关节，不允许运动的特殊关节

                子标签:

                        parent(必需的)

                        parent link的名字是一个强制的属性：

                                link:父级连杆的名字，是这个link在机器人结构树中的名字。child(必需的)

                        child link的名字是一个强制的属性：

                                link:子级连杆的名字，是这个link在机器人结构树中的名字。

                        origin

                                属性: xyz=各轴线上的偏移量 rpy=各轴线上的偏移弧度。

                        axis

                                属性: xyz用于设置围绕哪个关节轴运动。

                案例实现需求：创建机器人模型，底盘为长方体，在长方体的前面添加一摄像头，摄像头可以沿着 Z 轴 360 度旋转。

                注意：

                a）用base\_footprint优化urdf:前面实现的机器人模型是半沉到地下的，因为默认情况下: 底盘的中心点位于地图原点上，所以会导致这种情况产生，可以使用的优化策略，将初始 link 设置为一个尺寸极小的 link(比如半径为 0.001m 的球体，或边长为 0.001m 的立方体)，然后再在初始 link 上添加底盘等刚体，这样实现，虽然仍然存在初始link半沉的现象，但是基本可以忽略了。这个初始 link 一般称之为 base\_footprint.

                b）可能遇到的问题及其解决(一般不会遇到，程序正常运行)

                问题1：命令行输出如下错误提示：

```vbnet
UnicodeEncodeError:

[joint_state_publisher-3] process has died [pid 4443, exit code 1, cmd /opt/ros/melodic/lib/joint_state_publisher/joint_state_publisher __name:=joint_state_publisher __log:=/home/rosmelodic/.ros/log/b38967c0-0acb-11eb-aee3-0800278ee10c/joint_state_publisher-3.log].

logfile:/home/rosmelodic/.ros/log/b38967c0-0acb-11eb-aee3-0800278ee10c/joint_state_publisher-3*.log
```

                该报错是rviz中提示坐标变换异常，导致机器人部件显示结构异常。

                原因：编码问题导致

                解决：去除URDF中的中文注释

                问题2：命令行出现如下报错：

```scss
[ERROR] [1584370263.037038]: Could not find the GUI, install the 'joint_state_publisher_gui' package
```

                解决：在终端输入sudo apt install ros-noetic-joint-state-publisher-gui

                4）练习

                需求描述:创建一个四轮圆柱状机器人模型，机器人参数如下,底盘为圆柱状，半径 10cm，高 8cm，四轮由两个驱动轮和两个万向支撑轮组成，两个驱动轮半径为 3.25cm,轮胎宽度1.5cm，两个万向轮为球状，半径 0.75cm，底盘离地间距为 1.5cm(与万向轮直径一致)

                实现流程:

                创建机器人模型可以分步骤实现:

                        a. 新建 urdf 文件，并与 launch 文件集成

                        b. 搭建底盘

                        c. 在底盘上添加两个驱动轮

                        d. 在底盘上添加两个万向轮

                5）URDF工具

                        a. 使用URDF工具前先安装：

```undefined
sudo apt install liburdfdom-tools
```

                        b. check\_urdf语法检查

                        进入urdf文件所属目录，调用:check\_urdf urdf文件名，如果不抛出异常，说明文件合法,否则非法(有Error)。

                        c. urdf\_to\_graphiz结构查看

                       进入urdf文件所属目录，调用:

```cobol
urdf_to_graphiz urdf文件名
```

                       当前目录下会生成 pdf 文件。

###         4. URDF优化\_xacro

####                 1）Xacro\_语法详解

                  在使用 xacro 生成 urdf 时，根标签robot中必须包含命名空间声明:xmlns:xacro="http://wiki.ros.org/xacro"

                   a. 属性与算术运算

                        用于封装 URDF 中的一些字段，比如: PI 值，小车的尺寸，轮子半径 ....

                        属性定义：<xacro:property name="xxxx" value="yyyy" />

                        属性调用：${属性名称}

                        算术运算：${数学表达式}

                   b. 宏

                        类似于函数实现，提高代码复用率，优化代码结构，提高安全性

                        宏定义：

```cobol
<xacro:macro name="宏名称" params="参数列表(多参数之间使用空格分隔)">

.....

参数调用格式: ${参数名}

</xacro:macro>
```

                        宏调用：

```cobol
<xacro:宏名称 参数1=xxx 参数2=xxx/>
```

                   c. 文件包含

                机器人由多部件组成，不同部件可能封装为单独的 xacro 文件，最后再将不同的文件集成，组合为完整机器人，可以使用文件包含实现。

                方法：

```cobol
<robot name="xxx" xmlns:xacro="http://wiki.ros.org/xacro">

<xacro:include filename="my_base.xacro" />

<xacro:include filename="my_camera.xacro" />

<xacro:include filename="my_laser.xacro" />

....

</robot>
```

####                 2）Xacro\_完整使用案例

                需求描述: 使用 Xacro 优化 URDF 版的小车底盘模型实现（具体实现可参考教程）

###         5. Rviz中控制机器人模型运动

                Arbotix使用流程：

                需求描述：控制机器人模型在rviz中做圆周运动

                实现流程:

                        a）安装 Arbotix

                        b）创建新功能包，准备机器人 urdf、xacro 文件

                        c）添加 Arbotix 配置文件

                        d）编写 launch 文件配置 Arbotix

                        e）启动 launch 文件并控制机器人模型运动

                a.安装Arbotix

                   输入

```scss
sudo apt-get install ros-<<VersionName()>>-arbotix
```

                    将 <<VsersionName()>> 替换成当前 ROS 版本名称，这里是noetic版本。

                b. 添加arbotix所需配置文件

                    在config目录下新建一个文件arbotix.yaml，在该文件下编写arbotix配置文件。（配置文件具体写法见教程）

                c. launch文件中配置arbotix节点（launch文件具体写法见教程）

                d. 启动launch文件并控制机器人模型运动

                     启动launch后:

                        a）配置rviz

                        b）控制小车运动

                     此时调用 rostopic list 会发现一个熟悉的话题: /cmd\_vel

                     下面我们就可以调用以下命令发布cmd\_vel话题消息控制小车运动了：

```cobol
rostopic pub -r 10 /cmd_vel geometry_msgs/Twist '{linear: {x: 0.2, y: 0, z: 0}, angular: {x: 0, y: 0, z: 0.5}}'
```

###         6. URDF集成Gazebo

####                 1）URDF与Gazebo基本集成流程

                        a. 创建功能包，导入依赖项

                        b. 编写 URDF 或 Xacro 文件

                        c. 启动 Gazebo 并显示机器人模型

注意1：当 URDF 需要与 Gazebo 集成（在Gazebo中搭建仿真环境）时，和 Rviz 有明显区别:

1.必须使用 collision 标签，因为既然是仿真环境，那么必然涉及到碰撞检测，collision 提供碰撞检测的依据。

2.必须使用 inertial 标签，此标签标注了当前机器人某个刚体部分的惯性矩阵，用于一些力学相关的仿真计算。

3.颜色设置，也需要重新使用 gazebo 标签标注，因为之前的颜色设置为了方便调试包含透明度，仿真环境下没有此选项。

注意2：launch文件实现中启动Gazebo仿真环境的代码不要改动！即：

```xml
<include file="$(find gazebo_ros)/launch/empty_world.launch" />
```

以上代码不要乱改！文件路径是固定的！

####                 2）相关设置

                        a. collision

                        如果机器人link是标准的几何体形状，和link的 visual 属性设置一致即可。

                        b. inertial

                        惯性矩阵的设置需要结合link的质量与外形参数动态生成，标准的球体、圆柱与立方体的惯性矩阵公式如下(已经封装为 xacro 实现):

                        球体惯性矩阵：

```cobol
<xacro:macro name="sphere_inertial_matrix" params="m r">

<inertial>

<mass value="${m}" />

<inertia ixx="${2*m*r*r/5}" ixy="0" ixz="0"

iyy="${2*m*r*r/5}" iyz="0"

izz="${2*m*r*r/5}" />

</inertial>

</xacro:macro>
```

                        圆柱惯性矩阵：

```cobol
<xacro:macro name="cylinder_inertial_matrix" params="m r h">

<inertial>

<mass value="${m}" />

<inertia ixx="${m*(3*r*r+h*h)/12}" ixy = "0" ixz = "0"

iyy="${m*(3*r*r+h*h)/12}" iyz = "0"

izz="${m*r*r/2}" />

</inertial>

</xacro:macro>
```

                        立方体惯性矩阵：

```cobol
<xacro:macro name="Box_inertial_matrix" params="m l w h">

<inertial>

<mass value="${m}" />

<inertia ixx="${m*(h*h + l*l)/12}" ixy = "0" ixz = "0"

iyy="${m*(w*w + l*l)/12}" iyz= "0"

izz="${m*(w*w + h*h)/12}" />

</inertial>

</xacro:macro>
```

        注意：除了 base\_footprint 外，机器人的每个刚体部分都需要设置惯性矩阵，且惯性矩阵必须经计算得出，如果随意定义刚体部分的惯性矩阵，那么可能会导致机器人在 Gazebo 中出现抖动，移动等现象。

                        c. 颜色设置

                        在 gazebo 中显示 link 的颜色，必须要使用指定的标签:

```cobol
<gazebo reference="link节点名称">

<material>Gazebo/Blue</material>

</gazebo>
```

                        注意： material 标签中，设置的值区分大小写，颜色可以设置为 Red Blue Green Black(首字母大写)等。

####                 3）Gazebo仿真环境搭建

                        a. 直接添加内置组件创建仿真环境

                                a）启动Gazebo并添加组件(在终端中输入 gazebo 即可)

                                b）保存仿真环境(注意命名时要加文件后缀：xxx.world)

                                c）启动

                        b. 自定义仿真环境

                                a）启动gazebo打开构建面板，绘制仿真环境

                                b）保存构建的环境

                                        先点击左上角 file

                                ---> Save (保存路径功能包下的: models)

                                --->然后再点击 file

                                ---> Exit Building Editor

                                c）同方式1，插入一些组件，保存为world文件

                                d）启动(同方式1)

                        c. 使用官方提供的插件

                                a）下载官方模型库

                                git clone [https://github.com/osrf/gazebo\_models](https://github.com/osrf/gazebo_models "https://github.com/osrf/gazebo_models")

                                这种方法一开始行不通，尝试了很多解决方法仍然有如下报错：

```cobol
fatal: 无法访问 'https://github.com/osrf/gazebo_models/'：Failed to connect to github.com port 443: 连接超时
```

                                那么就换一种方法：直接进入github定位到以上网址将模型文件直接下载下来，这种方法是有效的。

                                b）将模型库复制进gazebo

                                将得到的gazebo\_models文件夹内容复制到 /usr/share/gazebo-\*/models

                                c）应用

                                重启Gazebo，选择左侧菜单栏的insert就可以选择并插入相关道具了。

                        注意：

                        a）若在上面的过程中某次运行launch文件时出现如下报错：

```cobol
[gazebo-2] process has died [pid 196712, exit code 255, cmd /opt/ros/noetic/lib/gazebo_ros/gzserver -e ode worlds/empty.world __name:=gazebo__log:=/home/zhenghao/.ros/log/87787dae-b6c8-11ee-a95d-bd8992a062f5/gazebo-2.log].log file: /home/zhenghao/.ros/log/87787dae-b6c8-11ee-a95d-bd8992a062f5/gazebo-2*.log
```

                        或输入gazebo时无法打开gazebo界面，

                        输入以下指令后再启动launch文件即可正常运行了：

```undefined
killall gzserver
```

                        b）若要打开已建好的models中的文件，先用gazebo指令进入Gazebo，找到insert窗口点击AddPath，选择要打开的model文件即可在insert窗口中将该model文件加入插件中，要用时直接在界面中插入已建好的模型即可。

                        c）将已建好的模型保存为world：点击file，选中Save World As即可。

###         7. URDF/Gazebo/Rviz综合应用

####                 1）机器人运动控制以及里程计信息显示

                        a. ros\_control（教程中有较详细的介绍，不是重点）

                        b. 运动控制实现流程(Gazebo)

                        基本流程：

                                a）已经创建完毕的机器人模型，编写一个单独的 xacro 文件，为机器人模型添加传动装置以及控制器。

                                b）将此文件集成进xacro文件

                                c）启动 Gazebo 并发布 /cmd\_vel 消息控制机器人运动

                        注意：

                                a）输入roslaunch指令启动launch时可能会出现以下报错：

```cobol
RLException: Invalid roslaunch XML syntax: mismatched tag: line 9, column 6 The traceback for the exception was written to the log file
```

此时检查一下<include>标签，是否有多余的“/”(include标签的功能未结束时一定不能加“/”，会把该功能提前结束掉，下面语句不被执行) 例如我这里是在

<include file="$(find gazebo\_ros)/launch/empty\_world.launch" > 最后多加了“/”：

<include file="$(find gazebo\_ros)/launch/empty\_world.launch" /> include提前结束，导致后面的语句无法正常执行。去掉“/”即可运行。

                                b）启动 launch 文件，使用 topic list 查看话题列表，会发现多了 /cmd\_vel 然后发布 cmd\_vel 消息控制即可。使用命令控制：

```cobol
rostopic pub -r 10 /cmd_vel geometry_msgs/Twist '{linear: {x: 0.2, y: 0, z: 0}, angular: {x: 0, y: 0, z: 0.5}}'
```

                        c. Rviz查看里程计信息

                        里程计：机器人相对出发点坐标系的位姿状态(X 坐标 Y 坐标 Z坐标以及朝向)

                                a）启动Rviz(修改launch文件)

                                b）添加组件(这里最好在launch文件中保存配置，即在launch文件中添加rviz的值(args)固定配置，以后打开不再需要重复配置)

####                 2）雷达信息仿真以及显示

                        a. Gazebo仿真雷达

                                a）新建 Xacro 文件，配置雷达传感器信息

                                b）xacro文件集成

                                c）启动仿真环境

                        b. Rviz显示雷达数据

                                a）启动rviz，添加雷达信息插件

                                b）保存配置，将配置写至launch文件中

                注意: 若雷达一直无法显示,就将Rviz中LaserScan功能块展开,将其中的Topic改成/scan就能在rviz中显示雷达扫描结果.(记得保存配置)

####                 3）摄像头信息仿真以及显示

                        a. Gazebo仿真摄像头

                                a）新建xacro文件，配置摄像头传感器信息

                                b）xacro文件集成

                                c）启动仿真环境

                        b. Rviz显示摄像头数据

####                 4）kinect信息仿真以及显示

                        a. Gazebo仿真kinect

                                a）新建xacro文件，配置kinetic传感器信息

                                b）xacro文件集成

                                c）启动仿真环境

                        b. Rviz显示kinect数据

补充: 在kinect中也可以以点云的方式显示感知周围环境，在 rviz 中点击Add添加PointCloud2即可.

注意: 点云图未显示很可能是kinetic的xacro文件中kinect link名称未改正确，一定要改成support！

可能出现的问题: 在rviz中显示时错位。

原因: 在kinect中图像数据与点云数据使用了两套坐标系统，且两套坐标系统位姿并不一致。

解决:

        a）在插件中为kinect设置坐标系，修改配置文件的<frameName>标签内容：

```cobol
<frameName>support_depth</frameName>
```

        b）发布新设置的坐标系到kinect连杆的坐标变换关系，在启动rviz的launch中，添加:

```cobol
<node pkg="tf2_ros" type="static_transform_publisher" name="static_transform_publisher" args="0 0 0 -1.57 0 -1.57 /support /support_depth" />
```

                启动rviz,重新显示即可.

注意:

        a）到此为止若有雷达或摄像头无法显示的情况,就多关几个终端,把命令控制(即rostopic pub -r 10 /cmd\_vel geometry\_msgs/Twist '{linear: {x: 0.2, y: 0, z: 0}, angular: {x: 0, y: 0, z: 0.5}}')关掉,再启动roslaunch即可正常显示雷达扫描和摄像头拍摄结果.

        b）键盘控制小车运动:

        安装指令:

```csharp
sudo apt-get install ros-noetic-teleop-twist-keyboard
```

        运行指令:

```undefined
rosrun teleop_twist_keyboard teleop_twist_keyboard.py
```

        即可通过键盘控制小车,注意控制的时候一定要将光标放在控制台上。

## 十一. 机器人导航(仿真)

###         1. 导航实现

                先安装相关的ROS功能包:

                a. 安装 gmapping 包(用于构建地图):

```cobol
a.sudo apt install ros-<ROS版本>-gmapping
```

                b. 安装地图服务包(用于保存与读取地图):

```cobol
sudo apt install ros-<ROS版本>-map-server
```

                c. 安装 navigation 包(用于定位以及路径规划):

```cobol
sudo apt install ros-<ROS版本>-navigation
```

                注意: <ROS版本>这里是noetic

                d. 新建功能包，并导入依赖:

```cobol
gmapping map_server amcl move_base
```

####                 1）SLAM建图

                        a. 订阅的topic

                                a）tf (tf/tfMessage)

                                        用于雷达、底盘与里程计之间的坐标变换消息。

                                b）scan(sensor\_msgs/LaserScan)

                                        SLAM所需的雷达信息。

                        b. 发布的topic

                                a）map\_metadata(nav\_msgs/MapMetaData)

                                        地图元数据，包括地图的宽度、高度、分辨率等，该消息会固定更新。

                                b）map(nav\_msgs/OccupancyGrid)

                                        地图栅格数据，一般会在rviz中以图形化的方式显示。

                                c）~entropy(std\_msgs/Float64)

                                        机器人姿态分布熵估计(值越大，不确定性越大)。

                        c.服务

                                dynamic\_map(nav\_msgs/GetMap)

                                用于获取地图数据。

                        d. 参数

                                a）~base\_frame(string, default:"base\_link")

                                        机器人基坐标系。

                                b）~map\_frame(string, default:"map")

                                        地图坐标系。

                                c）~odom\_frame(string, default:"odom")

                                        里程计坐标系。

                                d）~map\_update\_interval(float, default: 5.0)

                                        地图更新频率，根据指定的值设计更新间隔。

                                e）~maxUrange(float, default: 80.0)

                                        激光探测的最大可用范围(超出此阈值，被截断)。

                                f）~maxRange(float)

                                        激光探测的最大范围。

                        e. 所需的坐标变换

                                a）雷达坐标系→基坐标系

                                一般由 robot\_state\_publisher 或 static\_transform\_publisher 发布。

                                b）基坐标系→里程计坐标系

                                一般由里程计节点发布。

                        f. gmapping的使用

                                a）编写gmapping节点相关launch文件

                                b）执行

                                        1.先启动 Gazebo 仿真环境(此过程略)；

                                        2.然后再启动地图绘制的 launch 文件:

                        注意:

                        若有如下报错:

```vbscript
[ERROR] [1705739160.085443573]: Could not find parameter robot_description on parameter server
```

在参数服务器上找不到参数 robots\_desctiption,也就是在launch文件中找不到机器人描述的URDF文件.可以将gazebo\_pkg中xacro目录的文件全部拷到新软件包中,然后在launch文件中添加:

```cobol
<param name="robot_description" command="$(find xacro)/xacro $(find navigation_pkg)/xacro/mix_result.xacro" />
```

即可正常运行launch文件了

                                        3.启动键盘键盘控制节点，用于控制机器人运动建图

```undefined
rosrun teleop_twist_keyboard teleop_twist_keyboard.py
```

                                        4.在 rviz 中添加组件，显示栅格地图

这样就可以通过键盘控制gazebo中的机器人运动，同时，在rviz中也可显示gmapping发布的栅格地图数据.

####                 2）地图服务

                        a. map\_server使用之地图保存节点(map\_saver)

                                a）节点说明

                                b）地图保存launch文件

                        地图保存的语法如下:

```undefined
rosrun map_server map_saver -f 地图名称
```

                        (教程中编写launch的方式太复杂)

                        注意：地图没有保存好之前千万不要把建图与键盘控制节点进程关闭！

                                c）保存结果解释

                                xxx.pgm 本质是一张图片，直接使用图片查看程序即可打开。

                                xxx.yaml 保存的是地图的元数据信息，用于描述图片，内容格式如下:

```cobol
image: /home/rosmelodic/ws02_nav/src/mycar_nav/map/nav.pgm

resolution: 0.050000

origin: [-50.000000, -50.000000, 0.000000]

negate: 0

occupied_thresh: 0.65

free_thresh: 0.196
```

-                    解释：

                        image:被描述的图片资源路径，可以是绝对路径也可以是相对路径。

                        resolution: 图片分辨率(单位: m/像素)。

                        origin: 地图中左下像素的二维姿势，为（x，y，偏航），偏航为逆时针旋转（偏航= 0表示无旋转）。

                        occupied\_thresh: 占用概率大于此阈值的像素被视为完全占用。

                        free\_thresh: 占用率小于此阈值的像素被视为完全空闲。

                        negate: 是否应该颠倒白色/黑色自由/占用的语义。

-                    map\_server 中障碍物计算规则:

                a）地图中的每一个像素取值在 \[0,255\] 之间，白色为 255，黑色为 0，该值设为 x；

                b）map\_server 会将像素值作为判断是否是障碍物的依据，首先计算比例: p = (255 - x) / 255.0，白色为0，黑色为1(negate为true，则p = x / 255.0)；

                c）根据步骤2计算的比例判断是否是障碍物，如果 p > occupied\_thresh 那么视为障碍物，如果 p < free\_thresh 那么视为无物。

                        b. map\_server使用之地图服务

                                a）map\_\_server节点说明

                                b）地图读取

                通过 map\_server 的 map\_server 节点可以读取栅格地图数据，编写 launch 文件如下:

```xml
<launch>

<arg name="map" default="nav.yaml" />

<node name="map_server" pkg="map_server" type="map_server" args="$(find mycar_nav)/map/$(arg map)"/>

</launch>
```

                其中参数是地图描述文件的资源路径，执行该launch文件，该节点会发布话题:map(nav\_msgs/OccupancyGrid)

                注意：这里是读取，不是显示。

                                c）地图显示

                                在 rviz 中使用 map 组件可以显示栅格地图

                                注意：可以直接把启动rviz的代码加到launch文件中，这样读取地图后可以直接在rviz中显示地图。

####                 3）定位

                注意：这里amcl节点相关的launch文件与测试launch文件是两个不同的launch文件(amcl.launch需要引用diff\_amcl.launch)，不能合成在同一个launch中。

####                 4）路径规划

                        a. 不同配置文件的相关解释：

                                a）costmap\_common\_params.yaml

                                调用情况：全局路径规划与本地路径规划

                                涉及参数：机器人的尺寸、距离障碍物的安全距离、传感器信息等

                                b）global\_costmap\_params.yaml

                                调用情况：该文件用于全局代价地图参数设置

                                c）local\_costmap\_params.yaml

                                调用情况：该文件用于局部代价地图参数设置

                                d）base\_local\_planner\_params

                                调用情况：局部规划器参数配置

                                涉及参数：机器人的最大和最小速度限制值，也设定了加速度的阈值

                                e）参数配置技巧

                                全局代价地图可以将膨胀半径和障碍物系数设置的偏大一些；

                                本地代价地图可以将膨胀半径和障碍物系数设置的偏小一些。

####                 5）导航与SLAM建图

                        注意：运行时若有如下报错：

```cobol
RLException: roslaunch file contains multiple nodes named [/rviz].

Please check all <node> 'name' attributes to make sure they are unique.

Also check that $(anon id) use different ids.

The traceback for the exception was written to the log file
```

说明所涉及的launch文件中有多个运行rviz且其命名或地址不同的指令，这不被允许，因此只需要将此处无用的rviz注释掉即可。

###         2. 导航相关消息

                在导航实现中，导航相关消息已经在rviz中做了可视化处理，下面主要介绍这些消息的具体格式。而事实上在实际操作时很少涉及消息的具体格式，基本都会选择在rviz中做直观的可视化处理，因此下面的内容不是重点，浏览即可。

                1）地图

```cobol
rosmsg info nav_msgs/MapMetaData
```

```cobol
rosmsg info nav_msgs/OccupancyGrid
```

                2）里程计

```cobol
rosmsg info nav_msgs/Odometry
```

                3）坐标变换

```bash
rosmsg info tf/tfMessage
```

                4）定位

```cobol
rosmsg info geometry_msgs/PoseArray
```

                5）目标点与路径规划

                        a. 目标点

```cobol
rosmsg info move_base_msgs/MoveBaseActionGoal
```

                        b. 路径规划

```cobol
rosmsg info nav_msgs/Path
```

                6）激光雷达

```cobol
rosmsg info sensor_msgs/LaserScan
```

                7）相机

                a. 一般的图像数据(sensor\_msgs/Image)

```cobol
rosmsg info sensor_msgs/Image
```

                b. 压缩后的图像数据(sensor\_msgs/CompressedImage)

```cobol
rosmsg info sensor_msgs/CompressedImage
```

                c. 点云数据(带有深度信息的图像)(sensor\_msgs/PointClouds2)

```cobol
rosmsg info sensor_msgs/PointCloud2
```

                8）深度图像转激光数据

                a. 深度相机安装：

```vbscript
sudo apt-get install ros-<ROS版本>-depthimage-to-laserscan
```

                <ROS版本>这里是noetic

                b. 节点参数

                只需注意节点参数中一般需要设置的是output\_frame\_id：

                ~output\_frame\_id(str, default: camera\_depth\_frame)

                激光信息的ID

        注意：

        a）需要进行必要的urdf文件的修改。经过信息转换之后，深度相机也将发布雷达数据，为了不产生混淆，可以注释掉 xacro 文件中的关于激光雷达的部分内容。

        b）深度相机未显示点云图很可能是kinetic的xacro文件中kinect link名称未改正确，一定要改成support！

        c）若要达到机器人运动建图的效果，还要开启地图绘制的launch文件(slam.launch)，可以直接将启动slam节点和运行move\_base节点的代码用<include file>的方式写入depthimage.launch文件中，启动方便。

## _总结_

        上面汇总了ROS概述与环境搭建，ROS通信机制，ROS通信机制进阶，ROS运行管理，ROS常用组件，机器人系统仿真，机器人导航（仿真）七个部分的内容。最后的ROS进阶部分主要是对第二章通信机制存在的问题提出对应的优化策略，主要涉及Action通信，动态参数，pluginlib,nodelet。但由于该章知识点日后应用不多，因此对该部分内容的学习较为粗略。而对于机器人平台设计和机器人导航（实体），由于这两部分内容涉及导航技术在实车上的应用，而现在还未到该阶段，因此暂时不做学习，日后会有所补充。

        最后，若有错误或写得不清楚的地方，敬请指出，我会再修改或说明。