# Getting Started with the DevKit

## Drivers
Most of this information comes, indirectly, from [1] 

    Before connecting STM32MP157A-DK1 or STM32MP157C-DK2 to a Windows® (7, 8, or 10) PC via USB, 
    a driver for ST-LINK/V2.1 must be installed (not required for Windows® 10). It is available from 
    the www.st.com website.In case the STM32MP157A-DK1 or STM32MP157C-DK2 Discovery kit is connected
    to the PC before the driver isinstalled, some Discovery kit interfaces may be declared as 
    “Unknown” in the PC device manager. In this case,the user must install the dedicated driver 
    files, and update the driver of the connected device from the devicemanager.

    The ST-LINK/V2-1 embeds a firmware upgrade mechanism for in-situ upgrade through the USB port. 
    As firmwaremay evolve during the lifetime of the ST-LINK/V2-1 product (addition of new 
    functionalities, bug fixes, or supportof new microprocessor families), it is recommended to visit
    periodically the www.st.com website before starting touse the STM32MP157A-DK1 or STM32MP157C-DK2
    Discovery kit to stay up-to-date with the latest firmwareversion.

The driver refered to by the above paragraphs can be found at [2] the "STSW-LINK009". Once downloaded, this can be
installed via the `stlink_winusb_install.bat` in administrator mode.

## Initial Booting

Once the drivers are installed you can initially boot the board and get output. The DevKit comes with
a pre-loaded µSD card and is good to boot from it on arival. 

### Setup

The wiring setup is pretty simple. 

  1. Connect power (USB-C near Ethernet), the PC is enough to run it. 
  2. Connect µUSB (near HDMI) for serial communication. 
  3. Start terminal output (see next section)
  4. reboot the board

Step 4 isn't strictly required, but the ordering is important and that means you'll initially miss the
boot logs, just by default. 

### Getting Terminal Output

Resource [4] contains all the information for terminal interactions for both Linux and Windows both local
and remote. Specifcally I setup for local on Windows 10. Initially I attempted PuTTY for serial 
communication, however that didn't work. Not sure why. However [TeraTerm](https://tera-term.en.lo4d.com/windows)
works perfectly. 
  
  1. Download TeraTerm and install on your PC
  2. Check device manager for the STLink COM port number: `STMicroelectronics STLink Virtual COM Port (COM6)`
  3. Open TeraTerm to COMX(X=6 for me) and set to a `115200` baud rate

### Initial Output

Once the boot is successful and TeraTerm is connected and gathering data as it should, you'll see something
along the lines of the below (truncated) output:

    NOTICE:  CPU: STM32MP157AAC Rev.B
    NOTICE:  Model: STMicroelectronics STM32MP157C-DK2 Discovery Board
    NOTICE:  Board: MB1272 Var1 Rev.C-01
    INFO:    Reset reason (0x14):
    INFO:      Pad Reset from NRST
    ...
    INFO:    SP_MIN: Initializing runtime services
    INFO:    SP_MIN: Preparing exit to normal world

    U-Boot 2018.11-stm32mp-r4 (Jun 25 2020 - 12:01:15 +0000)

    CPU: STM32MP157AAC Rev.B
    Model: STMicroelectronics STM32MP157C-DK2 Discovery Board
    ...
    XIP Kernel Image ... OK
    Using Device Tree in place at c4000000, end c4014f69

    Starting kernel ...

    [    0.000000] Booting Linux on physical CPU 0x0
    [    0.000000] Linux version 4.19.94 (oe-user@oe-host) (gcc version 8.2.0 (GCC)) #1 SMP PREEMPT Thu Jun 25 11:54:35 UTC 2020
    [    0.000000] CPU: ARMv7 Processor [410fc075] revision 5 (ARMv7), cr=10c5387d    
    ...

This shows everything is booting successfully and you have a 2018.11 u-boot with a 4.19.94 kernel.

## Flashing New Images

### Default Images

ST has various ["Packages"](https://wiki.st.com/stm32mpu/wiki/Which_STM32MPU_Embedded_Software_Package_better_suits_your_needs). In a nutshell:
  * Starter - This is prebuilt binaries 
  * Developer - Gives the ability to build a STLinux based Kernel, Opp-Tee applications and u-boot binary
  * Distribution - Yocto based build, full RFS as well as low level embedded portions

### Starter Images

Initially, it's important to understand flashing new images over anything else. In that spirt, we start with the Starter Package and the 
pregenerated images[6] that ST has to offer. Once the starter images tarball are downloaded and extracted we see the following file structure:

    stm32mp1-openstlinux-5-4-dunfell-mp1-20-11-12
    └──images
       └──stm32mp1
          ├── arm-trusted-firmware //binaries for FSBL partitions and serial boot mode
          |   ├── *.optee.stm32      // optee boot chain
          |   ├── *.serialboot.stm32 // STM32CubeProgrammer
          |   └── *.trusted.stm32    // trusted boot chain (default)
          ├── bootloader  // U-Boot binaries for FSBL, SSBL partitions and supported boot chains
          ├── flashlayout_st-image-weston // Flash layout files (text description of the partitions) for the supported boot chains on supported boot devices and boards
          ├── optee // Optee binaries for TEE partitions required for optee boot chains
          ├── scripts // helper scripts
          └── other files...  // binaries for other partitions on eMMC, microSD card, and NAND devices

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

1. https://www.st.com/resource/en/user_manual/dm00591354-discovery-kits-with-stm32mp157-mpus-stmicroelectronics.pdf
2. https://my.st.com/content/my_st_com/en/products/development-tools/software-development-tools/stm32-software-development-tools/stm32-utilities/stsw-link009.license=1610949972923.product=STSW-LINK009.version=2.0.1.html
3. https://wiki.st.com/stm32mpu/wiki/STM32MP15_Discovery_kits_-_Starter_Package
4. https://wiki.st.com/stm32mpu/wiki/How_to_get_Terminal
5. https://wiki.st.com/stm32mpu/wiki/STM32MP15_Discovery_kits_-_Starter_Package#Downloading_the_image_and_flashing_it_on_the_board
6. https://st.com/content/ccc/resource/technical/software/firmware/group0/e2/88/93/37/bc/48/41/cb/STM32MP15_OpenSTLinux_Starter_Package/files/FLASH-stm32mp1-openstlinux-5-4-dunfell-mp1-20-11-12.tar.xz/jcr:content/translations/en.FLASH-stm32mp1-openstlinux-5-4-dunfell-mp1-20-11-12.tar.xz
7. https://wiki.st.com/stm32mpu/wiki/STM32MP15_Discovery_kits_-_Starter_Package#Installing_the_tools
8. https://wiki.st.com/stm32mpu/wiki/STM32MP1_Developer_Package
9. https://wiki.st.com/stm32mpu/wiki/STM32MP1_Developer_Package#Installing_the_Linux_kernel
