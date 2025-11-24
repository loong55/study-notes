### 一、控制节点ControlNode

#### 1.sequence  序列节点

![image-20251124215143569](pic_linux/image-20251124215143569.png)

-->框表示次序控制节点，所有旗下的行为依次运行，必须全部成功，才表示成功；当其下的某个行为失败时，终止所有行为。

其中，找球，捡球，放球为**动作节点(ActionNode)**；

#### 2.fallback  回退节点

其下行为依次运行，如果有一个行为成功，则终止其下后续行为；如果全部失败，返回failure;下图扩展了捡球行为

![image-20251125003022578](pic_linux/image-20251125003022578.png)

“ ？”框表示 fallback 回退节点，椭圆框表示**ConditionNode（条件节点）**，

- `BallClose`：检查是否接近球
- `BallGrasped`：检查是否抓住球

#### 3.控制节点例子

![image-20251125000125448](pic_linux/image-20251125000125448.png)

我们现在可以改进“给我拿杯啤酒”的例子，使用颜色“绿色”来表示返回SUCCESS的节点，使用“红色”来表示返回FAILURE的节点。
成功打开冰箱门，然后拿啤酒，因为没有拿到啤酒，强制失败，关门。

### 二、装饰节点

![image-20251125011110669](pic_linux/image-20251125011110669.png)

装饰节点通常用于修改子节点的行为，例如：

- InverterNode：反转子节点的返回值
- RetryNode：重试执行子节点
- TimeoutNode：设置子节点超时

isDoorClose的节点的反转器等价于 门是否打开？ 如何未打开，返回failure,开门最多5次直到成功，否则中断所有行为；如果打开了，先进入房间，再关门。

### 三、代码示例

```cpp
// The simplest callback you can wrap into a BT Action
NodeStatus HelloTick()
{
  std::cout << "Hello World\n"; 
  return NodeStatus::SUCCESS;
}

// Allow the library to create Actions that invoke HelloTick()
// (explained in the tutorials)
factory.registerSimpleAction("Hello", std::bind(HelloTick));
```

`HelloTick`**函数指针**

```
TreeNode
```

- `ActionNodeBase`
- `ConditionNode`
- `DecoratorNode`

```xml
 <root BTCPP_format="4">
     <BehaviorTree ID="MainTree">
        <Sequence name="root_sequence">
            <SaySomething   name="action_hello" message="Hello"/>
            <OpenGripper    name="open_gripper"/>
            <ApproachObject name="approach_object"/>
            <CloseGripper   name="close_gripper"/>
        </Sequence>
     </BehaviorTree>
 </root>
```

- 端口是使用属性配置的。在前面的示例中，操作`SaySomething`需要输入端口`message`。

- 在子节点数量方面：
  - `ControlNodes`包含**1到N个子节点**。
  - `DecoratorNodes`和子树**仅包含1个子节点**。
  - `ActionNodes`和`ConditionNodes`都**没有子节点**。

```xml
 <root BTCPP_format="4" >
     <BehaviorTree ID="MainTree">
        <Sequence name="root_sequence">
            <SaySomething message="Hello"/>
            <SaySomething message="{my_message}"/>
        </Sequence>
     </BehaviorTree>
 </root>
```

- 序列的第一个孩子打印“Hello”，
- 第二个孩子读写黑板条目中包含的值，称为“my_message”；

为了使我们树的紧凑版本与Groot兼容，必须按如下方式修改XML：

```xml
 <root BTCPP_format="4" >
     <BehaviorTree ID="MainTree">
        <Sequence name="root_sequence">
           <SaySomething   name="action_hello" message="Hello"/>
           <OpenGripper    name="open_gripper"/>
           <ApproachObject name="approach_object"/>
           <CloseGripper   name="close_gripper"/>
        </Sequence>
    </BehaviorTree>
    
    <!-- the BT executor don't require this, but Groot does -->     
    <TreeNodeModel>
        <Action ID="SaySomething">
            <input_port name="message" type="std::string" />
        </Action>
        <Action ID="OpenGripper"/>
        <Action ID="ApproachObject"/>
        <Action ID="CloseGripper"/>      
    </TreeNodeModel>
 </root>
```

也可以使用显示语法，与groot兼容

```xml
 <root BTCPP_format="4" >
     <BehaviorTree ID="MainTree">
        <Sequence name="root_sequence">
           <Action ID="SaySomething"   name="action_hello" message="Hello"/>
           <Action ID="OpenGripper"    name="open_gripper"/>
           <Action ID="ApproachObject" name="approach_object"/>
           <Action ID="CloseGripper"   name="close_gripper"/>
        </Sequence>
     </BehaviorTree>
 </root>
```

**子树**

如：将子树GraspObject封装在主树MainTree中

```xml
 <root BTCPP_format="4" >
 
     <BehaviorTree ID="MainTree">
        <Sequence>
           <Action  ID="SaySomething"  message="Hello World"/>
           <!-- 将子树GraspObject封装在主树MainTree中 -->    
           <SubTree ID="GraspObject"/>
           <!-- 将子树GraspObject封装在主树MainTree中 -->   
        </Sequence>
     </BehaviorTree>
     
     <BehaviorTree ID="GraspObject">
        <Sequence>
           <Action ID="OpenGripper"/>
           <Action ID="ApproachObject"/>
           <Action ID="CloseGripper"/>
        </Sequence>
     </BehaviorTree>  
 </root>
```

**包括外部文件**

grasp.xml

```xml
 <!-- file grasp.xml -->

 <root BTCPP_format="4" >
     <BehaviorTree ID="GraspObject">
        <Sequence>
           <Action ID="OpenGripper"/>
           <Action ID="ApproachObject"/>
           <Action ID="CloseGripper"/>
        </Sequence>
     </BehaviorTree>  
 </root>
```

aintree.xml

```xml
 <!-- file maintree.xml -->

 <root BTCPP_format="4" >
     <!-- 将子树文件grasp.xml包括进主树maintree.xml文件中 -->
     <include path="grasp.xml"/>
     
     <BehaviorTree ID="MainTree">
        <Sequence>
           <Action  ID="SaySomething"  message="Hello World"/>
           <!-- 将子树GraspObject封装在主树MainTree中 -->
           <SubTree ID="GraspObject"/>
        </Sequence>
     </BehaviorTree>
  </root>
```

如果要查找[ROS包](http://wiki.ros.org/Packages)中的文件：

```xml
<include ros_pkg="name_package"  path="path_relative_to_pkg/grasp.xml"/>
```

