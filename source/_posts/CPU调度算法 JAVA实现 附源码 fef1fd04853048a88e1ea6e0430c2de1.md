---
title: CPU调度算法 JAVA实现 附源码
date: 2023-04-24 
description: CPU调度算法是操作系统中用于决定哪个进程可以获得CPU时间并在CPU上执行的算法
categories: 操作系统
tags: 
  - 算法
---
# CPU调度算法 JAVA实现 附源码

OS实验课程

## 概述

CPU调度算法是操作系统中用于决定哪个进程可以获得CPU时间并在CPU上执行的算法。常见的CPU调度算法包括：

- 先来先服务（FCFS）：按照作业到达的顺序依次执行，即先到达的作业先执行，后到达的作业后执行。
- 短作业优先（SJF）：选择执行时间最短的作业优先执行，以减少平均等待时间，这种算法有抢占式和非抢占式的。
- 最高优先级优先（Priority Scheduling）：根据作业的优先级确定执行顺序，优先级高的作业先执行。
- 时间片轮转（Round Robin）：将CPU时间划分成多个时间片，每个作业依次执行一个时间片，如果作业在一个时间片内没有执行完，则被放到就绪队列的末尾等待下一次执行。
- 最短剩余时间优先（SRTF）：在SJF的基础上，动态调整作业的执行顺序，选择剩余执行时间最短的作业优先执行。
- 多级反馈队列（Multilevel Feedback Queue Scheduling）：将作业划分成多个队列，根据作业的特性和优先级选择对应的队列进行调度。

这里我用Java实现FCFS、SJF、RR、SRTF、MFQS这几种调度算法，图片只截取了部分运行结果，可以复制源码执行，查看完整打印的运行结果。

### 实现代码

主要有PCB、CPU两个类，PCB（Process Control Block，进程控制块）用来表示进程实体，PCB中保存着进程各种信息。

```java
public class PCB {

    private int pid;   // 进程号

    private char state;// 进程状态,'N'新建,'R'运行,'W'等待CPU(就绪),'T'终止

    private int priority; // 进程优先级,1~3,数字越小优先级越高

    private int arrival; // 到达时间

    private int burst;  // CPU区间时间

    private int remain;  //剩余时间
    
    public void subRest(){
        this.remain--;
    }

    @Override
    public String toString() {
        return "PCB"+this.pid;
    }

    public void initPCB(int i){
        this.pid = i+1;
        this.state = 'N';
        this.priority = (int)(Math.random()*3+1);
        this.arrival = (int)(Math.random()*10);
        this.burst = (int)(Math.random()*10+1);
        this.remain = this.burst;
//        System.out.println("进程"+this.pid+"创建成功");
    }
}
```

CPU类负责执行各种调度算法，内部也有重要的公共属性

```java
public class CPU {
    public static int progressNum = 5;               // 进程数
    public static int time = 0;                      // 计时器用来模拟计算机时钟信号
    public static int timeSlice = 2;                 // 时间片
    public List<PCB> pcbList = new ArrayList<>();    //pcb队列
    public List<PCB> runList = new ArrayList<>();    // 默认等待运行队列
    public List<PCB> midRunList = new ArrayList<>();  //反馈调节中级等待队列
    public List<PCB> lowRunList = new ArrayList<>();  //反馈调节低级等待队列
    
    //初始化进程
    public void initProgress() {
        for (int i = 0; i < progressNum; i++) {
            PCB pcb = new PCB();
            pcb.initPCB(i);
            pcbList.add(pcb);
        }
    }
		
		//打印进程所有信息
    public void printPCBList()
		
		//打印默认等待队列进程信息
    public void printRunList() 
    
    //打印反馈调节三级队列
    public void printMultiQueueList()

    //执行程序
    public void process(PCB pcb, String type,String level)
    
    //Multilevel Feedback Queue Scheduling
    private void MFQS(PCB pcb,String level) 

    //Shortest Job First(抢占)
    private void SJF(PCB pcb) 

    //Shortest Remaining Time First
    private void SRTF(PCB pcb) 
    
    //Round Robin
    private void RR(PCB pcb) 

    //First-Come, First-Served
    public void FCFS(PCB pcb)
    
    //CPU运行
    public void run(String type)
    
    //反馈调节CPU运行
    public void runMFQS(String type)

		//等待队列是否为空
    private boolean queueEmpty()
		
		//模拟任务到达
    private void processArrival()
}
```

