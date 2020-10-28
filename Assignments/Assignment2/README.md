# Assignment 2

## Team members
- Jayashree Sridhar
- Praneetha Sripada

## Members contributions
#### Jayashree Sridhar
- Setup outer VM and linux source code in outer VM.
- Researched solutions for the issues in running make modules-install and fixing rebooting of Ubuntu after the kernel module was built.


## Steps
### Host machine specs
- OS: MacOS Catalina 10.15.7
- Processor: 1.4 GHz Quad-Core Intel Core i5
- RAM: 8 GB

### Step up of outer VM
1. Downloaded and installed VMWare Fusion Player version 12 from https://my.vmware.com/web/vmware/evalcenter?p=fusion-player-personal 
2. Downloaded the ISO image of Ubuntu 20.04.1 from https://ubuntu.com/download/desktop
3. Followed Linux Easy Install steps as mentioned in https://docs.vmware.com/en/VMware-Fusion/12/com.vmware.fusion.using.doc/GUID-E9883D0F-875C-48C6-8EA4-FCEFB5254625.html
4. Configured VM with 4 CPUs and 200GB memory.
5. Enabled nested virtualization capability for VM.
6. Logged into the VM and checked if *vmx* is present among CPU capabilities by executing *cat /proc/cpuinfo | more*
7. Install necessary tools using apt install for make, gcc and git
8. Clone this git repository.
9. Execute *cd linux*

### Building the linux kernal
1. Enter the root mode 
```
sudo bash
```
2. Prepare the compile environment by installing these tools
```
apt-get install build-essential kernel-package fakeroot libncurses5-dev libssl-dev ccache bison flex libelf-dev
```
3. Setting configurations
```
uname -a
```
This returns the version 5.4.0-52-generic in this system. Use this version in next command.
```
cp /boot/config-4.15.0-112-generic ./.config
make oldconfig
```
4. Build the kernal using the commands
```
make && make modules && make install && make modules_install
```
> Issues faced in this step:
> In the assignment description, the last make command was wrongly given as modules-install.
> The command used to fail with no rule to make target 'modules-install'
> Section 2.3 in https://www.kernel.org/doc/Documentation/kbuild/modules.txt helped me figure out the correct target as modules_install

5. Reboot the VM 
```
reboot
```
> Issues faced during reboot: 
> The VM was got stuck in initramf BusyBox with error as mentioned in https://unix.stackexchange.com/questions/589899/missing-modules-cat-proc-modules-ls-dev-and-uuid-doesnt-exist-in-busybox  
> How it was solved:
> - Shutdown the VM
> - Start the VM
> - Enter GRUB menu
> - Choose the linux kernel version 5.4.0-52-generic (the installed version)
> - It boots successfully finally  
> The issue seemed to be that it was selecting another kernel version 5.9.0 while trying to boot.




