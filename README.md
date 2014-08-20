M3 Installation instructions
==============

This wiki describes the full installation of m3 software to control/simulate the Meka robot at Ensta ParisTech.

>  *Author* : Antoine Hoarau <hoarau.robotics@gmail.com>


| ***OS Tested*** | ***Status*** | ***Notes***
|:------------------|:----:|:---------------:
| Ubuntu 12.04 x86 | OK | Kinects might cause issues 
| Ubuntu 12.04 x64 | OK | Kinects might cause issues 
| Ubuntu 12.10 x86 | OK | On the Meka 04.04.14 -> will update to 14.04 LTS when ready 
| Ubuntu 12.10 x64 | OK | Now working 04.04.14 
| Ubuntu 13.04 x86 | OK | 
| Ubuntu 13.04 x64 | OK | 
| Ubuntu 13.10 x86 | OK | w ROS Indigo 
| Ubuntu 14.04 x64 | OK | w ROS Indigo/MoveIt! 


## Ubuntu 12.04 - 14.04 (x86/x64) w/ ROS Hydro/Indigo

### Prerequisites
#### Necessary 
```bash
sudo apt-get install cmake git libeigen3-dev libprotobuf-dev protobuf-compiler gnuplot-x11 libboost-dev python-dev python-protobuf python-matplotlib python-yaml python-gnuplot python-scipy python-sip-dev python-sip sip-dev swig
```
#### Nice to have to maybe compile a kernel later
```bash
sudo apt-get install libqt4-dev moc g++ libncurses5-dev kernel-package gcc-multilib libc6-dev libtool automake  openssh-server openssh-client
```
------

### The RTAI-patched kernel
#### Download
```bash
# Determine if x86 or x64 (x86_x64)
_platform=$(uname -m) 

# Get the Rtai4.0 patched kernel headers
wget http://perso.ensta-paristech.fr/~hoarau/rtmeka-kern/$_platform/linux-headers-rt.deb

# Get the Rtai4.0 patched kernel image
wget http://perso.ensta-paristech.fr/~hoarau/rtmeka-kern/$_platform/linux-image-rt.deb
```
#### Installation

```bash
sudo dpkg -i linux-headers-rt.deb linux-image-rt.deb
```

Now **boot** on the new kernel using **grub** at **startup**. Please note the name of the kernel.
> Note : you might have to either hold sift on startup or update the grub config to boot on the rtai patched kernel: 
```bash
sudo nano /etc/defaults/grub
```

### RTAI 4.0 installation 
#### Download
```bash
wget --no-check-certificate https://www.rtai.org/userfiles/downloads/RTAI/rtai-4.0.tar.bz2
tar xjf rtai-4.0.tar.bz2
```

#### Installation
```sh
cd rtai-4.0
mkdir build; cd build
../configure --disable-comedi-lxrt --enable-cpus=$(nproc) --enable-math-c99 --with-linux-dir=/usr/src/linux-headers-3.8.13-rtmeka4.0
make -j$(nproc)
sudo make install
```
> **Note** : The --with-linux-dir option has to match the rtai-patched kernel

> ----
> **Know issues** : On 64-bit CPUs, if an error regarding -mpreferred-cache-boundary=3 shows up, edit line 57 in /usr/src/linux/arch/x86/Makefile (where linux is your rtai patched kernel) to set this parameter to 4:
```bash
KBUILD_CFLAGS += $(call cc-option,-mno-sse -mpreferred-stack-boundary=4)
```
Part of the explanation: http://mail.rtai.org/pipermail/rtai/2013-December/026198.html

> ----
> **Know issues** : on 12.04 32 bits machines, rtai fails to compile (some header is missing)
```bash
sudo apt-get install gcc-multilib g++-multilib libc6-dev
sudo ln -s /usr/include/i386-linux-gnu/gnu/stubs-32.h /usr/include/gnu/stubs-32.h
```

### Post install
Update the ld library path to find rtai:
```bash
sudo -s
echo /usr/realtime/lib/ > /etc/ld.so.conf.d/rtai.conf
exit
sudo ldconfig
```