### CPU类中的run和process方法

- run方法主要用来模拟计算机时钟，当我们的进程没有执行结束CPU就一直工作
- `processArrival` 方法来模拟即将到来的进程，遍历初始化好的PCB列表，来选出到达时间和time一样的PCB，加入等待队列，同时设置PCB状态为W“等待”
- `process` 方法获取等待运行队列的第一个进程，然后运行1个time，这里我用switch控制调度算法（具体算法后面说）

```java

public void run(String type) {
    while (!queueEmpty()) {
        //模拟任务到达
        processArrival();
        if (!runList.isEmpty()) {
            //模拟CPU执行任务
            process(runList.get(0), type,null);
        }
        time++;
    }
}

//通过判断初始化的进程状态，如果有不是“T终止”的，就让CPU一直工作 
private boolean queueEmpty() {
    for (PCB pcb : pcbList) {
        if (pcb.getState() != 'T')
            return false;
    }
    return true;
}

private void processArrival() {
    for (int i = 0; i < progressNum; i++) {
        PCB pcb = pcbList.get(i);
        if (pcb.getArrival() == time) {
            pcb.setState('W');
            runList.add(pcb);
        }
    }
}

public void process(PCB pcb, String type,String level) {
	  if (pcb != null) {
	      switch (type) {
	          case "FCFS":
	              FCFS(pcb);
	              break;
	          case "RR":
	              RR(pcb);
	              break;
	          case "SJF":
	              SJF(pcb);
	              break;
	          case "SRTF":
	              SRTF(pcb);
	              break;
	          case "MFQS":
	              MFQS(pcb,level);
	              break;
	      }
	  }
}
```

### FCFS先来先服务

FCFS（First-Come, First-Served）是一个最简单的进程调度算法，也被称为先来先服务调度算法。在FCFS调度算法中，进程按照它们到达的顺序被依次调度执行，即先到达的进程先被执行，直到进程执行完成。

优点：

1. 简单直观：FCFS是一种非常简单和直观的调度算法，适用于简单的场景。
2. 公平性：由于按照进程到达的顺序进行调度，因此FCFS保证了公平性，即先到达的进程先被执行。
3. 非抢占：FCFS是一种非抢占式调度算法，一旦进程开始执行，直到完成或者阻塞，才会释放CPU资源。

缺点：

1. 长等待时间：FCFS可能导致长作业等待时间，特别是当一个长作业在前面排队时，后面的短作业需要等待很长时间才能执行。
2. 低响应性：由于FCFS是非抢占式的，一旦一个进程开始执行，其他进程需要等待它完成才能执行，可能导致低响应性。
3. 不适合实时系统：FCFS无法满足实时系统对响应时间的要求，因为无法保证进程在规定的时间内被执行。

FCFS调度算法适用于简单的场景。

### 算法实现

这个很简单，设置PCB状态为R“运行”，剩余时间减1，判断是否执行结束

```java
public void FCFS(PCB pcb) {
    pcb.setState('R');
    pcb.subRest();
    if (pcb.getRemain() == 0) {
        pcb.setState('T');
        runList.remove(0);
    }
}
```

![Untitled](/pic/CPU%E8%B0%83%E5%BA%A6%E7%AE%97%E6%B3%95%20JAVA%E5%AE%9E%E7%8E%B0%20%E9%99%84%E6%BA%90%E7%A0%81%20fef1fd04853048a88e1ea6e0430c2de1/Untitled.png)

### RR时间片轮转

