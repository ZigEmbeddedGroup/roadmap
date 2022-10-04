# Provided Tooling

MicroZig/ZigEmbeddedGroup should provide all the tools required to work with supported systems.

## Flashing Tools

### Generic Purpose

- [`dfu-util`](https://dfu-util.sourceforge.net/) for flashing devices with a DFU bootloader
- [`mtools`](https://www.gnu.org/software/mtools/) for simple cross-platform flashing of UF2 devices, and general modification of FAT32 file systems

### Device Specific

- [`avrdude`](https://www.nongnu.org/avrdude/) for flashing AVR devices
- [`lpc21isp`](https://sourceforge.net/projects/lpc21isp/) for NXP LPC microcontrollers
- [`stm32flash`](https://github.com/stm32duino/stm32flash) for ST microcontrollers

## Debug Tools

- Serial terminal emulator (putty?)
- gdb/device debugger