## Install ROS-HYDRO
!### If not at ENSTA
```bash
codename=`cat /etc/lsb-release | grep -m 1 "DISTRIB_CODENAME=" | cut -d "=" -f2`
sudo sh -c "echo 'deb http://packages.ros.org/ros/ubuntu $codename main' > /etc/apt/sources.list.d/ros-latest.list"
```
!### If at ENSTA (Just to go faster, you can use the above one)
```bash
codename=`cat /etc/lsb-release | grep -m 1 "DISTRIB_CODENAME=" | cut -d "=" -f2`
sudo sh -c "echo 'deb http://fermion.ensta.fr/ros/ubuntu $codename main' > /etc/apt/sources.list.d/ros-latest.list"
```
```bash
wget http://packages.ros.org/ros.key -O - | sudo apt-key add -
sudo apt-get update
export ROS_DISTRO=hydro
sudo apt-get install ros-$ROS_DISTRO-desktop-full ros-$ROS_DISTRO-moveit-full ros-$ROS_DISTRO-openni* ros-$ROS_DISTRO-ros-control ros-$ROS_DISTRO-ros-controllers python-rosinstall python-pip
sudo apt-get update
#sudo apt-get install --only-upgrade python-rosdistro python-rosdep #python-rosinstall python-rosinstall-generator python-bloom
#sudo pip install --upgrade rosdistro
```

```bash
#_user=$(id -u)
#_group=$(id -g)
#sudo chown -R $_user:$_group /usr/local/ 
#sudo apt-get install curl
#curl https://bitbucket.org/pypa/setuptools/raw/bootstrap/ez_setup.py | #python
#sudo pip install --upgrade rosdistro
```
```bash
sudo -E rosdep init
rosdep update

source /opt/ros/hydro/setup.bash
## Create the ROS-workspace
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/src
catkin_init_workspace
cd ~/catkin_ws/
catkin_make
```
```bash
sudo -s
echo '/usr/local/lib' >> /etc/ld.so.conf
exit
sudo ldconfig
```

## (FIXED) Workaround for KDL issues (Go back here AFTER install)

The function SetPayload that allow to modify how much weight the robot carries requires a tiny patch on the KDL library. This is a temporary solution.

```bash
cd ~/catkin_ws/src
git clone https://github.com/ahoarau/orocos_kinematics_dynamics
cd orocos_kinematics_dynamics/orocos_kdl
mkdir build;cd build; cmake ..
make -j5
sudo make install
cd ../../python_orocos_kdl/
mkdir build;cd build;cmake ..
make -j5
sudo make install 
```



##Install some M3 required libraries
##!Generic libraries
```bash
sudo apt-get install cmake libboost-dev libtool swig python-dev ipython subversion g++ python-dev python-yaml python-gnuplot python-matplotlib  libprotobuf-dev libprotoc-dev python-protobuf nfs-common vim git libeigen2-dev libeigen2-doc libyaml-0-2 ntp autoconf python-numpy python-scipy python-matplotlib ipython ipython-notebook python-pandas python-sympy python-nose ssh
```

##(Recommended) Install some IDEs

#Python (for most users): '''Eclipse+PyDev'''
```bash
sudo apt-get install eclipse
```

#ROS and C++ Real-time (Advanced users): '''Qt creator''' and/or Kdevelop
```bash
sudo apt-get install qtcreator 
sudo apt-get install kdevelop
```

##Get the whole ENSTA M3 

