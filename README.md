# KVM-Spoofing-
A guide for spoofing KVM and making it undetectable

Hello. This is a repost of a KVM guide I wrote about a year ago where I made my Kernel based virtual machine undetectable. This guide has been used by thousands of users across multiple sites, such as Reddit, Malware Anaylsis Forums, Gaming Forums, and anti-cheat reverse engineering sites. I decided it was worth a repost and update because even a year later, I occasionally still get asked about it.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
**Modifying QEMU**

Modify the following strings : 

QEMU HARDDISK inside of /hw/ide/core.c  &  /hw/scsi/scsi-disk.c

QEMU DVD-ROM  inside of /hw/ide/core.c  &  /hw/ide/atapi.c

QEMU CD-ROM   inside of /hw/ide/core.c  &  /hw/scsi/scsi-disk.c

QEMU MICRODRIVE inside of /hw/ide/core.c
QEMU PenPartner tablet inside of /hw/usb/dev-wacom.c & /hw/scsi/scsi-disk.c

padstr inside of /hw/ide/atapi.c

KVMKVMKVM\\0\\0\\0 inside of /target/i386/kvm.c

bochs inside of /block/bochs.c

Bochs Pseudo inside of /roms/ipxe/src/drivers/net/pnic.c

Modify these to legitimate vendors, for example QEMUHARDDISK could be changed to something such like " Toshiba MQ01ABD " - It is unlikely that some system will actually check this on a deeper level, but try to use a disk model that comes in same size as your virtual disk. The truth is that most applications will simply check if Harddisk model == QEMU 

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
**Modifying SMBIOS ( Seabios or OVMF )**

If you are using Seabios, modify the following strings :

Bochs & BXPC          inside of src/config.h ( multiple instances )

/QEMU\/Bochs/ & qemu  inside of vgasrc/Kconfig

/06\/23\/99/          inside ofsrc/misc.c

/04\/01\/2014/        inside of src/fw/biostables.c

s/01\/01\/2011       inside of src/fw/smbios.c

seabios               inside of src/fw/biostables.c

If you are using OVMF, modify the following strings :                       // THIS IS NOT UP TO DATE

EFI Development Kit II / OVMF\0 inside of edk2/OvmfPkg/SmbiosPlatformDxe/SmbiosPlatformDxe.c

0.0.0\0                         inside of edk2/OvmfPkg/SmbiosPlatformDxe/SmbiosPlatformDxe.c

02/06/2015\0                    inside of edk2/OvmfPkg/SmbiosPlatformDxe/SmbiosPlatformDxe.c


-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
**Modifying the Linux Kernel to spoof VM_EXIT on RDTSC**

VIM to /x86/kvm/vmx/vmx.c and create a function called handle_RDTSC 

`static int handle_rdtsc(struct kvm_vcpu *vcpu) {                 // This code only works for Intel CPUs. AMD CPUs will need their own function and exit handler in SVM. 

uint32_t data;     

data = 500; 

printk("[vmkernel] handling fake rdtsc from cpl %i\n", vmx_get_cpl(vcpu));  

vcpu->arch.regs[VCPU_REGS_RAX] = data & -1u;     

vcpu->arch.regs[VCPU_REGS_RDX] = (data >> 32) & -1u;   

skip_emulated_instruction(vcpu); 

return 1; 
}`

After the previous function is created, create an exit handler for RDTSC :
[EXIT_REASON_RDTSC] = handle_rdtsc

This is the simplest way to handle 99% of VM_Exit checks that I use, however some software may check the actual timings, in which the example would fail. For this, you would need to use something like https://github.com/SamuelTulach/BetterTiming

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
**Installing and modifying our Virtual Machine**

After you are done modifying the previous files, recompile each package and install your KVM on virt machine manager. Install it with GPU passthrough. 
I am not going to explain how to install a KVM since it would be too long, but a good guide I recommend is https://github.com/vanities/GPU-Passthrough-Arch-Linux-to-Windows10
I must note that if you are using Seabios & a Debian based linux distro, I recommend modifying QEMU and Seabios by using https://github.com/doomedraven/Tools/blob/master/Virtualization/kvm-qemu.sh - Personally I cant use this as I use an Arch based distro and OVMF

While installing a KVM, set realistic RAM and Harddisk sizes, ie for 

8   GB RAM : 8192  MBs

16  GB RAM : 16384 MBs

32  GB RAM : 32768 MBs

Disk size is entirely upon the purpose of your KVM, but try to make the size equivalenet to the model of your harddisk picked in /hw/ide/core.c
Make the SN of the harddrive look realistic!

**For our last step, we will need to modify our KVM's XML file. In your XML, modify the following ** 


Set: <cpu mode = "host-passthrough" check="none"/>                                     
  
Set: type="raw" cache="none" io="native" discard="ignore" detect_zeroes="off"      
  
Set: vendor_id state="on" value="XXXX"                                                  
  
Set:  <kvm> <hidden state="on"/> </kvm>                                 
  
Set: feature policy="disable" name="hypervisor"                             
  
  
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  If all is done properly, your KVM should be completely undetectable! If your guest machine is on Windows, I recommend using PAFish to check it.
  
  RESULTS : ![image](https://user-images.githubusercontent.com/88210134/130307422-b019ebcb-8c9f-4f0c-a028-1b0270475a2b.png)

