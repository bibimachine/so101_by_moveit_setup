# so101_moveit_setup

SO-101 机械臂的 MoveIt 运动规划配置包,由 **MoveIt Setup Assistant** 自动生成,并附带使用 mock 硬件直接进行运动规划仿真的完整配置。

## 简介

本包为 SO-101(5 轴手臂 + 1 夹爪)提供 MoveIt 所需的全部配置:SRDF(运动学组、夹爪定义)、OMPL 规划配置、控制器配置、关节限位以及演示 launch 文件。机器人模型引自 [`so101_moveit_gazebo`](https://github.com/bibimachine/so101_moveit_gazebo) 包,ros2_control 默认使用 `mock_components/GenericSystem` 假硬件,无需真实机械臂或 Gazebo 即可在 RViz 中做运动规划。

## 依赖

- ROS 2 Humble
- MoveIt 2(Humble 分支,`ws_moveit2` 源码编译版本 2.5.9 验证通过)
- [so101_moveit_gazebo](https://github.com/bibimachine/so101_moveit_gazebo)(提供 `urdf/so101.urdf.xacro`)

## 构建

```bash
cd <你的工作空间根目录>
colcon build
source install/setup.bash
```

## 使用

启动 MoveIt demo(mock 硬件 + RViz):

```bash
ros2 launch so101_moveit_setup demo.launch.py
```

启动后可拖动夹爪目标姿态,在 RViz 中规划并执行;假硬件会把命令回显为关节状态。

其他 launch 文件:

| 文件 | 作用 |
| --- | --- |
| `src/launch/demo.launch.py` | 完整 demo(move_group + RViz + 假硬件) |
| `src/launch/move_group.launch.py` | 仅启动 move_group |
| `src/launch/moveit_rviz.launch.py` | 仅启动 RViz(MoveIt 配置) |
| `src/launch/spawn_controllers.launch.py` | 加载 ros2_controllers 配置的控制器 |
| `src/launch/rsp.launch.py` | robot_state_publisher |
| `src/launch/setup_assistant.launch.py` | 重新打开 MoveIt Setup Assistant 编辑本配置 |

## 目录结构

```
src/
├── config/
│   ├── so101.urdf.xacro          # 顶层 URDF(引入 gazebo 包模型 + ros2_control)
│   ├── so101.ros2_control.xacro  # ros2_control 假硬件定义(Setup Assistant 生成)
│   ├── so101.srdf                # SRDF:arm 运动学组 + hand 夹爪
│   ├── kinematics.yaml           # KDL 运动学求解器
│   ├── joint_limits.yaml         # 关节速度/加速度限位
│   ├── moveit_controllers.yaml   # MoveIt 控制器管理配置
│   ├── ros2_controllers.yaml     # ros2_control 控制器(JTC + GripperAction)
│   ├── initial_positions.yaml    # 假硬件初始关节角
│   ├── pilz_cartesian_limits.yaml
│   └── moveit.rviz
├── launch/                       # 上述 launch 文件
├── package.xml
└── CMakeLists.txt
```

## 已知问题与修改记录

Setup Assistant 生成的原始配置存在以下问题,本仓库已修复:

1. **`ros2_controllers.yaml`**:生成的 `arm_controller` 同时声明了 `position` + `effort` 命令接口,JointTrajectoryController 不允许 effort 与其他接口混用,导致控制器加载失败。已删除 `effort`,只保留 `position`。
2. **`joint_limits.yaml`**:速度限位写成整数(`max_velocity: 10`),MoveIt 读取时要求 double 类型会直接崩溃。已改为 `10.0` / `0.0`。
3. **URDF 中 ros2_control 硬件重复**(WARN,未修复):`config/so101.urdf.xacro` 引入的 gazebo 包模型已自带 `SO101MockSystem`(手臂)+ `SO101HandMockSystem`(夹爪)两个硬件,Setup Assistant 又追加了一个覆盖全部 6 关节的 `FakeSystem`,启动时会打印接口重复的警告。不影响 mock 仿真运行;接真机/Gazebo 前应删除 `config/so101.urdf.xacro` 中多余的 `FakeSystem` 块。

## License

BSD
