# CMPE283 Assignment 1

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
5. Else we identify ecx value corresponds to exits defined in SDM and enabled in kvm. Then we write the **value corresponding to number of exits for exit reason in ecx to eax and value of 0 to registers ebx,ecx and edx.***

#### Modifying arch/x86/kvm/vmx/vmx.c:
