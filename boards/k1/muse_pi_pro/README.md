# MUSE Pi Pro (K1) Board — openvela Porting Guide

[ English | [简体中文](README_zh-cn.md) ]

> ⚠️ **Status: NOT yet ported**
>
> openvela has **not** been ported to the MUSE Pi Pro (SpacemiT K1) board
> yet. This directory currently contains **no board support code** — it only
> collects the official hardware and documentation references below so that
> developers can carry out the port themselves.
>
> Contributions of a working board port are welcome.

## Board

- **Board**: MUSE Pi Pro
- **SoC**: SpacemiT **K1** (M1 module, 8-core RISC-V X60, 2.0 TOPS AI fusion)

## Reference Material

| Resource | Link |
|----------|------|
| K1 Datasheet (PDF) | https://cdn-resource.spacemit.com/file/chip/K1/K1_datasheet_en.pdf |
| K1 Datasheet (Markdown) | https://github.com/spacemit-com/docs-chip/blob/main/en/key_stone/k1/k1_docs/k1_ds.md |
| SpacemiT developer documentation | https://developer.spacemit.com/ |
| SpacemiT firmware archive (Bianbu / UEFI / titanflasher) | https://archive.spacemit.com/ |
| K1 U-Boot flashing guide (mainline U-Boot mailing-list patch) | https://www.mail-archive.com/u-boot@lists.denx.de/msg577513.html |
| Muse Pi Pro openEuler porting test report (with flashing flow) | https://matrix.ruyisdk.org/reports/Muse_Pi_Pro-openEuler-README/ |
| RISC-V IME AI customized instruction spec | https://github.com/spacemit-com/riscv-ime-extension-spec |

## How to Contribute a Port

1. Study the hardware docs above to understand the K1 memory map, clocks,
   pinmux and the on-board peripherals exposed by the MUSE Pi Pro.
2. Add the board support package under this directory
   (`boards/k1/muse_pi_pro/`): `Kconfig`, `include/board.h`, `src/`,
   `configs/<name>/defconfig`, etc., following the layout of other vendor
   boards in openvela.
3. Bring up the serial console first, then storage, display and other
   peripherals incrementally.
4. Submit the port as a pull request to this branch.
