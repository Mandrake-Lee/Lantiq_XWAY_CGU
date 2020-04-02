# Lantiq XWAY CGU

## Introduction
Lantiq/Intel hasn't released the datasheet for his family of XWAY Soc's, hence every effort to port to modern Linux some especific features is always based in guesses and inverse engineering of the leftovers of code.

**CGU stands for Clock Generation Unit** and we believe that contains all pll's, dividers, gating, etc needed to rule the system, from CPU, DDR until USB, PCIe or simple GPIO's acting as clocks.

The purpose of this repository is to document the findings in the scattered sources available.

## Applicability
XWAY family spans for: Danube, Amazon, AR9, VR9/XRX200 GRX300

## Physical layout
This chapter is a huge guess so please take info with care.

### Schematic
![Schematic draft](https://github.com/Mandrake-Lee/Lantiq_XWAY_CGU/blob/master/CGU_shematic_draft_20200420.PNG)

### Oscillator
Analysing [[3]](#References):

#### Danube, Amazon
* 36 MHz
* 35.328 MHz

#### AR9
* 36 MHz
* 35.328 MHz
* 25 MHz

#### VR9, AR10 (possibly)
* 36 MHz (default)
* 6 Mz (changed to 36MHz via CPLD)
* 25 MHz - only for GRX255
### PLL's
There are 3 PPL's in the board. However they are not exactly identical most probably because they don't feed to the same items.
* PLL0 seems to be the master
* PLL1 seems to be the brother
* PLL2 seems to be the auxilliary or little brother. The register information differs from PLL0 and PLL1. Also it seems it contains a divider in addition to a PLL circuitry.

This is the register layout as far as we know. See [[1]](#References) & [[5]](#References):
|BIT|PLL0|PLL1|PLL2|
|---|---|---|---|
|0|ENABLE|ENABLE|DDR_SEL?|
|1|LOCKED|LOCKED|DDR_SEL?|
|2|CGF_PLLM|CGF_PLLM||
|3|CGF_PLLM|CGF_PLLM||
|4|CGF_PLLM|CGF_PLLM||
|5|CGF_PLLM|CGF_PLLM||
|6|CFG_PLLN|CFG_PLLN||
|7|CFG_PLLN|CFG_PLLN||
|8|CFG_PLLN|CFG_PLLN||
|9|CFG_PLLN|CFG_PLLN||
|10|CFG_PLLN|CFG_PLLN||
|11|CFG_PLLN|CFG_PLLN||
|12||||
|13|||CFG_INPUT_DIV|
|14|||CFG_INPUT_DIV|
|15|||CFG_INPUT_DIV|
|16|||CFG_INPUT_DIV|
|17|CFG_PLLK|CFG_PLLK?|SRC|
|18|CFG_PLLK|CFG_PLLK?|SRC|
|19|CFG_PLLK|CFG_PLLK?||
|20|CFG_PLLK|CFG_PLLK?|PHASE_DIVIDER|
|21|CFG_PLLK|CFG_PLLK?||
|22|CFG_PLLK|CFG_PLLK?||
|23|CFG_PLLK|CFG_PLLK?||
|24|CFG_PLLK|CFG_PLLK?||
|25|CFG_PLLK|CFG_PLLK?||
|26|CFG_PLLK|CFG_PLLK?||
|27|CFG_FRAC|||
|28|CFG_DSMSEL|||
|29|SRC|||
|30|BYPASS|||
|31|PHASE_DIVIDER|SRC||

* BYPASS. If enable, the PLL will output the oscillator/input frequency
* DDR_SEL. It's a divider from PLL0 to DDR/FPI/IO bus
* SRC:
  * For PPL0 & PLL1, if enable, freq_in will be the one of the USB bus i.e 12MHz
  * For PPL2, this property is always active with following choices:
    * 0 : PPL2_freq_in = PPL0_freq_out affected by PLL2 divider
    * 1 : PPL2_freq_in depends on the PHASE_DIVIDER; either 36MHz or 35.328MHz
    * 2 : PPL2_freq_in is the same as USB bus, i.e. 12MHz

## Memory layout
Based on [[3]](#References):

### DANUBE, AMAZON, AR9

|0xbf103000|0x00|0x04|0x08|0x0c|
|---|---|---|---|---|
|0x00|CGU_BASE|PLL0_CFG|PLL1_CFG|PLL2_CFG|
|0x10|CGU_SYS|CGU_UPDATE|IF_CLK|CGU_OSC_CTRL |
|0x20|CGU_SMD||CGU_CT1SR|CGU_CT2SR|
|0x30|CGU_PCMCR|PCI_CR|CGU_MIPS_PWR_DWN |CLK_MEASURE|

### VR9

|0xbf103000|0x00|0x04|0x08|0x0c|
|---|---|---|---|---|
|0x00|CGU_BASE|PLL0_CFG|PLL1_CFG|CGU_SYS|
|0x10|CGU_CLKFSR|CGU_CLKGSR|CGU_CLKGCR0|CGU_CLKGCR1|
|0x20|CGU_UPDATE|IF_CLK|CGU_DDR|CGU_CT1SR|
|0x30|CGU_CT_KVAL|CGU_PCMCR|PCI_CR|reserved1|
|0x40|GPHY1_CFG|GPHY0_CFG|reserved|reserved|
|0x50|reserved|reserved|reserved|reserved|
|0x60|PLL2_CFG| | | |

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
* PLL2_CFG

### AR10

|0xbf103000|0x00|0x04|0x08|0x0c|
|---|---|---|---|---|
|0x00|CGU_BASE|PLL0_CFG|PLL1_CFG|CGU_SYS|
|0x10|CGU_CLKFSR|CGU_CLKGSR|CGU_CLKGCR0|CGU_CLKGCR1|
|0x20|CGU_UPDATE|IF_CLK||CGU_CT1SR|
|0x30|CGU_CT_KVAL|CGU_PCMCR|||
|0x40|EPHY1_CFG|EPHY2_CFG|EPHY0_CFG||

# References
[1] [U-Boot sources Daniel Schwierzeck](https://github.com/danielschwierzeck/u-boot-lantiq/tree/openwrt/v2013.10/arch/mips/cpu/mips32/vrx200) Files *cgu.c, cgu_init.S*

[2] U-Boot UGW6.1 sources. File *vr9.h*

[3] [U-Boot IFX board patches](https://github.com/uwehermann/easybox-904-lte-firmware/blob/master/package/infineon-utilities/feeds/ifx_feeds_uboot/open_uboot/patches/)

[4] [clk-xway.c](https://github.com/Cl3Kener/UBER-M/blob/master/arch/mips/lantiq/xway/clk-xway.c) 
