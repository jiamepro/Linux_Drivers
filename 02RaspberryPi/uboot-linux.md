# 树莓派移植u-boot和linux
* 既然离真实板卡的Linux移植只差临门一脚了，不如直接上手，这篇讲一下树莓派4B移植Linux系统，为什么是树莓派？因为手上刚好有这块板子，实验室传下来的，方便。更现实的因素，树莓派比其他的ARM开发板要便宜多了
* 主要参考(https://blog.csdn.net/weixin_43328157/article/details/130241572)
## 板卡信息
* 树莓派4B的 bcm2835(ARM-cotexA53架构,使用ARMv8指令集)
 (https://blog.csdn.net/u011303443/article/details/53522913)
 (https://blog.csdn.net/SkyGloaming/article/details/120659138)
 (https://blog.csdn.net/m0_51430584/article/details/130186451)
x86-64、amd64、arm、aarch64(https://blog.csdn.net/fuhanghang/article/details/129183693)
* 不同的架构使用的交叉编译器不同，（https://blog.csdn.net/gxy199902/article/details/127162898），64位的arm架构，所以交叉编译器选择aarch64-linux-gnueabi-
* 安装交叉编译器 sudo apt-get install gcc-aarch64-linux-gnu
交叉编译器区别(https://blog.csdn.net/Namcodream521/article/details/88379307)
system：交叉编译链所选择的库函数和目标映像的规范，如gnu，gnueabi等。其中gnu等价于glibc+oabi；gnueabi等价于glibc+eabi（https://blog.csdn.net/gybwq/article/details/115842756）。若不指定，则也可以留空不填；
1、ABI与EABI
ABI：二进制应用程序接口（Application Binary Interface）。在计算机中，应用二进制接口描述了应用程序（或者其他类型）和操作系统之间或其他应用程序的低级接口；
EABI：即嵌入式ABI，应用于嵌入式系统的二进制应用程序接口（Embeded Application Binary Interface）。EABI指定了文件格式、数据类型、寄存器使用、堆积组织优化和在一个嵌入式软件中的参数的标准约定。开发者使用自己的汇编语言也可以使用 EABI 作为与兼容的编译器生成的汇编语言的接口。
两者主要区别是，ABI是计算机上的，EABI是嵌入式平台上（如ARM，MIPS等）。
什么是OABI 和 EABI(https://blog.csdn.net/GMstart/article/details/6615864)
## U-boot
* 下载地址（http://www.denx.de/wiki/U-Boot/)
* 命令：git clone https://source.denx.de/u-boot/u-boot.git
下面的操作应该很熟悉了
* 编译U-Boot
```
export CROSS_COMPILE=aarch64-linux-gnu-（目的是指向编译器）
make distclean （清除之前编译源码的缓存，首次执行会很快，因为没有缓存）
make rpi_4_defconfig （选择树莓派4的默认配置）
make -j8 （编译）
```

* 树莓派的启动过程是这样的，设备上电,GPU启动并从启动分区读取config.txt，该文件包含 GPU 用于进一步调整启动顺序的一些配置参数。然后将kernel8.img被加载到内存中并执行。
为了能够运行我们的简单操作系统，该config.txt文件应如下所示：（https://blog.csdn.net/coolwriter/article/details/77719003）
https://www.raspberrypi.org/documentation/configuration/config-txt.md

https://blog.csdn.net/weixin_43702920/article/details/134707344

改一下树莓派默认的config.txt,改了kernel的加载！！!要在boot分区下！！！
kernel=u-boot.bin，u-boot.bin文件cp到能正常启动树莓派系统的TF卡的boot分区中

* 通过串口进入控制台：minicom -D /dev/ttyUSBx
https://blog.csdn.net/lu_embedded/article/details/106309451
sudo apt-get install minicom

可能有误解，先用树莓派镜像烧录器烧入树莓系统，再改这个系统里的config.txt？？？？分区自动分好了  ————————对的

或者要先搞好分区？？？

串口没输出？？ 串口有输出，但无法输入，通常是由于开启了硬件流控导致的。 硬件流硬件流硬件流
https://blog.csdn.net/weixin_43328157/article/details/130241572

https://blog.csdn.net/lu_embedded/article/details/106309451

https://blog.csdn.net/yuhangfeng/article/details/124888832

https://blog.csdn.net/weixin_43328157/article/details/130218995

https://blog.csdn.net/weixin_44278698/article/details/124297000

111

感觉树莓派的uboot移植不涉及uboot的运行的根本原理，这样不好，作为一个小结讲就行


# 详解U-Boot移植：链接、重定位、自我复制
## U-boot启动过程
### 
###
## qemu vexpress-A9 U-boot移植
## （真实板卡）树莓派的U-boot移植