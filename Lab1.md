# Compile the Kernal

## 1 Building the Environment

Firstly, we luanch the virtual machine, and we will see the desktop like this:

![desktop](/Users/kehao/CodeSpace/SYSU-OS/Appendix/desktop.png)

OK! Then we will have a journey with that 🪼

Hit the terminal, the icon like a black box is up to the dustbin. Then you could type some pure word command to control the computer!

> Terminal is another control method in computer which is early than the Graphic one. 

### 1.1 Change the Source of downloading

Type this code script in command line.

```bash
sudo mv /etc/apt/source.list /etc/apt/source.list.backup
```

> sudo is the super mode to control the computer! Just input your secret set previously.

Supposed your ubantu version is the same with me, and your architecture is ARM64

`gedit /etc/apt/sources.list` and then copy the code below:

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

This procedure is intended to **speed up** the process of downloading from internet.

Accordingly, it's optional, expecially you have a VPN tunnel. However, i personally recommand you to take this step.

After copying, don't forget to `save`. To check it out, input `sudo apt update`

> Oppps, what is apt then? 
>
> - APT means *advanced package tool*. Frankly speaking, just let you to download app in command line. Or you can compare it to appstore in apple device.

### 1.2 Prepare for c/c++

```bash
sudo apt install binutils
sudo apt install gcc
```

`binutils` is a set of program to create and manage binary file.

`gcc` is a suite of tools to compile.

**Besides, install this also:**

```bash
sudo apt install i686-linux-gnu-gcc
sudo apt install i686-linux-gnu-g++ 
sudo apt install gdb-multiarch
```

To check, `program_name --version` or `program_name -v`

### 1.3 Other tools

```bash
sudo apt install qemu-system-i386
sudo apt install cmake   
sudo apt install libncurses5-dev
sudo apt install bison 
sudo apt install flex 
sudo apt install libssl-dev

sudo apt install libc6-dev-i386-cross
sudo apt install gcc-multilib-i686-linux-gnu
# well, still not found g++-multilib...so, go to hell then.
sudo apt install nasm
```

Then, downloading vscode, i love that editor! It has sea of themes for you to choice!

Don't forget you have a browser on your ubuntu. His name is Firefox. Enter this web: https://code.visualstudio.com/#alt-downloads 

**Drag it to the bottom**, you will see this:

![download_vscode](/Users/kehao/CodeSpace/SYSU-OS/Appendix/download_vscode.png)

Just install with option default...

After installed, to quickstart vscode, `gedit ./bash` and add one line up

```bash
export PATH=$PATH:/home/parallels/application/VSCode-linux-arm64/bin
```

==**Attention! Change your own path!**==

And then you could open the editor elegantly with `code .` in your target directory.

## 2 Compile the kernal

### 2.1 download the kernal

Why download another kernal?

The ubuntu itself have a kernal itself! You can't change it! Supposed you are change something that is related to the boost program, then you have totally lost your first ubantu friend in os world...Cuz u can not  say hello to her anymore...

**BUT!** you just download another kernal, you can make any changes you want to it without any anxiety! **Cuz it is just a virtual operating system equipped on your virtual machine.**

Oh, don't forget the version, every code below need to change your own version!

### 2.2 compile

First thing to do, equip your ubante with **cross complie environment**

```bash
gedit ~/.bashrc
```

Add up:

```text
export ARCH=x86
export CROSS_COMPILE=i686-linux-gnu-
```

Then, we can happily run the command now! 

## 3 Boost the kernal and debug

```bash
qemu-system-i386 -kernel linux-5.10.210/arch/x86/boot/bzImage -s -S -append "console=ttyS0" -nographic
```

Open **another** terminal:

```
gdb-multiarch
```

![debug_with_gdb](/Users/kehao/CodeSpace/SYSU-OS/Appendix/debug_with_gdb.png)

# 4 Create Initramfs

Use `i686-linux-gnu-gcc` , downloaded previously, to replace `gcc` for multi-compile.

```bash
i686-linux-gnu-gcc -o helloworld -m32 -static helloworld.c
```

To quit the qemu : `control+a`+`x`

To quit the gdb: `quit`

## 5 Build busybox

Busybox is a set of basic program on linux, which is so tiny that it can run in the embeded machine.

*Don't forget to enter the directory before you `make`*

When put the new busybox in your own place `~/lab1/mybusybox`, we need to make a true guy rather than helloworld.

Wait, create a init program in `~/lab1/mybusybox` first.**Focus the change on the first line:**

```bash
#!/bin/sh
mount -t proc none /proc
mount -t sysfs none /sys
echo -e "\nBoost took $(cut -d' ' -f1 /proc/uptime) seconds\n"
exec /bin/sh
```

> - `#!/bin/sh` specify the interpreter of the init program reader afterwards.

Everything done well, you would see the file tree like this:

```bash
parallels@ubuntu-linux-22-04-02-desktop:~/lab1$ ls
hello.c     hwinitramfs                    linux-5.10.210
helloworld  initramfs-busybox-x86.cpio.gz  mybusybox
```



```bash
parallels@ubuntu-linux-22-04-02-desktop:~/lab1/mybusybox$ ls
bin  etc  init  linuxrc  proc  sbin  sys  usr
```

Then just go with the oringinal tutorial!

*Here is the final! Congulation for finishing the first lab!*







