# 📊 硬件连接与配置指南

## 🔌 硬件连接图

### 系统架构图
```
                    Tiaozhanbei2.0 系统架构
                           
    ┌─────────────────┐    USB 3.0    ┌──────────────────┐
    │                 │◄──────────────┤ Intel RealSense  │
    │                 │               │ D455 相机        │
    │   计算机/PC     │               └──────────────────┘
    │  (主控制器)     │                        │
    │                 │               ┌──────────────────┐
    │                 │    USB/串口   │   USB转TTL模块   │
    │                 │◄──────────────┤  (通信转换器)    │
    └─────────────────┘               └──────────────────┘
                                               │ UART
                                               │ TX/RX/GND
                                               ▼
                                      ┌──────────────────┐
                                      │  DJI RoboMaster  │
                                      │    C型开发板     │
                                      └──────────────────┘
                                               │
                                               │ 控制信号
                                               ▼
                                      ┌──────────────────┐
                                      │   机器人平台     │
                                      │  (移动底盘/机械臂) │
                                      └──────────────────┘
```

### 详细连接示意图
```
[PC端口分配]
├── USB 3.0 Port 1 ────────────── RealSense D455
├── USB 2.0 Port 2 ────────────── USB转TTL模块
└── 电源接口 ──────────────────── 外部供电 (可选)

[RealSense D455]
├── USB 3.0 接口 ────────────────► PC USB 3.0
├── 电源指示灯 (蓝色)
└── 工作指示灯 (绿色)

[USB转TTL模块]
├── USB接口 ────────────────────► PC USB 2.0  
├── VCC (5V) ──────────────────── 不连接 (C板自供电)
├── GND ──────────────────────── C板 GND
├── TXD ──────────────────────── C板 RX
└── RXD ──────────────────────── C板 TX

[DJI RoboMaster C板]
├── 电源输入 (12V) ────────────── 机器人电源系统
├── UART接口 ─────────────────── USB转TTL模块
│   ├── TX (PA9)
│   ├── RX (PA10) 
│   └── GND
├── CAN接口 ─────────────────── 电机控制器
└── GPIO ───────────────────── 传感器/执行器
```

## ⚙️ 硬件配置详解

### 🎥 RealSense D455 配置

#### 接口规格
- **连接接口**: USB 3.0 Type-C
- **供电方式**: USB总线供电 (5V, 最大900mA)
- **数据传输**: USB 3.0 SuperSpeed (5Gbps)

#### 安装位置建议
```
推荐安装高度: 0.8m - 1.5m
水平视角覆盖: 87° × 58°
最佳检测距离: 0.5m - 3.0m
避免强光直射和反光表面
```

#### 固定方案
1. **三脚架固定**: 标准1/4英寸螺丝接口
2. **机器人安装**: 使用官方安装支架
3. **临时固定**: 夹具或磁性底座

### 🔗 串口通信配置

#### USB转TTL模块选择
| 模块型号 | 芯片方案 | 支持系统 | 推荐度 |
|---------|---------|----------|--------|
| CH340G  | CH340   | Win/Linux/Mac | ⭐⭐⭐⭐ |
| CP2102  | Silicon Labs | Win/Linux/Mac | ⭐⭐⭐⭐⭐ |
| FT232RL | FTDI    | Win/Linux/Mac | ⭐⭐⭐⭐⭐ |
| PL2303  | Prolific | Win/Linux | ⭐⭐⭐ |

#### 接线对照表
| USB转TTL | DJI C板 | 信号说明 |
|----------|---------|----------|
| VCC      | 不连接   | C板自供电 |
| GND      | GND     | 公共地线 |
| TXD      | RX (PA10) | 数据发送 |
| RXD      | TX (PA9)  | 数据接收 |

### 🤖 DJI C板配置

#### 硬件版本兼容性
- **DJI RoboMaster C型开发板 V1.0**: ✅ 完全支持
- **DJI RoboMaster C型开发板 V2.0**: ✅ 完全支持  
- **DJI RoboMaster A型开发板**: ⚠️ 需要修改通信协议

#### UART配置参数
```c
// C板端配置示例
UART_InitTypeDef UART_InitStructure;
UART_InitStructure.UART_BaudRate = 115200;    // 波特率
UART_InitStructure.UART_WordLength = UART_WordLength_8b;  // 数据位
UART_InitStructure.UART_StopBits = UART_StopBits_1;      // 停止位
UART_InitStructure.UART_Parity = UART_Parity_No;         // 校验位
UART_InitStructure.UART_Mode = UART_Mode_Rx | UART_Mode_Tx;
```

## 🔧 软件配置示例

### 相机参数配置
```python
# src/config.py - CameraConfig类
class CameraConfig:
    # === 基础参数 ===
    width = 1280              # 图像宽度 (可选: 640, 1280, 1920)
    height = 720              # 图像高度 (可选: 480, 720, 1080)  
    fps = 30                  # 帧率 (可选: 15, 30, 60)
    
    # === 深度配置 ===
    depth_scale = 0.001       # 深度比例 (mm -> m)
    depth_range = (0.2, 5.0)  # 有效深度范围 (米)
    depth_unit = 0.001        # 深度单位 (毫米)
    
    # === 图像处理 ===
    enable_emitter = True     # 启用红外发射器
    laser_power = 150         # 激光功率 (0-360)
    accuracy = 2              # 精度等级 (1-3)
    
    # === 滤波器配置 ===
    spatial_filter = True     # 空间滤波器
    temporal_filter = True    # 时间滤波器
    hole_filling = True       # 孔洞填充
    
    # === 高级参数 ===
    auto_exposure = True      # 自动曝光
    exposure_time = 8500      # 曝光时间 (us)
    gain = 16                 # 增益值
```

