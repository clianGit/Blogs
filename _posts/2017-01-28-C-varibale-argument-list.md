---
layout: post
title:  "C语言变参"
author: chenlian
categories: C
tags: C
excerpt: C 语言变参解决了将不定个数的参数传入给同一个函数，并能得到正确的处理结果。
---


* content
{:toc}
C 语言变参解决了将多个参数传入给同一个函数，并能得到正确的处理结果。

## C 语言变参用法

1. 在文件头文件中需要包含进stdarg.h 头文件，stdarg.h 头文件中包含了需要的va_list(),va_start(),va_arg(),va_end()等宏。
2. 在函数的原型中至少需要一个形参，再在形参的逗号后面中添加`...`。
3. 在函数里定义一个va_list型的变量。
4. 用va_start()宏初始化定义的va_list变量。
5. 用va_arg()宏返回可变的参数，va_arg()的第二个参数是要返回的参数的类型。
6. 使用va_end()宏结束可变参数的获取。


## 一个简单的例子


```c
#include<stdio.h>
#include<stdarg.h>

int sum(int num,...)
{
    va_list valist;
    int i,sum = 0;

    va_start(valist,num);
    for(i = 0;i < num ;i++){
        sum += va_arg(valist,int);
    }
    va_end(valist);

    return sum;
}

int main(void)
{
    printf("Sum :%d\n",sum(8,1,2,3,4,5,6,7,8));

    return 0;
}
```

## 函数调用规范


当高级语言函数被编译成机器码时，cpu无从知道函数调用需要的参数类型和参数个数，传递参数的工作必须由函数调用者和函数本身来协调。常见的函数调用有如下几种：


stdcall 的调用方式：


1. 参数从右到左压入堆栈。
2. 由被调用函数来恢复堆栈。
3. 函数名自动加前导下划线，后面紧跟者一个@其后紧跟参数的尺寸。


cdel 的调用方式：


1. 参数从右向左依次压入堆栈。
2. 由调用者恢复堆栈。
3. 函数名自动加前导下划线。


fastcall 的调用方式：


1. 函数的第一个和第二个DWORD参数通过ecx 和 edx 传递，后面的参数从右向左的顺序入栈。
2. 被调用函数清理堆栈。
3. 函数名自动加前导下划线，后面紧跟者一个@其后紧跟参数的尺寸。

thiscall 的调用方式：


1. 参数从右向左压入堆栈。如果参数个数确定，this 指针通过ecx 传递给被调用者。如果参数个数不确定，this 指针在所有参数压入栈后被压入栈。
2. 参数个数不定的，有调用者清理堆栈，否则由函数自己清理。


gcc x86_64 中使用%rdi,%rsi,%rdx,%rcx,%r8,%r9 用作函数参数，依次对应函数的第1个参数，第2个参数，第3个参数，第4个参数，第5个参数，第6个参数；对于多于6个参数的函数，会使用堆栈来保存函数参数。


## 汇编代码分析


上述变参例子的汇编代码


