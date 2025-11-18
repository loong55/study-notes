### 第一章 ros2概述与系统搭建

#### 1.搭建流程

创建命名空间：

```shell
mkdir -p ws/src
cd ws
colcon build
```

创建功能包

```shell
ros2 pkg create pkg01_helloworld_cpp --build-type ament_cmake --dependencies rclcpp --node-name helloworld
```

编辑源码pkg01_helloworld_cpp/src

```cpp
#include "rclcpp/rclcpp.hpp"

class MyNode: public rclcpp::Node{
public:
    MyNode():Node("node_name"){
        RCLCPP_INFO(this->get_logger(),"hello world!");
    }

};

int main(int argc, char *argv[])
{
    rclcpp::init(argc,argv);
    auto node = std::make_shared<MyNode>();
    rclcpp::shutdown();
    return 0;
}
```

include爆红：包含路径属性添加

/opt/ros/humble/include/**

```cmake
# 声明cmake的最低版本
cmake_minimum_required(VERSION 3.8)
# 包名，需要与package.xml中的包名一致
project(pkg01_helloworld_cpp)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)
# 引入外部依赖包
find_package(rclcpp REQUIRED)

# 映射源文件与可执行文件
add_executable(helloworld src/helloworld.cpp)
# 设置目标依赖库
ament_target_dependencies(helloworld rclcpp)
# 定义安装规则
install(TARGETS 
  helloworld
  DESTINATION lib/${PROJECT_NAME})
.......
```

编译

```shell
cd ~/ws
colcon build
#自定义编译功能包：colcon build --packages-select cpp01_topic
```

执行

```shell
cd ~/ws
source ./install/setup.bash
ros2 run pkg01_helloworld_cpp helloworld
#ros2 run 功能包 可执行程序 参数
```

#### 2.auto字符

```cpp
#include<iostream>

int main()
{
    auto x = 5;
    auto y = 5.5;
    auto z = 'a';

    std::cout << "x = " << x << std::endl;
    std::cout << "y = " << y << std::endl;
    std::cout << "z = " << z << std::endl;

    // x = 5
    // y = 5.5
    // z = a
    return 0;
}
```

#### 3.函数包装器 function

```cpp
#include <iostream>
#include <functional>
//函数包装器 function
using namespace std;
void save_with_free_fun(const std::string& filename){
    cout << "自由函数 " << filename << endl;
}

class FileSaver{
public:
    void save_with_member_fun(const std::string& filename){
        cout << "成员函数 " << filename << endl;
    };
};

int main(){
    FileSaver fs;
    auto save_with_lambda_fun = [](const std::string& filename) -> void
    {
        cout << "lambda函数 " << filename << endl;
    };

    std::function<void(const std::string&)> save_1 = save_with_free_fun;

    //std::function<void(const std::string&)> save_2 = fs.save_with_member_fun;不合法，类似于
    std::function<void(const std::string&)> save_2 = std::bind(&FileSaver::save_with_member_fun, &fs, std::placeholders::_1);

    std::function<void(const std::string&)> save_3 = save_with_lambda_fun;

    save_1("file1.txt");
    save_2("file2.txt");
    save_3("file3.txt");

    return 0;

}
```

#### 4.lambda函数

```cpp
#include <iostream>
#include <algorithm>

int main()
{
    // Lambda expression
    // [capture list](parameters) -> return type { function body }
    // [捕获列表](参数) -> 返回类型 { 函数体 }

    auto add = [](int a,int b) -> int { return a + b; };
    int sum = add(3, 5);
    auto print_sum = [sum]() ->void { std::cout << "3 + 5 = " << sum << std::endl; };
    print_sum();
    
    return 0;
}
```

#### 5.make_shared智能指针

```cpp
#include <iostream>
#include <memory>
//智能指针使用
using namespace std;
int main()
{
    auto p1 = std::make_shared<string>("hello world");
    cout << "p1 use_count: " << p1.use_count() << "     p1 address: " << p1.get() << endl;
    auto p2 = p1;
    cout << "p1 use_count: " << p1.use_count() << "     p1 address: " << p1.get() << endl;
    cout << "p2 use_count: " << p2.use_count() << "     p2 address: " << p2.get() << endl;

    p1.reset();
    cout << "p1 use_count: " << p1.use_count() << "     p1 address: " << p1.get() << endl;
    cout << "p2 use_count: " << p2.use_count() << "     p2 address: " << p2.get() << endl;

    cout << "p2 value: " << p2->c_str() << endl;
    return 0;

    // p1 use_count: 1     p1 address: 0x5dbf5afefec0
    // p1 use_count: 2     p1 address: 0x5dbf5afefec0
    // p2 use_count: 2     p2 address: 0x5dbf5afefec0
    // p1 use_count: 0     p1 address: 0
    // p2 use_count: 1     p2 address: 0x5dbf5afefec0
    // p2 value: hello world
}
```

#### 6.thread多线程

```cpp
#include <iostream>
#include <thread>
#include <chrono>
#include <functional>
#include <cpp-httplib/httplib.h>

using namespace std;
using namespace std::placeholders;
class Download
{
public:
    void download(const string &host, const string &path, const function<void(const string &, const string &)> &callback){
        cout << "线程id: " << this_thread::get_id() << endl;
        httplib::Client cli(host);
        auto res = cli.Get(path);
        if (res && res->status == 200) {
            callback(path, res->body);
        }        
    }

    void start_download(const string &host, const string &path, const function<void(const string &, const string &)> &callback){
        //创建一个可调用对象 download_fun，它绑定了当前对象的 download 成员函数，并准备接收三个参数。
        auto download_fun = bind(&Download::download, this, _1, _2, _3);
        //创建一个新的线程 download_thread，并在该线程中执行 download_fun，同时传入 host, path, callback 作为参数
        thread download_thread(download_fun, host, path, callback);
        //download_thread 线程与 thread 对象分离，使下载任务在后台独立运行。
        download_thread.detach();
    }
};

int main()
{
    Download download;
    auto download_finish_callback = [](const string &path, const string &result) -> void{
        cout << "下载完成: " << path << "共： " << result.length() << "字,内容为： " << result.substr(0, 16) << endl;
    };

    download.start_download("http://localhost:8000", "/novel1.txt", download_finish_callback);
    download.start_download("http://localhost:8000", "/novel2.txt", download_finish_callback);
    download.start_download("http://localhost:8000", "/novel3.txt", download_finish_callback);

    //让当前线程休眠10秒钟。在这10秒内，线程不会执行任何操作，只是等待。
    this_thread::sleep_for(chrono::milliseconds(1000 * 10));
    return 0;
}
```

#### 7.launch 

1.启动多节点

```python
import launch
import launch_ros

def generate_launch_description():
	# 创建参数声明 action，用于解析 launch 命令后的参数
    action_declare_arg_max_spped = launch.actions.DeclareLaunchArgument('launch_max_speed', default_value='2.0')

    action_node_turtle_control = launch_ros.actions.Node(
        package='demo_cpp_service',
        executable="turtle_control",
        output='screen',
        
        # 使用launch 中参数 launch_max_speed 值替换节点中的max_speed 参数值
        parameters=[{'max_speed': launch.substitutions.LaunchConfiguration('launch_max_speed', default='2.0')}],
    )
    
    # 单个节点定义
    action_node_patrol_client = launch_ros.actions.Node(
        package='demo_cpp_service',
        executable="patrol_client",
        output='log',
    )
    action_node_turtlesim_node = launch_ros.actions.Node(
        package='turtlesim',
        executable='turtlesim_node',
        output='both',
    )
   # 合成启动描述并返回
    launch_description = launch.LaunchDescription([
        action_declare_arg_max_spped,
        action_node_turtle_control,
        action_node_patrol_client,
        action_node_turtlesim_node
    ])
    return launch_description
```

```shell
source install/setup.bash
ros2 launch demo_cpp_service demo.launch.py
```

```shell
ros2 launch demo_cpp_service demo.launch.py launch_max_speed:=3.0
```

2.其他动作

```python
# 导入launch相关的模块，用于ROS2 launch文件的编写
import launch
import launch.launch_description_sources
import launch_ros
# 用于获取ROS2包的共享目录路径
from ament_index_python.packages import get_package_share_directory
# 运行launch时，添加参数判断参数，控制启动节点
from launch.conditions import IfCondition


def generate_launch_description():

    #包含其他launch 文件    IncludeLaunchDescription 
    action_include_launch = launch.actions.IncludeLaunchDescription(
        launch.launch_description_sources.PythonLaunchDescriptionSource(
            [get_package_share_directory("turtlesim"), "/launch", "/multisim.launch.py"]
            #[功能包名, launch文件所在目录, launch文件名]
        )
    )

    # 执行命令行            ExecuteProcess 在坐标(1,1)处生成一个新的海龟
    action_ececuteprocess = launch.actions.ExecuteProcess(
        cmd= ['ros2', 'service', 'call','/turtlesim1/spawn','turtlesim/srv/Spawn','{x: 1, y: 1}']
    )

    #输出日志信息           LogInfo 
    action_log_info = launch.actions.LogInfo(msg= '使用launch来调用服务生成海龟')

    #动作组合              GroupAction
    #定时器                TimerAction   依次启动日志输出和进程执行
    action_group = launch.actions.GroupAction([
        launch.actions.TimerAction(period=2.0,actions=[action_log_info]),
        launch.actions.TimerAction(period=3.0,actions=[action_ececuteprocess]),])

    #条件判断               IfCondition
    #创建rqt节点，但只在use_rqt为true时执行
    rqt_node = launch_ros.actions.Node(
        package='rqt_gui',
        executable='rqt_gui',
        name='rqt_gui',
        # LaunchConfiguration是ROS 2 launch系统中的一个替代类（substitution），
        # 它的主要功能是在launch执行过程中获取启动参数的值。
        condition=IfCondition(launch.substitutions.LaunchConfiguration('use_rqt'))
    )
    #合成启动描述并返回     LaunchDescription
    launch_descripion = launch.LaunchDescription([
        action_include_launch, 
        action_group,
        rqt_node])
    
    return launch_descripion
```

```shell
ros2 launch demo_cpp_service demo02.launch.py use_rqt:=true
```

#### 8.坐标变换

base_link -----> base_laser -----> wall_point

发布坐标变换话题

```shell
ros2 run tf2_ros static_transform_publisher --x 0.1 --y 0.0 --z 0.2 --roll 0.0 --pitch 0.0 --yaw 0.0 --frame-id base_link --child-frame-id base_lase
```

```shell
ros2 run tf2_ros static_transform_publisher --x 0.3 --y 0.0 --z 0.0 --roll 0.0 --pitch 0.0 --yaw 0.0 --frame-id base_laser --child-frame-id wall_point
```

查看base_link与wall_point之间的坐标关系

```shell
ros2 run tf2_ros tf2_echo base_link wall_point
```

将坐标变化，用图形方式显示

```shell
ros2 run tf2_tools view_frames
```

查看/tf_static 话题内容

```shell
ros2 topic echo /tf_static
```

静态坐标变换

```cpp
#include <memory>  // 提供智能指针等内存管理工具
#include "geometry_msgs/msg/transform_stamped.hpp"  // 提供消息接口
#include "rclcpp/rclcpp.hpp"  // ROS 2 C++客户端库
#include "tf2/LinearMath/Quaternion.h"  // 提供 tf2::Quaternion 类，用于四元数计算
#include "tf2_geometry_msgs/tf2_geometry_msgs.hpp"  // 提供消息类型转换函数
#include "tf2_ros/static_transform_broadcaster.h"  // 提供静态坐标广播器类

/*静态地图发布 map和target_point之间的关系*/

