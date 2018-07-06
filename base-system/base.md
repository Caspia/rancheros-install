# Basic rancherOS install for VirtualBox

This directory contains details and prebuilt artifacts for a basic installation of [rancherOS](https://rancher.com/rancher-os/) as a virtual machine under VirtualBox.
These instructions are applied to an online Windows system with Github and VirtualBox installed, as described in the main repository README.md file.
The goal here is to create an exportable virtual machine image, along with necessary support files, than can be directly imported into a VirtualBox installation on an offline system.

Note that the security settings that may be recommended here assume that the ultimate system is run isolated in a single machine, and may be inappropriate for a rancherOS system exposed to the internet.

## Installation of virtual machine
* Download file **rancherOS.iso** from [https://github.com/rancher/os/releases/latest]
* Create a rancherOS guest on VirtualBox: Run VirtualBox and
  * click **New**, and set parameter values as follows:  
  -- Name: rancherOS  
  -- Type: Linux  
  -- Version: Other Linux (64-bit)  
  -- Memory Size: 2048 MB  
  -- (Do not add a virtual hard disk)
  * Select the new guest machine **rancherOS**, click **Settings**, and set or modify values as follows:  
  -- System/Processor/Processor(s): 2 cpus  
  -- Storage/Storage devices: Delete the IDE controller.  
  -- Storage/Storage devices: Add a SCSI controller.  
  -- Storage/Storage devices: Add an optical drive, when it asks for disk select rancherOS.iso downloaded previously (may be called rancherOS_1_4_0.iso)  
  -- Storage/Storage devices: Add a hard disk. Select **Create new disk**, choose defaults except name: rancherOS, size: 100 GB.  
  -- Network/Adapter 2. Enable, set **Attached to** as **Host-only Adapter** (Leave Adapter 1 at default settings, enabled on a NAT network.)

* Create ssh keys for login: Run git-bash, and enter the following:  
   ```bash
   ssh-keygen -t rsa -b 4096 -N '' -C rancherOS -f ~/.ssh/rancherOS
   ```

* Create cloud-config.yml file including the public ssh key: Copy and paste this script into git-bash and execute.
(Typically all but the last line are executed immediately, then press ENTER to run the last command):  
   ```bash
   cat > cloud-config.yml << EOF
     #cloud-config
   ssh_authorized_keys:
   EOF
   echo -n '  - ' >> cloud-config.xml
   cat ~/.ssh/rancherOS.pub >> cloud-config.yml
   ```

* Start rancherOS, and configure to allow password ssh:  
-- Run **VirtualBox manager**, select the previously created rancherOS guest VM, and click **Run**  
-- When rancherOS starts, it shows the ip addresses of the interfaces. Save the ip address of interface eth1 for later use.  
-- Within the bash prompt for the rancherOS system, enter the following to set a password rancher for user rancher:  
  ```bash
  echo "rancher:rancher" | sudo chpasswd
  ```  

* Copy the created cloud-config.yml file to the rancherOS system:  
  -- Enter the following command in git-bash.
  If prompted to confirm the server identity, say yes.
  When prompted for the password, enter (without quotes) "rancher".
  The example shows the ip address 192.168.56.101, instead use the ip address determined above for eth1.
  ```bash
  scp cloud-config.yml rancher@192.168.56.101:
  ```
  -- Enter the following command in git-bash.
  The example shows the ip address 192.168.56.101, instead use the actual ip address determined above for eth1:  
  ```bash
  ssh-keygen -R 192.168.56.101
  ```

* Install rancherOS to disk.  
  -- In **VirtualBox Manager**, select the rancherOS guest system (which should be called rancherOS), select **Storage**, select the optical drive, then click on the DVD icon and select **Remove disk from virtual drive**  
  -- In the bash prompt for rancherOS, enter the following command:  
  ```bash
  sudo ros install -c cloud-config.yml -d /dev/sda
  ```  
  You will be prompted if you want to continue. Type **y**  
  You will be prompted if you should continue with reboot. Type **y**.

* Test the new installation. Run the following in a git-bash prompt. The example shows the ip address 192.168.56.101, instead use the actual ip address determined for eth1:  
  ```bash
  ssh -i ~/.ssh/rancherOS rancher@192.168.56.101
  ```
  You should be able to connect to the rancherOS with no password, and get the prompt:
  ```bash
  [rancher@rancher ~]$ 
  ```
  In rancherOS, run the test hello-world docker container:
  ```bash
  docker run hello-world
  ```
  In rancherOS, confirm that the hello-world image was downloaded:
  ```
  docker images
  ```
  
## Post-installation
After installation, you may want to modify the rancherOS configuration. Here are instructions.
### Change to static ip address
Connect to rancherOS from a bash prompt (see "test the new installation above) and enter the following to change the ip address of the host interface to 192.168.56.10:
  ```bash
  sudo ros config set rancher.network.interfaces.eth0.dhcp true
  sudo ros config set rancher.network.interfaces.eth1.dhcp false
  sudo ros config set rancher.network.interfaces.eth1.address 192.168.56.10/24
  ```
### Restart
After configuration changes are done, enter to reboot (from rancherOS):
  ```bash
  sudo su reboot
  ```
### Troubleshooting
* Occasionally when modifying configurations, there are problems getting the new ip address to appear or connect with ssh. It may help to stop rancherOS, stop the VirtualBox manager, and try again.

* If you get an error message from ssh like this:  
  ```bash
  The authenticity of host '192.168.56.10 (192.168.56.10)' can't be established.
  ```  
  then the fix is to remove the ssh signature of that address:  
  ```bash
  ssh-keygen -R 192.168.56.10
  ```