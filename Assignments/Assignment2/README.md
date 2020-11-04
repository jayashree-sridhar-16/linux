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

#### Modifying arch/x86/kvm/cpuid.c:
1. Declared and intialized atomic variables for tracking number of VM exits and number of CPU cycles spent in each VM exit. The variables are exported so that they can be modified by other files.
```
atomic_t number_of_exits = ATOMIC_INIT(0);
atomic_long_t number_of_cycles = ATOMIC_INIT(0);
EXPORT_SYMBOL(number_of_exits);
EXPORT_SYMBOL(number_of_cycles);
```
2. Added a check for the cpuid leaf function eax=0x4FFFFFFFF inside *kvm_emulate_cpuid* method. On matching the condition eax=0x4FFFFFFFF, it will store **number_of_exits** value into *eax*, higher and lower bits of **number_ of_cycles** into *ebx* and *ecx* respectively, and set *edx* to 0.

```
if (eax == 0x4FFFFFFF) {
		u32 total_exits = atomic_read(&number_of_exits);
		u64 cycles = atomic_long_read(&number_of_cycles);
		u32 max = 4294967295;
		u32 high_bits = cycles >> 32;
		u32 low_bits = cycles & max;

		printk("Assignment 2: Total Exits = %u\n", total_exits);
		printk("Assignment 2: Total Cycles = %llu\n", cycles);	

		kvm_rax_write(vcpu, atomic_read(&number_of_exits));
		kvm_rbx_write(vcpu, high_bits);
		kvm_rcx_write(vcpu, low_bits);
		kvm_rdx_write(vcpu, 0);
	}
```
> printk statement will print the message to kernel logs, which we can view by running *dmesg* command in terminal

#### Modifying arch/x86/kvm/vmx/vmx.c:
1. Declare extern variables for **number_of_exits** and **number_ of_cycles** defined in cpuid.c 
```
extern atomic_t number_of_exits;
extern atomic_long_t number_of_cycles;
```
2. Define a function for executing **rdtsc** instruction and return the current time stamp
```
static uint64_t get_RDTSC_value(void) {
	uint64_t msr;

	asm volatile ( "rdtsc\n\t"    // Returns the time in EDX:EAX.
	        "shl $32, %%rdx\n\t"  // Shift the upper bits left.
	        "or %%rdx, %0"        // 'Or' in the lower bits.
	        : "=a" (msr)
	        :
	        : "rdx");

	return msr;
}
```
> Reference: https://dmalcolm.fedorapeople.org/gcc/2015-08-31/rst-experiment/how-to-use-inline-assembly-language-in-c-code.html#volatile

3. Declare  **start_time, end_time and total_time** inside function *vmx_handle_exit* for recording time lapsed during VM exit handling.
4. Increment **number_of_exits** every time *vmx_handle_exit* has been invoked.
5. Calculation of time spent inside *vmx_handle_exit* is done as follows
  - Store **start_time** at the beginning of *vmx_handle_exit* and **end_time** before return statement.
  - Calculate the lapsed time by finding the delta between start_time and end_time
> Whenever there is a return statement, we are putting it in **return_value** and calculating end time, by reading rdtsc time stamp at that point.
```
unsigned long start_time= 0;
unsigned long end_time = 0;
unsigned long total_time = 0;

atomic_fetch_add(1, &number_of_exits); //Increment no of exits

start_time = get_RDTSC_value();

// VM exit handling

end_time = get_RDTSC_value();
total_time = end_time - start_time;
atomic_long_add(total_time, &number_of_cycles);
```


#### Next steps:
1. After we save our changes, we run same make commands as before, to build the kernel.
2. Once we have successfully built the kernel and rebooted our outer vm, we start up the inner vm and run our test code inside it.
```
#include <stdio.h>

static inline void native_cpuid(unsigned long* eax, unsigned long* ebx, unsigned long* ecx, unsigned long* edx) {
	asm volatile("cpuid"
					:"=a" (*eax),"=b" (*ebx),"=c" (*ecx),"=d" (*edx)
					: "0" (*eax)
				);
}


int main(int argc, char **argv) {
	unsigned long eax, ebc, ecx, edx;

	eax=0x4FFFFFFF;
	native_cpuid(&eax, &ebx, &ecx, &edx);
  unsigned long total_cycles = ((( unsigned long) ebx << 32 ) | ecx); 
	printf("Total no of exits: %lu\n", eax);
	printf("Total no of cycles: %lu\n", total_cycles);

	return 0;
}
```

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




