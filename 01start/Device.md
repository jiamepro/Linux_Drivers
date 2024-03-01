# 驱动开发教程
@[toc]

第一次构建内核真的困难重重，终于成功了，将步骤公开，希望小伙伴们少走弯路, Uboot和Linux内核移植的特殊地方就在于需要根据你的板卡选择相应的默认配置信息/boot/configs

参考书上的代码构建根文件系统，失败！！！
---[ end Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0) ]---
怀疑：根文件系统构建有问题
> 2024年2月20日12:04:44：应该是qemu配置指定根文件系统那句没写对 也就是 -append 后边那句
重新构建文件系统，一个简单例子，参考[使用qemu搭建linux内核开发环境详细教程-五]（https://blog.csdn.net/weixin_38227420/article/details/88402738）
重新构建文件系统，先来个简单的
```
vim init.c

arm-linux-gnueabi-gcc -static -o init init.c
echo init|cpio -o --format=newc > initramfs

qemu-system-arm -M vexpress-a9 -smp 4 -m 256M -kernel linux-6.6.16/arch/arm/boot/zImage -dtb linux-6.6.16/arch/arm/boot/dts/arm/vexpress-v2p-ca9.dtb -append "init=/init console=ttyAMA0" -nographic -initrd ./initramfs

```

对比下来rcS和fstab写得不一样
写一个/etc/init.d/rcS文件,注意给权限

```
#！/bin/sh
mount -t proc none /proc
mount -t sysfs none /sys
sbin/mdev -s

```
不要/etc/fstab文件，手动挂载了
```
```
制作镜像
find ./rootfs/ | cpio -o --format=newc > ./rootfs.img

跟镜像没关系，一样的错
qemu-system-arm -M vexpress-a9 -smp 4 -m 256M -kernel linux-6.6.16/arch/arm/boot/zImage -dtb linux-6.6.16/arch/arm/boot/dts/arm/vexpress-v2p-ca9.dtb -append "root=/dev/ram rdinit=sbin/init console=ttyAMA0" -nographic -initrd ./rootfs.img

![Alt text](image-7.png)
出现一样的问题，不能挂载根文件系统在未知的块
参考（https://blog.csdn.net/qq153471503/article/details/126976481）

！！！！！！！！！！！看原子哥的资料，可能是因为root = /dev/mmcblk

**SD/MMC/EMMC(卡) /dev/mmcblk0,/dev/mmcblk1 mmcblk=mmc卡**

挂载root的原因，之后试一下

还是根文件系统没构建好，参考这篇文章再重新弄吧（https://blog.csdn.net/cotex_A9/article/details/132354963）
路径要改一下，完美运行了！！！

> 1. [WSL2下Ubuntu22.04使用Qemu搭建虚拟Vexpress-A9开发板（四）——通过U-boot引导加载内核](https://blog.csdn.net/cotex_A9/article/details/132365036)
> 2. 正点原子的I.MX6U嵌入式Linux驱动开发指南V1.81（正点原子网站开源获取，前面有链接）


### 给热爱学习的小伙伴开的书单
* 深度探索Linux操作系统：系统构建和原理解析
    可以看出作者十分热爱技术，文字里充满激情，本文章借鉴了本书1——4章的内容。
* 奔跑吧Linux内核：入门篇
    这本书对初学者不太友好，我认为对于细节的处理是一本教材的灵魂，这本书恰恰在细节处的处理很潦草，有一些地方没展开讲，但本书以实验促进学习，使用qemu平台的思想很好，本系列也算是借鉴这本书的讲解思想。
* 正点原子的I.MX6U嵌入式Linux驱动开发指南V1.81（原子哥网站开源获取）
    原子哥的教程非常全，从Linux系统使用开始讲的，而且有配套实践视频，缺点就是太多了太全了，我认为初学者学习还是先有主线，先学重点，然后逐渐衍生学习才好
* qemu的help手册 boot特殊命令

### 欢迎评论
* 博主也是嵌入式初学，水平有限，文章难免有错误和不准确的地方，恳请广大读者提出宝贵意见！！！十分愿意与大家交流！！！


