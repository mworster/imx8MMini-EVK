# Getting Started with the DevKit

## Hardware Overview

Compute Module:

    • i.MX 8M Mini applications processor with up to five cores:
      – 4× Arm® Cortex®-A53
      – 1× Arm Cortex-M4
    • 2 GB, 32-bit LPDDR4
    • eMMC 5.1, 16 GB
    • 32MB QSPI NOR flash
    • Power Management IC (PMIC)
    • MIMO 1x1 Wi-Fi 802.11a/b/g/n/ac and BT4.1

Base Board:

    • MicroSD card connector
    • 2 USB2.0 Type-C connectors, Port2 is the only power supply port
    • 1 Gbps Ethernet
    • Mini-SAS MIPI-DSI connector for display
    • Mini-SAS MIPI-CSI connector for camera
    • USB to serial converter for debug
    • Infrared receiver
    • LEDs for power indication and general-purpose use
    • M.2 connector for Wi-Fi/BT (PCIe, I2C, etc.)
    • 3.5 mm audio jack for amplified speakers

![](https://github.com/mworster/imx8MMini-EVK/blob/main/i.MX_8M_EVK_HW.jpg)

Resource[1] list this hardware information and more.

## Drivers
The serial communication with the EVK may, or may not, require drivers to be installed. Connect the 
micro-B end of a USB cable into debug port J1701 and the other end of the cable to a PC acting as a
host terminal. If the device manager (Windows) shows an unknown device then you'll need to download
and install drivers.

Windows users may need to update the serial drivers on your computer. The drivers can be found at
https://www.ftdichip.com/Drivers/VCP.htm

Assuming a Windows setup, you'll want to navigate to the VCP Driver page and scroll down to find
the `setup executable` link. That will lead you to a downloadable .exe file with a wizard to install
the latest drivers:

![](https://github.com/mworster/imx8MMini-EVK/blob/main/InstallDrivers.jpg)

Resource [1] contains driver information.

## Initial Booting

Once the drivers are installed you can initially boot the board and get output. The EVK comes with
a pre-loaded Android image in eMMC and is good to boot from it on arival. 


### Setup

The wiring setup is pretty simple. 

  1. Verify boot switches are set to boot from eMMC.
      * SW1101 - 0110110001
      * SW1102 - 0001010100
  2. Connect µUSB for serial communication. 
  3. Start terminal output (see next section)
  4. Connect the USB Type-C plug of the Power Supply to J302 (USB Type-C Port2), then power up the board using switch SW101.

See resource [1] for setup and other boot modes.

### Getting Terminal Output

Resource [1] contains the basics of drivers and terminal setup. When interacting with the i.MX8M EVK, there should be 2 serial
ports listed in the device manager. The first one is for the M4 and the second is for the A53. When you initially check your 
setting in the device manager you should see something like the below reference image:

![](https://github.com/mworster/imx8MMini-EVK/blob/main/no_imx8m_plug.jpg)

After Setup Step 2 (before power is applied), the device manager should update to show the two new ports:

![](https://github.com/mworster/imx8MMini-EVK/blob/main/yes_imx8m_plug.jpg)

In this section I specifcally setup for Windows 10. On Windows I used [TeraTerm](https://tera-term.en.lo4d.com/windows)
for serial communication. 
  
  1. Download TeraTerm and install on your PC
  2. Check device manager for the A53 COM port number: `USB Serial Port (COMX)`
  3. Open TeraTerm to COMX(larger of the two, 8 in my case) and set to a `115200` baud rate

### Initial Output

Once the boot is successful and TeraTerm is connected and gathering data as it should, you'll see something
along the lines of the below (truncated) output:

    ...
    U-Boot 2018.03 (Dec 19 2018 - 17:39:10 +0800)

    CPU:   Freescale i.MX8MMQ rev1.0 1800 MHz (running at 1200 MHz)
    CPU:   Commercial temperature grade (0C to 95C) at 35C
    Reset cause: POR
    Model: FSL i.MX8MM EVK board
    DRAM:  2 GiB
    TCPC:  Vendor ID [0x1fc9], Product ID [0x5110], Addr [I2C1 0x52]
    Power supply on USB2
    TCPC:  Vendor ID [0x1fc9], Product ID [0x5110], Addr [I2C1 0x50]
    MMC:   FSL_SDHC: 0, FSL_SDHC: 1
    Loading Environment from MMC... *** Warning - bad CRC, using default environment
    ...
    Booting using the fdt blob at 0x43400000
    Using Device Tree in place at 0000000043400000, end 000000004340da7d

    Starting kernel ...

    [    0.000000] Booting Linux on physical CPU 0x0
    [    0.000000] Linux version 4.14.78 (bamboo@szandroidmpu-PowerEdge-T430) (gcc version 4.9.x 20150123 (prerelease) (GCC)) #1 SMP PREEMPT Wed Dec 19 17:42:18 CST 2018
    [    0.000000] Boot CPU: AArch64 Processor [410fd034]
    [    0.000000] Machine model: FSL i.MX8MM EVK board 
    ...

This shows everything is booting successfully and you have a 2018.03 u-boot with a 4.14.78 kernel. See Resource [2] for full initial log.

## Flashing New Images

For many years, NXP used MFGTools for flashing images on the i.MX family of devices. As of 2018 this has been replaced with the Universal 
Update Utility (UUU).

### Universal Update Utility (UUU) Overview

NXP's UUU is used for loading bootloader/recovering devices over USB recovery (SDP) protocol. It also loads any binary in RAM over USB for
flashing. It also has advanced scripting and fastboot protocol support. Universal use on Linux, Windows, and MacOS. 

The source lives at: https://github.com/NXPmicro/mfgtools

And releases are available at: https://github.com/NXPmicro/mfgtools/releases

The UUU is a command line program, so clicking on the `uuu.exe` will seemingly do nothing, and this needs to be run via a cmdprompt or 
Windows PowerShell window. 

### Setting up to use the UUU

In order to use the UUU on the i.MX8M EVK, we need to set BOOT_MODE_0 and BOOT_MODE_1 pins to serial download mode:
  * SW1101 - 10XXXXXXXX

The below image shows the two pins in the default configuration, these needs to be switched for serial download.

![](https://github.com/mworster/imx8MMini-EVK/blob/main/Serial_Download_jumpers.jpg)

Once serial download has been selected, next we connect a USB type C cable from the host computer to the EVK on Port 1 (Download).

### Install Programming Tool

The STM32CubeProgrammer tool is used to flash the STM32MP157x-DKx Discovery kit, the µSD card is default flash device, and the trusted boot chain is the default solution for booting.
The tools[7] can be downloaded from the link [here.](https://www.st.com/en/development-tools/stm32cubeprog.html). Once the package has been downloded, it can be extracted
and (on windows) `SetupSTM32CubeProgrammer-2.5.0.exe` can be run to install the tools. Follow the wizard to install the tools on your PC in the default paths. 

### Setup the board for flashing

Let's flash the downloaded image on the microSD card:

    Set the boot switches (1) to the off position
    Connect the USB Type-C™ (OTG) port (2) to the host PC that contains the downloaded image
    Insert the delivered microSD card into the dedicated slot (3)
    Connect the delivered power supply to the USB Type-C™ port (4)
    Press the reset button (5) to reset the board
    
![](https://github.com/mworster/stm32mp157-dev-kit/blob/master/STM32MP157x-DKx_flashing_configuration.png)

### Flashing Commands

Now we have the board setup for flashing, the tools for flashing, and the images to be flashed. The next step is to run the commands. 
When we installed the tools, this included a set of CLIs which we'll be using for this work. On my machine the default path was:
`C:\Program Files\STMicroelectronics\STM32Cube\STM32CubeProgrammer` and in the associated `bin` directory we can find the various exe's which will be used here.

    cd "C:\Program Files\STMicroelectronics\STM32Cube\STM32CubeProgrammer\bin"
    C:\Program Files\STMicroelectronics\STM32Cube\STM32CubeProgrammer\bin> .\STM32_Programmer_CLI.exe -l usb
      -------------------------------------------------------------------
                       STM32CubeProgrammer v2.5.0
      -------------------------------------------------------------------

      =====  DFU Interface   =====

      Total number of available STM32 device in DFU mode: 1

        Device Index           : USB1
        USB Bus Number         : 002
        USB Address Number     : 002
        Product ID             : DFU in HS Mode @Device ID /0x500, @Revision ID /0x0000
        Serial number          : 001C002E3438511538333630
        Firmware version       : 0x0110
        Device ID              : 0x0500

This first command verifes we're connected and read to flash. This also notes we're using the "USB1" device. This gets fed into the actuall flashing command:

    C:\Program Files\STMicroelectronics\STM32Cube\STM32CubeProgrammer\bin> .\STM32_Programmer_CLI.exe -c port=usb1 -w FlashLayout_sdcard_stm32mp157a-dk1-trusted.tsv
    
The file pointed to by the `w` flag, is a text based file (tsv) that defines the partitions we're going to create and the location of the binaries to populate each one:

    #Opt	Id	Name	Type	IP	Offset	Binary
    -	0x01	fsbl1-boot	Binary	none	0x0	arm-trusted-firmware/tf-a-stm32mp157a-dk1-serialboot.stm32
    -	0x03	ssbl-boot	Binary	none	0x0	bootloader/u-boot-stm32mp157a-dk1-trusted.stm32
    P	0x04	fsbl1	Binary	mmc0	0x00004400	arm-trusted-firmware/tf-a-stm32mp157a-dk1-trusted.stm32
    P	0x05	fsbl2	Binary	mmc0	0x00044400	arm-trusted-firmware/tf-a-stm32mp157a-dk1-trusted.stm32
    PD	0x06	ssbl	Binary	mmc0	0x00084400	bootloader/u-boot-stm32mp157a-dk1-trusted.stm32
    P	0x21	boot	System	mmc0	0x00284400	st-image-bootfs-openstlinux-weston-stm32mp1.ext4
    P	0x22	vendorfs	FileSystem	mmc0	0x04284400	st-image-vendorfs-openstlinux-weston-stm32mp1.ext4
    P	0x23	rootfs	FileSystem	mmc0	0x05284400	st-image-weston-openstlinux-weston-stm32mp1.ext4
    P	0x24	userfs	FileSystem	mmc0	0x33C84400	st-image-userfs-openstlinux-weston-stm32mp1.ext4

## Building Our Own Files

To build our own files, we'll select the Developer Package[8]. This package allows developments on top of the STM32MPU Embedded Software distribution, 
or, in our case, to replace the Starter Package pre-built binaries. The Developer Package is generated from the Distribution Package.

We'll assume for this section that building is done on a Linux machine and all directions in this section will be for Linux (Debian) unless otherwise noted.

### Host Machine Prep

Prior to building anything on our host machine, we need to ensure the correct prerequisits are installed. 
Assuming a Diabian based OS, the following set of commands will install all required packages:

    $> sudo apt-get update
    $> sudo apt-get install gawk wget git-core diffstat unzip texinfo gcc-multilib build-essential chrpath socat cpio python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev pylint3 pylint xterm
    $> sudo apt-get install make xsltproc docbook-utils fop dblatex xmlto
    $> sudo apt-get install libncurses5 libncurses5-dev libncursesw5-dev libssl-dev linux-headers-generic u-boot-tools device-tree-compiler bison flex g++ libyaml-dev

NOTE: More for fun than anything required... You can check the number of cores your machine has to be used in the `-j` flag during the build process via the following command:

    $> grep -P '^core id\t' /proc/cpuinfo | sort -u | wc -l

### Prereqs on Target

Before flashing anything created on our own, you must flash the Starter Package binaries. This enforces that all partitions are populated, verifies the
flashing procedure, and the boards functionality. 
It should also be noted that the dev contents and Starter binaries will be from the same source if downloaded at the same time, this helps ensure 
continuity in test.

### SDK

The STM32MP1 SDK is delivered through a tarball file named : [en.SDK-x86_64-stm32mp1-openstlinux-5-4-dunfell-mp1-20-11-12.tar.xz](http://st.com/content/ccc/resource/technical/software/sw_development_suite/group0/10/c7/78/3d/89/d1/43/c9/stm32mp1dev_yocto_sdk/files/SDK-x86_64-stm32mp1-openstlinux-5-4-dunfell-mp1-20-11-12.tar.xz/jcr:content/translations/en.SDK-x86_64-stm32mp1-openstlinux-5-4-dunfell-mp1-20-11-12.tar.xz)

Once downloaded, you'll need to uncompress the tarball file to get the SDK installation script, and possibly update the permissions:

    $> tar xvf en.SDK-x86_64-stm32mp1-openstlinux-5-4-dunfell-mp1-20-11-12.tar.xz
    $> chmod +x stm32mp1-openstlinux-5.4-dunfell-mp1-20-11-12/sdk/st-image-weston-openstlinux-weston-stm32mp1-x86_64-toolchain-3.1-openstlinux-5.4-dunfell-mp1-20-11-12.sh

Run the SDK installation script. Use the -d <SDK installation directory absolute path> option to specify the absolute path to the directory in which you want to install the SDK (<SDK installation directory>)

    $> ./stm32mp1-openstlinux-5.4-dunfell-mp1-20-11-12/sdk/st-image-weston-openstlinux-weston-stm32mp1-x86_64-toolchain-3.1-openstlinux-5.4-dunfell-mp1-20-11-12.sh -d <working directory absolute path>/Developer-Package/SDK

A successful installation outputs the following log:

    ST OpenSTLinux - Weston - (A Yocto Project Based Distro) SDK installer version 3.1-openstlinux-5-4-dunfell-mp1-20-11-12
    ===========================================================================================
    You are about to install the SDK to "<working directory absolute path>/Developer-Package/SDK". Proceed [Y/n]? 
    Extracting     SDK................................................................................................................................................................................................................done
    Setting it up...done
    SDK has been successfully set up and is ready to be used.
    Each time you wish to use the SDK in a new shell session, you need to source the environment setup script e.g.
     $> . <working directory absolute path>/Developer-Package/SDK/environment-setup-cortexa7t2hf-neon-vfpv4-ostl-linux-gnueabi
 
 After this initial installulation, you'll need to run a single command each time you're in a new window starting to work on cross-compiling the code:

    $> source <SDK installation directory>/environment-setup-cortexa7t2hf-neon-vfpv4-ostl-linux-gnueabi

And the following check allows you to ensure that the environment is correctly setup:

    $> echo $ARCH
    arm

### Linux Kernel

For building the Linux kernel we'll follow the steps from the developer package[9]. This will end up building the following partitions:

*  the bootfs partition that contains the Linux kernel U-Boot image (uImage) and device tree
*  the rootfs partition that contains the Linux kernel modules


This initially starts by downloading the OpenSTLinux version of the Linux kernel. The latest package can be found via the link on ST's 
site, but to grab the (current) most recent we can use the following commands:

    $> wget https://st.com/content/ccc/resource/technical/sw-updater/firmware2/group0/c4/f4/cd/e7/dc/4f/44/c4/STM32cube_Standard_A7_BSP_components_kernel/files/SOURCES-kernel-stm32mp1-openstlinux-5-4-dunfell-mp1-20-11-12.tar.xz/jcr:content/translations/en.SOURCES-kernel-stm32mp1-openstlinux-5-4-dunfell-mp1-20-11-12.tar.xz
    $> tar -xvf en.SOURCES-kernel-stm32mp1-openstlinux-5-4-dunfell-mp1-20-11-12.tar.xz
    $> cd stm32mp1-openstlinux-5-4-dunfell-mp1-20-11-12/sources/arm-ostl-linux-gnueabi
    $> tar -xvf linux-5.4.56.tar.xz

This set of commands will download the Linux source code for the 5.4 kernel along with a series of patch files and a README file on how to use everything. 

#### Patching Kernel

The next step is to patch the source code to build:

    $> cd linux-5.4.56
    $> for p in `ls -1 ../*.patch`; do patch -p1 < $p; done

These commands will move you into the core Linux source code and patch all ST provided files (listed in the partent directory) to prep the files. We'll be building

#### Configuring the Kernel

Configuration and build of the kernel code can be done in the current source code directory, or in a targeted directory. To simplify we'll just use the curernt directory
in the following examples. Configure for ST's code starts with "fragment"s on the config. We need to enter the kernel code directory (assuming we're not already there) 
and run `make`:

    $> cd <directory to kernel source code>
    $> make ARCH=arm multi_v7_defconfig fragment*.config

If there are multiple fragments, apply them manually one by one (We will not be using this):

    $> scripts/kconfig/merge_config.sh -m -r .config ../fragment-01-xxxx.config
    $> scripts/kconfig/merge_config.sh -m -r .config ../fragment-02-xxxx.config
    ...
    $> yes '' | make oldconfig

or, via a `for` loop (Run this version for our demo code):

    $> for f in `ls -1 ../fragment*.config`; do scripts/kconfig/merge_config.sh -m -r .config $f; done
    $> yes '' | make ARCH=arm oldconfig

*NOTE* Two types of fragments are provided:

* official fragments (fragment-xxx.config)
* optional fragments as example (optional-fragment-xxx.config) to add a feature not enabled by default.

The order in which fragments are applied is determined by the number of the fragment filename (fragment-001, fragment-002, e.g.).
Please pay special attention to the naming of your optional fragments to ensure you select the right features.

#### Build the Kernel

You MUST compile from the directory on which the configuration has been done (i.e. the directory which contains the '.config' file).

It's preconized to use the method with dedicated build directory for a better managment of changes made on source code (as all build artifacts will be located
inside the dedicated build directory). However since we're just "going fast" the following notes are for building within the source dirctory.

Compile and install on the current source code directory:

    $> cd <directory to kernel source code>
        
Build kernel images (uImage and vmlinux) and device tree (dtbs)

    $> make -j<num_of_cores> ARCH=arm uImage vmlinux dtbs LOADADDR=0xC2000040

Build kernel module

    $> make -j<num_of_cores> ARCH=arm modules
    
Generate output build artifacts

    $> make ARCH=arm INSTALL_MOD_PATH="$PWD/install_artifact" modules_install
    $> mkdir -p $PWD/install_artifact/boot/
    $> cp $PWD/arch/arm/boot/uImage $PWD/install_artifact/boot/
    $> cp $PWD/arch/arm/boot/dts/st*.dtb $PWD/install_artifact/boot/

#### Kernel Artifacts

Generated files are :

    $> $PWD/install_artifact/boot/uImage
    $> $PWD/install_artifact/boot/<stm32-boards>.dtb

## Resources

1. https://github.com/mworster/imx8MMini-EVK/blob/main/8MMINIEVKQSG.PDF
2. https://github.com/mworster/imx8MMini-EVK/blob/main/initial_eeprom_boot.log
3. https://www.nxp.com/design/development-boards/i-mx-evaluation-and-development-boards/evaluation-kit-for-the-i-mx-8m-mini-applications-processor:8MMINILPD4-EVK?tid=vanimx8mminievk
4. https://github.com/NXPmicro/mfgtools/releases
5. 
6. https://wiki.st.com/stm32mpu/wiki/How_to_get_Terminal
7. https://wiki.st.com/stm32mpu/wiki/STM32MP15_Discovery_kits_-_Starter_Package#Downloading_the_image_and_flashing_it_on_the_board
8. https://st.com/content/ccc/resource/technical/software/firmware/group0/e2/88/93/37/bc/48/41/cb/STM32MP15_OpenSTLinux_Starter_Package/files/FLASH-stm32mp1-openstlinux-5-4-dunfell-mp1-20-11-12.tar.xz/jcr:content/translations/en.FLASH-stm32mp1-openstlinux-5-4-dunfell-mp1-20-11-12.tar.xz
9. https://wiki.st.com/stm32mpu/wiki/STM32MP15_Discovery_kits_-_Starter_Package#Installing_the_tools
10. https://wiki.st.com/stm32mpu/wiki/STM32MP1_Developer_Package
11. https://wiki.st.com/stm32mpu/wiki/STM32MP1_Developer_Package#Installing_the_Linux_kernel
