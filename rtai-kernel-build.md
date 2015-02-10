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

##### ShabbyX repo

```bash
git clone https://github.com/ShabbyX/RTAI.git rtai
```

##### Deprecated (use shabbyx repo instead)
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
yes "" | make oldconfig
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

Now reboot on the new kernel. 
