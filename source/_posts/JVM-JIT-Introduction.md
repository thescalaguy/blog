---
title: JVM JIT - Introduction
tags:
  - compilers
date: 2018-04-17 20:31:00
---

## Motivation
My day job requires me to write code in Clojure. This means the code is eventually compiled to bytecode and run on the JVM. Intrigued by how the JVM does what it does, I decided to dig a little deeper and look at how it optimizes the code on the fly. In this series of posts I will be looking at JVM JIT (Just-In-Time) compiler.

## Myriad ways to run a program

Before I go into how JVM JIT works, I want to take a quick look at how interpreted and compiled languages work. For this post, I'll take a look at the working of Python (an interpreted language) and C (a compiled language).

### Python

Python, by default, ships with [CPython](https://github.com/python/cpython) - the original Python interpreter that runs C code for every bytecode. There's other implementations like [IronPython](http://ironpython.net/) or [PyPy](https://pypy.org). IronPython turns Python into a fully compiled language running on top of Microsoft's .NET Common Language Runtime (CLR) whereas PyPy turns Python into a JIT compiled language. For the sake of this post, however, I will look at CPython and how it works.

I'll start with some code which will print the bytecodes for another Python file that is passed to it. 

{% codeblock bytecode.py lang:python %}
from sys import argv
from dis import dis

script, path = argv
source_file = open(path)
source_code = source_file.read()
compiled = compile(source_code, "<string>", "exec")
bytecodes = dis(compiled)

print(bytecodes)
source_file.close()
{% endcodeblock %}

Next, here's some code that'll print numbers.

{% codeblock print_numbers.py lang:python %}
for n in [101, 102, 103]:
    print(n)
{% endcodeblock %}

Now, let's run the code and see the bytecodes we get.

{% codeblock lang:bash %}
python3 bytecode.py print_numbers.py
{% endcodeblock %}

Output:

{% codeblock %}
  1           0 SETUP_LOOP              20 (to 22)
              2 LOAD_CONST               4 ((101, 102, 103))
              4 GET_ITER
        >>    6 FOR_ITER                12 (to 20)
              8 STORE_NAME               0 (n)

  2          10 LOAD_NAME                1 (print)
             12 LOAD_NAME                0 (n)
             14 CALL_FUNCTION            1
             16 POP_TOP
             18 JUMP_ABSOLUTE            6
        >>   20 POP_BLOCK
        >>   22 LOAD_CONST               3 (None)
             24 RETURN_VALUE
None
{% endcodeblock %}

The loop starts on line #4. For every element in the list, we're pushing `print` and `n` onto the stack, calling the function, popping the stack, and repeating the loop. For each of the bytecodes, there's associated C code i.e. `FOR_ITER`, `STORE_NAME`, etc. have associated C code.   

This is a very simple way to run a program and also a very inefficient one. We're repeating the stack operations and jumps over and over again. There's no scope for optimizations like loop unrolling.

### C

In contrast to Python is C. All the C code is compiled to assembly ahead-of-time. Here's a simple C program which will print "EVEN" if a number is even. 

{% codeblock numbers.c lang:c %}
#include<stdio.h>

int main() {
  for(int i = 1; i < 10000; i += 2) {
    if( i % 2 == 0 ) {
      printf("EVEN!");
    } else {
      printf("");
    }
  }
  return 0;
}
{% endcodeblock %}

Next, let's compile this code.

{% codeblock %}
gcc -S numbers.c
{% endcodeblock %}

This will generate `numbers.s`. The assembly is fairly long so I'll just cover the relevant parts.

{% codeblock numbers.s lang:assembly %}
LBB0_1:
        cmpl    $10000, -8(%rbp) 
        jge     LBB0_7
        ...
        idivl   %ecx
        cmpl    $0, %edx
        jne     LBB0_4
        leaq    L_.str(%rip), %rdi
        ...
        callq   _printf
        movl    %eax, -16(%rbp)
        jmp     LBB0_5
LBB0_4:
        leaq    L_.str.1(%rip), %rdi
        movb    $0, %al
        callq   _printf
        ...
LBB0_5:
        jmp     LBB0_6
LBB0_6:
        ...
        addl    $2, %eax
        ...
        jmp     LBB0_1
        ...
LBB0_7:
        ...
        retq
        ...
L_.str:
        .asciz  "EVEN!"
L_.str.1:
        .space  1
{% endcodeblock %}

Lines #2 - #3 show that if we've reached the limit of 10k, we'll jump to `LBB0_7` and the program ends.   
If not, on line #5 we perform a signed division (`idivl`) and check if it is not zero. If it is not zero, we jump to `LBB0_4` and print `L_.str.1` which is just a whitespace.  

We will always end up making this jump because we'll never reach the condition where we have an even number. This is the problem with ahead-of-time compilation where you cannot speculate what the data is going to be and therefore you have to be ready to handle all the possibilities.  

### JVM JIT

JVM JIT combines the best of both the worlds. When you execute your program the first time, the bytecodes are interpreted. As the code continues to execute, JVM collects statistics about it and the more frequently used code ("hot" code) is compiled to assembly. In addition, there are optimizations like loop unrolling. Loop unrolling looks like this:

{% codeblock lang:java %}
// Plain ol' loop
for(i = 0; i < 3; i++) {
    System.out.println(arr[i]);
}

// Unrolled
System.out.println(arr[0]);
System.out.println(arr[1]);
System.out.println(arr[2]);
{% endcodeblock %}

Unrolling a loop helps avoid jumps and thus makes execution faster. 

Also, since JVM collects statistics about code, it can make optimizations on the fly. For example, in the case where an even number is never reached, JVM can generate assembly code that'll only have the `else` part of the branch.

## Conclusion

JVM does some fairly intersting optimizations under the hood. The aim of this series of posts is to cover as much of this as possible. We'll start simple and build upon this as we go. 
