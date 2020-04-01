# Lantiq_XWAY_CGU
Purpose is to document the findings on the Lantiq XWAY CGU as per the scattered sources available.

## Introduction
Lantiq/Intel hasn't released the datasheet for his family of XWAY Soc's, hence every effort to port to modern Linux some especific features is always based in guesses and inverse engineering of the leftovers of code.

**CGU stands for Clock Generation Unit** and we believe that contains all pll's, dividers, gating, etc needed to rule the system, from CPU, DDR until USB, PCIe or simple GPIO's acting as clocks.

The purpose of this repository is to document the findings in the scattered sources available.

## Applicability
XWAY family spans for: Danube, ASE, GRX300, XRX200/VR9

## Physical layout
This chapter is a huge guess so please take info with care.
### VR9
#### Oscillator
Analysing [4](#References), it seems that there can be the following oscillators:
* 36 MHz (default)
* 6 Mz (changed to 36MHz via CPLD)
* 25 MHz - only for GRX255
### PLL's
There are 3 PPL's in the board. However they are not exactly identical most probably because they don't feed to the same items.
This is the register layout as far as we know.
|BIT|PLL0|PLL1|PLL2|
|---|---|---|---|
|0|RESET|||
|1|RESET|||
|2|RESET|||
|3|RESET|||
|4|RESET|||
|5|RESET|||
|6|RESET|||
|7|RESET|||
|8|RESET|||
|20|||PHASE_DIVIDER|
|27|CFG_FRAC|||
|28|CFG_DSMSEL|||
|30|BYPASS|||
|31|PHASE_DIVIDER|||

## Memory layout
Based on cgu.c and cgu_init.S [[1]](#References) we can see the following information:

### VR9
|0xbf103000|0x04|0x08|0x0c|0x0f|
|---|---|---|---|---|
|0x10|CGU_BASE|PLL0_CFG|PLL1_CFG|CGU_SYS|
|0x20|CGU_CLKFSR|CGU_CLKGSR|CGU_CLKGCR0|CGU_CLKGCR1|
|0x30|CGU_UPDATE|IF_CLK|CGU_DDR|CGU_CT1SR|
|0x40|CGU_CT_KVAL|CGU_PCMCR|PCI_CR|reserved1|
|0x50|GPHY1_CFG|GPHY0_CFG|reserved|reserved|
|0x60|reserved|reserved|reserved|reserved|
|0x70|PLL2_CFG| | | |

* CGU_BASE
* PLL0_CFG. PPL0 configuration register
* PLL1_CFG. PPL1 configuration register
* CGU_SYS. Most read/write operations are done via this register
* CGU_CLKFSR. Frequency selector
* CGU_CLKGSR. Gate status
* CGU_CLKGCR0. Gate control 0
* CGU_CLKGCR0. Gate control 1
* CGU_UPDATE. Whenever registers are updated, write 0x01 for update
* IF_CLK.
* CGU_DDR. DDR control. Divider?
* CGU_CT1SR. Counter1 status?
* CGU_CT_KVAL. Couter1 K-value
* CGU_PCMCR. PCM control
* PCI_CR. PCI control. Valid rates 60M, 83M, 111M, 133M, 167M, 333M (in Hz).
* GPHY1_CFG
* GPHY0_CFG
* PLL2_CFG. (is this address 0x60)?

# References
[1] U-Boot sources Daniel Schwierzeck(https://github.com/danielschwierzeck/u-boot-lantiq/tree/openwrt/v2013.10/arch/mips/cpu/mips32/vrx200)

[2] U-Boot UGW6.1 sources. File *vr9.h*

[3] [U-Boot sources IFXMIPS](https://github.com/zioproto/SDK.UBNT.v5.3.3/blob/master/package/uboot-ifxmips/files/cpu/mips/danube/ifx_cgu.c)

[4] [U-boot IFX patches](https://github.com/uwehermann/easybox-904-lte-firmware/blob/master/package/infineon-utilities/feeds/ifx_feeds_uboot/open_uboot/patches/504-board-vr9.patch)
