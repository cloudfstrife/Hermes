---
title: "ELF文件格式"
date: 2024-06-14T11:43:50+08:00
lastmod: 2024-06-14T11:43:50+08:00

categories:
  - other
tags:
  - ELF
keywords:
  - ELF

# 开启数学公式渲染，可选值： mathjax, katex
#math: mathjax

# 开启各种图渲染，如流程图、时序图、类图等
#mermaid: true
---

ELF (**E**xecutable and **L**inkable **F**orma) 格式是一种对类 UNIX 系统环境中的可执行文件、目标文件和库文件的**格式标准**，与 Windows 的 PE 文件格式类似。ELF 格式是 UNIX 系统实验室作为 ABI （**A**pplication **B**inary **I**nterface）而设计的，到现在已经是 Linux 环境下的标准格式了。

<!--more-->

## 类 UNIX 系统中的ELF格式文件

类 UNIX 系统中的ELF格式文件一共有四种：

* **可重定位文件(Relocatable File)**

  这类文件包含了代码和数据，可被用来链接成可执行文件或者共享目录文件，扩展名为 `.o`

* **可执行文件(Executable File)**

  这类文件包含了可以直接执行的程序，一般没有扩展名

* **共享对象文件(Shared Object File)**

  这类文件包含了代码和数据，一般扩展名为 `.so` 和 `.a`。

* **核心转储文件(Core Dump File)**

  当进程意外终止时，系统可以将该进程的地址空间的内容以及其他信息转储在 Core Dump File 中。

## 如何查看 ELF 信息

Linux环境下可以通过 `readelf` 或者 `objdump` 工具查看 

### 一个示例

```c
#include <stdio.h>

int add(int a,int  b)
{
  return a + b;
}

int main()
{
  int a,b,c;
  a = 3;
  b = 4;
  c = 5;
  int ret = add(a,b);
  printf("%d + %d = %d , c = %d\n",a,b,ret,c);
  return 0;
}
```

编译

```bash
# 编译
$ gcc -c main.c -o main.o
# 链接
$ gcc main.o -o main
```

查看文件信息

```bash
$ file main.o
main.o: ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), not stripped
$ file main
main: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=eb202b6ac03af3a38ce6924d829c1b2d29e09618, for GNU/Linux 3.2.0, not stripped
```

## ELF 的结构

![ELF-format](/images/other/ELF/ELF-format.svg)

上图描述的 ELF 的结构主要包含以下部分：

* ELF Header
* Program Header Table
* Section Header Table
* Segments
* Sections

> Program Header Table 和 Section Header Table 是可选的

### ELF Header

ELF 头包含一些元信息，以及 Program Header Table 和 Section Header Table 在文件中的位置

文件 `/usr/include/elf.h` 中定义了相关的数据结构，下面是64位操作系统的 ELF Header 定义，增加了一些中文注释

```c
#define EI_NIDENT (16)

typedef struct
{
  /*ELF的一些标识信息，固定值*/
  unsigned char e_ident[EI_NIDENT];     /* Magic number and other info */
  /*目标文件类型：1-可重定位文件，2-可执行文件，3-共享目标文件等*/
  Elf64_Half    e_type;                 /* Object file type */
  /*文件的目标体系结构类型：3-intel 80386*/
  Elf64_Half    e_machine;              /* Architecture */
  /*目标文件版本：1-当前版本*/
  Elf64_Word    e_version;              /* Object file version */
  /*程序入口的虚拟地址，如果没有入口，可为0*/
  Elf64_Addr    e_entry;                /* Entry point virtual address */
  /*程序头表(segment header table)的偏移量，如果没有，可为0*/
  Elf64_Off     e_phoff;                /* Program header table file offset */
  /*节区头表(section header table)的偏移量，没有可为0*/
  Elf64_Off     e_shoff;                /* Section header table file offset */
  /*与文件相关的，特定于处理器的标志*/
  Elf64_Word    e_flags;                /* Processor-specific flags */
  /*ELF头部的大小，单位字节*/
  Elf64_Half    e_ehsize;               /* ELF header size in bytes */
  /*程序头表每个表项的大小，单位字节*/
  Elf64_Half    e_phentsize;            /* Program header table entry size */
  /*程序头表表项的个数*/
  Elf64_Half    e_phnum;                /* Program header table entry count */
  /*节区头表每个表项的大小，单位字节*/
  Elf64_Half    e_shentsize;            /* Section header table entry size */
  /*节区头表表项的数目*/
  Elf64_Half    e_shnum;                /* Section header table entry count */
  /*某些节区中包含固定大小的项目，如符号表。对于这类节区，此成员给出每个表项的长度字节数。*/
  Elf64_Half    e_shstrndx;             /* Section header string table index */
} Elf64_Ehdr;
```

