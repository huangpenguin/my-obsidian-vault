基于客户现在已经明确的要求，我会把系统定成：

> **VLM → 3D Geometry → Key Pose Generator → Planner Backend（MoveIt / cuRobo）→ FD-ST Adapter → FD-ST验证 → DAIHEN实机**

而且有一个很重要的调整：

> **MoveIt 和 cuRobo 不是串联关系，而是两个可替换的 Planning Backend。**
> 
> ① Phase 1 主线优先 MoveIt；只有做到③、需要更快的 runtime planning 时，再考虑 cuRobo。

客户①本身只要求生成约 **1～5 个 TCP 6D Pose**，考虑可达性和障碍物，并且不要求实时；Lv2/Lv3才增加轨迹种类和速度，因此并不需要先训练 VLA。

---

# 1. 我建议的完整系统架构

```
                             ┌─────────────────────────┐
                             │       User Input        │
                             │ 日本語自然言語指示       │
                             └────────────┬────────────┘
                                          │
                                          ▼
┌────────────────────────────────────────────────────────────────────┐
│ ① Scene Input Layer                                                │
│                                                                    │
│  Mesh / CAD / RGB-D / Point Cloud                                  │
│  Robot URDF / Tool Model / Camera parameters                       │
│                                                                    │
│  Optional: Atlas / 3DGS → Mesh / Render Views                      │
└───────────────────────────────┬────────────────────────────────────┘
                                │
                                ▼
┌────────────────────────────────────────────────────────────────────┐
│ ② Scene Normalization                                              │
│                                                                    │
│ Open3D / trimesh                                                   │
│                                                                    │
│ - 单位全部转成 meter                                               │
│ - 坐标统一 robot_base                                              │
│ - Mesh加载                                                         │
│ - Camera intrinsic / extrinsic                                     │
│ - Render multi-view RGB / Depth / Object ID                        │
└───────────────────────────────┬────────────────────────────────────┘
                                │
                RGB Views       │       Geometry
                  ┌─────────────┴──────────────┐
                  ▼                            ▼
┌──────────────────────────┐       ┌────────────────────────────┐
│ ③ VLM Semantic Grounding │       │ ④ 3D Geometry Engine       │
│                          │       │                            │
│ VLM / optional SAM       │       │ Open3D                    │
│                          │       │ raycast / normal / depth   │
│ 找：                      │       │                            │
│ - 哪个物体                │       │ 算：                       │
│ - 哪个按钮                │       │ - XYZ                      │
│ - 哪个区域                │       │ - Surface normal           │
│ - 什么Action              │       │ - direction                │
└─────────────┬────────────┘       └────────────┬───────────────┘
              │                                  │
              └────────────────┬─────────────────┘
                               ▼
┌────────────────────────────────────────────────────────────────────┐
│ ⑤ Affordance / Key Pose Generator                                 │
│                                                                    │
│ Task Adapter                                                       │
│                                                                    │
│ PICK       → pregrasp / grasp / retreat                            │
│ PUSH       → approach / contact / push / retreat                   │
│ OPEN_DOOR  → approach / contact / move                             │
│                                                                    │
│ Output: 1～5 TCP 6D Pose                                           │
└───────────────────────────────┬────────────────────────────────────┘
                                │
                                ▼
┌────────────────────────────────────────────────────────────────────┐
│ ⑥ Robot Planning Layer                                             │
│                                                                    │
│              Planner Interface                                     │
│               ┌───────┴───────┐                                   │
│               ▼               ▼                                   │
│            MoveIt 2          cuRobo                                │
│            主线              Optional / ③                          │
│                                                                    │
│ IK / Reachability / Joint Limit / Self Collision / World Collision │
│ Trajectory Generation                                              │
└───────────────────────────────┬────────────────────────────────────┘
                                │
                                ▼
┌────────────────────────────────────────────────────────────────────┐
│ ⑦ Generic Validation                                               │
│                                                                    │
│ - pose reachable?                                                  │
│ - collision free?                                                  │
│ - joint limits?                                                    │
│ - trajectory valid?                                                │
│ - clearance                                                        │
└───────────────────────────────┬────────────────────────────────────┘
                                │
                                ▼
┌────────────────────────────────────────────────────────────────────┐
│ ⑧ DAIHEN Adapter                                                   │
│                                                                    │
│ Internal JSON / Pose Sequence                                      │
│         ↓                                                          │
│ DAIHEN coordinate convention                                      │
│ m → mm                                                             │
│ quaternion → DAIHEN orientation representation                     │
│         ↓                                                          │
│ FD-ST API / File Import / Robot Program    ← 客户确认后实现         │
└───────────────────────────────┬────────────────────────────────────┘
                                │
                                ▼
                      ┌─────────────────┐
                      │ ⑨ FD-ST         │
                      │                 │
                      │ DAIHEN-specific │
                      │ simulation      │
                      │ interference    │
                      │ tact check      │
                      └────────┬────────┘
                               │ Human Approval
                               ▼
                      ┌─────────────────┐
                      │ DAIHEN Controller│
                      │      ↓          │
                      │   Real Robot    │
                      └─────────────────┘
```

