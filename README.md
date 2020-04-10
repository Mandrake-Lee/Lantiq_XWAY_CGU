# Lantiq XWAY CGU

## Introduction
Lantiq/Intel hasn't released the datasheet for his family of XWAY Soc's, hence every effort to port to modern Linux some especific features is always based in guesses and inverse engineering of the leftovers of code.

**CGU stands for Clock Generation Unit** and we believe that contains all pll's, dividers, gating, etc needed to rule the system, from CPU, DDR until USB, PCIe or simple GPIO's acting as clocks.

The purpose of this repository is to document the findings in the scattered sources available.

## Applicability
XWAY family spans for: AR9, AR10, VR9/XRX200 and GRX300.

## Physical layout
This chapter is a guess based on reverse engineering of the sources. The best hints are [AR9][2].

### Schematic
We could differentiate 2 parts of the CGU:
* Clock generation
* Clock selection

#### Clock generation
The clock signals are generated via PLL's. After that they're divided to feed the different systems.

It's worth noting that the PLL's are setup during U-Boot and they're not supposed to be changed afterwards.

![Schematic draft](https://github.com/Mandrake-Lee/Lantiq_XWAY_CGU/blob/master/CGU_schematic_generation_20200410.PNG)

#### Clock selection
The speed of each device is done via MUXing the different sources. For instance, below some examples for VR9:



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
There are 3 PPL's in the board. According [[1]](#References) for ARX100:
> /*
> * Supported clock modes
> * PLL0: rational PLL running at 500 MHz
> * PLL1: fractional PLL running at 393.219 MHz
> */

_Note: With an upported driver taken from [[4]](#References), tested in a VR9-AVR7519RW22 the freq_out values are:_

||f_out (MHz)|
|---|---|
|PLL0|500|
|PLL1|392.736328|

In the same code, one can also notice that CPU is either fed from PLL0 and PLL1, depending on the desired frequency.

We must assume that the hardware is different per each kind. Most notorious is PLL2. The register information differs from PLL0 and PLL1. Also it seems it contains a divider in addition to a PLL circuitry.

This is the register layout as far as we know. Sometimes the sources are contradictory, giving different functions for the same register bit. However, the *ifxmips_clk.c* (see [here][1], [here][2] or [here][3]) series are quite consistent with the following definition that we must take as true for AR9, AR10 and VR9. The only source that differs is *clk-xway.c*[here][6]
| BIT | PLL0        | PLL1          | PLL2      |
|-----|-------------|---------------|-----------|
| 0   | ENABLE      | ENABLE        | ENABLE    |
| 1   | LOCKED      | LOCKED        | LOCKED    |
| 2   | CFG_PLL_M   | CFG_PLL_M     | CFG_PLL_M |
| 3   | CFG_PLL_M   | CFG_PLL_M     | CFG_PLL_M |
| 4   | CFG_PLL_M   | CFG_PLL_M     | CFG_PLL_M |
| 5   | CFG_PLL_M   | CFG_PLL_M     | CFG_PLL_M |
| 6   | CFG_PLL_N   | CFG_PLL_N     | CFG_PLL_N |
| 7   | CFG_PLL_N   | CFG_PLL_N     | CFG_PLL_N |
| 8   | CFG_PLL_N   | CFG_PLL_N     | CFG_PLL_N |
| 9   | CFG_PLL_N   | CFG_PLL_N     | CFG_PLL_N |
| 10  | CFG_PLL_N   | CFG_PLL_N     | CFG_PLL_N |
| 11  | CFG_PLL_N   | CFG_PLL_N     | CFG_PLL_N |
| 12  | CFG_PLL_N   | CFG_PLL_N     | CFG_PLL_N |
| 13  | CFG_PLL_N   | CFG_PLL_D     | CFG_PLL_N |
| 14  |             | CFG_PLL_D     |           |
| 15  |             | CFG_PLL_D     |           |
| 16  |             | CFG_PLL_D     |           |
| 17  |             | CFG_PLL_K     |           |
| 18  |             | CFG_PLL_K     |           |
| 19  |             | CFG_PLL_K     |           |
| 20  | PS1_EN      | CFG_PLL_K     | PLL1_K_LO |
| 21  | PS2_EN      | CFG_PLL_K     | PLL1_K_LO |
| 22  | PLL1_FMOD_S | CFG_PLL_K     | PLL1_K_LO |
| 23  | PLL1_FMOD_S | CFG_PLL_K     | PLL1_K_LO |
| 24  |             | CFG_PLL_K     | PLL1_K_LO |
| 25  |             | CFG_PLL_K     | PLL1_K_LO |
| 26  |             | CFG_PLL_K     | PLL1_K_LO |
| 27  |             | CFG_FRAC      | PLL1_K_LO |
| 28  |             | CFG_DSMSEL    | PLL1_K_LO |
| 29  |             | CFG_CTEN      | PLL1_K_LO |
| 30  | BYPASS      | BYPASS        | BYPASS    |
| 31  |             | PHASE_DIVIDER |           |

* ENABLE. If PLL is activated.
* LOCKED. If PLL has reached stability (locked)
* CFG_PLL_M. Definition of the M value, denominator coefficient
* CFG_PLL_N. Definition of the N value, numerator integer coefficient
* CFGP_PLL_K. Definition of the numerator fraction coefficient
* CFG_PLL_D. Unknown. Maybe a divider
* CFG_FRAC. If enable, the PLL_K values are defined and used
* DDR_SEL. It's a divider from PLL0 to DDR/FPI/IO bus
* CFG_DSMSEL. Activates the Delta Sigma Modulator for fractional PLL
* CFG_CTEN. Unknown.
* BYPASS. If enable, the PLL will output the oscillator/input frequency
* PHASE_DIVIDER. If enable, pulls information from PLL0 stored in PLL1_FMOD_S as it is needed for modulo calculations
* PS1_EN & PS2_EN. Phase Shifter Enable. It is not clear but there're hints in AR9 [code][2] that this can act as a fractional divider of the PLL0 freq_out by ratios x1/2, x2/3 and x5/3.
* PLL1_FMOD_S. This is a PLL1 value stored in PLL0 (!). This value is needed when PHASE_DIVIDER is activated
* PLL1_K_LO. Unknown

## Memory layout
Based on [U-Boot IFX board patches][5]:

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
* CGU_SYS. Most read/write operations are done via this register. The structure is as follows:
 
 |BYTES|5-8|1-4|
 |---|---|---|
 |AREA|MASTER|SHADOW|
 |OPERATION|WRITE|READ|
* CGU_CLKFSR. Frequency selector
 
 |BYTES|7-8|5-6|3-4|1-2|
 |---|---|---|---|---|
 |CONTROL|ETH|ETH|PPE|PPE|
 |AREA|MASTER|SHADOW|MASTER|SHADOW|
 |OPERATION|WRITE|READ|WRITE|READ|
* CGU_CLKGSR. Gate status
* CGU_CLKGCR0. Gate control 0
* CGU_CLKGCR0. Gate control 1
* CGU_UPDATE. Whenever registers are updated, write 0x01 for update
* IF_CLK.
* CGU_DDR. DDR control. Divider?
* CGU_CT1SR. Counter1 status?
* CGU_CT_KVAL. Counter1 K-value
* CGU_PCMCR. PCM control
* PCI_CR. PCI control. Valid rates 60M, 83M, 111M, 133M, 167M, 333M (in Hz).
* GPHY1_CFG
* GPHY0_CFG
* PLL2_CFG. PLL2 configuration register

### AR10

|0xbf103000|0x00|0x04|0x08|0x0c|
|---|---|---|---|---|
|0x00|CGU_BASE|PLL0_CFG|PLL1_CFG|CGU_SYS|
|0x10|CGU_CLKFSR|CGU_CLKGSR|CGU_CLKGCR0|CGU_CLKGCR1|
|0x20|CGU_UPDATE|IF_CLK||CGU_CT1SR|
|0x30|CGU_CT_KVAL|CGU_PCMCR|||
|0x40|EPHY1_CFG|EPHY2_CFG|EPHY0_CFG||

# Glossary
* PPE. It means Packet Processing Engine. It is a separate on-chip CPU for
network offloading functions which only runs with a propietary
firmware from Lantiq. On AR9 the PPE clock is fixed to 250 MHz. Taken from [here][7]

# References
1. [VR9 ifxmips_clk.c][1]
1. [AR9 ifxmips_clk.c][2]
1. [AR10 ifxmips_clk.c][3]
1. [U-Boot sources Daniel Schwierzeck][4] Files *cgu.c, cgu_init.S*
1. U-Boot UGW6.1 sources. File *vr9.h*
1. [U-Boot IFX board patches][5]
1. [clk-xway.c][6] 

[1]: https://github.com/uwehermann/easybox-904-lte-firmware/blob/master/linux/linux-2.6.32.32/arch/mips/infineon/vr9/ifxmips_clk.c
[2]: https://github.com/uwehermann/easybox-904-lte-firmware/blob/master/linux/linux-2.6.32.32/arch/mips/infineon/ar9/ifxmips_clk.c
[3]: https://github.com/uwehermann/easybox-904-lte-firmware/blob/master/linux/linux-2.6.32.32/arch/mips/infineon/ar10/ifxmips_clk.c
[4]: https://github.com/danielschwierzeck/u-boot-lantiq/tree/openwrt/v2013.10/arch/mips/cpu/mips32/vrx200
[5]: https://github.com/uwehermann/easybox-904-lte-firmware/blob/master/package/infineon-utilities/feeds/ifx_feeds_uboot/open_uboot/patches/
[6]: https://github.com/Cl3Kener/UBER-M/blob/master/arch/mips/lantiq/xway/clk-xway.c
[7]: https://openwrt-devel.openwrt.narkive.com/sbyBZdj7/uboot-lantiq-cgu-settings-for-ramboot-image
