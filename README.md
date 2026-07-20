# SpacemiT Chip Vendor Support for openvela

\[ English | [简体中文](README_zh-cn.md) \]

## Introduction

This repository contains the board support packages (BSP), chip drivers, and
product projects from **SpacemiT** (进迭时空) for the openvela operating
system. SpacemiT is a RISC-V processor semiconductor company focused on
high-performance computing, building on its self-developed X60™ RISC-V CPU
core to deliver complete platform solutions for AIoT, edge computing and the
developer ecosystem.

This repository currently serves the **openvela AI Hardware Developer
Contest**, with `dev-ai-contest-2026` as the primary branch.

> ⚠️ **Repository status: early bootstrap stage**
>
> This repository is in the early bootstrap stage on the contest branch
> `dev-ai-contest-2026`: the chip-side (`chips/k1/`) and board-side
> (`boards/k1/muse_pi_pro/`) openvela adaptation code is being prepared and
> will land in subsequent PRs. This README describes the **target support
> state** and hardware reference specs; build commands and device-node tables
> will be filled in once the adaptation code is merged.

## Supported Hardware

### SpacemiT Key Stone® K1 SoC

**SpacemiT K1** is a high-performance, ultra-low-power SoC for AIoT and edge
computing, integrating 8 RISC-V CPU cores with SpacemiT® Daoyi™ AI computing
capability. Key features:

- **Application Processor (AP)**: SpacemiT® X60™ RISC-V dual-cluster 8-core
  - RISC-V 64GCVB architecture, RVA22 profile
  - **Cluster 0**: quad-core with 2.0 TOPS AI computing power (CPU-AI fusion
    via RISC-V customized instructions)
    - 32KB L1-I cache + 32KB L1-D cache per core
    - 512KB L2 cache + 512KB TCM (Tight-Coupled Memory, for AI extension)
    - 256-bit vector extension (RVV1.0, VLEN 256/128-bit, dual-issue)
  - **Cluster 1**: quad-core (without AI extension)
    - 32KB L1-I cache + 32KB L1-D cache per core
    - 512KB L2 cache + 256-bit vector extension
  - DVFS adaptive voltage 0.6V ~ 1.05V
- **GPU**: Imagination IMG BXE-2-32 @819MHz
  - 32KB SLC, OpenGL ES 1.1/3.2, Vulkan 1.3, OpenCL 3.0
  - 20 GFLOPS (FP32), TBDR architecture, up to 8 virtual GPUs
- **VPU**: hardware video codec
  - H.265/H.264/VP8/VP9/MPEG4/MPEG2 decode @ 4K@60fps
  - H.265/H.264/VP8/VP9 encode @ 4K@30fps
  - Simultaneous encode/decode @ 1080P@60fps
- **Image subsystem**: dual ISP + dual MIPI CSI
  - 16M@30fps dual-ISP throughput
  - Hardware JPEG codec (up to 23M)
  - AF/AE/AWB, face detection, PDAF, PiP, continuous video AF, HW 3D denoise
