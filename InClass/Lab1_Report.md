 
> [! note]
>**实验课程**: 操作系统原理实验
>**任课教师**：刘宁
>**实验题目**：实验一：编译内核利用已有内核构建OS
>**专业名称**：计算机科学与技术（系统结构方向）
>**学生姓名**：???
>**学生学号**：???
>**实验地点**：实验中心D503
>**实验时间**： Week1 & Week2

# Section 1 实验概述

本次实验我们将要熟悉现有Linux内核的编译过程和启动过程， 并在自行编译内核的基础上构建简单应用并启动。
同时，利用精简的Busybox工具集构建简单的OS， 熟悉现代操作系统的构建过程。 
此外会熟悉编译环境、相关工具集，并能够实现内核远程调试。


# Section 2 预备知识与实验环境

> 该节总结实验需要用到的基本知识，以及主机型号、代码编译工具、重要三方库的版本号信息等。

- 预备知识：x86汇编语言程序设计、Linux系统命令行工具
- 实验环境：
	- 虚拟机版本/处理器型号：`Parallel 19.2/Ubantu 20.04.02 ARM64`
	- 代码编辑环境：`vscode & vim`
	- 代码编译工具：`gcc-multiarch`
	- 重要三方库信息：
# Section 3 实验任务
- [x] 搭建OS内核开发环境包括：代码编辑环境、编译环境、运行环境、调试环境等。
- [x] 下载并编译i386（32位）内核，并利用qemu启动内核。
- [x] 熟悉制作initramfs的方法。
- [x] 编写简单应用程序随内核启动运行。
- [x] 编译i386版本的Busybox，随内核启动，构建简单的OS。
- [x] 开启远程调试功能，进行调试跟踪代码运行。
- [x] 撰写实验报告。
# Section 4 实验步骤与实验结果

该节描述每个实验任务的具体的完成过程，包括思路分析、代码实现与执行、结果展示三个部分，实验任务之间的划分应当清晰明了，实验思路分析做到有逻辑、有条理。

---
## 实验任务1 搭建环境

**任务要求**：搭建OS内核开发环境

**思路分析**：基本步骤按照实验Tutorial给出，但是！由于我的系统架构是==ARM64==，实验教程是根据普遍的X86架构进行撰写的，相关软件适配也是如此，所以很多步骤会有巨大的区别。
如编译器的选择，编译环境的配置等。

**实验步骤**：
第一个不同点是换源的内容不同，尤其注意版本兼容问题，如果版本不同的话`sudo apt update`都不能执行。ARM架构下的Ubantu 20.04.02 替换清华源如下
```text
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-backports main restricted universe multiverse

deb http://ports.ubuntu.com/ubuntu-ports/ jammy-security main restricted universe multiverse
# deb-src http://ports.ubuntu.com/ubuntu-ports/ jammy-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-proposed main restricted universe multiverse
# # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-proposed main restricted universe multiverse
```

下面继续配置C语言编译环境，需要用到交叉编译组建，教材提供的工具不兼容。
```bash
sudo apt install i686-linux-gnu-gcc
sudo apt install i686-linux-gnu-g++ 
sudo apt install gdb-multiarch
sudo apt install binutils
sudo apt install gcc
```

然后是一些其他的工具，其中部分工具可行，部分工具不可行，根据网上资料进行配置
```bash
sudo apt install qemu-system-i386
sudo apt install cmake   
sudo apt install libncurses5-dev
sudo apt install bison 
sudo apt install flex 
sudo apt install libssl-dev

sudo apt install libc6-dev-i386-cross
sudo apt install gcc-multilib-i686-linux-gnu
sudo apt install nasm
```

所有工具安装完毕后，本来想试着查看安装情况`apt list --installed`，但是...![[apt_list.png]]

发现`apt`安装的时候系统自带也有很多不知名不知用的包...没办法，至少之前在安装的时候是一个个安装检查情况了的，这里就不一一检查了。

