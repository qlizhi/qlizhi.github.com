---
layout: post
styles: [syntax]
title: Linux网络驱动
---

## Linux内核启动过程（2.6内核）

1

## 内核空间和用户空间

内核态和用户态是硬件上的说法，从Intel80386开始CPU可以运行在ring0~ring3从高到低四个不同权限模式，但实际只用两个。“内核态”对应CPU的ring0层，是内核运行的模式，设备驱动程序就是运行在该模式下。“内核态”下可以控制内存映射方式、特殊状态寄存器、中断和DMA，可以无限制的访问所有处理器指令集以及全部内存和I/O空间；另一个被称为“用户态”对应ring3层，所有的用户程序都运行在该层。   
内核空间和用户空间是软件上的说法，Linux的虚拟地址空间为0~4G，Linux将这4G字节的空间分成两部分。最高的1G字节供内核使用，称为“内核空间”。两个模式分别引用不同的地址映射，换句话说，程序代码使用不同的地址空间。由此可见，想直接通过指针把“用户空间”的数据地址传递给内核是不可能的，必须要经过一步地址转换将“用户态”的地址转换成“内核态”的地址。如copy_to_user、copy_from_user等。

* * * * * *
## Linux设备驱动基础

Linux一开始就将外部设备全都当成“文件”处理。从某种意义上讲，凡是能产生或接受消息的都是“文件”。  
应用程序通过/dev目录中的设备节点操作设备，通过/sys目录中的节点收集设备信息。

## Linux设备模型之udev

几年前，Linux操作系统还不成熟，管理设备节点的工作一点都不好玩。所有需要的节点（达到数千个）都必须事先在/dev目录下静态创建。在2.4内核中，引入了devfs，它支持设备节点的动态创建。devfs提供了在内存内的文件系统中创建设备节点的能力，而命名节点的任务还是落在设备驱动程序头上。但是，设备命名策略是可管理的，不应该与内核混在一起。而udev将设备管理的任务推向了用户空间。udev的工作取决于以下几项：
  
1. 内核中的sysfs支持。sysfs是Linux设备模型的一个重要组成部分，位于内存中，在启动时被挂载在/sys目录下（见/etc/fstab）。  
1. 一套用户空间守护程序和实用工具，如udevd和udevinfo。  
1. 用户自定义的规则，位于/etc/udev/rules.d/目录中。可以根据对应设备的特点设置规则。  

为了理解udev的用法，我们先看一个例子。假定你有一个USB DVD驱动器或一个USB CD-RW驱动器。根据热插拔设备顺序的不同，一个被命名为/dev/sr0,另一个被命名为/dev/sr1。在没有udev的情况下，必须执行区分这些名字对应的设备。但是，有了udev以后，不管以什么顺序热插拔它们，都能分辨出二者，如DVD命名为/dev/subdvd,CD-RW命名为/dev/usbcdrw。  
首先，从sysfs相应的文件中提取产品信息。假定DVD驱动器被分配的设备节点为/dev/sr0，CD-RW驱动器的为/dev/sr1,使用udevinfo可收集设备信息：  
![Alt "1"](/static/images/post/LinuxDriver1.jpg)  
![Alt "1"](/static/images/post/LinuxDriver2.jpg)  
接下来，使用搜集到的产品信息标识设备并且添加udev命名规则。创建/etc/udev/rules.d/40-cdvd. rules文件并添加如下规则信息：  
![Alt "1"](/static/images/post/LinuxDriver3.jpg)  
第1条规则告诉udev，一旦发现一个USB设备的产品ID为0x0701，厂商ID为0x05e3，就增加一个以sr开始的名称，udev将在/dev目录下创建一个同名的节点并为之创建一个名为usbdvd的符号链接。类似的，第2条规则为CD-RW驱动器创建一个名为usbcdrw的符号链接。  
为了测试新创建规则的语法错误，可以对/sys/block/sr* 运行udevtest。为了打开/var/log/messages  中的相关提示信息，可以将/etc/udev/udev.conf文件中的udev_log设置为“yes”。为了在运行过程中对/dev目录应用新增规则，可以运行udevstart重启udev。此后，你的DVD驱动器在系统中将始终为/dev/usbdvd。你肯定可以使用如下命令从shell脚本中挂载设备：  
`mount /dev/usbdvd /mnt/dvd`  
设备节点（以及网络接口）的一致性命名并非udev的唯一功能。实际上，udev已经演变为Linux热插拔管理器，并且也承担了按需自动加载模块和为设备下载微码的任务。

