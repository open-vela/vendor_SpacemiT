# MUSE Pi Pro（K1）开发板 — openvela 适配指引

[ [English](README.md) | 简体中文 ]

> ⚠️ **状态：尚未适配**
>
> openvela **尚未** 适配 MUSE Pi Pro（SpacemiT K1）开发板。本目录当前
> **不包含任何板级支持代码**，仅汇总下列官方硬件与文档资料，供开发者自行
> 完成适配。
>
> 欢迎提交可用的板级适配 PR。

## 开发板

- **开发板**：MUSE Pi Pro
- **芯片**：SpacemiT **K1**（M1 模组，8 核 RISC-V X60，2.0 TOPS AI 融合）

## 参考资料

| 资料 | 链接 |
|------|------|
| K1 Datasheet（PDF） | https://cdn-resource.spacemit.com/file/chip/K1/K1_datasheet_en.pdf |
| K1 Datasheet（Markdown） | https://github.com/spacemit-com/docs-chip/blob/main/en/key_stone/k1/k1_docs/k1_ds.md |
| SpacemiT 开发者文档站 | https://developer.spacemit.com/ |
| SpacemiT 固件归档（Bianbu / UEFI / titanflasher） | https://archive.spacemit.com/ |
| K1 U-Boot 烧录指南（主线 U-Boot 邮件列表 patch） | https://www.mail-archive.com/u-boot@lists.denx.de/msg577513.html |
| Muse Pi Pro openEuler 适配测试报告（含烧录流程） | https://matrix.ruyisdk.org/reports/Muse_Pi_Pro-openEuler-README/ |
| RISC-V IME AI 自定义指令规范 | https://github.com/spacemit-com/riscv-ime-extension-spec |

## 如何贡献适配

1. 先阅读上述硬件资料，了解 K1 的内存映射、时钟、引脚复用，以及
   MUSE Pi Pro 板上引出的外设。
2. 在本目录（`boards/k1/muse_pi_pro/`）下添加板级支持包：`Kconfig`、
   `include/board.h`、`src/`、`configs/<名称>/defconfig` 等，参照 openvela
   其他 vendor 板子的目录结构。
3. 建议先把串口控制台跑通，再逐步适配存储、显示及其他外设。
4. 将适配以 PR 形式提交到本分支。
