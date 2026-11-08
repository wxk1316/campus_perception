# Campus perception

面向RS-Helios-5515 32线、Jetson AGX Orin 64GB和ROS2 Humble的独立激光感知模块。提供CenterPoint base、可选HARP-NeXt道路语义、Patchwork++/几何障碍、CV或CV+CA IMM卡尔曼滤波、匈牙利匹配，以及RViz MarkerArray。

## 仓库内容

- `src/campus_lidar_perception`：完整ROS2/Python源码、native CUDA/TensorRT10、模型配置、下载/提取/验证脚本、RViz、测试。
- `integration/car_launch`：全链路、单独检测、单独定位和定位+检测入口，合并到现有car_launch即可；不覆盖旧launch。
- `interfaces/rda_dwa_real_msgs`：兼容消息定义参考，已安装该消息包时无需重复建包。此目录默认COLCON_IGNORE。
- 模型下载固定官方revision；本地已下载权重保留，仓库通过脚本重建，不提交运行记录/venv/二进制engine。

## 两个独立开关

`profile=centerpoint_geometry`或`centerpoint_harpnext`选择几何/道路语义组合。`tracking_filter=cv`或`imm`选择动态跟踪。静止/未确认对象内部保持CV，确认运动后启用所选滤波器。

```bash
ros2 launch campus_lidar_perception perception.launch.py \
  profile:=centerpoint_harpnext tracking_filter:=imm
```

规划消息仍为map系`/rda/obstacles` ObstacleArray，rich状态发布到`/campus_perception/tracks`，栅格到`/rda/drivable_area`。MarkerArray在`/rda/obstacles_markers`显示框、语义/速度/滤波器及概率文字、速度箭头和历史轨迹。现有规划继续消费障碍消息，尚不直接消费语义栅格。

## 部署

将`src/campus_lidar_perception`放入既有ROS工作空间，保持现有消息、定位、规划、控制及驱动包。需要新消息包时，复制interfaces里的rda_dwa_real_msgs到工作空间src目录；不要与已有包重复。将integration中的4个新launch合并到既有car_launch/launch，package.xml添加`<exec_depend>campus_lidar_perception</exec_depend>`。

下载及准备模型（需Python、NumPy、ONNX和支持safe_globals的PyTorch）：

```bash
python3 src/campus_lidar_perception/scripts/download_models.py
python3 src/campus_lidar_perception/scripts/prepare_models.py
```

Orin构建与GPU验证、全部参数、录像比较步骤见[模块部署文档](src/campus_lidar_perception/README.md)。使用车机对应的NVIDIA CUDA版PyTorch；不得用CPU wheel代替GPU环境。首次运行在Orin生成engine，实际地图/外参使用原车数据。

## 验证边界

本机macOS完成CPU数学、真实HARP CPU前向、真实Patchwork++、消息/预处理/launch结构测试；ROS/DDS、CUDA编译、Orin并行时延及校园精度需目标车机验收。合成轨迹只用于逻辑验证，不作为实车结果。源码保持旧检测/定位/规划接口。

许可证Apache-2.0；第三方来源见[THIRD_PARTY](src/campus_lidar_perception/THIRD_PARTY.md)。