```s
        .file   "test4.c"
        .text
        .globl  sum
        .type   sum, @function
sum:
.LFB0:
        .cfi_startproc
        pushq   %rbp
        .cfi_def_cfa_offset 16
        .cfi_offset 6, -16
        movq    %rsp, %rbp
        .cfi_def_cfa_register 6
        subq    $104, %rsp
        movl    %edi, -212(%rbp)                ; arg1  num
        movq    %rsi, -168(%rbp)                ; arg2   1
        movq    %rdx, -160(%rbp)                ; arg3   2
        movq    %rcx, -152(%rbp)                ; arg4   3
        movq    %r8, -144(%rbp)                 ; arg5   4
        movq    %r9, -136(%rbp)                 ; arg6   5
        testb   %al, %al
        je      .L8
        movaps  %xmm0, -128(%rbp)
        movaps  %xmm1, -112(%rbp)
        movaps  %xmm2, -96(%rbp)
        movaps  %xmm3, -80(%rbp)
        movaps  %xmm4, -64(%rbp)
        movaps  %xmm5, -48(%rbp)
        movaps  %xmm6, -32(%rbp)
        movaps  %xmm7, -16(%rbp)
.L8:
        movl    $0, -184(%rbp)                  ; sum = 0
        movl    $8, -208(%rbp)                  
        movl    $48, -204(%rbp)
        leaq    16(%rbp), %rax
        movq    %rax, -200(%rbp)
        leaq    -176(%rbp), %rax
        movq    %rax, -192(%rbp)
        movl    $0, -180(%rbp)                  ; i = 0
        jmp     .L3
.L6:
        movl    -208(%rbp), %eax
        cmpl    $47, %eax
        ja      .L4
        movq    -192(%rbp), %rax
        movl    -208(%rbp), %edx
        movl    %edx, %edx
        addq    %rdx, %rax
        movl    -208(%rbp), %edx
        addl    $8, %edx
        movl    %edx, -208(%rbp)
        jmp     .L5
.L4:
        movq    -200(%rbp), %rax
        leaq    8(%rax), %rdx
        movq    %rdx, -200(%rbp)
.L5:
        movl    (%rax), %eax
        addl    %eax, -184(%rbp)                ; sum = sum + ?
        addl    $1, -180(%rbp)                  ; i++
.L3:
        movl    -180(%rbp), %eax
        cmpl    -212(%rbp), %eax                ; if i < num
        jl      .L6
        movl    -184(%rbp), %eax
        leave
        .cfi_def_cfa 7, 8
        ret
        .cfi_endproc
.LFE0:
        .size   sum, .-sum
        .section        .rodata
.LC0:
        .string "Sum :%d\n"
        .text
        .globl  main
        .type   main, @function
main:
.LFB1:
        .cfi_startproc
        pushq   %rbp
        .cfi_def_cfa_offset 16
        .cfi_offset 6, -16
        movq    %rsp, %rbp
        .cfi_def_cfa_register 6
        subq    $8, %rsp
        pushq   $8
        pushq   $7
        pushq   $6
        movl    $5, %r9d
        movl    $4, %r8d
        movl    $3, %ecx
        movl    $2, %edx
        movl    $1, %esi
        movl    $8, %edi
        movl    $0, %eax
        call    sum
        addq    $32, %rsp
        movl    %eax, %esi
        movl    $.LC0, %edi
        movl    $0, %eax
        call    printf
        movl    $0, %eax
        leave
        .cfi_def_cfa 7, 8
        ret
        .cfi_endproc
.LFE1:
        .size   main, .-main
        .ident  "GCC: (GNU) 6.3.1 20161221 (Red Hat 6.3.1-1)"
        .section        .note.GNU-stack,"",@progbits

```

变参中传参时，参数在内存中的分布：


```
address			value
(low)
-212   			 num   edi
-208			  8    
-204			  48
-200			  address of arg7
-192			  arddres of arg2 - 8，即-176（bsp）
-184			  sum
-180			  i
-176			 （变参的第一个参数的地址-8）
-168			 arg2 = 1		rsi
-160			 arg3 = 2		rdx
-152			 arg4 = 3		rcx
-144			 arg5 = 4		r8
-136			 arg6 = 5		r9
-128
-120
-112			 
-104  (bsp)	
.	 
.
.
.
-0   (rbp)
+8  			 function return address
+16 			 arg7 = 6
+24 			 arg8 = 7
+32 			 arg9 = 8
（high）
```

部分汇编代码解析


```s
        movl    -208(%rbp), %eax
        cmpl    $47, %eax
        ja      .L4
```


-208(%rbp) 中，每次取变参的数值时，其值加8（变参大小为8个地址偏移），其初始值为8。因为前5个函数参数地址和后面的参数的地址存放在不同位置（`Note:前5个函数参数在-168~-136，后面存放在+16~+32`），当其值为48（8 +８×５）大于47时，函数的变参的起始地址发生变化（`Note:使用栈存放`），因此程序跳转。


```s
        movq    -192(%rbp), %rax
        movl    -208(%rbp), %edx
        movl    %edx, %edx
        addq    %rdx, %rax
        movl    -208(%rbp), %edx
        addl    $8, %edx
        movl    %edx, -208(%rbp)
        jmp     .L5
```


而当小于或等于47时，-192（%rbp）存放的是变参第一个参数的地址（`为-178（%rbp）`）减8。而第一个变参的地址之后是接着的第2，3，4，5变参。因此，通过地址加8循环操作即可获取变参的值。

## stdarg.h 宏分析


va_list arg


va_list（）宏  变参的类型，包含了va_start（）、va_arg（）、va_end（）需要的信息。


va_start(va_list ap,last_arg) 


va_start() 宏会初始化ap，last_arg 最后一个非变参的变量，也即是函数参数列表中第一个变参前一个变量。变参需要通过它来定位地址。


va_arg(va_list ap，type）


va_arg（）宏将 ap 转为需要的类型的值返回，并更新 ap 为变参的下一个。


va_end(va_list ap)


va_end() 宏 结束ap，在这之后，ap 的值未定义。




