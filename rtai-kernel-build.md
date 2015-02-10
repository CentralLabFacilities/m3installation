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

Let's use your computer's full power (multi-threaded build):
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

#### Example for kernel 3.10.32

* General Setup ---> local version -append to kernel release: -rtai4.0
* General Setup ---> Timer Subsystem ---> Timer tick handling ---> Periodic timer ticks (constant rate, no dynticks)
* General Setup ---> Timer Subsystem ---> High Resolution Timer Support ---> disabled
* General Setup ---> Enable loadable module support ---> Module versioning support ---> disable
* Processor type and features ---> Processor Family ---> Core2/nexer Xenon if "more /proc/cpuinfo | grep family" says "6"
* Processor type and features ---> SMT (Hyperthreading) scheduler support ---> disabled
* Processor type and features ---> Preemption Model ---> Preemptible Kernel (Low-Latency Desktop)
* Processor type and features ---> Timer Frequency ---> 1000 Hz
* Power management and ACPI options ---> remove everything
* Power management and ACPI options ---> ACPI ----> remove every thing **except** Power Management Timer Support (and button, let it be a kernel module (dot) )
* Power management and ACPI options ---> CPU Frequency scaling ---> disabled
* Power management and ACPI options ---> Memory Power savings ---> Intel chipset idle memory power saving driver ---> disabled 


Save and close the window.


> Note: You can find some configuration hints on the xenomai website : http://xenomai.org/2014/06/configuring-for-x86-based-dual-kernels/

#### Compile the new RT kernel

```bash
make-kpkg clean
make-kpkg --rootcmd fakeroot --initrd kernel_image kernel_headers
```
Now take a cofee and come back in ~20min.


#### Install the new RT kernel

```bash
cd ~/linux-$kernel_version-rtai-$rtai_version/
sudo dpkg -i linux-headers-$kernel_version-rtai-$rtai_version_$kernel_version-rtai-$rtai_version-10.00.Custom_i386.deb
sudo dpkg -i linux-image-$kernel_version-rtai-$rtai_version_$kernel_version-rtai-$rtai_version-10.00.Custom_i386.deb
```

Now reboot on the new kernel to test it.