FD-ST 官方定位就是 DAIHEN 的 offline teaching / simulation 软件，与其 controller 高度兼容，还能在 3D model 上 snap、自动生成部分工艺路径和较高精度的 tact simulation，因此比较适合作为最后的 **DAIHEN-specific validator**，而不是 AI 算法开发环境。

---

# 2. 第一层：3D Scene 输入

客户现在提出的候选包括：

- 无标签 Mesh；
- 无标签 3DGS；
- 用户给部分 Mesh/GS 加标签作为 fallback。

我建议内部统一成一个 `SceneManifest`。

## 推荐目录

```
scene_001/
├── scene.yaml
├── meshes/
│   ├── machine.glb
│   ├── table.glb
│   ├── workpiece_01.glb
│   └── robot_cell.glb
├── collision/
│   ├── machine.stl
│   └── table.stl
├── cameras/
│   ├── cam01.yaml
│   ├── cam02.yaml
│   └── cam03.yaml
└── images/
    ├── cam01.png
    ├── cam02.png
    └── cam03.png
```

### `scene.yaml`

建议自己定义：

```
frame_id: robot_base
unit: meter

objects:
  - id: machine_01
    visual_mesh: meshes/machine.glb
    collision_mesh: collision/machine.stl
    pose:
      xyz: [0.8, 0.2, 0.0]
      quaternion_xyzw: [0, 0, 0, 1]

  - id: work_01
    visual_mesh: meshes/workpiece_01.glb
    pose:
      xyz: [0.45, -0.15, 0.82]
      quaternion_xyzw: [0, 0, 0, 1]
```

### 格式建议

|内容|推荐格式|
|---|---|
|Visual Mesh|`.glb` / `.obj`|
|Collision Mesh|`.stl` / `.obj`|
|Point Cloud|`.ply` / `.pcd`|
|RGB|`.png`|
|Depth|`float32 .npy` 或 16-bit PNG|
|Scene metadata|YAML|
|Transform|4×4 matrix / xyz+quaternion|

### 内部统一原则

**全部：**

```
长度 → meter
角度 → radian
姿态 → quaternion
坐标 → robot_base
```

只有到 FD-ST Adapter 时才转换成：

```
mm
degree
DAIHEN orientation convention
```

这样能避免最常见的工程事故。

---

# 3. Robot 数据

这里要求客户给你：

```
robot URDF / CAD
joint limits
TCP
tool geometry
controller information
```

MoveIt本身就是以 **URDF + SRDF** 为机器人描述基础，SRDF还负责 planning group、end-effector、自碰撞配置等。MoveIt Setup Assistant 可以从 URDF 生成这套配置。

推荐：

```
daihen_robot_description/
├── urdf/
│   └── robot.urdf.xacro
├── meshes/
│   └── ...
└── config/

daihen_robot_moveit_config/
├── config/
│   ├── robot.srdf
│   ├── kinematics.yaml
│   ├── joint_limits.yaml
│   ├── ompl_planning.yaml
│   └── pilz_cartesian_limits.yaml
└── launch/
```

MoveIt官方配置包本身也采用这种结构。

---

# 4. Scene Renderer：先把 3D Scene 转成 VLM 能看的数据

这一层我认为特别重要。

普通 VLM 不直接看 STL：

```
Mesh
 ↓
Multi-view Renderer
 ↓
RGB Views
```

