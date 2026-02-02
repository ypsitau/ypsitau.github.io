---
layout: pico-jxglib-nonavi
title: pico-jxglib
---
<div class="jumbotron">
  <div class="container-fluid">
    <h1 class="display-4">pico-jxglib
      <span class="float-right">
        <a class="btn btn-secondary" href="https://github.com/ypsitau/pico-jxglib">
          <i class="fab fa-github mr-2"></i>View on GitHub
        </a>
      </span>
    </h1>
    <p class="lead">
    </p>
  [---
  title: "About Pico SDK and pico-jxglib"
  emoji: "🕌"
  type: "tech" # tech: Technical article / idea: Idea
  published: true
  ---
  **pico-jxglib** is a library that supports Pico SDK programming for the single-board microcontroller Raspberry Pi Pico.

  This article explains an overview and how to introduce **pico-jxglib**. For details on the features provided by the library, please refer to the following articles:

  ▶️ [pico-jxglib and TFT LCD](https://zenn.dev/ypsitau/articles/2025-01-27-tft-lcd)
  ▶️ [pico-jxglib and TFT LCD (continued)](https://zenn.dev/ypsitau/articles/2025-01-31-tft-lcd-cont)
  ▶️ [pico-jxglib and LVGL](https://zenn.dev/ypsitau/articles/2025-02-04-lvgl)
  ▶️ [pico-jxglib and Terminal](https://zenn.dev/ypsitau/articles/2025-02-19-terminal)
  ▶️ [Connecting USB Keyboard/Mouse to Pico Board with pico-jxglib](https://zenn.dev/ypsitau/articles/2025-04-02-usbhost-keyboard-mouse)
  ▶️ [pico-jxglib and Command Line Editor](https://zenn.dev/ypsitau/articles/2025-04-06-cmdline-editor)
  ▶️ [Connecting USB Gamepad to Pico Board and Enjoying Games with pico-jxglib](https://zenn.dev/ypsitau/articles/2025-04-23-usbhost-gamepad)
  ▶️ [Implementing a Shell on Pico Board with pico-jxglib](https://zenn.dev/ypsitau/articles/2025-05-08-shell)
  ▶️ [Implementing a File System on Pico Board and Fully Utilizing Flash Memory with pico-jxglib](https://zenn.dev/ypsitau/articles/2025-05-31-fs-flash)
  ▶️ [Connecting SD Card or USB Storage to Pico Board with pico-jxglib](https://zenn.dev/ypsitau/articles/2025-06-06-fs-media)
  ▶️ [Operating File System with Shell in pico-jxglib (Easy Input with Auto-completion and History)](https://zenn.dev/ypsitau/articles/2025-06-09-fs-shell)
  ▶️ [Connecting RTC to Pico Board and Recording Timestamps in File System with pico-jxglib](https://zenn.dev/ypsitau/articles/2025-06-22-rtc)

  ## Motivation

  I want to make a robot. Not something too grand, but something that moves around a room with a camera and sensors, powered by motors.

  I'm a software engineer, so while I understand electronics, I'm a complete beginner when it comes to mechanics. Anyway, I needed tools, so I bought a 3D printer and a scroll saw.

  The first thing I made was a robot arm to learn about linkage mechanisms.

  ![RobotArm.jpg](https://raw.githubusercontent.com/ypsitau/zenn/main/images/2025-01-24-jxglib-intro/RobotArm.jpg)

  Each joint and rotation table has a servo motor. The control box has volume knobs, which send PWM signals to the servo motors. By the way, the robot arm, control box, and knobs were all made with a 3D printer. The only ready-made parts, apart from electronics, are Tamiya universal plates and M3x5 screws. 3D printers are amazing.

  Next, I made this: an RC car that moves by receiving commands over Wi-Fi.

  ![RobotCar.jpg](https://raw.githubusercontent.com/ypsitau/zenn/main/images/2025-01-24-jxglib-intro/RobotCar.jpg)

  It moves with stepper motors. I attached an OLED display to show Wi-Fi connection status and IP address.

  Moving things are fun! Adding mechanics to the software and electronics I've worked with so far has really broadened my world. I'm thinking of adding a camera and various sensors, connecting a color LCD to monitor captured images, sending images and sensor data to a PC, and so on. At this point, I realized something.

  Using the Pico SDK API directly, programming for PWM control, stepper motors, and OLED display isn't too complicated. But as I add more features, the software will become chaotic and hard to manage, and programming won't be fun. To improve development efficiency, I wanted middleware that provides a set of APIs for certain features.

  **pico-jxglib** was created in this process. So, the features implemented in this library are related to the robot development I'm currently working on. But since the elements that make up a robot are so broad, I might end up covering everything you can do with the Pico.

  It's a personal project, but I thought some people might be interested in the TFT LCD features, so I decided to make it public in hopes it might be useful to someone.

  ## About CMake

  To incorporate **pico-jxglib** into a Pico SDK project, edit the CMake configuration file `CMakeLists.txt`. You only need to add a few lines, so you can incorporate it without knowing the details, but understanding what CMake actually does is helpful. Here, I'll explain the basics of CMake.

  CMake generates build tool files like make or ninja based on the contents of `CMakeLists.txt`. You might think you could just write a Makefile directly, but CMake conveniently handles differences in compilers and OS, which is why many development platforms use it.

  Here's the basic structure extracted from a `CMakeLists.txt` generated by the Pico SDK:

  ```cmake:CMakeLists.txt
  add_executable(hoge hoge.c)

  target_link_libraries(hoge
    pico_stdlib)
    
  target_include_directories(hoge PRIVATE
    ${CMAKE_CURRENT_LIST_DIR})
  ```

  The processing is as follows:

  - `add_executable()` declares the creation of an executable file, with the target identifier and source files as arguments. If there are multiple source files, separate them with spaces or newlines. The target identifier is used as the base name for the executable file (e.g., `hoge.elf` or `hoge.uf2`), and as the first argument for commands starting with `target_`.
    There is also `add_library()`, which declares the creation of a library.
  - `target_link_libraries()` specifies the libraries to link to this target.
  - `target_include_directories()` specifies the include paths when compiling source files. `PRIVATE` means the include path is only valid within this `CMakeLists.txt`. There are also `PUBLIC` and `INTERFACE` specifiers; specify one of them here.

  The variable `CMAKE_CURRENT_LIST_DIR` contains the directory where this `CMakeLists.txt` exists.

  You can combine multiple libraries in one `target_link_libraries()` call:

  ```cmake
  target_link_libraries(hoge
    pico_stdlib
    hardware_i2c)
  ```

  Or split them into separate calls:

  ```cmake
  target_link_libraries(hoge
    pico_stdlib)

  target_link_libraries(hoge
    hardware_i2c)
  ```

  ---
  Up to this point, you might think of it as just a slightly different way to write a Makefile. But the behavior of `target_link_libraries()` needs some explanation.

  Looking at the `target_link_libraries()` in this `CMakeLists.txt`, you might be curious and search for files like `pico_stdlib.lib`, `pico_stdlib.a`, or `pico_stdlib.so` in the `.pico_sdk` or `build` directories. However, you won't find such files. The `pico_stdlib` specified here is not a file name, but the identifier of a target declared by the `add_library()` command in the Pico SDK's `CMakeLists.txt`.

  Here's the `CMakeLists.txt` that declares `pico_stdlib` (slightly modified for clarity):

  ```cmake:pico-sdk/src/host/pico_stdlib/CMakeLists.txt
  add_library(pico_stdlib INTERFACE)

  target_sources(pico_stdlib INTERFACE
    ${CMAKE_CURRENT_LIST_DIR}/stdlib.c
  )

  target_link_libraries(pico_stdlib INTERFACE
    pico_stdlib_headers
    pico_platform
    pico_time
    pico_divider
    pico_binary_info
    pico_printf
    pico_runtime
    pico_stdio
    hardware_gpio
    hardware_uart
  )
  ```

  The `INTERFACE` specifier is important. Targets declared with this specifier do not actually compile, but instead convey information such as source files, include paths, and libraries to link to the caller of `target_link_libraries()`.

  In the file above, `pico_stdlib` also links to libraries like `pico_stdlib_headers` and `pico_platform`, all of which are declared as `INTERFACE`. So, all the information provided by these files is also conveyed to the original caller of `target_link_libraries()`.

  Since libraries declared as `INTERFACE` do not have pre-built library files, the project that calls `target_link_libraries()` compiles the source files that make them up.

  This method has major advantages. Since the library is compiled from source files, users can use preprocessor macros to compile only the necessary code or switch processing at compile time. This improves execution performance and reduces object size.

  Also, the fact that include path information is conveyed just by calling `target_link_libraries()` is important. You are completely freed from the hassle of having to look up and specify include paths for each library you use.

  **pico-jxglib** follows the Pico SDK and declares its libraries as `INTERFACE`.

  ## How to Incorporate pico-jxglib

  You can get the latest version of **pico-jxglib** by cloning it from GitHub as follows:

  ```bash
  git clone https://github.com/ypsitau/pico-jxglib.git
  cd pico-jxglib
  git submodule update --init
  ```

  You incorporate this directory into your project with the `add_subdirectory()` command, but the way you specify it differs depending on the directory location.

  - If you place the `pico-jxglib` directory under your project directory:

    ```text
    └── your-project/
      ├── pico-jxglib/
      ├── CMakeLists.txt
      ├── your-project.cpp
      └── ...
    ```

    Add the following command to your `CMakeLists.txt`:

    ```cmake:CMakeLists.txt
    add_subdirectory(${CMAKE_CURRENT_LIST_DIR}/pico-jxglib)
    ```

  - If you place the `pico-jxglib` directory above your project's `CMakeLists.txt`:

    ```text
    ├── pico-jxglib/
    └── your-project/
      ├── CMakeLists.txt
      ├── your-project.cpp
      └── ...
    ```

    Add the following command to your `CMakeLists.txt`:

    ```cmake:CMakeLists.txt
    add_subdirectory(${CMAKE_CURRENT_LIST_DIR}/../pico-jxglib pico-jxglib)
    ```

    Note that the second argument `pico-jxglib` is required when the directory contains `..`.

  ## Actual Project Example

  Let's actually create a "blinky" program using **pico-jxglib**.

  From the VSCode command palette, run `>Raspberry Pi Pico: New Pico Project` and create a project with the following settings:

  - **Name** ... Enter the project name. For this example, enter `blink`.
  - **Board type** ... Select the board type.
  - **Location** ... Select the parent directory where the project directory will be created.
  - **Code generation options** ... **Check `Generate C++ code`**

  Assume the project directory and `pico-jxglib` are arranged as follows:

  ```text
  ├── pico-jxglib/
  └── blink/
    ├── CMakeLists.txt
    ├── blink.cpp
    └── ...
  ```

  Add the following lines to the end of `CMakeLists.txt`:

  ```cmake:CMakeLists.txt
  target_link_libraries(blink jxglib_Common)
  add_subdirectory(${CMAKE_CURRENT_LIST_DIR}/../pico-jxglib pico-jxglib)
  ```

  `jxglib_Common` is the library to add this time. The order of `target_link_libraries()` and `add_subdirectory()` does not matter. You can also add to an existing `target_link_libraries()` command.

  Edit the source file as follows:

  ```cpp:blink.cpp
  #include "pico/stdlib.h"
  #include "jxglib/Common.h"

  using namespace jxglib;

  int main()
  {
    GPIO15.init().set_dir_OUT();
    while (true) {
      GPIO15.put(true);
      ::sleep_ms(500);
      GPIO15.put(false);
      ::sleep_ms(500);
    }
  }
  ```

  `using namespace jxglib;` is a "magic spell" to use **pico-jxglib** functions and classes in this code.

  All functions, classes, and global variables of **pico-jxglib** are defined in the `jxglib` namespace to avoid name collisions with other libraries. For example, `GPIO15` in this code should be written as `jxglib::GPIO15`, but using `using namespace` allows you to omit the namespace prefix.

  For how to build and write to the board, see ["Getting Started with Pico SDK"](https://zenn.dev/ypsitau/articles/2025-01-17-picosdk).
