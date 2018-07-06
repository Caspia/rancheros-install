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

These instructions envision two systems. One is a Windows workstation with internet access, called the *online system*, used to setup and initially configure artifacts. The second is a Windows workstation without internet access, called the *offline system*, located inside a controlled environment such as a prison, but to which prepared files may be brought and installed using for example a DVD or flashdrive.

## Prerequisites
Both the offline and online systems need to have a minimal suite of programs installed prior to applying any instructions here, described below.
### Required
* Install [Git](https://git-scm.com/download/win) version control system, mostly needed for the bash prompt it provides.
* Install **VirtualBox platform package for windows hosts** from [https://www.virtualbox.org/wiki/Downloads], for example [https://download.virtualbox.org/virtualbox/5.2.12/VirtualBox-5.2.12-122591-Win.exe]
### Optional
* Download **VirtualBox Oracle VM VirtualBox Extension Pack** from [https://www.virtualbox.org/wiki/Downloads], that matches the version of VirtualBox installed, and install (double-click downloaded file to open for installation within VirtualBox). For example: [https://download.virtualbox.org/virtualbox/5.2.12/Oracle_VM_VirtualBox_Extension_Pack-5.2.12.vbox-extpack]
* Download and install the [putty ssh client](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html), for example [https://the.earth.li/~sgtatham/putty/latest/w64/putty-64bit-0.70-installer.msi]
## Installation
### From the online system:
* Create a rancheros guest on VirtualBox: Run VirtualBox and
  * click **New**, and set parameter values as follows:  
  -- Name: dockeros  
  -- Type: Linux  
  -- Version: Other Linux (64-bit)  
  -- Memory Size: 2048 MB  
  -- (Do not add a virtual hard disk)
  * Select the new guest machine **dockeros**, click **Settings**, and set or modify values as follows:  
  -- System/Processor/Processor(s): 2 cpus  
  -- Storage/Storage devices: Delete the IDE controller.  
  -- Storage/Storage devices: Add a SCSI controller.  
  -- Storage/Storage devices: Add an optical drive, when it asks for disk select rancheros.iso downloaded in prerequistes (may be called rancheros_1_4_0.iso)  
  -- Storage/Storage devices: Add a hard disk. Select **Create new disk**, choose defaults except name: rancheros, size: 100 GB.  
  -- Network/Adapter 2. Enable, set **Attached to** as **Host-only Adapter** (Leave Adapter 1 at default settings, enabled on a NAT network.)

* Create ssh keys for login: Run git-bash, and enter the following:  
   ```bash
   ssh-keygen -t rsa -b 4096 -N '' -C rancheros -f ~/.ssh/rancheros
   ```

* Create cloud-config.yml file including the public ssh key: Copy and paste this script into git-bash and ENTER to run the last command:  
   ```bash
   cat > cloud-config.yml << EOF
     #cloud-config
   ssh_authorized_keys:
   EOF
   echo -n '  - ' >> cloud-config.xml
   cat ~/.ssh/rancheros.pub >> cloud-config.yml
   ```

* Start rancheros, and configure to allow password ssh:  
-- Run **VirtualBox manager**, select the previously created rancheros guest VM (which should be called dockeros), and click **Run**  
-- When rancheros starts, it shows the ip addresses of the interfaces. Save the ip address of interface eth1 for later use.  
-- Within the bash prompt for the rancheros system, enter the following to set a password rancher for user rancher:  
  ```bash
  echo "rancher:rancher" | sudo chpasswd
  ```  

* Copy the created cloud-config.yml file to the rancheros system:  
  -- Use the following command in git-bash. If prompted to confirm the server identity, say yes. When prompted for the password, enter (without quotes) "rancher".  The example shows the ip address 192.168.56.101, instead use the ip address determined above for eth1.
  ```bash
  scp cloud-config.yml rancher@192.168.56.101:
  ```
  -- Enter the following command in git-bash. The example shows the ip address 192.168.56.101, instead use the actual ip address determined above for eth1:  
  ```bash
  ssh-keygen -R 192.168.56.101
  ```

* Install the rancheros to disk.  
  -- In **VirtualBox Manager**, select the rancheros guest system (which should be called dockeros), select **Storage**, select the optical drive, then click on the DVD icon and select **Remove disk from virtual drive**  
  -- In the bash prompt for rancheros, enter the following command:  
  ```bash
  sudo ros install -c cloud-config.yml -d /dev/sda
  ```  
  You will be prompted if you want to continue. Type **y**  
  You will be prompted if you should continue with reboot. Type **y**.

* Test the new installation. Run the following in a git-bash prompt. The example shows the ip address 192.168.56.101, instead use the actual ip address determined for eth1:  
  ```bash
  ssh -i ~/.ssh/rancheros rancher@192.168.56.101
  ```
  You should be able to connect to the rancheros with no password.