RR（Round-Robin）是一种常见的进程调度算法，它是一种基于时间片的调度算法。在RR调度算法中，每个进程被分配一个时间片（时间量），当进程开始执行时，它将在一个时间片内执行，如果在时间片结束时进程还没有执行完成，则该进程会被暂停，放回就绪队列的末尾，等待下一次调度。被暂停的进程等待的时间片会在后续的调度中继续执行。

优点：

1. 公平性：RR调度算法保证了每个进程都能获得执行的机会，每个进程都会被分配一个时间片，从而实现公平性。
2. 高响应性：由于每个进程都有一个时间片，因此RR调度算法对于短作业和实时性要求较高的系统具有较好的响应性。
3. 预防饥饿：由于进程被分配时间片，因此RR调度算法可以避免某些进程长时间等待，从而预防饥饿现象的发生。

缺点：

1. 时间片长度选择：时间片的长度会影响系统的性能，如果时间片太短，会导致频繁的上下文切换，影响系统效率；如果时间片太长，可能会影响系统的响应速度。
2. 高开销：由于需要频繁进行上下文切换，RR调度算法可能会带来一定的开销，特别是在进程数量较多时。

RR调度算法适用于需要保证公平性和响应性的系统，特别是对于短作业和实时性要求较高的场景。

### 算法实现

刚开始还是一样，设置PCB状态为R“运行”，剩余时间减1，判断是否执行结束，这里如果有进程执行结束需要恢复时间片为默认值，这里默认为2，否则假如上一个进程执行了一个时间就结束了，那么下一个进程只能执行一个时间，就会轮替掉。

重新入队时会添加到队列尾部

```java
//Round Robin
private void RR(PCB pcb) {
    pcb.setState('R');
    pcb.subRest();
    timeSlice--;
    //执行完
    if (pcb.getRemain() == 0) {
        pcb.setState('T');
        runList.remove(0);
        timeSlice = 2;
    }

    //重新初始化时间片
    if (timeSlice == 0) {
        timeSlice = 2;
        //重新入队
        if (pcb.getRemain() != 0) {
            runList.remove(0);
            pcb.setState('W');
            runList.add(pcb);
        }
    }
}
```

![Untitled](/pic/CPU%E8%B0%83%E5%BA%A6%E7%AE%97%E6%B3%95%20JAVA%E5%AE%9E%E7%8E%B0%20%E9%99%84%E6%BA%90%E7%A0%81%20fef1fd04853048a88e1ea6e0430c2de1/Untitled%201.png)

### SJF最短任务优先、SRTF最短剩余时间优先

SJF（Shortest Job First）和SRTF（Shortest Remaining Time First）都是基于作业执行时间的调度算法，它们都优先选择执行时间最短的作业来执行。下面分别介绍它们的特点：

1. SJF（Shortest Job First）：
    - SJF调度算法是一种非抢占式的调度算法，即一旦进程开始执行，直到完成或者阻塞，才会释放CPU资源。
    - SJF根据进程的执行时间选择最短的作业来执行，以减少平均等待时间和周转时间。
    - SJF算法可以分为两种：非抢占式SJF和抢占式SJF。非抢占式SJF是在进程到达时确定作业的执行时间，而抢占式SJF在进程执行过程中可以根据后续作业的到达情况重新调度。
    - SJF调度算法可能导致长作业等待时间，特别是当一个长作业在前面排队时，后面的短作业需要等待很长时间才能执行。
2. SRTF（Shortest Remaining Time First）：
    - SRTF调度算法是SJF的一种抢占式形式，即在进程执行过程中可以根据后续作业的到达情况重新调度。
    - SRTF根据进程的剩余执行时间选择最短的作业来执行，因此可能会中断当前正在执行的作业，选择更短的作业执行。
    - SRTF可以最大程度地减少作业的等待时间和周转时间，提高系统的效率和响应速度。
    - SRTF调度算法可能会引起频繁的上下文切换，因为需要不断检查剩余执行时间最短的作业。
    

### 算法实现