使用 `readelf` 查看文件的 ELF Header

```bash
$ readelf -h main
ELF 头：  Magic：  7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  类别:                              ELF64
  数据:                              2 补码，小端序 (little endian)
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI 版本:                          0
  类型:                              DYN (Position-Independent Executable file)
  系统架构:                          Advanced Micro Devices X86-64
  版本:                              0x1
  入口点地址：              0x1050
  程序头起点：              64 (bytes into file)
  Start of section headers:          14000 (bytes into file)
  标志：             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         13
  Size of section headers:           64 (bytes)
  Number of section headers:         31
  Section header string table index: 30
```

### Program Header Table

Program Header Table 是从加载器的角度来看 ELF 文件的，注意：目标文件没有该部分。

Program Header Table 中的每一个表项提供了各 Segment 在虚拟地址空间和物理地址空间的大小、位置、标志、访问权限和其它方面的信息


```c
typedef struct
{
  /*segment的类型：PT_LOAD= 1 可加载的段*/
  Elf64_Word    p_type;                 /* Segment type */
  /*从文件头到该段第一个字节的偏移*/
  Elf64_Word    p_flags;                /* Segment flags */
  /*该段第一个字节被放到内存中的虚拟地址*/
  Elf64_Off     p_offset;               /* Segment file offset */
  /*在linux中这个成员没有任何意义，值与p_vaddr相同*/
  Elf64_Addr    p_vaddr;                /* Segment virtual address */
  /*该段在文件映像中所占的字节数*/
  Elf64_Addr    p_paddr;                /* Segment physical address */
  /*该段在内存映像中占用的字节数*/
  Elf64_Xword   p_filesz;               /* Segment size in file */
  /*段标志*/
  Elf64_Xword   p_memsz;                /* Segment size in memory */
  /*p_vaddr是否对齐*/
  Elf64_Xword   p_align;                /* Segment alignment */
} Elf64_Phdr;
```

使用 `readelf` 查看文件的 Program Header Table

```text
$ readelf -l main.o

本文件中没有程序头。
$ readelf -lW main

Elf 文件类型为 DYN (Position-Independent Executable file)
Entry point 0x1050
There are 13 program headers, starting at offset 64

程序头：  Type           Offset   VirtAddr           PhysAddr           FileSiz  MemSiz   Flg Align
  PHDR           0x000040 0x0000000000000040 0x0000000000000040 0x0002d8 0x0002d8 R   0x8
  INTERP         0x000318 0x0000000000000318 0x0000000000000318 0x00001c 0x00001c R   0x1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
  LOAD           0x000000 0x0000000000000000 0x0000000000000000 0x000618 0x000618 R   0x1000
  LOAD           0x001000 0x0000000000001000 0x0000000000001000 0x0001b1 0x0001b1 R E 0x1000
  LOAD           0x002000 0x0000000000002000 0x0000000000002000 0x00011c 0x00011c R   0x1000
  LOAD           0x002dd0 0x0000000000003dd0 0x0000000000003dd0 0x000248 0x000250 RW  0x1000
  DYNAMIC        0x002de0 0x0000000000003de0 0x0000000000003de0 0x0001e0 0x0001e0 RW  0x8
  NOTE           0x000338 0x0000000000000338 0x0000000000000338 0x000020 0x000020 R   0x8
  NOTE           0x000358 0x0000000000000358 0x0000000000000358 0x000044 0x000044 R   0x4
  GNU_PROPERTY   0x000338 0x0000000000000338 0x0000000000000338 0x000020 0x000020 R   0x8
  GNU_EH_FRAME   0x00201c 0x000000000000201c 0x000000000000201c 0x000034 0x000034 R   0x4
  GNU_STACK      0x000000 0x0000000000000000 0x0000000000000000 0x000000 0x000000 RW  0x10
  GNU_RELRO      0x002dd0 0x0000000000003dd0 0x0000000000003dd0 0x000230 0x000230 R   0x1

 Section to Segment mapping:
  段节...
   00     
   01     .interp 
   02     .interp .note.gnu.property .note.gnu.build-id .note.ABI-tag .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_r .rela.dyn .rela.plt 
   03     .init .plt .plt.got .text .fini 
   04     .rodata .eh_frame_hdr .eh_frame 
   05     .init_array .fini_array .dynamic .got .got.plt .data .bss 
   06     .dynamic 
   07     .note.gnu.property 
   08     .note.gnu.build-id .note.ABI-tag 
   09     .note.gnu.property 
   10     .eh_frame_hdr 
   11     
   12     .init_array .fini_array .dynamic .got 
```

类型定义如下：

