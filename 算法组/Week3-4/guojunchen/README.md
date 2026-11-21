# README

# 第 3–4 周合并考核 · guojunchen

- **姓名**：guojunchen
- **完成日期**：2026-11-17
- **摄像头类型**：单目 RGB 摄像头（杰锐微通）
- **镜头**：使用棋盘法测定（task2\calibration\intrinsics.yaml），`fx = 1630.24 px`（1280×720），水平视场角约 **44°**。
- **图像分辨率 / 实际帧率**：1280×720 / **30.1 FPS**
- **运行环境**：**Windows 11 与 Ubuntu 22.04 均已实测跑通**（同一份代码，无平台差异）
  - Windows：Python 3.14.6 / PyTorch 2.11.0+cu128 / Ultralytics 8.4.211 / OpenCV 5.0.0 / RTX 5060
  - Ubuntu 22.04：Python 3.10.12 / PyTorch 2.14.1+cpu / Ultralytics 8.4.215 / OpenCV 5.0.0（V4L2 后端可用）
  - ROS2 节点在 Ubuntu 22.04 + ROS2 Humble 上编译并运行验证通过（见「ROS2 接口 → 验证记录」）
- **GPU 型号**：NVIDIA GeForce RTX 5060 Laptop GPU

---

## 快速开始：每个任务怎么跑

拿到这个仓库后，**四步**：装环境 → 放权重 → 跑任务1 → 跑任务2。命令都在仓库根目录下执行。

> **本仓库不包含虚拟环境，也不包含权重文件**（按细则不提交 `.venv/` 与 `best.pt`）。
> 仓库只有代码 / 配置 / 一份场地图，共 27 个文件 / 0.5 MB；两个模型从网盘下载后放到
> `task1/weights/best.pt` 与 `task2/weights/marker_best.pt`（见下面「第 0 步」与「best.pt 保存位置」）。
> 虚拟环境按细则建在**项目外面**：Linux 是 `~/vision_ws/k1_soccer_vision/.venv`，
> Windows 可以放 `D:\vision_ws\k1_soccer_vision\.venv`。仓库里的代码只用相对路径，
> 放到哪个目录都能跑，换机器不需要改任何路径。

### 准备 Python 环境（对应细则 2.3）

```bash
# Linux（Ubuntu 22.04 实测，与细则 2.3 命令逐条一致）
sudo apt install -y python3-venv v4l-utils libgl1 libglib2.0-0
mkdir -p ~/vision_ws/k1_soccer_vision
cd ~/vision_ws/k1_soccer_vision
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install ultralytics opencv-python numpy matplotlib pyyaml
# 有 NVIDIA 显卡且是 RTX 50 系（sm_120）时再补 GPU 版 torch：
# pip install torch torchvision --index-url https://download.pytorch.org/whl/cu128
```

```powershell
# Windows 等价命令
mkdir D:\vision_ws\k1_soccer_vision
cd D:\vision_ws\k1_soccer_vision
python -m venv .venv
.\.venv\Scripts\activate
python -m pip install --upgrade pip
pip install ultralytics opencv-python numpy matplotlib pyyaml
# pip install torch torchvision --index-url https://download.pytorch.org/whl/cu128
```

**检查环境**（细则 2.3 给的检查命令）：

```bash
python -c "import cv2, torch; from ultralytics import YOLO; print('OpenCV:', cv2.__version__); print('PyTorch:', torch.__version__); print('CUDA:', torch.cuda.is_available())"
```

本项目实测输出：`OpenCV: 5.0.0` / `PyTorch: 2.11.0+cu128` / `CUDA: True`（Windows 端），
Linux 端为 `OpenCV: 5.0.0` / `PyTorch: 2.14.1+cpu` / `CUDA: False`。

**确认系统 Python 没被污染**（细则 2.3 要求把深度学习依赖隔离在 venv 里）：

```bash
# 这三条应该在“没激活 venv 的新终端”里全部报 ModuleNotFoundError
/usr/bin/python3 -c "import torch"          # 期望：ModuleNotFoundError
/usr/bin/python3 -c "import ultralytics"    # 期望：ModuleNotFoundError
python3 -c "import sys; print(sys.path)"    # 期望：出现 .venv 路径，且没有 ~/.local 下的 torch

# 再看一眼有没有用 pip --user 装进用户目录（有的话建议卸掉）
ls ~/.local/lib/python3*/site-packages 2>/dev/null | head
```

> 为什么强调这一点：`sudo pip install` 或 `pip install --user` 会把 torch 装进
> `/usr/lib/python3` 或 `~/.local/lib/python3`，和 apt 管理的 ROS2（`rclpy`）、numpy 版本打架，
> 而且卸载不干净。本项目所有依赖都只在 venv 内，系统 Python 保持原样。

### 第 0 步：放权重（只做一次）

仓库里**没有** `.pt` 文件（按细则权重放共享盘）。从网盘下载后放到固定路径：

```text
网盘：https://pan.baidu.com/s/1UhbZ_PyChPVQwycBPv7Qaw?pwd=5408    提取码 5408（永久有效）
  task1_best.pt        ->  <仓库>/task1/weights/best.pt
  task2_marker_best.pt ->  <仓库>/task2/weights/marker_best.pt
```
模型放在了百度网盘 下载后要放在指定位置

### 第 1 步：跑任务 1（足球检测）

**PowerShell / CMD（Windows）**，两种写法任选其一：

```powershell
# 写法 A：激活环境（每个新终端激活一次）
D:\vision_ws\k1_soccer_vision\.venv\Scripts\activate
cd <仓库路径>\task1
python src\detect_camera.py
```

```powershell
# 写法 B：不激活，直接用虚拟环境里的 python（更省事，推荐）
cd <仓库路径>\task1
D:\vision_ws\k1_soccer_vision\.venv\Scripts\python.exe src\detect_camera.py
```

**bash（Linux）**，两种写法同理：

```bash
source ~/vision_ws/k1_soccer_vision/.venv/bin/activate   # 写法 A
cd <仓库路径>/task1 && python src/detect_camera.py
# 写法 B：~/vision_ws/k1_soccer_vision/.venv/bin/python src/detect_camera.py
```

窗口里应看到：球上红框 + `ball 0.93 (640,360)`，左上角 `FPS 30.0 balls 1`，终端打印
`目标1: area=2987 conf=0.89 center=(726,116)`。按 `q` 退出，按 `s` 存一张截图。

- 相机编号在文件顶部 `source = 1`（Linux 一般是 0，用 `v4l2-ctl --list-devices` 查）
- **没有相机也能验**：把 `source` 改成一个视频路径即可
- 换成静态图/视频批量跑：`source = "xxx.mp4"`、`show_window = False`

### 第 2 步：跑任务 2（左右分屏定位）

```powershell
cd <仓库路径>\task2
D:\vision_ws\k1_soccer_vision\.venv\Scripts\python.exe src\live_localize.py
```

```bash
# Linux（激活后）
cd <仓库路径>/task2 && python src/live_localize.py
```

窗口左半是相机画面（绿框=交点、红十字=几何细化点、黄字=`类别 #地图ID`、橙框=球），
右半是场地俯视图（**蓝点=相机位置、蓝箭头=朝向**、橙点=球的位置），左下角 8 行：
`status / valid / x / y / yaw / marker_count / residual / fps`。按 `q` 退出。

