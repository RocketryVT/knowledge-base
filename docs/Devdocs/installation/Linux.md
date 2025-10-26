## Intro: Down in the Weeds
If you're using a Linux system, your installation process is going to involve mainly configuring the backend manually. This does mean that you can use any editor that you want to, but it requires a bit more initial setup. We'll be installing a bunch of system packages, cloning a couple of repositories and building/installing them, and setting and modifying a couple of system variables. This guide assumes you're familiar with the basic usage of a terminal and are comfortable moving around between directories and executing commands; if you aren't, check out the accompanying guide labeled `Bash Terminal` in the sidebar for a refresher.

Note that for the purposes of this guide, we'll assume that you're using Debian Linux (i.e. Ubuntu), and as such, your default package manager is aptitude (`apt`). If you're using a different non-Debian distro with a separate package manager, we'll assume you're familiar enough with its usage to be able to look up and install the packages listed here, as the rest of the steps listed should be the same.
## Manual Installation Guide
Pretty much this entire process will be done in a terminal environment, but luckily it shouldn't take long - so open one up and get comfy. Contrary to the other tutorial pages (which are formatted with fake `>_` characters to simulate a terminal for demonstration), all of the command boxes in this section should be ready to copy-paste into a terminal and execute directly.

1. First, install all the necessary packages for development by running these commands. This may take several minutes; the packages named something like `gcc-arm-none-eabi` or `build-essential` are the full C++ toolchains necessary for pico development, so they may take up a couple gigabytes.
```bash
sudo apt update
sudo apt install git cmake gcc-arm-none-eabi libnewlib-arm-none-eabi build-essential libstdc++-arm-none-eabi-newlib pkg-config libusb-1.0-0-dev minicom
```

2. Next, we only need to install `picotool` natively should you wish to upload your code using it. Clone the repository itself wherever you'd prefer, then `cd` into it:
```bash
git clone https://github.com/raspberrypi/picotool.git
cd picotool
```

3. Finally, run a `CMake` build on the repo, and call `sudo make install` afterwards to set the appropriate environment variables.
```bash
mkdir build
cd build
cmake ..
sudo make install
```

Note that normally, there would be additional steps included for cloning and building the `pico-sdk` repo as well; this repo is all of the SDK files and headers necessary for development with the RP2040/2350. Luckily, it's included as a submodule in our shared repos, so it is installed and configured automatically when you call `git submodule update --init --recursive` at the top level in your development repo; that saves you a couple steps here in the installation process.

