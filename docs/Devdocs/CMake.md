## What Actually *Is* CMake?
CMake is a **compilation tool** designed to simplify C++ compilation by turning it into a series of terminal commands that use config files to provide all the compilation flags that would otherwise have to be set by hand each time you build. It has its own syntax that needs to be learned, but once the build is reasonably configured, it tends to be pretty set-and-forget. The process for using it takes a bit of getting used to, but it's relatively easy to learn.

## How CMake Works
CMake is **very directly** tied to the directory structure of the project that you're trying to build. Both the parent directory and each subdirectory with `.cpp` files that you want to compile will have a `CMakeLists.txt` file in it - this is that config file mentioned earlier. The parent directory's `CMakeLists.txt` will have all of the compiler definitions (such as the versions of C and C++ you're using, the locations of your library files, the compiler flags and versions, and other overhead things), whereas each subdirectory's file will just have the specific definitions for the `.cpp` files in that directory. In practice, **this means you'll just edit the `CMakeLists.txt` file in the subdirectory containing the files you're working on** if you add any or modify the way they need to be included. 

When CMake is running, it will dump all the intermediary steps of the process to a configured directory. **In almost all cases, this will be a subdirectory in the project called `build/`**; we'll go over how to configure this directory and use it for the actual build process below. 

## Example CMakeLists.txt File
Below is the file found at `src/CMakeLists.txt` from the [Active Drag System](https://github.com/RocketryVT/active-drag-system/blob/main/src/CMakeLists.txt); this notably is *not* the parent directory - it is specifically just the compile targets for the `src/active_drag_system.cpp` file, and all of the necessary header paths that it needs to include.
```cmake
add_executable(active_drag_system
	${PROJECT_SOURCE_DIR}/src/active_drag_system.cpp
	${PROJECT_SOURCE_DIR}/src/ms5607.cpp
	${PROJECT_SOURCE_DIR}/src/adxl375.cpp
	${PROJECT_SOURCE_DIR}/src/iim42653.cpp
	${PROJECT_SOURCE_DIR}/src/mmc5983ma.cpp
	${PROJECT_SOURCE_DIR}/src/pwm.cpp
	${PROJECT_SOURCE_DIR}/src/log_format.cpp
	${PROJECT_SOURCE_DIR}/src/heartbeat.cpp
	${PROJECT_SOURCE_DIR}/src/serial.cpp
)

pico_set_binary_type(active_drag_system copy_to_ram)

target_link_libraries(active_drag_system pico_stdlib pico_logger pico_flash pico_rand pico_multicore pico_sync hardware_i2c hardware_adc hardware_timer hardware_pwm FreeRTOS-Kernel FreeRTOS-Kernel-Heap4 libfixmath libfixmatrix libfixkalman)
target_include_directories(active_drag_system PUBLIC ${PROJECT_SOURCE_DIR}/include)
target_compile_definitions(active_drag_system PRIVATE
	USE_FREERTOS=1
	#        DEBUG=1
	PICO_STDIO_STACK_BUFFER_SIZE=64 # use a small printf on stack buffer
)

pico_enable_stdio_usb(active_drag_system 1)
pico_enable_stdio_uart(active_drag_system 0)

pico_add_extra_outputs(active_drag_system)
```

Let's break down what each of these lines means step-by-step.

1. `add_executable()` is how we **define the output of the build process.** When you complete a compilation, it will output a `.uf2` file that is then copied to the RP2040 (either manually or via `picotool`); this is where the name of that executable is defined, and which files are included in it. Note that this means the "main" file (the one that will be executed on the RP2040) for `active_drag_system` is `src/active_drag_system`, and all of the other `.cpp` files are included into it. If you add any additional files to the project that need to be included, **you need to add their `.cpp` files to this list for the linker to be able to find them.** You can, for reference, add multiple compilation targets to the same `CMakeLists.txt` file to compile them all at once, and then select which one you want to actually upload after the fact; see the file in our [tools](https://github.com/RocketryVT/active-drag-system/blob/main/tools/CMakeLists.txt) directory for an example of how to do this.
```cmake
add_executable(active_drag_system
	${PROJECT_SOURCE_DIR}/src/active_drag_system.cpp
	${PROJECT_SOURCE_DIR}/src/ms5607.cpp
	${PROJECT_SOURCE_DIR}/src/adxl375.cpp
	${PROJECT_SOURCE_DIR}/src/iim42653.cpp
	${PROJECT_SOURCE_DIR}/src/mmc5983ma.cpp
	${PROJECT_SOURCE_DIR}/src/pwm.cpp
	${PROJECT_SOURCE_DIR}/src/log_format.cpp
	${PROJECT_SOURCE_DIR}/src/heartbeat.cpp
	${PROJECT_SOURCE_DIR}/src/serial.cpp
)
```

2. `pico_set_binary_type()` is just how we define *where* the flash process will actually put the compiled code when you upload it. The above option shouldn't need to be changed; the alternative is storing the code directly in SRAM as opposed to in-memory, which we don't want.
```cmake
pico_set_binary_type(active_drag_system copy_to_ram)
```

3. `target_link_libraries()` is **where we define which library headers to include in the project.** When you add a `#include <hardware/gpio.h>` statement to a file, the underlying library ("hardware_gpio" in this case) needs to be added to this block for the compiler to be able to find that library; all libraries are added in a single call, separated by newlines or spaces in the call. 
```cmake 
target_link_libraries(active_drag_system pico_stdlib pico_logger pico_flash pico_rand pico_multicore pico_sync hardware_i2c hardware_adc hardware_timer hardware_pwm FreeRTOS-Kernel FreeRTOS-Kernel-Heap4 libfixmath libfixmatrix libfixkalman)
```

4. `target_include_directories()` is where we **define the locations of the header files for our `.cpp` files.** As you can see, this is literally done by providing the path from the parent directory to any `include/` subdirectories that are needed. For our case, this usually won't have to change.
```cmake
target_include_directories(active_drag_system PUBLIC ${PROJECT_SOURCE_DIR}/include)
```

5. `target_compile_definitions()` is where we **set any in-code compiler flags**, i.e. whether to `#define` various flags or not. For example, below the CMake snippet provided is a block from inside `src/active_drag_system.cpp` that includes a header file full of old launch data *only* if the `DEBUG` flag is defined; if you wanted to do so, the corresponding flag in the `CMakeLists.txt` file just needs to be uncommented so the compiler knows it.
```cmake
target_compile_definitions(active_drag_system PRIVATE
	USE_FREERTOS=1
	#        DEBUG=1
	PICO_STDIO_STACK_BUFFER_SIZE=64 # use a small printf on stack buffer
)
```
```cpp
#if (DEBUG == 1)
	#include "ohio_test_data.h"
#endif
```

6. Finally, we have a couple of definitions just to define various things about the compilation process. `pico_enable_stdio_usb()` and `pico_enable_stdio_uart()` are both called with their respective flags (1 or 0) to tell the compiler to set `stdio`, the standard serial bus, to run a USB instance instead of a UART instance; all this does is tell the system that we want to be able to talk to it over a USB cable. Similarly, `pico_add_extra_outputs()` just tells the compiler that we want it to convert the native output file type (`.elf`) to a `.uf2`, the one that we use; for practical purposes, none of these will ever need to change.
```cmake
pico_enable_stdio_usb(active_drag_system 1)
pico_enable_stdio_uart(active_drag_system 0)

pico_add_extra_outputs(active_drag_system)
```

## CMake Usage
CMake, to put it bluntly, is entirely command based. There are ways to bind it to various plugins for VSCode or other editors, but for this documentation, we'll use just the raw terminal commands to make it as platform-agnostic as possible. First, let's look at a brief high-level list of the steps to compile any CMake project from when you first clone its repository:

1. Clone the repository of whatever project you're working on (wherever you want it to be)
2. Initialize the git submodules of the directory, if necessary
3. Make the `build/` directory within the project's parent directory
4. Initialize CMake inside the `build/` directory
5. Confirm the settings in all `CMakeLists.txt` files
6. Navigate **to the project parent directory** before building
7. Run the build process by calling CMake, and telling it where the `build/` directory is
8. Take the output `.uf2` file that CMake generates, and use your desired tool to upload it!

Steps 1-4 will usually only have to be done once, the first time you clone the repository; once the `build/` directory is made and initialized, it usually won't have to be touched. Now, let's go over the actual commands necessary for each of these steps. 

1. We'll start by cloning the repository somewhere we can find it. We'll make and `cd` into a test directory called `demo`, and clone it via SSH:
```bash
> mkdir demo
> cd demo
> git clone git@github.com:RocketryVT/active-drag-system.git
Cloning into 'active-drag-system'...
remote: Enumerating objects: 938, done.
remote: Counting objects: 100% (166/166), done.
remote: Compressing objects: 100% (88/88), done.
remote: Total 938 (delta 94), reused 86 (delta 78), pack-reused 772 (from 1)
Receiving objects: 100% (938/938), 7.03 MiB | 17.61 MiB/s, done.
Resolving deltas: 100% (578/578), done.
> cd active_drag_system
> pwd
/home/cayenne/demo/active_drag_system/
>_
```

2. Following this, we need to make sure that the git submodules are initialized *before* configuring CMake for the first time.
```bash
> git submodule update --init --recursive
Submodule 'include/FreeRTOS-Kernel' (https://github.com/FreeRTOS/FreeRTOS-Kernel.git) registered for path 'include/FreeRTOS-Kernel'
Submodule 'include/eigen' (https://gitlab.com/libeigen/eigen.git) registered for path 'include/eigen'
Submodule 'include/libfixkalman' (https://github.com/sunsided/libfixkalman.git) registered for path 'include/libfixkalman'
# [there is a lot of output from this command, I'll skip most of it]
Submodule path 'include/pico-sdk/lib/mbedtls': checked out '5a764e5555c64337ed17444410269ff21cb617b1'
Submodule path 'include/pico-sdk/lib/tinyusb': checked out '86ad6e56c1700e85f1c5678607a762cfe3aa2f47'
>_
```

3. Next, we'll make the `build/` directory inside the project and `cd` into it.
```bash
> mkdir build
> cd build
> pwd
/home/cayenne/demo/active_drag_system/build
>_
```

4. Now, we'll initialize CMake **inside the `build/` directory.** This is done just by calling `cmake` and giving it the `..` path to the parent directory, with no flags.
```bash
> cmake ..
PICO_SDK_PATH is /home/cayenne/demo/active-drag-system/include/pico-sdk
Target board (PICO_BOARD) is 'rp2040_micro'.
Using board configuration from /home/cayenne/demo/active-drag-system/include/rp2040_micro.h
Defaulting platform (PICO_PLATFORM) to 'rp2040' since not specified.
-- Defaulting build type to 'Release' since not specified.
Defaulting compiler (PICO_COMPILER) to 'pico_arm_cortex_m0plus_gcc' since not specified.
Configuring toolchain based on PICO_COMPILER 'pico_arm_cortex_m0plus_gcc'
Setting FREERTOS_KERNEL_PATH to /home/cayenne/demo/active-drag-system/include/FreeRTOS-Kernel based on location of FreeRTOS-Kernel-import.cmake
Build type is Release
-- Found Python3: /usr/bin/python3.13 (found version "3.13.7") found components: Interpreter
TinyUSB available at /home/cayenne/demo/active-drag-system/include/pico-sdk/lib/tinyusb/src/portable/raspberrypi/rp2040; enabling build support for USB.
BTstack available at /home/cayenne/demo/active-drag-system/include/pico-sdk/lib/btstack
cyw43-driver available at /home/cayenne/demo/active-drag-system/include/pico-sdk/lib/cyw43-driver
lwIP available at /home/cayenne/demo/active-drag-system/include/pico-sdk/lib/lwip
mbedtls available at /home/cayenne/demo/active-drag-system/include/pico-sdk/lib/mbedtls
-- Configuring done (0.2s)
-- Generating done (0.0s)
-- Build files have been written to: /home/cayenne/demo/active-drag-system/build
>_
```

5. Looking at the `CMakeList.txt` files can be done however you prefer; usually, they'll already be ready to go from when you clone the repository, so I won't add any additional terminal commands here.
6. Once you've confirmed that your `CMakeLists.txt` files are set up appropriately, we need to make sure that we're in the project's parent directory; **if you're still in `build/`, `cd` back down to the parent before continuing.**
```bash
> cd ..
> pwd
/home/cayenne/demo/active_drag_system
>_
```
7. Finally, to actually run the cmake build itself, there are **two steps** that need to be done. The first is to set the path to the `build/` directory you made via `cmake -B`, and the second is to actually perform the build with `cmake --build`. This is done in two separate steps, because in the first one you'll also set any flags for the CMake process that you want; for our purposes, this mainly means setting `COMPILE_TOOLS` to 0 or 1, depending on if you want to build the additional scripts in the `tools/` subdirectory or not.
```bash
> cmake -B build -DCOMPILE_TOOLS=0
PICO_SDK_PATH is /home/cayenne/demo/active-drag-system/include/pico-sdk
Target board (PICO_BOARD) is 'rp2040_micro'.
Using board configuration from /home/cayenne/demo/active-drag-system/include/rp2040_micro.h
Pico Platform (PICO_PLATFORM) is 'rp2040'.
Build type is Release
TinyUSB available at /home/cayenne/demo/active-drag-system/include/pico-sdk/lib/tinyusb/src/portable/raspberrypi/rp2040; enabling build support for USB.
BTstack available at /home/cayenne/demo/active-drag-system/include/pico-sdk/lib/btstack
cyw43-driver available at /home/cayenne/demo/active-drag-system/include/pico-sdk/lib/cyw43-driver
lwIP available at /home/cayenne/demo/active-drag-system/include/pico-sdk/lib/lwip
mbedtls available at /home/cayenne/demo/active-drag-system/include/pico-sdk/lib/mbedtls
-- Configuring done (0.1s)
-- Generating done (0.0s)
-- Build files have been written to: /home/cayenne/demo/active-drag-system/build
> cmake --build build
[  2%] Built target bs2_default
[  4%] Built target bs2_default_library
[100%] Built target active_drag_system
# [there will usually be a lot more output during this stage]
>_
```
8. With the above done, the code has been compiled and is ready to upload using the method of your choice! The outputs can be found inside the `build/` directory you made earlier; they'll be in a subdirectory inside it matching the one the `CMakeLists.txt` where you called `add_executable()`was in in the parent directory. For example, the `active_drag_system` executable is defined in the `CMakeLists.txt` file [found in the src subdirectory](https://github.com/RocketryVT/active-drag-system/blob/320d5c7a5b3e29f38a31e0a5399f47f008e036fd/src/CMakeLists.txt#L1), so our compiled output file will be in `build/src`.
```bash
> ls build/src
active_drag_system.bin  active_drag_system.elf      active_drag_system.hex  CMakeFiles           Makefile
active_drag_system.dis  active_drag_system.elf.map  active_drag_system.uf2  cmake_install.cmake
>_ 
```
To use your freshly compiled code, you would upload `build/src/active_drag_system.uf2` using the method of your choice.

## Helpful CMake Commands
- To **configure and build your code** at the same time, you can just run them sequentially; running this line as one command will do the full build process.
```bash
cmake -B build -DCOMPILE_TOOLS=0; cmake --build build -j8
```
- To **do the above and then *upload*** the code afterwards, you can keep appending more commands to the end; for example, you can add a `picotool` call after the build file is done if you know the name of your desired executable.
```bash
cmake -B build -DCOMPILE_TOOLS=0; cmake --build build -j8; picotool load -f build/src/active_drag_system.uf2
```