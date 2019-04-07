#Lec 1

### Hint:
When you first boot jos with qemu, you may found a error like "Triple fault.  Halting for inspection via QEMU monitor."
Cause the default GNFMakefile lacks of GCC flag (-fno-pic).

### Exercise 2
Bootloader open A20 gate, load global page table, then alter into real mode.

### Exercies 3
1. At what point does the processor start executing 32-bit code? What exactly causes the switch from 16- to 32-bit mode?
```asm
lgdt    gdtdesc
movl    %cr0, %eax
orl     $CR0_PE_ON, %eax
movl    %eax, %cr0
```
before ```movl    %eax, %cr0``` bootloader is in 16-bit mode, after that bootloader is in 32-bit mode.

2. What is the last instruction of the boot loader executed, and what is the first instruction of the kernel it just loaded?
```
 ((void (*)(void)) (ELFHDR->e_entry))();
```
bootload find kern entry address from elf header info, then call into it.

3.Where is the first instruction of the kernel?
```
movw  $0x1234,0x472 # warm boot
```
4. How does the boot loader decide how many sectors it must read in order to fetch the entire kernel from disk? Where does it find this information?
(ELF Format)[http://www.skyfree.org/linux/references/ELF_Format.pdf]
ELFHDR -> e_phoff # This member holds the program header table’s file offset in bytes. If the file has no program header table, this member holds zero.
ELFHDR -> e_phnum # This member holds the number of entries in the program header table. Thus the product of e_phentsize and e_phnum gives the table’s size in bytes. If a file has no program header table, e_phnum holds the value zero.


### Exercies 4
