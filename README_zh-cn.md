# SpacemiT 芯片厂商对 openvela 的支持

\[ [English](README.md) | 简体中文 \]

## 简介

本仓库包含**进迭时空（SpacemiT）** 芯片平台为 openvela 操作系统提供的板级支持包（BSP）、芯片驱动与产品工程。SpacemiT 是一家专注于 RISC-V 处理器的高性能计算半导体厂商，基于自研 X60™ RISC-V CPU 核心，提供面向 AIoT、边缘计算与开发者生态的完整芯片平台解决方案。

目前本仓库聚焦于服务 **openvela AI 硬件开发者大赛**，主分支为 `dev-ai-contest-2026`。

> ⚠️ **仓库状态：初始建设阶段**
>
> 本仓库当前处于大赛分支 `dev-ai-contest-2026` 的初始建设阶段：芯片层（`chips/k1/`）与板级（`boards/k1/muse_pi_pro/`）的 openvela 适配代码正在准备中，将在后续 PR 中陆续合入。本 README 描述的是**目标支持状态**与硬件参考规格，编译命令与设备节点章节会在适配代码合入后补充。

## 支持的硬件

### SpacemiT Key Stone® K1 SoC

**SpacemiT K1** 是一颗面向 AIoT 与边缘计算的高性能、超低功耗 SoC，集成 8 核 RISC-V 处理器与 SpacemiT® 导义™ AI 计算能力。核心特性：

- **应用处理器（AP）**：SpacemiT® X60™ RISC-V 双簇 8 核处理器
  - 遵循 RISC-V 64GCVB 架构与 RVA22 标准
  - **Cluster 0**：4 核，集成 2.0 TOPS AI 计算能力（通过 RISC-V 自定义指令实现 CPU-AI 融合计算）
    - 每核 32KB L1 指令 cache + 32KB L1 数据 cache
    - 512KB L2 cache + 512KB TCM（紧耦合内存，用于 AI 扩展）
    - 256-bit 向量扩展（RVV1.0，VLEN 256/128-bit，双发宽度）
  - **Cluster 1**：4 核（不带 AI 扩展）
    - 每核 32KB L1 指令 cache + 32KB L1 数据 cache
    - 512KB L2 cache + 256-bit 向量扩展
  - DVFS 自适应电压 0.6V ~ 1.05V
- **GPU**：Imagination IMG BXE-2-32 @819MHz
  - 32KB SLC，支持 OpenGL ES 1.1/3.2、Vulkan 1.3、OpenCL 3.0
  - 20 GFLOPS（FP32），TBDR 架构，最多 8 个虚拟 GPU
- **VPU**：硬件视频编解码
  - H.265/H.264/VP8/VP9/MPEG4/MPEG2 解码 4K@60fps
  - H.265/H.264/VP8/VP9 编码 4K@30fps
  - 1080P@60fps 同步编解码
- **图像子系统**：双 ISP + 双 MIPI CSI
  - 16M@30fps 双 ISP 处理能力
  - 硬件 JPEG 编解码（最大 23M）
  - AF/AE/AWB、人脸检测、PDAF、PiP、连续视频 AF、硬件 3D 降噪