这里我实现的SJF是一种抢占式算法，如果队列中加入的进程总执行时间最短，那么它会抢占当前的CPU，在实现这两个算法时我发现它们非常像，在代码中其实只是修改了一个判断条件

```java
//Shortest Job First(抢占)
private void SJF(PCB pcb) {
    //找出当前运行队列中最短执行时间的任务，放在运行队列头部
    PCB shortRest = pcb;
    for (PCB item : runList) {
        if (pcb.getRemain() != 0 && pcb.getBurst() > item.getBurst()) {
            shortRest = item;
            //先在队列中移除
            runList.remove(shortRest);
            //添加到队列头部
            runList.add(0, shortRest);
        }
    }
    shortRest.setState('R');
    shortRest.subRest();
    //进程执行结束
    if (shortRest.getRemain() == 0) {
        shortRest.setState('T');
        runList.remove(0);
    }
}

//Shortest Remaining Time First
private void SRTF(PCB pcb) {
    //找出当前运行队列中最短剩余时间的任务，放在运行队列头部
    PCB shortRest = pcb;
    for (PCB item : runList) {
        if (pcb.getRemain() != 0 && pcb.getRemain() > item.getRemain()) {
            shortRest = item;
            //先在队列中移除
            runList.remove(shortRest);
            //添加到队列头部
            runList.add(0, shortRest);
        }
    }
    shortRest.setState('R');
    shortRest.subRest();
    //进程执行结束
    if (shortRest.getRemain() == 0) {
        shortRest.setState('T');
        runList.remove(0);
    }
}
```

![Untitled](/pic/CPU%E8%B0%83%E5%BA%A6%E7%AE%97%E6%B3%95%20JAVA%E5%AE%9E%E7%8E%B0%20%E9%99%84%E6%BA%90%E7%A0%81%20fef1fd04853048a88e1ea6e0430c2de1/Untitled%202.png)

### MFQS多级反馈调节

MFQS（Multi-level Feedback Queue Scheduling）是一种多级反馈队列调度算法，常用于操作系统中的进程调度。MFQS将进程队列划分为多个级别，每个级别对应一个优先级，不同优先级的队列具有不同的时间片大小和调度策略。以下是MFQS调度算法的一些特点和工作原理：

1. 多级反馈队列：MFQS将进程队列划分为多个级别，通常是三个或更多个级别。**每个级别都有一个时间片大小，通常随着优先级的增加而减小。高优先级队列的时间片较短，低优先级队列的时间片较长。**
2. 调度策略：MFQS采用多级反馈的调度策略，即一个进程在不同队列之间切换执行。当一个进程在某个队列的时间片用完后，如果还未执行完，则会被移到下一个更低优先级的队列中继续执行。如果一个进程在高优先级队列等待时间较长，系统可能会提升其优先级，以提高其执行机会。
3. 调度过程：MFQS的调度过程通常是按照优先级依次轮转执行各个队列中的进程。高优先级队列中的进程优先执行，如果一个进程在当前队列执行完毕，则进入下一个优先级更低的队列继续执行。这种方式可以保证高优先级的进程得到及时响应，同时也允许低优先级的进程有机会执行。
4. 优点：MFQS能够平衡系统的响应速度和吞吐量，高优先级的进程能够快速响应，低优先级的进程则有机会执行，避免了饥饿现象。此外，MFQS还能够适应不同类型的工作负载，提高系统的整体性能。

### 算法实现

它的run方法特殊一点，因为有三个队列需要处理

```java
public void runMFQS(String type) {
    while (!queueEmpty()) {
        //模拟任务到达
        processArrival();
        if (!runList.isEmpty()) {
            //模拟CPU执行任务
            process(runList.get(0), type,"top");
        } else if (!midRunList.isEmpty()) {
            //模拟CPU执行任务
            process(midRunList.get(0), type,"mid");
        } else if (!lowRunList.isEmpty()) {
            //模拟CPU执行任务
            process(lowRunList.get(0), type,"low");
        }
        time++;
    }
}
```

