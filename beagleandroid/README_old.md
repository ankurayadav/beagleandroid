This contains steps for creating android image for beaglebone.
========
####We will be using linux kernel 3.8 beacause it has device tree support which can be used for PWM module or ADC without any need to recompile the kernel.
####For this project we are going using Robert Nelson's kernel 3.8 and rowboat project to compile the android.

These details have been taken from [http://icculus.org/~hendersa/android](http://icculus.org/~hendersa/android)

##Step 1
Get rowboat's android project for beaglebone
	<pre><code>
		mkdir RobertNelsonKernel
		cd RobertNelsonKernel
		repo init -u git://gitorious.org/rowboat/manifest.git -m rowboat-jb-am335x.xml 
		repo sync
	</pre></code> 

##Step 2
Get and compile robert nelson kernel
	<pre><code>
		git clone https://github.com/RobertCNelson/bb-kernel.git
		cd linux-dev
		git checkout origin/am33x­v3.8 -b tmp
		./build_kernel.sh
	</pre></code> 
We will require to add some features in kernel's menuconfig

##Step 3
Now get android's bootloader [u-boot](http://www.denx.de/wiki/U-Boot)
	<pre><code>
		git clone git://git.denx.de/u-boot.git 
		cd u-boot/ 
		git checkout v2013.04 -b tmp 
		wget https://raw.github.com/eewiki/u-boot-patches/master/v2013.04/0001-am335x_evm-uEnv.txt-bootz-n-fixes.patch 
		patch -p1 < 0001-am335x_evm-uEnv.txt-bootz-n-fixes.patch
	</pre></code>

##Step 4
Now compile u-boot
	<pre><code>
		make ARCH=arm CROSS_COMPILE=~/RobertNelsonKernel/linux-dev/dl/gcc-linaro-arm-linux-gnueabihf-4.7­2013.04-­20130415_linux/bin/arm-linux-gnueabihf- distclean 
		make ARCH=arm CROSS_COMPILE=~/RobertNelsonKernel/linux-dev/dl/gcc-linaro-arm-linux-gnueabihf-4.7­2013.04-­20130415_linux/bin/arm-linux-gnueabihf- am335x_evm_config
		make ARCH=arm CROSS_COMPILE=~/RobertNelsonKernel/linux-dev/dl/gcc-linaro-arm-linux-gnueabihf-4.7­2013.04-­20130415_linux/bin/arm-linux-gnueabihf-
	</pre></code>

##Step 5
Compile android filesystem using kernel 3.8, for that change kernel link to rowboat project.
	<pre><code>
		cd rowboat-­android 
		mv kernel kernel.backup 
		ln -s ~/RobertNelsonKernel/linux-dev/KERNEL kernel 
	</pre></code>

Change permission of makefile using chmod
	<pre><code>
		chmod 644 Makefile
	</pre></code>

locate following line 
	<pre><code>
		export PATH :=$(PATH):$(ANDROID_INSTALL_DIR)/prebuilts/gcc/linux-x86/arm/arm-eabi-4.6/bin
	</pre></code>
comment this line and replace it with
	<pre><code>
		export PATH :=~/RobertNelsonKernel/linux-dev/dl/gcc-linaro-arm-linux-gnueabihf-4.7-2013.04-20130415_linux/bin:$(PATH)
		export CC_PREFIX := arm-linux-gnueabihf-
	</pre></code>

now locate this line
	<pre><code>
		$(MAKE) -C hardware/ti/wlan/mac80211/compat_wl18xx ANDROID_ROOT_DIR=$(ANDROID_INSTALL_DIR) CROSS_COMPILE=arm-eabi- ARCH=arm install
	</pre></code>
replace this with
	<pre><code>
		$(MAKE) -C hardware/ti/wlan/mac80211/compat_wl18xx ANDROID_ROOT_DIR=$(ANDROID_INSTALL_DIR) CROSS_COMPILE=$(CC_PREFIX) ARCH=arm install
	</pre></code>

now compile android's filesystem just add '-j2' or '-j4' if you have many cores to accelarate compilation process.
	<pre><code>
		make TARGET_PRODUCT=beagleboneblack OMAPES=4.x droid
	</pre></code>

##Step 6
now create root filesystem in rowboat-android directory
	<pre><code>
		cd out/target/product/beagleboneblack 
		mkdir android_rootfs 
		cp -r root/* android_rootfs 
		cp -r system android_rootfs 
		cd android_rootfs/system 
		tar -xvzf ~/RobertNelsonKernel/linux-dev/deploy/*modules.tar.gz 
	</pre></code>
Assuming that we are still in the out/target/product/beagleboneblack directory, we need to
backup the file android_rootfs/init.rc and then open it in a text editor, we are going to to
change init.rc file:

In line:
	<pre><code>
		import /init.${ro.hardware}.rc
	</pre></code>
change it to:
	<pre><code>
		import /init.am335xevm.rc
	</pre></code>

change the file android_rootfs/**fstab.am335xevm**, the first line is
	<pre><code>
		/dev/block/platform/omap/omap_hsmmc.0/mmcblk0p3
	</pre></code>
change it by
	<pre><code>
		/dev/block/mmcblk0p3
	</pre></code>

now we have to disable graphics card accelarator in file android_rootfs/system/**build.prop** beacause it is not available in kernel 3.8
	<pre><code>
		debug.egl.hw=0
		video.accelerate.hw=0
	</pre></code>

finally, in the out/target/product/beagleboneblack directory, execute
	<pre><code>
		../../../../build/tools/mktarball.sh ../../../host/linux­x86/bin/fs_get_stats android_rootfs . rootfs rootfs.tar.bz2
	</pre></code>
rootfs.tar.bz2 file will be created. This is the Android filesystem we will use.

##Step 7
create a image directory and gather all required files
	<pre><code>
		cd ~/RobertNelsonKernel
		mkdir image
		cp rowboat-android/external/ti_android_utilities/am335x/mk-mmc/* image 
		cp rowboat-android/out/target/product/beagleboneblack/rootfs.tar.bz2 image 
		cp u-boot/MLO image 
		cp u-boot/u-boot.img image 
		cp linux-dev/KERNEL/arch/arm/boot/zImage image 
		cp linux-dev/KERNEL/arch/arm/boot/dts/am335x-5boneblack.dtb image 
	</pre></code>

##Step 8
create u-boot configuration file uEnv.txt in image directory with following content
	<pre><code>
		kernel_file=zImage
		console=ttyO0,115200n8
		mmcroot=/dev/mmcblk0p2 rw
		mmcrootfstype=ext4 rootwait
		loadkernel=load mmc ${mmcdev}:${mmcpart} 0x80200000 ${kernel_file}
		loadfdt=load mmc ${mmcdev}:${mmcpart} 0x815f0000 ${fdtfile}
		boot_ftd=run loadkernel; run loadfdt
		mmcargs=setenv bootargs consoleblank=0 console=${console}
		androidboot.console=ttyO0 mem=512M root=${mmcroot} rootfstype=${mmcrootfstype}
		init=/init ip=off video=720x480-16@60 qemu=1 vt.global_cursor_default=0
		uenvcmd=run boot_ftd; run mmcargs; bootz 0x80200000 - 0x815f0000
	</pre></code>

##Step 9
now create uSD card. run the script mkmmc-android.sh
	<pre><code>
		cd image 
		./mkmmc­android.sh [YOUR DEVICE FILE FOR THE MICROSD CARD] MLO u-boot.img zImageuEnv.txt rootfs.tar.bz2 
	</pre></code>

##Step 10
Now mount "boot" partition of our sd card do following changes
	<pre><code>
		cd /media/boot 
		sudo mv uImage zImage 
		sudo cp ~/RobertNelsonKernel/image/am335x-boneblack.dtb . 
		sync
	</pre></code>
We added device tree file and we changed uImage for zImage.
Now our sd is ready to run Android with kernel 3.8 and device tree support.

**NOTE**: adb does not work by USB, we need to connect Beaglebone Black using Ethernet
connector.