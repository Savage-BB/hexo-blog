---
title: JVM（三） 垃圾回收
date: 2023-04-05
description: 如何自动回收对象，Java程序为什么会出现“stop the world”
categories: Java
tags: 
  - JVM
---
# JVM垃圾回收（三）

我们前面提到，Java会自动管理和释放内存，它不像C/C++那样要求我们手动管理内存，JVM提供了一套全自动的内存管理机制，当一个Java对象不再用到时，JVM会自动将其进行回收并释放内存，那么对象所占内存在什么时候被回收，如何判定对象可以被回收，以及如何去进行回收工作也是JVM需要关注的问题。

### 对象存活判定

首先我们来套讨论第一个问题，也就是：对象在什么情况下可以被判定为不再使用已经可以回收了？这里就需要提到以下几种垃圾回收算法了。

### **引用计数法**

如果我们要经常操作一个对象，那么首先一定会创建一个引用变量

```java
String str = "what the fuck";
```

这个算法的规则如下：

- 每个对象都包含一个 引用计数器，用于存放引用计数（其实就是存放被引用的次数）
- 每当有一个地方引用此对象时，引用计数+1
- 当引用失效（ 比如离开了局部变量的作用域或是引用被设定为null）时，引用计数-1
- 当引用计数为0时，表示此对象不可能再被使用

当引用为0时我们已经没有任何方法可以得到此对象的引用了，所以它无法解决循环引用的问题

```java
    
public class Main {
	public static void main(String[] args) {
		Test a = new Test();
		Test b = new Test();
	  a.another = b;
	  b.another = a;
	
	  //这里直接把a和b赋值为null，这样前面的两个对象我们不可能再得到了
	  a = b = null;
	}
	
	private static class Test{
	    Test another;
	}
}
```

按照引用计数算法，那么当出现以上情况时，虽然我们无法在得到此对象的引用了，并且此对象我们也无需再使用，但是由于这两个对象直接存在相互引用的情况，那么引用计数器的值将会永远是1，但是实际上此对象已经没有任何用途了。

### **可达性分析算法**

这种算法采用了类似于树结构的搜索机制**。**首先每个对象的引用都有机会成为树的根节点（GC Roots），可以被选定作为根节点条件如下：

- 位于虚拟机栈的栈帧中的本地变量表中所引用到的对象（其实就是我们方法中的局部变量）
- 本地方法栈中JNI引用的对象
- 类的静态成员变量引用的对象
- 方法区中，常量池里面引用的对象，比如我们之前提到的String类型对象
- 被添加了锁的对象（比如synchronized关键字）
- 虚拟机内部需要用到的对象

一旦已经存在的根节点不满足存在的条件时，那么根节点与对象之间的连接将被断开。此时虽然对象1仍存在对其他对象的引用，但是由于其没有任何根节点引用，所以此对象即可被判定为不再使用。

这样就能很好地解决我们刚刚提到的循环引用问题，我们再来重现一下出现循环引用的情况：

可以看到，对象1和对象2依然是存在循环引用的，但是只要他们各自的GC Roots断开，那么就会被判定为垃圾对象。

所以，我们最后进行一下总结：如果某个对象无法到达任何GC Roots，则证明此对象是不可能再被使用的。

**最终判定**
虽然在经历了可达性分析算法之后基本可能判定哪些对象能够被回收，但是并不代表此对象一定会被回收，我们依然可以在最终判定阶段对其进行挽留。

还记得我们之前在讲解`Object`类时提到的`finalize()`方法吗？

```java
/**
- Called by the garbage collector on an object when garbage collection
- determines that there are no more references to the object.
- A subclass overrides the {@code finalize} method to dispose of
- system resources or to perform other cleanup.

	protected void finalize() throws Throwable { }
**/
```

此方法正是最终判定方法，如果子类重写了此方法，那么子类对象在被判定为可回收时，会进行二次确认，也就是执行`finalize()`方法，而在此方法中，当前对象是完全有可能重新建立GC Roots的！

所以，如果在二次确认后对象不满足可回收的条件，那么此对象不会被回收，巧妙地逃过了垃圾回收的命运。比如下面这个例子：

```java
public class Main {
	private static Test a;
	public static void main(String[] args) throws InterruptedException {
		a = new Test();
    //这里直接把a赋值为null，这样前面的对象我们不可能再得到了
    a  = null;

    //手动申请执行垃圾回收操作（注意只是申请，并不一定会执行，但是一般情况下都会执行）
    System.gc();

    //等垃圾回收一下()
    Thread.sleep(1000);

    //我们来看看a有没有被回收
    System.out.println(a);
}

private static class Test{
    @Override
    protected void finalize() throws Throwable {
        System.out.println(this+" 开始了它的救赎之路！");
        a = this;
    }
}

```