```bash
ssh-keygen
```
Press Enter 3 times.
```bash
gedit ~/.ssh/id_rsa.pub
```
Copy the whole key, then go to [[https://bitbucket.org/|Bitbucket]] and add your ssh key to ensta-user account.
Top right corner,Manage account, SSh key, Add key (follow the instructions).

login : ensta-user 
password : &ROBOT!CS

```bash
#cd
#git clone git@bitbucket.org:ensta/kdl-meka
#cd kdl-meka
#mkdir build;cd build;cmake ..
#make -j5
#sudo make install
#cd ../../
#rm -rf kdl-meka
```

```bash
#cd
#git clone git@bitbucket.org:ensta/holomni_pcv.git
#cd holomni_pcv
#./autogen.sh
#./configure
#make -j5
#sudo make install
```

##Install ENSTA M3

### Download
```bash
git clone --recursive git@bitbucket.org:ensta/mekabot.git ~/mekabot
cd ~/mekabot
#git checkout cmakemigration
git submodule init
git submodule update
git submodule foreach git checkout master

#cd m3core
#git checkout cmakemigration
#cd m3meka
#git checkout cmakemigration
#cd holomni_pcv
#git checkout cmakemigration

cd m3ens
git checkout newvirtualmeka
```
```bash
cd ~/mekabot
source m3core/scripts/disable_ros
cd holomni_pcv
mkdir build;cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j5
sudo make install
cd ~/mekabot
mkdir build;cd build
cmake .. -DETHERCAT=0 -DCMAKE_BUILD_TYPE=Release
make -j5
sudo make install
```

```bash
#cd ~/mekabot/m3core
#./autogen.sh
#./configure --disable-ethercat
#make -j5
#sudo make install
#git add .
#git stash
```


```bash
#source scripts/disable_ros
#cd ~/mekabot/m3meka
#./autogen.sh
#./configure
#make -j5
#sudo make install
#git add .
#git stash
```

##!Update your bashrc
```bash
touch ~/.m3rc
echo '
## Meka
export M3_ROBOT=~/mekabot/m3ens/real_meka
export MALLOC_CHECK_=0

## ROS
export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:~/mekabot/m3core/ros:~/mekabot/m3meka/ros:~/mekabot/meka-ros-pkg
export ROS_MASTER_URI=http://meka-moch:11311
export ROS_IP=192.168.20.117
source /opt/ros/hydro/setup.bash

## ROS-workspace
source ~/catkin_ws/install_isolated/setup.bash
source ~/catkin_ws/devel/setup.bash

## Additional Meka-stuff
export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:~/mekabot/m3ens-demos/ros:~/mekabot/m3ens-tutos/ros:~/mekabot/m3ens-utils/ros:~/mekabot/meka-ros-pkg:~/mekabot/m3core/ros:~/mekabot/m3meka/ros
export PYTHONPATH=$PYTHONPATH:~/mekabot/m3ens-demos/scripts:~/mekabot/m3ens-utils/scripts:~/mekabot/m3ens-utils/python:~/mekabot/m3ens-utils/ros
'>>~/.m3rc

echo 'source ~/.m3rc' >> ~/.bashrc
source ~/.bashrc
```

##!Get time synchronization for ROS (HIGHLY RECOMMENDED)
```bash
sudo nano /etc/ntp.conf
```
Comment all the servers lines and add 'server ensta.ensta.fr'.
It should look like that : 
```bash
#server 0.ubuntu.pool.ntp.org
#server 1.ubuntu.pool.ntp.org
#server 2.ubuntu.pool.ntp.org
#server 3.ubuntu.pool.ntp.org
server ensta.ensta.fr

# Use Ubuntu's ntp server as a fallback (or not at ensta ;) )
server ntp.ubuntu.com
```
```bash
sudo service ntp restart
```

Force the time to update every day (can drift after long shutdown) (OPTIONAL)
```bash
sudo -s
touch /etc/cron.daily/ntpdate
echo '#!/bin/sh
ntpdate ensta.ensta.fr'>>/etc/cron.daily/ntpdate
exit
sudo chmod 755 /etc/cron.daily/ntpdate
```



##Setup robot's Pcs (OPTIONAL):
```bash
sudo -s
echo '192.168.20.117 meka-mob'>>/etc/hosts
echo '192.168.20.118 meka-moch'>>/etc/hosts
echo '192.168.20.119 meka-mud'>>/etc/hosts
exit
```


Now open the robot_config :

```bash
gedit $M3_ROBOT/robot_config/m3_config.yml
```

And change the host name to your computer's hostname.

```bash
more /etc/hostname
```




## You're done ! you can try to launch something now.
