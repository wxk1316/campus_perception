# 代码和预处理复核

2026-11-08，CV/IMM更新。52项本机测试通过；实际HARP CPU权重前向、Patchwork++地面处理、旧规划回调仍通过。审查来自新上下文代理，same-family/provisional；不视作实车认证。

## 修复的具体问题

1. 细立柱XY跨度小导致旧规划不可见：使用3D支持跨度并保留最小框，真实Patchwork++回归通过。
2. NaN TF可绕过空目标验证：TF和CloudFrame先检查有限性，无目标也不能跳过。
3. 语义缓存仅XY会跨高度套道路标签：改XYZ近邻和float64 map变换。
4. worker时限固定0.4s与参数不一致：两worker统一配置。
5. HARP非有限输出被当成功：完整logits/shape/confidence检查。
6. CPU柱格上边界算成480：与CUDA一致限制0..479。
7. 推理中纯姿态重定位能发布旧坐标：pipeline和发布前共享SE3跳变检测。
8. IMM漏检预测超过速度上限：预测也限速，并约束每步融合位移。

此外增加DNN/跟踪框栅格占用、有限高度自车过滤、原始sensor_frame限制、TTL关联前删除、启动参数冻结、独立sensor launch及模型NumPy1/2标量提取兼容。

## IMM与可视化验证

IMM交互保持总均值和包含模型间方差的协方差；创新似然及log归一化、Joseph更新、CV/CA模型矩阵、模型切换状态保持、动态确认/静止恢复、交叉和漏检通过。MarkerArray构造测试覆盖框/标签/速度箭头/有限轨迹和独立命名空间删除。

## 未验证边界

当前macOS没有ROS/CUDA，不声明DDS/RViz实际运行、TensorRT构建/GPU推理、Orin并发10Hz或校园精度。未做逐点去畸变；地面为局部单平面；自车高度/雷达外参须实测。概率转移按处理步解释，不能当作每秒概率。
