We need to finalize the public-facing API of the microzig abstractions.

----

Here are _examples_ of knowledge/information that [Marnix](https://github.com/marnix) thinks
these APIs (and the modules implementing them) should be able to express.

This might help drive decisions about where to place specific code,
and what _principles_ we want to follow.

- Chip I3G4250D is a gyroscope device, which
   * supports I2C with 7-bit device address 0b110100x, x = value of a pin.
   * supports SPI (clock=1 when idle, values starting 2nd clock transition),
     default in 4-wire mode,
     switched to 3-wire mode (SDI=SDO) by setting bit SIM (0x01) to 1 in SPI register CTRL_REG4 (0x23).
   * has a specific list of 1-byte 'registers' (read-only / read-write / write-only)
     that can be accessed by 'write register number, then read or write register value'.
      - e.g. register WHO_AM_I = 0x0F, which is read-only, and always has value 0b11010011.
- Chip LSM303AGR contains two devices, an accelerometer and a magnetometer.
   - The accelerometer
      * supports I2C with 7-bit device address 0b0011001.
      * supports SPI (...).
      * has a specific list of registers (...).
   - The magnetometer
      * supports I2C 7-bit device address 0b0011110.
      * supports SPI (...).
      * has a specific list of registers (...).
- MCUs STM32F303xB/C
   - have their I2C1 bus on pins PB6 + PB7 = I2C1_SCL + I2C1_SDA.
   - have their SPI1 bus on pins PA5 + PA7 + PA6 = SPC + SDI + SDO.
- Board STM32F3DISCOVERY, revision E02 (MB1035-F303C-E02)
   * has MCU STM32F303VCT6.
   * has 8 LEDs connected to 8 specific MCU pins.
   * has LSM303AGR,
     with both its devices connected to MCU bus I2C1
     \[and not to SPI\].
   * has I3G4250D connected to MCU bus SPI1,
     with MCU pin PE3 (active low) as chip select line.
   * ...and connected to shared MCU pins PA5 + PA7 = SCL + SDA
     (selected by keeping PE3 high; can be driven by software I2C on these pins).
   * has board pins that are map 1-to-1 to the MCU pins.