例如：

```
front_left.png
front_right.png
top.png
side.png
```

同时保存每张图：

```
{
  "view_id": "cam_01",
  "width": 1280,
  "height": 720,

  "K": [
    [900, 0, 640],
    [0, 900, 360],
    [0, 0, 1]
  ],

  "T_robot_base_camera": [
    [1,0,0,0.5],
    [0,1,0,0.2],
    [0,0,1,1.2],
    [0,0,0,1]
  ]
}
```

## 包

### `open3d`

主力。

它现在的 `RaycastingScene` 可以：

- mesh ray intersection；
- 找 hit triangle；
- 找 geometry ID；
- 返回 triangle normal；
- 根据 camera intrinsic/extrinsic 创建 rays。

### `trimesh`

建议辅助：

```
mesh loading
mesh conversion
simplification
transform
bounding box
```

### `numpy/scipy`

矩阵与 SE(3) 处理。

---

# 5. VLM 层到底输出什么？

**不要输出机器人 Pose。**

VLM应该只输出：

> “我要操作哪个东西、哪个区域、做什么。”

例如输入：

```
Instruction:
「左側にある加工機の実行ボタンを押す」

Images:
cam01.png
cam02.png
cam03.png
```

输出：

```
{
  "action": "push",
  "target": {
    "view_id": "cam_02",
    "description": "加工機操作パネル上の緑色の実行ボタン",
    "bbox_normalized": [0.42, 0.31, 0.48, 0.39],
    "point_normalized": [0.451, 0.352]
  },
  "confidence": 0.88
}
```

---

# 6. 推荐定义一个严格 Schema

用：

```
pydantic
jsonschema
```

比如：

```
class SemanticTarget:
    action: Literal["pick", "push", "open"]
    view_id: str
    point_uv: tuple[float, float]
    bbox: tuple[float, float, float, float] | None
    object_hint: str
    confidence: float
```

这样可以强制 VLM：

```
VLM自然语言
↓
validated JSON
↓
Geometry
```

而不是让后面的机器人代码解析自然语言。

---

# 7. 是否需要 SAM / Segmentation？

不是第一天必须。

最小版本：

```
VLM
→ 一个pixel
```

就能做。

复杂一点：

```
VLM
↓
target bbox
↓
SAM / segmentation
↓
mask
```

然后可以从一整块 mask 获取：

```
3D point cloud
surface
centroid
normal
```

对于按钮之类，会比单 pixel 稳。

所以这是：

```
Phase 1 baseline:
VLM point / bbox

Phase 1.5:
VLM + segmentation
```

---

# 8. 2D → 3D Geometry

这里是系统里最重要的确定性部分。

例如 VLM 给：

```
cam02
pixel = (632, 351)
```

已知：

```
Camera K
Camera Pose
Mesh
```

Open3D：

```
pixel
 ↓
camera ray
 ↓
RaycastingScene
 ↓
hit triangle
 ↓
3D XYZ
+
primitive normal
+
geometry ID
```

Open3D 官方 `RaycastingScene` 正好会返回 `t_hit`、`geometry_ids`、`primitive_ids` 和 triangle normal。

最后得到：

```
{
  "frame_id": "robot_base",
  "object_id": "machine_01",
  "point_m": [0.812, -0.234, 1.053],
  "normal": [-0.998, 0.031, 0.048],
  "action": "push"
}
```

---

# 9. 然后才是 Affordance

例如 `push`：

```
surface point = P
normal = n
```

生成：

```
P_contact = P

P_approach =
P + 0.05 * n

P_push =
P - 0.005 * n

P_retreat =
P + 0.05 * n
```

这样你就自然得到了：

### 4个 Pose

```
approach
contact
push
retreat
```

这就是客户所谓 **1～5 个 TCP Pose** 的一种具体实现。

---

# 10. 内部 KeyPose 数据格式

这是我最建议你认真设计的一层。

不要把：

```
[x,y,z,rx,ry,rz]
```

直接到处传。

内部定义：

