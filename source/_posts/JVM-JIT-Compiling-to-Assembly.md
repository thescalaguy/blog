---
title: JVM JIT - Compiling to Assembly
tags:
  - compilers
date: 2018-05-10 21:14:30
---


In the [previous post](/2018/04/29/JVM-JIT-Inlining/) we saw how JIT inlining works. We also saw how the JVM performs OSR to replace the interpreted version of the method to the compiled version on the fly. In this post we'll dig even deeper and see the assembly code that is generated when the method gets compiled. 

## Prerequisites  

The flag which enables us to see assembly code is `-XX:+PrintAssembly`. However, viewing assembly code does not work out of the box. You'll need to have the disassembler on your path. You'll need to get `hsdis` (HotSpot Disassembler) and build it for your system. There's a prebuilt version available for Mac and that's the one I am going to use. 

{% codeblock %}
git clone https://github.com/liuzhengyang/hsdis.git
{% endcodeblock %}

Once we have that, we'll add it to `LD_LIBRARY_PATH`. 

{% codeblock %}
export LD_LIBRARY_PATH=./hsdis/build/macosx-amd64
{% endcodeblock %}

Now we're all set to see how JVM generates assembly code. 

### Printing assembly code

We'll reuse the same inlining code from last time:  

{% codeblock lang:java %}
public class Inline {
    public static void main(String[] args) {
        long upto = Long.parseLong(args[0]);

        for(int i = 0; i < upto; i++) {
            int x = inline1();
        }
    }

    public static int inline1() {
        return inline2();
    }

    public static int inline2() {
        return inline3();
    }

    public static int inline3() {
        return 4;
    }
}
{% endcodeblock %}

`-XX:+PrintAssembly` is a diagnostic flag so we'll need to unlock JVM's disgnostic options first. Here's how:

{% codeblock %}
java -XX:+UnlockDiagnosticVMOptions -XX:+PrintAssembly Inline 100000
{% endcodeblock %}

This will generate a lot of assembly code. We will, however, look at the assembly code generated for `inline1`.

{% codeblock %}
Decoding compiled method 0x000000010d6095d0:
Code:
[Disassembling for mach='i386:x86-64']
[Entry Point]
[Verified Entry Point]
[Constants]
  # {method} 'inline1' '()I' in 'Inline'
  #           [sp+0x20]  (sp of caller)
  0x000000010d609700: sub    $0x18,%rsp
  0x000000010d609707: mov    %rbp,0x10(%rsp)    ;*synchronization entry
                                                ; - Inline::inline1@-1 (line 11)
  0x000000010d60970c: mov    $0x4,%eax
  0x000000010d609711: add    $0x10,%rsp
  0x000000010d609715: pop    %rbp
  0x000000010d609716: test   %eax,-0x496b71c(%rip)        # 0x0000000108c9e000
                                                ;   {poll_return}
  0x000000010d60971c: retq
  0x000000010d60971d: hlt
  0x000000010d60971e: hlt
  0x000000010d60971f: hlt
[Exception Handler]
[Stub Code]
  0x000000010d609720: jmpq   0x000000010d6050a0  ;   {no_reloc}
[Deopt Handler Code]
  0x000000010d609725: callq  0x000000010d60972a
  0x000000010d60972a: subq   $0x5,(%rsp)
  0x000000010d60972f: jmpq   0x000000010d5deb00  ;   {runtime_call}
  0x000000010d609734: hlt
  0x000000010d609735: hlt
  0x000000010d609736: hlt
  0x000000010d609737: hlt    Decoding compiled method 0x000000010d6064d0:
{% endcodeblock %}

So this is the assembly code that we get when we run the program. It's a lot to grok in one go so let's break it down.  

Line #7 and #8 are self explanatory; they show which method we're looking at. Line #9 and #10 (and #13 to #17) are for thread synchronization. The JVM can get rid of thread synchronization if it sees that there is no need for it (lock eliding) but since we are using `static` methods here, it needs to add code for synchronization. It doesn't know that we only have only one thread running.  

Our actual program is on line #11 where we are moving the value `4` to `%eax` register. This is the register which holds, by convention, the return value for our methods. This shows that the JVM has optimized our code. Our call chain was `inline1` ⟶ `inline2` ⟶ `inline3` and it was `inline3` which returned `4`. However, JVM is smart enough to see that these method calls are superfluous and decided to get rid of them. Very nifty!  

Line #21 to #23 has code to handle exceptions. We know there won't be any exceptions but the JVM doesn't so it has to be prepared to deal with that.  

And finally, there's code to _deoptimize_. In addition to static optimizations, there are some optimizations that the JVM makes which are speculative. This means that the JVM generates assembly code expecting things to go a certain way after it has profiled the interpreted code. However, if the speculation is wrong, the JVM can go back to running the interpreted version.

### Which flags control compilation?

`-XX:CompileThreshold` is the flag which controls the number of call / branch invocations after which the JVM compiles bytecodes to assembly. You can use `-XX:+PrintFlagsFinal` to see the value. By default it is `10000`.

Compiling a method to assembly depends on two factors: the number of times that method has been invoked (method entry counter) and the number of times a loop has been executed (back-edge counter). Once the sum of the two counters is above `CompileThreshold`, the method will be compiled to assembly.  

Maintaining the two counters separately is very useful. If the back-edge counter alone exceeds the threshold, the JVM can compile just the loop (and not the entire method) to assembly. It will perform an OSR and start using the compiled version of the loop while the loop is executing instead of waiting for the next method invocation. When the method is invoked the next time around, it'll use the compiled version of the code.  

So since compiled code is better than interpreted code, and `CompileThreshold` controls when a method will be compiled to assembly, reducing the `CompileThreshold` would mean we have a lot more assembly code.

There is one advantage to reducing the `CompileThreshold` - it will reduce the time taken for the branches / methods to be deemed hot i.e. reduce the JVM warmup time. 

In older JDKs, there was another reason to reduce `CompileThreshold`. The method entry and back-edge counters would decay at every safepoint. This would mean that some methods would not compile to assembly since the counters kept decaying. These are the "lukewarm" methods that never became hot. With JDK 8+, the counters no longer decay at safepoints so there won't be any lukewarm methods.  

In addition, JDK 8+ come with tiered compilation enabled and the `CompileThreshold` is ignored. The idea of there being a "compile threshold", though, does not change. I'm defering the topic of tiered compilation for the sake of simplicity.

### Where is the compiled code stored?  

The compiled code is stored in JVM's code cache. As more methods become hot, the cache starts to get filled. Once the cache is filled, the JVM can no longer compile anything to assembly and will resort to purely interpreteting the bytecodes.   

The size of code cache is platform dependent.  

Also, JVM ensures that the access to cache is optimized. The `hlt` instructions in the assembly code exist for aligning the addresses. It is much more efficient for the CPU to read from even addresses than it is to read from odd addresses in memory. The `hlt` instructions ensure that the code is at an even address in memory.

### Which flags control code cache size?  

There are two flags which are important in setting the code cache size - `InitialCodeCacheSize` and `ReservedCodeCacheSize`. The first flag indicates the code cache size the JVM will start with and the latter indicates the size to which the code cache can grow. With JDK 8+, `ReservedCodeCacheSize` is large enough so you don't need to set it explicitly. On my machine it is `240` MB (5x what it is for Java 7, `48` MB).  

### Conclusion  

The JVM compiles hot code to assembly and stores it at even addresses in it's code cache for faster access. Executing assembly code is much more efficient than interpreting the bytecodes. You don't really need to look at the assembly code generated everyday but knowing what is generated as your code executes gives you an insight into what the JVM does to make your code run faster.
