#Lec 1

### Hint:
When you first boot jos with qemu, you may found a error like "Triple fault.  Halting for inspection via QEMU monitor."
Cause the default GNFMakefile lacks of GCC flag (-fno-pic), you should add this flag into GNUMakefile.

### Exercise 2
  Use GDB's si (Step Instruction) command to trace into the ROM BIOS for a few more instructions, and try to guess what it might be doing. You might want to look at Phil Storrs I/O Ports Description, as well as other materials on the 6.828 reference materials page. No need to figure out all the details - just the general idea of what the BIOS is doing first.

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

  ELF doc -> [ELF Format](http://www.skyfree.org/linux/references/ELF_Format.pdf)

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
  Trace through the first few instructions of the boot loader again and identify the first instruction that would "break" or otherwise do the wrong thing if you were to get the boot loader's link address wrong. Then change the link address in boot/Makefrag to something wrong, run make clean, recompile the lab with make, and trace into the boot loader again to see what happens. Don't forget to change the link address back and make clean again afterward!

### Exercise 6
  Reset the machine (exit QEMU/GDB and start them again). Examine the 8 words of memory at 0x00100000 at the point the BIOS enters the boot loader, and then again at the point the boot loader enters the kernel. Why are they different? What is there at the second breakpoint? (You do not really need to use QEMU to answer this question. Just think.)

When system enters boot loader there is nothing in address 0x100000.
After enters kernel, there is some content because bootloader load the kernel between(960KB-1MB 64KB).

### Exercise 7
  What is the first instruction after the new mapping is established that would fail to work properly if the mapping weren't in place? Comment out the movl %eax, %cr0 in kern/entry.S, trace into it, and see if you were right.


Because we are not truly mapping our virtual address 0xf0100000 to 0x00100000, so we are not able to access such high memory, which will cause page falut.

### Exercise 8
  We have omitted a small fragment of code - the code necessary to print octal numbers using patterns of the form "%o". Find and fill in this code fragment.

```c
diff --git a/lib/printfmt.c b/lib/printfmt.c
index 28e01c9..c0976de 100644
--- a/lib/printfmt.c
+++ b/lib/printfmt.c
@@ -206,10 +206,9 @@ vprintfmt(void (*putch)(int, void*), void *putdat, const char *fmt, va_list ap)
 		// (unsigned) octal
 		case 'o':
 			// Replace this with your code.
-			putch('X', putdat);
-			putch('X', putdat);
-			putch('X', putdat);
-			break;
+      num = getuint(&ap, lflag);
+      base = 8;
+      goto number;
 
 		// pointer
 		case 'p':
```

1. Explain the interface between printf.c and console.c. Specifically, what function does console.c export? How is this function used by printf.c?
console.c is a bridge of software to display, it exports some methods to display content in monitor. printf.c will use the function cputchar.

2. Explain the following from console.c:

```c
1      if (crt_pos >= CRT_SIZE) {
2              int i;
3              memmove(crt_buf, crt_buf + CRT_COLS, (CRT_SIZE - CRT_COLS) * sizeof(uint16_t));
4              for (i = CRT_SIZE - CRT_COLS; i < CRT_SIZE; i++)
5                      crt_buf[i] = 0x0700 | ' ';
6              crt_pos -= CRT_COLS;
7      }>)}
```

This is will refresh the monitor and append a new line.

### Exercise 9
Determine where the kernel initializes its stack, and exactly where in memory its stack is located. How does the kernel reserve space for its stack? And at which "end" of this reserved area is the stack pointer initialized to point to?

```c
movl $(bootstacktop),%esp # at kern/entry.S Line 77
```

### Exercise 11 

```c
diff --git a/kern/monitor.c b/kern/monitor.c
index e137e92..afe2f3b 100644
--- a/kern/monitor.c
+++ b/kern/monitor.c
@@ -58,6 +58,18 @@ int
 mon_backtrace(int argc, char **argv, struct Trapframe *tf)
 {
 	// Your code here.
+  uint32_t eip;
+  uint32_t* ebp = (uint32_t *) read_ebp();
+  while (ebp) {
+    eip = *(ebp + 1);
+    cprintf("ebp %x eip %x args", ebp, eip);
+    uint32_t *args = ebp + 2;
+    for (int i=0; i<5;i++) {
+      cprintf(" %08x ", (uint32_t) args[i]);
+    }
+    cprintf("\n");
+    ebp = (uint32_t *) *ebp;
+  }
 	return 0;
 }
```
### Exercise 12
Modify your stack backtrace function to display, for each eip, the function name, source file name, and line number corresponding to that eip.

In this exercise we should understand struct of stabs segement which stores debug info. 
```c
diff --git a/kern/kdebug.c b/kern/kdebug.c
index 9547143..1b2a655 100644
--- a/kern/kdebug.c
+++ b/kern/kdebug.c
@@ -179,7 +179,12 @@ debuginfo_eip(uintptr_t addr, struct Eipdebuginfo *info)
          //      Look at the STABS documentation and <inc/stab.h> to find
          //      which one.
          // Your code here.
  -
  +  stab_binsearch(stabs, &lline, &rline, N_SLINE, addr);
  +  if (lline <= rline) {
  +      info->eip_line = stabs[lline].n_desc;
  +  } else {
  +      return -1;
  +  }
  
          // Search backwards from the line number for the relevant filename
          // stab.>)

--- a/kern/monitor.c
+++ b/kern/monitor.c
@@ -24,6 +24,7 @@ struct Command {
 static struct Command commands[] = {
         { "help", "Display this list of commands", mon_help },
         { "kerninfo", "Display information about the kernel", mon_kerninfo },
 +  { "backtrace", "Display backtrace info", mon_backtrace },
  };

 /***** Implementations of basic kernel monitor commands *****/
@@ -57,7 +58,23 @@ mon_kerninfo(int argc, char **argv, struct Trapframe *tf)
 int
 mon_backtrace(int argc, char **argv, struct Trapframe *tf)
 {
 -       // Your code here.
 +  uint32_t eip;
 +  uint32_t* ebp = (uint32_t *) read_ebp();
 +  while (ebp) {
 +    eip = *(ebp + 1);
 +    cprintf("ebp %x eip %x args", ebp, eip);
 +    uint32_t *args = ebp + 2;
 +    for (int i=0; i<5;i++) {
 +      cprintf(" %08x ", (uint32_t) args[i]);
 +    }
 +    cprintf("\n");
 +               struct Eipdebuginfo debug_info;
 +    debuginfo_eip(eip, &debug_info);
 +               cprintf("\t%s:%d: %.*s+%d\n",
     +                       debug_info.eip_file, debug_info.eip_line, debug_info.eip_fn_namelen,
     +                       debug_info.eip_fn_name, eip - debug_info.eip_fn_addr);
 +    ebp = (uint32_t *) *ebp;
 +  }
         return 0;
          }>)}}}
```


