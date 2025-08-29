# topaz2m-orin-nano
Topaz2M camera driver for Nvidia Orin Nano Developper Kit


# Build from scratch
The following applications are needed:

    sudo apt install -y flex quilt bison

On the host machine Install the `Jetpack 5.1.2` using Nvidia SDK Manager (from Nvidia website):
    
    sdkmanager --archived-versions

 Select the correct SDK version and the following Target Hardware:

    Jetson Orin Nano [8GB developer kit version]
    P3767-0005 module
    P3768-0000 carrier board

Please install it in the following folder : `~/JetPack-5.1.2`

Download sources:

	cd ~/JetPack-5.1.2/JetPack_5.1.2_Linux_JETSON_ORIN_NANO_TARGETS/Linux_for_Tegra/
	sudo wget https://developer.nvidia.com/downloads/embedded/l4t/r35_release_v4.1/sources/public_sources.tbz2/ -O public_sources.tbz2

Extract sources:

	sudo tar -xjf public_sources.tbz2 Linux_for_Tegra/source/public/kernel_src.tbz2 --strip-components 3

Copy sources:

	mkdir sources
	tar -xjf kernel_src.tbz2 -C ./sources/

Download the toolchain:

	cd ~/JetPack-5.1.2
	mkdir toolchain
	cd toolchain
	wget -O toolchain.tar.xz https://developer.nvidia.com/embedded/jetson-linux/bootlin-toolchain-gcc-93
	
Extract the toolchain:

	sudo tar -xf toolchain.tar.xz

Open SSL library is also mandatory:

	sudo apt-get install libssl-dev

## Patch the source codes

Go to the sources directory:

	cd ~/JetPack-5.1.2/JetPack_5.1.2_Linux_JETSON_ORIN_NANO_TARGETS/Linux_for_Tegra/sources/
	
Undo the last driver patch if needed:

	quilt pop -a
	rm -rf patches/

Copy the patch from this project:

    cp -r ~/topaz2m-orin-nano/patches .

Verify the existence of the patch files:

	echo -n "Patches directory check: ";if test -e "patches"; then echo "PASS"; else echo "FAIL"; fi;
	echo -n "Series file check: ";if test -f "patches/series"; then echo "PASS"; else echo "FAIL"; fi;
	while read p; do echo -n "$p file check: "; if test -f "patches/$p"; then echo "PASS"; else echo"FAIL";fi ; done < patches/series

Apply the patch:

	quilt push -a

Here is the content:

	Applying patch patches/5.1.2_nano_teledyne_topaz2m_v0.1.0.patch
	patching file hardware/nvidia/platform/t23x/p3768/kernel-dts/cvb/tegra234-camera-topaz2m.dtsi
	patching file hardware/nvidia/platform/t23x/p3768/kernel-dts/cvb/tegra234-p3509-a02.dtsi
	patching file hardware/nvidia/platform/t23x/p3768/kernel-dts/cvb/tegra234-p3768-0000-a0.dtsi
	patching file kernel/kernel-5.10/arch/arm64/configs/defconfig
	patching file kernel/nvidia/drivers/media/i2c/Kconfig
	patching file kernel/nvidia/drivers/media/i2c/Makefile
	patching file kernel/nvidia/drivers/media/i2c/topaz2m.c
	patching file kernel/nvidia/drivers/media/i2c/topaz2m_mode_tbls.h
	patching file kernel/nvidia/drivers/media/platform/tegra/camera/camera_common.c
	patching file kernel/nvidia/drivers/media/platform/tegra/camera/sensor_common.c
	patching file kernel/nvidia/drivers/media/platform/tegra/camera/tegracam_core.c
	patching file kernel/nvidia/drivers/media/platform/tegra/camera/tegracam_ctrls.c
	patching file kernel/nvidia/drivers/media/platform/tegra/camera/tegracam_v4l2.c
	patching file kernel/nvidia/drivers/media/platform/tegra/camera/vi/vi2_formats.h
	patching file kernel/nvidia/drivers/media/platform/tegra/camera/vi/vi4_formats.h
	patching file kernel/nvidia/drivers/media/platform/tegra/camera/vi/vi5_formats.h
	patching file kernel/nvidia/include/media/camera_common.h
	patching file kernel/nvidia/include/media/tegra-v4l2-camera.h
	patching file kernel/nvidia/include/media/tegracam_core.h

	Now at patch patches/5.1.2_nano_teledyne_topaz2m_v0.1.0.patch


## Compile

