---
layout: post
title: "Performance Tips"
date: 2014-05-16 11:28:00 +0800
comments: true
categories: [Android]
---
> 注：此为毕业设计中学院要求的翻译与自己所做毕设相关且不少于2万字符英文原始资料的任务，由于自己毕设做的是Android方面的开发，所以决定翻译一下Android官方文档中Training和API Guides中的部分内容。由于水平有限，如有错误，望理解。

##[原文链接](http://developer.android.com/training/articles/perf-tips.html)
<br/>

这份文档主要涉及当结合使用一些微优化时可以提高应用程序的整体性能，但让这些优化给你的应用程序带来显著的性能影响是不可能的。选择正确的算法与数据结构应该总是你首先要考虑的，但这些内容在本份文档的范围之外。你可以用这份文档中所写的这些提示作为通用的编程技巧，而且为了通用的代码效率你也可以融入到你的编程习惯当中。

编写高效代码有两条基本规则：

* 不要做你不需要做的工作。

* 如果能避免分配内存就不要分配内存。

当微优化一个Android应用程序时面临最棘手的问题之一就是你的应用程序肯定要运行在多种类型的硬件上。不同版本的虚拟机运行在不同的处理器上的运行速度也是不同的。甚至不是一般的情况例如简单的说“X设备比Y设备快（慢）F倍”以及从一个设备上得到的结果扩展到其它设备上。特别是在模拟器上的测试几乎不会告诉你什么关于在任何设备上的性能。在带与不带JIT的设备之间也有巨大的差异：对带JIT设备来说是最好的代码但在不带JIT的设备上不总是最好的。

为确保你的应用在各种各样的设备上表现良好，请确保你的各级代码是高效的并致力优化应用的性能。

### 避免创建不必要的对象

创建对象从来不是不需要代价的。即使带有线程临时对象分配池的分代垃圾回收器可以使分配时的代价低廉一点，但分配内存怎么都比不分配内存的代价昂贵。

如果你在你的应用里分配更多的对象，你就会强制定期的垃圾收集，创造一种“打嗝”的用户体验。在Android 2.3版本引入的并发垃圾收集器有所补救，但应始终避免不必要的工作。

因此，你应该避免创建你不需要的对象实例。一些可以起到帮助的东西的例子：

