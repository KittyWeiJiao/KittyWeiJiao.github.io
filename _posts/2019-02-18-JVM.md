https://blog.csdn.net/zz18435842675/article/details/82752997
https://blog.csdn.net/qq_31582127/article/details/85163430

https://blog.csdn.net/u014727260/article/details/55003402 -- 吴

http://www.importnew.com/tag/jvm

分布式存储、分布式缓存，RPC，微服务

### 一
    


### 三、类加载的流程？能否在加载类的时候，对字节码进行修改？

	2、使用Java探针技术，能够实现（ -javaagent:/app/apm/aliapm.jar）

    类加载的流程：
	1) 加载：通过类的全限定名来获取类的二进制字节流；然后将字节流所代表的静态存储结构转化为方法区的运行时数据结构；最后在内存中生成一个代表这个类的java.lang.Class对象，作为在方法区的这个类的各种数据的访问入口； 
	2) 验证：首先验证Class文件的字节流的信息是否符合当前虚拟机要求；然后再验证代码是否会危害虚拟机自身安全； 
	3) 准备：会为类变量分配内存，并赋初值
	4) 解析：将常量池类的符号引用替换为直接引用
	5) 初始化：执行<clinit>方法，由编译器自动收集类中的所有类变量的赋值动作和静态语句块中的语句合并产生的

    #### 补充：对象创建的过程
	当虚拟机遇到new指令时 
	1) 加载：去常量池当中定位类的符号引用； 
	2) 验证：检查这个类的符号引用是否被加载，解析和初始化过；如果没有，则进行类加载的过程，如果已经加载过，则虚拟机为对象分配内存； 
	3) 准备：虚拟机为分配到的内存空间都赋零值； 
	4) 解析：虚拟机对对象进行必要的设置，比如对象是哪个类的实例，对象的哈希码，对象的GC分代年龄等信息； 
	5) 初始：执行<init>方法，按照程序意愿初始化对象，即执行类的初始化函数 ，执行完成之后，就得到了一个新的对象。

### 四、JVM

    堆（heap）、方法区（perm gen）、栈（stack）

	1、堆 (新生代 : 老年代 = 1 : 2)
	##### 1.1 新生代 (Minor GC)	
	    Eden Space 和 Survivor Space(from survivor, tosurvivor) : 				
		1) Eden Space : 对象被创建时存放此区域，进行垃圾回收(Minor GC)后，不能被回收的对象被放入到空的survivor区域。
		2) Survivor Space 幸存者区，保存eden space内存区域中经过垃圾回收(Minor GC)后没有被回收的对象
	       执行垃圾回收时Eden区域不能被回收的对象被放入到空的to survivor，from survivor里不能被回收的对象也会被放入这个to survivor，然后To Survivor 和 From Survivor的标记会互换，始终保证一个(to survivor)是空的。
	    3) Eden : from : to = 8 : 1 : 1 ( 可以通过参数 –XX:SurvivorRatio 来设定 )
	    4) 每一次Minor GC(Young GC)后留下来的对象age加1			
	##### 1.2 老年代 (Full GC)
		Old Gen老年代，用于存放新生代中经过多次(默认是15岁，可以通过参数 -XX:MaxTenuringThreshold 来设定 )垃圾回收仍然存活的对象，或是新生代分配不了内存的大对象会直接进入老年代。			
	结论：Survivor的存在意义，就是减少被送到老年代的对象，进而减少Full GC的发生，Survivor的预筛选保证，只有经历16次Minor GC还能在新生代中存活的对象，才会被送到老年代
	
	2 方法区	
		存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据
	
	3 栈	
		程序计数器
		虚拟机栈
		本地方法栈

#### JVM内存模型：

	JVM将内存组织为主内存和工作内存两个部分。
	1、主内存：所有的线程所共享的,主要包括本地方法区和堆。
	2、工作内存：每个线程都有一个工作内存不是共享的，工作内存中主要包括两个部分：	
		a: 一个是属于该线程私有的栈;
		b: 对主存部分变量拷贝的寄存器(包括程序计数器PC和cup工作的高速缓存区)。
	
	1.所有的变量都存储在主内存中(虚拟机内存的一部分)，对于所有线程都是共享的。
	2.每条线程都有自己的工作内存，工作内存中保存的是主存中某些变量的拷贝，
	  线程对变量的所有操作都必须在工作内存中进行，而不能直接读写主内存中的变量。
	3.线程之间无法直接访问对方的工作内存中的变量，线程间变量的传递均需要通过主内存来完成,即：线程、主内存、工作内存。

