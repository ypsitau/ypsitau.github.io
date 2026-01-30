---
layout: default
title: pico-jxgLABO
---

# Grant Your Pico Board Programs Logic Analyzer and Network Capabilities! This is How to Embed pico-jxgLABO

[pico-jxgLABO](https://ypsitau.github.io/pico-jxgLABO/) is an experimental platform that allows you to try various functions of the RaspberryPi Pico microcontroller board with just a USB cable.

This article introduces how to embed pico-jxgLABO into existing programs or add new features based on this platform. Let's unleash the true power of pico-jxgLABO beyond being just an experimental platform.

**Note:** pico-jxgLABO is developed in C/C++ and is based on the [Pico SDK](https://datasheets.raspberrypi.com/pico/raspberry-pi-pico-c-sdk.pdf) provided by the Raspberry Pi Foundation. Many people may have experience developing with MicroPython or Arduino but are new to the Pico SDK. This article carefully explains everything from preparing the Pico SDK development environment to how to embed pico-jxgLABO. It's simpler than you think!

## Benefits of Embedding pico-jxgLABO in Your Own Programs

When you write the pico-jxgLABO binary (UF2 file) to the Pico board, you can use GPIO control and logic analyzer functions through command operations. This is sufficient for simple experiments, but since you can only use pre-prepared command-level functions, there are limits to what you can do.

The true power of this platform is realized by incorporating the pico-jxgLABO source code into your own programs. By embedding pico-jxgLABO, you can enjoy the following services:

- **Interactive System**: Using the pico-jxgLABO shell, you can interactively operate the Pico board by sending commands from a PC's terminal software. This is convenient for checking program operation status or changing parameters
- **Logic Analyzer**: Using the logic analyzer function, you can verify whether the program outputs the expected signals to GPIO. You can also analyze communication protocols such as I2C and SPI
- **Built-in Flash Drive**: pico-jxgLABO makes the Pico board's built-in flash memory available as a file system. This allows programs that incorporate this platform to save and load data to/from files
- **USB Drive**: The above built-in flash drive can be accessed from a PC as a USB drive. This makes it easy to exchange data between PC and Pico board
- **USB Serial Ports**: Provides 2 USB serial ports. One is for command input from terminal software, and the other can be used for communication with other applications

All you need for embedding is to [add 3 lines to the build configuration file `CMakeLists.txt` and 3 lines of code to any C/C++ program](#how-to-embed-pico-jxglabo). Let's take a look at the embedding method right away!

The development environment assumes Windows.

## Pico SDK Development Environment Tutorial

This section explains how to set up the development environment and create a Pico SDK project for those who don't have a Pico SDK environment.

### Development Environment Setup

First, please install the following tools:

- Visual Studio Code: https://code.visualstudio.com/download
- Git tools: https://git-scm.com/downloads
- Terminal software: Here we use [Tera Term](https://teratermproject.github.io/)

Also, since we'll be using the command line interface a lot to open projects and write programs, it's convenient to create shortcuts for PowerShell or Command Prompt.

The Pico SDK (tools and libraries needed for C/C++ builds) is installed as a Visual Studio Code (hereinafter VSCode) extension. After launching VSCode, press `[F1]` to open the command palette (a text box that allows you to execute VSCode functions with commands).

Enter `>Extensions: Install Extensions` (the initial `>` is already in the command palette) and open the extension installation screen. When you type something like `pico` in the search box, you'll find an extension called `Raspberry Pi Pico` in the list. Click the `[Install]` button to install it.

This completes all the settings needed for Pico SDK development.

### Creating a Pico SDK Project

To create a Pico SDK project, press `[F1]` in VSCode to bring up the command palette and execute `>Raspberry Pi Pico: New Pico Project`. A dropdown list called `New Pico Project` will appear asking for the language to use. Select `C/C++` and the following screen will be displayed.

Set the following items. Leave the others as default:

- **Basic Settings**
  - `Name`: Enter the project name. Here we'll use `P00_Simple`
  - `Board type`: Select the board type you're using (`Pico`, `Pico 2`, `Pico W`, `Pico 2 W`, or `Other`)
  - `Location`: Click the `[Change]` button and select the directory one level up from where you want to create the project directory. Here we'll use `C:\Users\YOUR-NAME\labo-project`
- **Code generation options**
  - `Generate C++ code`: Check this

Clicking the `[Create]` button at the bottom right creates the project. If this is the first time you've created a project, it will download and install the Pico SDK, which takes quite a long time (about 20 minutes). It's only the first time, so please be patient.

When project creation is complete, the project directory `C:\Users\YOUR-NAME\labo-project\P00_Simple` is created, and VSCode automatically opens with that directory as the workspace.

The following files are created in the `P00_Simple` directory:

- `CMakeLists.txt`: Project build configuration file
- `P00_Simple.cpp`: Main C++ program

The C++ program content should look like this:

```cpp
#include <stdio.h>
#include "pico/stdlib.h"


int main()
{
    stdio_init_all();

    while (true) {
        printf("Hello, world!\n");
        sleep_ms(1000);
    }
}
```

To build, press `[F7]`. If this is your first build after creating the project, a dropdown list like the following will be displayed, so select `Pico Using compilers: ...`.

When the build succeeds, the `C:\Users\YOUR-NAME\labo-project\P00_Simple\build` directory is created and a binary file named `P00_Simple.uf2` is generated.

When you connect the Pico board to the PC with a USB cable while holding down the BOOTSEL button, the Pico board is recognized as a USB drive. Copy the `P00_Simple.uf2` file to this drive, and the Pico board will automatically reboot and execute the program.

By the way, where is the `Hello, world!` from `printf()` output to? The Pico SDK allows flexible customization of the `printf()` output destination to UART, USB serial, etc., but at this point it's not configured, so it's not output anywhere yet. Since it will be output to the USB serial port provided by pico-jxgLABO, please proceed to the continuation of the article ["How to Embed pico-jxgLABO"](#how-to-embed-pico-jxglabo).

### How to Open a Pico SDK Project

To open an existing Pico SDK project, navigate to the project directory in the command prompt and execute the `code .` command. For example, to open the `C:\Users\YOUR-NAME\labo-project\P00_Simple` directory:

```text
cd C:\Users\YOUR-NAME\labo-project\P00_Simple
code .
```

At this time, if there is no `.vscode` directory that records Pico SDK configuration information, the following dialog will be displayed.

Ignore the `Select a Kit for ...` dropdown list and click the `[Yes]` button in the dialog at the bottom right that says `Do you want to import this project as Raspberry Pi Pico project?`. The following screen will be displayed.

Click the `[Import]` button to create the `.vscode` directory and make it openable as a Pico SDK project.

## How to Embed pico-jxgLABO

Let's embed pico-jxgLABO into your own program. In this article, we'll introduce the embedding method using the `P00_Simple` project created in the ["Pico SDK Development Environment Tutorial"](#pico-sdk-development-environment-tutorial) above as an example. Although `P00_Simple` is a C++ project, you can embed it in C language projects using the same procedure.

**Note:** When embedding in an existing project, if `Console over USB` is enabled in Stdio, it will conflict with pico-jxgLABO's USB functionality. Disable it by setting the argument to `0` in the `pico_enable_stdio_usb()` function call in `CMakeLists.txt`.

```cmake
pico_enable_stdio_usb(PROJECT_NAME 0)
```

pico-jxgLABO is built on top of a library group called pico-jxglib, which is managed in a [GitHub repository](https://github.com/ypsitau/pico-jxglib). You'll use the git tool to download (clone) the pico-jxglib source code, and the positional relationship with the project directory to embed should be as follows:

```text
C:\Users\YOUR-NAME\labo-project
  ├── pico-jxglib/
  └── P00_Simple/
      ├── CMakeLists.txt
      ├── P00_Simple.cpp
      └── ...
```

Execute the following commands:

```text
cd C:\Users\YOUR-NAME\labo-project
git clone https://github.com/ypsitau/pico-jxglib.git
cd pico-jxglib
git submodule update --init --recursive
```

Move to the `P00_Simple` project directory and run VSCode:

```text
cd C:\Users\YOUR-NAME\labo-project\P00_Simple
code .
```

Add the following lines to the end of `CMakeLists.txt`. For other projects, replace `P00_Simple` with the appropriate project name:

```cmake
target_link_libraries(P00_Simple jxglib_LABOPlatform_FullCmd)          # 1
add_subdirectory(${CMAKE_CURRENT_LIST_DIR}/../pico-jxglib pico-jxglib) # 2
jxglib_configure_LABOPlatform(P00_Simple)                              # 3
```

The processing content is as follows:

1. Link the pico-jxgLABO library `jxglib_LABOPlatform_FullCmd` to the `P00_Simple` target. This library includes the core functions of pico-jxgLABO and all currently provided shell commands. Since `target_link_libraries()` is already described in `CMakeLists.txt`, you can add this library there
2. Specify the pico-jxgLABO project directory. `${CMAKE_CURRENT_LIST_DIR}` refers to the directory where this `CMakeLists.txt` file is located. The second argument `pico-jxglib` is the directory name where intermediate files generated during build are placed
3. Generate configuration files for tinyusb and FatFs used in pico-jxgLABO

Change the contents of the source file `P00_Simple.cpp` as follows:

```cpp
#include <stdio.h>
#include "pico/stdlib.h"
#include "jxglib/LABOPlatform.h"        // 1

int main()
{
    stdio_init_all();
    jxglib_labo_init(false);            // 2

    while (true) {
        printf("Hello, world!\n");
        jxglib_sleep(1000);             // 3
    }
}
```

The changes made are as follows:

1. Include the `jxglib/LABOPlatform.h` header file. This allows you to use the pico-jxgLABO API
2. Call `jxglib_labo_init()` to initialize pico-jxgLABO
3. Use `jxglib_sleep()` instead of `sleep_ms()`. The pico-jxgLABO tasks are executed within this function

This completes the embedding. Press `[F7]` to build. If the build succeeds, a `P00_Simple.uf2` file is generated in the `build` directory under the project directory. Copy it to the drive recognized by connecting the USB cable while holding down the BOOTSEL button. When copying is complete, the Pico board will automatically reboot and execute the program.

Launch Tera Term for serial communication. Select `[Setup (S)]` - `[Serial port (E)...]` from the menu bar and select the serial port of the Pico board to connect.

pico-jxgLABO provides two serial ports. One is for the terminal, and the other is for applications such as logic analyzers and plotters. The first serial port provided (in the above example, `COM21`) is for the terminal. Select this and click `[New Open (N)]` or `[Reconfigure current connection (N)]`.

If `Hello, world!` is displayed every second in the Tera Term window, it's a success. Congratulations!

By the way, how should pico-jxgLABO be started? Actually, it's already running! Try pressing `[Enter]` in Tera Term. A prompt like this should appear mixed in with the `Hello, world!` display:

```text
L:/>
```

In this state, all the functions introduced in articles like [this](https://ypsitau.github.io/pico-jxgLABO/) are available. pico-jxgLABO **operates in the background**, providing various services without stopping the program's original operation.

Let's enter the `dir` command. A list of files stored in the Pico board's built-in flash memory will be displayed. The `Hello, world!` is a bit noisy though...

```text
L:/>dir
-a--- 2000-01-01 00:00:00     77 README.txt
```

pico-jxgLABO also provides USB drive functionality, and you can access the built-in flash memory from the host PC. In many cases, it will be recognized as the `D:` drive. After copying an appropriate file from the PC to this drive, try executing the `dir` command in Tera Term (here we copied `P00_Simple.cpp`):

```text
L:/>dir
-a--- 2000-01-01 00:00:00     77 README.txt
-a--- 2025-09-10 15:09:54    245 P00_Simple.cpp
```

You can also check the contents of files on the Pico board using the `cat` command:

```text
L:/>cat P00_Simple.cpp
#include <stdio.h>
#include "pico/stdlib.h"
#include "jxglib/LABOPlatform.h"

int main()
{
    stdio_init_all();
    jxglib_labo_init(false);
    while (true) {
        printf("Hello, world!\n");
        jxglib_sleep(1000);
    }
}
```

If you don't need a delay, execute `jxglib_tick()` in the main loop instead of `jxglib_sleep()`:

```cpp
#include <stdio.h>
#include "pico/stdlib.h"
#include "jxglib/LABOPlatform.h"

int main()
{
    stdio_init_all();
    jxglib_labo_init(false);

    while (true) {
        //
        // any jobs
        //
        jxglib_tick();
    }
}
```

In this article, we introduced how to embed pico-jxgLABO into a Pico SDK program generated as a template by VSCode, but the same applies when embedding into existing programs. Only two steps are needed to activate pico-jxgLABO:

- Call `jxglib_labo_init()` once
- Periodically execute either `jxglib_sleep()` or `jxglib_tick()` in the main loop

## How to Utilize Embedded pico-jxgLABO in Programs

Using the API provided by pico-jxgLABO in your programs allows you to receive various services, but we'll introduce that in another article. Even at this stage, the following utilization methods can be considered:

- **Logic Analyzer**: By activating the logic analyzer with the `la` command, you can check the timing and waveform shape of signals generated by the program to verify whether it's operating as expected. You can also observe and analyze external signals coming in through GPIO. For example, by connecting an infrared receiver module to GPIO, you can analyze infrared remote control signals with the logic analyzer, allowing you to develop while reflecting the results in the program
- **GPIO Control**: Using the `gpio` command, you can change GPIO input/output status during program operation or read sensor values connected to GPIO. Since you can check and change GPIO status without stopping program operation, it's convenient for debugging and experiments
- **PWM Control**: Using the `pwm` command, you can output PWM signals to GPIO. For example, you can use it as a handy signal generator for adjusting LED brightness or controlling motor speed
- **Serial Communication**: Using the `i2c`, `spi`, `uart` commands, you can control devices connected to these interfaces. You can test-control devices with commands and develop while reflecting the results in the program

## Summary

This article explained how to set up the Pico SDK development environment and how to embed pico-jxgLABO. By embedding pico-jxgLABO, you can use various functions such as interactive system, logic analyzer, built-in flash drive, USB drive, and USB serial ports. Let's proceed with development on the Pico board by utilizing these functions.
