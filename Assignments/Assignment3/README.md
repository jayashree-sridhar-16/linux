# CMPE283 Assignment 3

## Team Members

* **Jayashree Sridhar (014608581)**
* **Praneetha Sripada (014353664)**

### Q1 - Contribution by each member
* **Jayashree Sridhar:** 
- Setup the outer and inner VM
- Made changes in VMX code for incrementing exit reason for each VM exit for corresponding VM exit reason
- Added conditional blocks in CPUID leaf 0x4FFFFFFE for valid and enabled VM exit reasons
- Added conditional blocks in CPUID leaf 0x4FFFFFFE for exit reasons not enabled by kvm
- Wrote test code for running inside inner VM which invokes the CPUID leaf function

* **Praneetha Sripada:**  
- Setup the outer and inner VM
- Made changes in CPUID code for adding leaf function for 0x4FFFFFFE
- Added conditional blocks in CPUID leaf 0x4FFFFFFE for exit reasons not defined in SDM
- Added conditional blocks in CPUID leaf 0x4FFFFFFE for exit reasons not enabled by kvm
- Checked and analyzed exit counts for vm exits from test code output

## Files modified in linux repository for the assignment
- https://github.com/jayashree-sridhar-16/linux/blob/master/arch/x86/kvm/cpuid.c
- https://github.com/jayashree-sridhar-16/linux/blob/master/arch/x86/kvm/vmx/vmx.c

### Q2 - Steps
- Please refer to the setup in https://github.com/jayashree-sridhar-16/linux/tree/master/Assignments/Assignment2#steps

### CPU leaf 0x4FFFFFFE code implementation:

#### Modifying arch/x86/kvm/cpuid.c:
1. Declared and initialized atomic array for storing VM exit reasons which will track number of exits per reason. These are exported so that they can be modified in other files. The array is declared with size 69, to accommodate exit reasons from number 0 to 68, with each exit number corresponding to the array index. Each array index will store the number of times the vm exit has occured for that exit reason.

```
atomic_t exitReasonArray[69] = {ATOMIC_INIT(0)};
EXPORT_SYMBOL(exitReasonArray);
```
2. Added a new cpuid leaf function eax=0x4FFFFFFE inside *kvm_emulate_cpuid* method. On matching the condition eax=0x4FFFFFFFE, it will check for exit reason number in **ecx** as follows:
```
if (eax == 0x4FFFFFFE) { //Assignment 3
		u32 numOfExits = atomic_read(&exitReasonArray[ecx]);
		printk("Assignment 3: Total Exits = %u for exit reason= %u\n", numOfExits, ecx);

		if (ecx == 35 || ecx == 38 || ecx == 42 || ecx == 65 || ecx > 68 || ecx < 0) {
			// Values not defined in SDM

			kvm_rax_write(vcpu, 0);
			kvm_rbx_write(vcpu, 0);
			kvm_rcx_write(vcpu, 0);
			kvm_rdx_write(vcpu, 0xFFFFFFFF);

		} else if (ecx == 3 || ecx == 4 || ecx == 5 || ecx == 6 || ecx == 11 || ecx == 16 || ecx == 17 
			|| ecx == 33 || ecx == 34 || ecx == 51 || ecx == 63 || ecx == 64 
			|| ecx == 66 || ecx == 67 || ecx == 68) {

			// Exits not enabled by kvm
			kvm_rax_write(vcpu, 0);
			kvm_rbx_write(vcpu, 0);
			kvm_rcx_write(vcpu, 0);
			kvm_rdx_write(vcpu, 0);

		} else {
			// Exits defined in SDM and enabled in the kvm

			kvm_rax_write(vcpu, numOfExits);
			kvm_rbx_write(vcpu, 0);
			kvm_rcx_write(vcpu, 0);
			kvm_rdx_write(vcpu, 0);
		}
```
3. Check if ecx value matches any exit reasons not defined in SDM. If yes, we write **value 0 to registers eax, ebx, ecx and value 0xFFFFFFFF** to edx.
4. Else check if ecx value matches any exit reasons not enabled in kvm. If yes, we write **value 0 to all the registers eax, ebx, ecx and edx**
5. Else we identify ecx value corresponds to exits defined in SDM and enabled in kvm. Then we write the **value corresponding to number of exits for exit reason in ecx to eax and value of 0 to registers ebx,ecx and edx.**

