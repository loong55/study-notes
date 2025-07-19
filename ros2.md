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