我使用三个队列来实现，不同队列的时间片都是2，我这里没有对优先级进行特殊处理。你还可以对不同级别的队列设置不同的时间片，对等待时间超过一定阈值的进程把他提升到更高优先级的队列，以实现真正的“反馈调节”

```java
//Multilevel Feedback Queue Scheduling
private void MFQS(PCB pcb,String level) {
    pcb.setState('R');
    pcb.subRest();
    timeSlice--;
    //执行完
    if (pcb.getRemain() == 0) {
        pcb.setState('T');
        timeSlice = 2;
        switch (level){
            case "top":
                runList.remove(0);
                break;
            case "mid":
                midRunList.remove(0);
                break;
            case "low":
                lowRunList.remove(0);
                break;
        }
    }
    //重新初始化时间片
    if (timeSlice == 0) {
        timeSlice = 2;
        //重新入队
        if (pcb.getRemain() != 0) {
            switch (level){
                case "top":
                    runList.remove(0);
                    pcb.setState('W');
                    midRunList.add(pcb);
                    break;
                case "mid":
                    midRunList.remove(0);
                    pcb.setState('W');
                    lowRunList.add(pcb);
                    break;
                case "low":
                    lowRunList.remove(0);
                    pcb.setState('W');
                    lowRunList.add(pcb);
                    break;
            }
        }
    }
}
```

![Untitled](/pic/CPU%E8%B0%83%E5%BA%A6%E7%AE%97%E6%B3%95%20JAVA%E5%AE%9E%E7%8E%B0%20%E9%99%84%E6%BA%90%E7%A0%81%20fef1fd04853048a88e1ea6e0430c2de1/Untitled%203.png)

### 总结

通过学习CPU调度算法，可以更全面的认识操作系统，不同的CPU调度算法适用于不同的场景和需求，可以根据系统的特点和要求选择合适的调度算法，以提高系统的性能和效率。

其实这些算法大多也是来源于生活，像队列、抢占、优先级（特权），在写代码时应该多想想如果在现实中你会怎么做。

**`PCB.class`**

```java
public class PCB {

    private int pid;   // 进程号

    private char state;// 进程状态,'N'新建,'R'运行,'W'等待CPU(就绪),'T'终止

    private int priority; // 进程优先级,1~3,数字越小优先级越高

    private int arrival; // 到达时间

    private int burst;  // CPU区间时间

    private int remain;  //剩余时间

    public int getPid() {
        return pid;
    }

    public void setPid(int pid) {
        this.pid = pid;
    }

    public char getState() {
        return state;
    }

    public void setState(char state) {
        this.state = state;
    }

    public int getPriority() {
        return priority;
    }

    public void setPriority(int priority) {
        this.priority = priority;
    }

    public int getArrival() {
        return arrival;
    }

    public void setArrival(int arrival) {
        this.arrival = arrival;
    }

    public int getBurst() {
        return burst;
    }

    public void setBurst(int burst) {
        this.burst = burst;
    }

    public int getRemain() {
        return remain;
    }

    public void setRemain(int remain) {
        this.remain = remain;
    }
    public void subRest(){
        this.remain--;
    }

    @Override
    public String toString() {
        return "PCB"+this.pid;
    }

    public void initPCB(int i){
        this.pid = i+1;
        this.state = 'N';
        this.priority = (int)(Math.random()*3+1);
        this.arrival = (int)(Math.random()*10);
        this.burst = (int)(Math.random()*10+1);
        this.remain = this.burst;
//        System.out.println("进程"+this.pid+"创建成功");
    }
}

```

**`CPU.class`**

