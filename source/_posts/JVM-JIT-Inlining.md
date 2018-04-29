---
title: JVM JIT - Inlining
tags:
  - compilers
date: 2018-04-29 10:34:32
---


In the previous post we looked at how interpreted and compiled languages work. To recap, an interpreter works by generating assembly code for every bytecode it encounters. This is a very simple way to execute a program and also a very slow one. It ends up redoing a lot of translation from bytecode to assembly. Also, this simplistic approach means that the interpreter cannot do optimizations as it executes the bytecodes. Then there are compilers which produce assembly ahead-of-time. This overcomes having to generate assembly again and again but once the assembly is generated it cannot be changed on the fly.  

JVM comes with both an interpreter and a compiler. When the execution of the code begins, the bytecodes are interpreted. For the sake of this series, I'll be looking at Oracle HotSpot JVM which looks for "hot spots" in the code as the bytecodes get interpreted. These are the parts of the code which are most frequently executed and the performance of the application depends on these. Once the code is identified as "hot", JVM can go from interpreting the code to compiling it to assembly i.e. the code is compiled "just-in-time". In addition, since the code is being profiled as it is run, the compiled code is optimized. 

In this post we'll look at one such optimization: inlining.

## Inlining

Inlining is an optimization where the call to a method is replaced by the body of the called method i.e. at the call site, the caller and the callee are melded together. When a method is called, the JVM has to push a stack frame so that it can resume from where it left off after the called method has finished executing. Inlining improves performance since JVM will not have to push a stack frame.  

I'll start with a simple example to demonstrate how inlining works. 

{% codeblock Inline.java lang:java %}
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

Next, let's compile and run the code. 

{% codeblock %}
java -XX:+PrintCompilation -XX:+UnlockDiagnosticVMOptions -XX:+PrintInlining Inline 100000
{% endcodeblock %}

Output:

{% codeblock %}
     63    1             Inline::inline1 (4 bytes)
     63    2             Inline::inline2 (4 bytes)
                            @ 0   Inline::inline3 (2 bytes)   inline (hot)
                            @ 0   Inline::inline2 (4 bytes)   inline (hot)
                              @ 0   Inline::inline3 (2 bytes)   inline (hot)
     66    3             Inline::inline3 (2 bytes)
     66    4 %           Inline::main @ 9 (28 bytes)
                            @ 16   Inline::inline1 (4 bytes)   inline (hot)
                              @ 0   Inline::inline2 (4 bytes)   inline (hot)
                                @ 0   Inline::inline3 (2 bytes)   inline (hot)
     66    4 %           Inline::main @ -2 (28 bytes)   made not entrant
{% endcodeblock %}

Line #1 shows that `inline1` was compiled to assembly. Line #2 and Line #6 show that `inline2` and `inline3` were also compiled to assembly. Line #3 to line #5 show inlining. We can see that `inline3` was merged into `inline2`. Similarly, line #8 and #9 show that `inline2` was merged into `inline1`. So basically, all the methods were inlined into `inline1`. This means that once a certain threshold is crossed, we'll no longer be making methods calls at all. This gives a significant performance boost.

 
### Which flags control inlining?

When you run a Java program, you can view the flags with which it ran using `-XX:+PrintFlagsFinal`. Let's do that and look at a few flags of interest.

{% codeblock %}
java -XX:+PrintFlagsFinal Inline 10000
{% endcodeblock %}

You'll see a bunch of flags and their default values. The ones we are interested in are `CompileThreshold`, `MaxInlineLevel`, `MaxInlineSize`, and `FreqInlineSize`.   

`CompileThreshold` is the number of invocations before compiling a method to native.   
`MaxInlineLevel` is a limit on how deep you'd go before you stop inlining. The default value is `9`. This means if we had method calls like `inline1` ⟶ `inline2` ... ⟶ `inline20`, we'd only inline upto `inline10`. There after, we'd invoke `inline11`.  
`MaxInlineSize` decides the maximum size of a method, in bytecodes, to be inlined. The default value is `35`. This means that if the method to be inlined has mre than `35` bytecodes, it will not be inlined.  
`FreqInlineSize`, in contrast, decides the maximum size of a hot method, in bytecodes, to be inlined. This is a platform-dependent value and on my machine it is `325`.  

You can tweak these flags to change how inlining behaves for your program.  

### What is On Stack Replacement (OSR)?  

When we make a method call, JVM pushes a stack frame. When a method is deemed hot, the JVM replaces the intrepreted version with the compiled version by replacing the old stack frame with a new one. This is done while the method is running. We saw OSR being indicated in our example. The `%` indicates that an OSR was made.

{% codeblock %}
 66    4 %           Inline::main @ -2 (28 bytes)   made not entrant
{% endcodeblock %}

Let's write some code to see OSR in action once again.  

{% codeblock lang:java %}
import java.lang.ref.WeakReference;

public class OSR {
    public static void main(String[] args) {
        Object unused = new Object();
        WeakReference<Object> ref = new WeakReference<>(unused);

        int x = 0;

        while( ref.get() != null ) {
            x += 1;
            System.out.println(x);
        }

        System.out.println("Finished!");
    }
}
{% endcodeblock %}

So this is a loop that will never terminate, right? Let's run the program and see.

{% codeblock %}
...
434062
  16828   59 %           OSR::main @ -2 (48 bytes)   made not entrant
Finished!
{% endcodeblock %}

What just happened? When the JVM decided to perform an OSR, it saw that there was no use for the `unused` object and decided to set it to null, causing the `WeakReference` to return `null` and thus breaking the loop. When an OSR is performed, the method that is invoked doesn't restart execution from the start. Rather, It continues from the "back-edge". In our case, it would be the loop. Since the JVM saw that there was no use for the `unused` object after this back-edge, it was removed and the loop could terminate.   

Being able to resume execution from the back-edge is very efficient. This means that once a method has been compiled to native code it can be used rightaway rather than at the next invocation of the method.  

## Conclusion  

To recap, we saw how JVM inlines code. Fusing the caller and the callee provides for improved performance since the overhead of method dispatch is avoided. We saw the flags which control inlining and we saw how JVM performs OSR.  

Inlining is a very useful optimization because it forms the basis for other optimizations like escape analysis and dead code elimination.
