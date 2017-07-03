---
title: Linux内核学习之知识储备
tags: Linux内核，计算机工作原理
grammar_cjkRuby: true
---

# 1 汇编基础知识
## 1.1 汇编语言的五种寻址方式

|   寻址方式  | 格式    | 注释    |
| --- | --- | --- |
| 立即寻址 | $数字 | 例如"$0x123"，对应的值为123（0x123就是所需的值）。 |
| 直接寻址 | 数字 |  例如"0x123"，对应的值为0x123指向的内存中存放的值（0x123指的是地址）。 |
| 寄存器寻址 | %寄存器 | 例如”%exb“，对应的值为寄存器exb中的值（寄存器中存放的就是需要的值）。 |
| 间接寻址 | (%寄存器) | 例如"(%exb)"，对应的值为寄存器exb中地址指向的地址中的值（寄存器中存放的是地址）。 |
| 变址寻址 | 偏移量（%寄存器） | 例如”4（%exb）“，对应的值为exb中的值加上4，所指向内存中存放的值（寄存器exb中是相对地址，需要加上偏移量构成绝对地址）。 |

## 1.2 X86体系计算机常用寄存器
| 寄存器名 |  表示 | 数量 | 功能描述 |
| --- | --- | --- | --- |
| 数据寄存器 |  EAX、EBX、ECX、EDX | 4个 | 数据寄存器主要用来保存操作数和运算结果等信息，从而节省读取操作数所需占用总线和访问存储器的时间 |
| 变址寄存器 | ESI、EDI | 2个 | 主要用于存放存储单元在段内的偏移量，用它们可实现多种存储器操作数的寻址方式，为以不同的地址形式访问存储单元提供方便 |
| 指针寄存器 | EBP、ESP | 2个 | 主要用于存放堆栈内存储单元的偏移量，用它们可实现多种存储器操作数的寻址方式，为以不同的地址形式访问存储单元提供方便 |
| 段寄存器 | ES、CS、SS、DS、FS和GS | 6个 | 段寄存器是根据内存分段的管理模式而设置的。内存单元的物理地址由段寄存器的值和一个偏移量组合而成
的，这样可用两个较少位数的值组合成一个可访问较大物理空间的内存地址 |
| 指令指针寄存器 | EIP | 1个 | 存放下次将要执行的指令在代码段的偏移量 |
| 标志寄存器 | EFlags | 1个 | 运算结果标志位 <br> 状态控制标志位 <br> 32位标志寄存器增加的标志位 |