## Script Install (for RPi-Native ONLY)
Luckily for us, the Raspberry Pi Foundation recognized that this process is a bit of a pain, and so they automated much of it for us via a readily-available install script - the caveat is that it's designed specifically to *only* be used on another Raspberry Pi SBC, which is already running Raspbian. If you want to do all of your development while SSH'd into a spare Raspberry Pi you have laying around or are using one as a computer itself for your development environment, then this script is your friend - the guide for using it can be found [here](https://datasheets.raspberrypi.com/pico/getting-started-with-pico.pdf#%5B%7B%22num%22%3A34%2C%22gen%22%3A0%7D%2C%7B%22name%22%3A%22XYZ%22%7D%2C115%2C709.282%2Cnull%5D), and it's not worth it to duplicate the exact steps listed in it into this guide.

## Uploading Guide - Picotool
Once you have all of these packages natively installed and configured, to actually upload and build your code you have two options. You can either run `picotool` (which was installed in steps 2 and 3 above) which includes a bunch of scripts for automatically rebooting the microcontroller and uploading the files without having to un/re-plug the USB cable into the board, or you can do it manually via doing the latter and manually putting the board into boot mode and copying the file over yourself. This guide will only detail using `picotool`, as it is by far the easier option; should you want to drag and drop the file manually, the process is exactly the same as on Windows, with some nice extra Linux sparkle for mounting the board as a USB drive to do so.

Before using `picotool`, we quickly just need to make sure that your device recognizes the board and is ready to upload code to it. In a terminal, run the command `lsusb` (included in the `libusb-1.0-0-dev` package we had you install in step 1) - this will list out all of the devices connected to your USB ports, and will likely include a couple of adapters, hubs, and even sometimes full devices like WiFi or Bluetooth hosts (which means those devices are internally communicating with your computer over USB). 
```bash
> lsusb
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 003 Device 002: ID 32ac:0002 Framework HDMI Expansion Card
Bus 003 Device 003: ID 27c6:609c Shenzhen Goodix Technology Co.,Ltd. Goodix USB2.0 MISC
Bus 003 Device 004: ID 8087:0032 Intel Corp. AX210 Bluetooth
Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
```
The one you're looking for is `Raspberry Pi Pico` - if you see one, that means your system recognizes the board, and is able to talk to it properly. If you *don't* see one, and you've already uploaded code, try re-plugging in the USB cable, and if that doesn't work, try re-plugging in the cable while holding down the `BOOTSEL` button on the board to force it into upload mode. If you *haven't* yet uploaded code, try both of the above, and if those don't work, try a different port and/or a different cable. **It is unfortunately quite common for micro-USB cables to only have their power lines properly wired internally, especially if the cable is "flat"** - there's a good chance you just need to find a different cable with its data lines wired as well, and for lack of a better term, these cables tend to be "nicer" than their power-only counterparts.

//TODO: Include sample output with pico listed as formatted

Once you've done the above and confirmed that you can see/talk to the board, the process for using `picotool` is nice and easy - just run the following command after performing a build with `cmake`, and set the path flag (below example is `build/src/your_executable.uf2`) to wherever the `.uf2` file that `cmake` generated was spit out. Note that your build files will almost *always* be stored at `build/src/[name].uf2`, inside the build folder and matching subdirectory for where your executable was configured in its `CMakeLists.txt`. The `-f` flag tells the tool to `force` the board into boot mode so it can copy the file into it, without any external intervention from you.
```bash
picotool load -f build/src/your_executable.uf2
```

Do note: if the executable that you supply does not exist, `picotool` will reboot the device to upload it, but fail in the process; **this means whatever code was on the board beforehand will stop running, but no new code will replace it.** In our experience, it is common for a successful `picotool` upload following a non-successful one (i.e. if you're using a compound command to build and then upload and try to upload broken code, then fix it and try again with compiling code) to require re-plugging in the USB cable to reboot the device. Be wary of this - **always try re-plugging in the cable and trying again before modifying your code if your hardware is being strange.**
## Debugging Guide - Minicom
To interact with the board over a wire and use the serial tool, we luckily only need to run a terminal-native serial interpreter, as Linux natively exposes your USB ports to the terminal (as it exposes *everything* to the terminal). The keen-eyed reader may have spotted that we already had you install `minicom` as part of the initial big block of packages installed in step 1 above, so you should already have it. Now, we just have to tell you how to use it.

Before booting `minicom` itself, we need to figure out which internal port your OS has assigned to the board so we can tell it what to "listen" to. Assuming that your board is plugged in and operating as usual (i.e. you've just uploaded code and the board is operating as standard), run the command `ls /dev` in a terminal - the output of this will likely be quite long, which is perfectly normal. For reasons outside the scope of explaining in this document, Linux internally recognizes ***every*** device connected to it as a file - those files are stored in the `/dev` directory, so effectively what we're doing is listing out all the devices internally connected to the OS.
```bash
> ls /dev
acpi_thermal_rel  fd         loop6         nvme0n1p4  tty    tty27  tty46  tty8    ttyS26       vcs1   vcsu6
autofs            full       loop7         nvme0n1p5  tty0   tty28  tty47  tty9    ttyS27       vcs2   vcsu7
block             fuse       loop8         nvram      tty1   tty29  tty48  ttyS0   ttyS28       vcs3   vfio
btrfs-control     gpiochip0  loop-control  port       tty10  tty3   tty49  ttyS1   ttyS29       vcs4   vga_arbiter
bus               gpiochip1  mapper        ppp        tty11  tty30  tty5   ttyS10  ttyS3        vcs5   vhci
char              hidraw0    mei0          psaux      tty12  tty31  tty50  ttyS11  ttyS30       vcs6   vhost-net
console           hidraw1    mem           ptmx       tty13  tty32  tty51  ttyS12  ttyS31       vcs7   vhost-vsock
core              hidraw2    mqueue        ptp0       tty14  tty33  tty52  ttyS13  ttyS4        vcsa   watchdog
cpu               hpet       mtd           pts        tty15  tty34  tty53  ttyS14  ttyS5        vcsa1  watchdog0
cpu_dma_latency   hugepages  mtd0          random     tty16  tty35  tty54  ttyS15  ttyS6        vcsa2  watchdog1
cros_ec           hwrng      mtd0ro        rfkill     tty17  tty36  tty55  ttyS16  ttyS7        vcsa3  zero
cuse              input      mtd1          rtc        tty18  tty37  tty56  ttyS17  ttyS8        vcsa4
disk              kmsg       mtd1ro        rtc0       tty19  tty38  tty57  ttyS18  ttyS9        vcsa5
dma_heap          kvm        net           shm        tty2   tty39  tty58  ttyS19  udmabuf      vcsa6
dri               log        ng0n1         snapshot   tty20  tty4   tty59  ttyS2   uhid         vcsa7
drm_dp_aux0       loop0      null          snd        tty21  tty40  tty6   ttyS20  uinput       vcsu
drm_dp_aux1       loop1      nvme0         stderr     tty22  tty41  tty60  ttyS21  urandom      vcsu1
drm_dp_aux2       loop2      nvme0n1       stdin      tty23  tty42  tty61  ttyS22  usb          vcsu2
drm_dp_aux3       loop3      nvme0n1p1     stdout     tty24  tty43  tty62  ttyS23  userfaultfd  vcsu3
drm_dp_aux4       loop4      nvme0n1p2     tpm0       tty25  tty44  tty63  ttyS24  userio       vcsu4
fb0               loop5      nvme0n1p3     tpmrm0     tty26  tty45  tty7   ttyS25  vcs          vcsu5
```

What we're looking for is a device labeled `tty[something]` - this is the port that the board is sending in its serial data over. You may have to run the command twice with and without the board to figure out what it's connected to, as it won't be listed in `/dev` if it's not plugged in. 99% of the time (if only one board is plugged in), this will end up being `ttyUSB0` or `ttyACM0`, for a full path of `/dev/ttyUSB0` or `/dev/ttyACM0`.

//TODO: Test/add alternative method using `ls /dev/serial/by-id` instead of just `ls /dev`

With this port found, opening it with `minicom` is luckily *very* simple - just a single command. Run `minicom -D /dev/port_path`, where `port_path` gets replaced with the path to the device port that you found in the previous step. 

With `minicom` open, the board should echo any characters you type back to you - you'll know that it's working if when you type things (which will *send* those characters over serial), they show up in the `minicom` window (which means you *received* those characters over serial after the board sent them back). If you can see what you type, congratulations! You're now communicating with the board, and are ready to start debugging your code and running tests - see the `Serial Tool` guide on the sidebar for how to use the serial tool to execute commands on the board.