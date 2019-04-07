#Lec 1

### Hint:
When you first boot jos with qemu, you may found a error like "Triple fault.  Halting for inspection via QEMU monitor."
Cause the default GNFMakefile lacks of GCC flag (-fno-pic).

### Exercise 2
Bootloader open A20 gate, load global page table, then alter into real mode.

### Exercise 3
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

3. Where is the first instruction of the kernel?
```
	movw  $0x1234,0x472 # warm boot
```
4. How does the boot loader decide how many sectors it must read in order to fetch the entire kernel from disk? Where does it find this information?

[ELF Format](http://www.skyfree.org/linux/references/ELF_Format.pdf)

```
	ELFHDR -> e_phoff # This member holds the program header table’s file offset in bytes. If the file has no program header table, this member holds zero.
	ELFHDR -> e_phnum # This member holds the number of entries in the program header table. Thus the product of e_phentsize and e_phnum gives the table’s size in bytes. If a file has no program header table, e_phnum holds the value zero.
```

### Exercise 4
Read about programming with pointers in C. The best reference for the C language is The C Programming Language by Brian Kernighan and Dennis Ritchie (known as 'K&R'). We recommend that students purchase this book (here is an Amazon Link) or find one of MIT's 7 copies.

Read 5.1 (Pointers and Addresses) through 5.5 (Character Pointers and Functions) in K&R. Then download the code for pointers.c, run it, and make sure you understand where all of the printed values come from. In particular, make sure you understand where the pointer addresses in printed lines 1 and 6 come from, how all the values in printed lines 2 through 4 get there, and why the values printed in line 5 are seemingly corrupted.

There are other references on pointers in C (e.g., A tutorial by Ted Jensen that cites K&R heavily), though not as strongly recommended.

Warning: Unless you are already thoroughly versed in C, do not skip or even skim this reading exercise. If you do not really understand pointers in C, you will suffer untold pain and misery in subsequent labs, and then eventually come to understand them the hard way. Trust us; you don't want to find out what "the hard way" is.

### Exercise 5

### Exercise 6
Reset the machine (exit QEMU/GDB and start them again). Examine the 8 words of memory at 0x00100000 at the point the BIOS enters the boot loader, and then again at the point the boot loader enters the kernel. Why are they different? What is there at the second breakpoint? (You do not really need to use QEMU to answer this question. Just think.)

When system enters boot loader there is nothing in address 0x100000.
After enters kernel, there is some content because bootloader load the kernel between(960KB-1MB 64KB).