注意`finalize()`方法并不是在主线程调用的，而是虚拟机自动建立的一个低优先级的**Finalizer**线程（正是因为优先级比较低，所以前面才需要等待1秒钟）进行处理，我们可以稍微修改一下看看：

```java
private static class Test{
	@Override
	protected void finalize() throws Throwable {
		System.out.println(Thread.currentThread());
		a = this;
	}
}

//输出结果
Thread[Finalizer,8,system]
com.test.Main$Test@232204a1
```

而且，同一个对象的`finalize()`方法只会有一次调用机会，也就是说，如果我们连续两次这样操作，那么第二次，对象不会再调用`finalize()`方法，没有建立CG Root 的机会了，所以第二次必定被回收

```java

public static void main(String[] args) throws InterruptedException {
	a = new Test();
	
	//这里直接把a赋值为null，这样前面的对象我们不可能再得到了
	a  = null;
	
	//手动申请执行垃圾回收操作（注意只是申请，并不一定会执行，但是一般情况下都会执行）
	System.gc();
	
	//等垃圾回收一下
	Thread.sleep(1000);
	System.out.println(a);
	
	//这里直接把a赋值为null，这样前面的对象我们不可能再得到了
	a  = null;
	
	//手动申请执行垃圾回收操作（注意只是申请，并不一定会执行，但是一般情况下都会执行）
	System.gc();
	
	//等垃圾回收一下
	Thread.sleep(1000);
	System.out.println(a);
}
```

当然，`finalize()`方法也并不是专门防止对象被回收的，我们可以使用它来释放一些程序使用中的资源等。

### 垃圾回收算法

前面我们介绍了对象存活判定算法，现在我们已经可以准确地知道堆中的哪些对象可以被回收了，那么，接下来就该考虑如何对对象进行回收了，垃圾收集器会不定期地检查堆中的对象，查看它们是否满足被回收的条件。我们该如何对这些对象进行回收，是一个一个判断是否需要回收吗？

**分代收集机制**
实际上，如果我们对堆中的每一个对象都依次判断是否需要回收，这样的效率其实是很低的，那么有没有更好地回收机制呢？

第一步，我们可以对堆中的对象进行分代管理。

比如某些对象，在多次垃圾回收时，都未被判定为可回收对象，我们完全可以将这一部分对象放在一起，并让垃圾收集器减少回收此区域对象的频率，这样就能很好地提高垃圾回收的效率了。

因此，Java虚拟机将堆内存划分为新生代、老年代和永久代（其中永久代是HotSpot虚拟机特有的概念，在JDK8之前方法区实际上就是采用的永久代作为实现，而在JDK8之后，方法区由元空间实现，并且使用的是本地内存，容量大小取决于物理机实际大小，之后会详细介绍）这里我们主要讨论的是新生代和老年代。

不同的分代内存回收机制也存在一些不同之处，在HotSpot虚拟机中，新生代被划分为三块，一块较大的Eden空间和两块较小的Survivor空间，默认比例为8：1：1，老年代的GC评率相对较低，永久代一般存放类信息等（其实就是方法区的实现）。那么它是如何运作的呢？

首先，所有新创建的对象，在一开始都会进入到新生代的Eden区（如果是大对象会被直接丢进老年代），在进行新生代区域的垃圾回收时，首先会对所有新生代区域的对象进行扫描，并回收那些不再使用对象。

接着，在一次垃圾回收之后，Eden区域没有被回收的对象，会进入到Survivor区。在一开始From和To都是空的，而GC之后，所有Eden区域存活的对象都会直接被放入到From区，最后From和To会发生一次交换，也就是说目前存放我们对象的From区，变为To区，而To区变为From区。

然后就是下一次垃圾回收了，操作与上面是一样的，不过这时由于我们From区域中已经存在对象了，所以，在Eden区的存活对象复制到From区之后，所有To区域中的对象会进行年龄判定（每经历一轮GC年龄+1，如果对象的年龄大于默认值为15，那么会直接进入到老年代，否则移动到From区）

最后像上面一样交换To区和From区，之后不断重复以上步骤。

垃圾收集分为
**部分收集（Partial GC）**：