## Linux设备模型之sysfs、kobject和设备类

sysfs、kobject和设备类是设备模型的组成模块，它们主要在总线和核心层的实现中使用，隐藏在为设备驱动程序提供服务的API中。
- 内核的结构化设备模型在用户空间就称为sysfs。它与procfs类似，二者都位于内存的文件系统中，而且包含内核数据结构的信息。但是，procfs是查看内核内部的一个通用视窗，而sysfs则特定地对应于设备模型。因而，sysfs并非procfs的替代品。进程描述符、sysctl参数等信息属于procfs而非sysfs。很快我们会发现，udev的大多数功能都取决于sysfs。  
- kobject封装了一些公用的对象属性，如引用计数。它们通常被嵌在更大的数据结构中。kobject的主要字段如下（定义于include/linux/kobject.h文件中）。  
  1. kref对象，用于引用计数管理。kref_init()接口用于初始化kref接口，kref_get()用于增加kref相关的引用计数，而kref_put()则用于减少引用计数，当没有剩下的引用后，对象会被释放。  
  1. kset的指针，表征kobject归属的对象集（object set）。  
  1. kobj_type，用于描述kobject的对象类型。  
- 设备类是设备模型的另一个特点，在驱动程序中，我们更有可能用到此接口。类接口抽象了这一理念，即每个设备都属于某一个总括性的分类。例如，usb鼠标、ps/2键盘和操纵杆都属于输入设备类，都在/sys/class/input/目录下拥有入口。

以RTC（drivers/char/rtc.c）驱动程序为例， 它是一个misc类驱动程序。当加载rtc驱动时 `modprobe rtc`。  
![Alt "1"](/static/images/post/LinuxDriver4.jpg)   
如上图所示，会创建/sys/class/misc/rtc/目录，创建/sys/class/misc/rtc/dev以及/sys/class/misc/rtc/uevent文件，并且创建/dev/rtc设备节点。  

设备模型的另一个抽象是总线--设备--驱动程序。内核把设备划为总线、设备和驱动程序。这使得单独的驱动程序更容易实现。bus_register()会为/sys/bus增加一个相应的入口，而device_register()会为/sys/devices/增加相应的入口。driver_register()注册相应驱动。

## Linux设备模型之热插拔和冷插拔

在运行过程中往系统中插入设备称为热插拔，而在系统启动前就连接的设备称为冷插拔。以前，内核通过调用由/proc文件系统注册的辅助程序来向用户空间通知热插拔事件。而在当前的内核里，侦测到热插拔事件后，它们会通过netlink套接字向用户空间派生uevent。netlink套接字是一种在内核空间和用户空间透过套接字API进行通信的有效机制。用户空间的udevd（管理设备节点创建和移除的守护程序）会接收uevent并管理热插拔。udev也处理冷插拔。由于udev是用户空间的一部分，仅仅在内核启动后才开始运行，所以需要一种特殊的机制针对冷插拔设备模拟热插拔事件。启动时，内核为所有设备在sysfs下创建了一个名为uevent的文件，并将冷插拔事件记录在这些文件中。当udev开始运行后，它读取/sys下所有的uevent文件，并为每个冷插拔设备产生热插拔uevent。   
[uevent](http://blog.csdn.net/bingqingsuimeng/article/details/7924625)的事件在kobject_action中定义：
{% highlight javascript linenos %}
enum kobject_action {
       KOBJ_ADD,
       KOBJ_REMOVE,
       KOBJ_CHANGE,
       KOBJ_MOVE,
       KOBJ_ONLINE,
       KOBJ_OFFLINE,
       KOBJ_MAX
};
{% endhighlight %}


## Linux设备模型之模块自动加载

按需自动加载内核模块是Linux支持的一种方便特性。为了理解内核怎样发起“模块缺失”事件以及udev怎么处理它，下面以向笔记本计算机的PC卡插槽插入Xircom CardBus以太网适配器为例进行说明。 
 
1. 在编译的时候，驱动程序包含了一个它所支持的设备的标识列表。查看Xircom CardBus网卡的驱动程序（drivers/net/tulip/xircom_cb.c）,可以发现如下的代码片段：
{% highlight javascript linenos %}
static struct pci_device_id xircom_pci_table[] = {
	{0x115D, 0x0003, PCI_ANY_ID, PCI_ANY_ID,},
	{0,},
};

/* Mark the device table */
MODULE_DEVICE_TABLE(pci, xircom_pci_table);
{% endhighlight %}

## 参考文献

Essential Linux Device Drivers（精通Linux设备驱动程序开发）