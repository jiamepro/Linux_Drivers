@[toc]
# 安装Linux系统和QEMU
> 工欲善其事必先利其器，搭建一个良好的开发环境是进行开发的基础，可以少出很多bug

1. 需要准备一个**Linux系统**的环境，可以用虚拟机搭建，网上有很多文章讲这个，就不赘述了。正点原子提供了开源的虚拟机和系统镜像的资料和搭建教程，完全开源的，请自取：[正点原子资料](http://www.openedv.com/docs/boards/arm-linux/zdyz-i.mx6ull.html)
2.  本文使用笔记本电脑装的Linux双系统作为开发环境，读者如果要安装双系统，可以查找自己电脑型号对应的双系统安装教程，也比较繁杂，但是之后就是在真正的Linux里搞开发，不存在虚拟机里莫名其妙的问题。
> 在实践过程中发现用虚拟机会出现挺多问题，比如虚拟机网络问题导致不能安装一些库、虚拟机导致电脑蓝屏、虚拟机卡慢等等问题

平台信息：
```
Ubuntu版本：22.04
gcc版本： 11.4.0
主机linux内核版本： 6.5.0-15-generic
所构建的Linux内核版本：6.6.16
```
Linux系统安装好了之后**安装qemu**，在命令行输入：
```bash
sudo apt-get install qemu
sudo apt-get install qemu-system
```
# 编译Linux内核
首先编译Linux内核，我们构建Linux内核需要编译后产生的内核镜像zImage和对应的设备树文件
* 内核下载地址如下，官方网站可能下载比较慢，下载6.6.16版本的内核：
[内核下载官方网站](https://mirrors.edge.kernel.org/pub/linux/kernel/)
[内核下载镜像网站](https://cdn.kernel.org/pub/linux/kernel/)
* 下载后解压，进入cd进入内核文件夹linux-6.6.16
* 配置内核，输入一下命令：
	```bash
	export ARCH=arm
	export CROSS_COMPILE=arm-linux-gnueabi-
	make vexpress_defconfig
	make menuconfig
	```
	弹出的界面不设置，按两次esc退出，

	>1. Uboot和Linux内核移植需要根据你的板卡选择相应的默认配置信息`/boot/configs`，我们选择Vexpress-A9开发板的配置信息。
	> 2. export命令：用于显示或设置环境变量，但是效果仅限于当前登录的终端，这样我们不用每次make都指定参数了
	> 3. vexpress_defconfig：是ARM Versatile Express开发板的内核默认配置文件，文件夹里有很多厂家板子的默认配置文件，放在`arch/arm/configs`

* 接下来，我们编译内核镜像文件和设备树文件，时间有一点久，耐心等待：
	```bash
	make bzImage
	make dtbs
	```
	> 编译结果注意zImage的地址和设备树的地址，后续会用到
	```
	make bzImage 的结果如下：
	0BJCopy arch/arm/boot/zImage
	Kernel: arch/arm/boot/zImage is ready
	make dtbs 的结果如下：
	DTC		arch/arm/boot/dts/arm/vexpress-v2p-ca5s.dtb
	DTC		arch/arm/boot/dts/arm/vexpress-v2p-ca9.dtb
	DTC		arch/arm/boot/dts/arm/vexpress-v2p-ca15-tc1.dtb
	DTC		arch/arm/boot/dts/arm/vexpress-v2p-ca15 a7.dtb
	```
	
	> 可以看到，Linux内核镜像文件zInage保存在`arch/arm/boot/zImage`，设备树文件：vexpress-v2p-ca9.dtb(在`arch/arm/boot/dts/`文件夹下)

* 在内核文件夹里，运行qemu：
	```bash
	qemu-system-arm -M vexpress-a9 -smp 4 -m 256M -kernel arch/arm/boot/zImage -append "console=ttyAMA0" -dtb arch/arm/boot/dts/arm/vexpress-v2p-ca9.dtb -nographic
	```
	输出信息出现`Booting Linux on physical CPU 0x0`，说明成功进入内核了，我们编译的内核和设备树没问题，但是我们没有构建和指定根文件系统，所以内核会崩溃显示`---[ end Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0) ]---`
*  QEMU中，输入ctrl+a再输入x，退出QEMU，准备构建根文件系统
# 构建根文件系统（BusyBox）
> 使用BusyBox来构建根文件系统，根文件系统的根目录和子目录中会有很多Linux 运行所必须的文件，比如库、常用的软件和命令、设备文件、配置文件等等，BusyBox 就集成了这些大量的 Linux 命令和工具
* 博主下载的版本是1.36.1，下载网站：[busybox下载](https://busybox.net/)
* 解压，cd进入busybox，对busybox进行配置，使用默认配置defconfig
	```bash
	make defconfig
	make menuconfig
	```
	跳出的界面里配置静态编译：Setting->Build static binary，点空格把它选上，然后Esc退出
* 编译根文件系统：
	```bash
	mkdir /home/rootfs
	make
	make install CONFIG_PREFIX=/home/rootfs 
	```
	> 设置CONFIG_PREFIX参数，将编译结果指定存放在根文件系统目录下
* 接下来需要完善根文件系统
	* 创建etc,dev,mnt,sys,tmp,proc,root,lib等目录，安装动态链接库
		```bash
		mkdir etc,dev,mnt,sys,tmp,proc,root
		sudo cp -d /usr/arm-linux-gnueabi/lib/*.so* ./lib
		```
	* 创建设备节点
		```bash
		cd dev/
		sudo mknod -m 666 console c 5 1
		sudo mknod -m 666 null c 1 3
		sudo mknod -m 666 dev/tty1 c 4 1
		sudo mknod -m 666 dev/tty2 c 4 2
		sudo mknod -m 666 dev/tty3 c 4 3
		sudo mknod -m 666 dev/tty4 c 4 4
		```
	* 创建并写入三个配置文件`/etc/init.d/rcS`，`/etc/fstab`，`/etc/inittab`，其内容分别为：
		>1.  /etc/init.d/rcS:  shell 脚本, 规定Linux 内核启动以后启动哪些文件的脚本，该文件一定要给与可执行权限！！！
		> 2. /etc/fstab：Linux 开机以后自动配置哪些需要自动挂载的分区
		> 3. /etc/inittab：内核初始化的init 程序会读取/etc/inittab这个文件，inittab 由若干条指令组成
		三个文件的写法以及环境变量配置参考这篇文章：[WSL2下Ubuntu22.04使用Qemu搭建虚拟Vexpress-A9开发板（三）——挂载根文件系统](https://blog.csdn.net/cotex_A9/article/details/132354963)
	
		/etc/init.d/rcS：
		```bash
		#!/bin/sh
		PATH=/bin:/sbin:/usr/bin:/usr/sbin
		export LD_LIBRARY_PATH=/lib:/usr/lib
		/bin/mount -n -t ramfs ramfs /var
		/bin/mount -n -t ramfs ramfs /tmp
		/bin/mount -n -t sysfs none /sys
		/bin/mount -n -t ramfs none /dev
		/bin/mkdir /var/tmp
		/bin/mkdir /var/modules
		/bin/mkdir /var/run
		/bin/mkdir /var/log
		/bin/mkdir -p /dev/pts
		/bin/mkdir -p /dev/shm
		/sbin/mdev -s
		/bin/mount -a
		echo "-----------------------------------"
		echo "*****welcome to vexpress board*****"
		echo "-----------------------------------"
		```
		/etc/fstab：
		```c
		proc    /proc           proc    defaults        0       0
		none    /dev/pts        devpts  mode=0622       0       0
		mdev    /dev            ramfs   defaults        0       0
		sysfs   /sys            sysfs   defaults        0       0
		tmpfs   /dev/shm        tmpfs   defaults        0       0
		tmpfs   /dev            tmpfs   defaults        0       0
		tmpfs   /mnt            tmpfs   defaults        0       0
		var     /dev            tmpfs   defaults        0       0
		ramfs   /dev            ramfs   defaults        0       0
		```
		/etc/inittab:
		```c
		::sysinit:/etc/init.d/rcS 
		::askfirst:-/bin/sh
		::restart:/sbin/init 
		::ctrlaltdel:/sbin/reboot
		::shutdown:/bin/umount -a -r
		```
	> 写了shell脚本自动完成本节完善根文件系统的工作，自取，切换到`/home/rootfs`目录下执行：[完善根文件系统的shell脚本](https://github.com/jiamepro/Divece_drivers/blob/main/code/01/init_rootfs)

# 创建根文件系统镜像，SD挂载
* 在根文件系统目录下执行下列命令，创建根文件系统镜像
	```bash
	cd /home
	sudo qemu-img create -f raw linux.img 256M
	sudo mkfs -t ext4 ./linux.img
	
	sudo mkdir tmpfs
	sudo mount -o loop ./linux.img tmpfs/
	sudo cp -r rootfs/* tmpfs/
	sudo umount tmpfs 
	sudo rm -r tmpfs
	file linux.img
	```
	根文件系统镜像创建成功显示：`linux.img: Linux rev 1.0 ext4 filesystem data, UUID=c0f6a2f3-b70e-480f-8726-3c2d66ce9120 (extents) (64bit) (large files) (huge files)`
* 把编译出的内核镜像zImage和设备树文件拷贝到`/home`，运行QEMU：
	```bash 
	sudo qemu-system-arm \
			-M vexpress-a9 \
	        -m 256M \
	        -kernel ./zImage \
	        -dtb ./vexpress-v2p-ca9.dtb \
	        -nographic \
	        -append "root=/dev/mmcblk0 rw console=ttyAMA0" \
	        -sd linux.img
	```
	> 注意：-append选项的参数，我们从sd卡启动，root所指定的位置是`/dev/mmcblk0` 就是sd卡设备挂载的目录, `rw` 表示根文件系统是可以读写的，console 用来设置 linux 终端，表示通过什么设备来和 Linux 交互，设置` console = ttyAMA0`，因为 linux启动以后板卡的串口 1 挂载在 linux 下的设备文件就是`/dev/ttyAMA0`

* 构建内核成功！可以在QEMU模拟的arm开发板中输入linux指令了
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/8ee4996ab3f049dd831a9b142d70ad94.png)
# 总结
1. 编译内核的时候要注意选择对应板卡的内核配置信息
2. Linux内核的构建需要三个文件，编译Linux内核得到的内核镜像文件zImage和设备树文件以及根文件系统镜像文件
3. Linux启动还需要设置一些参数,比如root，console等等，qemu用`-append`选项设置
4. 对真实板卡构建Linux内核先要构建uboot，这个实验没有构建uboot也能运行内核是因为使用qemu直接将内核镜像装载在了内存里
5. 直接用qemu运行内核跟利用uboot运行内核的不同之处：
	1. 在uboot中，Linux启动需要配置的参数(root, console......)是通过设置bootargs环境变量配置的，
	2. 利用uboot构建内核的时候涉及nfs或者tftp网络的使用，利用网络将zImage和设备树文件传到uboot，然后通过网络挂载根文件系统。
6. uboot构建过程跟Linux内核的构建差不多