- Minor GC - 次要垃圾回收，主要进行新生代区域的垃圾收集
- Major GC - 主要垃圾回收，主要进行老年代的垃圾收集

**整堆收集（Full GC）**

- Full GC - 完全垃圾回收，对整个Java堆内存和方法区进行垃圾回收

我们可以添加启动参数来查看JVM的GC日志：

```java
public class Main {
	public static void main(String[] args) {
		Object o = new Object();
		o = null;
		System.gc();
	}
}
```

```java
[GC (System.gc()) [PSYoungGen: 2621K->528K(76288K)] 2621K->528K(251392K), 0.0006874 secs] [Times: user=0.01 sys=0.01, real=0.00 secs]
[Full GC (System.gc()) [PSYoungGen: 528K->0K(76288K)] [ParOldGen: 0K->332K(175104K)] 528K->332K(251392K), [Metaspace: 3073K->3073K(1056768K)], 0.0022693 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
Heap
PSYoungGen      total 76288K, used 3277K [0x000000076ab00000, 0x0000000770000000, 0x00000007c0000000)
eden space 65536K, 5% used [0x000000076ab00000,0x000000076ae334d8,0x000000076eb00000)
from space 10752K, 0% used [0x000000076eb00000,0x000000076eb00000,0x000000076f580000)
to   space 10752K, 0% used [0x000000076f580000,0x000000076f580000,0x0000000770000000)
ParOldGen       total 175104K, used 332K [0x00000006c0000000, 0x00000006cab00000, 0x000000076ab00000)
object space 175104K, 0% used [0x00000006c0000000,0x00000006c00532d8,0x00000006cab00000)
Metaspace       used 3096K, capacity 4496K, committed 4864K, reserved 1056768K
class space    used 333K, capacity 388K, committed 512K, reserved 1048576K
```

GC日志中记录了以下内容，通过分析 GC 日志，可以了解到应用程序的内存使用情况，发现内存泄漏问题，评估 GC 的性能表现，调整 JVM 参数来优化应用程序的性能等

- **GC 类型和触发原因：** 日志中会指明发生了哪种类型的 GC，比如新生代GC（Minor GC）、老年代GC（Major GC或Full GC），以及触发 GC 的原因，比如年轻代满、老年代满、系统调用等。
- **GC 开始和结束时间戳：** 记录了 GC 开始和结束的时间点，可以用来计算 GC 持续的时间。
- **前后堆内存情况：** 记录了 GC 执行前后堆内存的使用情况，包括新生代、老年代、永久代（如果存在）的使用量和总量，以及 GC 前后各代的使用率。
- **暂停时间：** 记录了 GC 暂停的时间，即应用程序因进行垃圾收集而停顿的时间。
- **日志标记：** 有些 GC 日志中会标记一些重要的事件，比如 Young GC、Full GC、System GC 等，以便后续分析。
- **相关参数：** 记录了 JVM 启动参数和 GC 相关参数的设置，比如堆大小、新生代和老年代的大小、GC 算法等。

**空间分配担保**
我们可以思考一下，有没有这样一种极端情况（正常情况下新生代的回收率是很高的，所以说不用太担心会经常出现这种问题），在一次GC后，新生代Eden区仍然存在大量的对象（因为GC之后存活对象会进入到一个Survivor区，但是很明显这时已经超出Survivor区的容量了，肯定是装不下的）那么现在该怎么办？

这时就需要用到空间分配担保机制了，可以把Survivor区无法容纳的对象直接送到老年代，让老年代进行分配担保（当然老年代也得装得下才行）在现实生活中，贷款会指定担保人，就是当借款人还不起钱的时候由担保人来还钱。

当新生代无法容纳更多的的对象时，可以把新生代中的对象移动到老年代中，这样新生代就腾出了空间来容纳更多的对象。

那既然新生代装不下就丢给老年代，那么要是老年代也装不下新生代的数据呢？这时，老年代肯定担保人是当不成了，那么这样的话，首先会判断一下之前的每次垃圾回收进入老年代的平均大小是否小于当前老年代的剩余空间，如果小于，那么说明也许可以放得下（不过也仅仅是也许，依然有可能放不下，因为判断的实际上只是平均值，万一这一次突然非常大呢），否则，会先来一次Full GC，进行一次大规模垃圾回收，来尝试腾出空间，再次判断老年代是否有空间存放，要是还是装不下，直接抛出OOM错误，摆烂。

### 标记-清除算法