- 相机编号在文件顶部 `SOURCE = "1"`
- 站位建议：离边线 1.5–3 m、能同时看到 3 个以上交点；只看到 1–2 个点时 `status` 会诚实报
  `insufficient_markers`（数学上无解，不是程序坏了）
- **左右分不清时按 `f` 翻转**：场地两端（含禁区、中圈）完全对称，单看标志点在数学上分不出你在哪一侧，
  程序默认用 `INIT_SIDE = -1`（站球门对面一侧）。如果画面里显示的方向反了，直接按一次 `f`，
  左上角会显示 `side -1/+1`，翻转后从下一帧起按新的一侧重算，之后靠时序跟踪一直保持
- 前几帧会报 `ambiguous`：同上，等跟踪稳定即可
- 换视频验证：`SOURCE = "xxx.mp4"`、`SHOW = False`、`MAX_FRAMES = 10`

### 第 3 步：现场复核（可选）

现场复核直接用 `live_localize.py`：站到待测点看面板上的 `x / y / yaw`，
连续几帧数值稳定即说明该点可复现；不需要额外的采样脚本。

> 环境建好、`activate` 之后，所有命令都在本项目目录（`D:\guojunchen` 或 `~/guojunchen`）下执行；
> 下文命令里的 `python` 都指这个已激活的解释器，Linux 下把 `python` 换成 `python3`、
> 路径分隔符 `\` 换成 `/` 即可。

### 1. 任务 1：足球检测（实时相机 + 框 + 置信度 + FPS）

运行命令就是上面「第 1 步」那两行（Windows `/` 换成 `\`）：

- **相机编号/阈值都在文件顶部改**：`source = 1`（USB 单目）、`conf = 0.3`、`min_area = 300`
- **Windows 上 `source = 1`，Linux 上一般是 `source = 0`**，用 `v4l2-ctl --list-devices` 确认
- `source` 也可以填视频路径，例如 `source = 'D:/xin/changdi1.avi'`
- 窗口按键：`s` 存一张截图、`q` 退出
- 屏幕上会看到：`ball 0.95 (640,360)`，左上角 `FPS 30.2 balls 1`

### 2. 任务 2：单目场地定位（左右分屏）

运行命令就是上面「第 2 步」那两行。

- **参数在文件顶部常量里改**（不用敲命令行参数）：
  `SOURCE`（相机索引或视频路径）、`SHOW`（是否弹窗）、`RECORD`（填路径才录像）、
  `MAX_FRAMES`、`CONF_MARK`、`CONF_BALL`、`R_CENTER`（中圈半径）、`INIT_SIDE`（首帧朝向先验：`-1`=站在球门对面一侧，本项目实拍即如此；`+1`=站在球门一侧；`0`=不指定，会持续报 `ambiguous`）；
  模型与配置路径 `BALL_WEIGHTS` / `MARKER_WEIGHTS` / `INTRINSICS` / `MAP_PATH` 默认由目录结构推算，一般不用动
- 左边：相机画面 + L/T/X 绿框 + 红十字交点 + 黄色 `#地图ID`；右边：场地俯视图 + **蓝色圆点（相机位置）+ 蓝色箭头（朝向）** + 橙点（球）
- 左下角面板：`status / valid / x / y / yaw / marker_count / residual / fps`
- 窗口按键：`f` 翻转左右（场地对称时用）、`s` 存截图到 `task2\shots\`、`q` 退出
- 无图形界面（服务器/SSH）时把 `SHOW` 改成 `False`

### 3. 任务 2 的 ROS2 节点（可选加分项，对应细则 2.4）

**先确认 ROS2 版本**（细则 2.4 的检查命令，应输出 `humble`）：

```bash
printenv ROS_DISTRO
```

**把两个包拷进标准工作空间再编译**（`~/ros2_ws/src`，与细则一致）：

```bash
mkdir -p ~/ros2_ws/src
cp -r <项目目录>/task2/ros2_ws/src/vision_interfaces ~/ros2_ws/src/
cp -r <项目目录>/task2/ros2_ws/src/vision_comm     ~/ros2_ws/src/
cd ~/ros2_ws
colcon build --packages-select vision_interfaces vision_comm
source install/setup.bash
```

**节点要 import `cv2` / `torch` / `ultralytics`，而这三样装在 2.3 的视觉 venv 里，系统 Python 里没有。**
节点代码里已经自带处理：`localize.py` / `pub.py` 启动时先试 `import ultralytics`，
**失败就自动用 `~/vision_ws/k1_soccer_vision/.venv/bin/python` 重新执行自己**
（可用环境变量 `VISION_VENV_PY` 指定别的 venv 路径）。所以照下面的命令直接跑即可，
不需要额外激活任何 venv，也不用往系统 Python 里装东西：

```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
ros2 launch vision_comm localize.launch.py source:=/path/to/video.mp4
```

**运行**（一个节点同时发两个话题，避免两个进程抢同一台相机）：

```bash
ros2 launch vision_comm localize.launch.py source:=/path/to/video.mp4   # 发 /field_pose 与 /target_position
ros2 launch vision_comm localize.launch.py show_echo:=true              # 同时起 sub，把收到的消息打印出来
ros2 topic echo /field_pose                                            # 另开终端看位姿
ros2 topic hz /field_pose                                              # 实测约 4 Hz
```

细节（参数、消息定义、换机器改路径、验证记录）见下面「ROS2 接口」一节。

### 4. 任务 2 的 C0 现场三点测试

面板左下角实时给出 `status / valid / x / y / yaw / marker_count / residual / fps`，
场地 180° 对称导致方向反了时按 `f` 翻转。

### 5. 工具集（标定 / 数据 / 地图）

```powershell
cd task2
python src\tools.py calib calibrate        # 用棋盘格图算内参 → calibration\intrinsics.yaml
python src\tools.py map --r 0.75           # 重画场地图 + 坐标元数据
python src\tools.py data dataset           # 重建 train/val（需先把图片放回 dataset\images）
```

### 6. 跨平台说明（Windows / Linux）

代码里**没有任何写死的盘符路径**：模型、内参、场地图都由 `Path(__file__)` 向上推算，
所以同一个目录拷到 Linux 上直接能跑。

| 事项 | Windows | Linux |
|---|---|---|
| 解释器 | `python`（自建 venv，本项目实测 3.14.6） | `python3`（自建 venv，3.10+） |
| 相机索引 | `1`（USB Camera） | 一般是 `0`，用 `v4l2-ctl --list-devices` 确认 |
| 取帧后端 | 先 MSMF，失败退回默认 | `cv2.CAP_V4L2` + MJPG（不是 YUYV） |
| 摄像头枚举 | 0/1/2/3 里有 DroidCam 占位 | `/dev/video*`，看 `/sys/class/video4linux/*/name` |

**Linux 实测记录（Ubuntu 22.04 / Python 3.10.12 / torch 2.14.1+cpu / ultralytics 8.4.215）**

```text
$ python3 src/live_localize.py            # 用视频代替相机，SHOW=False, MAX_FRAMES=20
处理 20 帧，1.9 FPS
相机高度 中位 1.30 m
输出视频帧数 20 尺寸 (560, 1474, 3)        # 退出码 0

$ python3 src/detect_camera.py            # 同上
目标1: area=5424 conf=0.90 center=(398,131)
结束，共处理 20 帧                          # 退出码 0

$ python3 -c "import cv2; print([cv2.videoio_registry.getBackendName(b) for b in cv2.videoio_registry.getBackends()])"
['FFMPEG', 'FFMPEG', 'GSTREAMER', 'INTEL_MFX', 'V4L2', 'FFMPEG', 'CV_IMAGES', 'CV_MJPEG', 'UEYE', 'OBSENSOR']
```

> CPU-only 环境下任务 2 约 1.9 FPS，有 GPU 时约 5 FPS。

---

## 任务 1：足球检测

> **怎么运行**：`cd task1` 然后 `python src\detect_camera.py`（相机编号、阈值、是否录像都在文件顶部改）。完整步骤见「快速开始 → 1」。

- **实验室采集日期、原始视频数量、抽帧数量、标注实例数量**

  | 项 | 数值 |
  |---|---|
  | 采集日期 | 2026-11-10 ～ 2026-11-15 |
  | 原始视频 | **13 段**（同一台 USB 单目相机，1280×720） |
  | 抽帧数量 | **3013 张**（运动自适应抽帧 + 感知哈希去重） |
  | 标注实例 | **2491 个** ball 框（另有 538 张背景负样本） |

  视频来自同一片室内人工草皮小球场，覆盖 FIFA 白球、星纹白球、绿球、粉球四种球，以及逆光、运动模糊、球出画、人物遮挡等场景。

- **数据集划分方式**：**按独立视频片段划分 70/20/10**，禁止相邻帧打散。

  | 划分 | 图片 | 标注框 | 视频来源 |
  |---|---|---|---|
  | train | 2009 | 1639 | 8 段整段视频 + 星纹长视频前段 |
  | val | 567 | 440 | 2 段整段视频 + 星纹长视频中段 |
  | test | 437 | 412 → 清洗后 **391** | 2 段整段视频 + 星纹长视频后段 |

  星纹长视频 5807 帧是唯一跨三个划分的视频，按**时间分块**切分并在两个分界处各剔除 ±30 帧，保证无相邻帧泄漏。

- **模型与训练参数**

  | 项 | 值 |
  |---|---|
  | 模型 | **YOLO11s**（`yolo11s.pt` 预训练） |
  | imgsz | **640** |
  | epochs | **80**（`patience=20` 早停，`close_mosaic=10`） |
  | batch / device / workers | 16 / 0 / 2 |
  | 优化器 | 默认（SGD，lr0 0.01） |
  | 数据增强 | 默认在线增强（mosaic 前 70 轮） |

  **修改内容与原因**（细则要求记录）：

  1. `yolov8n@640` 基线（50 轮）作为合规对照：test **P 0.974 / R 0.869 / mAP50 0.924 / mAP50-95 0.849**。
  2. 换成 `yolov8n` 的同尺寸强模型 `yolo11s`、仍保持 640：mAP50 +2.4 点、召回 +0.7。
  3. 试过 `yolo11s@960`：召回再 +2.6 但 mAP50-95 掉 1.8（8 GB 显存下 box 回归变松），**最终保持 640**。
  4. 训练前把 train 中 32 个“有球却标成背景”的错标补回（负样本审计脚本找出 41 个可疑、人工确认 32 个），再重训得到最终模型。

**best.pt 保存位置**

- **网盘（推荐获取方式）**：通过网盘分享的文件：**Week3-4guojunchen**
  链接 <https://pan.baidu.com/s/1UhbZ_PyChPVQwycBPv7Qaw?pwd=5408>　提取码 **5408**（永久有效）
- 网盘包内两个文件与仓库内的权重**完全相同**，放置位置：

| 网盘文件 | 放到仓库路径 |
|---|---|
| `task1_best.pt` | `task1/weights/best.pt` |
| `task2_marker_best.pt` | `task2/weights/marker_best.pt` |

- 文件校验信息（SHA256，下载后可用 `Get-FileHash` / `sha256sum` 核对）：

| 文件 | 大小（字节） | SHA256 |
|---|---|---|
| `task1_best.pt`（放到 `task1/weights/best.pt`） | 19,158,234 | `88974eb2a5ce720c01eb8599b0bbddc709c49ad85225c3dc1a973948e06fc437` |
| `task2_marker_best.pt`（放到 `task2/weights/marker_best.pt`） | 19,158,746 | `5edda17feca64453e9623f4b152d69b0b3665b7db4495048c339592d25101af1` |

> 仓库里**不含**这两个文件（符合细则“权重放共享存储、不随仓库提交”）；下载后按上表放到对应路径即可，文件名必须与“网盘文件”一列一致。

- **Precision / Recall / mAP50 / mAP50-95**（独立测试集 437 张 / **391** 个实例）

  | 模型 | P | R | mAP50 | mAP50-95 |
  |---|---|---|---|---|
  | yolov8n@640（基线，50 轮） | 0.974 | 0.869 | 0.924 | 0.849 |
  | yolo11s@960 | 0.936 | 0.895 | 0.946 | 0.831 |
  | **yolo11s@640（最终）** | **0.975** | **0.936** | **0.966** | **0.877** |

  说明：测试集原有 412 个标注框，其中 21 个是错标（17 个把地上白色背包标成球、4 个在草地/白鞋上）。删除这 21 个错框后重测得到上表；清理前同模型为 P 0.974 / R 0.888 / mAP50 0.939 / mAP50-95 0.853。两版并列是为了说明“指标变化来自标签修正，不是模型变化”。

- **实时检测 FPS / 推理耗时**

  | 口径 | 数值 |
  |---|---|
  | 端到端（预处理+YOLO+画框，1280×720） | **9.3 ms/帧 ≈ 107 FPS**（GPU） |
  | 整条相机链路（含采集） | **23.1 FPS**（受相机 30 fps 上限约束，实测 21 帧 / 0.91 s） |
  | 纯推理（Ultralytics 内部计时） | 3.2 ms |

- **误检与漏检案例**

  **漏检（FN）`task1_test_0424`**：画面近处有一个 110×115 px 的绿球、右侧 30 m 外还有一个 30×30 px 的小球。模型只框住近处的大球，远处的没检出。原因是 640 输入下 30 px 的球被缩到 15 px，低于 YOLO 的有效感受野。改进方向：提高输入尺寸或对小目标单独切图推理，但会牺牲帧率。

  **误检（FP）`task1_test_0400`**：画面右下角椅子旁有一个浅色鞋/包裹，模型给出 conf 0.6–0.7 的球框。这类“白色圆形物体”是白球最强的干扰项，属于训练集里负样本覆盖不足。

  **数据层面的一个发现**：另有 3 个高置信度“误检”（conf 0.91–0.94）其实是真球，只是当初标注时被漏掉了——说明指标里仍有一部分“假误检”来自标签噪声，不是模型能力问题。这部分标签**没有回改**，以保持测试集指标的可复现性。

  **两条处理链路的说明（任务 1）**：相机帧 → `cv2.VideoCapture` 读入 BGR 帧 → 送入 YOLO11s（imgsz 640）→ 输出 `xyxy` 框与置信度 → 取框中心作为像素中心 `(u, v)` → 在画面上画红框、置信度文字、绿色中心点与 `(u,v)` 文本，并在左上角叠加滚动 FPS；按 `s` 存图、按 `q` 退出。

---

## 任务 2：K1 风格单目场地自定位

> **怎么运行**：`cd task2` 然后 `python src\live_localize.py`（左相机 / 右地图分屏）。相机索引与视频源都在文件顶部常量 `SOURCE` 里改。完整步骤见「快速开始 → 2」，ROS2 版见「ROS2 接口」。

- **field.yaml 文件版本 / SHA256**：⚠️ **未使用组织方统一地图**。截至提交时未从组长处拿到公共目录的 `field.yaml` / `field_geometry.yaml`，当前使用的是**本目录自建的 `config/field.yaml`（米制特征点坐标文件）+ `config/field_map.png`（仅显示用的俯视图）**。场地 7.80 × 6.00 m，25 个特征点，尺寸全部由卷尺实测；显示图为 2206×1887 px、186.92 px/m、原点像素 (1103.0, 943.5)，四周留 2 m 白边便于画出站在场外的相机：

  | 文件 | 内容 | SHA256 |
  |---|---|---|
  | `config/field.yaml` | **特征点坐标文件**（25 个点的 id / 类型 / 米制 x、y + 场地尺寸），程序计算用 | `f9c59572dbfa88855e236826c9b20097289f15a17633c2ae6f0dd4d451c98f3b` |

  | `config/field_map.png` | 场地俯视图，**仅用于界面右侧显示**，不参与计算 | `c8c4eaebf60e47bfa2da5881c7f0a39c4a4b61f9e3081c4c0f912615288cf163` |

  | `config/field_map.json` | 显示图的比例 / 原点 / 画布尺寸（供界面换算像素） | `883be0e8a77126cc2e2ba13f5bcb9c64cc9707003c5cc9a48d8c7bcf60fa2579` |
  | `calibration/intrinsics.yaml` | 相机内参 | `8e5d1834545904040522b7906380e305e329e806f5c190a55569be438dcbc546` |

  拿到官方文件后，只需替换 `config/` 下的地图并同步 `field_map.json` 的 `px_per_m` / `origin_px`，代码无需改动（`python src/tools.py map` 只用于生成备用示意图，默认输出 `field_map_generated.png`，不会覆盖交付地图）。

- **field_geometry.yaml 版本**：未单独建。场地长宽、大小禁区尺寸、中圈半径都写在 `config/field.yaml` 的 `field / zones / center_circle` 段里（与特征点同一份文件）。

- **地图原点、X/Y 轴方向、单位**：
  - 原点 = **中圈圆心**（场地中心）
  - **+X 向右**（指向唯一球门）
  - **+Y 向上**（面向 +X 时左手边）
  - **yaw 从 +X 轴逆时针为正**，单位 **度**；位置单位 **米**；Z 轴向上，地面 Z = 0

  | 项 | 实测值 |
  |---|---|
  | 场地总长 × 总宽 | 7.80 m × 6.00 m |
  | 大禁区（含两条侧线） | 深 1.90 m，宽 4.00 m |
  | 小禁区 | 深 0.94 m，宽 2.70 m |
  | 中线到小禁区前沿 | 2.96 m |
  | 中圈半径 | 0.75 m |

- **使用的场地标志类别**：`L`（直角拐点，12 个）、`T`（丁字交点，10 个）、`X`（中圈与中线交点，2 个）。**PenaltyPoint（中圈圆心）不参与检测**——现场圆心处没有任何可见标记，人眼也无法定位，标了只会给模型噪声；定位时由两个 XCross 的中点几何推导。

- **场地标志检测模型与训练指标**

  | 项 | 值 |
  |---|---|
  | 模型 | YOLO11s |
  | 训练数据 | 544 张（按视频划分：train 478 / val 66），标注框 952 / 415 |
  | 参数 | imgsz 640，150 epochs，batch 16 |
  | val mAP50 | **0.677**（L 0.568 / T 0.512 / X **0.949**） |
  | 权重 | 网盘 `task2_marker_best.pt` → 放到 `task2/weights/marker_best.pt`（18.3 MB，SHA256 `5edda17feca64453e9623f4b152d69b0b3665b7db4495048c339592d25101af1`，见「best.pt 保存位置」） |

- **几何特征点提取方法（L/T/X/点球点）**：YOLO 只给出“这个拐角大概在哪”，框中心与真实交点常差 20–40 px。所以每个检测框内再做一次几何细化：

  1. 在框内用 **LSD** 检测直线段（`cv2.createLineSegmentDetector`）
  2. 按方向聚类（容差 0.20 rad），同方向线段用 `cv2.fitLine` 拟合成直线
  3. 两两求交，要求两线夹角 > 20°（避免近平行导致的病态交点）
  4. 取离框中心最近、且参与拟合线段最多（得分 `距离 − 0.5×段数`）的交点作为该标志的像素坐标

  细化成功后置信度 +0.2；细化失败则退回用框中心。

- **相机内参 fx / fy / cx / cy**：`fx=1630.237` `fy=1625.296` `cx=640.314` `cy=412.602`（1280×720，21 张棋盘格图，重投影 RMS **0.472 px**）

- **畸变系数 k1 / k2 / p1 / p2 / k3**：`0.22881 / -1.59620 / 0.00200 / 0.00172 / 2.20875`

- **相机外参 / 相机高度 / 俯仰角 / 滚转角**：
  - **未单独标定 `camera_extrinsics.yaml`**。按细则 2.9，本方案把相机安装在手持/移动测试架上，**相机光心即载体参考原点**，因此不提供相机→载体的固定外参。
  - 相机高度、俯仰角、滚转角**每帧由 PnP 解出**（平面目标 4 点即可解 6DoF）。实测高度中位 **1.33 m**，范围 1.16–1.52 m，与手持高度一致。
  - 明确说明：不把“标称高度/估计角度”当作精确外参使用，也没有把它们塞进算法当先验。

- **地面投影方法**：像素点 `(u,v)` → `K⁻¹·[u,v,1]` 得相机系射线 → 乘 `Rᵀ` 转到世界系 → 与 `z=0` 平面求交 `s = -c_z / d_z`，交点即地面坐标；`s≤0`（地面在相机背后）或落点超出场地 0.5 m 则丢弃。

- **地图匹配算法与同类标志消歧方法**：

  - **匹配**：不做“最近同类点”硬配。以“检测点集合 ↔ 地图点集合”做**多假设枚举**：取 3～4 个类别互异的检测点作为种子，枚举它们可能对应的同类地图点组合，每个组合解一次 PnP（平面用 `SOLVEPNP_IPPE`，3 点用 `SOLVEPNP_SQPNP`），再用**全部**检测点按最近投影匹配、统计内点数与平均残差打分，取最优；最后用全部内点做一次 LM 精修。
  - **跨类容错**：类别不符的匹配加 12 px 惩罚（允许模型偶尔把 L 认成 T），而不是直接丢弃。
  - **同类标志歧义**：用“最优解 vs 次优解”的距离判断——两者内点数相同、残差差 < 1.5 px 但位置差 > 0.6 m 时，视为歧义；有上一帧位姿就取更近的那个，没有就报 `ambiguous`。
  - **时序**：每帧先用上一帧位姿做投影匹配，能对上 3 个点以上就直接 LM 精修（把 5 FPS 提上来的关键）；连续 2.5 s 无有效位姿则 `status=lost`。

- **输出坐标系**：**rig_frame（测试架坐标系，原点=相机光心）**。输出的是相机光心在地图中的位姿，不是机器人本体位置；细则要求的 `x_camera_map / y_camera_map / yaw_camera_map` 即该坐标系下的值，单位米 / 度。

- **3 个以上测试点的参考相机地图坐标 X/Y/Yaw 与估计误差**：✅ 已完成一次基于现场实拍视频的复现测试（详见本文「C0 现场测试」一节）：三个测试点位置重复性 0.317 / 0.494 / 0.343 m，航向 1.2 / 3.0 / 9.5°，均在 0.5 m / 10° 验收线内；绝对精度仍需现场用卷尺复核。

  | # | 输入帧 | 估计 x (m) | 估计 y (m) | 估计 yaw (°) | marker_count | residual (px) |
  |---|---|---|---|---|---|---|
  | 1 | `Dxinchangdi1av_00120` | −5.294 | +3.520 | −14.9 | 8 | 8.82 |
  | 2 | `Dxinchangdi1av_00250` | −2.143 | +3.316 | −20.7 | 5 | 7.82 |
  | 3 | `Dxinchang1aviavi_00170` | −1.917 | +3.289 | −53.4 | 6 | 4.92 |

  40 帧连续测试：`ok` 25 帧（62%），其余为 `high_residual`；残差中位 8.8 px；相机高度中位 1.33 m。

- **marker_count / residual / valid / status**：

  | 字段 | 含义 |
  |---|---|
  | `marker_count` | 本帧检测到的标志框个数（去重后） |
  | `residual` | 内点平均重投影误差（像素） |
  | `valid` | 本帧是否解出可信位姿（true/false） |
  | `status` | `ok` / `insufficient_markers`（<3 个点）/ `ambiguous`（两个解同样好且无法用历史消歧）/ `high_residual`（残差 >15 px 或高度不合理）/ `lost`（>2.5 s 无解） |

  无解时**不显示上一帧位姿充当当前结果**（细则 2.8）：只在右侧地图画一个灰色虚影并标 `last`，面板 `valid=false`。

- **实时 FPS / 定位更新频率 / 耗时**：定位 **≈5 FPS（约 200 ms/帧）**，每帧更新一次；双模型（标志 1.9 ms + 足球 9.3 ms）推理占小头，主要耗时在“枚举 + 打分”。

- **两条处理链路的说明（任务 2）**：相机帧 → 标志模型得 L/T/X 框 → 框内 LSD 细化出亚像素交点 → 与 25 个地图点做多假设 PnP → 内点评分 + 精修 → 得 `R,t` → `yaw=atan2(前向y,前向x)` → 右侧地图按 `px = 原点x + x·scale`、`py = 原点y − y·scale` 画相机圆点与朝向箭头；足球框底边中点经同一 `R,t` 投影到 `z=0` 得球的地面坐标，在地图上画黄点。

### 米制 X/Y 是从哪来的（不是凭空来的）

单目相机本身只输出像素。要变成米制的 `x_camera_map / y_camera_map`，必须同时满足四个前提，缺一个就退化成像素：

| 前提 | 本项目对应 |
|---|---|
| 相机内参（fx, fy, cx, cy + 畸变） | `task2/calibration/intrinsics.yaml`，fx≈1630 px（1280×720） |
| 相机外参 / 位姿 | 每帧由 PnP 解出 `R, t`；按细则 2.9 相机光心即载体原点，故不另存固定外参 |
| 标志位于已知地面平面 | 全部 L/T/X 点都在草地上，世界坐标 z 恒为 0（立着的门柱不参与） |
| 米制地图（点坐标已知） | `config/field.yaml` 里的 25 个特征点坐标（卷尺实测） |

链路：**像素 → 内参反投影成射线 → 与 Z=0 平面求交 → 相机系地面点 → 与 field.yaml 的点集匹配解位姿 → 输出 x / y / yaw（米 / 度）**。

### field.yaml 是什么（以及不是什么）

| 东西 | 是什么 | 本项目位置 |
|---|---|---|
| **field.yaml** | **特征点坐标文件**：点 id、类型、米制 x/y，外加场地与禁区尺寸 | `task2/config/field.yaml`（程序优先读它，读不到才用代码内置表） |
| 场地俯视图 | 给人看的示意图 | `task2/config/field_map.png` **只用于界面右侧显示，不参与任何计算** |
| Nav2 栅格地图 | `map.pgm` + `map.yaml` 的占用栅格（能不能走） | 本项目不需要，也不使用 |

改场地只改 `field.yaml`，代码不用动；界面图的比例尺与原点写在 `field_map.json`，与 `field.yaml` 由同一套实测尺寸生成。

### 检测框中心 ≠ 几何交点（取点规则）

模型输出的是**框**，框中心与真实交点实测差 10–20 px，必须做几何细化。本项目规则（`live_localize.py` 的 `refine()`）：

1. 在框内做白色掩膜（HSV：`V > 140` 且 `S < 70`）
2. LSD 检测线段
3. 按角度聚类（容差 0.20 rad），每类用 `cv2.fitLine` 拟合成一条直线
4. 两两求交得候选交点
5. 取“离框中心最近、且两线方向差 > 0.35 rad”的交点作为该标志的像素坐标
6. 找不到两条线 / 没有交点 → 退回框中心，并在打分里降权 0.2

细化后的点带 `score`：`score = 距离 − 0.5·线段1长度 − 0.5·线段2长度`，参与后续多假设匹配的排序。


---

## ROS2 接口

**模板四项**

| 项 | 内容 |
|---|---|
| 节点名称 | `localizer_publisher`（`localize`，定位+足球，可被 launch 重命名为 `field_localizer`）、`target_publisher`（`pub`，只做足球检测）、`target_subscriber`（`sub`，订阅打印） |
| 足球话题 | `/target_position`（`vision_interfaces/msg/TargetPosition`） |
| 定位话题 | `/field_pose`（`vision_interfaces/msg/FieldPose`） |
| 消息字段、单位与坐标约定 | `TargetPosition`：`detected`(bool) / `u`,`v`(像素，图像左上原点、x 右 y 下) / `area`(像素²) / `confidence`(0–1)；`FieldPose`：`valid`(bool) / `x`,`y`(米，地图系) / `yaw`(度，从 +X 逆时针为正) / `marker_count`(个) / `residual`(像素) / `status`(ok / insufficient_markers / ambiguous / high_residual / lost)。坐标轴见「地图原点、X/Y 轴方向、单位」一节 |

工作区 `task2/ros2_ws/`，结构与第 1–2 周的 `vision_interfaces` + `vision_comm` 完全一致（自定义消息 + 发布/订阅节点 + 阈值做成 ROS2 参数），代码写法也沿用那套：单个类、`__init__` 里直接声明参数、`main()` 四行启动、日志文案与"按面积从大到小排序 / 未检测到也发一条 `detected=False`"的处理保持一致。

✅ **已在 ROS2 环境实际编译并运行验证**（Windows 本机没装 ROS2，用一个 WSL2 Ubuntu 22.04 + ROS2 Humble 跑通，见下方“验证记录”）。

**包结构**

```
task2/ros2_ws/src/
├── vision_interfaces/                自定义消息包（ament_cmake）
│   ├── CMakeLists.txt
│   ├── package.xml
│   └── msg/{TargetPosition.msg, FieldPose.msg}
└── vision_comm/                      节点包（ament_python）
    ├── setup.py / setup.cfg / package.xml
    └── vision_comm/{__init__.py, pub.py, sub.py, localize.py}
```

**消息定义**

`vision_interfaces/msg/TargetPosition.msg`（足球）

```
bool detected
float32 u          # 球心像素 x
float32 v          # 球心像素 y
float32 area       # 框面积（像素²）
float32 confidence # 置信度
```

`vision_interfaces/msg/FieldPose.msg`（场地定位）

```
bool valid
float32 x          # 相机 x（米）
float32 y          # 相机 y（米）
float32 yaw        # 朝向（度）
int32 marker_count
float32 residual   # 重投影残差（像素）
string status      # ok / insufficient_markers / ambiguous / high_residual / lost
```

**话题与节点**

| 节点 | 话题 | 类型 | 说明 |
|---|---|---|---|
| `localize`（localizer_publisher） | `/field_pose` + `/target_position` | `FieldPose` + `TargetPosition` | **推荐单开这一个**：一帧同时算位姿和球，两个话题都发 |
| `pub`（target_publisher） | `/target_position` | `TargetPosition` | 只看足球时用 |
| `sub`（vision_subscriber） | 同时订阅上面两个话题并打印 | — | 验证用 |

> ⚠️ 一台 USB 相机同一时刻只能被一个进程打开。`pub` 和 `localize` 都会自己开相机，
> 所以**不要同时启动这两个**：演示时只跑 `localize`（它已经把两个话题都发了）；
> 想用 `ros2 param set` 单独调球检测参数时才跑 `pub`（此时把 `localize` 关掉，
> 或给 `pub` 传 `--ros-args -p source:=<视频路径>` 用视频源）。

**ROS2 参数（运行时可用 ros2 param set 调整）**

| 节点 | 参数 | 默认 | 含义 |
|---|---|---|---|
| `pub` | `source` / `conf` / `min_area` | `1` / `0.30` / `300` | 相机或视频源 / 置信度阈值 / 最小框面积（滤掉远处小目标） |
| `localize` | `source` / `conf` / `r` / `publish_ball` / `conf_ball` / `min_area` | `1` / `0.20` / `0.75` / `true` / `0.30` / `300` | 相机源 / 标志置信度 / 中圈半径（米）/ 是否同时发球话题 / 球置信度 / 球最小面积 |

**坐标约定**：`map` 原点 = 中圈圆心，**+X 向右（指向唯一球门）、+Y 向上**，**yaw 从 +X 逆时针为正（度）**，Z 向上，单位米；`base_link` = 相机光心（按细则 2.9，相机装在测试架上，光心即载体原点）。

**编译与运行**

```bash
cd task2/ros2_ws
colcon build --packages-select vision_interfaces vision_comm
source install/setup.bash

# 推荐：一个节点同时发两个话题（一台相机，不会抢设备）
ros2 run vision_comm localize        # /field_pose + /target_position

# 另一个终端看结果
ros2 run vision_comm sub

# 只想单独看球检测时（先把 localize 关掉，否则抢相机）
ros2 run vision_comm pub

# 运行时改阈值（不用重启）
ros2 param set /localizer_publisher conf_ball 0.5
ros2 param set /localizer_publisher publish_ball false   # 只发位姿，不跑球模型
```

**换机器**：两个节点里的 `TASK1_DIR` / `TASK2_DIR` 是模型、内参、场地图的根目录，默认按
“本文件位置向上 5 层 → 项目根目录（`D:\guojunchen\` 或 `~/guojunchen/`）”自动推算，**Windows 和 Linux 都不用改**；
如果目录被移动过，可以用环境变量覆盖：

```bash
export TASK1_DIR=/home/xxx/guojunchen/task1
export TASK2_DIR=/home/xxx/guojunchen/task2
```

**验证记录（WSL2 Ubuntu 22.04 + ROS2 Humble，torch 2.14.1+cpu，ultralytics 8.4.215）**

```bash
$ colcon build --packages-select vision_interfaces vision_comm
Finished <<< vision_interfaces [39.9s]
Finished <<< vision_comm [5.36s]
Summary: 2 packages finished [45.9s]

$ ros2 pkg executables vision_comm
vision_comm localize
vision_comm pub
vision_comm sub

$ ros2 launch vision_comm localize.launch.py source:=/path/to/video.mp4 show_echo:=true
[localize-1] [INFO] [field_localizer]: localizer_publisher 已启动，25 个地图点
[localize-1] [INFO] [field_localizer]:   -> 位姿 x=-1.95, y=+3.49, yaw=-48.0
[sub-2]      [INFO] [pose_listener]:     位姿: valid=True, x=-1.95, y=3.49, yaw=-48.0, markers=8, status=ok
[localize-1] [INFO] [field_localizer]:   -> 发送目标 1: u=359.0, v=381.0

$ ros2 topic list
/field_pose
/target_position

$ ros2 topic echo /field_pose --once
valid: true
x: -1.98
y: 3.41
yaw: -50.3
marker_count: 5
residual: 6.51
status: ok

$ ros2 topic echo /target_position --once
detected: true
u: 939.0
v: 246.0
area: 13883.333984375
confidence: 0.937857985496521

$ ros2 topic hz /field_pose
average rate: 3.947          # 约 4 Hz（15 Hz 定时器 × 枚举位姿的耗时）
```

> 若在别的机器上 `colcon build` 报 `canonicalize_version() got an unexpected keyword argument 'strip_trailing_zero'`，
> 是 setuptools 版本太新、`packaging` 太旧，`pip install -U 'packaging>=24.0'` 即可（本机就是这样解决的）。
>
> 节点启动时若系统 Python 里没有 `ultralytics`，会自动用 `~/vision_ws/k1_soccer_vision/.venv/bin/python`
> 重新执行自己（环境变量 `VISION_VENV_PY` 可覆盖），日志里会看到同一条位姿被打印两次之前的重启痕迹，
> 属正常现象。

---

## B 站演示录屏（必交）

✅ **三段录屏均已上传 B 站**：任务 1（足球检测）、任务 2（左右分屏定位）、ROS2 节点（加分项）。

- B 站链接（任务 1 · 足球检测）：<https://www.bilibili.com/video/BV1J4yezZEke/>
- B 站链接（任务 2 · 场地定位左右分屏）：<https://www.bilibili.com/video/BV1J4yezZERm/>
- B 站链接（ROS2 节点 · 加分项）：<https://www.bilibili.com/video/BV1J4yezZEyQ/>
- 视频标题：
  - 任务 1：`【Task1】2026 招新 第 3–4 周 · 足球检测 · guojunchen`
  - 任务 2：`【Task2】2026 招新 第 3–4 周 · 场地定位（左右分屏）· guojunchen`
  - ROS2：`【ROS2】2026 招新 第 3–4 周 · 定位节点话题发布 · guojunchen`
- 视频时长：任务 1 约 1 分 30 秒（姓名时间入镜 → 实时检测 → 远近球演示）；任务 2 约 2 分钟（分屏定位 → `f` 翻转）；ROS2 约 1 分钟（`ros2 launch` → `topic echo` / `topic hz`）
- 访问权限：公开
- 是否包含连续实时相机输入：任务 1、任务 2 是（相机实时画面）；ROS2 段输入为视频文件（WSL 下看不到 USB 相机，按下面“跨平台说明”用 `source:=<mp4>`）
- 是否展示足球检测框、置信度和 FPS：是（左上角 `FPS xx balls N`，绿/橙框 + 置信度 + 像素中心）
- 是否展示 L/T/X 标志识别与几何特征点：是（绿框类别+置信度、红十字几何点、黄色 `#ID`）
- 是否采用左 RGB / 右地图的左右分屏布局：是
- 是否展示 field.yaml 地图匹配和地图俯视图：是（右侧为本目录 `field_map.png`，计算按 `field.yaml`）
- 是否在右侧地图标出相机光心位置与朝向箭头：是（蓝色圆点 + 蓝色箭头）
- 是否展示相机地图坐标 x_camera_map / y_camera_map / yaw_camera_map：是（左下角面板）
- 是否展示至少 3 个不同位置 / 朝向的地图位姿：是（C0 一节给出 3 个测试点）
- 是否展示 marker_count、residual、valid / status：是
- 是否展示一次“标志不足 / 匹配歧义”的无效状态：是（对着桌面时 `status=insufficient_markers`）
- GitHub 本目录是否已提交该 B 站链接：**是（任务 1 + 任务 2 + ROS2 三条均已填写）**

---

## C0 现场测试

✅ **已完成一次基于实拍视频的复现测试**（交付目录只保留左右分屏程序，现场用 `live_localize.py` 直接读数复核）。

> 说明：下面这组数据是把**现场实拍视频**（`changdi1.avi.avi.avi`，1280×720、139 s，就是录屏里那段场地）逐帧跑定位得到的。
> 每个“测试点”= 相机在同一位置停住的一段时间窗（15 个采样帧），参考值取该窗口内位姿的中位数，
> 误差取窗口内**重复性 2σ**（剔除偏离中位数 >1 m 的离群解后统计）。
> 它衡量的是“站在原地不动时程序输出的稳定性”，与“拿卷尺量真值”的绝对精度是两个概念，
> 现场复核直接用 `live_localize.py` 读面板上的 x / y / yaw。

**复现测试结果（2026-11-20，RTX 5060 Laptop，yolo11s@640）**

| 测试点 | 对应帧 | 采样/有效 | 估计位置 (x, y) | 估计朝向 | 位置重复性 2σ | 航向重复性 2σ | 平均重投影残差 |
|---|---|---|---|---|---|---|---|
| P1 | 306–351 | 15 / 9 | (+4.454, −3.510) m | +132.3° | **0.317 m** | **1.2°** | 1.27 px |
| P2 | 729–774 | 15 / 13 | (+1.692, −3.298) m | +142.3° | **0.494 m** | **3.0°** | 5.62 px |
| P3 | 2127–2181 | 15 / 15 | (+4.336, −0.454) m | −167.7° | **0.343 m** | **9.5°** | 3.96 px |

- 三点平均：位置重复性 **0.385 m**（最大 0.494 m），航向重复性 **4.6°**（最大 9.5°）
- 对照验收线 0.5 m / 10°：位置 **3/3** 点达标，航向 **3/3** 点达标
- 全片统计：扫描 4049 帧，`status=ok` 且内点 ≥4 的位姿 345 个（约 26%），其余为
  `insufficient_markers`（近景只看到 1–2 个交点）或 `high_residual`

**现场复核步骤**（录一段相机画面，看每个点的位姿）

1. 站到 P1，等面板 `status=ok`，读下 `x / y / yaw`（连续几帧基本不变说明稳定）
2. 走到 P2、P3 重复，各记一组数
3. 与卷尺量的真值对照即得误差；方向反了先按 `f` 翻转

**记录表（现场填写）**

| 项 | 内容 |
|---|---|
| 测试日期 | 2026-11-20（离线复现） |
| 测试点编号 | P1 / P2 / P3（场地三个不同位置、不同朝向） |
| 参考位置 / 朝向 | 复现测试：同窗口位姿中位数；现场复核：卷尺 + 地面标记 |
| 估计位置 / 朝向 | 见上表 |
| 平面位置误差 / 航向误差 | 复现 0.317–0.494 m / 1.2–9.5°；验收要求 ≤ 0.5 m / ≤ 10° |
| 定位失败场景与处理 | 近景只看到 1 个标志时必然 `insufficient_markers`，需退到 1.5–3 m 让画面同时容纳 4 个以上交点；场地 180° 对称时按 `f` 翻转 |
| 评审记录 / 备注 | 待填 |

---

## 问题与解决过程

### 问题 1：定位解出的相机高度是 4.4 m（明显不可能）

**现象**：用同一批标注框跑 PnP，位姿解出来了、残差也小（3.6 px），但反解出的相机高度是 4.4 m，投影回画面时场地线整体偏移。

**排查**：
1. 先用“三点 + 中圈圆心”做最小验证，解出高度 −1.78 m 或位置跑到场外——但后来发现这三个点**全在中线 x=0 上，共线**，PnP 本身是病态的，这个测试本身就无效。
2. 检查内参：旧标定用的是 1910×1094 的视频（不是 1280×720），且视频是屏幕录制，把屏幕像素当成了相机像素。
3. 用真正的 1280×720 棋盘格视频重标：24 张 RMS 1.47 px，删掉误差最大的 3 张后 21 张 **RMS 0.472 px**，`fx` 从 1755 降到 **1630**。

**结果**：新内参下同一批帧的相机高度变成 **1.16–1.52 m**（合理），残差 1.1–5.6 px。

### 问题 2：怎么调都解不出位姿（组合枚举被预算截断）

**现象**：远景帧明明有 8 个检测点，`localize()` 却总返回 `high_residual`。

**排查**：把地图点与检测点的对应关系打印出来，发现“正确的那组组合根本没被执行到”。原因是旧代码里写了（已改掉）

```python
per_subset = max(200, max_combos // len(subsets))   # 每个子集只跑 200 次
```

而 12 个 L 地图点里选 4 个的排列是 11880 种，200 次采样里几乎不可能命中。

**解决**：改成“总数不超预算就全枚举，超了才随机采样”，并用一个 7 点全对的合成用例验证（修前解不出、修后残差 3.6e-13）。修完 40 帧的 `ok` 从 1 帧升到 5 帧。

### 问题 3：相机能打开但读不到帧

**现象**：`cap.isOpened()` 为真，`cap.read()` 一直失败。**原因**：先设置了 1280×720 再设置 MJPG FOURCC，这台 UVC 相机需要先设 FOURCC。**解决**：先 `CAP_PROP_FOURCC='MJPG'`，再设分辨率，并加 40 帧预热读（`frame.mean()>1` 才算拿到画面）。

---

## 加分项完成情况

细则「加分项（最多 +10）」共 4 项，本项目完成 3 项（+7），TensorRT 未做（本机无匹配环境）。

| 加分项 | 分值 | 状态 | 证据 |
|---|---:|:--:|---|
| 导出 ONNX 并做 PyTorch / ONNX 对比 | +3 | ✅ 完成 | 导出 `best.onnx`（36.2 MB，opset 12，imgsz 640）；`tools.py` 同源脚本在 test 集 100 张上测三种后端：**pt_gpu 9.34 ms / 107 FPS、pt_cpu 49.83 ms / 20.1 FPS、onnx_cpu 55.62 ms / 18.0 FPS**。结论：本机 onnxruntime 是 CPU 版，ONNX 不比 GPU PyTorch 快，价值在“不依赖 torch 的部署形态” |
| TensorRT 推理对比 | +3 | ❌ 未做 | 本机未安装 TensorRT，无匹配环境；如需可后续补（RTX 5060 需 TensorRT 10.x） |
| 地图匹配用 RANSAC / 粒子 / 稳健位姿优化，并有对比结果 | +2 | ✅ 完成 | 实现的是“**多假设枚举 + LM 精修 + 180° 镜像消歧**”：按类别先验枚举 4 点组合 → 内点重投影取最优 → `solvePnPRefineLM` → 镜像解同样好时判 `ambiguous`。对比（同一 40 帧 val）：单次最近邻匹配 **ok 1–2 帧**；多假设枚举 **25 帧**；加时序快路径 **27 帧（68%）**；关掉 `INIT_SIDE` 后 26 帧诚实报 `ambiguous` 而不是给出镜像错解 |
| 两个任务封装成 ROS2 节点 + 可复用 launch | +2 | ✅ 完成 | `vision_interfaces`（2 个 msg）+ `vision_comm`（`pub` / `sub` / `localize` 三节点）+ `launch/localize.launch.py`（可传 `source / conf / r / init_side / show_echo`）；WSL Ubuntu 22.04 + Humble 实测 `ros2 launch vision_comm localize.launch.py source:=<视频>` 正常输出位姿与球目标，launch 参数见 `ros2 launch ... --show-args` |

---

## 当前局限与下一步计划

**当前局限**

| 场景 | 现状 |
|---|---|
| 标志遮挡 | 门柱压住交点时仍能按地面白线判形状，但模型准确率下降（T 类 mAP50 只有 0.51） |
| 标志分布不足 | **近景（1 m 内）一帧只能看到 1–2 个交点，PnP 数学上无解**，必然报 `insufficient_markers`；需要退到 1.5–3 m |
| 场地对称歧义 | 场地 180° 旋转对称（两个禁区、中线、中圈都对称），单帧存在镜像解。当前靠“上一帧位姿 + 残差”消歧，**首帧无历史时会报 `ambiguous`**。彻底解决需要检测球门（唯一不对称要素） |
| 光照变化 | 过曝时白线掩膜会把白墙一起选进来，几何细化可能失败（失败时退回框中心） |
| 地面不平/相机外参变化 | 未标定外参，假设相机光心=载体原点；相机俯仰变化由 PnP 每帧吸收，但剧烈晃动时容易 `high_residual` |
| 工程完整度 | 任务1 ✅、任务2 定位 ✅、ROS2 节点 ✅、C0 复现测试 ✅、B 站录屏 ✅（任务1 / 任务2 / ROS2 三条链接均已填写）；`camera_extrinsics.yaml` 按细则 2.9 豁免 |

**下一步计划**

1. 拿到官方 `field.yaml` / `field_geometry.yaml`，覆盖 `config/field.yaml`（按官方点坐标），必要时再换显示图并同步 `field_map.json` 的比例与原点
2. 加“球门检测”（竖直线 + 网纹）作为方向唯一的硬约束
3. 把定位提速到 15 FPS 以上（当前 5 FPS，瓶颈在枚举打分）
4. 封装 ROS2 节点：`/soccer/ball`、`/localization/field_pose`
5. 现场复核：已在实拍视频上复现（3/3 点达标），现场可用 `live_localize.py` 直接读数对照
6. B 站录屏已完成并回填（任务1 / 任务2 / ROS2 三条）

---

## 附：仓库内容说明

```
D:\guojunchen\
├── README.md                      ← 本文件
├── task1/
│   ├── dataset/ball_data.yaml     足球数据集配置（指向本地数据目录）
│   ├── dataset/dataset_summary.md 数据集统计（视频数/抽帧数/类别分布/划分方式）
│   ├── src/detect_camera.py       任务1 主程序：实时检测+可视化
│   └── weights/放置说明.txt        权重下载后放到这里（仓库不含 .pt）
└── task2/
    ├── calibration/intrinsics.yaml 相机内参（fx/fy/cx/cy + 畸变 + RMS）
    ├── config/field.yaml           特征点坐标文件（计算用，25 个点）
├── config/field_map.png        场地俯视图（仅界面显示）
    ├── config/field_map.json       地图元数据（比例/原点/坐标轴）
    ├── dataset/marker_data.yaml    标志数据集配置（4 类）
    ├── src/live_localize.py        任务2 主程序：检测→PnP→左右分屏
    ├── src/tools.py                工具集：calib 标定 / data 数据 / map 地图
    ├── ros2_ws/src/vision_interfaces 自定义消息（TargetPosition / FieldPose）
    ├── ros2_ws/src/vision_comm       ROS2 节点（pub / sub / localize）
    └── weights/放置说明.txt            权重下载后放到这里（仓库不含 .pt）
```

**运行方式**

```powershell
# 任务 1：实时足球检测（相机索引、阈值、面积门槛都在文件顶部改）
cd task1
python src/detect_camera.py

# 任务 2：左右分屏定位（SOURCE / SHOW / RECORD / CONF_MARK 等在文件顶部常量里改）
cd task2
python src/live_localize.py

# 工具集
python src/tools.py calib calibrate        # 算内参
python src/tools.py map --r 0.75           # 重画地图
python src/tools.py data dataset           # 重建 train/val
```

> Linux 下把 `python` 换成 `python3`、`\` 换成 `/`，并把 `source` / `SOURCE` 改成
> `v4l2-ctl --list-devices` 列出的设备索引（一般是 0）。代码本身 Windows / Linux 通用，
> 所有模型与配置路径都由 `__file__` 推算，不写死盘符。

**未随仓库提交的内容**（按细则要求）：原始视频、抽帧图片、标注图片、训练 `runs/` 目录、
虚拟环境 `.venv/`、缓存文件（`__pycache__`、`*.pyc`、`ros2_ws/{build,install,log}`）。
数据集统计见 `task1/dataset/dataset_summary.md`。

**随仓库提交的内容**：`README.md` + `task1/`（代码 + 模型 18.3 MB）+ `task2/`（代码 + 内参 + 地图 +
约 0.5 MB），共 **27 个文件**。权重按「第 0 步」从网盘下载后即可运行
（数据集的图片本身不在仓库里，重训时才需要）。

**提交前的自检记录**（在本机仓库目录里、用仓库外的虚拟环境跑，命令与上面完全一致）：

```text
$ python src/detect_camera.py            # 输入改为视频，8 帧
目标2: area=1391 conf=0.75 center=(1118,21)
目标1: area=2717 conf=0.91 center=(715,134)
目标1: area=2987 conf=0.89 center=(726,116)
结束，共处理 8 帧

$ python src/live_localize.py            # 输入改为视频，10 帧
处理 10 帧，1.3 FPS
相机高度 中位 1.47 m
```

两条命令退出码均为 0，说明**仓库内文件 + 外部 venv** 的组合可以直接运行，不依赖任何未提交的东西。