> From Intel SDM Vol 3D, Appendix C VMX Basic Exit reasons, we found that there is no exit reason defined for numbers 35, 38, 42 and 65
> For finding which exit control are not enabled by kvm, we refer to the array of exit handlers defined in vmx.c as mentioned below
> Exit reasons not present in this array have not been enabled by the kvm 
> The exit reason not enabled in the kvm were 3, 4, 5, 6, 11, 16, 17, 33, 34, 51, 63, 64, 66, 67 and 68
```
[EXIT_REASON_EXCEPTION_NMI]           = handle_exception_nmi, // Exit reason 0
[EXIT_REASON_EXTERNAL_INTERRUPT]      = handle_external_interrupt, // Exit reason 1
[EXIT_REASON_TRIPLE_FAULT]            = handle_triple_fault, // Exit reason 2
[EXIT_REASON_NMI_WINDOW]	      = handle_nmi_window, // Exit reason 8
[EXIT_REASON_IO_INSTRUCTION]          = handle_io, // Exit reason 30
[EXIT_REASON_CR_ACCESS]               = handle_cr, // Exit reason 28
[EXIT_REASON_DR_ACCESS]               = handle_dr, // Exit reason 29
[EXIT_REASON_CPUID]                   = kvm_emulate_cpuid, // Exit reason 10
[EXIT_REASON_MSR_READ]                = kvm_emulate_rdmsr, // Exit reason 31
[EXIT_REASON_MSR_WRITE]               = kvm_emulate_wrmsr, // Exit reason 32
[EXIT_REASON_INTERRUPT_WINDOW]        = handle_interrupt_window, // Exit reason 7
[EXIT_REASON_HLT]                     = kvm_emulate_halt, // Exit reason 12
[EXIT_REASON_INVD]		      = handle_invd, // Exit reason 13
[EXIT_REASON_INVLPG]		      = handle_invlpg, // Exit reason 14
[EXIT_REASON_RDPMC]                   = handle_rdpmc, // Exit reason 15
[EXIT_REASON_VMCALL]                  = handle_vmcall, // Exit reason 18
[EXIT_REASON_VMCLEAR]		      = handle_vmx_instruction, // Exit reason 19
[EXIT_REASON_VMLAUNCH]		      = handle_vmx_instruction, // Exit reason 20
[EXIT_REASON_VMPTRLD]		      = handle_vmx_instruction, // Exit reason 21
[EXIT_REASON_VMPTRST]		      = handle_vmx_instruction, // Exit reason 22
[EXIT_REASON_VMREAD]		      = handle_vmx_instruction, // Exit reason 23
[EXIT_REASON_VMRESUME]		      = handle_vmx_instruction, // Exit reason 24
[EXIT_REASON_VMWRITE]		      = handle_vmx_instruction, // Exit reason 25
[EXIT_REASON_VMOFF]		      = handle_vmx_instruction, // Exit reason 26
[EXIT_REASON_VMON]		      = handle_vmx_instruction, // Exit reason 27
[EXIT_REASON_TPR_BELOW_THRESHOLD]     = handle_tpr_below_threshold, // Exit reason 43
[EXIT_REASON_APIC_ACCESS]             = handle_apic_access, // Exit reason 44
[EXIT_REASON_APIC_WRITE]              = handle_apic_write, // Exit reason 56
[EXIT_REASON_EOI_INDUCED]             = handle_apic_eoi_induced, // Exit reason 45
[EXIT_REASON_WBINVD]                  = handle_wbinvd, // Exit reason 54
[EXIT_REASON_XSETBV]                  = handle_xsetbv, // Exit reason 55
[EXIT_REASON_TASK_SWITCH]             = handle_task_switch, // Exit reason 9
[EXIT_REASON_MCE_DURING_VMENTRY]      = handle_machine_check, // Exit reason 41
[EXIT_REASON_GDTR_IDTR]		      = handle_desc, // Exit reason 46
[EXIT_REASON_LDTR_TR]		      = handle_desc, // Exit reason 47
[EXIT_REASON_EPT_VIOLATION]	      = handle_ept_violation, // Exit reason 48
[EXIT_REASON_EPT_MISCONFIG]           = handle_ept_misconfig, // Exit reason 49
[EXIT_REASON_PAUSE_INSTRUCTION]       = handle_pause, // Exit reason 40
[EXIT_REASON_MWAIT_INSTRUCTION]	      = handle_mwait, // Exit reason 36
[EXIT_REASON_MONITOR_TRAP_FLAG]       = handle_monitor_trap, // Exit reason 37
[EXIT_REASON_MONITOR_INSTRUCTION]     = handle_monitor, // Exit reason 39
[EXIT_REASON_INVEPT]                  = handle_vmx_instruction, // Exit reason 50
[EXIT_REASON_INVVPID]                 = handle_vmx_instruction, // Exit reason 53
[EXIT_REASON_RDRAND]                  = handle_invalid_op, // Exit reason 57
[EXIT_REASON_RDSEED]                  = handle_invalid_op, // Exit reason 61
[EXIT_REASON_PML_FULL]		      = handle_pml_full, // Exit reason 62
[EXIT_REASON_INVPCID]                 = handle_invpcid, // Exit reason 58
[EXIT_REASON_VMFUNC]		      = handle_vmx_instruction, // Exit reason 59
[EXIT_REASON_PREEMPTION_TIMER]	      = handle_preemption_timer, // Exit reason 52
[EXIT_REASON_ENCLS]		      = handle_encls, // Exit reason 60
```

