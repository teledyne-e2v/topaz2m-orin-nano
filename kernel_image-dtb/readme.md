For JetPack 5.1.2 on Nvidia Jetson Orin Nano Dev kit

On the Jetson - copy thes files to /tmp

Move the kernel to /boot:

	sudo mv /tmp/Image /boot

and the device tree:

	sudo mv /tmp/tegra234-p3767-0003-p3768-0000-a0.dtb /boot
	
Set the board to load your device tree by modifying the file

	sudo gedit /boot/extlinux/extlinux.conf

Note that if the files is not opening, stop the gedit command and try again

Modify the extlinux.conf file, including FDT /boot/tegra234-p3767-0003-p3768-0000-a0.dtb under LABEL primary like this:

	LABEL primary
	MENU LABEL primary kernel
	LINUX /boot/Image
	FDT /boot/tegra234-p3767-0003-p3768-0000-a0.dtb
	INITRD /boot/initrd
	APPEND ${cbootargs} quiet root=/dev/mmcblk0p1 rw rootwait rootfstype=ext4
	console=ttyS0,115200n8 console=tty0 fbcon=map:0 net.ifnames=0
	
Reboot the Jetson - the driver should be loaded, check the log:

	sudo dmesg | grep topaz
	
The device should be created:

	ls /dev/video*
	
Test the video stream with:

	v4l2-ctl -d /dev/video0 --set-fmt-video=width=1920,height=1080,pixelformat='GREY' --set-ctrl sensor_mode=2
	gst-launch-1.0 v4l2src ! "video/x-raw,width=1920,height=1080,format=GRAY8" ! queue ! videoconvert ! queue ! xvimagesink sync=false


	
	


