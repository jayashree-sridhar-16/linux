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
2. Added a new cpuid leaf function eax=0x4FFFFFFE inside *kvm_emulate_cpuid* method. On matching the condition eax=0x4FFFFFFFE, it will check for exit reason value in **ecx**