```c
/* Legal values for p_type (segment type).  */

#define PT_NULL         0               /* Program header table entry unused */
#define PT_LOAD         1               /* Loadable program segment */
#define PT_DYNAMIC      2               /* Dynamic linking information */
#define PT_INTERP       3               /* Program interpreter */
#define PT_NOTE         4               /* Auxiliary information */
#define PT_SHLIB        5               /* Reserved */
#define PT_PHDR         6               /* Entry for header table itself */
#define PT_TLS          7               /* Thread-local storage segment */
#define PT_NUM          8               /* Number of defined types */
#define PT_LOOS         0x60000000      /* Start of OS-specific */
#define PT_GNU_EH_FRAME 0x6474e550      /* GCC .eh_frame_hdr segment */
#define PT_GNU_STACK    0x6474e551      /* Indicates stack executability */
#define PT_GNU_RELRO    0x6474e552      /* Read-only after relocation */
#define PT_GNU_PROPERTY 0x6474e553      /* GNU property */
#define PT_LOSUNW       0x6ffffffa
#define PT_SUNWBSS      0x6ffffffa      /* Sun Specific segment */
#define PT_SUNWSTACK    0x6ffffffb      /* Stack segment */
#define PT_HISUNW       0x6fffffff
#define PT_HIOS         0x6fffffff      /* End of OS-specific */
#define PT_LOPROC       0x70000000      /* Start of processor-specific */
#define PT_HIPROC       0x7fffffff      /* End of processor-specific */
```
重要的类型如下：

* **PT_PHDR** - Program Header Table
* **PT_INTERP** - 表示此段中保存的是链接器绝对路径的字符串
* **PT_LOAD** - 表示此段是一个需要从二进制文件映射到虚拟地址空间的段。
* **PT_DYNAMIC** - 表示此段保存了由动态链接器（即 INTERP 中指定的解释器）使用的信息。
* **PT_NOTE** - 表示此段保存专有的编译器信息


### Section Header Table

Section Header Table 包含了文件中的各个节，每个节都指定了一个类型，定义了节数据的语义。大小和在二进制文件内部的偏移

使用 `readelf` 查看文件的 Section Header Table

```text
$ readelf -SW main
There are 31 section headers, starting at offset 0x36b0:

节头：  [Nr] Name              Type            Address          Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            0000000000000000 000000 000000 00      0   0  0
  [ 1] .interp           PROGBITS        0000000000000318 000318 00001c 00   A  0   0  1
  [ 2] .note.gnu.property NOTE            0000000000000338 000338 000020 00   A  0   0  8
  [ 3] .note.gnu.build-id NOTE            0000000000000358 000358 000024 00   A  0   0  4
  [ 4] .note.ABI-tag     NOTE            000000000000037c 00037c 000020 00   A  0   0  4
  [ 5] .gnu.hash         GNU_HASH        00000000000003a0 0003a0 000024 00   A  6   0  8
  [ 6] .dynsym           DYNSYM          00000000000003c8 0003c8 0000a8 18   A  7   1  8
  [ 7] .dynstr           STRTAB          0000000000000470 000470 00008f 00   A  0   0  1
  [ 8] .gnu.version      VERSYM          0000000000000500 000500 00000e 02   A  6   0  2
  [ 9] .gnu.version_r    VERNEED         0000000000000510 000510 000030 00   A  7   1  8
  [10] .rela.dyn         RELA            0000000000000540 000540 0000c0 18   A  6   0  8
  [11] .rela.plt         RELA            0000000000000600 000600 000018 18  AI  6  24  8
  [12] .init             PROGBITS        0000000000001000 001000 000017 00  AX  0   0  4
  [13] .plt              PROGBITS        0000000000001020 001020 000020 10  AX  0   0 16
  [14] .plt.got          PROGBITS        0000000000001040 001040 000008 08  AX  0   0  8
  [15] .text             PROGBITS        0000000000001050 001050 000158 00  AX  0   0 16
  [16] .fini             PROGBITS        00000000000011a8 0011a8 000009 00  AX  0   0  4
  [17] .rodata           PROGBITS        0000000000002000 002000 00001b 00   A  0   0  4
  [18] .eh_frame_hdr     PROGBITS        000000000000201c 00201c 000034 00   A  0   0  4
  [19] .eh_frame         PROGBITS        0000000000002050 002050 0000cc 00   A  0   0  8
  [20] .init_array       INIT_ARRAY      0000000000003dd0 002dd0 000008 08  WA  0   0  8
  [21] .fini_array       FINI_ARRAY      0000000000003dd8 002dd8 000008 08  WA  0   0  8
  [22] .dynamic          DYNAMIC         0000000000003de0 002de0 0001e0 10  WA  7   0  8
  [23] .got              PROGBITS        0000000000003fc0 002fc0 000028 08  WA  0   0  8
  [24] .got.plt          PROGBITS        0000000000003fe8 002fe8 000020 08  WA  0   0  8
  [25] .data             PROGBITS        0000000000004008 003008 000010 00  WA  0   0  8
  [26] .bss              NOBITS          0000000000004018 003018 000008 00  WA  0   0  1
  [27] .comment          PROGBITS        0000000000000000 003018 00001f 01  MS  0   0  1
  [28] .symtab           SYMTAB          0000000000000000 003038 000378 18     29  18  8
  [29] .strtab           STRTAB          0000000000000000 0033b0 0001e0 00      0   0  1
  [30] .shstrtab         STRTAB          0000000000000000 003590 00011a 00      0   0  1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  D (mbind), l (large), p (processor specific)
```

