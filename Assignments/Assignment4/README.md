# CMPE283 Assignment 4

## Team Members

* **Jayashree Sridhar (014608581)**
* **Praneetha Sripada (014353664)**

### Q1 - Contribution by each member
* **Jayashree Sridhar:** 


* **Praneetha Sripada:**  


  

### Q2 - Steps taken for assignment
-The configuration setup for assignment 3 was used for this assignment.

###  Implementation
1. Boot inner VM with test code.
2. Once VM is booted, for CPUID leaf function 0x4FFFFFFE, record total exit count information (total count for each type of exit handled by KVM).
3. Shutdown your inner VM.
4. Remove the ‘kvm-intel’ module from running kernel.
```
rmmod kvm-intel
```
5. Reload the kvm-intel module with the parameter ept=0 to enable shadow paging
6. Module is found in /lib/modules/5.9.0+/kernel/arch/x86/kvm
```
insmod /lib/modules/5.9.0+/kernel/arch/x86/kvm/kvm-intel.ko ept=0
```
7. Boot the same inner VM again, and capture the same output as you did in step 2.

## Questions/Answers/Observations

### Include a sample of your print of exit count output from dmesg from “with ept” and “without ept”.

### What did you learn from the count of exits? Was the count what you expected? If not, why not?
- Nested paging mode (with ept) involves much less number of exits to handle paging operations as compared to shadow paging mode (without ept).

### What changed between the two runs (ept vs no-ept)?
- With EPT mode, two layer page table is used for address translation. Guest VM does not exit for INVLPG instruction or for any operation on CR3/0/4 does not require VM exit, as Guest VM owns the page table
-In shadow paging mode, VMM has to emulate any operation on CR3/0/4 and INVLPG instruction, as Guest VM does not own the page table.