#### Modifying arch/x86/kvm/vmx/vmx.c:
1. Declare extern variable **exitReasonArray[]**
```
extern atomic_t exitReasonArray[69];
```
2. Increment the *ith* element of *exitReasonArray* every time vmx_handle_exit has been invoked for exit reason number *i*.
```
atomic_fetch_add(1, &exitReasonArray[exit_reason]);
```

#### Next steps:
1. After we save our changes, we run same make commands as before, to build the kernel.
2. Once we have successfully built the kernel and rebooted our outer vm, we start up the inner vm and run our test code inside it.
We invoke cpuid instruction in a loop by iterating over exit reason values 0 to 68.
```
#include <stdio.h>
#include <limits.h>

static inline void native_cpuid(unsigned int* eax, unsigned int* ebx, unsigned int* ecx, unsigned int* edx) {
	asm volatile("cpuid"
					:"=a" (*eax),
					 "=b" (*ebx),
					 "=c" (*ecx),
					 "=d" (*edx)
					: "0" (*eax), "2" (*ecx)
				);
}

int main(int argc, char **argv) {
	unsigned int eax, ebx, ecx, edx;
	for (int i=0; i < 69; i++) {
		eax=0x4FFFFFFE;
		ecx=i;
		ebx = 0;
		edx = 0;
		native_cpuid(&eax, &ebx, &ecx, &edx);
		unsigned long d = ( unsigned long) edx;
		printf("====================================\n");
		printf("Total no of exits: %u for exit number %u\n", eax, i);
		printf("Register values: %u, %u, %u, 0x%lx\n", eax, ebx, ecx, d);
	}
	

	return 0;
}
```
We notice the expected output in registers eax, ebx, ecx, and edx for the use cases: valid and enabled exit controls, undefined exit values and defind but not enabled exit controls.  


## Questions/Answers/Observations
### Comment on the frequency of exits – does the number of exits increase at a stable rate? Or are there more exits performed during certain VM operations? Approximately how many exits does a full VM boot entail?
We tested the count of VM exits for 3 scenarios - after inner VM boot up, opening files and editors in inner VM, and opening browser and playing videos in Inner VM. We noticed the below counts for some of the VM exit reasons
| VM Exit reason | After inner VM boot up | During file open, read and editing operations | After playing video in browser | 
| :----: | :----:| :----: |:----:|
|0|16896 | 16896 | 16896 | 
|1|58509 | 186426 | 449454 | 
|7|8823 | 20760 | 69124 | 
|10|54837 | 56108 | 56233 |
|12|17169 | 54210 | 176332 |
|28|775415 | 775415 | 775415 |
|29|4 | 4 | 4 |
|30|315638 | 319736 | 322724 |
|31|795 | 1148 | 1665 |
|32|75136 | 217585 | 741167 |
|40|2328 | 7780 | 23191 |
|48|6505288 | 6768126 | 6859193 |
|49|47321 | 64345 | 178878 |


- From the above we notice there is gradual increments in the count of vm exits for most like External interrupt, Interrupt window, CPUID and HLT. 
- There are some exits which remain constant after boot up like exception based, CR and DR accesses.
- There is a spike in counts noticed in 3rd scenario for exit reason 32 and 49. 
- Hence the frequency of exits varies for different exit reasons and according to different operations performed in inner VM.



### Of the exit types defined in the SDM, which are the most frequent? Least?
- Exit reasons 48 EPT violation had the highest frequency of VM exits (6505288), followed by exit reasons 1, 28, 30, 32, 49.
- Exit reasons such as 29, 31, 40, 54, had very low number of exits and some other had no vm exit occuring at all.
