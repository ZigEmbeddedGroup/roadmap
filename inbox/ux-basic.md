# Basic UX Example

> This file outlines how a microzig project should be set up.

```sh-session
[user@host project]$ zig init-exe --gitfiles
info: Created .gitignore
info: Created .gitattributes
info: Created build.zig
info: Created src/main.zig
info: Next, try `zig build --help` or `zig build test`
[user@host project]$ zig pkg add https://microzig.tech/pkgs/microzig
[user@host project]$ zig pkg add https://microzig.tech/pkgs/microzig.avr
[user@host project]$ $EDITOR build.zig    # see below
[user@host project]$ $EDITOR src/main.zig # see below
[user@host project]$ zig build
Memory Usage (atmega328p):

  Program:    2963 bytes (9.0% Full)
  (.text + .data + .bootloader)

  Data:        125 bytes (6.1% Full)
  (.data + .bss + .noinit)

[user@host project]$ zig build flash
Please select target device:
-> * /dev/ttyUSB0 (Future Technology Devices International, Ltd FT232 USB-Serial (UART) IC)
   * /dev/ttyUSB1 (Silicon Labs CP210x UART Bridge)
   * /dev/ttyUSB2 (QinHeng Electronics HL-340 USB-Serial adapter)
   * /dev/ttyACM0 (Cygnal Integrated Products, Inc. CP2102/CP2109 UART Bridge Controller [CP210x family])
[user@host project]$ zig build terminal
? h
commands:
  h: print this help
  x: says hello
? x
hello, world!
? z
unknown command: z
?
```

##### `build.zig`

```zig
const std = @import("std");
const microzig = @import("microzig");

pub fn build(b: *std.build.Builder) !void {
    const mode = b.standardReleaseOptions();

    const flash_step = b.step("flash", "Programs the microcontroller.");
    const terminal_step = b.step("terminal", "Opens a terminal to the microcontroller");

    const select_serial = microzig.addSelectSerial(b, null); // add vid/pid-filter here

    const exe = try microzig.addEmbeddedExecutable(
        b,
        "blinky",
        "src/main.zig",
        microzig.boards.arduino_nano,
        .{},
    );
    exe.show_memory_usage = true; // shows how much memory we use after successful build
    exe.setBuildMode(mode);
    exe.install();

    const program = microzig.addProgramStep(
        exe,
        .{
            // use the arduino bootloader
            .arduino = .{
                .port = select_serial.getSelectedPort(),
            },
        },
    );
    flash_step.dependOn(&program.step);

    const run_term = microzig.addTerminalStep(
        select_serial.getSelectedPort(),
        .{
            .baud = 115_200,
            .rts = .low,
            .dts = .low,
        },
    );
    terminal_step.dependOn(&run_term.step);
}
```

##### `src/main.zig`

```zig
const std = @import("std");
const micro = @import("microzig");

pub fn main() !void {
    var uart = try micro.Uart(0).init(.{
        .baud_rate = 115_200,
    });

    const writer = uart.writer();
    const reader = uart.reader();

    while (true) {
        try writer.writeAll("? ");
        const cmd = try reader.readByte();
        try writer.print("{c}\r\n", .{cmd});

        switch (cmd) {
            'h' => try writer.writeAll("commands:\r\n  h: print this help\r\n  x: says hello\r\n"),
            'x' => try writer.writeAll("hello, world!\r\n"),
            else => try writer.print("unknown command: {c}\r\n", .{cmd}),
        }
    }
}
```