- **AI computing extension** (Cluster 0): 4 categories of RISC-V customized instructions
  - `smt.vmadot` / `smt.vmadotu` / `smt.vmadotsu` / `smt.vmadotus`
    (8-bit integer dot-product matrix multiply-accumulate)
  - `smt.vmadot1` ~ `smt.vmadot3` series (8-bit integer sliding-window
    dot-product matrix multiply-accumulate)
  - Spec: [riscv-ime-extension-spec](https://github.com/spacemit-com/riscv-ime-extension-spec)
- **Memory subsystem**:
  - 128KB boot-ROM
  - 256KB SRAM shared between AP and RCPU
  - DDR controller: LPDDR4/LPDDR4x @ 2666Mbps (max 16GB) / LPDDR3 @ 1866Mbps
    (max 4GB), 32-bit data width, dual Chip Select
- **Peripheral controllers**:
  - GPIO × 128 (104× 1.8V IO + 24× 1.8V/3.3V IO, programmable pull)
  - UART × 10 (AP/BT/print)
  - I2C × 10 (8× AP_I2C + 1× HDMI I2C + 1× PWR I2C)
  - SPI × 4 (1× QSPI + 1× SPI LCD + 2× SPI, master/slave)
  - USB × 3 (USB 2.0 OTG + USB 2.0 Host + USB 3.0 combo PCIe PortA)
  - PCIe 2.1 × 3 (PortA Gen2x1 + PortB Gen2x2 + PortC Gen2x2)
  - GMAC × 2 (10/100/1000 Mbps, RGMII)
  - SDIO × 1 (WiFi, 4-bit SDIO 3.0 UHS-I, up to SDR104 208MHz)
  - SD × 1 (TF card, 4-bit SD 3.0 UHS-I, up to SDR104 208MHz)
  - eMMC × 1 (8-bit eMMC 5.1, up to HS400 200MHz)
  - MIPI CSI (CSI-2 v1.1) 4-Lane × 2 (4+4 / 4+2 / 4+2+2 multi-sensor)
  - MIPI DSI (DSI v1.1) 4-Lane × 1
  - PWM × 20
  - CAN-FD × 1
  - IR-RX × 1
- **RCPU (Real-Time CPU)**: independent real-time core
  - SRAM 256KB
  - R_CAN-FD × 1, R_I2C × 1, R_SPI × 2, HDMI Audio, R_UART × 2,
    R_PWM × 10, DMA × 1, R_IR_RX × 1, R_Debug
- **Security subsystem**:
  - RISC-V PMP security, Secure Boot, Secure eFuse 4K bits
  - Cryptographic engine (TRNG, AES, RSA, ECC, SHA2, HMAC)
- **Debug system**: dual JTAG (one for CPU, one for MCU), UARTs,
  CPU/IO register snapshot after watchdog reboot
- **Boot system**: 128KB boot-ROM, boots from SPI-Nand / SPI-Nor Flash /
  eMMC / SD
- **Multimedia display**:
  - 1× MIPI DSI-4 lane or SPI, up to HD+ (1920×1080@60fps)
  - 4 full-size layer composer, up to 8 layers via RDMA up-down reuse
  - AFBC compression, cmdlist mechanism, write-back, dither/crop/rotation
  - HDMI 1.4
- **Operating temperature**: -40°C ~ +85°C (industrial-grade)
- **TDP**: 3 ~ 5W

### MUSE Pi Pro Development Board

**MUSE Pi Pro** is the official RISC-V single-board computer from SpacemiT
based on the K1 (M1 module). Measuring 85×56mm, it follows the Raspberry Pi 4
header layout, targeting RISC-V developer ecosystem and edge AI inference
scenarios. Main specifications:

- **SoC**: SpacemiT M1 (K1) 8-core RISC-V @ 1.6/1.8 GHz
- **Memory**: 8GB / 16GB LPDDR4X @ 2400MT/s
- **Storage**:
  - Onboard 64GB / 128GB eMMC 5.1
  - M.2 M-Key 2230 NVMe SSD slot
  - microSD UHS-II card slot
- **Networking**:
  - Gigabit Ethernet RJ45 (RGMII)
  - Onboard Wi-Fi 6 + Bluetooth 5.2/5.3
- **USB**: 4× USB 3.0 Type-A (host) + 1× USB 2.0 Type-C (device/OTG)
- **Display**: HDMI 1.4 (1080P@60Hz) + 4-lane MIPI DSI FPC (1080P@60Hz)
- **Camera**: 4-lane MIPI CSI FPC
- **Expansion**:
  - 40-pin GPIO header (Raspberry Pi HAT-compatible)
  - Full-size miniPCIe slot (4G/5G/wireless modules)
  - Second M.2 M-Key 2230 slot (SSD / PCIe-to-SATA / comms modules)
- **Clock**: onboard RTC with battery backup
- **Power**: USB-C PD (5V/3A, 9V/3A, 12V/3A)
- **Buttons**: PWR / RST / FDL (top to bottom when Ethernet port faces up)
- **Dimensions**: 85 × 56mm (credit-card sized)

> For full hardware documentation, schematic and the official getting-started
> guide, see the SpacemiT official docs:
>
> - [K1 Datasheet (PDF)](https://cdn-resource.spacemit.com/file/chip/K1/K1_datasheet_en.pdf)
> - [K1 Datasheet (Markdown)](https://github.com/spacemit-com/docs-chip/blob/main/en/key_stone/k1/k1_docs/k1_ds.md)
> - [SpacemiT Developer Documentation](https://developer.spacemit.com/)
> - [SpacemiT Firmware Archive](https://archive.spacemit.com/)

## Repository Structure (target)

```
vendor/spacemit/
├── chips/                 # K1 chip-level drivers and HAL (to be merged)
│   └── k1/
│       ├── drivers/       # low-level HAL and RTOS adaptation
│       └── drv/           # NuttX driver implementation
├── boards/                # Board support packages
│   └── k1/
│       └── muse_pi_pro/  # MUSE Pi Pro (first board in this repo)
│           ├── Kconfig    # board-level Kconfig options
│           ├── include/   # board headers (board.h, memory map, etc.)
│           ├── src/        # board bring-up sources (boot, init, LEDs)
│           ├── scripts/   # link scripts and build rules
│           └── configs/   # build configs (nsh, etc.)
├── Make.defs              # global build rules (to be merged)
├── Kconfig                # global config entry (to be merged)
└── README.md / README_zh-cn.md
```

> The current branch only contains `.gitee/` and `.github/` (Issue/PR
> templates and CI workflows); the openvela adaptation code under `chips/`
> and `boards/` will land in subsequent PRs.

## Core Features

### RISC-V Computing & AI Fusion
- **8-core X60™ RISC-V**: dual-cluster, Cluster 0 with 2.0 TOPS AI extension,
  Cluster 1 for general-purpose compute
- **RVA22 profile + RVV1.0**: 256-bit vector extension, 2× SIMD parallel
  throughput vs ARM Neon
- **CPU-AI fusion**: Cluster 0 executes INT8 matrix multiply-accumulate
  directly on the CPU via `smt.vmadot*` RISC-V customized instructions,
  reaching 2.0 TOPS without a discrete NPU
- **AI frameworks**: TensorFlow Lite, TensorFlow, ONNX Runtime

### Graphics & Video
- **IMG BXE-2-32 GPU**: Vulkan 1.3 / OpenGL ES 3.2 / OpenCL 3.0, TBDR, 8 virtual GPUs
- **4K@60fps video codec**: H.265/H.264/VP8/VP9 hardware codec
- **HD+ display**: MIPI DSI 4-lane or SPI, up to 1920×1080@60fps, AFBC
  compression, write-back, dither/crop/rotation

### Camera & Image Processing
- **Dual ISP**: 16M@30fps dual RAW stream
- **Dual MIPI CSI-2**: 4+4 / 4+2 / 4+2+2 multi-sensor configurations
- **Hardware JPEG codec**: up to 23M
- **ISP features**: AF/AE/AWB, face detection, PDAF, PiP, continuous video AF,
  HW 3D denoise

### Real-Time Core (RCPU)
- Independent 256KB SRAM real-time core, asymmetric with AP
- R_CAN-FD / R_I2C / R_SPI × 2 / R_UART × 2 / R_PWM × 10 / DMA / R_IR_RX
- HDMI audio channel

### Security & Boot
- **Secure Boot** + Secure eFuse 4K bits
- **Cryptographic engine**: TRNG / AES / RSA / ECC / SHA2 / HMAC
- **Multi-source boot**: SPI-Nand / SPI-Nor Flash / eMMC / SD
- **128KB boot-ROM**

## Integration with openvela

openvela is an embedded RTOS for AIoT, widely deployed in smart watches,
smart speakers, headphones, smart home devices and robots. This repository
integrates the SpacemiT K1 platform into openvela through:

1. **Kconfig integration**: chip and board configs accessible via
   `menuconfig` (after code merge)
2. **Build system**: integrated with NuttX / openvela build infrastructure
3. **Device drivers**: standard NuttX driver interfaces
4. **RTOS support**: full POSIX compatibility, SV39 virtual memory,
   32 PMP security entries

## Build

> 🚧 **To be merged**
>
> Until the board-level `configs/` defconfig and `chips/k1/` driver code
> land in this repo, there is no buildable openvela configuration yet.
>
> Once the code is merged, the build entry (nsh config as example):
>
> ```bash
> ./build.sh vendor/spacemit/boards/k1/muse_pi_pro/configs/nsh -j8
> ```
>
> See [boards/k1/muse_pi_pro/README.md](boards/k1/muse_pi_pro/README.md).

## Flashing & Boot

The K1 BootROM requires the first-stage bootloader (FSBL) to be signed.
Flashing is a two-step process: write the bootloader and system image to
SPI NOR flash or eMMC, then power on to boot.

### Entering BootROM Fastboot Mode

1. Disconnect the USB-C cable to power off
2. **Press and hold the FDL button** (third button from the top, next to the
   USB-A ports, with the Ethernet port facing up)
3. Connect the USB-C cable from the J15 OTG port to the host
4. Release the FDL button

The host should detect a fastboot device:

```bash
$ sudo fastboot devices
dfu-device       DFU download
```

### Flashing tools

SpacemiT provides three flashing methods, pick one:

| Tool | Form | Use case |
|------|------|----------|
| `titanflasher` | GUI | Windows / Linux desktop GUI, beginner-friendly |
| `flashserver` | CLI | SpacemiT-modified `fastboot`, supports private protocol, CI recommended |
| `fastboot` | CLI | Mainline Android `fastboot` (35.0.1+), some scenarios need `flashserver` |

Downloads:
- `titanflasher`: https://archive.spacemit.com/tools/titanflasher/
- `flashserver`: extract from the `titanflasher` AppImage
  (`./titantools_for_linux-*.AppImage --appimage-extract resources/app/flashserver`)
- `fastboot`: distro `android-tools-fastboot` package or
  [Google SDK](https://developer.android.com/tools/releases/platform-tools)

### Typical flashing commands (fastboot)

```bash
# Flash FSBL (must be signed first with SpacemiT's fsbl.sh)
sudo fastboot stage factory/FSBL.bin
sudo fastboot continue
sleep 1

# Flash U-Boot / OpenSBI
sudo fastboot stage u-boot.itb
sudo fastboot continue
sleep 1

# Flash partition table and mtd partitions
sudo fastboot flash mtd partition_2M.json
sudo fastboot flash mtd-bootinfo factory/bootinfo_spinor.bin
sudo fastboot flash mtd-fsbl factory/FSBL.bin
sudo fastboot flash mtd-env env.bin
sudo fastboot flash mtd-opensbi fw_dynamic.itb
sudo fastboot flash mtd-uboot u-boot.itb
```

### Boot order

The U-Boot environment probes boot devices in this order by default:

1. USB (USB 0:1)
2. SD card (mmc 0)
3. NVMe (scan partitions)
4. MMC (mmc 1 / mmc 2)

Within each device it tries extlinux → U-Boot script → UEFI bootefi.

## Hardware Purchase & Consultation

The MUSE Pi Pro board is sold directly by SpacemiT, priced around
$122 ~ $138 depending on 8GB / 16GB RAM configuration. Available through:

- **SpacemiT official site**: https://www.spacemit.com/
- **SpacemiT developer docs**: https://developer.spacemit.com/
- **SpacemiT firmware archive**: https://archive.spacemit.com/
- **SpacemiT GitHub org**: https://github.com/spacemit-com

## License

Files in this directory follow the license declared in each file header.
The openvela mainline code is Apache-2.0; SpacemiT-provided firmware
binaries (DDR training firmware, FSBL signing tool, etc.) follow SpacemiT's
official licensing terms.

## Contributing

- **SpacemiT platform-specific issues**: follow SpacemiT's developer process
  at [developer.spacemit.com](https://developer.spacemit.com/)
- **openvela integration issues**: see the
  [openvela Contributing Guide](../../docs/CONTRIBUTING.md)

## Related Resources

- [K1 Datasheet (PDF)](https://cdn-resource.spacemit.com/file/chip/K1/K1_datasheet_en.pdf)
- [K1 Datasheet (Markdown)](https://github.com/spacemit-com/docs-chip/blob/main/en/key_stone/k1/k1_docs/k1_ds.md)
- [K1 Block Diagram and X60™ microarchitecture](https://github.com/spacemit-com/docs-chip/tree/main/en/key_stone/k1/k1_docs)
- [RISC-V IME Extension Spec (AI customized instructions)](https://github.com/spacemit-com/riscv-ime-extension-spec)
- [SpacemiT firmware archive (Bianbu / UEFI / titanflasher)](https://archive.spacemit.com/)
- [DDR Training Firmware](https://github.com/spacemit-com/spacemit-firmware/tree/master/k1/v0.2)
- [openvela documentation](../../docs/)
- [openvela AI Hardware Developer Contest](https://openvela.com/contest)
