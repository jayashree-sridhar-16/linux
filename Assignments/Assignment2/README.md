# Assignment 2

## Team members
- Jayashree Sridhar
- Praneetha Sripada

## Members contributions
#### Jayashree Sridhar
- Setup outer VM and linux source code in outer VM.
- Researched solutions for the issues in running make modules-install and fixing rebooting of Ubuntu after the kernel module was built.

### Praneetha Sripada
- System setup and configuration of inner VM and outer VM.
- Fixed issues encountered during kernel build and inner VM setup.
- Implemented code for determining number of exits and worked collaboratively for implementing other code functionality required for this assignment. 



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

## Modifying cpuid.c:
1. To count no.of exits and number of cycles, we are defining atomic variables **number_of_exits** and **number_of_cycles** in cpuid.c.
2. These variables are initialized to 0.
3. We check for the cupid leaf function and if it is leaf function i.e.,eax=0x4FFFFFFFF, we read the values as number of exits and cycles and write these values to registers **rax, rbx and rcx.** (for number of cycles, we store low 32 bits in rcx and high 32 bits in rbx respectively) while rdx is set to 0.

## Modifying vmx.c:
1. We export the variables no.of cycles and no.of exits declared in cpuid.c to vmx.c
2. We define a function **get_RDTSC_value** to read the value of time stamp using **rdtsc** instruction and store it in msr.
3. Whenever vm exits, we go into function **vmx_handle_exit**, where we declare variables to store the **start time, end time and total time.** At the same time,    we keep adding to no.of exits.
4. We read the rdtsc time stamp value into **start time.**
5. Whenever there is a return statement, we are putting it in **return_value** and calculating end time, by reading rdtsc time stamp at that point.
6. We proceed by calculating **total time (end time-start time)** and keep adding this value to **number of cycles.**


## Next steps:
1. After we save our changes, we run same commands as before, to build the kernel.
2. Once we have successfully built the kernel and rebooted our outer vm, we start up the inner vm and run our test code inside it.
3. We observe the output file displays values for no.of exits and no.of cycles.





