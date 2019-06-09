# Lab 2 Memory Management

## Memory Layout

```
/*
 * Virtual memory map:                                Permissions
 *                                                    kernel/user
 *
 *    4 Gig -------->  +------------------------------+
 *                     |                              | RW/--
 *                     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 *                     :              .               :
 *                     :              .               :
 *                     :              .               :
 *                     |~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~| RW/--
 *                     |                              | RW/--
 *                     |   Remapped Physical Memory   | RW/--
 *                     |                              | RW/--
 *    KERNBASE, ---->  +------------------------------+ 0xf0000000      --+
 *    KSTACKTOP        |     CPU0's Kernel Stack      | RW/--  KSTKSIZE   |
 *                     | - - - - - - - - - - - - - - -|                   |
 *                     |      Invalid Memory (*)      | --/--  KSTKGAP    |
 *                     +------------------------------+                   |
 *                     |     CPU1's Kernel Stack      | RW/--  KSTKSIZE   |
 *                     | - - - - - - - - - - - - - - -|                 PTSIZE
 *                     |      Invalid Memory (*)      | --/--  KSTKGAP    |
 *                     +------------------------------+                   |
 *                     :              .               :                   |
 *                     :              .               :                   |
 *    MMIOLIM ------>  +------------------------------+ 0xefc00000      --+
 *                     |       Memory-mapped I/O      | RW/--  PTSIZE
 * ULIM, MMIOBASE -->  +------------------------------+ 0xef800000
 *                     |  Cur. Page Table (User R-)   | R-/R-  PTSIZE
 *    UVPT      ---->  +------------------------------+ 0xef400000
 *                     |          RO PAGES            | R-/R-  PTSIZE
 *    UPAGES    ---->  +------------------------------+ 0xef000000
 *                     |           RO ENVS            | R-/R-  PTSIZE
 * UTOP,UENVS ------>  +------------------------------+ 0xeec00000
 * UXSTACKTOP -/       |     User Exception Stack     | RW/RW  PGSIZE
 *                     +------------------------------+ 0xeebff000
 *                     |       Empty Memory (*)       | --/--  PGSIZE
 *    USTACKTOP  --->  +------------------------------+ 0xeebfe000
 *                     |      Normal User Stack       | RW/RW  PGSIZE
 *                     +------------------------------+ 0xeebfd000
 *                     |                              |
 *                     |                              |
 *                     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 *                     .                              .
 *                     .                              .
 *                     .                              .
 *                     |~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~|
 *                     |     Program Data & Heap      |
 *    UTEXT -------->  +------------------------------+ 0x00800000
 *    PFTEMP ------->  |       Empty Memory (*)       |        PTSIZE
 *                     |                              |
 *    UTEMP -------->  +------------------------------+ 0x00400000      --+
 *                     |       Empty Memory (*)       |                   |
 *                     | - - - - - - - - - - - - - - -|                   |
 *                     |  User STAB Data (optional)   |                 PTSIZE
 *    USTABDATA ---->  +------------------------------+ 0x00200000        |
 *                     |       Empty Memory (*)       |                   |
 *    0 ------------>  +------------------------------+                 --+
 *
 * (*) Note: The kernel ensures that "Invalid Memory" is *never* mapped.
 *     "Empty Memory" is normally unmapped, but user programs may map pages
 *     there if desired.  JOS user programs map pages temporarily at UTEMP.
 */

```

Virtual Address(VA) -> Linear Address(LA) -> Physical Address(PA), they are all 32bits.

LA = Page Directory Index (10bit, 1024) | Page Table Index (10bit, 1024) | Offset (12bit, 4K)


## Exercise 1
1. We should understand the structure of memory as above.
2. Know the structure of Page Directory and Page Table. As JOS shows we have 1024 entities for Page Directory, each of them is 4 Bytes.
And we have 1024 * 1024 Page Table entities, each of them is also 4 Bytes. 

## Exercise 2
```
A C pointer is the "offset" component of the virtual address. In boot/boot.S, we installed a Global Descriptor Table (GDT) that effectively disabled segment translation by setting all segment base addresses to 0 and limits to 0xffffffff. Hence the "selector" has no effect and the linear address always equals the offset of the virtual address. In lab 3, we'll have to interact a little more with segmentation to set up privilege levels, but as for memory translation, we can ignore segmentation throughout the JOS labs and focus solely on page translation.
```
1. Actually we do not need to care about the segment translation(va -> la), cause we mappeded them in places. we only cares about page translation(la -> pa).
2. Each Page Table entities is formed like 31 -- 12 Page Num | 11 --- 0 PLP(page level protection).
PLP describes the supervisor level which indicates page is for operating system or user.

## Question
```
Assuming that the following JOS kernel code is correct, what type should variable x have, uintptr_t or physaddr_t?
	mystery_t x;
	char* value = return_a_pointer();
	*value = 10;
	x = (mystery_t) value;
```
It should be uintptr_t, all addresses in JOS are treated as a virtual address. so JOS remaps all of physical memory starting from physical address 0 at virtual address 0xf0000000 is to help the kernel read and write memory for which it knows just the physical address.

## Exercise 4
```
Exercise 4. In the file kern/pmap.c, you must implement code for the following functions.

        pgdir_walk()
        boot_map_region()
        page_lookup()
        page_remove()
        page_insert()

check_page(), called from mem_init(), tests your page table management routines. You should make sure it reports success before proceeding.
```
Should noticed that PDE and PTE all stores the physical address, also need noticed the permission bits.

## Exercise 5
```
Exercise 5. Fill in the missing code in mem_init() after the call to check_page().

Your code should now pass the check_kern_pgdir() and check_page_installed_pgdir() checks.
```
hint: bootstack will be the bottom of kernel stack.

## Question
```
2. What entries (rows) in the page directory have been filled in at this point? What addresses do they map and where do they point? In other words, fill out this table as much as possible:
3. We have placed the kernel and user environment in the same address space. Why will user programs not be able to read or write the kernel's memory? What specific mechanisms protect the kernel memory?
4. What is the maximum amount of physical memory that this operating system can support? Why?
5. How much space overhead is there for managing memory, if we actually had the maximum amount of physical memory? How is this overhead broken down?
6. Revisit the page table setup in kern/entry.S and kern/entrypgdir.c. Immediately after we turn on paging, EIP is still a low number (a little over 1MB). At what point do we transition to running at an EIP above KERNBASE? What makes it possible for us to continue executing at a low EIP between when we enable paging and when we begin running at an EIP above KERNBASE? Why is this transition necessary?
```
2. 
| index | virtual address | info |
| ---- | ---- | ----|
| 960 | 0xf0000000 | First 4MB physical memory |
| 959 | 0xefc00000 | KERNEL STACK |
| 957 | 0xefc00000 | KERNEL Page Directory |
| 956 | 0xefc00000 | KERNEL Pages |

3. page table contains some permission bits in it's value, so we can restirct user access by checking them. 
4. 2GB, cause we only have 4MB / 8bytes = 512K pages, every page indicates 4KB memory, so we can map 2GB in total.
5. 4MB for Page Info, 512K * 4bytes = 2MB for Page table, 512K / 1024 * 4bytes = 2KB for Page directory. 6MB_ + 2KB for all.
6. after call jmp %eax, eip will jump to a high address.
   