class StaticTFBroadcaster : public rclcpp::Node {
 public:
  StaticTFBroadcaster() : Node("tf_broadcaster_node") {
    // 创建静态广播发发布器并发布
    broadcaster_ = std::make_shared<tf2_ros::StaticTransformBroadcaster>(this);
    this->publish_tf();
  }

  void publish_tf() {
    geometry_msgs::msg::TransformStamped transform;
    transform.header.stamp = this->get_clock()->now();
    transform.header.frame_id = "map";
    transform.child_frame_id = "target_point";
    transform.transform.translation.x = 5.0;
    transform.transform.translation.y = 3.0;
    transform.transform.translation.z = 0.0;
    tf2::Quaternion quat;
    quat.setRPY(0, 0, 60 * M_PI / 180);  // 弧度制欧拉角转四元数
    transform.transform.rotation = tf2::toMsg(quat);  // 转成消息接口类型
    broadcaster_->sendTransform(transform);
  }

 private:
  std::shared_ptr<tf2_ros::StaticTransformBroadcaster> broadcaster_;
};

int main(int argc, char** argv) {
  rclcpp::init(argc, argv);
  auto node = std::make_shared<StaticTFBroadcaster>();
  rclcpp::spin(node);
  rclcpp::shutdown();
  return 0;
}
```

动态坐标变换

```cpp
#include <memory>
#include "geometry_msgs/msg/transform_stamped.hpp" // 提供消息接口
#include "rclcpp/rclcpp.hpp"
#include "tf2/LinearMath/Quaternion.h"             // 提供 tf2::Quaternion 类
#include "tf2_geometry_msgs/tf2_geometry_msgs.hpp" // 提供消息类型转换函数
#include "tf2_ros/transform_broadcaster.h"         // 提供坐标广播器类
#include <chrono>                                  // 引入时间相关头文件
// 使用时间单位的字面量，可以在代码中使用 s 和 ms 表示时间
using namespace std::chrono_literals;

