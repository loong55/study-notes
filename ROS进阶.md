## 1.pluginlib与nodelet

### 1.1pluginlib

**pluginlib**直译是插件库，所谓插件字面意思就是可插拔的组件，比如:以计算机为例，可以通过USB接口自由插拔的键盘、鼠标、U盘...都可以看作是插件实现，其基本原理就是通过规范化的USB接口协议实现计算机与USB设备的自由组合。同理，在软件编程中，插件是一种遵循一定规范的应用程序接口编写出来的程序，插件程序依赖于某个应用程序，且应用程序可以与不同的插件程序自由组合。在ROS中，也会经常使用到插件，场景如下:

> 1.导航插件:在导航中，涉及到路径规划模块，路径规划算法有多种，也可以自实现，导航应用时，可能需要测试不同算法的优劣以选择更合适的实现，这种场景下，ROS中就是通过插件的方式来实现不同算法的灵活切换的。
>
> 2.rviz插件:在rviz中已经提供了丰富的功能实现，但是即便如此，特定场景下，开发者可能需要实现某些定制化功能并集成到rviz中，这一集成过程也是基于插件的。

___

**概念**

**pluginlib**是一个c++库， 用来从一个ROS功能包中加载和卸载插件(plugin)。插件是指从运行时库中动态加载的类。通过使用Pluginlib，不必将某个应用程序显式地链接到包含某个类的库，Pluginlib可以随时打开包含类的库，而不需要应用程序事先知道包含类定义的库或者头文件。

插件是动态加载的c++类（通过pluginlib库，将其封装为动态链接库，即so文件），这些类需继承预定义的抽象基类，并在编译后通过Pluginlib的接口，在运行时加载到主程序中；pluginlib库可理解为usb拓展坞，插件可理解为u盘、键盘、鼠标。

```cpp
/*抽象基类*/
class base{
public:
    virtual void fun1() = 0;	//纯虚函数
    virtual void fun2() = 0;
    // ...其他函数    
    virtual ~base() = default;
}；
```

| 情况              | 是否支持多态       | 调用的函数版本     |
| ----------------- | ------------------ | ------------------ |
| 没有virtual       | 不支持             | 基类时的版本       |
| virtual           | 支持               | 子类版本（如果有） |
| 同时有virtual和=0 | 支持，强制子类实现 | 子类版本           |

如果基类中至少存在一个纯虚函数，那这个基类就被称为 抽象基类

抽象基类的特点：不能被实例化，只能被继承。并且继承抽象基类的子类必须要实现抽象基类中所有的纯虚函数

```cpp
/*派生类*/
class son1 : public base{
public:
    void fun1() override {/*求和*/}	//override代表覆写纯虚函数（.h声明后可以不用）
    void fun2() override {/*求积*/}
    // ...其他技能
};
```

派生类经过编译后，生成的so文件，才能称之为插件类，类似游戏的Mod，自身单独编译和主程序分离，有效减低代码的耦合性

**作用**

-   结构清晰；

-   低耦合，易修改，可维护性强；

-   可移植性强，更具复用性；

-   结构容易调整，插件可以自由增减；

___

**另请参考:**

