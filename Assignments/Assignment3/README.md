# CMPE283 Assignment 3

## Team Members

* **Jayashree Sridhar (014608581)**
* **Praneetha Sripada (014353664)**

### Q1 - Contribution by each member
* **Jayashree Sridhar:** 


* **Praneetha Sripada:**  


  

### Q2 - Steps taken for assignment
-All steps implemented in assignment 2 were followed for the environment setup.

### CPU leaf 0x4FFFFFFE code implementation:

#### Modifying arch/x86/kvm/cpuid.c:
1. Declared and initialized atomic array for storing VM exit reasons and atomic variable for tracking number of exits. These are exported so that they can be modified in other files.
```
atomic_t number_of_exits = ATOMIC_INIT(0);
atomic_t exitReasonArray[69] = {ATOMIC_INIT(0)};
EXPORT_SYMBOL(number_of_exits);
EXPORT_SYMBOL(exitReasonArray);
```
2. Added a new cpuid leaf function eax=0x4FFFFFFE inside *kvm_emulate_cpuid* method. On matching the condition eax=0x4FFFFFFFE, it will check for exit reason value in **ecx** as follows:
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

#### Modifying arch/x86/kvm/vmx/vmx.c:
1. Declare extern variable **number_of_exits** and **exitReasonArray[]**
```
extern atomic_t number_of_exits;
extern atomic_long_t number_of_cycles;
extern atomic_t exitReasonArray[69];
```
2. Fetch the values of number of exits and corresponding exit reason from *vmx_handle_exit* function 
```
atomic_fetch_add(1, &number_of_exits);
atomic_fetch_add(1, &exitReasonArray[exit_reason]);
```

#### Next steps:
1. After we save our changes, we run same make commands as before, to build the kernel.
2. Once we have successfully built the kernel and rebooted our outer vm, we start up the inner vm and run our test code inside it.
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

	eax=0x4FFFFFFE;
	for (int i=0; i < 69; i++) {
		//int i = 35;
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

## Questions/Answers/Observations
### Comment on the frequency of exits – does the number of exits increase at a stable rate? Or are there more exits performed during certain VM operations? Approximately how many exits does a full VM boot entail?
-

### Of the exit types defined in the SDM, which are the most frequent? Least?
-We observed that exits pertaining to exit code **'29-MOV DR'** occured least number of times **2 times** while exits for exit code **'48-EPT violation'** were frequent with a frequency of **214059**