前面我们已经了解了整个堆内存实际上是以分代收集机制为主，但是依然没有讲到具体的收集过程，那么，具体的回收过程又是什么样的呢？首先我们来了解一下最古老的标记-清除算法。

首先标记出所有需要回收的对象，然后再依次回收掉被标记的对象，或是标记出所有不需要回收的对象，只回收未标记的对象。实际上这种算法是非常基础的，并且最易于理解的（这里对象我就以一个方框代替了，当然实际上存放是我们前说到的GC Roots形式）。

虽然此方法非常简单，但是缺点也是非常明显的 ，首先如果内存中存在大量的对象，那么可能就会存在大量的标记，并且大规模进行清除。并且一次标记清除之后，连续的内存空间可能会出现许许多多的空隙，碎片化会导致连续内存空间利用率降低。

### 标记-复制算法

既然标记清除算法在面对大量对象时效率低，那么我们可以采用标记-复制算法。

标记复制算法，实际上就是将内存区域划分为大小相同的两块区域，每次只使用其中的一块区域，每次垃圾回收结束后，将所有存活的对象全部复制到另一块区域中，并一次性清空当前区域。虽然浪费了一些时间进行复制操作，但是这样能够很好地解决对象大面积回收后空间碎片化严重的问题。

这种算法就非常适用于新生代（因为新生代的回收效率极高，一般不会留下太多的对象）的垃圾回收，而我们之前所说的新生代Survivor区其实就是这个思路，包括8:1:1的比例也正是为了对标记复制算法进行优化而采取的。

### 标记-整理算法

虽然标记-复制算法能够很好地应对新生代高回收率的场景，但是放到老年代，它就显得很鸡肋了。我们知道，一般长期都回收不到的对象，才有机会进入到老年代，所以老年代一般都是些钉子户，可能一次GC后，仍然存留很多对象。而标记复制算法会在GC后完整复制整个区域内容，并且会折损50%的区域，显然这并不适用于老年代。

那么我们能否这样，在标记所有待回收对象之后，不急着去进行回收操作，而是将所有待回收的对象整齐排列在一段内存空间中，而需要回收的对象全部往后丢，这样，前半部分的所有对象都是无需进行回收的，而后半部分直接一次性清除即可。

虽然这样能保证内存空间充分使用，并且也没有标记复制算法那么繁杂，但是缺点也是显而易见的，它的效率比前两者都低。甚至，由于需要修改对象在内存中的位置，此时程序必须要暂停才可以，在极端情况下，可能会导致整个程序发生停顿（被称为“Stop The World”）。

所以，我们可以将标记清除算法和标记整理算法混合使用，在内存空间还不是很凌乱的时候，采用标记清除算法其实是没有多大问题的，当内存空间凌乱到一定程度后，我们可以进行一次标记整理算法。

### 垃圾收集器实现

接着我们就要看看具体有哪些垃圾回收器的实现了。我们可以自由地为新生代和老年代选择更适合它们的收集器。

**Serial收集器**
这款垃圾收集器也是元老级别的收集器了，在JDK1.3.1之前，是虚拟机新生代区域收集器的唯一选择。这是一款单线程的垃圾收集器，也就是说，当开始进行垃圾回收时，需要暂停所有的线程，直到垃圾收集工作结束。它的新生代收集算法采用的是标记复制算法，老年代采用的是标记整理算法。

可以看到，当进入到垃圾回收阶段时，所有的用户线程必须等待GC线程完成工作，就相当于你打一把LOL 40分钟，中途每隔1分钟网络就卡5秒钟，可能这时你正在打团，结果你被物理控制直接在那里站了5秒钟，这确实让人难以接受。

虽然缺点很明显，但是优势也是显而易见的：

设计简单而高效，在用户的桌面应用场景中，内存一般不大，可以在较短时间内完成垃圾收集，只要不频繁发生，使用串行回收器是可以接受的。

所以，在客户端模式（一般用于一些桌面级图形化界面应用程序）下的新生代中，默认垃圾收集器至今依然是Serial收集器。我们可以在java -version中查看默认的客户端模式：

```java
java version "1.8.0_301"
Java(TM) SE Runtime Environment (build 1.8.0_301-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.301-b09, mixed mode) 
```

### 其他引用类型

最后，我们来介绍一下其他引用类型。

**强引用**

我们知道，在Java中，如果变量是一个对象类型的，那么它实际上存放的是对象的引用，但是如果是一个基本类型，那么存放的就是基本类型的值。实际上我们平时代码中类似于：