- **AI 计算扩展**（Cluster 0）：4 类 RISC-V 自定义指令
  - `smt.vmadot` / `smt.vmadotu` / `smt.vmadotsu` / `smt.vmadotus`（8 位整数点乘矩阵乘累加）
  - `smt.vmadot1` ~ `smt.vmadot3` 系列（8 位整数滑窗点乘矩阵乘累加）
  - 详细规范参见 [riscv-ime-extension-spec](https://github.com/spacemit-com/riscv-ime-extension-spec)
- **存储子系统**：
  - 128KB boot-ROM
  - 256KB SRAM（AP 与 RCPU 共享）
  - DDR 控制器：LPDDR4/LPDDR4x @ 2666Mbps（最大 16GB）/ LPDDR3 @ 1866Mbps（最大 4GB），32-bit 数据宽度，双 Chip Select
- **外设控制器**：
  - GPIO × 128（104× 1.8V IO + 24× 1.8V/3.3V IO，上下拉可编程）
  - UART × 10（AP/BT/print）
  - I2C × 10（8× AP_I2C + 1× HDMI I2C + 1× PWR I2C，用于摄像头、G-Sensor、罗盘、接近、光感、陀螺仪、指纹、NFC、PMIC、触摸等）
  - SPI × 4（1× QSPI + 1× SPI LCD + 2× SPI，支持主从模式）
  - USB × 3（USB 2.0 OTG + USB 2.0 Host + USB 3.0 combo PCIe PortA）
  - PCIe 2.1 × 3（PortA Gen2x1 + PortB Gen2x2 + PortC Gen2x2）
  - GMAC × 2（10/100/1000 Mbps，RGMII）
  - SDIO × 1（WiFi，4-bit SDIO 3.0 UHS-I，最高 SDR104 208MHz）
  - SD × 1（TF 卡，4-bit SD 3.0 UHS-I，最高 SDR104 208MHz）
  - eMMC × 1（8-bit eMMC 5.1，最高 HS400 200MHz）
  - MIPI CSI（CSI-2 v1.1）4-Lane × 2（支持 4+4 / 4+2 / 4+2+2 多传感器模式）
  - MIPI DSI（DSI v1.1）4-Lane × 1
  - PWM × 20
  - CAN-FD × 1
  - IR-RX × 1
- **RCPU（实时 CPU）**：独立实时核
  - SRAM 256KB
  - R_CAN-FD × 1、R_I2C × 1、R_SPI × 2、HDMI Audio、R_UART × 2、R_PWM × 10、DMA × 1、R_IR_RX × 1、R_Debug
- **安全子系统**：
  - RISC-V PMP 安全、Secure Boot、Secure eFuse 4K bits
  - 密码学引擎（TRNG、AES、RSA、ECC、SHA2、HMAC）
- **调试系统**：双 JTAG（CPU/MCU 各一）、UART、看门狗复位后 CPU/IO 寄存器快照
- **启动系统**：128KB boot-ROM，支持 SPI-Nand / SPI-Nor Flash / eMMC / SD 启动
- **多媒体显示**：
  - 1× MIPI DSI-4 lane 或 SPI 接口，最大 HD+（1920×1080@60fps）
  - 4 全尺寸图层 composer，通过 RDMA 通道上下复用最大 8 层
  - 支持 AFBC 压缩、cmdlist 机制、write-back、dither/crop/rotation
  - HDMI 1.4
- **工作温度**：-40°C ~ +85°C（工业级）
- **TDP**：3 ~ 5W

### MUSE Pi Pro 开发板

**MUSE Pi Pro** 是 SpacemiT 官方推出的基于 K1（M1 模组）的 RISC-V 单板计算机，尺寸 85×56mm，兼容 Raspberry Pi 4 排针布局，面向 RISC-V 开发者生态与边缘 AI 推理场景。主要规格：

- **SoC**：SpacemiT M1（K1）8 核 RISC-V @ 1.6/1.8 GHz
- **内存**：8GB / 16GB LPDDR4X @ 2400MT/s
- **存储**：
  - 板载 64GB / 128GB eMMC 5.1
  - M.2 M-Key 2230 NVMe SSD 插槽
  - microSD UHS-II 卡槽
- **网络**：
  - 千兆以太网 RJ45（RGMII）
  - 板载 Wi-Fi 6 + Bluetooth 5.2/5.3
- **USB**：4× USB 3.0 Type-A（host）+ 1× USB 2.0 Type-C（device/OTG）
- **显示**：HDMI 1.4（1080P@60Hz）+ 4-lane MIPI DSI FPC（1080P@60Hz）
- **摄像头**：4-lane MIPI CSI FPC
- **扩展**：
  - 40-pin GPIO 排针（兼容 Raspberry Pi HAT）
  - 全尺寸 miniPCIe 插槽（4G/5G/无线模块）
  - 第二 M.2 M-Key 2230 插槽（SSD/PCIe-to-SATA/通信模块）
- **时钟**：板载 RTC + 电池供电
- **电源**：USB-C PD（5V/3A、9V/3A、12V/3A）
- **按钮**：PWR / RST / FDL（以太网口朝上时自上而下）
- **尺寸**：85 × 56mm（信用卡大小）

> 完整硬件说明、原理图与官方上手指南，请参考 SpacemiT 官方文档：
>
> - [K1 Datasheet (PDF)](https://cdn-resource.spacemit.com/file/chip/K1/K1_datasheet_en.pdf)
> - [K1 Datasheet (Markdown)](https://github.com/spacemit-com/docs-chip/blob/main/en/key_stone/k1/k1_docs/k1_ds.md)
> - [SpacemiT 开发者文档站](https://developer.spacemit.com/)
> - [SpacemiT 固件归档](https://archive.spacemit.com/)

## 仓库结构（目标）

```
vendor/spacemit/
├── chips/                 # K1 芯片级驱动与硬件抽象（待合入）
│   └── k1/
│       ├── drivers/       # 底层 HAL 与 RTOS 适配层
│       └── drv/           # NuttX 驱动实现层
├── boards/                # 板级支持包
│   └── k1/
│       └── muse_pi_pro/  # MUSE Pi Pro 开发板（本仓库首块适配板）
│           ├── Kconfig    # 板级 Kconfig 选项
│           ├── include/   # 板级头文件（board.h、内存映射等）
│           ├── src/       # 板级 bring-up 源码（启动、初始化、LED 等）
│           ├── scripts/   # 链接脚本与构建规则
│           └── configs/   # 编译配置（nsh 等）
├── Make.defs              # 全局构建规则（待合入）
├── Kconfig                # 全局配置入口（待合入）
└── README.md / README_zh-cn.md
```

> 当前分支仅包含 `.gitee/` 与 `.github/`（Issue/PR 模板与 CI workflows），`chips/` 与 `boards/` 的 openvela 适配代码将在后续 PR 中陆续合入。

## 核心特性

### RISC-V 计算与 AI 融合
- **8 核 X60™ RISC-V**：双簇架构，Cluster 0 带 2.0 TOPS AI 扩展，Cluster 1 为通用计算核
- **RVA22 标准 + RVV1.0**：256-bit 向量扩展，SIMD 并行处理能力是 ARM Neon 的 2 倍
- **CPU-AI 融合计算**：Cluster 0 通过 `smt.vmadot*` 系列 RISC-V 自定义指令直接在 CPU 上执行 INT8 矩阵乘累加，无需独立 NPU 即可达到 2.0 TOPS 算力
- **AI 框架支持**：TensorFlow Lite、TensorFlow、ONNX Runtime

### 图形与视频
- **IMG BXE-2-32 GPU**：Vulkan 1.3 / OpenGL ES 3.2 / OpenCL 3.0，TBDR 架构，支持 8 虚拟 GPU
- **4K@60fps 视频编解码**：H.265/H.264/VP8/VP9 硬件编解码
- **HD+ 显示**：MIPI DSI 4-lane 或 SPI，最大 1920×1080@60fps，支持 AFBC 压缩、write-back、dither/crop/rotation

### 摄像头与图像处理
- **双 ISP**：16M@30fps 双 RAW 流处理
- **双 MIPI CSI-2**：支持 4+4 / 4+2 / 4+2+2 多传感器配置
- **硬件 JPEG 编解码**：最大 23M
- **ISP 功能**：AF/AE/AWB、人脸检测、PDAF、PiP、连续视频 AF、硬件 3D 降噪

### 实时核（RCPU）
- 独立 256KB SRAM 实时核，与 AP 异构
- R_CAN-FD / R_I2C / R_SPI × 2 / R_UART × 2 / R_PWM × 10 / DMA / R_IR_RX
- HDMI 音频通道

### 安全与启动
- **Secure Boot** + Secure eFuse 4K bits
- **密码学引擎**：TRNG / AES / RSA / ECC / SHA2 / HMAC
- **多源启动**：SPI-Nand / SPI-Nor Flash / eMMC / SD
- **128KB boot-ROM**

## 与 openvela 集成

openvela 是面向 AIoT 的嵌入式实时操作系统，已广泛应用于智能手表、智能音箱、耳机、智能家居与机器人等设备。本仓库通过以下方式将 SpacemiT K1 平台集成进 openvela：

1. **Kconfig 集成**：芯片与板级配置可通过 `menuconfig` 访问（代码合入后）
2. **构建系统**：与 NuttX / openvela 构建基础设施集成
3. **设备驱动**：标准 NuttX 驱动接口
4. **RTOS 支持**：完整 POSIX 兼容，支持 SV39 虚拟内存、32 PMP 安全条目

## 编译

> 🚧 **待合入**
>
> 板级 `configs/` defconfig 与 `chips/k1/` 驱动代码合入前，本仓库暂无可编译的 openvela 配置。
>
> 代码合入后，编译入口示例（以 nsh 配置为例）：
>
> ```bash
> ./build.sh vendor/spacemit/boards/k1/muse_pi_pro/configs/nsh -j8
> ```
>
> 详见 [boards/k1/muse_pi_pro/README_zh-cn.md](boards/k1/muse_pi_pro/README_zh-cn.md)。

## 烧录与启动

K1 SoC 的 BootROM 要求一级 bootloader（FSBL）必须签名。烧录分两步：先把 bootloader 与系统镜像写入 SPI NOR flash 或 eMMC，再正常上电启动。

### 进入 BootROM Fastboot 模式

1. 拔掉 USB-C 电缆断电
2. **按住 FDL 按钮**（USB-A 口侧自上而下第三个按钮）
3. 用 USB-C 电缆连接 J15 OTG 口到主机
4. 释放 FDL 按钮

主机应识别到 fastboot 设备：

```bash
$ sudo fastboot devices
dfu-device       DFU download
```

### 烧录工具

SpacemiT 提供三种烧录方式，任选其一：

| 工具 | 形态 | 适用场景 |
|------|------|----------|
| `titanflasher` | GUI | Windows / Linux 桌面图形工具，新手友好 |
| `flashserver` | CLI | `fastboot` 改版，支持 SpacemiT 私有协议，CI 自动化推荐 |
| `fastboot` | CLI | 主线 Android `fastboot`（35.0.1+），部分场景需配合 `flashserver` |

下载地址：
- `titanflasher`：https://archive.spacemit.com/tools/titanflasher/
- `flashserver`：从 `titanflasher` AppImage 中提取（`./titantools_for_linux-*.AppImage --appimage-extract resources/app/flashserver`）
- `fastboot`：发行版 `android-tools-fastboot` 包或 [Google SDK](https://developer.android.com/tools/releases/platform-tools)

### 典型烧录命令（fastboot）

```bash
# 烧录 FSBL（需先用 SpacemiT 的 fsbl.sh 脚本对 SPL 签名生成）
sudo fastboot stage factory/FSBL.bin
sudo fastboot continue
sleep 1

# 烧录 U-Boot / OpenSBI
sudo fastboot stage u-boot.itb
sudo fastboot continue
sleep 1

# 烧录分区表与各 mtd 分区
sudo fastboot flash mtd partition_2M.json
sudo fastboot flash mtd-bootinfo factory/bootinfo_spinor.bin
sudo fastboot flash mtd-fsbl factory/FSBL.bin
sudo fastboot flash mtd-env env.bin
sudo fastboot flash mtd-opensbi fw_dynamic.itb
sudo fastboot flash mtd-uboot u-boot.itb
```

### 启动顺序

U-Boot 环境默认按以下顺序探测启动设备：

1. USB（USB 0:1）
2. SD 卡（mmc 0）
3. NVMe（nvme 扫描分区）
4. MMC（mmc 1 / mmc 2）

启动方法依次尝试 extlinux → U-Boot script → UEFI bootefi。

## 硬件购买与咨询

MUSE Pi Pro 开发板由 SpacemiT 官方直售，价格约 $122 ~ $138（8GB / 16GB 配置差异），可通过以下渠道购买与咨询：

- **SpacemiT 官网**：https://www.spacemit.com/
- **SpacemiT 开发者文档**：https://developer.spacemit.com/
- **SpacemiT 固件归档**：https://archive.spacemit.com/
- **SpacemiT GitHub 组织**：https://github.com/spacemit-com

## 许可协议

本目录下文件遵循各自文件头部声明的许可协议。openvela 主体代码遵循 Apache-2.0；SpacemiT 提供的固件二进制（如 DDR training firmware、FSBL 签名工具）请遵循 SpacemiT 官方许可条款。

## 贡献

- **SpacemiT 平台特定问题**：请遵循 SpacemiT 官方开发者流程，参考 [developer.spacemit.com](https://developer.spacemit.com/)
- **openvela 集成问题**：请参考 [openvela 贡献指南](../../docs/CONTRIBUTING.md)

## 相关资源

- [K1 Datasheet (PDF)](https://cdn-resource.spacemit.com/file/chip/K1/K1_datasheet_en.pdf)
- [K1 Datasheet (Markdown)](https://github.com/spacemit-com/docs-chip/blob/main/en/key_stone/k1/k1_docs/k1_ds.md)
- [K1 Block Diagram 与 X60™ 微架构](https://github.com/spacemit-com/docs-chip/tree/main/en/key_stone/k1/k1_docs)
- [RISC-V IME Extension Spec（AI 自定义指令规范）](https://github.com/spacemit-com/riscv-ime-extension-spec)
- [SpacemiT 固件归档（含 Bianbu / UEFI / titanflasher）](https://archive.spacemit.com/)
- [DDR Training Firmware](https://github.com/spacemit-com/spacemit-firmware/tree/master/k1/v0.2)
- [openvela 文档](../../docs/)
- [openvela AI 硬件开发者大赛](https://openvela.com/contest)