```java
public class CPU {
    public static int progressNum = 5; // 进程数
    public static int time = 0; // 时间
    public static int timeSlice = 2; // 时间片
    public List<PCB> pcbList = new ArrayList<>();
    public List<PCB> runList = new ArrayList<>();
    public List<PCB> midRunList = new ArrayList<>();
    public List<PCB> lowRunList = new ArrayList<>();

    public void initProgress() {
        for (int i = 0; i < progressNum; i++) {
            PCB pcb = new PCB();
            pcb.initPCB(i);
            pcbList.add(pcb);
        }
    }

    public void printPCBList() {
        System.out.println("进程号  进程状态  优先级  到达时间  CPU区间时间  剩余时间");
        for (int i = 0; i < progressNum; i++) {
            PCB pcb = pcbList.get(i);
            System.out.println(pcb.getPid() + "        " + pcb.getState() + "        " + pcb.getPriority() + "        " + pcb.getArrival() + "        " + pcb.getBurst() + "        " + pcb.getRemain());
        }
    }

    public void printRunList() {
//        System.out.println("-------------------------time:--" + time + "------------------------------------------------------------------------------------");
        for (int i = 0; i < runList.size(); i++) {
            PCB pcb = runList.get(i);
            System.out.print("进程号:" + pcb.getPid());
            System.out.print(" 剩余时间:" + pcb.getRemain());
            System.out.print("  |  ");
        }
        System.out.println();
    }
    public void printMultiQueueList() {
        System.out.print("一级队列 ===>>");
        for (int i = 0; i < runList.size(); i++) {
            PCB pcb = runList.get(i);
            System.out.print("进程号:" + pcb.getPid());
            System.out.print(" 剩余时间:" + pcb.getRemain());
            System.out.print("  |  ");
        }
        System.out.println();
        System.out.print("二级队列 ===>>");
        for (int i = 0; i < midRunList.size(); i++) {
            PCB pcb = midRunList.get(i);
            System.out.print("进程号:" + pcb.getPid());
            System.out.print(" 剩余时间:" + pcb.getRemain());
            System.out.print("  |  ");
        }
        System.out.println();
        System.out.print("三级队列 ===>>");
        for (int i = 0; i < lowRunList.size(); i++) {
            PCB pcb = lowRunList.get(i);
            System.out.print("进程号:" + pcb.getPid());
            System.out.print(" 剩余时间:" + pcb.getRemain());
            System.out.print("  |  ");
        }
        System.out.println();
    }

    //执行程序
    public void process(PCB pcb, String type,String level) {
        if (pcb != null) {
            switch (type) {
                case "FCFS":
                    FCFS(pcb);
                    break;
                case "RR":
                    RR(pcb);
                    break;
                case "SJF":
                    SJF(pcb);
                    break;
                case "SRTF":
                    SRTF(pcb);
                    break;
                case "MFQS":
                    MFQS(pcb,level);
                    break;
            }
//            System.out.println("time:"+time+" 当前执行进程"+pcb.getPid() +" 剩余时间："+pcb.getRest());
        } else {
//            System.out.println("time:"+time+" 当前CPU为空");
        }
    }

    //Multilevel Feedback Queue Scheduling
    private void MFQS(PCB pcb,String level) {
        pcb.setState('R');
        pcb.subRest();
        timeSlice--;
        //执行完
        if (pcb.getRemain() == 0) {
            pcb.setState('T');
            timeSlice = 2;
            switch (level){
                case "top":
                    runList.remove(0);
                    break;
                case "mid":
                    midRunList.remove(0);
                    break;
                case "low":
                    lowRunList.remove(0);
                    break;
            }

        }
        //重新初始化时间片
        if (timeSlice == 0) {
            timeSlice = 2;
            //重新入队
            if (pcb.getRemain() != 0) {
                switch (level){
                    case "top":
                        runList.remove(0);
                        pcb.setState('W');
                        midRunList.add(pcb);
                        break;
                    case "mid":
                        midRunList.remove(0);
                        pcb.setState('W');
                        lowRunList.add(pcb);
                        break;
                    case "low":
                        lowRunList.remove(0);
                        pcb.setState('W');
                        lowRunList.add(pcb);
                        break;
                }

            }
        }
    }

    //Shortest Job First(抢占)
    private void SJF(PCB pcb) {
        //找出当前运行队列中最短执行时间的任务，放在运行队列头部
        PCB shortRest = pcb;
        for (PCB item : runList) {
            if (pcb.getRemain() != 0 && pcb.getBurst() > item.getBurst()) {
                shortRest = item;
                //先在队列中移除
                runList.remove(shortRest);
                //添加到队列头部
                runList.add(0, shortRest);
                System.out.println("--------------------");
                System.out.println("进程："+shortRest.getPid()+" 发生抢占  |");
            }
        }
        shortRest.setState('R');
        shortRest.subRest();
        //进程执行结束
        if (shortRest.getRemain() == 0) {
            shortRest.setState('T');
            runList.remove(0);
        }

    }

    //Shortest Remaining Time First
    private void SRTF(PCB pcb) {
        //找出当前运行队列中最短剩余时间的任务，放在运行队列头部
        PCB shortRest = pcb;
        for (PCB item : runList) {
            if (pcb.getRemain() != 0 && pcb.getRemain() > item.getRemain()) {

                shortRest = item;
                //先在队列中移除
                runList.remove(shortRest);
                //添加到队列头部
                runList.add(0, shortRest);
                System.out.println("--------------------");
                System.out.println("进程："+shortRest.getPid()+" 发生抢占  |");
            }
        }
        shortRest.setState('R');
        shortRest.subRest();
        //进程执行结束
        if (shortRest.getRemain() == 0) {
            shortRest.setState('T');
            runList.remove(0);
        }

    }

    //Round Robin
    private void RR(PCB pcb) {
        pcb.setState('R');
        pcb.subRest();
        timeSlice--;
        //执行完
        if (pcb.getRemain() == 0) {
            pcb.setState('T');
            runList.remove(0);
            timeSlice = 2;
        }

        //重新初始化时间片
        if (timeSlice == 0) {
            timeSlice = 2;
            //重新入队
            if (pcb.getRemain() != 0) {
                runList.remove(0);
                pcb.setState('W');
                runList.add(pcb);
            }
        }
    }

    //First-Come, First-Served
    public void FCFS(PCB pcb) {
        pcb.setState('R');
        pcb.subRest();
        if (pcb.getRemain() == 0) {
            pcb.setState('T');
            runList.remove(0);
        }
    }

    public void run(String type) {
        while (!queueEmpty()) {
            System.out.println("-------------------------time:--" + time + "------------------------------------------------------------------------------------");
            //模拟任务到达
            processArrival();
            if (!runList.isEmpty()) {
                printRunList();
                //模拟CPU执行任务
                process(runList.get(0), type,null);
            }
            time++;
        }
    }
    public void runMFQS(String type) {
        while (!queueEmpty()) {
            System.out.println("-------------------------time:--" + time + "------------------------------------------------------------------------------------");
            //模拟任务到达
            processArrival();

            if (!runList.isEmpty()) {
                //模拟CPU执行任务
                process(runList.get(0), type,"top");
                printMultiQueueList();
            } else if (!midRunList.isEmpty()) {
                //模拟CPU执行任务
                process(midRunList.get(0), type,"mid");
                printMultiQueueList();
            } else if (!lowRunList.isEmpty()) {
                //模拟CPU执行任务
                process(lowRunList.get(0), type,"low");
                printMultiQueueList();
            }
            time++;
        }
    }

    private boolean queueEmpty() {
        for (PCB pcb : pcbList) {
            if (pcb.getState() != 'T')
                return false;
        }
        return true;
    }

    private void processArrival() {
        for (int i = 0; i < progressNum; i++) {
            PCB pcb = pcbList.get(i);
            if (pcb.getArrival() == time) {
                pcb.setState('W');
                runList.add(pcb);
            }
        }
    }

    public static void main(String[] args) {
        CPU cpu = new CPU();
        cpu.initProgress();
        cpu.printPCBList();
        cpu.run("FCFS");
//        cpu.run("RR");
//        cpu.run("SRTF");
//        cpu.run("SJF");
//        cpu.runMFQS("MFQS");
        cpu.printPCBList();

    }
}

```