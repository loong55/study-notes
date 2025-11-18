### 前言

本系列博文是本人的学习笔记，自用为主，不是教程，学习请移步其他大佬的相关教程。前几篇学习资源来自鱼香ROS大佬的详细教程，适合深入学习，但对本人这样的初学者不算友好，后续笔记将以@[古月居](https://so.csdn.net/so/search?q=%E5%8F%A4%E6%9C%88%E5%B1%85&spm=1001.2101.3001.7020)的ROS2入门21讲为主，侵权即删。

### 一、学习目标

1.  理解 Launch 文件的核心价值 —— 解决 “多节点需多终端启动” 的痛点
2.  掌握 Launch 文件的固定结构（Python 风格），能独立编写基础启动脚本
3.  学会 5 大核心功能：多节点启动、命令行参数传递、资源重映射、参数配置、Launch 包含
4.  区分容易混淆的概念（`argument` vs `parameter`），避免踩坑
5.  知道如何配置功能包编译，确保 Launch 文件能被 ROS 识别

### 二、先搞懂：为什么需要 Launch 文件？（痛点解决）

之前启动 [ROS 节点](https://so.csdn.net/so/search?q=ROS%20%E8%8A%82%E7%82%B9&spm=1001.2101.3001.7020)的方式：

-   启动 1 个节点 → 打开 1 个终端，输入 1 条命令；
-   启动 5 个节点 → 打开 5 个终端，输入 5 条命令；
-   机器人系统可能有十几个节点（相机、电机、视觉、导航），手动启动效率极低，还容易漏命令。

**Launch 文件的作用**：用 1 个 Python 脚本，一次性启动多个节点，并配置节点参数、话题名、文件路径等，相当于 “节点启动的总指挥”。

### 三、Launch 文件基础：固定结构（小白必背）

ROS2 的 Launch 文件是**Python 风格**的脚本，所有 Launch 文件都遵循以下固定结构，记熟这个 “模板”，后续所有案例都是在此基础上扩展：

```python
# 1. 导入必需的类（固定开头，类似“工具箱”）
from launch import LaunchDescription  # 描述Launch文件的“容器”，所有节点/参数都放这里面
from launch_ros.actions import Node   # 描述“节点”的类，每个Node对应一个要启动的节点
 
# 2. 定义生成Launch描述的函数（固定函数名，ROS会自动调用）
def generate_launch_description():
    # 3. 创建节点对象（每个节点对应一个Node()）
    node1 = Node(
        package="功能包名",    # 节点所在的功能包（必须正确，否则找不到节点）
        executable="节点命令", # 启动节点的命令（比如ros2 run 功能包 命令 中的“命令”）
        name="节点别名",       # 可选：给节点起个别名，避免重名（默认用executable名）
        # 后续扩展：参数、重映射、命令行参数等都加在这里
    )
 
    # 4. 将所有节点放入LaunchDescription容器
    return LaunchDescription([
        node1,  # 把节点1加入容器
        # 其他节点（node2, node3...）也加在这里
    ])
```

**核心规律**：

-   所有要启动的节点，都用`Node()`类描述；
-   所有节点 / 参数 / 动作，最终都要放入`LaunchDescription`容器中返回。

### 四、实战案例：从基础到进阶（带详细注释）

按 “难度递增” 排序，每个案例都包含 “场景需求→运行效果→带注释代码→核心知识点”。

#### 案例 1：最基础 —— 多节点启动（Hello World 发布 + 订阅）

##### 场景需求

用 1 个 Launch 文件，同时启动 “Hello World 发布者” 和 “Hello World 订阅者” 节点（来自之前的`learning_topic`功能包）。

##### 运行效果

1.  终端输入命令：`ros2 launch learning_launch simple.launch.py`
2.  终端会同时打印发布者和订阅者的日志，不用再开两个终端。

##### 带注释代码（`learning_launch/simple.launch.py`）

```python
# 导入必需的类（固定开头）
from launch import LaunchDescription  # Launch描述容器
from launch_ros.actions import Node   # 节点描述类
 
def generate_launch_description():
    # 1. 配置“Hello World发布者”节点
    pub_node = Node(
        package='learning_topic',          # 节点所在功能包：learning_topic
        executable='topic_helloworld_pub', # 节点命令：topic_helloworld_pub（对应ros2 run learning_topic topic_helloworld_pub）
        name='helloworld_pub'              # 可选：给节点起别名（避免和其他节点重名）
    )
 
    # 2. 配置“Hello World订阅者”节点
    sub_node = Node(
        package='learning_topic',          # 节点所在功能包：learning_topic
        executable='topic_helloworld_sub', # 节点命令：topic_helloworld_sub
        name='helloworld_sub'              # 可选：节点别名
    )
 
    # 3. 将两个节点放入Launch容器，返回给ROS
    return LaunchDescription([
        pub_node,  # 加入发布者节点
        sub_node   # 加入订阅者节点
    ])
```

##### 核心知识点

-   多节点启动的本质：在`LaunchDescription`中放入多个`Node()`对象；
-   必须确保`package`和`executable`正确（和`ros2 run`命令中的一致），否则会报错 “找不到节点”。

#### 案例 2：传递命令行参数（RViz 加载配置文件）

##### 场景需求

启动 RViz 时，需要加载自定义配置文件（比如小海龟的 RViz 配置）。命令行方式是：`ros2 run rviz2 rviz2 -d /path/to/turtle_rviz.rviz`用 Launch 文件自动处理路径，避免手动输入长路径。

##### 运行效果

1.  终端输入命令：`ros2 launch learning_launch rviz.launch.py`
2.  RViz 自动启动，并加载指定的 `turtle_rviz.rviz`配置文件（不用手动选配置）。

##### 带注释代码（`learning_launch/rviz.launch.py`）

```python
# 导入额外的库：处理文件路径
import os
# 导入查询功能包路径的工具（关键：自动找到功能包下的配置文件）
from ament_index_python.packages import get_package_share_directory
 
# 导入Launch基础类
from launch import LaunchDescription
from launch_ros.actions import Node
 
def generate_launch_description():
    # 1. 自动找到“learning_launch”功能包的安装路径
    # 作用：不管功能包装在电脑哪个位置，都能找到配置文件（避免硬编码路径）
    pkg_path = get_package_share_directory('learning_launch')
    
    # 2. 拼接配置文件的完整路径
    # 配置文件位置：learning_launch功能包下的“rviz”文件夹→“turtle_rviz.rviz”
    rviz_config_path = os.path.join(
        pkg_path,          # 功能包路径
        'rviz',            # 配置文件所在文件夹
        'turtle_rviz.rviz' # 配置文件名
    )
 
    # 3. 配置RViz节点，传递命令行参数
    rviz_node = Node(
        package='rviz2',       # 节点所在功能包：rviz2
        executable='rviz2',    # 节点命令：rviz2
        name='rviz2',          # 节点别名
        # 关键：传递命令行参数（对应命令中的“-d 配置路径”）
        arguments=['-d', rviz_config_path]  # arguments是列表，每个元素对应命令行的一个部分
    )
 
    # 4. 返回Launch容器
    return LaunchDescription([
        rviz_node
    ])
```

##### 核心知识点

-   命令行参数用`arguments`传递：格式是列表，比如`['-d', 路径]`对应命令行的`-d 路径`；
-   用`get_package_share_directory`+`os.path.join`自动找路径，避免写死路径（比如`/home/user/dev_ws/install/...`），代码更通用。

#### 案例 3：资源重映射（让两个小海龟联动）

##### 场景需求

小海龟的`mimic`节点功能：订阅一个海龟的`pose`话题，生成相同的`cmd_vel`话题，让另一个海龟模仿运动。需要将`mimic`的默认话题（`/input/pose`、`/output/cmd_vel`）重映射为两个海龟的话题，实现 “海龟 1 动，海龟 2 跟着动”。

##### 运行效果

1.  终端输入命令：`ros2 launch learning_launch remapping.launch.py`
2.  弹出两个小海龟窗口；
3.  新终端输入：`ros2 topic pub --rate 1 /turtlesim1/turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0}, angular: {z: 1.8}}"`
4.  两个海龟同时转圈运动。

##### 带注释代码（`learning_launch/remapping.launch.py`）

```python
# 导入基础类
from launch import LaunchDescription
from launch_ros.actions import Node
 
def generate_launch_description():
    # 1. 启动第一个小海龟（命名空间：turtlesim1）
    turtle1_node = Node(
        package='turtlesim',          # 小海龟功能包
        executable='turtlesim_node',  # 小海龟仿真器命令
        namespace='turtlesim1',       # 关键：命名空间（给节点分组，避免两个海龟节点重名）
        name='sim'                    # 节点别名（在命名空间下，名字是/turtlesim1/sim）
    )
 
    # 2. 启动第二个小海龟（命名空间：turtlesim2）
    turtle2_node = Node(
        package='turtlesim',
        executable='turtlesim_node',
        namespace='turtlesim2',       # 第二个命名空间，和第一个区分
        name='sim'
    )
 
    # 3. 启动mimic节点（实现模仿功能，核心是话题重映射）
    mimic_node = Node(
        package='turtlesim',
        executable='mimic',           # mimic节点命令
        name='mimic',
        # 关键：资源重映射（将默认话题改成两个海龟的话题）
        remappings=[
            # 格式：('原话题名', '目标话题名')
            ('/input/pose', '/turtlesim1/turtle1/pose'),  # mimic订阅海龟1的pose
            ('/output/cmd_vel', '/turtlesim2/turtle1/cmd_vel')  # mimic发布命令给海龟2
        ]
    )
 
    # 4. 返回所有节点
    return LaunchDescription([
        turtle1_node,
        turtle2_node,
        mimic_node
    ])
```

##### 核心知识点

-   资源重映射用`remappings`：格式是列表，每个元素是元组`(原名, 目标名)`，作用是 “将节点的默认话题 / 服务名改成自定义的”；
-   命名空间`namespace`：给节点分组，避免重名（比如两个 `turtlesim_node`节点，用 `turtlesim1`和 `turtlesim2`分组后，不会冲突）。

#### 案例 4：ROS 参数设置（修改小海龟背景色）

##### 场景需求

启动小海龟时，直接在 Launch 文件中设置背景色参数（`background_r`、`background_g`、`background_b`），不用启动后再用`ros2 param set`修改。

##### 运行效果

1.  终端输入命令：`ros2 launch learning_launch parameters.launch.py`
2.  小海龟窗口背景色变成 “深青色”（不是默认的蓝色），参数已在 Launch 中生效。

##### 带注释代码（`learning_launch/parameters.launch.py`）

```python
# 导入基础类+参数相关类
from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument  # 声明Launch内部参数（arg）
from launch.substitutions import LaunchConfiguration, TextSubstitution  # 引用arg的工具
from launch_ros.actions import Node  # ROS节点类
 
def generate_launch_description():
    # 1. 声明Launch内部参数（argument，简称arg）
    # 作用：仅限Launch文件内部使用，可设置默认值，方便修改（类似函数的形参）
    declare_bg_r_arg = DeclareLaunchArgument(
        'background_r',  # arg的名字（后续引用时用这个名）
        default_value=TextSubstitution(text='0')  # 默认值：红色通道0（0-255）
    )
    declare_bg_g_arg = DeclareLaunchArgument(
        'background_g',
        default_value=TextSubstitution(text='84')  # 绿色通道84
    )
    declare_bg_b_arg = DeclareLaunchArgument(
        'background_b',
        default_value=TextSubstitution(text='122') # 蓝色通道122
    )
 
    # 2. 配置小海龟节点，设置ROS参数（parameter）
    turtle_node = Node(
        package='turtlesim',
        executable='turtlesim_node',
        name='sim',
        # 关键：设置ROS系统参数（parameter，简称param）
        parameters=[{
            # 格式：'参数名': 参数值（这里引用上面声明的arg）
            'background_r': LaunchConfiguration('background_r'),  # 引用arg：background_r
            'background_g': LaunchConfiguration('background_g'),  # 引用arg：background_g
            'background_b': LaunchConfiguration('background_b')   # 引用arg：background_b
        }]
    )
 
    # 3. 返回Launch容器（注意：要先加入arg的声明，再加入节点）
    return LaunchDescription([
        declare_bg_r_arg,  # 先声明arg
        declare_bg_g_arg,
        declare_bg_b_arg,
        turtle_node        # 再启动节点（节点要引用arg，所以arg必须先声明）
    ])
```

##### 关键：区分`argument`和`parameter`（小白必看）

很多初学者混淆这两个 “参数”，用表格彻底分清：

| 特性     | `argument`（Launch 内部参数）                 | `parameter`（ROS 系统参数）              |
| -------- | --------------------------------------------- | ---------------------------------------- |
| 作用范围 | 仅限当前 Launch 文件内部使用                  | 整个 ROS 系统可见，所有节点都能访问      |
| 定义方式 | 用`DeclareLaunchArgument()`声明               | 用 Node () 中的`parameters`配置          |
| 引用方式 | 用`LaunchConfiguration('arg名')`引用          | 节点中用`get_parameter('param名')`读取   |
| 类比     | 函数的 “形参”（只在函数内部用）               | 全局变量（整个程序都能用）               |
| 例子     | `background_r`（Launch 中临时存值，方便修改） | `turtlesim.background_r`（ROS 系统参数） |

#### 案例 5：加载 YAML 参数文件（批量设置参数）

##### 场景需求

如果有几十个 ROS 参数（比如导航参数、视觉参数），在 Launch 文件中逐个写`parameters`会很繁琐。此时可以用**YAML 文件批量存储参数**，再在 Launch 中加载整个文件。

##### 步骤 1：创建 YAML 参数文件（`learning_launch/config/turtlesim.yaml`）

先在`learning_launch`功能包下创建`config`文件夹，新建 `turtlesim.yaml`，内容如下：

```python
# YAML格式：键值对，对应ROS参数
turtlesim2:  # 命名空间（对应节点的namespace）
  sim:        # 节点名（对应节点的name）
    ros__parameters:  # 固定格式，ROS识别参数的关键字
      background_r: 255  # 红色通道255（白色）
      background_g: 255  # 绿色通道255
      background_b: 255  # 蓝色通道255
```

##### 步骤 2：Launch 文件加载 YAML（`learning_launch/parameters_yaml.launch.py`）

```python
# 导入处理路径的库
import os
from ament_index_python.packages import get_package_share_directory
 
# 导入Launch基础类
from launch import LaunchDescription
from launch_ros.actions import Node
 
def generate_launch_description():
    # 1. 自动找到YAML参数文件的路径
    pkg_path = get_package_share_directory('learning_launch')
    yaml_config_path = os.path.join(
        pkg_path,
        'config',          # YAML文件所在文件夹
        'turtlesim.yaml'   # YAML文件名
    )
 
    # 2. 配置小海龟节点，加载YAML参数文件
    turtle_node = Node(
        package='turtlesim',
        executable='turtlesim_node',
        namespace='turtlesim2',  # 必须和YAML中的命名空间一致！
        name='sim',              # 必须和YAML中的节点名一致！
        parameters=[yaml_config_path]  # 直接传入YAML文件路径，批量加载参数
    )
 
    # 3. 返回Launch容器
    return LaunchDescription([
        turtle_node
    ])
```

##### 运行效果

1.  终端输入命令：`ros2 launch learning_launch parameters_yaml.launch.py`
2.  小海龟窗口背景色变成白色（YAML 中设置的`background_r/g/b=255`）。

##### 核心知识点

-   YAML 文件的格式必须正确：`命名空间→节点名→ros__parameters→参数键值对`；
-   节点的`namespace`和`name`必须和 YAML 中的一致，否则参数无法匹配到节点；
-   适合参数数量多的场景（比如导航、SLAM），批量管理更清晰。

#### 案例 6：Launch 文件包含（复杂系统拆分）

##### 场景需求

大型机器人系统可能有多个 Launch 文件（比如 “相机启动.launch.py”“导航启动.launch.py”“视觉启动.launch.py”），可以用 “包含” 机制，在 1 个 “总 Launch” 中调用其他 Launch，实现 “一键启动整个系统”。

```python
# robot_bringup.launch.py
from launch import LaunchDescription
from launch.actions import IncludeLaunchDescription
from launch.launch_description_sources import PythonLaunchDescriptionSource
from ament_index_python.packages import get_package_share_directory
import os

def generate_launch_description():
    pkg_share = get_package_share_directory('my_robot_bringup')

    perception_launch = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(os.path.join(pkg_share, 'launch', 'perception.launch.py'))
    )
    
    localization_launch = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(os.path.join(pkg_share, 'launch', 'localization.launch.py'))
    )

    navigation_launch = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(os.path.join(pkg_share, 'launch', 'navigation.launch.py'))
    )

    return LaunchDescription([
        perception_launch,
        localization_launch,
        navigation_launch,
    ])

```



##### 带注释代码（`learning_launch/namespaces.launch.py`）

```python
# 导入处理路径和包含的库
import os
from ament_index_python.packages import get_package_share_directory
 
# 导入包含Launch所需的类
from launch import LaunchDescription
from launch.actions import IncludeLaunchDescription  # 包含其他Launch的类
from launch.launch_description_sources import PythonLaunchDescriptionSource  # 指定Launch文件类型（Python）
from launch.actions import GroupAction  # 给包含的Launch加“组”（比如统一命名空间）
from launch_ros.actions import PushRosNamespace  # 给组内节点统一加命名空间
 
def generate_launch_description():
    # 1. 找到要包含的Launch文件路径（比如之前的parameters_nonamespace.launch.py）
    pkg_path = get_package_share_directory('learning_launch')
    included_launch_path = os.path.join(
        pkg_path,
        'launch',  # 被包含的Launch文件所在文件夹
        'parameters_nonamespace.launch.py'  # 被包含的Launch文件名
    )
 
    # 2. 配置“包含的Launch”，并统一加命名空间
    included_launch = GroupAction(
        actions=[
            # 第一步：给组内所有节点加命名空间“turtlesim2”
            PushRosNamespace('turtlesim2'),
            # 第二步：包含目标Launch文件
            IncludeLaunchDescription(
                PythonLaunchDescriptionSource(included_launch_path)  # 指定要包含的Launch文件
            )
        ]
    )
 
    # 3. 返回总Launch容器
    return LaunchDescription([
        included_launch  # 加入包含的Launch
    ])
```

##### 核心知识点

-   包含 Launch 用`IncludeLaunchDescription`，必须指定`PythonLaunchDescriptionSource`（因为 ROS2 Launch 是 Python 脚本）；
-   `GroupAction`+`PushRosNamespace`：给包含的所有节点统一加命名空间，避免节点重名；
-   适合大型系统：将功能拆分到不同 Launch（相机、导航、视觉），总 Launch 负责 “汇总调用”，代码更易维护。

### 五、关键步骤：功能包编译配置（让 ROS 找到 Launch 文件）

写好的 Launch 文件和配置文件（RViz、YAML），需要在`setup.py`中配置 “数据文件路径”，否则 ROS 编译后找不到这些文件，运行时会报错。

#### 配置`setup.py`中的`data_files`

打开`learning_launch/setup.py`，找到`data_files`字段，添加以下内容（确保 Launch、RViz、YAML 文件被识别）：

```python
import os
from glob import glob  # 用于批量匹配文件
 
# ... 其他代码 ...
 
setup(
    # ... 其他配置 ...
 
    # 关键：配置数据文件路径（Launch、RViz、YAML都在这里）
    data_files=[
        # 1. 固定配置：ROS包索引（不用改）
        ('share/ament_index/resource_index/packages',
            ['resource/' + package_name]),
        # 2. 固定配置：package.xml（不用改）
        ('share/' + package_name, ['package.xml']),
        # 3. 配置Launch文件路径：让ROS找到所有.launch.py文件
        (os.path.join('share', package_name, 'launch'),  # 目标路径（编译后存放Launch的位置）
         glob(os.path.join('launch', '*.launch.py'))),    # 源路径（当前Launch文件的位置，*.launch.py匹配所有Launch文件）
        # 4. 配置RViz文件路径：让ROS找到rviz文件夹下的所有文件
        (os.path.join('share', package_name, 'rviz'),
         glob(os.path.join('rviz', '*.*'))),
        # 5. 配置YAML参数文件路径：让ROS找到config文件夹下的所有文件
        (os.path.join('share', package_name, 'config'),
         glob(os.path.join('config', '*.*'))),
    ],
 
    # ... 其他配置 ...
)
```

#### 编译与生效

配置完成后，在工作空间根目录编译：

```bash
cd dev_ws
colcon build --packages-select learning_launch  # 只编译learning_launch功能包
source install/setup.bash  # 加载环境变量，让ROS识别新配置的文件
```

### 六、复习要点总结（小白必背）

1.  **Launch 文件固定结构**：`from launch import LaunchDescription` + `from launch_ros.actions import Node` + `def generate_launch_description(): return LaunchDescription([节点列表])`
    
2.  **5 大核心功能实现方式**：
    
    -   多节点启动：在`LaunchDescription`中放多个`Node()`；
    -   命令行参数：`Node(arguments=[参数列表])`；
    -   资源重映射：`Node(remappings=[(原名, 目标名)])`；
    -   参数设置：`Node(parameters=[{键值对}]`（单个）或`parameters=[YAML路径]`（批量）；
    -   Launch 包含：`IncludeLaunchDescription(PythonLaunchDescriptionSource(路径))`。
3.  **易混淆概念**：
    
    -   `argument`：Launch 内部参数，用`DeclareLaunchArgument`声明，`LaunchConfiguration`引用；
    -   `parameter`：ROS 系统参数，用`Node(parameters=...)`配置，节点内`get_parameter`读取。
4.  **编译配置关键**：在`setup.py`的`data_files`中配置 Launch、RViz、YAML 的路径，否则 ROS 找不到文件。
    

掌握这些内容，就能应对 90% 以上的 ROS 节点启动场景，后续复杂系统（如导航、SLAM）的 Launch 文件也只是在此基础上扩展