# CMPE283 Assignment 1

## Team Members

* **Jayashree Sridhar (014608581)**
* **Praneetha Sripada (014353664)**

### Q1 - Contribution by each member
* **Jayashree Sridhar:** 
1. Created a Ubuntu based linux VM using VMWare Fusion. Used the scenario 1 setup mentioned in the assignment instructions video.
2. Used Intel SDM Vol. 3, section 24.6.2 for implementating the part of code that displays primary processor based capabilities supported by CPU.
3. Used Intel SDM Vol. 3, section 24.8 for implementating the part of code that displays VMX entry capabilities supported by CPU.

* **Praneetha Sripada:**  
1. Created a Ubuntu based linux VM using VMWare Fusion. Used the scenario 1 setup mentioned in the assignment instructions video.
2. Used Intel SDM Vol. 3, section 24.6.2 for implementating the part of code that displays secondary processor based capabilities supported by CPU.
3. Used Intel SDM Vol. 3, section 24.7 for implementating the part of code that displays VMX exit capabilities supported by CPU.

  

### Q2 - Steps taken for assignment
- Downloaded the ISO image of Ubuntu 20.04.1 from https://ubuntu.com/download/desktop
- Downloaded and installed VMWare Fusion Player version 12 from https://my.vmware.com/web/vmware/evalcenter?p=fusion-player-personal
- Followed Linux Easy Install steps as mentioned in https://docs.vmware.com/en/VMware-Fusion/12/com.vmware.fusion.using.doc/GUID-E9883D0F-875C-48C6-8EA4-FCEFB5254625.html
- Configured VM with 4 CPUs and 200GB memory.
* Enabled nested virtualization capability for VM.
* Started the VM and logged into the VM.
* Executed command "*cat /proc/cpuinfo | more*" and observed that the flags displayed had **vmx**, to confirm the system was setup right and nested virtualization capability was enabled on system.
* Cloned this git repository on vm.
* Developed the code for *processor based controls, entry and exit controls*, following steps outlined for *pin based control*.
* Installed gcc and make packages on VM.
* Used "*make*" command to build the kernel module.
* Inserted module using command "*sudo insmod ./filename.ko*" (*sudo* used as inserting module requires privileged access).
* Executed "*dmesg*" to obtain output logs. Our observations were:
  * Observed that the output did not have any address beginning with *000* **and**.
  * None of the bits had "*Can set & Can clear*" simultaneously set to **no**.
* As this is the expected behavior, this confirms that our output is accurate.


**NOTE:**
This host system specifications are as follows:
OS: MacOS Catalina 10.15.7
Processor: 1.4 GHz Quad-Core Intel Core i5
RAM: 8 GB