下面安装一下vscode，注意官网刚打开的页面不是ARM架构的VSCODE！所以需要往下拉找到Deb的ARM64的VScode然后安装。

想起来之前在mac中可以快速启动，Linux应该也是可以的，于是配置`~/.bashrc`文件：
主要是前几行，人工隔离了一个自己的修改区域，添加了PATH和交叉编译的配置信息。
后续就可以`code .`优雅的打开美丽的vscode了。
![[Screenshot 2024-03-04 at 11.11.15.png]]

---
## 实验任务2 内核安装编译启动
**任务要求**：下载并编译i386（32位）内核，并利用qemu启动内核

**思路分析**：上面解决了`qemu`架构不同的问题，安装的是`qemu-system-i386`编译的时候给核给多一点编译的快很多。然后注意内核版本问题。

l**实验步骤**：
我下载的是5.10.210版本，后面的操作都要改前缀。无脑下载解压编译工作做完了之后，启动内核进行调试。
```bash
qemu-system-i386 -kernel linux-5.10.210/arch/x86/boot/bzImage -s -S -append "console=ttyS0" -nographic
```
写命令行的时候实在不舒服，太长了而且没有自动补全，遂安了一个小插件方便编辑。
如下图，启动`gdb`要用`gdb-multiarch`如下图
![[Screenshot 2024-03-04 at 11.23.02.png]]
搞错了，`gdb`也要进入目标目录，重来
![[Screenshot 2024-03-04 at 11.27.17.png]]
注意到`1.743799`的地方`panic`了，OK完成。
没有挂盘子，下面挂盘子继续
```bash
parallels@ubuntu-linux-22-04-02-desktop:~/lab1$ ls
hello.c  helloworld  hwinitramfs  initramfs-busybox-x86.cpio.gz  linux-5.10.210  mybusybox
```
![[Screenshot 2024-03-04 at 11.33.28.png]]
`gdb` 了好几次`continue`没反应，突然想起来是程序里跑了个死循环，怪不得。

---
## 实验任务3 Busybox

**任务要求**：编译安装搭载Busybox

**思路分析**：根据实验要求进行即可。遇到的唯一困难是`init`文件无法被kernel识别。经过QQ群友提示或许是因为kernel无法识别未知类型文件，换句话说，就是不知道怎么去读取它。解决了这个问题后就顺利了。

**实验步骤**：
先按照实验教程配置好了busybox
![[Screenshot 2024-03-04 at 11.37.38.png]]
下面进行打包，其中init文件要修改一下
```bash
#!/bin/sh
mount -t proc none /proc
mount -t sysfs none /sys
echo -e "\nBoost took $(cut -d' ' -f1 /proc/uptime) seconds\n"
exec /bin/sh
```
好，完成后制作initrm启动内核，并且加载，实验结果如下。
![[Screenshot 2024-03-04 at 11.40.10.png]]
# Section 5 实验总结与心得体会
主要问题在于架构不兼容，但是想了想，主要不兼容的问题出在编译器上。
因为实验修改内核，这个内核的程序是模拟了x86架构的，然后qemu本身是一个虚拟机，无论在什么架构上，它都可以为kernel执行提供硬件的模拟。那么这一个层面是没有问题的。

继续想想，qemu模拟内核没有问题，那么问题就在qemu的补上了。换句话说，不跑在qemu里面的程序，但是需要放到kernel中运行的需要编译的程序，就会出现问题。

本实验涉及到了一个，就是`hello_world.c`的编译。如果用ARM架构下的编译器，编译出来的机器代码是给ARM看的，不是给X86的Kernel看的，但是这个`hello_world`又需要加载到内核中，所以，使用交叉编译程序。
```bash
i686-linux-gnu-gcc -o helloworld -m32 -static helloworld.c
```
同理，后面用g++也是一个道理。
这里就是多了一串前缀而已，其实一点都不麻烦。