```
{
  "task_id": "task_0003",
  "frame_id": "robot_base",
  "tcp_frame": "tool_tcp",

  "poses": [
    {
      "id": "P1",
      "role": "approach",
      "position_m": [0.812, -0.234, 1.103],
      "quaternion_xyzw": [0.02, 0.70, 0.01, 0.71],
      "motion_hint": "PTP"
    },
    {
      "id": "P2",
      "role": "contact",
      "position_m": [0.812, -0.234, 1.053],
      "quaternion_xyzw": [0.02, 0.70, 0.01, 0.71],
      "motion_hint": "LIN"
    }
  ]
}
```

这应该成为：

> **整个项目最核心的中间表示 IR（Intermediate Representation）。**

---

# 11. 为什么内部姿态一定用 Quaternion

不要内部使用：

```
rx ry rz
```

因为：

- Euler angle convention 很多；
- XYZ / ZYX；
- intrinsic / extrinsic；
- degree / radians；
- singularity。

内部统一：

```
[x,y,z]
+
[qx,qy,qz,qw]
```

到 DAIHEN Adapter 最后才转换。

ROS 的 `geometry_msgs/Pose` 本身也正好如此。

---

# 12. ROS2 这一层

推荐：

```
ROS 2
├── tf2
├── robot_state_publisher
├── MoveIt 2
├── RViz2
└── 自己的 Nodes
```

你的节点可以是：

```
scene_server
vlm_grounding_node
geometry_node
keypose_node
motion_planner_node
fdst_adapter_node
```

---

# 13. ROS消息格式

## Pose

```
geometry_msgs/msg/PoseStamped
```

类似：

```
header.frame_id = "robot_base"

pose.position
pose.orientation
```

---

## Robot State

```
sensor_msgs/msg/JointState
```

---

## Scene / Obstacles

MoveIt：

```
moveit_msgs/msg/CollisionObject
moveit_msgs/msg/PlanningScene
```

MoveIt 的 `PlanningSceneInterface` 就是用这些对象增删碰撞物体。

---

## Trajectory

最终：

```
moveit_msgs/msg/RobotTrajectory
```

里面主要：

```
trajectory_msgs/msg/JointTrajectory
```

即：

```
time
q1...qN
velocity
acceleration
```

---

# 14. MoveIt怎么用

这里我会把 MoveIt 定为 **Phase 1 主 Planner**。

因为①：

> 不要求实时。

MoveIt已经提供：

- pose goal；
- IK；
- planning；
- collision；
- scene objects；
- trajectory。

`MoveGroupInterface` 可以直接设置 Pose goal、进行规划以及处理环境对象。

---

# 15. 你这个项目其实特别适合 MoveIt Task Constructor

因为你输出的不是一个 Pose，而是：

```
P1
↓
P2
↓
P3
```

MoveIt Task Constructor 本身就是：

> 把复杂 robot task 拆成多个 stage。

官方 Pick/Place 示例就是：

```
Current State
↓
Approach
↓
Grasp
↓
Lift
↓
Move
↓
Place
↓
Retreat
```

非常接近你这个项目。

所以以后可以：

```
KeyPoseSequence
        ↓
MTC Task Builder
        ↓
Stage 1 P1
Stage 2 P2
Stage 3 P3
        ↓
RobotTrajectory
```

---

# 16. Lv2的 PTP / LIN / CIRC 其实 MoveIt已经很好对应

客户要求：

> 直线 / 圆弧 / 指定なし。

MoveIt 的 **Pilz Industrial Motion Planner** 已经直接支持：

```
PTP
LIN
CIRC
```

并且 `MotionPlanRequest.planner_id` 就可以设置为 `"PTP"`、`"LIN"`、`"CIRC"`。

因此客户的 Lv2 基本可以映射：

|客户|MoveIt|
|---|---|
|指定なし|OMPL / PTP|
|直線|LIN|
|円弧|CIRC|

这非常适合这个项目。

---

# 17. cuRobo应该什么时候进来？

**不要一开始同时搞 MoveIt + cuRobo。**

第一阶段：

```
MoveIt only
```

因为稳定、ROS接口清楚、你也熟悉。

---

当做到③：

> 目标约3秒。

再尝试：

```
PlannerBackend
      │
 ┌────┴─────┐
 ▼          ▼
MoveIt    cuRobo
```

cuRobo现在提供 GPU：

- IK；
- collision；
- trajectory optimization；
- geometric planning；
- depth / TSDF / ESDF world mapping。

