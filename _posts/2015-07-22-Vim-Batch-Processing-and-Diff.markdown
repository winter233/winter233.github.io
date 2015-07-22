---
layout: post
title: Vim处理文本实例以及比较文件
categories: vim
tags: vim
---

标签（空格分隔）： vim

---

这两天一直在验证自己写的反汇编器的正确性。今天下午突然想直接把自己的反汇编结果和objdump的结果diff一下，这样就比较直观了。待处理的文件为opencv 2.3.1的imgproc动态库文件。

处理前：

```vim
   21b44:	e59d2018 	ldr	r2, [sp, #24]
   21b48:	e59d3008 	ldr	r3, [sp, #8]
   21b4c:	e3a00001 	mov	r0, #1
   21b50:	e5832000 	str	r2, [r3]
   21b54:	eaffffd3 	b	21aa8 <_ZL15icvSklansky_32fPP12CvPoint2D32fiiPiii+0x21c>

00021b58 <_ZL24icvCalcAndWritePtIndicesPP7CvPointPiiiP5CvSeqP11CvSeqWriter.clone.9>:
   21b58:	e92d4ff0 	push	{r4, r5, r6, r7, r8, r9, sl, fp, lr}
   21b5c:	e59fc2ec 	ldr	ip, [pc, #748]	; 21e50 <_ZL24icvCalcAndWritePtIndicesPP7CvPointPiiiP5CvSeqP11CvSeqWriter.clone.9+0x2f8>
   21b60:	e24dd07c 	sub	sp, sp, #124	; 0x7c
```

用的比较多的是vim的全局命令`:global`，以及替换命令`:substitute`。获取库文件的反汇编：

```sh
objdump -D /usr/local/lib/libopencv_imgproc.so.2.3.1 > ~/imgproc.txt
```

由于反汇编了整个文件，需要删除`.text`节区以外的部分：

```vim
/.text
dgg
/.fini
dG
```

删除`.text`节区的前三个函数（反汇编器不处理这三个函数），方法也是`dgg`。删除所有函数头所在行（以“:”结尾）：

```vim
:g/:$/d
```

再将每一行中的指令地址和编码删去，只留下翻译后的指令：

```vim
gg0<C-v>Gwwwx
```
然后再删除`b`指令后面“<>”之间的内容。最初我的设想是：

```vim
:g/</D
```
这里并不识别`D`，因此有了下面的方法，下找到"<.*>"，再将其整体替换为空格。命令中的`+`相当于`/`为分隔符。`:global`命令的一般形式为：`:[range]global/{pattern}/{command}`。`:substitute`命令一般形式为：`:[range]substitute/from/to/[flags]`

```vim
:g+<.*+s/<.*//g
:g+;.*+s/;.*//g
```
第二句用于删除行末注释（以";"开始）。

删除所有空白行：

```vim
:g/^$/d
```

删除所有行末空格：

```vim
：%s\s\+$//g
```

处理完毕，效果如图：

```asm
ldr	r2, [sp, #24]
ldr	r3, [sp, #8]
mov	r0, #1
str	r2, [r3]
b	21aa8
push	{r4, r5, r6, r7, r8, r9, sl, fp, lr}
ldr	ip, [pc, #748]
sub	sp, sp, #124
```

##使用vim diff比较文件

1. 在terminal下直接输入命令：

    ```sh
    vimdiff a.txt b.txt
    ```
2. 已经打开a.txt：

    ```vim
    :vertical diffsplit b.txt
    ```
3. a.txt和b.txt都已打开，且处于垂直分屏状态：

    ```vim
    :windo diffthis
    ```


