# MUSE Pi Pro Board Support for openvela

\[ English | [简体中文](README_zh-cn.md) \]

## Introduction

This directory provides openvela board support (BSP) for the **SpacemiT MUSE
Pi Pro** development board, on the `dev-ai-contest-2026` branch. MUSE Pi Pro
is the official RISC-V single-board computer from SpacemiT, based on the K1
(M1 module), measuring 85×56mm, compatible with the Raspberry Pi 4 header
layout, targeting the RISC-V developer ecosystem and edge AI inference
scenarios.

Key hardware specifications (per SpacemiT official documentation):

- **SoC**: SpacemiT M1 (K1), 8-core RISC-V X60 @ 1.6/1.8 GHz
- **AI computing power**: Cluster 0 delivers 2.0 TOPS (INT8), via RISC-V
  customized instructions for CPU-AI fusion
- **GPU**: Imagination IMG BXE-2-32, Vulkan 1.3 / OpenGL ES 3.2 / OpenCL 3.0,
  20 GFLOPS (FP32)
- **VPU**: H.265/H.264/VP8/VP9 hardware codec, 4K@60fps decode / 4K@30fps encode
- **Memory**: 8GB / 16GB LPDDR4X @ 2400MT/s
- **Storage**: 64GB / 128GB eMMC 5.1 + M.2 M-Key 2230 NVMe + microSD UHS-II
- **Networking**: Gigabit Ethernet + Wi-Fi 6 + Bluetooth 5.2/5.3
- **Display**: HDMI 1.4 (1080P@60) + 4-lane MIPI DSI FPC
- **Camera**: 4-lane MIPI CSI FPC
- **Expansion**: 40-pin GPIO (Raspberry Pi HAT-compatible) + miniPCIe + second M.2 M-Key
- **Power**: USB-C PD (5V/3A, 9V/3A, 12V/3A)
- **Buttons**: PWR / RST / FDL
- **Dimensions**: 85 × 56mm (credit-card sized)