类型定义如下：

```c
/* Legal values for sh_type (section type).  */

#define SHT_NULL          0             /* Section header table entry unused */
#define SHT_PROGBITS      1             /* Program data */
#define SHT_SYMTAB        2             /* Symbol table */
#define SHT_STRTAB        3             /* String table */
#define SHT_RELA          4             /* Relocation entries with addends */
#define SHT_HASH          5             /* Symbol hash table */
#define SHT_DYNAMIC       6             /* Dynamic linking information */
#define SHT_NOTE          7             /* Notes */
#define SHT_NOBITS        8             /* Program space with no data (bss) */
#define SHT_REL           9             /* Relocation entries, no addends */
#define SHT_SHLIB         10            /* Reserved */
#define SHT_DYNSYM        11            /* Dynamic linker symbol table */
#define SHT_INIT_ARRAY    14            /* Array of constructors */
#define SHT_FINI_ARRAY    15            /* Array of destructors */
#define SHT_PREINIT_ARRAY 16            /* Array of pre-constructors */
#define SHT_GROUP         17            /* Section group */
#define SHT_SYMTAB_SHNDX  18            /* Extended section indices */
#define SHT_RELR          19            /* RELR relative relocations */
#define SHT_NUM           20            /* Number of defined types.  */
#define SHT_LOOS          0x60000000    /* Start OS-specific.  */
#define SHT_GNU_ATTRIBUTES 0x6ffffff5   /* Object attributes.  */
#define SHT_GNU_HASH      0x6ffffff6    /* GNU-style hash table.  */
#define SHT_GNU_LIBLIST   0x6ffffff7    /* Prelink library list */
#define SHT_CHECKSUM      0x6ffffff8    /* Checksum for DSO content.  */
#define SHT_LOSUNW        0x6ffffffa    /* Sun-specific low bound.  */
#define SHT_SUNW_move     0x6ffffffa
#define SHT_SUNW_COMDAT   0x6ffffffb
#define SHT_SUNW_syminfo  0x6ffffffc
#define SHT_GNU_verdef    0x6ffffffd    /* Version definition section.  */
#define SHT_GNU_verneed   0x6ffffffe    /* Version needs section.  */
#define SHT_GNU_versym    0x6fffffff    /* Version symbol table.  */
#define SHT_HISUNW        0x6fffffff    /* Sun-specific high bound.  */
#define SHT_HIOS          0x6fffffff    /* End OS-specific type */
#define SHT_LOPROC        0x70000000    /* Start of processor-specific */
#define SHT_HIPROC        0x7fffffff    /* End of processor-specific */
#define SHT_LOUSER        0x80000000    /* Start of application-specific */
#define SHT_HIUSER        0x8fffffff    /* End of application-specific */
```

重要的部分：

* **.interp** - 保存了解释器的文件名，这是一个ASCII字符串
* **.gnu.hash** - 是一个GNU 风格的 HASH 表，用于快速访问所有的符号表项
* **.init** - 保存了进程初始化代码，通常由编译器自动添加
* **.text** - 用于保存程序中的代码片段
* **.fini** - 保存了进程结束代码，通常由编译器自动添加
* **.rodata** - 保存了只读数据，可以读取但不能修改。例如，编译器将出现在printf语句中的所有静态字符串封装到该节
* **.data** - 保存初始化的数据，这是普通程序数据部分，可以再程序运行时修改
* **.bss字段** - 用于保存未初始化的全局变量和局部变量
* **.comment** - 保存编译器版本信息
* **.symtab** - 符号表，各个目标文件链接的接口
* **.strtab** - 字符串表，保存符号的名字，因为各个字符串大小不一，所以统一把所有字符串放到这个段里，后续其他段通过某个符号在字符串标中的偏移可以取到符号。
* **.shstrtab** - 段表的字符串表