class DynamicTFBroadcaster : public rclcpp::Node
{
public:
  DynamicTFBroadcaster() : Node("dynamic_tf_broadcaster")
  {
    tf_broadcaster_ = std::make_shared<tf2_ros::TransformBroadcaster>(this);
    timer_ = create_wall_timer(10ms, std::bind(&DynamicTFBroadcaster::publishTransform, this));
  }

  void publishTransform()
  {
    geometry_msgs::msg::TransformStamped transform;
    transform.header.stamp = this->get_clock()->now();
    transform.header.frame_id = "map";
    transform.child_frame_id = "base_link";
    transform.transform.translation.x = 2.0;
    transform.transform.translation.y = 3.0;
    transform.transform.translation.z = 0.0;
    tf2::Quaternion quat;
    quat.setRPY(0, 0, 30 * M_PI / 180);              // 弧度制欧拉角转四元数
    transform.transform.rotation = tf2::toMsg(quat); // 转成消息接口类型
    tf_broadcaster_->sendTransform(transform);
  }

private:
  std::shared_ptr<tf2_ros::TransformBroadcaster> tf_broadcaster_;
  rclcpp::TimerBase::SharedPtr timer_;
};

int main(int argc, char **argv)
{
  rclcpp::init(argc, argv);
  auto node = std::make_shared<DynamicTFBroadcaster>();
  rclcpp::spin(node);
  rclcpp::shutdown();
  return 0;
}
```