> For full hardware documentation, schematic and the official getting-started
> guide, see the SpacemiT official docs:
>
> - [K1 Datasheet (PDF)](https://cdn-resource.spacemit.com/file/chip/K1/K1_datasheet_en.pdf)
> - [K1 Datasheet (Markdown)](https://github.com/spacemit-com/docs-chip/blob/main/en/key_stone/k1/k1_docs/k1_ds.md)
> - [SpacemiT Developer Documentation](https://developer.spacemit.com/)
> - [SpacemiT Firmware Archive](https://archive.spacemit.com/)

> ⚠️ **Branch dependency**
>
> This board overlay only builds on the `dev-ai-contest-2026` branch of
> `open-vela/nuttx` and `open-vela/vendor_SpacemiT`. Building it from
> `trunk` or `dev` will fail because the chip-side dependencies are not
> yet upstream.

> 🚧 **Board adaptation code pending merge**
>
> This repository is currently in the early bootstrap stage on the contest
> branch `dev-ai-contest-2026`: the board-side `src/`, `include/`,
> `configs/` and `chips/k1/` driver code is being prepared and will land
> in subsequent PRs. This README describes the **target support state** and
> hardware reference specs; the "Directory structure", "Supported
> peripherals", "GPIO pinout", "Build", "First boot verification" sections
> will be filled in once the adaptation code is merged.

## Directory Structure (target)

```
vendor/spacemit/boards/k1/muse_pi_pro/
├── Kconfig            # board-level Kconfig options (to be merged)
├── include/           # board headers (board.h, memory map, etc., to be merged)
├── src/               # board bring-up sources (boot, init, LEDs, etc., to be merged)
├── scripts/           # link scripts and build rules (to be merged)
└── configs/           # build configs
    └── nsh            # basic NSH command-line config (to be merged)
```

## Board Hardware Specifications

| Item | Spec |
|------|------|
| SoC | SpacemiT M1 (K1), 8-core RISC-V X60 @ 1.6/1.8 GHz, dual-cluster (Cluster 0 with 2.0 TOPS AI, Cluster 1 without AI) |
| Architecture | RISC-V 64GCVB, RVA22 profile, RVV1.0 VLEN 256/128-bit dual-issue |
| GPU | Imagination IMG BXE-2-32 @819MHz, 32KB SLC, Vulkan 1.3 / OpenGL ES 3.2 / OpenCL 3.0, 20 GFLOPS (FP32) |
| VPU | H.265/H.264/VP8/VP9 decode 4K@60fps, encode 4K@30fps, 1080P@60fps simultaneous codec |
| NPU/AI | 2.0 TOPS (INT8), Cluster 0 via `smt.vmadot*` RISC-V customized instructions for CPU-AI fusion |
| ISP | Dual ISP, 16M@30fps, HW JPEG codec (up to 23M), AF/AE/AWB, PDAF, PiP, 3D denoise |
| Memory | 8GB / 16GB LPDDR4X @ 2400MT/s (K1 DDR controller supports LPDDR4/LPDDR4x @ 2666Mbps, max 16GB) |
| Boot ROM | 128KB boot-ROM |
| SRAM | 256KB (shared between AP and RCPU) |
| eMMC | 64GB / 128GB eMMC 5.1 (K1 eMMC controller supports 8-bit eMMC 5.1, up to HS400 200MHz) |
| NVMe | M.2 M-Key 2230 NVMe SSD slot (K1 PCIe 2.1 PortB Gen2x2 or PortC Gen2x2) |
| SD card | microSD UHS-II (K1 SD/MMC controller supports 4-bit SD 3.0 UHS-I, up to SDR104 208MHz) |
| USB | 4× USB 3.0 Type-A (host) + 1× USB 2.0 Type-C (device/OTG) |
| Ethernet | Gigabit RJ45 (K1 GMAC × 2, 10/100/1000 Mbps, RGMII) |
| Wi-Fi/BT | Onboard Wi-Fi 6 + Bluetooth 5.2/5.3 (via SDIO × 1 + UART) |
| HDMI | HDMI 1.4, 1080P@60Hz |
| MIPI DSI | 4-lane MIPI DSI FPC (DSI v1.1), 1080P@60Hz |
| MIPI CSI | 4-lane MIPI CSI FPC (CSI-2 v1.1) |
| 40-pin GPIO | Raspberry Pi HAT-compatible, 3.3V IO level |
| miniPCIe | Full-size miniPCIe slot (4G/5G/wireless modules) |
| Second M.2 | M.2 M-Key 2230 slot (SSD / PCIe-to-SATA / comms modules) |
| RTC | Onboard RTC with battery backup |
| Power | USB-C PD (5V/3A, 9V/3A, 12V/3A) |
| Buttons | PWR / RST / FDL (top to bottom when Ethernet port faces up) |
| Dimensions | 85 × 56mm (credit-card sized) |
| Operating temperature | -40°C ~ +85°C (industrial-grade, K1 SoC spec) |
| TDP | 3 ~ 5W (K1 SoC spec) |

## K1 SoC Peripheral Controller Overview

The K1 SoC provides the following peripheral controllers; the openvela
adaptation can enable them on demand once the board-level code is merged:

| Category | Count / Spec | Notes |
|----------|--------------|-------|
| GPIO | × 128 | 104× 1.8V IO + 24× 1.8V/3.3V IO, programmable pull |
| UART | × 10 | AP / BT / print |
| I2C | × 10 | 8× AP_I2C (I2C0/1/7 camera-dedicated) + 1× HDMI I2C + 1× PWR I2C |
| SPI | × 4 | 1× QSPI + 1× SPI LCD + 2× SPI, master/slave |
| USB | × 3 | USB 2.0 OTG + USB 2.0 Host + USB 3.0 (combo PCIe PortA) |
| PCIe 2.1 | × 3 | PortA Gen2x1 + PortB Gen2x2 + PortC Gen2x2 |
| GMAC | × 2 | 10/100/1000 Mbps, RGMII |
| SDIO | × 1 | 4-bit SDIO 3.0 UHS-I, up to SDR104 208MHz (WiFi) |
| SD | × 1 | 4-bit SD 3.0 UHS-I, up to SDR104 208MHz (TF card) |
| eMMC | × 1 | 8-bit eMMC 5.1, up to HS400 200MHz |
| MIPI CSI | × 2 | CSI-2 v1.1, 4-Lane, supports 4+4 / 4+2 / 4+2+2 multi-sensor |
| MIPI DSI | × 1 | DSI v1.1, 4-Lane |
| PWM | × 20 | |
| CAN-FD | × 1 | |
| IR-RX | × 1 | |
| HDMI | 1.4 | 1080P@60Hz |
| **RCPU (real-time core)** | independent RT core | SRAM 256KB, R_CAN-FD × 1, R_I2C × 1, R_SPI × 2, R_UART × 2, R_PWM × 10, DMA × 1, R_IR_RX × 1, HDMI Audio, R_Debug |
| Security | Secure Boot + Secure eFuse 4K bits | Cryptographic engine: TRNG / AES / RSA / ECC / SHA2 / HMAC |
| Boot | 128KB boot-ROM | Boots from SPI-Nand / SPI-Nor Flash / eMMC / SD |
| Debug | Dual JTAG | One for CPU, one for MCU, UARTs, watchdog-reboot register snapshot |

## 40-pin GPIO Pinout

The MUSE Pi Pro 40-pin GPIO header is compatible with the Raspberry Pi HAT
header layout, 3.3V IO level. For detailed pin-net mapping (SoC GPIO numbers,
alternate functions, power/ground pins), refer to the SpacemiT official
schematic and the pinout chapter of the K1 datasheet:

- [K1 Datasheet (PDF) — pinout chapter](https://cdn-resource.spacemit.com/file/chip/K1/K1_datasheet_en.pdf)
- [SpacemiT Developer Documentation](https://developer.spacemit.com/)

> Once the board-level `include/board.h` and `src/` pinmux configuration code
> is merged, this section will be extended with the concrete GPIO mapping
> table under the openvela defconfig (following the format of
> `vendor/sifli/boards/sf32lb52/lckfb_huangshan_pi/README.md`).

## Build

> 🚧 **Pending board-level code merge**
>
> Until the board-level `configs/nsh/` defconfig and `chips/k1/` driver code
> land in this repo, there is no buildable openvela configuration yet.
>
> Once the code is merged, the build entry (nsh config as example):
>
> ```bash
> # Optional: only clean when switching configs or after menuconfig changes
> ./build.sh vendor/spacemit/boards/k1/muse_pi_pro/configs/nsh -j8 distclean
>
> # Build
> ./build.sh vendor/spacemit/boards/k1/muse_pi_pro/configs/nsh -j8
> ```

## Flashing

The K1 BootROM requires the first-stage bootloader (FSBL) to be signed.
Flashing is a two-step process: write the bootloader and system image to
SPI NOR flash or eMMC, then power on to boot.

### Entering BootROM Fastboot Mode

1. Disconnect the USB-C cable to power off
2. **Press and hold the FDL button** (third button from the top next to the
   USB-A ports, with the Ethernet port facing up)
3. Connect the USB-C cable from the J15 OTG port to the host
4. Release the FDL button

The host should detect a fastboot device:

```bash
$ sudo fastboot devices
dfu-device       DFU download
```

> If `fastboot` cannot connect, make sure you are in the `dialout`/`plugdev`
> group or use `sudo`; alternatively use SpacemiT's `flashserver` tool as
> a drop-in replacement for mainline fastboot.

### Flashing tools

SpacemiT provides three flashing methods, pick one:

| Tool | Form | Use case | Download |
|------|------|----------|----------|
| `titanflasher` | GUI | Windows / Linux desktop GUI, beginner-friendly | [archive.spacemit.com/tools/titanflasher/](https://archive.spacemit.com/tools/titanflasher/) |
| `flashserver` | CLI | SpacemiT-modified `fastboot`, supports private protocol, CI recommended | Extract from the `titanflasher` AppImage: `./titantools_for_linux-*.AppImage --appimage-extract resources/app/flashserver` |
| `fastboot` | CLI | Mainline Android `fastboot` (35.0.1+), some scenarios need `flashserver` | Distro `android-tools-fastboot` package or [Google SDK](https://developer.android.com/tools/releases/platform-tools) |

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

To flash a full system image to eMMC (Bianbu / openEuler style):

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

### Boot order

The U-Boot environment probes boot devices in this order by default:

1. USB (USB 0:1)
2. SD card (mmc 0)
3. NVMe (scan partitions)
4. MMC (mmc 1 / mmc 2)

Within each device it tries extlinux → U-Boot script → UEFI bootefi.

## First Boot & Quick Verification

> 🚧 **Pending board-level code merge**
>
> Until the openvela board adaptation code merges, this repo cannot produce
> a bootable `nuttx.bin`. The flow below applies to **boards already flashed
> with the SpacemiT official Bianbu / openEuler system**, as a hardware
> functional verification reference.
>
> Once the code is merged, this section will be replaced with the openvela
> NSH boot log and peripheral verification commands (following the format of
> `vendor/sifli/boards/sf32lb52/lckfb_huangshan_pi/README.md`'s "First boot &
> quick verification" section).

### Console configuration

The MUSE Pi Pro onboard USB-C cable also provides a debug UART (via a
USB-UART bridge), baud rate 115200 8N1:

```bash
picocom -b 115200 /dev/ttyUSB*
# or: minicom -D /dev/ttyUSB0 -b 115200 -8 -o
```

Press the **RST** button on the board, you should see the U-Boot / OpenSBI /
kernel boot log.

## Custom Configuration

> 🚧 **Pending board-level code merge**
>
> Once the code is merged:
>
> ```bash
> cd openvela/nuttx
> make menuconfig          # interactive tweaking
> make savedefconfig
> cp defconfig \
>    ../vendor/spacemit/boards/k1/muse_pi_pro/configs/nsh/defconfig
> ```

## Debugging

- **Serial log** —— `syslog`/`printf` outputs to USB-UART, capture with
  `picocom`/`tio`
- **USB-JTAG GDB** —— K1 provides dual JTAG (one for CPU, one for MCU),
  connect with `openocd` + RISC-V GDB
- **Crash analysis** —— save the full register / stack dump, K1 SoC supports
  CPU/IO register snapshot after watchdog reboot for easier debugging

## Known Limitations

1. **Board adaptation code pending merge**: the `chips/k1/` and
   `boards/k1/muse_pi_pro/` directories currently have no openvela
   adaptation code, so an openvela image cannot be built directly. This
   README describes the target support state and hardware reference specs.
2. **FSBL must be signed**: the K1 BootROM requires the first-stage
   bootloader to be signed. Before flashing, use SpacemiT's `fsbl.sh` script
   (depends on the vendor U-Boot repo `gitee.com/bianbu-linux/uboot-2022.10`'s
   `tools/build_binary_file.py`) to sign the SPL and produce `FSBL.bin`.
3. **DDR training firmware**: SPL DDR init requires downloading `ddr_fw.bin`
   from [github.com/spacemit-com/spacemit-firmware/tree/master/k1/v0.2](https://github.com/spacemit-com/spacemit-firmware/tree/master/k1/v0.2)
   and passing it via the `DDR_FW_FILE` build variable.
4. **Multiple boot sources**: K1 supports booting from SPI-Nand /
   SPI-Nor Flash / eMMC / SD; pick the matching `partition_*.json` partition
   table based on your actual boot media when flashing.
5. **RISC-V software ecosystem still maturing**: some RISC-V SBCs are
   only recommended for headless scenarios without graphics/video
   acceleration; follow SpacemiT official docs and mainline Linux / U-Boot
   progress for graphics & video acceleration support.

## License

All files in this directory are licensed under Apache-2.0 (SPDX identifier
`Apache-2.0`); see each file's header for details. SpacemiT-provided firmware
binaries (DDR training firmware, FSBL signing tool, etc.) follow SpacemiT's
official licensing terms.
