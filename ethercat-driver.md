Build an EtherCAT driver for a non-supported kernel
=====
The Meka robot at Ensta uses a combination of [Rtai](https://www.rtai.org/) + [EtherCAT](http://etherlab.org/en/ethercat/) to control the motors.

> Update 22 august 2014
## Introduction
* The ethercat master 1.5.2 for linux (i.e etherlab)  [only supports a specific set of kernels](http://etherlab.org/en/ethercat/hardware.php) (2.6.x, 3.2, 3.4) with specific Ethernet drivers : the one on the meka is **r8169**, which is supported on almost all listed kernels.

* Rtai also supports a limited set of kernels (3.4.67, 3.5.7, 3.8.13 for the official rtai-4.0), so we need to build a kernel supported by both Rtai and Ethercat master.

The only common kernel here is **3.4.67**. 

> Issues :
* The PC used for realtime on the meka is equipped with an i7-3770S Sandy Bridge, with integrated graphics, which are NOT supported on 3.4. All the computation is software, i.e slow and
