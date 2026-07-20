# MUSE Pi Pro 开发板对 openvela 的支持

\[ [English](README.md) | 简体中文 \]

## 简介

本目录为 **SpacemiT MUSE Pi Pro** 开发板提供 openvela 板级支持（BSP），基于
`dev-ai-contest-2026` 分支。MUSE Pi Pro 是 SpacemiT 官方推出的基于 K1（M1
模组）的 RISC-V 单板计算机，尺寸 85×56mm，兼容 Raspberry Pi 4 排针布局，
面向 RISC-V 开发者生态与边缘 AI 推理场景。

主要硬件规格（以 SpacemiT 官方文档为准）：

- **SoC**：SpacemiT M1（K1），8 核 RISC-V X60 @ 1.6/1.8 GHz
- **AI 算力**：Cluster 0 提供 2.0 TOPS（INT8），通过 RISC-V 自定义指令实现 CPU-AI 融合计算
- **GPU**：Imagination IMG BXE-2-32，支持 Vulkan 1.3 / OpenGL ES 3.2 / OpenCL 3.0，20 GFLOPS（FP32）
- **VPU**：H.265/H.264/VP8/VP9 硬件编解码，4K@60fps 解码 / 4K@30fps 编码
- **内存**：8GB / 16GB LPDDR4X @ 2400MT/s
- **存储**：64GB / 128GB eMMC 5.1 + M.2 M-Key 2230 NVMe + microSD UHS-II
- **网络**：千兆以太网 + Wi-Fi 6 + Bluetooth 5.2/5.3
- **显示**：HDMI 1.4（1080P@60）+ 4-lane MIPI DSI FPC
- **摄像头**：4-lane MIPI CSI FPC
- **扩展**：40-pin GPIO（Raspberry Pi HAT 兼容）+ miniPCIe + 第二 M.2 M-Key
- **电源**：USB-C PD（5V/3A、9V/3A、12V/3A）
- **按钮**：PWR / RST / FDL
- **尺寸**：85 × 56mm（信用卡大小）

