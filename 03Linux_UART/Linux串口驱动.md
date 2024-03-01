# Linux设备的串口驱动
使用ARM Versatile/926EJ-S板
## 内存映射
用的CPU芯片是ARM926EJ-S，整个板子叫ARM Versatile，连接了外设的板子才有内存映射！！！
Versatile 应用底板手册

还是搞树莓派4b吧 64位的, 资料好找，还有硬件

[树莓派文档](https://datasheets.raspberrypi.com/?_gl=1*1m98280*_ga*MTgzNDQzMzY2LjE2OTc4MDUxNTM.*_ga_22FD70LWDS*MTcwOTE5NTg3Ni40LjAuMTcwOTE5NTg3Ni4wLjAuMA..)
bcm2835/bcm2835-peripherals.pdf

* 下载U-boot
命令：git clone https://source.denx.de/u-boot/u-boot.git

* 编译U-Boot
```
export CROSS_COMPILE=aarch64-linux-gnu-
make distclean 
make rpi_4_defconfig
make -j8 
```
* xxx_defconfig 就是不同板子的配置文件，这些配置文件都在 uboot/configs 目录中。

* 可以看出，编译完成以后 uboot 源码多了一些文件，其中 u-boot.bin 就是编译出来的 uboot二进制文件。uboot 是个裸机程序，因此需要在其前面加上头部(IVT、DCD 等数据)才能在 I.MX6U上执行，图 3ls
0.2.4 中的 u-boot.imx 文件就是添加头部以后的 u-boot.bin，u-boot.imx 就是我们最终要烧写到开发板中的 uboot 镜像文件。

* [qume_rpi](https://www.qemu.org/docs/master/system/arm/raspi.html)

QEMU中实施的设备
ARM1176JZF-S, Cortex-A7, Cortex-A53 or Cortex-A72 CPU
Interrupt controller
DMA controller
Clock and reset controller (CPRMAN)
System Timer
GPIO controller
Serial ports (BCM2835 AUX - 16550 based - and PL011)
Random Number Generator (RNG)
Frame Buffer
USB host (USBH)
GPIO controller
SD/MMC host controller
SoC thermal sensor
USB2 host controller (DWC2 and MPHI)
MailBox controller (MBOX)
VideoCore firmware (property)
Peripheral SPI controller (SPI)
[树莓派4b]https://github.com/dhruvvyas90/qemu-rpi-kernel
raspi4b

2024年2月29日 星期四 ： 最高支撑3b
资料： https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#raspberry-pi-3-model-b-2
Cortex-A72 (4 cores), 2 GiB of RAM
	```bash 
	sudo qemu-system-aarch64 -M raspi3b -m 1G -kernel ./u-boot.bin  -nographic -serial null -serial mon:stdio
	```
网卡驱动没问题
Net:   eth0: ethernet@7d580000 

树莓派默认账号：pi
树莓派默认密码：raspberry
不太行！！！使用硬件问题多，实践问题多，先多看