### 五、你知道哪几种垃圾收集器，各自的优缺点，重点讲下cms和G1，包括原理，流程，优缺点。
	
	https://www.jianshu.com/p/898a3a559065
	1、CMS收集器
		CMS: Concurrent Mark Sweep 并发标记-清理收集器
		基于标记-清除算法
		CMS是一款以获取最短回收停顿时间为目标的收集器, 重视服务的响应速度.
		-XX:+UseConcMarkSweepGC, 这个参数表示对于老年代的回收采用CMS
		
		运作机制 (Stop The World : SWT 暂停所有当前运行的线程)
		1.初始标记(initial mark) : SWT  标记GC Roots能直接关联到的对象, 速度很快.
		2.并发标记(concurrent mark) : GC Roots Tracing. remark是为了修正concurrent mark阶段因用户程序继续运行而导致变动的那一部分对象的标记记录, 停顿时间比initial mark稍长
		3.重新标记(remark) SWT  标记出那些在并发标记过程中遗漏的，或者内部引用发生变化的对象
		4.并发清除(concurrent sweep)：如果发现一个Region中没有存活对象，则把该Region加入到空闲列表中	  
		收集器线程可以与用户线程一起工作, 忽略掉initial mark 和remark , 从总体时间上来看可以当作CMS收集器回收过程是与用户线程一起并发执行的.

		优点
		1.并发收集(concurrent)
		2.低停顿
		缺点
		1.对CPU资源非常敏感, 并发阶段会降低系统吞吐量
		2.无法处理浮动垃圾(Floating Garbage)
		3.标记-清除算法会产生大量空间碎片. 大量的碎片会导致老年代空间剩余很大却无法被充分利用. 难以找到连续的空间分配大对象, 导致触发Full GC.

	2、G1收集器 （http://www.importnew.com/27793.html）
		G1: Garbage First
		## 相比CMS收集器, G1收集器有两个改进点
		1.G1基于标记-整理算法, 不会产生空间碎片
		2.G1可以精确控制停顿, 既能让使用者明确指定在一个长度为M毫秒的时间片段内, 消耗在垃圾收集器上的时间不得超过N毫秒, 几乎是实时Java垃圾收集器的特征了.
		## G1中提供了三种模式垃圾回收模式
		1、young gc ： 年轻代的GC算法，当所有eden region被耗尽无法申请内存时，就会触发一次young gc
		2、mixed gc ： 当越来越多的对象晋升到老年代old region时，为了避免堆内存被耗尽，虚拟机会触发一个混合的垃圾收集器，即mixed gc，回收整个young region 和 部分 old regio
        3、full gc ：老年代被填满，就会触发一次full gc

	3、Serial收集器
		单线程
		最原生的收集器, jdk1.3以前唯一的选择.
		单线程收集器. 这里并不是指使用一个CPU或一条收集线程去完成垃圾收集工作, 而是指它在收集垃圾时, 必须暂停用户工作线程(Stop The World).
		优点：简单而高效. Client模式下新生代收集器常用. 收集100m左右新生代几十毫秒-100毫秒
		缺点：用户停顿感体验差

### 六、垃圾回收算法 

（https://blog.csdn.net/newchenxf/article/details/78071804）

	标记-清除算法        CMS、老年代
	标记-整理算法        G1、老年代
	复制算法            新生代、ParNew收集器
	分代收集算法
	
	垃圾回收算法，都首先要依赖一个基础算法，姑且称为标记算法，用于找出哪些对象可以回收。比较出名的是引用计数算法和可达性分析算法。


### 七、内存分配策略

	对象优先分配在eden区域；
	大对象直接进入老年代；
	通过引用计数算法，长期存活对象进入老年代（如对象在新生代中经历15次回收依然存在）；

### 八、请解释如下jvm参数的含义

https://www.cnblogs.com/redcreen/archive/2011/05/04/2037057.html

	-server 
	-Xms512m   初始堆大小
	-Xmx512m   最大堆大小
	-XX:PermSize=256m      持久代初始内存的大小
	-XX:MaxPermSize=512m   持久代的最大内存
	-Xss1024K  线程的栈大小
	-Xmn128M   新生代的大小（等同于-XX:NewSize=NNN和-XX:MaxNewSize=NNN）
	-XX:NewRatio=N 老年代和新生代的比例
	-XX:SurvivorRatio=N 新生代中eden和survivior(包括From和To)的比例
					（N为8表示Eden区与Survivor区的大小比值是8:1:1)。
	-XX:TargetSurvivorRatio=N   新生代survivior空间可使用率，达到此值则把To空间的对象移动到老年代。
	-XX:MaxTenuringThreshold=20  晋升年龄最大阈值，默认15。在新生代中对象存活次数(经过YGC的次数)后仍然存活，就会晋升到老年代。每经过一次YGC，年龄加1，当survivor区的对象年龄达到TenuringThreshold时，表示该对象是长存活对象，就会直接晋升到老年代。	

    -XX:+UseConcMarkSweepGC 用CMS垃圾收集器

	-XX:CMSInitiatingOccupancyFraction=80   设定CMS在对内存占用率达到80%的时候开始GC
	-XX:+UseCMSInitiatingOccupancyOnly  设定的回收阈值(上面指定的80%),如果不指定,JVM仅在第一次使用设定值,后续则自动调整.
	这两个设置一般配合使用,一般用于『降低CMS GC频率或者增加频率、减少GC时长』的需求
