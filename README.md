# Lantiq_XWAY_CGU
Purpose is to document the findings on the Lantiq XWAY CGU as per the scattered sources available.

## Introduction
Lantiq/Intel hasn't released the datasheet for his family of XWAY Soc's, hence every effort to port to modern Linux some especific features is always based in guesses and inverse engineering of the leftovers of code.

**CGU stands for Clock Generation Unit** and we believe that contains all pll's, dividers, gating, etc needed to rule the system, from CPU, DDR until USB, PCIe or simple GPIO's acting as clocks.

The purpose of this repository is to document the findings in the scattered sources available.

## Applicability
XWAY family spans for: Danube, ASE, GRX300, XRX200/VR9

## Memory layout
Based on cgu.c and cgu_init.S [[1]](#References) we can see the following information:

### VR9
|0xbf103000|0x04|0x08|0x0c|0xf|
|---|---|---|---|---|
|0x10|CGU_BASE|PLL0_CFG|PLL1_CFG|CGU_SYS|



# References
[1] [U-Boot sources] (https://github.com/danielschwierzeck/u-boot-lantiq/tree/openwrt/v2013.10/arch/mips/cpu/mips32/vrx200)