```java
Object o = new Object()
```

这样的的引用类型，细分之后可以称为强引用。

我们通过前面的学习可以明确，如果方法中存在这样的强引用类型，现在需要回收强引用所指向的对象，那么要么此方法运行结束，要么引用连接断开，否则被引用的对象是无法被判定为可回收的，因为我们说不定后面还要使用它。

所以，当JVM内存空间不足时，JVM宁愿抛出OutOfMemoryError使程序异常终止，也不会靠随意回收具有强引用的“存活”对象来解决内存不足的问题。

**软引用**
除了强引用之外，Java也为我们提供了三种额外的引用类型，软引用不像强引用那样不可回收，当 JVM 认为内存不足时，会去试图回收软引用指向的对象，即JVM 会确保在抛出 OutOfMemoryError 之前，清理软引用指向的对象。当然，如果内存充足，那么是不会轻易被回收的。

我们可以通过以下方式来创建一个软引用：

```java
public class Main {
	public static void main(String[] args) {
		//强引用写法：Object obj = new Object();
		//软引用写法：
		SoftReference<Object> reference = new SoftReference<>(new Object());
		//使用get方法就可以获取到软引用所指向的对象了
		System.out.println(reference.get());
	}
}

```

可以看到软引用还存在一个带队列的构造方法，软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收器回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列中。

**弱引用**
弱引用比软引用的生命周期还要短，在进行垃圾回收时，不管当前内存空间是否充足，都会回收它的内存。

我们可以像这样创建一个弱引用：

```java
public class Main {
	public static void main(String[] args) {
		WeakReference<Object> reference = new WeakReference<>(new Object());
		System.out.println(reference.get());
	}
}
```

使用方法和软引用是差不多的，但是如果我们在这之前手动进行一次GC：

```java
public class Main {
	public static void main(String[] args) {
		SoftReference<Object> softReference = new SoftReference<>(new Object());
		WeakReference<Object> weakReference = new WeakReference<>(new Object());
    //手动GC
    System.gc();

    System.out.println("软引用对象："+softReference.get());
    System.out.println("弱引用对象："+weakReference.get());
  }
}
```

可以看到，弱引用对象直接就被回收了，而软引用对象没有被回收。同样的，它也支持ReferenceQueue，和软引用用法一致，这里就不多做介绍了。

WeakHashMap正是一种类似于弱引用的HashMap类，如果Map中的Key没有其他引用那么此Map会自动丢弃此键值对。

```java
public class Main {
	public static void main(String[] args) {
		Integer a = new Integer(1);
    WeakHashMap<Integer, String> weakHashMap = new WeakHashMap<>();
    weakHashMap.put(a, "yyds");
    System.out.println(weakHashMap);

    a = null;
    System.gc();

    System.out.println(weakHashMap);
}
```

可以看到，当变量a的引用断开后，这时只有WeakHashMap本身对此对象存在引用，所以在GC之后，这个键值对就自动被舍弃了。所以说这玩意，就挺适合拿去做缓存的。

**虚引用（鬼引用）**
虚引用相当于没有引用，随时都有可能会被回收。

看看它的源码，非常简单：

```java
public class PhantomReference<T> extends Reference<T> {
	/**
	 * Returns this reference object's referent.  Because the referent of a
	 * phantom reference is always inaccessible, this method always returns
	 * <code>null</code>.
	 *
	 * @return  <code>null</code>
	 */
	public T get() {
	    return null;
	}
	
	/**
	 * Creates a new phantom reference that refers to the given object and
	 * is registered with the given queue.
	 *
	 * <p> It is possible to create a phantom reference with a <tt>null</tt>
	 * queue, but such a reference is completely useless: Its <tt>get</tt>
	 * method will always return null and, since it does not have a queue, it
	 * will never be enqueued.
	 *
	 * @param referent the object the new phantom reference will refer to
	 * @param q the queue with which the reference is to be registered,
	 *          or <tt>null</tt> if registration is not required
	 */
	public PhantomReference(T referent, ReferenceQueue<? super T> q) {
	    super(referent, q);
	}
}
也就是说我们无论调用多少此get()方法得到的永远都是null，因为虚引用本身就不算是个引用，相当于这个对象不存在任何引用，并且只能使用带队列的构造方法，以便对象被回收时接到通知。
```

最后，Java中4种引用的级别由高到低依次为： 强引用 > 软引用 > 弱引用 > 虚引用

> B站「青空の霞光」
> 

> 《深入理解Java虚拟机》
>