How to build an RTAI patched kernel from scratch
======

# The Linux Kernel
Current version (Works also with 2.6 Kernels, 3.4.6, 3.4.67 , and 3.5.7).
```bash
kernel_version=3.8.13
```

## Prerequisites
For menuconfig to work:
```bash
sudo apt-get install libncurses5-dev kernel-package libc6-dev
```
And for xconfig to work:
```bash
sudo apt-get install qt4-qmake libqt4-dev
```

## Preparation
```bash
wget https://www.kernel.org/pub/linux/kernel/v3.x/linux-3.4.67.tar.bz2
tar xjf linux-3.4.67.tar.bz2
mv linux-3.4.67 linux-build-3.4.67-rtai-4.0
ln -snf linux-build-3.4.67-rtai-4.0 linux
sudo chmod g-s linux-build-3.4.67-rtai-4.0 -R
```

## RTAI 4.0
#### Download

```bash
#linux
```



```bash
cd /usr/local/src
wget --no-check-certificate https://www.rtai.org/userfiles/downloads/RTAI/rtai-4.0.tar.bz2
tar xjf rtai-4.0.tar.bz2
ln -s rtai-4.0 rtai
```

Let's use your computer's full power (speeds up the compilation):
```bash
sudo -s
echo "CONCURRENCY_LEVEL=$[$(nproc)+1]" >> /etc/kernel-pkg.conf

exit
```

```bash
cd /usr/src/linux
sudo patch -p1 -b < /usr/local/src/rtai/base/arch/x86/patches/hal-linux-3.4.67-x86-4.patch
```

```bash
cd /usr/src/linux
sudo cp /boot/config-`uname -r` ./.config
sudo make oldconfig
sudo make xconfig
```

* Loadable module support ---> Module versioning support ---> disabled
* Processor type and features ---> Processor Family ---> Core2/Xenon
* Processor type and features ---> HPET Timer Support ---> disabled
* Processor type and features ---> Interrupt pipeline ---> enabled [*]
* Processor type and features ---> Support sparse irq numbering ---> disabled
* Device Drivers ---> Staging Drivers ---> disabled
* Device Drivers ---> Sound Card Support ---> Open Sound System ---> disabled
* Power management options > Power Management support = no
* Power management options > CPU Frequency scaling > CPU Frequency scaling = no

* General setup > Local version - append to kernel release = -rtai-4.0

```bash
sudo make-kpkg clean
sudo  make-kpkg --rootcmd fakeroot --initrd kernel_image kernel_headers
```

```bash
sudo make-kpkg clean 
```

```bash
cd /usr/src/
sudo dpkg -i linux-headers-3.4.67-rtai-4.0_3.4.67-rtai-4.0-10.00.Custom_i386.deb
sudo dpkg -i linux-image-3.4.67-rtai-4.0_3.4.67-rtai-4.0-10.00.Custom_i386.deb
```

```bash
cd /usr/src/linux
sudo make modules_prepare
```

```bash
cd /usr/local/src/rtai
sudo mkdir build
cd build
sudo make -f ../makefile menuconfig
```

* General > Linux source tree = /usr/src/linux
* Machine (x86) > Number of CPUs = (2) for Core-Duo or (4) for Quad-Core
* DISABLE COMEDI
* ENABLE MATH FUNC IN KERNEL

```bash
sudo apt-get install gcc-multilib g++-multilib libc6-dev
sudo ln -s /usr/include/i386-linux-gnu/gnu/stubs-32.h /usr/include/gnu/stubs-32.h
sudo make install
```

```bash
sudo cp -a /dev/rtai_shm /lib/udev/devices/
sudo cp -a /dev/rtf[0-9] /lib/udev/devices/
```

```bash

 sudo -s
      echo /usr/realtime/lib/ > /etc/ld.so.conf.d/rtai.conf
      exit
      sudo ldconfig


The RTAI binaries directory can be added automatically to the $PATH variable. To do that,add

      /usr/realtime/bin
to /etc/environment and then

   source /etc/environment
```
```bash
sudo apt-get install udev
cd /usr/local/src
wget http://www.etherlab.org/download/ethercat/ethercat-1.5.2.tar.bz2
tar xjf ethercat-1.5.2.tar.bz2
ln -s ethercat-1.5.2 ethercat
cd ethercat
./configure --enable-cycles --with-rtai-dir=/usr/realtime --enable-r8169 --disable-8139too --enable-e1000 --enable-e1000e
make all modules
sudo make modules_install install
sudo depmod
```
```bash
sudo ifconfig
sudo mkdir /etc/sysconfig/
sudo cp /opt/etherlab/etc/sysconfig/ethercat /etc/sysconfig/
sudo nano /etc/sysconfig/ethercat
```


MASTER0_DEVICE="00:04:A7:09:77:68"
DEVICE_MODULES="r8169"


```bash
cd /opt/etherlab
sudo cp etc/init.d/ethercat /etc/init.d/
sudo chmod a+x /etc/init.d/ethercat
sudo update-rc.d ethercat start 51 S .
```
```bash
sudo ln -s /opt/etherlab/bin/ethercat /usr/local/bin/ethercat
sudo /etc/init.d/ethercat start
```
```bash
wget http://perso.ensta-paristech.fr/~hoarau/rtmeka-kern/x86/linux-headers-3.8.13-rtmeka4.0_3.8.13-rtmeka4.0-10.00.Custom_i386.deb
wget http://perso.ensta-paristech.fr/~hoarau/rtmeka-kern/x86/linux-image-3.8.13-rtmeka4.0_3.8.13-rtmeka4.0-10.00.Custom_i386.deb
```