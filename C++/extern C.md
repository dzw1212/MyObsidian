`extern "C"`是C++特有的指令，用于支持C++和C的混合编程；
该指令会告诉C++编译器使用C的规则编译指定的代码段，==除了函数重载外，不影响其他C++特性==；

C++与C的不同之一就是支持**函数重载**，因此在C++中无法直接通过函数名确定具体的函数，在编译阶段需要将形参列表作为附加项增加到函数符号中；

比如同样的一段代码，C++和C的汇编代码存在以下差别：
```C
void Function(int a, int b)
{
	printf("Hello, a = %d, b = %d.\n", a, b);
}

//C汇编结果
Function:
.LFB11:
    .cfi_startproc
    movl    %esi, %edx
    xorl    %eax, %eax
    movl    %edi, %esi
    movl    $.LC0, %edi
    jmp    printf
    .cfi_endproc


//C++汇编结果
_Z8Functionii:
.LFB12:
    .cfi_startproc
    movl    %esi, %edx
    xorl    %eax, %eax
    movl    %edi, %esi
    movl    $.LC0, %edi
    jmp    printf
    .cfi_endproc
```