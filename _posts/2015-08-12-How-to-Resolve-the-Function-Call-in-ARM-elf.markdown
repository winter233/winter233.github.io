---
layout: post
title: How to resolve the function call in arm disassembly
categories: disassembly
posts: [arm,disassembly,relocation]
---

In ARM ELF, function or procedure are performed via `bl` and `blx`, the [aapcs](http://infocenter.arm.com/help/topic/com.arm.doc.ihi0042e/IHI0042E_aapcs.pdf) writes:

>Both the ARM and Thumb instruction sets contain a primitive subroutine call instruction, BL, which performs a branch-with-link operation. The effect of executing BL is to transfer the sequentially next value of the program counter, the return address, into the link register (LR) and the destination address into the program counter (PC). Bit 0 of the link register will be set to 1 if the BL instruction was executed from Thumb state, and to 0 if executed from ARM state. The result is to transfer control to the destination address, passing the return address in LR as an additional parameter to the called subroutine.

In disassembling, the address of `bl`/`blx` should be resolved to correspondent symbol. Some of the symbols could be found in `dynsym` (also in `symtab`), but others couldn't.

Here is an example. I disassemble libopencv_imgproc.so, which is a part of `openCV` lib. 

```
00176dac <cvResize>:
  176dac:	e92d40f0 	push	{r4, r5, r6, r7, lr}
  176db0:	e1a03000 	mov	r3, r0
  176db4:	e24dd0fc 	sub	sp, sp, #252	; 0xfc
  ...
  176dd4:	e59f4358 	ldr	r4, [pc, #856]	; 177134 <cvResize+0x388>
  176dd8:	e58d5000 	str	r5, [sp]
  176ddc:	ebfaa33a 	bl	1facc <_init+0x2cc>
  ...
```
 
 First, locate the target address of `blx`--`1facc`
 
```
   1facc:	e28fc601 	add	ip, pc, #1048576	; 0x100000
   1fad0:	e28cca7c 	add	ip, ip, #507904	; 0x7c000
   1fad4:	e5bcf61c 	ldr	pc, [ip, #1564]!	; 0x61c
```

calculate the result of `pc`: `0x1fad4 + 0x100000 + 0x7c000 + 0x61c = 0x19c0f0`. `0x19c0f0` is a relocation address, which can be found in the `.rel.plt` table (To get the relocation symbol of a file, using `objdump -r` or `readelf -R`).

```
 Offset     Info    Type            Sym.Value  Sym. Name
0019c0f0  00001316 R_ARM_JUMP_SLOT   00000000   _ZN2cv10cvarrToMatEPKv
```

Why relocation? 

>Relocation information is used by linkers in order to bind symbols and addresses that could not be determined when the initial object was generated.

the type of `rel.plt` section is `REL`, which is defined as

```c
typedef struct {
    Elf32_Addr r_offset;
    Elf32_Word r_info;
} Elf32_Rel;
```

We can only acquire two item of a relocation symbol directly, but there are 5 five items in the table. the other three can be calculated from the `r_info` variable.

`r_offset` is the relocation address. For an executable file or a shared object, the value is the virtual address of the storage unit affected by the relocation.

`r_info` gives both the symbol table index with respect to which the relocation must be made, and the type of relocation to apply. 

```
#define ELF32_R_SYM(i) ((i)>>8)
#define ELF32_R_TYPE(i) ((unsigned char)(i))
#define ELF32_R_INFO(s,t) (((s)<<8)+(unsigned char)(t))
```

As described above, the name of the symbol is the 19th `(0x1319>>8)` entry of the `dynsym`,

```
Symbol table '.dynsym' contains 1656 entries:
   Num:    Value  Size Type    Bind   Vis      Ndx Name
    19: 00000000     0 FUNC    GLOBAL DEFAULT  UND _ZN2cv10cvarrToMatEPKvbbi
```

type is `(unsigned char)0x1316=0x16` `R_ARM_JUMP_SLOT`, which can be found in the [aaelf](http://infocenter.arm.com/help/topic/com.arm.doc.ihi0044e/IHI0044E_aaelf.pdf). Relocation types are processor-specific.

When dealing with a relocation symbol, first calculate the relocation address via `plt` section, then find the correspondant item in `rel.plt`, finally get the index in `dynsym`.

There are not only functions need to be relocated, some global variables also need relocate. The relocation of global variables can be found in `rel.dyn`, which would be discribed in a later post.

ref:  
1. [aaelf](http://infocenter.arm.com/help/topic/com.arm.doc.ihi0044e/IHI0044E_aaelf.pdf)  
2. [my question on stakcoverflow](http://reverseengineering.stackexchange.com/questions/9033/how-to-recognize-the-function-call-in-a-dynamic-lib)