Define the development directory:

	export DEVDIR=~/JetPack-5.1.2/JetPack_5.1.2_Linux_JETSON_ORIN_NANO_TARGETS/Linux_for_Tegra/
	cd $DEVDIR

Create the environment:

	export KERNEL_OUT=$DEVDIR/images
	export KERNEL_MODULES_OUT=$DEVDIR/images/modules
	mkdir $KERNEL_OUT
	mkdir $KERNEL_MODULES_OUT
	export ARCH=arm64
	export KERNEL_DIR=$DEVDIR/sources/kernel/kernel-5.10
	export CROSS_COMPILE_AARCH64=~/JetPack-5.1.2/toolchain/bin/aarch64-buildroot-linux-gnu-
	export LOCALVERSION=-tegra

Configure the kernel:

	cd $KERNEL_DIR
	make ARCH=$ARCH O=$KERNEL_OUT LOCALVERSION=$LOCALVERSION CROSS_COMPILE=$CROSS_COMPILE_AARCH64 tegra_defconfig

Launch configuration menu:

	make ARCH=$ARCH O=$KERNEL_OUT LOCALVERSION=$LOCALVERSION CROSS_COMPILE=$CROSS_COMPILE_AARCH64 menuconfig

Note: By default, the driver is selected with an asterisk. For that reason, if you go back by hitting
the double Esc key the message: Do you want to save your new configuration? will not appear.

	Device Drivers --->
	<*> Multimedia support --->
	Media ancillary drivers --->
	NVIDIA overlay Encoders, decoders, sensors and other helper chips --->
	<*> TOPAZ2M camera sensor support

If the driver is not selected, press the Y key in order to select the TOPAZ2M option. Go back by
hitting the double Esc key until you get the message: Do you want to save your new
configuration?, select Yes and press Enter


Note: To compile the driver as a module, the driver must be selected with an M. If the driver is
not selected, press the M key in order to select the TOPAZ2M option. Go back by hitting the
double Esc key until you get the message: Do you want to save your new configuration?, select
Yes and press Enter'

	Device Drivers --->
	<*> Multimedia support --->
	Media ancillary drivers --->
	NVIDIA overlay Encoders, decoders, sensors and other helper chips --->
	<M> TOPAZ2M camera sensor support
	
To activate IOCTLs to read and write to the sensor EEPROM, in the terminal menu
select:

	Device Drivers --->
	<*> Multimedia support --->
	Video4Linux options --->
	[*] Enable advanced debug functionallity on V4L2 drivers
	
If the option is not selected, press the Y key in order to enable advanced debug functionallity for
the driver, which includes the IOCTLs used to read and write to the EEPROM. Go back by
hitting the double Esc key until you get the message: Do you want to save your new
configuration?, select Yes and press Enter



Compile the kernel:

	make ARCH=$ARCH O=$KERNEL_OUT LOCALVERSION=$LOCALVERSION CROSS_COMPILE=$CROSS_COMPILE_AARCH64 Image

Compile the device tree:

	make ARCH=$ARCH O=$KERNEL_OUT LOCALVERSION=$LOCALVERSION CROSS_COMPILE=$CROSS_COMPILE_AARCH64 dtbs

Compile the device modules:

	make ARCH=$ARCH O=$KERNEL_OUT LOCALVERSION=$LOCALVERSION CROSS_COMPILE=$CROSS_COMPILE_AARCH64 modules

Install modules:

	make ARCH=$ARCH O=$KERNEL_OUT LOCALVERSION=$LOCALVERSION CROSS_COMPILE=$CROSS_COMPILE_AARCH64 modules_install INSTALL_MOD_PATH=$KERNEL_MODULES_OUT

## Flash the board

### Prepare to flash:
Copy the compiled image and device tree to the kernel folder

	cp $KERNEL_OUT/arch/arm64/boot/Image $DEVDIR/kernel/.
	cp $KERNEL_OUT/arch/arm64/boot/dts/nvidia/tegra234-p3767-0003-p3768-0000-a0.dtb $DEVDIR/kernel/dtb/.

In case of compilation as module copy also the .ko file:

	sudo cp $KERNEL_MODULES_OUT/lib/modules/5.10.120-tegra/kernel/drivers/media/i2c/topaz2m.ko $DEVDIR/rootfs/lib/modules/5.10.120-tegra/extra

Run the NVIDIA setup scripts to integrate your changes:

	cd $DEVDIR
	sudo ./apply_binaries.sh
	sudo ./tools/l4t_flash_prerequisites.sh

Define default user/password (optional):

	 sudo ./tools/l4t_create_default_user.sh -u USER_NAME -p USER_PASSWORD

