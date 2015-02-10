How to build an RTAI patched kernel from scratch
======

## The Linux Kernel
Installation parameters : 
```bash
# linux Kernel
kernel_version_major=3
kernel_version_minor=10
kernel_version_patch=32

kernel_version=$kernel_version_major.$kernel_version_minor.$kernel_version_patch
# rtai
rtai_version=4.0

```
Let's create a nice folder to put everything (make sure you have at least ~2GB free space) :
```bash
mkdir ~/linux-$kernel_version-rtai-$rtai_version
cd ~/linux-$kernel_version-rtai-$rtai_version
```

#### Prerequisites
For menuconfig to work:
```bash
sudo apt-get install libncurses5-dev kernel-package libc6-dev
```
And for xconfig to work:
```bash
sudo apt-get install qt4-dev-tools
```

#### Download the kernel

```bash
cd ~/linux-$kernel_version-rtai-$rtai_version
wget https://www.kernel.org/pub/linux/kernel/v$kernel_version_major.x/linux-$kernel_version.tar.gz
tar xvf linux-$kernel_version.tar.gz
ln -snf linux-$kernel_version linux
```

## RTAI 4.x
#### Download

```bash
cd ~/linux-$kernel_version-rtai-$rtai_version
wget --no-check-certificate https://www.rtai.org/userfiles/downloads/RTAI/rtai-$rtai_version.tar.bz2
tar xjf rtai-$rtai_version.tar.bz2
ln -snf rtai-$rtai_version rtai
```

Let's use your computer's full power (speeds up the compilation):
```bash
sudo -s
echo "CONCURRENCY_LEVEL=$[$(nproc)+1]" >> /etc/kernel-pkg.conf
exit
```
#### Apply the patch to the kernel 

```bash
cd ~/linux-$kernel_version-rtai-$rtai_version/linux
patch=$(find ~/linux-$kernel_version-rtai-$rtai_version/rtai/base/arch/x86/patches/ -name "*$kernel_version*" | tail -1)
echo "Applying $patch"
patch -p1 -b < $patch
```


#### Configure your realtime kernel 
```bash
cd ~/linux-$kernel_version-rtai-$rtai_version/linux
make oldconfig
make xconfig
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
make-kpkg clean
make-kpkg --rootcmd fakeroot --initrd kernel_image kernel_headers
```


```bash
cd ~/linux-$kernel_version-rtai-$rtai_version/
sudo dpkg -i linux-headers-$kernel_version-rtai-$rtai_version_$kernel_version-rtai-$rtai_version-10.00.Custom_i386.deb
sudo dpkg -i linux-image-$kernel_version-rtai-$rtai_version_$kernel_version-rtai-$rtai_version-10.00.Custom_i386.deb
```


```bash
cd ~/linux-$kernel_version-rtai-$rtai_version/rtai
mkdir build
cd build
make -f ../makefile menuconfig
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