### 机器人通信配置
```python
# src/config.py - RobotConfig类  
class RobotConfig:
    # === 串口基础配置 ===
    port = "COM3"             # Windows串口号
    # port = "/dev/ttyUSB0"   # Linux串口号
    # port = "/dev/tty.usbserial-0001"  # macOS串口号
    
    baudrate = 115200         # 波特率
    bytesize = 8              # 数据位
    parity = 'N'              # 校验位 (N=无, E=偶, O=奇)
    stopbits = 1              # 停止位
    timeout = 1.0             # 读取超时 (秒)
    write_timeout = 1.0       # 写入超时 (秒)
    
    # === 通信协议 ===
    command_frequency = 10    # 指令发送频率 (Hz)
    heartbeat_interval = 1.0  # 心跳间隔 (秒)
    max_retries = 3           # 最大重试次数
    
    # === 运动限制 ===
    max_linear_speed = 1.0    # 最大线速度 (m/s)
    max_angular_speed = 1.0   # 最大角速度 (rad/s)
    acceleration_limit = 2.0  # 加速度限制 (m/s²)
    
    # === 数据格式 ===
    data_format = "json"      # 数据格式 (json/binary)
    enable_checksum = True    # 启用校验和
    little_endian = True      # 字节序
```

### 感知算法配置
```python
# src/config.py - PerceptionConfig类
class PerceptionConfig:
    # === 管道检测参数 ===
    pipe_radius_range = (0.05, 0.5)     # 管道半径范围 (米)
    pipe_length_threshold = 1.0          # 最小管道长度 (米)
    pipe_detection_method = "hough"      # 检测方法 (hough/contour/ml)
    
    # === 法兰检测参数 ===
    flange_radius_range = (0.1, 0.8)    # 法兰半径范围 (米)
    circle_detection_threshold = 50      # 圆检测阈值
    min_circle_distance = 0.3            # 最小圆间距 (米)
    
    # === RANSAC参数 ===
    ransac_iterations = 1000             # 迭代次数
    ransac_threshold = 0.01              # 距离阈值 (米)
    ransac_min_samples = 100             # 最小样本数
    
    # === 滤波参数 ===
    kalman_filter = True                 # 启用卡尔曼滤波
    temporal_consistency = True          # 时间一致性检查
    outlier_rejection = True             # 异常值剔除
    
    # === 点云处理 ===
    voxel_size = 0.01                    # 体素大小 (米)
    statistical_outlier_removal = True   # 统计异常值移除
    radius_outlier_removal = True        # 半径异常值移除
```

## 🔍 调试与测试配置

### 日志级别配置
```python
# src/utils/logger.py
LOGGING_CONFIG = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'detailed': {
            'format': '%(asctime)s [%(levelname)s] %(name)s: %(message)s'
        }
    },
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
            'level': 'INFO',
            'formatter': 'detailed'
        },
        'file': {
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': 'output/logs/tiaozhanbei.log',
            'maxBytes': 10485760,  # 10MB
            'backupCount': 5,
            'level': 'DEBUG',
            'formatter': 'detailed'
        }
    },
    'root': {
        'level': 'DEBUG',
        'handlers': ['console', 'file']
    }
}
```

### 性能优化配置
```python
# 高性能配置示例
class HighPerformanceConfig:
    # 降低分辨率提高帧率
    camera_width = 640
    camera_height = 480
    camera_fps = 60
    
    # 减少处理线程数
    processing_threads = 2
    
    # 禁用耗时的滤波器
    spatial_filter = False
    temporal_filter = False
    
    # 简化算法参数
    ransac_iterations = 500
    detection_roi = (100, 100, 440, 280)  # 限制检测区域
```

## 🛠️ 环境变量配置

### Windows环境变量
```cmd
:: 设置Python路径
set PYTHONPATH=%PYTHONPATH%;F:\Tiaozhanbei2.0\src

:: 设置OpenCV后端
set OPENCV_DNN_BACKEND=CUDA

:: 设置RealSense日志级别
set LIBREALSENSE_LOG_LEVEL=INFO
```

### Linux/Mac环境变量
```bash
# 添加到 ~/.bashrc 或 ~/.zshrc
export PYTHONPATH=$PYTHONPATH:/path/to/Tiaozhanbei2.0/src
export OPENCV_DNN_BACKEND=CUDA
export LIBREALSENSE_LOG_LEVEL=INFO

# 串口权限 (Linux)
export SERIAL_DEVICE=/dev/ttyUSB0
sudo chmod 666 $SERIAL_DEVICE
```

## 📋 配置检查清单

### ✅ 硬件检查
- [ ] RealSense D455 USB 3.0连接正常
- [ ] 相机指示灯正常 (蓝色电源灯 + 绿色工作灯)
- [ ] USB转TTL模块正确识别
- [ ] DJI C板串口通信正常
- [ ] 机器人平台供电充足

### ✅ 软件检查  
- [ ] Python 3.8+ 环境正确安装
- [ ] 所有依赖包安装完成
- [ ] Intel RealSense SDK 正确安装
- [ ] 串口驱动程序安装
- [ ] 项目配置文件正确设置

### ✅ 功能检查
- [ ] 运行 `python src/main.py --config-check` 通过
- [ ] 演示模式正常运行
- [ ] 相机图像显示正常
- [ ] 串口通信测试成功
- [ ] 算法检测功能正常

---

**💡 提示**: 如果遇到配置问题，请参考主README文档的故障排除部分，或运行完整演示脚本进行自动检测。
