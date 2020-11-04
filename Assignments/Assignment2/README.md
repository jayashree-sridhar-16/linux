# Assignment 2

## Team members
- Jayashree Sridhar
- Praneetha Sripada

## Members contributions
#### Jayashree Sridhar
- Setup outer VM and linux source code in outer VM.
- Researched solutions for the issues in 1 - 3 mentioned below in setup instructions.
- Researched on measuring CPU cycles using RDTSC instruction.
- Created test code in inner vm for testing the kernel code changes implemented for CPUID leaf function 0x4FFFFFFF.
- Worked on calculating number of CPU cycles spent in VM exits.


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
> Issue 1:
> In the assignment description, the last make command was wrongly given as modules-install.
> The command used to fail with no rule to make target 'modules-install'
> Section 2.3 in https://www.kernel.org/doc/Documentation/kbuild/modules.txt helped me figure out the correct target as modules_install

5. Reboot the VM 
```
reboot
```
> Issue 2: 
> The VM was got stuck in initramf BusyBox with error as mentioned in https://unix.stackexchange.com/questions/589899/missing-modules-cat-proc-modules-ls-dev-and-uuid-doesnt-exist-in-busybox  
> How it was solved:
> - Shutdown the VM
> - Start the VM
> - Enter GRUB menu
> - Choose the linux kernel version 5.4.0-52-generic (the installed version)
> - It boots successfully finally.    
> For reasons unclear, after the first build and reboot, loading 5.9.0 version was not booting, but for subsequent compile, build and reboot, the kernel was able to successfully boot kernel version 5.9.0 without any intervention.


### Inner VM Setup
1. Downloaded the ISO image of Ubuntu 20.04.1 from https://ubuntu.com/download/desktop
2. Download virt-manager for managing virtual machines on the outer VM. (See https://virt-manager.org/ )
3. Download QEMU/KVM for running VMs and start. Refer https://www.tecmint.com/install-kvm-on-ubuntu/
```
sudo apt-get install virt-manager
sudo apt install -y qemu qemu-kvm libvirt-daemon libvirt-clients bridge-utils virt-manager
sudo systemctl enable --now libvirtd
```
4. Open virt-manager from command line or Ubuntu applications. Refer https://www.tecmint.com/install-kvm-on-ubuntu/ for steps.
> Use the downloaded Ubuntu ISO for creating the VM
5. Start and login into Ubuntu inner VM.
> Issue 3:
> - After successful launch of inner VM, it operated properly, but was crashing whenever we come out of inner VM and access host or outer VM application. It seemed to be some improper Ubuntu software installation issue. After installing the software properly, VM stopped crashing.
> - After the crashing issue was resolved, the inner VM did not have internet connection working properly. Changing NIC setting for the inner VM from 'NAT' to Host device and restarting the inner VM after applying the changes resolved the issue. 


### CPU leaf 0x4FFFFFFF code implementation:



## Questions/Answers/Observations

#### Comment on the frequency of exits – does the number of exits increase at a stable rate? Or are there more exits performed during certain VM operations?
- The number of exits does not increase at a steady rate. It varies depending on the VM operations performed. 

#### Approximately how many exits does a full VM boot entail?
- The observed number of VM exits during booting of inner VM was 4119437. A printk statement was added inside vmx.c to print the exit reason number and number of exits occured, every time a VM exit occuers.

```
printk("Assignment 2: Exit Reason = %u, Total exits = %u\n", exit_reason, atomic_read(&number_of_exits));
```
number_of_exits is set to 0 during outer VM kernel load. It gets modified after starting a VM inside the outer VM.
Once the inner VM has fully booted till displaying Ubuntu home screen, we execute below command in outer VM terminal and find the 4119437 number of exits have occured during inner VM boot.
```
dmesg
```




