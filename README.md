# BUILD A MINIMAL LINUX OS FOR RASPBERRY PI 4

  In this tutorial, we are going to build a simple linux operating system for Raspberry Pi. I used Raspberry Pi 4 and this tutorial explains the procedure to build 64 bit linux system. If you are using any other Raspberry Pi board, still you can follow the procedure with minor changes here and there.

### REQUIREMENTS:

  1. Raspberry Pi 4 (Preferred) or Any other Raspberry Pi Board
  2. Micro SD Card
  3. USB to TTL Convertor
  4. Ubuntu Host System
  5. Patience 😉



### PREPARE SD CARD

  1. Use Gparted tool
  2. Create 2 Partitions
  3. First Partition - FAT32 Format, At least 256MB (boot partition)
  4. Second Partition - EXT4 Format, Remaining SD Card Space (rootfs partition)



### PROPRIETARY RASPBERRY PI FILES 

  1. Download start4.elf, fixup4.dat from the below link
  
    https://github.com/raspberrypi/firmware/tree/master/boot
  2. Copy the files to boot partion



### TOOLCHAIN

  1. Install required dependencies
  
    $ sudo apt install git bc bison flex libssl-dev make libc6-dev libncurses5-dev
  2. Download the toolchain
  
    $ wget https://armkeil.blob.core.windows.net/developer/Files/downloads/gnu-a/10.2-2020.11/binrel/gcc-arm-10.2-2020.11-x86_64-aarch64-none-linux-gnu.tar.xz
  3. Set the compiler path. For example, 
  
    $ export PATH=$PATH:/home/rish/Downloads/aarch64-none-linux-gnu/bin



### BOOTLOADER

  1. Download U-Boot source code
  
    $ git clone git://git.denx.de/u-boot.git
    $ cd u-boot
  2. Select the cross compiler
  
    $ export CROSS_COMPILE=aarch64-none-linux-gnu-
  3. Select the configuration
  
    $ make rpi_4_defconfig
  4. Build the u-boot.bin with the following command:
  
    $ make -j5 
  5. You have now created a u-boot.bin binary file in the same directory where you ran the make command
  6. Copy u-boot.bin to boot partition



### KERNEL

  1. Download the Linux source code
  
    $ git clone --depth=1 --branch=1_33_1 https://github.com/raspberrypi/linux
  
    $ cd linux
  2. Build the source code
  
    $export ARCH=arm64
    $export CROSS_COMPILE=aarch64-none-linux-gnu-
    $make bcm2711_defconfig
    $make Image dtbs modules
  3. Copy arch/arm/boot/Image to boot partition. Rename it to kernel8.img
  4. Copy arch/arm/boot/dts/bcm2711-rpi-4-b.dtb to boot partition
  5. Install modules to rootfs partition

    $sudo env PATH=$PATH make ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu- INSTALL_MOD_PATH=/mnt/rootfs modules_install
    
    (Replace /mnt/rootfs with the path where you have mounted rootfs partition)



### ROOT FILE SYSTEM 

  1. Download Busybox source code

    $git clone git://busybox.net/busybox.git
    $cd busybox
  2. We need to change some settings
  
    a. Settings --> Build static binary (no shared libraries) : Enable
   
    b. Settings --> Cross compiler prefix : aarch64-none-linux-gnu-
    
    c. Settings --> Destination path for 'make install' : <path where you have mounted rootfs partition>
    
  3. Build the source code and Install
  
    $ make -j5
    $ sudo env PATH=$PATH make install
  4. Now, you should see the directories bin, sbin, usr in the rootfs partition
  5. Still we need to add few directories in rootfs partition
  
    $ mkdir dev
    $ mkdir sys
    $ mkdir pro
    $ mkdir etc
    $ mkdir etc/init.dat
  6. Create an empty file and make it executable
  
    $ touch etc/init.d/rcS
    $ chmod +x etc/init.d/rcS
  7. Add the following lines and save the file:
  
    #!/bin/sh
    mount -t sysfs none /sys
    mount -t proc none /proc
    echo /sbin/mdev > /proc/sys/kernel/hotplug
    mdev -s
	
	
  
### BOOTING THE BOARD
  
  1. Now, we need a boot script to automate the boot process
  2. Create a file 'boot.cmd' and add the following lines:
  
    mmc dev 0
    fatload mmc 0:1 ${kernel_addr_r} kernel8.img
    fatload mmc 0:1 ${fdt_addr} bcm2711-rpi-4-b.dtb
    setenv bootargs 8250.nr_uarts=1 console=tty1 console=ttyS0,115200 root=/dev/mmcblk0p2 rootfstype=ext4 rootwait panic=5
    booti ${kernel_addr_r} - ${fdt_addr}
  3. Make an executable file
  
    mkimage -A arm64 -T script -C none -d  boot.cmd boot.scr
  4. Copy boot.scr to boot partition
  5. Create a file 'config.txt' in boot partition
  
    $sudo nano config.txt
  6. Add the following lines and save the file:
  
    arm_64bit=1
    enable_uart=1
    kernel=u-boot.bin
  7. Connect USB to TTL converter to UART pins of Raspberry Pi 4 (Pins 8 & 10)
  8. Plug in the USB to Host System
  9. Open a serial terminal with the baud rate of 115200
  10. Insert the SD card into Raspberry Pi and Turn ON the board
  11. You should see the U-boot print scripts, kernel print scripts and a message "Please press Enter to activate this console" 🏆
  
    /# echo "Hello World!"
    Hello World!