## 1.3 汇编语言常用命令
| 命令| 实例 | 注释 |
| --- | --- | --- |
| 入栈 | PUSH SRC  |  例子：Pushl %eax <br>将%eax寄存器中的值压入栈中,等同于如下两条指令：<br>1.sub $4, %esp //栈顶指针减4，栈向下延伸 <br> 2.Movl %eax, (%esp) //将eax中的值放入栈顶指针指向的内存位置 |
| 出栈 | POP DST | 例子：popl %eax <br> 将栈顶值取出放入寄存器%eax中，等同于如下两条指令：<br>1.movl %esp %eax //取出栈顶值放入寄存器eax中 <br>2.add $4 %esp //栈顶指针加4，栈向上收缩 |
| 调用子程序 | call address | 例子：call 0x123 <br>将当前cpu切换到地址0x123运行，原来的运行地址入站保存，等同于如下指令：<br> 1.push1 %eip //保存当前运行地址<br>2.mov1 0x123, %eip |
| 返回指令 | ret  | 将栈顶的返回地址弹出到EIP，然后按照EIP此时指示的指令地址继续执行程序,如下指令<br>1.pop1 %eip //出栈还原上次执行位置到eip，上次执行位置在跳转前被压入栈中
[ 离开指令 | leave | 将栈指针指向帧指针，然后POP备份的原帧指针到%EBP如下指令: <br> 1. mov %ebp， %esp <br> 2. pop %ebp |
| 比较指令 | jmp x, y | 比如：<br>mov ax,8 <br>mov bx,3 <br>cmp ax,bx <br>执行后：<br>ax=8,ZF=0,PF=1,SF=0,CF=0,OF=0. <br>通过cmp指令执行后，相关标志位的值就可以看出比较的结果。<br>cmp ax,bx的逻辑含义是比较ax,bx中的值。如果执行后：<br>ZF=1则AX=BX <br>ZF=0则AX！=BX <br>CF=1则AX<BX <br>CF=0则AX>=BX <br>CF=0并ZF=0则AX>BX <br>CF=1或ZF=1则AX<=BX |
| 无条件跳转 |  jmp flag | 例子：jmp .L2 <br>直接跳转到.L2执行 |
| 根据 CX、ECX 寄存器的值跳转 |  JCXZ(CX 为 0 则跳转) <br>JECXZ(ECX 为 0 则跳转) | 例子: |
| 根据 EFLAGS 寄存器的标志位跳转, 这个太多了 | JE   ;等于则跳转<br>JNE  ;不等于则跳转<br>JZ   ;为 0 则跳转<br>JNZ  ;不为 0 则跳转<br>JS   ;为负则跳转<br>JNS  ;不为负则跳转<br>JC   ;进位则跳转<br>JNC  ;不进位则跳转<br>JO   ;溢出则跳转<br>JNO  ;不溢出则跳转<br>JA   ;无符号大于则跳转<br>JNA  ;无符号不大于则跳转<br>JAE  ;无符号大于等于则跳转<br>JNAE ;无符号不大于等于则跳转<br>JG   ;有符号大于则跳转<br>JNG  ;有符号不大于则跳转<br>JGE  ;有符号大于等于则跳转<br>JNGE ;有符号不大于等于则跳转<br>JB   ;无符号小于则跳转<br>JNB  ;无符号不小于则跳转<br>JBE  ;无符号小于等于则跳转<br>JNBE ;无符号不小于等于则跳转<br>JL   ;有符号小于则跳转<br>JNL  ;有符号不小于则跳转<br>JLE  ;有符号小于等于则跳转<br>JNLE ;有符号不小于等于则跳转<br>JP   ;奇偶位置位则跳转<br>JNP  ;奇偶位清除则跳转<br>JPE  ;奇偶位相等则跳转<br>JPO  ;奇偶位不等则跳转  | 每个字母对应的缩写如下：<br>J-->jmp<br> E-->equal<br> N-->NO<br>B-->BIG<br> L-->LITTLE<br> ![enter description here][1] |
* 图解 ”pushl“ 和 ”popl“
![enter description here][2]

* 图解"call"和“ret”
![enter description here][3]


 ## 1.4 实验
 本次实验是讲一个用C语言编写的程序，编译成汇编程序，在汇编语言的基础上尽心图解分析。
1. 打开vim编写如下程序：
```c
  1 int max(int x, int y) {
  2     if (x > y) {
  3         return x;
  4     }
  5 
  6     return y;
  7 }
  8 int main(void) {
  9     return max(3, 5);                                                                                                                                                                    
 10 }

```
2. 运行执行
 ```sh
	$ gcc -S -o main.s main.c -m32 // 生汇编代码保存值main.s
 ```
3. 打开main.s删除其中已"."开始的行得到如下汇编代码
```s
  1 max:
  2     pushl   %ebp                                                                                                                                                                         
  3     movl    %esp, %ebp
  4     movl    8(%ebp), %eax
  5     cmpl    12(%ebp), %eax
  6     jle.L2
  7     movl    8(%ebp), %eax
  8     jmp.L3
  9 .L2:
 10     movl    12(%ebp), %eax
 11 .L3:
 12     popl    %ebp
 13     ret
 14 main:
 15     pushl   .%ebp
 16     movl    %esp, %ebp
 17     subl    $8, %esp
 18     movl    $5, 4(%esp)
 19     movl    $3, (%esp)
 20     call    max
 21     leave
 22     ret
```
4. 图解汇编代码
* 内存中堆栈是向下生长的，即堆栈丁的地址<=堆栈底的地址；这里为了方便描述，在程序开始执行时的，我们假定堆栈底和堆栈顶对应的地址编号是0，堆栈向下每增加一个位置，编号加1；
* EAX 寄存器用于存储函数的返回值，图示中会标出其值；其他三个寄存器EIP、EBP、ESP分别用箭头来表示其当前值；EIP就是指令寄存器，太总是指向当前正在执行的汇编指令的下一条汇编指令；EBP、ESP分别指向当前堆栈的底部和顶部；
![开始执行][4]
![enter description here][5]
![enter description here][6]
![enter description here][7]
![enter description here][8]  
![enter description here][9]
![enter description here][10]
![enter description here][11]
![enter description here][12]
![enter description here][13]
![enter description here][14]
![enter description here][15]
![enter description here][16]
![enter description here][17]
![enter description here][18]


  [1]: ./images/1499050144839.jpg "1499050144839"
  [2]: ./images/1498809451051.jpg "1498809451051"
  [3]: ./images/1498811629574.jpg "1498811629574"
  [4]: ./images/1499052483745.jpg "1499052483745"
  [5]: ./images/1499052503748.jpg "1499052503748"
  [6]: ./images/1499052521927.jpg "1499052521927"
  [7]: ./images/1499052551690.jpg "1499052551690"
  [8]: ./images/1499052565278.jpg "1499052565278"
  [9]: ./images/1499052574924.jpg "1499052574924"
  [10]: ./images/1499052588776.jpg "1499052588776"
  [11]: ./images/1499052602673.jpg "1499052602673"
  [12]: ./images/1499052618346.jpg "1499052618346"
  [13]: ./images/1499052632485.jpg "1499052632485"
  [14]: ./images/1499052642995.jpg "1499052642995"
  [15]: ./images/1499052653823.jpg "1499052653823"
  [16]: ./images/1499052666578.jpg "1499052666578"
  [17]: ./images/1499052676208.jpg "1499052676208"
  [18]: ./images/1499052685888.jpg "1499052685888"