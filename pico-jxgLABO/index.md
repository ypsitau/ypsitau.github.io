---
layout: pico-jxgLABO-nonavi
title: pico-jxgLABO
---

# Pico Board is a Laboratory! Introducing pico-jxgLABO and Trying Out Logic Analyzer Functions

The Raspberry Pi Pico is a small and affordable (~$6) microcontroller board provided by Raspberry Pi Ltd. The original Pico features a 125MHz 32-bit ARM dual-core processor, 264KB SRAM, and 2MB flash memory. The Pico 2 goes even further with a 150MHz dual-core CPU, 520KB SRAM, 4MB flash memory, and hardware floating-point support, making it an extremely high-performance microcontroller board.

![pico-and-pico2.jpg](https://raw.githubusercontent.com/ypsitau/zenn/main/images/2025-08-01-labo-intro/pico-and-pico2.jpg)
*Original Pico and Pico 2*

However, even with such abundant features, they're wasted without knowing how to use them. Therefore, I developed **pico-jxgLABO**, an experimental platform that allows you to actually try various functions on the Pico board. The main features are as follows:

- **Easy Environment Setup**: Just connect the Pico board with a USB cable and you can start experimenting immediately
- **Logic Analyzer**: Equipped with a logic analyzer function that can sample up to 50MHz (for Pico 2). You can capture internal signals without wiring. Of course, you can also capture external signals by connecting to GPIO pins
- **Serial Communication**: Send and receive data using various serial communication protocols such as UART, SPI, and I2C
- **GPIO Control**: Control GPIO pins for input, output, PWM control, etc.
- **File System Shared with Host PC**: Mount the Pico board's flash memory as a file system and share it with the host PC
- **PC-like File Operations**: Perform PC-like file operations such as copying, deleting files, and creating directories

**pico-jxgLABO** is not just an experimental platform. You can incorporate the rich features of pico-jxgLABO into any program by [adding just 3 lines of code](articles/2025-09-16-labo-embed-en). For example, you can use the logic analyzer function to debug programs running on the Pico board, monitor serial communication, and more. This is actually my main use case.

## How to Install pico-jxgLABO

### Required Equipment

The equipment needed to use pico-jxgLABO is as follows:

- Host PC (Windows, Linux, etc.)
- Pico board (Pico, Pico 2, Pico W, Pico 2 W, etc.)
- USB cable

Install terminal software for serial communication on the host PC. The following instructions use a Windows PC as the host and [Tera Term](https://teratermproject.github.io/index-en.html) as the terminal software.

### Writing to the Pico Board

1. Download one of the following UF2 files:

   |Pico Board Type|UF2 File|
   |---------------|--------|
   |Pico     |[pico-jxgLABO.uf2](https://github.com/ypsitau/pico-jxgLABO/releases/latest/download/pico-jxgLABO.uf2)   |
   |Pico W   |[pico-w-jxgLABO.uf2](https://github.com/ypsitau/pico-jxgLABO/releases/latest/download/pico-w-jxgLABO.uf2) |
   |Pico 2   |[pico2-jxgLABO.uf2](https://github.com/ypsitau/pico-jxgLABO/releases/latest/download/pico2-jxgLABO.uf2)  |
   |Pico 2 W |[pico2-w-jxgLABO.uf2](https://github.com/ypsitau/pico-jxgLABO/releases/latest/download/pico2-w-jxgLABO.uf2)|

2. Connect the USB cable while holding down the BOOTSEL button on the Pico board, and it will be recognized as USB mass storage. In many cases, it will be recognized as the `D:` drive
3. Copy the downloaded UF2 file to the above drive. When copying is complete, the Pico board will automatically reboot and pico-jxgLABO will be available

## How to Use pico-jxgLABO

### Environment Setup and Operation Check

Launch Tera Term for serial communication. Select `[Setup (S)]` - `[Serial port (E)...]` from the menu bar and select the serial port of the Pico board to connect.

![teraterm-setting-en.png](https://raw.githubusercontent.com/ypsitau/zenn/main/images/2025-08-01-labo-intro/teraterm-setting-en.png)

pico-jxgLABO provides two serial ports. One is for the terminal, and the other is for applications such as logic analyzers and plotters. The first serial port provided (in the above example, `COM21`) is for the terminal. Select this and click `[New open]` or `[New setting]`.

When you press `[Enter]` in the terminal, a prompt like this will appear:

```text
L:/>
```

Use this prompt to operate pico-jxgLABO. The prompt input can be edited using cursor keys, delete keys, etc. You can complete command names and file names with the `TAB` key. You can also refer to command history with the up and down arrow keys.

First, let's enter the `help` command to display a list of available commands.

```text
L:/>help
.               executes the given script file
about-me        prints information about this own program
about-platform  prints information about the platform
adc             controls ADC (Analog-to-Digital Converter)
adc0            controls ADC (Analog-to-Digital Converter)
adc1            controls ADC (Analog-to-Digital Converter)
                        :
                        :
```

When you execute a command with the `--help` (or `-h`) option, detailed usage will be displayed.

```text
L:/>cp --help
Usage: cp [OPTION]... SOURCE... DEST
Options:
 -h --help      prints this help
 -r --recursive copies directories recursively
 -v --verbose   prints what is being done
 -f --force     overwrites existing files without prompting
```

Familiar file operation commands (`ls`, `cp`, `mv`, `rm`, `mkdir`, `cat`, etc.) are available. It also supports redirection, so you can save command output to files.

The `L:` drive is the Pico board's flash memory mounted as a FAT file system and is recognized as USB mass storage from the host PC. The drive letter assigned on the host PC side varies depending on the environment, but it is often recognized as the `D:` drive. Files created on the Pico board can be accessed from the host PC, and you can also copy files from the host PC to the Pico board.

### Trying the Logic Analyzer Function

Let's try the logic analyzer function of pico-jxgLABO. As a demonstration, we'll capture I2C interface signals.

Use the `la` command to start the logic analyzer. Use the `-p` option to specify the pins to capture. Here, we'll measure GPIO2 (I2C SDA) and GPIO3 (I2C SCL) on the Pico board.

```text
L:/>la -p 2,3 enable
enabled pio:2 12.5MHz (samplers:1) pins:2,3 events:1/88674 (heap-ratio:0.7)
```

The logic analyzer is now enabled. Signal capture is performed in the background. Since capture processing only occurs when there are signal changes, there's no need to rush the operation.

Next, use the `i2c1` command to output I2C protocol signals. Use the `-p` option to specify the SDA and SCL pins to use. The `scan` subcommand sends Read requests to addresses 0 through 127 and displays addresses that responded.

```text
L:/>i2c1 -p 2,3 scan
Bus Scan on I2C1
   0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F
00 -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
10 -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20 -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30 -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
40 -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50 -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
60 -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
70 -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
```

This generated I2C signals. Run the `la` command to check the logic analyzer status to see if capture was performed.

```text
L:/>la
enabled pio:2 12.5MHz (samplers:1) pins:2,3 events:3461/88674 (heap-ratio:0.7)
```

You can see that the number of events has increased and signal capture has been performed. Use the `la print` command to display the captured signals.

```text
L:/>la print
 Time [usec] P2  P3  
             │   │  
             :   :  
        0.00 └─┐ │  
        1.20   │ └─┐
      575.04 ┌─┘   │
      579.52 │   ┌─┘
      585.84 │   └─┐
      590.24 │   ┌─┘
      596.64 │   └─┐
      601.04 │   ┌─┘
      607.36 │   └─┐
      611.76 │   ┌─┘
      618.16 │   └─┐
      622.56 │   ┌─┘
      628.88 │   └─┐
      633.28 │   ┌─┘
      639.68 │   └─┐
      644.08 │   ┌─┘
      650.40 │   └─┐
      654.88 │   ┌─┘
      655.52 └─┐ │  
      661.20   │ └─┐
      665.60   │ ┌─┘
      671.92   │ └─┐
      676.32   │ ┌─┘
      676.64 ┌─┘ │  
      682.72 │   └─┐
      687.84 └─┐   │
         :
         :
```

The display resolution (time interval displayed per line) is 1000usec (1msec) by default. In the above example, the edge interval is about 5usec, so it must be shorter than that for proper display. Therefore, use the `--reso` option to set the display resolution to 4usec and display the waveform.

```text
L:/>la print --reso:4
 Time [usec] P2  P3  
             │   │  
             :   :  
        0.00 └─┐ │  
        1.20   │ └─┐
               :   :
      575.04 ┌─┘   │
             │     │
      579.52 │   ┌─┘
             │   │  
      585.84 │   └─┐
             │     │
      590.24 │   ┌─┘
             │   │  
      596.64 │   └─┐
             │     │
      601.04 │   ┌─┘
             │   │  
      607.36 │   └─┐
             │     │
      611.76 │   ┌─┘
             │   │  
      618.16 │   └─┐
             │     │
      622.56 │   ┌─┘
             │   │  
      628.88 │   └─┐
             │     │
      633.28 │   ┌─┘
             │   │  
      639.68 │   └─┐
             │     │
      644.08 │   ┌─┘
             │   │  
      650.40 │   └─┐
             │     │
      654.88 │   ┌─┘
      655.52 └─┐ │  
               │ │  
      661.20   │ └─┐
               │   │
      665.60   │ ┌─┘
               │ │  
      671.92   │ └─┐
               │   │
      676.32   │ ┌─┘
      676.64 ┌─┘ │  
             │   │  
      682.72 │   └─┐
             │     │
      687.84 └─┐   │
               │   │
         :
         :
```

The image below shows a screen capture rotated horizontally.

![la-i2c.png](https://raw.githubusercontent.com/ypsitau/zenn/main/images/2025-08-01-labo-intro/la-i2c.png)

The `la` command has a protocol decoder function (details [here](https://zenn.dev/ypsit/articles/2025-09-08-labo-la)) that can automatically analyze and display bit patterns. But if you dare to look at the signals with an I2C explanation article in hand... can't you see the bit patterns of start condition, address, Read/Write, ACK, and stop condition? It's somehow pleasing when the signal protocol, which was a black box, becomes completely clear by observing the waveform.

By default, `la print` displays the first 80 events. You can display all events using the `--part:all` option.

```text
L:/>la print --part:all
 Time [usec] P2  P3  
             │   │  
             :   :  
        0.00 └─┐ │  
        1.20   │ └─┐
             :
             :
```

You can interrupt the display by pressing `Ctrl-C`.

You can save the display content to a file using redirection. For example, to save to a file called `i2c.log`:

```text
L:/>la print --part:all > i2c.log
```

Since the `L:` drive is recognized as a USB mass storage device by the host PC, you can open the `i2c.log` file on the host PC side. Opening it in a PC text editor makes the content easier to read.

To redo the capture, execute the `la enable` command again. If you omit the options, the previous settings will be inherited.

```text
L:/>la enable
enabled pio:2 12.5MHz (samplers:1) pins:2,3 events:1/88674 (heap-ratio:0.7)
```

Execute the `la disable` command to disable the logic analyzer.

That covers the basic operations of the logic analyzer. Here are some examples of communication with devices using the `i2c` command, so please try various things.

- Read the time from the RTC module DS3231 (send data 0x00 to I2C address 0x68, then receive 7 bytes of data)

  ```text
  L:/>i2c1 -p 2,3 0x68 write:0x00 read:7
  ```

- Write data 0xaa, 0xbb, 0xcc, 0xdd to address 0x0186 of EEPROM 24LC64 (send data 0x01, 0x86, 0xaa, 0xbb, 0xcc, 0xdd to I2C address 0x50)

  ```text
  L:/>i2c1 -p 2,3 0x50 write:0x01,0x86,0xaa,0xbb,0xcc,0xdd
  ```

- Read 4 bytes of data from address 0x0186 of EEPROM 24LC64 (send data 0x01, 0x86 to I2C address 0x50, then receive 4 bytes of data)

  ```text
  L:/>i2c1 -p 2,3 0x50 write:0x01,0x86 read:4
  ```

### Other Features

pico-jxgLABO has many features besides logic analyzer and I2C communication. Here are some examples:

- **GPIO Control**: Use the `gpio` command to control GPIO pins. For example, to set GPIO6 to output mode and make it HIGH:

  ```text
  L:/>gpio6 func:sio dir:out put:1
  ``` 

- **PWM Control**: Use the `pwm` command to output PWM signals. For example, to output a 1kHz PWM signal with a 50% duty cycle on GPIO6:

  ```text
  L:/>pwm6 func:pwm freq:1000 duty:.5 enable
  ```

- **ADC Control**: Use the `adc` command to read analog signals. For example, to read the analog value of ADC0 (GPIO26):

  ```text
  L:/>adc0 read
  3.300V
  ```

- **UART Communication**: Use the `uart` command for UART communication. For example, to send and receive data using GPIO0 (TX) and GPIO1 (RX) (data reception times out because no communication partner device is connected):

  ```text
  L:/>uart0 -p 0,1 write:0x12,0x34,0x56,0x78 read:6
  No data received within timeout
  ```

- **SPI Communication**: Use the `spi` command for SPI communication. For example, to send and receive data using GPIO2 (SCK), GPIO3 (MOSI), GPIO4 (MISO):

  ```text
  L:/>spi0 -p 2,3,4 write:0x12,0x34,0x56,0x78 read:6
  00 00 00 00 00 00
  ```

  Since no communication partner device is connected, all received data is 0.

- **File Operations**: Use commands like `cp`, `mv`, `rm`, `mkdir` to manipulate files and directories. For example, to copy `test.txt` file to `backup.txt`:

  ```text
  L:/>cp test.txt backup.txt
  ```

- **Shell Script Execution**: You can create shell script files and execute multiple commands together. For example, write the following content in a `pwm-2-3-4-5.sh` file:

  ```text
  pwm --quiet 2,3,4,5 func:pwm freq:1000 counter:0
  pwm2 --quiet duty:.2
  pwm3 --quiet duty:.4
  pwm4 --quiet duty:.6
  pwm5 --quiet duty:.8
  pwm 2,3,4,5 enable
  ```
  
  Execute it as follows:

   ```text
   L:/>. pwm-2-3-4-5.sh
   ```

## Summary

This article introduced the installation method of pico-jxgLABO and focused on the logic analyzer function. Did you experience that detailed signal analysis is possible with an $6 Pico board?

In addition to the logic analyzer function introduced this time, pico-jxgLABO provides a wealth of features necessary for embedded development, such as GPIO control, PWM output, serial communication, and file operations. It can also be incorporated into existing projects with just 3 lines of code.

---

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details.

## GitHub Repository

[https://github.com/ypsitau/pico-jxgLABO](https://github.com/ypsitau/pico-jxgLABO)