-   [http://wiki.ros.org/pluginlib](http://wiki.ros.org/pluginlib)

-   [http://wiki.ros.org/pluginlib/Tutorials/Writing%20and%20Using%20a%20Simple%20Plugin](http://wiki.ros.org/pluginlib/Tutorials/Writing%20and%20Using%20a%20Simple%20Plugin)

**需求:**

以插件的方式实现正多边形的相关计算。

**实现流程:**

1.  准备；

2.  创建基类；

3.  创建插件类；

4.  注册插件;

5.  构建插件库;

6.  使插件可用于ROS工具链;

    -   配置xml

    -   导出插件

7.  使用插件;

8.  执行。

___

##### 1.准备

创建功能包demo03_plugin导入依赖: roscpp pluginlib。

在 VSCode中需要配置 .vascode/c\_cpp\_properties.json文件中关于 includepath 选项的设置。

```json
{
    "configurations": [
        {
            "browse": {
                "databaseFilename": "${default}",
                "limitSymbolsToIncludedHeaders": false
            },
            "includePath": [
                "/opt/ros/noetic/include/**",
                "/usr/include/**",
                "/home/ubuntu2004/advanced_ws/devel/include/**",
                "${workspaceFolder}/src/demo03_plugin/include"
            ],
            "name": "ROS",
            "intelliSenseMode": "gcc-x64",
            "compilerPath": "/usr/bin/gcc",
            "cStandard": "gnu17",
            "cppStandard": "c++17"
        }
    ],
    "version": 4
}
```

##### 2.创建基类

在 xxx/include/xxx下新建C++头文件: polygon\_base.h，所有的插件类都需要继承此基类，内容如下:

/home/book/ws/src/demo03_plugin/include/demo03_plugin/dbx_base.h

```cpp
#ifndef DBX_BASE_H_
#define DBX_BASE_H_

namespace dbx_base_ns
{
    /*
        注意：必须保证基类中包含无参构造
    */
  class Dbx_Base
  {
    protected:
      Dbx_Base(){}

    public:
      //计算周长的函数
      virtual double getlength() = 0;
      //初始化边长的函数
      virtual void initialize(double side_length) = 0;
      //析构函数
      virtual ~Dbx_Base(){}
  };
};
#endif
```

**PS:**基类必须提供无参构造函数，所以关于多边形的边长没有通过构造函数而是通过单独编写的initialize函数传参。

##### 3.创建插件

在 xxx/include/xxx下新建C++头文件:polygon\_plugins.h，内容如下:

/home/book/ws/src/demo03_plugin/include/demo03_plugin/dbx_plugins.h

```cpp
#ifndef DBX_PLUGINS_H_
#define DBX_PLUGINS_H_
#include "demo03_plugin/dbx_base.h"
namespace dbx_plugins_ns{
    //三边
    class SanBian: public dbx_base_ns::Dbx_Base{
        private:
        //私有成员变量
            double side_length;
        public:
            //构造函数
            SanBian(){
                side_length = 0.0;
            }
            void initialize(double side_length){
                this->side_length = side_length;
            }
            double getlength(){
                return side_length * 3;
            }
    };
    //四边
    class SiBian: public dbx_base_ns::Dbx_Base{
        private:
        //私有成员变量
            double side_length;
        public:
            //构造函数
            SiBian(){
                side_length = 0.0;
            }
            void initialize(double side_length){
                this->side_length = side_length;
            }
            double getlength(){
                return side_length * 4;
            }
    };
};
#endif
```

该文件中创建了正方形与三角形两个衍生类继承基类。

##### 4.注册插件

在 src 目录下新建 polygon_plugins.cpp 文件，内容如下:

/home/book/ws/src/demo03_plugin/src/plus.cpp

```cpp
#include "pluginlib/class_list_macros.h"
#include "demo03_plugin/dbx_base.h"
#include "demo03_plugin/dbx_plugins.h"

//外部导入拓展插件类,参数：子类，父类
PLUGINLIB_EXPORT_CLASS(dbx_plugins_ns::SanBian,dbx_base_ns::Dbx_Base)
PLUGINLIB_EXPORT_CLASS(dbx_plugins_ns::SiBian,dbx_base_ns::Dbx_Base)
```

该文件会将两个衍生类注册为插件。

##### 5.构建插件库

在 CMakeLists.txt 文件中设置内容如下:

```cmake
## Specify additional locations of header files
## Your package locations should be listed before other locations
include_directories(
include
  ${catkin_INCLUDE_DIRS}
)

## Declare a C++ library
add_library(plus
  src/plus.cpp
)
```

至此，可以调用 catkin\_make 编译，编译完成后，在工作空间/devel/lib目录下，会生成相关的 .so 文件。

##### 6.使插件可用于ROS工具链

6.1配置xml

功能包下新建文件:polygon\_plugins.xml,内容如下:

/home/book/ws/src/demo03_plugin/plus.xml

```xml
<!-- 
    需要定位动态链接库
        /home/book/ws/devel/lib/libplus.so
        library 根标签下的path属性
    声明子类与父类
        library 的子标签 class 声明
 -->
<!-- 插件库的相对路径 -->
<library path="lib/libplus">
  <!-- type="插件类" base_class_type="基类" -->
  <class type="dbx_plugins_ns::SanBian" base_class_type="dbx_base_ns::Dbx_Base">
    <!-- 描述信息 -->
    <description>正三边形插件</description>
  </class>
  <class type="dbx_plugins_ns::SiBian" base_class_type="dbx_base_ns::Dbx_Base">
    <!-- 描述信息 -->
    <description>正四边形插件</description>
  </class>
</library>
```

6.2导出插件

package.xml文件中设置内容如下:

```xml
  <!-- The export tag contains other, unspecified, tags -->
  <export>
    <!-- Other tools can request additional information be placed here -->
    <!-- ${prefix} 表示自动寻找功能包 -->
    <demo03_plugin plugin = "${prefix}/plus.xml" />
  </export>
```

标签<xxx />的名称应与基类所属的功能包名称一致，plugin属性值为上一步中创建的xml文件。

编译后，可以调用`rospack plugins --attrib=plugin xxx`命令查看配置是否正常，如无异常，会返回 .xml 文件的完整路径，这意味着插件已经正确的

集成到了ROS工具链。

##### 7.使用插件

src 下新建c++文件:polygon\_loader.cpp，内容如下:

/home/book/ws/src/demo03_plugin/src/use_plus.cpp

```cpp
//类加载器相关的头文件
#include "ros/ros.h"
#include "pluginlib/class_loader.h"
#include "demo03_plugin/dbx_base.h"
/*
    创建类加载器，根据需求加载相关插件
        1.创建类加载器
        2.使用类加载器实例化某个插件对象
        3.使用插件
*/
int main(int argc, char** argv)
{
    setlocale(LC_ALL,"");
  //1.创建类加载器 -- 参数1:基类功能包名称 参数2:基类全限定名称
  pluginlib::ClassLoader<dbx_base_ns::Dbx_Base> loader("demo03_plugin", "dbx_base_ns::Dbx_Base");

  try
  {
    //三角形周长
    //创建插件类实例 -- 参数:插件类全限定名称
    boost::shared_ptr<dbx_base_ns::Dbx_Base> san = loader.createInstance("dbx_plugins_ns::SanBian");
    //使用插件
    san->initialize(10);
    double length = san->getlength();
    ROS_INFO("三角形周长: %.2f", length);

    //四角形周长
    //创建插件类实例 -- 参数:插件类全限定名称
    boost::shared_ptr<dbx_base_ns::Dbx_Base> si = loader.createInstance("dbx_plugins_ns::SiBian");
    //使用插件
    si->initialize(10);
    double length2 = si->getlength();
    ROS_INFO("四边形周长: %.2f", length2);
  }
  catch(pluginlib::PluginlibException& ex)
  {
    ROS_ERROR("The plugin failed to load for some reason. Error: %s", ex.what());
  }
  return 0;
}
```

##### 8.执行

修改CMakeLists.txt文件，内容如下:

```cmake
add_executable(use_plus src/use_plus.cpp)
add_dependencies(use_plus ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
target_link_libraries(use_plus
  ${catkin_LIBRARIES}
)
```

编译然后执行:polygon\_loader，结果如下:

```shell
book@100ask:~/ws$ rosrun demo03_plugin use_plus 
[ INFO] [1736830990.742236624]: 三角形周长: 30.00
[ INFO] [1736830990.742361624]: 四边形周长: 40.00
```

___

### 1.2nodelet

ROS通信是基于Node(节点)的，Node使用方便、易于扩展，可以满足ROS中大多数应用场景，但是也存在一些局限性，由于一个Node启动之后独占一根进程，不同Node之间数据交互其实是不同进程之间的数据交互，当传输类似于图片、点云的大容量数据时，会出现延时与阻塞的情况，比如：

> 现在需要编写一个相机驱动，在该驱动中有两个节点实现:其中节点A负责发布原始图像数据，节点B订阅原始图像数据并在图像上标注人脸。如果节点A与节点B仍按照之前实现，两个节点分别对应不同的进程，在两个进程之间传递容量可观图像数据，可能就会出现延时的情况，那么该如何优化呢？

ROS中给出的解决方案是:Nodelet，通过Nodelet可以将多个节点集成进一个进程。

___

**概念**

nodelet软件包旨在提供在同一进程中运行多个算法(节点)的方式，不同算法之间通过传递指向数据的指针来代替了数据本身的传输(类似于编程传值与传址的区别)，从而实现零成本的数据拷贝。

nodelet功能包的核心实现也是插件，是对插件的进一步封装:

-   不同算法被封装进插件类，可以像单独的节点一样运行；
-   在该功能包中提供插件类实现的基类:Nodelet；
-   并且提供了加载插件类的类加载器:NodeletLoader。

**作用**

应用于大容量数据传输的场景，提高节点间的数据交互效率，避免延时与阻塞。

___

**另请参考:**

-   [http://wiki.ros.org/nodelet/](http://wiki.ros.org/nodelet/)

-   [http://wiki.ros.org/nodelet/Tutorials/Running%20a%20nodelet](http://wiki.ros.org/nodelet/Tutorials/Running%20a%20nodelet)

-   [https://github.com/ros/common\_tutorials/tree/noetic-devel/nodelet\_tutorial\_math](https://github.com/ros/common_tutorials/tree/noetic-devel/nodelet_tutorial_math)

```shell
book@100ask:~$ rosrun nodelet nodelet
Your usage: 
/opt/ros/melodic/lib/nodelet/nodelet 
nodelet usage:
nodelet load pkg/Type manager [--no-bond]   #将节点加载进管理器
nodelet standalone pkg/Type   #以独立进程加载某个节点
nodelet unload name manager   #移除某个管理器中的节点
nodelet manager              #管理器，管理不同节点
```

#### 10.4.1 使用演示

在ROS中内置了nodelet案例，我们先以该案例演示nodelet的基本使用语法，基本流程如下:

1.  案例简介；
2.  nodelet基本使用语法；
3.  内置案例调用。

1.案例简介

以“ros- \[ROS\_DISTRO\] -desktop-full”命令安装ROS时，nodelet默认被安装，如未安装，请调用如下命令自行安装:

```shell
sudo apt install ros-<<ROS_DISTRO>>-nodelet-tutorial-math
```

在该案例中，定义了一个Nodelet插件类:Plus，这个节点可以订阅一个数字，并将订阅到的数字与参数服务器中的 value 参数相加后再发布。



**需求:**再同一线程中启动两个Plus节点A与B，向A发布一个数字，然后经A处理后，再发布并作为B的输入，最后打印B的输出。

2.nodelet 基本使用语法

使用语法如下:

```
nodelet load pkg/Type manager - Launch a nodelet of type pkg/Type on manager manager
nodelet standalone pkg/Type   - Launch a nodelet of type pkg/Type in a standalone node
nodelet unload name manager   - Unload a nodelet a nodelet by name from manager
nodelet manager               - Launch a nodelet manager node
```

3.内置案例调用

1.启动roscore

```
roscore
```

2.启动manager

```
rosrun nodelet nodelet manager __name:=mymanager
```

\_\_name:= 用于设置管理器名称。

3.添加nodelet节点

添加第一个节点:

```
rosrun nodelet nodelet load nodelet_tutorial_math/Plus mymanager __name:=n1 _value:=100
```

添加第二个节点:

```
rosrun nodelet nodelet load nodelet_tutorial_math/Plus mymanager __name:=n2 _value:=-50 /n2/in:=/n1/out
```

PS: 解释

> rosrun nodelet nodelet load nodelet\_tutorial\_math/Plus mymanager \_\_name:=n1 \_value:=100
>
> 1.  rosnode list 查看，nodelet 的节点名称是: /n1；
> 2.  rostopic list 查看，订阅的话题是: /n1/in，发布的话题是: /n1/out；
> 3.  rosparam list查看，参数名称是: /n1/value。
>
> rosrun nodelet nodelet standalone nodelet\_tutorial\_math/Plus mymanager \_\_name:=n2 \_value:=-50 /n2/in:=/n1/out
>
> 1.  第二个nodelet 与第一个同理；
> 2.  第二个nodelet 订阅的话题由 /n2/in 重映射为 /n1/out。

**优化:**也可以将上述实现集成进launch文件:

```xml
<launch>
    <!-- 设置nodelet管理器 -->
    <node pkg="nodelet" type="nodelet" name="mymanager" args="manager" output="screen" />
    <!-- 启动节点1，名称为 n1, 参数 /n1/value 为100 -->
    <node pkg="nodelet" type="nodelet" name="n1" args="load nodelet_tutorial_math/Plus mymanager" output="screen" >
        <param name="value" value="100" />
    </node>
    <!-- 启动节点2，名称为 n2, 参数 /n2/value 为-50 -->
    <node pkg="nodelet" type="nodelet" name="n2" args="load nodelet_tutorial_math/Plus mymanager" output="screen" >
        <param name="value" value="-50" />
        <remap from="/n2/in" to="/n1/out" />
    </node>

</launch>
```

4.执行

向节点n1发布消息:

```
rostopic pub -r 10 /n1/in std_msgs/Float64 "data: 10.0"
```

打印节点n2发布的消息:

```
rostopic echo /n2/out
```

最终输出结果应该是:60。



#### 10.4.2 实现

nodelet本质也是插件，实现流程与插件实现流程类似，并且更为简单，不需要自定义接口，也不需要使用类加载器加载插件类。

**需求:**参考 nodelet 案例，编写 nodelet 插件类，可以订阅输入数据，设置参数，发布订阅数据与参数相加的结果。

**流程:**

1.  准备；

2.  创建插件类并注册插件;

3.  构建插件库;

4.  使插件可用于ROS工具链；

5.  执行。

1.准备

新建功能包demo04_nodelet，导入依赖: roscpp nodelet

2.创建插件类并注册插件

/home/book/ws/src/demo04_nodelet/src/myplus.cpp

```cpp
#include "nodelet/nodelet.h"
#include "pluginlib/class_list_macros.h"
#include "ros/ros.h"
#include "std_msgs/Float64.h"
/*
    需求：   首先，需要订阅一个浮点数据
            然后，将订阅的数据与参数服务器的指定参数相加
            最后，将最终结果发布
    流程：
        1.先确定需要的变量：订阅对象，发布对象，存储参数的变量；
        2.获取 NodleHandle;
        3.通过 NodleHandle 创建订阅对象和发布对象，解析参数；
        4.回调函数处理数据，并通过发布对象发布。
*/
namespace nodelet_demo_ns {
class MyPlus: public nodelet::Nodelet {
    public:
    MyPlus(){
        value = 0.0;
    }
    void onInit(){
        //获取 NodeHandle
        ros::NodeHandle& nh = getPrivateNodeHandle();
        //从参数服务器获取参数
        nh.getParam("value",value);
        //创建发布与订阅对象
        pub = nh.advertise<std_msgs::Float64>("out",100);//话题名称： /节点名/out
        sub = nh.subscribe<std_msgs::Float64>("in",100,&MyPlus::doCb,this);

    }
    //处理订阅的回调函数
    void doCb(const std_msgs::Float64::ConstPtr& p){
        double num = p->data;
        //数据处理
        double result = num + value;
        std_msgs::Float64 r;
        r.data = result;
        //发布
        pub.publish(r);
    }
    private:
    ros::Publisher pub;
    ros::Subscriber sub;
    double value;

};
}
PLUGINLIB_EXPORT_CLASS(nodelet_demo_ns::MyPlus,nodelet::Nodelet)
```

3.构建插件库

CMakeLists.txt配置如下：

```cmake
...
add_library(myplus
  src/myplus.cpp
)
...
target_link_libraries(myplus
  ${catkin_LIBRARIES}
)
```

编译后，会在 `工作空间/devel/lib/`先生成文件: libmyplus.so。

4.使插件可用于ROS工具链

4.1配置xml

新建 xml 文件，名称自定义(比如:my\_plus.xml)，内容如下：

/home/book/ws/src/demo04_nodelet/myplus.xml

```xml
<library path="lib/libmyplus">
    <class name="demo04_nodelet/MyPlus" type="nodelet_demo_ns::MyPlus" base_class_type="nodelet::Nodelet" >
        <description>hello</description>
    </class>
</library>
```

4.2导出插件

/home/book/ws/src/demo04_nodelet/package.xml

```xml
  <!-- The export tag contains other, unspecified, tags -->
  <export>
      <!-- Other tools can request additional information be placed here -->
      <nodelet plugin="${prefix}/myplus.xml" />
  </export>
```

5.执行

可以通过launch文件执行nodelet，示例内容如下:

```xml
<launch>
    <!-- 设置nodelet管理器 -->
    <node pkg="nodelet" type="nodelet" name="dasun" args="manager" output="screen" />
    <!-- 启动节点1，名称为 n1, 参数 /n1/value 为100 -->
    <node pkg="nodelet" type="nodelet" name="xiaowang" args="load demo04_nodelet/MyPlus dasun" output="screen" >
        <param name="value" value="100" />
    </node>
    <!-- 启动节点2，名称为 n2, 参数 /n2/value 为-50 -->
    <node pkg="nodelet" type="nodelet" name="ergou" args="load demo04_nodelet/MyPlus dasun" output="screen" >
        <param name="value" value="-50" />
        <remap from="/ergou/in" to="/xiaowang/out" />
    </node>

</launch>
```

运行launch文件，可以参考上一节方式向 p1发布数据，并订阅p2输出的数据，最终运行结果也与上一节类似。

4.执行

向节点n1发布消息:

```
rostopic pub -r 10 /xiaowang/in std_msgs/Float64 "data: 10.0"
```

打印节点n2发布的消息:

```
rostopic echo /ergou/out
```

最终输出结果应该是:60。

___



## 2.自定义全局路径规划器

参考视频：[香奈美教你在ROS中自定义全局路径规划器_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1qzZcYEEEx/?spm_id_from=333.1391.0.0)

#### 1.前情概念

**pluginlib**是一个c++库， 用来从一个ROS功能包中加载和卸载插件(plugin)。插件是指从运行时库中动态加载的类。通过使用Pluginlib，不必将某个应用程序显式地链接到包含某个类的库，Pluginlib可以随时打开包含类的库，而不需要应用程序事先知道包含类定义的库或者头文件。

插件是动态加载的c++类（通过pluginlib库，将其封装为动态链接库，即so文件），这些类需继承预定义的抽象基类，并在编译后通过Pluginlib的接口，在运行时加载到主程序中

```cpp
/*抽象基类*/
class base{
public:
    virtual void fun1() = 0;	//纯虚函数
    virtual void fun2() = 0;
    // ...其他函数    
    virtual ~base() = default;
}；
```

| 情况              | 是否支持多态       | 调用的函数版本     |
| ----------------- | ------------------ | ------------------ |
| 没有virtual       | 不支持             | 基类时的版本       |
| virtual           | 支持               | 子类版本（如果有） |
| 同时有virtual和=0 | 支持，强制子类实现 | 子类版本           |

如果基类中至少存在一个纯虚函数，那这个基类就被称为 抽象基类

抽象基类的特点：不能被实例化，只能被继承。并且继承抽象基类的子类必须要实现抽象基类中所有的纯虚函数

```cpp
/*派生类*/
class son1 : public base{
public:
    void fun1() override {/*求和*/}	//override代表覆写纯虚函数（.h声明后可以不用）
    void fun2() override {/*求积*/}
    // ...其他技能
};
```

派生类经过编译后，生成的so文件，才能称之为插件类，类似游戏的Mod，自身单独编译和主程序分离，有效减低代码的耦合性

#### 2.构建全局路径规划器流程

1.环境准备

> catkin目录创建
>
> ros包创建

2.简单全局路径规划器

> 1.头文件创建：继承抽象基类，声明函数
>
> 2..cpp具体实现：实现纯虚函数，注册插件接口
>
> 3.插件描述文件：plugin.xml编写，cmake参数更改

3.验证环节

> ros仿真环境测试

1.环境准备

​	catkin目录创建

```bash
bot@windows:~/nav$ mkdir -p catkin_wp/src
bot@windows:~/nav$ cd catkin_wp/
bot@windows:~/nav/catkin_wp$ catkin_make
bot@windows:~/nav/catkin_wp$ source ./devel/setup.bash
```

​	ros包创建

```bash
bot@windows:~/nav/catkin_wp/src$ catkin_create_pkg nav_global_planner 
	roscpp rospy pluginlib costmap_2d nav_core tf
```

```bash
bot@windows:~/nav/catkin_wp$ catkin_make
```

全局路径规划器参考网址：

[nav_core: nav_core::BaseGlobalPlanner Class Reference](https://docs.ros.org/en/api/nav_core/html/classnav__core_1_1BaseGlobalPlanner.html)

2.简单全局路径规划器

​	1.头文件创建：继承抽象基类，声明函数

src/nav_global_planner/include/nav_global_planner/nav_global_planner.h

```cpp
#ifndef NAV_GLOBAL_PLANNER_H
#define NAV_GLOBAL_PLANNER_H

#include <ros/ros.h>
#include <costmap_2d/costmap_2d_ros.h>
#include <nav_core/base_global_planner.h>
#include <vector>
#include <geometry_msgs/PoseStamped.h>

namespace nav_global_planner {
    //自定义全局路径规划器 抽象基类
    class NavGlobalPlanner : public nav_core::BaseGlobalPlanner{
        public:
            //声明类的构造函数
            NavGlobalPlanner();//先调用默认构造函数调用对象
            NavGlobalPlanner(std::string name,costmap_2d::Costmap2DROS * costmap_2d);//带参数的构造方法：避免直接调用initialize函数（代码规范）
            //实现纯虚函数
            void initialize(std::string name, costmap_2d::Costmap2DROS *costmap_ros);
            bool makePlan (const geometry_msgs::PoseStamped &start, 
                        const geometry_msgs::PoseStamped &goal,
                        std::vector< geometry_msgs::PoseStamped > &plan);
        
        private:
            costmap_2d::Costmap2D* costmap_;
            bool initialized_;
    };
}
#endif
```