它的 `MotionGen` 输入基本就是：

```
start JointState
+
goal Pose
+
world model
```

然后输出 collision-free trajectory。

所以我会设计一个抽象接口：

```
class MotionPlannerBackend:
    def plan(
        start_state,
        keyposes,
        scene
    ) -> PlanResult:
        ...
```

实现两个：

```
MoveItBackend
CuroboBackend
```

这样未来切换不用改 VLM/Geometry。

---

# 18. cuRobo自己的世界数据格式

它可以使用 YAML/dict world：

```
mesh:
  machine:
    pose: [x, y, z, qw, qx, qy, qz]
    file_path: machine.obj

cuboid:
  table:
    dims: [1.0, 0.8, 0.1]
    pose: [0, 0, 0.75, 1, 0, 0, 0]
```

官方也是类似这种 world config，并使用 meter + quaternion。

所以我们的内部 SceneManifest 转 cuRobo 很简单：

```
SceneManifest
    ├── → MoveIt PlanningScene
    └── → cuRobo WorldConfig
```

这非常值得这样设计。

---

# 19. FD-ST Adapter

这是现在**唯一不能提前写死格式**的地方。

客户已经明确会提供：

- FD-ST；
- Unity TP App及开发环境；
- robot内部规格。

但是我们目前没有公开资料证明 FD-ST 存在：

```
REST API
Python API
CLI
```

因此接口应该设计成：

```
                 Internal IR

              KeyPoseSequence
                     │
                PlanResult
                     │
                     ▼
              FDSTAdapter
                     │
         ┌───────────┼───────────┐
         ▼           ▼           ▼
       API          File       Manual
    if available   Import      Import
```

---

# 20. Adapter 做的事情

例如内部：

```
meter
quaternion
robot_base
```

DAIHEN可能要求：

```
mm
degree
XYZ Euler?
specific tool frame
specific robot configuration
```

所以：

```
FDSTAdapter
├── unit conversion
├── frame conversion
├── orientation conversion
├── robot posture/config conversion
├── instruction conversion
└── file/program serialization
```

这个模块必须跟 AI 完全分开。

---

# 21. FD-ST输出以后不要自动马上执行

最初：

```
AI
↓
Planner
↓
FD-ST
↓
Simulation
↓
Human Review
↓
Export
↓
Robot
```

而不是：

```
AI → Robot
```

因为客户①本来就是 Offline Teaching 场景。

---

# 22. 如果以后做到③，才变成

```
Camera
↓
RGB-D
↓
PointCloud
↓
Target Detection
↓
KeyPose
↓
cuRobo
↓
Collision Check
↓
DAIHEN Controller
```

这里 FD-ST可能甚至不会每一次 runtime 都跑。

因为③目标约3秒，如果：

```
Camera
→ Linux PC
→ FD-ST Windows
→ Controller
```

链路太重，很可能做不到。

**这个必须问客户。**

③的 runtime architecture 很可能最终要：

```
Linux Planning PC
↓
DAIHEN Controller
```

而 FD-ST只是 offline validation。

---

# 23. 如果加入多个 RealSense

客户未来③的话：

```
realsense-ros
```

现在 ROS2 wrapper 已经支持：

- RGB；
- depth；
- point cloud；
- TF；
- camera extrinsics；
- 多 camera namespace。

ROS层就是：

```
/cam01/color/image_raw
/cam01/depth/image_rect_raw
/cam01/depth/color/points
/cam01/camera_info

/cam02/...
```

然后：

```
tf2:
robot_base
 ├── cam01
 ├── cam02
 └── cam03
```

---

# 24. 推荐的软件包总表

|层|Package|Phase 1|
|---|---|---|
|基础|Python 3|必须|
|数学|NumPy / SciPy|必须|
|Schema|Pydantic / JSON Schema|必须|
|3D|**Open3D**|必须|
|Mesh辅助|trimesh|推荐|
|视觉|OpenCV|推荐|
|AI|VLM API SDK / transformers|必须|
|Segmentation|SAM类模型|Optional|
|ROS|ROS2|必须|
|TF|tf2_ros|必须|
|Robot model|URDF / Xacro|必须|
|Planner|**MoveIt 2**|必须|
|Multi-stage|**MoveIt Task Constructor**|推荐|
|Industrial path|**Pilz Planner**|Lv2推荐|
|GPU Planner|cuRobo|③ Optional|
|Viewer|RViz2|必须|
|RGB-D|realsense-ros|③|
|Recording|rosbag2|③|
|FD-ST|DAIHEN FD-ST|后半|
|MuJoCo / Isaac|Simulation / VLA研究|**①不必须**|

