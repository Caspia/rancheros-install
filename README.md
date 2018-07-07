# Rancheros install for VirtualBox

## Overview

This repository contains details and prebuilt artifacts for an installation of [RancherOS](https://rancher.com/rancher-os/) as a virtual machine under VirtualBox.
The motivation for these instructions are to support a minimal docker install on limited performance laptops typically used in offline prison classes such as those by [University Beyond Bars](http://www.universitybeyondbars.org/) but others may also find this useful.

## Introduction to RancherOS

RancherOS is designed as a minimal footprint operating system that supports running Docker containers.
To accomplish this, the design is very unusual, and normal expectations of how to initially install and configure a Linux system do not apply.
The available documentation on how to initially install RancherOS is confusing and incomplete.
Hence these instructions attempt to give complete details of how to install and configure RancherOS in this environment.
Where appropriate, the results of applying those instructions are also given in forms that can be used directly, such as virtual machine images. 


## Installation environment

These instructions envision two systems.
One is a Windows workstation with internet access, called the *online system*, used to setup and initially configure artifacts.
The second is a Windows workstation that in the final use is without internet access, called the *offline system*, located inside a controlled environment such as a prison.
This offline system requires some method of update, like prepared files may be brought and installed using for example a DVD or flashdrive.
Also, in some cases (such as when populating an NPM cache) the "offline system" may be installed with internet access.

## Prerequisites
Both the offline and online systems need to have a minimal suite of programs installed prior to applying any instructions here, described below.
### Required
* Install [Git](https://git-scm.com/download/win) version control system, mostly needed for the bash prompt it provides.
* Install **VirtualBox platform package for windows hosts** from [https://www.virtualbox.org/wiki/Downloads], for example [https://download.virtualbox.org/virtualbox/5.2.12/VirtualBox-5.2.12-122591-Win.exe]
### Optional
* Download **VirtualBox Oracle VM VirtualBox Extension Pack** from [https://www.virtualbox.org/wiki/Downloads], that matches the version of VirtualBox installed, and install (double-click downloaded file to open for installation within VirtualBox). For example: [https://download.virtualbox.org/virtualbox/5.2.12/Oracle_VM_VirtualBox_Extension_Pack-5.2.12.vbox-extpack]
* Download and install the [putty ssh client](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html), for example [https://the.earth.li/~sgtatham/putty/latest/w64/putty-64bit-0.70-installer.msi]
## Installation
See the instructions under base-system/base.md for instructions on how to install RancherOS from original sources. But in the release are of this repo is a .ova file which may be imported directly into a VirtualBox installation. That is the recommended way to get started. Note that, when used online, this RancherOS requires two ethernet econnections. One is NAT for internet access, the second is host-only for control of the system using ssh.
## TO BE CONTINUED