* 如果你有一个方法返回一个字符串，你知道它的结果无论如何总是被追加在一个[`StringBuffer`](http://developer.android.com/reference/java/lang/StringBuffer.html)后面，改变你的签名和实现使该函数可以直接追加，而不是创建一个短期的临时对象。

* 当从一组输入数据提取字符串，尝试返回原始数据的一个子串而不是创建它的一个副本时，你会创建一个新的String对象，但它会和这个数据共享`char[]`。(做一个权衡就是如果你只使用原始输入的一小部分，在这种方式下你得在内存处处保留着这份原始数据。)

一个有些更激进的想法是切片多维数组转换成并行的单个一维数组：

* `int`数组比`Integer`对象的数组更好，但这也推广得到一个事实，即整数的两个平行阵列也比`(int, int)`的对象数组的效率高很多。这同样适用于基本类型的任意组合。

* 如果你需要实现一个存储着`(Foo, Bar)`元组对象的容器，要记住，两个平行的`Foo[]`和`Bar[]`数组通常要比自定义的`(Foo, Bar)`对象的单个数组更好。（当然，唯一的例外是当你设计一个让其它代码来访问的API时，在速度方面做一个小的妥协通常更好来达到良好的API设计的目的。）

一般来说，尽可能地避免创建短期的临时对象。越少的对象创建意味着越少次数的垃圾收集，这对用户体验有直接的影响。

### 多使用静态的方法和属性

如果你不需要访问一个对象的字段，请确保你的方法是静态的。这样将会使调用大约有15%~20%的速度提升。这也是一个很好的习惯，因为你可以从方法签名告诉调用者调用该方法不能改变对象的状态。

### 将常量声明为Static和Final类型

思考下面的在类顶部的声明：
{% codeblock lang:java %}
static int intVal = 42;
static String strVal = "Hello, world!";
{% endcodeblock %}

当一个类第一次被使用时编译器生成一个叫做`<clinit>`的类初始化方法会被执行。该方法存储值42到变量`intVal`里，并为变量`strVal`从类文件字符串常量表里提取一个引用。当这些值被引用后，它们能用字段查找来访问。

> 注： 此优化只适用于原始类型和`String`常量，不是任意的引用类型。不过，尽可能地声明常量为`static final`是很好的做法。

### 避免内部的`Getters/Setters`

在本地语言例如C++很常见使用getters(`i = getCount()`)而不是直接访问字段`(i = mCount)`。这在C++里是一个很好的习惯并且也经常在其它面向对象语言例如C#和Java中实践，因为编译器通常是内联访问，如果你需要限制或调试字段访问你可以在任何时候添加代码。

然而，这在Android上不是一个很好的办法。虚方法的调用通常是非常耗时的，远远超过了实例字段查找。遵循共同的面向对象的编程习惯并有getter和setter的公有接口是合理的，但在一个类中你应该直接访问字段。

如果没有JIT，直接字段访问大约是调用一个微不足道的getter的3倍快。有了JIT之后(直接字段访问像访问本地一样快捷)直接字段访问大约是调用一个微不足道的getter的7倍快。

> 请注意，如果你使用了**ProGuard**，你可以两全其美因为**ProGuard**为你提供了内联访问。

### 使用增强的For循环语法

增强的`for`循环(有时也被称为"for-each"循环)，可用于实现了`Iterable`接口的集合和数组。如果使用集合，一个迭代器会被分配出来让接口调用`hasNext()`和`next()`方法。如果使用`ArrayList`，一个手写的计数循环大约是它的3倍快(不管有没有使用JIT)，但是对其他集合来说，增强的for循环语法完全等同于明确的迭代器的用法。

遍历一个数组通常有以下几种选择：

{% codeblock lang:java %}
static class Foo {
    int mSplat;
}

Foo[] mArray = ...

public void zero() {
    int sum = 0;
    for (int i = 0; i < mArray.length; ++i) {
        sum += mArray[i].mSplat;
    }
}

public void one() {
    int sum = 0;
    Foo[] localArray = mArray;
    int len = localArray.length;

    for (int i = 0; i < len; ++i) {
        sum += localArray[i].mSplat;
    }
}

public void two() {
    int sum = 0;
    for (Foo a : mArray) {
        sum += a.mSplat;
    }
}
{% endcodeblock %}

`zero()`方法是最慢的。因为JIT还不能优化掉每一次迭代循环时得到数组长度的代价。

`one()`方法是较快的。它把所有东西都放进了本地变量中从而避免了查找。仅仅数组长度提供了性能优势。

`two()`方法在没有JIT的设备上是最快的，在拥有JIT的设备上和`one()`方法没什么区别。它采用了Java编程语言1.5版本中引入的增强的for循环语法。

因此，你应该采用增强的for循环作为默认用法，但对性能关键的`ArrayList`迭代来说考虑用手写的技术循环。

> 提示：参见Josh Bloch's Effective Java中的第46条.

### 考虑用私有内部类包访问而不是私有访问

思考下面的类定义：

{% codeblock lang:java %}
public class Foo {
    private class Inner {
        void stuff() {
            Foo.this.doStuff(Foo.this.mValue);
        }
    }

    private int mValue;

    public void run() {
        Inner in = new Inner();
        mValue = 27;
        in.stuff();
    }

    private void doStuff(int value) {
        System.out.println("Value is " + value);
    }
}
{% endcodeblock %}

这里的关键是我们定义了一个私有内部类(`Foo$Inner`)直接访问外部类的私有方法和私有实例字段。这是合法的，并且如预期代码打印出了“Value is 27”。

问题是虚拟机认为从`Foo$Inner`访问Foo的私有成员是非法的因为`Foo`和`Foo$Inner`是不同的类，即使Java语言允许内部类访问外部类的私有成员。要弥补差距，编译器生成了一对合成方法：

{% codeblock lang:java %}
/*package*/ static int Foo.access$100(Foo foo) {
    return foo.mValue;
}
/*package*/ static void Foo.access$200(Foo foo, int value) {
    foo.doStuff(value);
}
{% endcodeblock %}

内部类会调用这些静态方法在它任何需要访问外部类`mValue`字段或者调用外部类`doStuff()`方法的时候。这句话的意思是上面的代码真正归结到你通过访问方法访问成员字段的情况。前面我们谈到了访问器是如何比直接字段访问慢的，所以这是一个特定的语言习语造成的“隐形”性能损失的例子。

如果你在性能热点上使用这样的代码，你可以通过声明内部类访问的字段和方法为包访问而不是私有访问来避免开销。不幸的是这意味着字段可以直接被其它类在同一个包中被访问，所以你不应该在公共API中使用这种方式。

### 避免使用浮点数

作为一个经验法则，在Android设备上浮点数比整数慢2倍。

在速度方面，`float`和`double`在更现代的硬件上没有区别。空间方面，double是float的两倍大。在台式机上，假设空间不是问题，你应该更倾向于用`double`而不是`float`。

此外，即使是整数，一些处理器具有硬件乘法但缺少硬件除法。在这种情况下，如果你是正在设计哈希表或做大量数学运算的人整数除法和取模操作在软件中的运行是你应该考虑的东西。

### 了解和使用库

除了所有常用的更喜欢用库代码而不是你自己的代码的理由外，记住，系统是可以自由更换调用库函数与手写代码的汇编，这可能比JIT为相应的Java生成的最好的代码要更好。这里典型的例子是`String.indexOf()`和相关的API，Dalvik用内联征替换了。类似的， 在带有JIT的Nexus One上`System.arraycopy()`方法大约是手写代码的循环的9倍快。

> 提示：参见Josh Bloch's Effective Java第47条。

### 谨慎使用本地方法

使用Android NDK的本地方法开发你的应用时不一定比用Java语言更有效率。一方面，有与Java到本地的过渡相关的成本，以及JIT不能跨界优化。如果你正在分配本地资源(本地堆上的内存，文件描述符或其它任何东西)，很明显地它在安排及时收集这些资源上更困难。你还需要为每个架构编译运行之上的代码（而不是依赖于拥有JIT）。你甚至可能为了考虑相同的架构要编译多个版本：为G1的ARM处理器编译的本地代码在Nexus One上的ARM处理器上不能充分利用，为Nexus One的ARM处理器编译的代码不能在G1的ARM处理器上运行。

本地代码主要有用的方面是当你有一个想要移植到Android的存在的本地代码库，而不是为了加速你用Java语言写的Android应用的某些部分。

如果你确实需要使用本地代码，你应该阅读我们的[JNI提示](http://developer.android.com/training/articles/perf-jni.html)。

> 参加Josh Bloch's Effective Java第54条。

### 性能神话

在没有JIT的设备上，通过一个确切类型的变量调用方法比接口调用方法更高效这是事实(因此，例如通过`HashMap map`比`Map map`调用方法更快捷，即使两种情况中的map类型都是`HashMap`)。但这种情况不是2倍慢的情况，真实的差别更像是6%。此外，JIT使这两种调用没有什么区别了。

在没有JIT的设备上，缓存字段访问比重复访问字段快20%。有了JIT，字段访问的成本约等于本地访问，所以这不是一个值得优化的地方除非你觉得它使你的代码更易于阅读。(final, static 和 static final字段也是如此。)

### 保持测量

在开始优化之前，确保你有需要解决的问题。确保你能精确地测量你现有的性能，否则你将无法衡量你所尝试的选择的收益。

本文档提出的每项主张都有一个基准支持。这些基准的资源都可以在[code.google.com "dalvik" project](code.google.com "dalvik" project)中找到。

这些基准是用Caliper microbenchmarking Java框架建立的。很难得到正确的微基准，所以Caliper用自己的方式为你做这项很艰难的事情，甚至检测出一些你没有测量你认为你在测量的东西。(因为，比如说虚拟机已经成功地优化了所有的代码了。)我们强烈建议你有Caliper来运行你自己的微基准。

你可能发现`TraceView`对分析很有用，但要认识到它目前是禁用JIT的，可能导致由JIT优化代码来的时间被弄错这一点很重要。尤其重要的是在通过TraceView数据的建议作出改变之后要确保产生的代码在没有TraceView时运行确实运行的更快。

如需更多分析和调试应用程序的帮助，请参阅下列文档:

* [Profiling with Traceview and dmtracedump](http://developer.android.com/tools/debugging/debugging-tracing.html)
* [Analysing Display and Performance with Systrace](http://developer.android.com/tools/debugging/systrace.html)
    





