---

# 25. 我建议的 Repository 结构

可以直接这么开项目：

```
daihen_genai_motion/
│
├── README.md
├── pyproject.toml
│
├── configs/
│   ├── scene.yaml
│   ├── task.yaml
│   ├── frames.yaml
│   └── vlm.yaml
│
├── assets/
│   ├── scenes/
│   ├── robots/
│   └── tools/
│
├── schemas/
│   ├── semantic_target.json
│   ├── interaction_target.json
│   ├── keypose_sequence.json
│   └── plan_result.json
│
├── src/
│   ├── scene_io/
│   ├── renderer/
│   ├── vlm_grounding/
│   ├── geometry/
│   ├── affordance/
│   ├── task_adapters/
│   │   ├── pick.py
│   │   ├── push.py
│   │   └── open_door.py
│   │
│   ├── planner/
│   │   ├── interface.py
│   │   ├── moveit_backend/
│   │   └── curobo_backend/
│   │
│   ├── fdst_adapter/
│   └── evaluation/
│
├── ros2_ws/
│   └── src/
│       ├── daihen_robot_description/
│       ├── daihen_robot_moveit_config/
│       ├── genai_motion_msgs/
│       ├── genai_motion_planner/
│       └── genai_motion_bringup/
│
└── tests/
    ├── scenes/
    ├── grounding/
    ├── geometry/
    └── planning/
```

---

# 26. 关键数据流，我建议固定成这 6 个 IR

不要让各个模块随便互传数据。

```
① SceneManifest
       ↓
② SemanticTarget
       ↓
③ InteractionTarget
       ↓
④ KeyPoseSequence
       ↓
⑤ PlanResult
       ↓
⑥ FDSTProgram / Export
```

对应：

```
scene.yaml

↓

semantic_target.json
“哪个东西”

↓

interaction_target.json
“3D里的哪个点/面”

↓

keypose_sequence.json
“机器人TCP去哪些地方”

↓

plan_result.json
“这条路线能不能走”

↓

DAIHEN format
“DAIHEN机器人怎么执行”
```

---

# 27. 这是整个系统最核心的设计思想

不要做成：

```
VLM
 ↓
MoveIt
 ↓
FD-ST
```

三个巨大的黑盒。

应该做成：

```
VLM
↓
Semantic Target
────────────────── 可单测

Geometry
↓
3D Interaction Target
────────────────── 可单测

Affordance
↓
1～5 KeyPose
────────────────── 可单测

MoveIt
↓
Trajectory
────────────────── 可单测

FD-ST
↓
DAIHEN validation
────────────────── 可单测
```

这样即使最后 VLM 效果不好，你仍然可以：

```
手动指定 SemanticTarget
↓
后面的整个系统继续工作
```

即使自动 grasp 不成功：

```
predefined grasp
↓
后面的系统继续工作
```

即使 FD-ST 没 API：

```
manual import
↓
仍然能完成 Demo
```

**这才是一个人五个月能够确保有成果的系统架构。**

---

## 我现在最建议你的具体技术选型

对于①第一版，我甚至会把系统缩到：

```
Python
│
├── VLM API
├── Open3D
├── trimesh
└── Pydantic

        ↓ JSON

ROS2
│
├── tf2
├── MoveIt 2
├── MoveIt Task Constructor
├── Pilz Planner
└── RViz2

        ↓ Pose / Trajectory Adapter

FD-ST

        ↓

DAIHEN Robot
```

**第一阶段不要上 MuJoCo、Isaac、cuRobo、VLA、Atlas。**

等这条链：

> **自然语言 → Target → 3D Point → 1～5 Pose → MoveIt成功规划 → FD-ST成功验证**

完整跑通之后，再依次加：

> `3DGS → grasp model → cuRobo → ③ runtime`

这样最稳。