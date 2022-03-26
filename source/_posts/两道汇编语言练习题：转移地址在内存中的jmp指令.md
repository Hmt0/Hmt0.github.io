---
title: 两道汇编语言练习题:转移地址在内存中的jmp指令
date: 2022-01-21 12:18:53
tags: 汇编语言
---

断断续续地汇编语言学到一半了，这次做课后作业可把我难坏了，赶紧记录一下。

#### 检测点9.1

##### （1）程序如下。

```assembly
assume cs:code
data segment
    ?
data ends

code segment
    start: mov ax, data
    mov ds,ax
    mov bx,0
    jmp word ptr [bx+1]
code ends
end start
```

*若要使程序中的`jmp`指令执行后，`CS:IP`指向程序的第一条指令，在data段中应该定义哪些数据？*

##### 思路：

首先把`jmp word ptr`的功能定义读了好几遍：从内存单元地址出开始存放着一个字，是转移的目的偏移地址。

也就是说`[bx+1]`指明的**内存地址**存放着目的偏移地址的**字**。

而`[bx]`表示段地址存放在`ds`，偏移地址存放在`bx`中的内存单元，在本题中就是`ax:0001H`，也就是data段中偏移地址为1的内存单元。

CS不变，所以我们要填入的数据应使**data段中偏移地址为1的内存单元**（字）存放的内容为0，这样才能跳转到code的第一条语句，所以答案应该是`db 2 dup(0)`

##### （2）程序如下

```assembly
assume cs:code

data segment
    dd 12345678h
data ends

code segment
    start: mov ax,data
           mov ds,ax
           mov bx,0
           mov [bx],__
           mov [bx+2],__
           jmp sword ptr ds:[0]
code ends
end start
```

*补全程序，使`jmp`指令执行后，`CS:IP`指向程序的第一条指令*

##### 思路：

这两道题思路差不多，就是实现细节上有点区别。

data段中定义了双字型数据，占内存中的2个字，4个字节。`jmp sword ptr`的功能是从内存单元地址处开始存放着两个字，高地址处的字是转移的目的段地址，低地址处事转移的目的偏移地址。

这段代码中，`[bx]`指明的内存内存地址应该是偏移地址（0），`[bx+2]`是段地址（cs）。我们的目的是使data段中定义的内容是`cs:0`。

明确了这个思路之后还是踩了两个小坑，一开始直接填了0，报错了，这是因为数据不能直接放入内存单元，应该通过寄存器传送；这也就是上一行`mov bx,0`的作用，填入`mov[bx],bx`正好实现把0传送到`ds:0`。这是偏移地址。

第二处补上`mov[bx+2],cs`就可以了。