Replacing `USER_NAME` and `USER_PASSWORD` by your data, for example:

 	sudo ./tools/l4t_create_default_user.sh -u teledyne -p teledyne2022

### Put the board in Force Recovery mode:
1. Turned off and disconnected from the power supply the Jetson Orin Nano devkit.
2. Get the USB-A to USB-C or USB-C to USB-C cable and connect the the USB-C end to your Orin Nano and the USB-A or USB-C end to your computer (the computer where you installed Jetpack)
3. Connect a jumper between FC-REC pin and any GND pin as shown in figure 1.
4. Connect the power supply.

![alt text](https://developer.ridgerun.com/wiki/images/9/90/Orin_Nano_devkit_recovery_mode.png)

At this point, the Orin Nano should be in recovery mode. To verify, you can run the following command on your host computer:

	lsusb

If the Orin is in recovery mode, you should see a line similar to the following among the command output:

	Bus 001 Device 011: ID 0955:7523 NVidia Corp


### Execute the Flash Script:
 
	cd $DEVDIR
	sudo ./tools/kernel_flash/l4t_initrd_flash.sh --external-device mmcblk1p1 \
	-c tools/kernel_flash/flash_l4t_external.xml -p "-c bootloader/t186ref/cfg/flash_t234_qspi.xml" \
	--showlogs --network usb0 jetson-orin-nano-devkit internal

### Create a massflash image:

This command will generate a tarball to easily flash several boards or simply save and share a complete image:

	cd $DEVDIR
	sudo ./tools/kernel_flash/l4t_initrd_flash.sh --no-flash --massflash 1 --network usb0 jetson-orin-nano-devkit internal

The image is available in the folder `$DEVDIR` under the name: `mfi_jetson-orin-nano-devkit.tar.gz`. 

### Install a massflash image:

To use massflash, extract the tarball:

	tar xpvf mfi_jetson-orin-nano-devkit.tar.gz
	cd mfi_jetson-orin-nano-devkit

and execute the script:

	sudo ./tools/kernel_flash/l4t_initrd_flash.sh \
  	--flash-only \
  	--massflash 1 \
  	--network usb0 \
	--showlogs

# Only copy kernel image and device tree

If a board has been previously flashed with the same kernel environment, that is possible to update with a different driver version. herre are the instructions.

Copy kernel image:

	scp $KERNEL_OUT/arch/arm64/boot/Image teledyne@192.168.255.59:/tmp
	
Copy device tree:

	scp $KERNEL_OUT/arch/arm64/boot/dts/nvidia/tegra234-p3767-0003-p3768-0000-a0.dtb teledyne@192.168.255.59:/tmp

On the Jetson, move the kernel to `/boot`:

	sudo mv /tmp/Image /boot

and the device tree:

	sudo mv /tmp/tegra234-p3767-0003-p3768-0000-a0.dtb /boot/dtb
	
Set the board to load your device tree by modifying `/boot/extlinux/extlinux.conf`
file, including `FDT /boot/dtb/<dtb>` under `LABEL primary`:

	LABEL primary
	MENU LABEL primary kernel
	LINUX /boot/Image
	FDT /boot/dtb/<dtb>
	INITRD /boot/initrd
	APPEND ${cbootargs} quiet root=/dev/mmcblk0p1 rw rootwait rootfstype=ext4
	console=ttyS0,115200n8 console=tty0 fbcon=map:0 net.ifnames=0

## Copy and run the module

In case of using a driver as a module, here are the instructions to copy and use it. Note that is necessary to have the kernel image and the 
copy compiled module:

	scp $KERNEL_MODULES_OUT/lib/modules/5.10.120-tegra/kernel/drivers/media/i2c/topaz2m.ko teledyne@192.168.255.59:/home/teledyne/.

load the module:

	insmod /home/teledyne/topaz2m.ko

Check the logs:

	sudo dmesg | grep topaz2m

verify the module is well loaded

	lsmod | grep topaz2m

to unload the module:

	sudo rmmod topaz2m

Atlenatively, the file can be moved to ```/lib/modules```

	sudo mv /tmp/topaz2m.ko /lib/modules/5.10.120-tegra/extra/.
	sudo depmod -a

and loaded with: 

	sudo modprobe topaz2m

in this case, you can make the module to be loaded the module automatically at system power-up:

	sudo echo "topaz2m" | sudo tee /etc/modules-load.d/camera.conf

After power, the driver insertion can be checked:

	sudo dmesg | grep topaz2m

Other informations can be printing using the command:

	modinfo topaz2m