> 完整硬件说明、原理图与官方上手指南请参考 SpacemiT 官方文档：
>
> - [K1 Datasheet (PDF)](https://cdn-resource.spacemit.com/file/chip/K1/K1_datasheet_en.pdf)
> - [K1 Datasheet (Markdown)](https://github.com/spacemit-com/docs-chip/blob/main/en/key_stone/k1/k1_docs/k1_ds.md)
> - [SpacemiT 开发者文档站](https://developer.spacemit.com/)
> - [SpacemiT 固件归档](https://archive.spacemit.com/)

> ⚠️ **分支依赖**
>
> 本板适配仅在 `open-vela/nuttx` 与 `open-vela/vendor_SpacemiT` 的
> `dev-ai-contest-2026` 分支上可编译。`trunk` 或 `dev` 分支由于尚未合入
> 芯片层依赖，无法编译。

> 🚧 **板级适配代码待合入**
>
> 当前仓库处于大赛分支 `dev-ai-contest-2026` 的初始建设阶段：板级
> `src/`、`include/`、`configs/` 与 `chips/k1/` 驱动代码正在准备中，将在
> 后续 PR 中陆续合入。本 README 描述的是**目标支持状态**与硬件参考规格，
> "目录结构"、"支持的外设"、"GPIO 引脚映射"、"编译"、"首次启动验证"
> 等章节会在适配代码合入后补充实际内容。

## 目录结构（目标）

```
vendor/spacemit/boards/k1/muse_pi_pro/
├── Kconfig            # 板级 Kconfig 选项（待合入）
├── include/           # 板级头文件（board.h、内存映射等，待合入）
├── src/               # 板级 bring-up 源码（启动、初始化、LED 等，待合入）
├── scripts/           # 链接脚本与构建规则（待合入）
└── configs/           # 编译配置
    └── nsh            # 基础 NSH 命令行配置（待合入）
```

## 板子硬件规格

| 项目 | 规格 |
|------|------|
| SoC | SpacemiT M1（K1），8 核 RISC-V X60 @ 1.6/1.8 GHz，双簇（Cluster 0 带 2.0 TOPS AI，Cluster 1 不带 AI） |
| 架构 | RISC-V 64GCVB，RVA22 标准，RVV1.0 VLEN 256/128-bit 双发 |
| GPU | Imagination IMG BXE-2-32 @819MHz，32KB SLC，Vulkan 1.3 / OpenGL ES 3.2 / OpenCL 3.0，20 GFLOPS（FP32） |
| VPU | H.265/H.264/VP8/VP9 解码 4K@60fps，编码 4K@30fps，1080P@60fps 同步编解码 |
| NPU/AI | 2.0 TOPS（INT8），Cluster 0 通过 `smt.vmadot*` RISC-V 自定义指令实现 CPU-AI 融合 |
| ISP | 双 ISP，16M@30fps，硬件 JPEG 编解码（最大 23M），AF/AE/AWB、PDAF、PiP、3D 降噪 |
| 内存 | 8GB / 16GB LPDDR4X @ 2400MT/s（K1 DDR 控制器支持 LPDDR4/LPDDR4x @ 2666Mbps，最大 16GB） |
| 启动 ROM | 128KB boot-ROM |
| SRAM | 256KB（AP 与 RCPU 共享） |
| eMMC | 64GB / 128GB eMMC 5.1（K1 eMMC 控制器支持 8-bit eMMC 5.1，最高 HS400 200MHz） |
| NVMe | M.2 M-Key 2230 NVMe SSD 插槽（K1 PCIe 2.1 PortB Gen2x2 或 PortC Gen2x2） |
| SD 卡 | microSD UHS-II（K1 SD/MMC 控制器支持 4-bit SD 3.0 UHS-I，最高 SDR104 208MHz） |
| USB | 4× USB 3.0 Type-A（host）+ 1× USB 2.0 Type-C（device/OTG） |
| 以太网 | 千兆 RJ45（K1 GMAC × 2，10/100/1000 Mbps，RGMII） |
| Wi-Fi/BT | 板载 Wi-Fi 6 + Bluetooth 5.2/5.3（通过 SDIO × 1 + UART） |
| HDMI | HDMI 1.4，1080P@60Hz |
| MIPI DSI | 4-lane MIPI DSI FPC（DSI v1.1），1080P@60Hz |
| MIPI CSI | 4-lane MIPI CSI FPC（CSI-2 v1.1） |
| 40-pin GPIO | Raspberry Pi HAT 兼容，3.3V IO 电平 |
| miniPCIe | 全尺寸 miniPCIe 插槽（4G/5G/无线模块） |
| 第二 M.2 | M.2 M-Key 2230 插槽（SSD / PCIe-to-SATA / 通信模块） |
| RTC | 板载 RTC + 电池供电 |
| 电源 | USB-C PD（5V/3A、9V/3A、12V/3A） |
| 按钮 | PWR / RST / FDL（以太网口朝上时自上而下） |
| 尺寸 | 85 × 56mm（信用卡大小） |
| 工作温度 | -40°C ~ +85°C（工业级，K1 SoC 规格） |
| TDP | 3 ~ 5W（K1 SoC 规格） |

## K1 SoC 外设控制器总览

K1 SoC 提供以下外设控制器，板级适配代码合入后 openvela 可按需启用：

| 类别 | 数量 / 规格 | 备注 |
|------|------------|------|
| GPIO | × 128 | 104× 1.8V IO + 24× 1.8V/3.3V IO，上下拉可编程 |
| UART | × 10 | AP / BT / print |
| I2C | × 10 | 8× AP_I2C（I2C0/1/7 摄像头专用）+ 1× HDMI I2C + 1× PWR I2C |
| SPI | × 4 | 1× QSPI + 1× SPI LCD + 2× SPI，支持主从模式 |
| USB | × 3 | USB 2.0 OTG + USB 2.0 Host + USB 3.0（combo PCIe PortA） |
| PCIe 2.1 | × 3 | PortA Gen2x1 + PortB Gen2x2 + PortC Gen2x2 |
| GMAC | × 2 | 10/100/1000 Mbps，RGMII |
| SDIO | × 1 | 4-bit SDIO 3.0 UHS-I，最高 SDR104 208MHz（WiFi） |
| SD | × 1 | 4-bit SD 3.0 UHS-I，最高 SDR104 208MHz（TF 卡） |
| eMMC | × 1 | 8-bit eMMC 5.1，最高 HS400 200MHz |
| MIPI CSI | × 2 | CSI-2 v1.1，4-Lane，支持 4+4 / 4+2 / 4+2+2 多传感器 |
| MIPI DSI | × 1 | DSI v1.1，4-Lane |
| PWM | × 20 | |
| CAN-FD | × 1 | |
| IR-RX | × 1 | |
| HDMI | 1.4 | 1080P@60Hz |
| **RCPU（实时核）** | 独立实时核 | SRAM 256KB、R_CAN-FD × 1、R_I2C × 1、R_SPI × 2、R_UART × 2、R_PWM × 10、DMA × 1、R_IR_RX × 1、HDMI Audio、R_Debug |
| 安全 | Secure Boot + Secure eFuse 4K bits | 密码学引擎：TRNG / AES / RSA / ECC / SHA2 / HMAC |
| 启动 | 128KB boot-ROM | 支持 SPI-Nand / SPI-Nor Flash / eMMC / SD 启动 |
| 调试 | 双 JTAG | CPU 与 MCU 各一，UART，看门狗复位寄存器快照 |

## 40-pin GPIO 引脚映射

MUSE Pi Pro 的 40-pin GPIO 排针兼容 Raspberry Pi HAT 排针布局，3.3V IO
电平。详细引脚网络映射（含 SoC GPIO 编号、复用功能、电源/地引脚）请参考
SpacemiT 官方原理图与 K1 datasheet 的 pinout 章节：

- [K1 Datasheet (PDF) — pinout 章节](https://cdn-resource.spacemit.com/file/chip/K1/K1_datasheet_en.pdf)
- [SpacemiT 开发者文档站](https://developer.spacemit.com/)

> 板级 `include/board.h` 与 `src/` 引脚复用配置代码合入后，本章节将补充
> openvela defconfig 下的具体 GPIO 映射表（参考
> `vendor/sifli/boards/sf32lb52/lckfb_huangshan_pi/README_zh-cn.md` 的格式）。

## 编译

> 🚧 **待板级代码合入**
>
> 板级 `configs/nsh/` defconfig 与 `chips/k1/` 驱动代码合入前，本仓库暂无
> 可编译的 openvela 配置。
>
> 代码合入后，编译入口示例（以 nsh 配置为例）：
>
> ```bash
> # 可选：仅在切换配置或修改 menuconfig 后需要清理
> ./build.sh vendor/spacemit/boards/k1/muse_pi_pro/configs/nsh -j8 distclean
>
> # 编译
> ./build.sh vendor/spacemit/boards/k1/muse_pi_pro/configs/nsh -j8
> ```

## 烧录

K1 SoC 的 BootROM 要求一级 bootloader（FSBL）必须签名。烧录分两步：先把
bootloader 与系统镜像写入 SPI NOR flash 或 eMMC，再正常上电启动。

### 进入 BootROM Fastboot 模式

1. 拔掉 USB-C 电缆断电
2. **按住 FDL 按钮**（USB-A 口侧自上而下第三个按钮，以太网口朝上时）
3. 用 USB-C 电缆连接 J15 OTG 口到主机
4. 释放 FDL 按钮

主机应识别到 fastboot 设备：

```bash
$ sudo fastboot devices
dfu-device       DFU download
```

> 如果 `fastboot` 连不上，确认是否已将自己加入 `dialout`/`plugdev` 组
> 或使用 `sudo`；也可用 SpacemiT 的 `flashserver` 工具替代主线 fastboot。

### 烧录工具

SpacemiT 提供三种烧录方式，任选其一：

| 工具 | 形态 | 适用场景 | 下载 |
|------|------|----------|------|
| `titanflasher` | GUI | Windows / Linux 桌面图形工具，新手友好 | [archive.spacemit.com/tools/titanflasher/](https://archive.spacemit.com/tools/titanflasher/) |
| `flashserver` | CLI | `fastboot` 改版，支持 SpacemiT 私有协议，CI 自动化推荐 | 从 `titanflasher` AppImage 提取：`./titantools_for_linux-*.AppImage --appimage-extract resources/app/flashserver` |
| `fastboot` | CLI | 主线 Android `fastboot`（35.0.1+），部分场景需配合 `flashserver` | 发行版 `android-tools-fastboot` 包或 [Google SDK](https://developer.android.com/tools/releases/platform-tools) |

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

如需烧录完整系统镜像到 eMMC（参考 Bianbu / openEuler 流程）：

```bash
sudo fastboot flash gpt partition_universal.json
sudo fastboot flash bootinfo factory/bootinfo_sd.bin
sudo fastboot flash fsbl factory/FSBL.bin
sudo fastboot flash env env.bin
sudo fastboot flash opensbi fw_dynamic.itb
sudo fastboot flash uboot u-boot.itb
sudo fastboot flash ESP efi.img
sudo fastboot flash bootfs_linux bootfs_linux.img
sudo fastboot flash rootfs_linux rootfs_linux.ext4
```

### 启动顺序

U-Boot 环境默认按以下顺序探测启动设备：

1. USB（USB 0:1）
2. SD 卡（mmc 0）
3. NVMe（扫描分区）
4. MMC（mmc 1 / mmc 2）

启动方法依次尝试 extlinux → U-Boot script → UEFI bootefi。

## 首次启动 & 快速验证

> 🚧 **待板级代码合入**
>
> openvela 板级适配代码合入前，本仓库暂不能生成可启动的 `nuttx.bin`。
> 下面的启动验证流程适用于**已烧录 SpacemiT 官方 Bianbu / openEuler 系统**
> 的板子，作为硬件功能验证参考。
>
> 代码合入后，本章节将替换为 openvela NSH 启动日志与外设验证命令
> （参考 `vendor/sifli/boards/sf32lb52/lckfb_huangshan_pi/README_zh-cn.md`
> 的"首次启动 & 快速验证"格式）。

### 控制台配置

MUSE Pi Pro 板载 USB-C 电缆同时提供调试 UART（通过 USB-UART 桥接），
波特率 115200 8N1：

```bash
picocom -b 115200 /dev/ttyUSB*
# 或: minicom -D /dev/ttyUSB0 -b 115200 -8 -o
```

按板上 **RST** 键，应当看到 U-Boot / OpenSBI / 内核启动日志。

## 自定义配置

> 🚧 **待板级代码合入**
>
> 代码合入后：
>
> ```bash
> cd openvela/nuttx
> make menuconfig          # 交互式调整
> make savedefconfig
> cp defconfig \
>    ../vendor/spacemit/boards/k1/muse_pi_pro/configs/nsh/defconfig
> ```

## 调试

- **串口日志** —— `syslog`/`printf` 输出到 USB-UART，用 `picocom`/`tio` 抓取
- **USB-JTAG GDB** —— K1 提供双 JTAG（CPU 与 MCU 各一），用 `openocd` + RISC-V GDB 连接
- **崩溃分析** —— 保存完整寄存器 / 堆栈 dump，K1 SoC 支持看门狗复位后 CPU/IO 寄存器快照，便于调试

## 已知限制

1. **板级适配代码待合入**：当前仓库 `chips/k1/` 与 `boards/k1/muse_pi_pro/`
   下尚无 openvela 适配代码，无法直接编译 openvela 镜像。本 README 描述的是
   目标支持状态与硬件参考规格。
2. **FSBL 必须签名**：K1 BootROM 要求一级 bootloader 必须签名，烧录前
   需用 SpacemiT 的 `fsbl.sh` 脚本（依赖 vendor U-Boot 仓库
   `gitee.com/bianbu-linux/uboot-2022.10` 的 `tools/build_binary_file.py`）
   对 SPL 签名生成 `FSBL.bin`。
3. **DDR training firmware**：SPL 集成 DDR training 需要从
   [github.com/spacemit-com/spacemit-firmware/tree/master/k1/v0.2](https://github.com/spacemit-com/spacemit-firmware/tree/master/k1/v0.2)
   下载 `ddr_fw.bin`，构建时通过 `DDR_FW_FILE` 变量传入。
4. **多启动源**：K1 支持从 SPI-Nand / SPI-Nor Flash / eMMC / SD 多源启动，
   烧录时需根据实际启动介质选择对应 `partition_*.json` 分区表。
5. **RISC-V 软件生态仍在建设中**：部分 RISC-V SBC 仅推荐用于无图形/视频
   加速的 headless 场景，图形与视频加速软件支持进度请关注 SpacemiT 官方
   文档与主线 Linux / U-Boot 进展。

## 许可协议

本目录下所有文件均使用 Apache-2.0 协议（SPDX 标识符
`Apache-2.0`）；详见各文件头部声明。SpacemiT 提供的固件二进制
（DDR training firmware、FSBL 签名工具等）遵循 SpacemiT 官方许可条款。
