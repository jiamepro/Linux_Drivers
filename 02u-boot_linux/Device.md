# 【Linux驱动开发（二）】编写SD驱动，模仿真实板卡移植linux
[toc]
# U-boot启动过程
* 裸机平台，比如STM32，我们可以将程序用烧录工具，如ST-Link和J-Link等设备，通过SWD和JTAG等传输协议配合Flash中固化的引导程序，将代码烧录到单片机的Flash里的特定位置里，点复位按键，裸机就开始执行代码了
* 需要安装操作系统的ARM平台，启动后，首先运行固件中板载的引导程序，ARMsoc系统上电时，先通过板载引导程序将SD卡中的BootLoader代码（比如U-boot）复制到CPU芯片内部的SRAM中的特定位置，并跳转该处（移交控制权给U-boot），再有U-boot执行将操作系统镜像加载进内存里的操作
* 但是在且qemu环境下，我们是直接将内核镜像装载到了内存里，直接运行的，这不能很好模拟真实情况，所以我们先写一个BootLoader来模仿固化在CPU中的引导代码，这段代码将SD卡中的U-Boot加载到内存里，并授予控制权
[模仿固件的BootLoader代码](./Bootloader.md)


# Vexpress-A9开发板硬件介绍
* 内存映射？不用Vexpress-A9开发板，用书上的板子
# 驱动SD卡和复制U-boot
* 介绍SD卡，如何驱动SD卡？
  * 了解
* SD卡分区？
  * 了解
* SD卡驱动编写？
  * 了解
* 复制SD卡/boot分区下的u-boot.bin
  * 实操
* 这里的u-boot.bin指的是不包含SPL的stage2部分的代码. 它会被SPL搬移到RAM的某个地址处开始运行. 
# 模仿真实vexpress-A9板卡移植Linux系统
* SD卡分区 /boot /kernel
* SD卡中放入U-boot和linux系统相关文件
* 方法一：到SD中装载linux
* 开机BootLoader代码自动装载U-boot
* 进入U-boot，指定bootarg，装载linux系统
* 方法二：通过NFS网络下载，再装载：在u-boot中配置网络？

qemu加载uboot.bin，实操一下NFS的工作！

讨论：为什么需要移植？？不同板卡哪里会不同，需要改哪些文件才能适配？？（正点原子）《microC/OS》P14
* CPU框架需要修改的内容？？？ CPU知识
  * 了解

# 参考文章
[单片机程序烧录](https://blog.csdn.net/ZHOU_YONG915/article/details/122842967)

接下来写UART驱动、LCD驱动、？计时器、键盘驱动