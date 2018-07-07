# Basic rancherOS install for VirtualBox

This directory contains details and prebuilt artifacts for a basic installation of [rancherOS](https://rancher.com/rancher-os/) as a virtual machine under VirtualBox.
These instructions are applied to an online Windows system with Github and VirtualBox installed, as described in the main repository README.md file.
The goal here is to create an exportable virtual machine image, along with necessary support files, than can be directly imported into a VirtualBox installation on an offline system.

Note that the security settings that may be recommended here assume that the ultimate system is run isolated in a single machine, and may be inappropriate for a rancherOS system exposed to the internet.
In particular, for the sample install, the public and private keys for that are publically available here.
If you installed the sample system on a machine with public internet access, anyone in the world could use the private key here to login to your system, with sudo to root privileges.

## Description of files in base-system directory

### base.md
This file contains the instructions to create a rancherOS install. If you want to install
rancherOS from scratch, follow the instructions here. All other files in this directory are
files that are generated during the installation from scratch, and are included here
in case you want to skip the installation from scratch, and just start using a rancherOS
virtual machine on your system.

### cloud-config.yml

To install rancherOS to disk, you need a configuration file that includes a public key.
This file is an appropriately configured file that uses the keys in this directory.

### rancheros

The private key file.

### rancheros.pub

The public key file.

### rancheros.ppk

The private key file, in the format expected by the putty program on Windows.

### rancheros.ova

(This file is not in the github repo, but only in the github release).
Contains an exported virtual machine suitable to import directly into VirtualBox, created using the instructions here.
Its use is described in the main README.md from the parent directory.

## Installation of virtual machine

Below are instructions to install rancherOS as a VirtualBox Linux install running with a Windows host system.

* (Prerequisites)  
  On the Windows system install VirtualBox and Git for Windows, as described in the README.md file in the parent directory.

* Download file **rancherOS.iso** from [https://github.com/rancher/os/releases/latest]
* Create a rancheros guest on VirtualBox: Run VirtualBox and
  * click **New**, and set parameter values as follows:  
  -- Name: rancheros  
  -- Type: Linux  
  -- Version: Other Linux (64-bit)  
  -- Memory Size: 2048 MB  
  -- (Do not add a virtual hard disk)
  * Select the new guest machine **rancheros**, click **Settings**, and set or modify values as follows:  
  -- System/Processor/Processor(s): 2 cpus  
  -- Storage/Storage devices: Delete the IDE controller.  
  -- Storage/Storage devices: Add a SCSI controller.  
  -- Storage/Storage devices: Add an optical drive, when it asks for disk select rancherOS.iso downloaded previously (may be called rancherOS_1_4_0.iso)  
  -- Storage/Storage devices: Add a hard disk. Select **Create new disk**, choose defaults except name: rancheros, size: 100 GB.  
  -- Network/Adapter 2. Enable, set **Attached to** as **Host-only Adapter** (Leave Adapter 1 at default settings, enabled on a NAT network.)

* Create ssh keys for login: Run git-bash, and enter the following:  
   ```bash
   ssh-keygen -t rsa -b 4096 -N '' -C rancheros -f ~/.ssh/rancheros
   ```
This will create files rancheros (with the private key) and rancheros.ppk (with the public key) in the directory ~/.ssh (This is identical to the windows directory c:\Users\(Your Name)\.ssh
* Create cloud-config.yml file including the public ssh key: Copy and paste this script into git-bash and execute.
(Typically all but the last line are executed immediately, then press ENTER to run the last command):  
   ```bash
   cat > cloud-config.yml << EOF
     #cloud-config
   ssh_authorized_keys:
   EOF
   echo -n '  - ' >> cloud-config.xml
   cat ~/.ssh/rancheros.pub >> cloud-config.yml
   ```

* Start rancherOS, and configure to allow password ssh:  
-- Run **VirtualBox manager**, select the previously created rancheros guest VM, and click **Run**  
-- When rancheros starts, it shows the ip addresses of the interfaces. Save the ip address of interface eth1 for later use.  
-- Within the bash prompt for the rancheros system, enter the following to set a password rancher for user rancher:  
  ```bash
  echo "rancher:rancher" | sudo chpasswd
  ```  

* Copy the created cloud-config.yml file to the rancheros system:  
  -- Enter the following command in git-bash.
  If prompted to confirm the server identity, say yes.
  When prompted for the password, enter (without quotes) "rancher".
  The example shows the ip address 192.168.56.101, instead use the ip address determined above for eth1.
  ```bash
  scp cloud-config.yml rancher@192.168.56.101:
  ```
  -- Enter the following command in git-bash.
  (This removes the saved machine key from ssh, which if left will cause issues later.)
  The example shows the ip address 192.168.56.101, instead use the actual ip address determined above for eth1:  
  ```bash
  ssh-keygen -R 192.168.56.101
  ```

* Install rancheros to disk.  
  -- In **VirtualBox Manager**, select the rancherOS guest system (which should be called rancheros), select **Storage**, select the optical drive, then click on the DVD icon and select **Remove disk from virtual drive**  
  -- In the bash prompt for rancheros, enter the following command:  
  ```bash
  sudo ros install -c cloud-config.yml -d /dev/sda
  ```  
  You will be prompted if you want to continue. Type **y**  
  You will be prompted if you should continue with reboot. Type **y**.

* Test the new installation. Run the following in a git-bash prompt on Windows.
The example shows the ip address 192.168.56.101, instead use the actual ip address determined for eth1:  
  ```bash
  ssh -i ~/.ssh/rancheros rancher@192.168.56.101
  ```
  You should be able to connect to the rancheros with no password, and get the prompt:
  ```bash
  [rancher@rancher ~]$ 
  ```
  In rancheros, run the test hello-world docker container:
  ```bash
  docker run hello-world
  ```
  In rancheros, confirm that the hello-world image was downloaded:
  ```
  docker images
  ```
  (This assumes that the rancheros system has internet access.)
## Post-installation
After installation, you may want to modify the rancheros configuration. Here are instructions.
### Change to static ip address
Connect to rancheros from a bash prompt (see "test the new installation above) and enter the following to change the ip address of the host interface to 192.168.56.10 (choose another address if appropriate.):
  ```bash
  sudo ros config set rancher.network.interfaces.eth0.dhcp true
  sudo ros config set rancher.network.interfaces.eth1.dhcp false
  sudo ros config set rancher.network.interfaces.eth1.address 192.168.56.10/24
  ```
### Restart
After configuration changes are done, enter to reboot (from rancheros):
  ```bash
  sudo su reboot
  ```
### Troubleshooting
* Occasionally when modifying configurations, there are problems getting the new ip address to appear or connect with ssh. It may help to stop rancheros, stop the VirtualBox manager, and try again.

* If you get an error message from ssh like this:  
  ```bash
  The authenticity of host '192.168.56.10 (192.168.56.10)' can't be established.
  ```  
  then the fix is to remove the ssh signature of that address:  
  ```bash
  ssh-keygen -R 192.168.56.10
  ```