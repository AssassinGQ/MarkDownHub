# 基本知识

----------

## 1.数据库
### 1.1.数据库ACID原则-数据库事务正确执行的四个基本要素
#### 1.1.1.原子性（Atomicity）：
###### 整个事务中的所有操作，要么全部完成，要么全部不完成不可能停滞在中间某个环节。事务在执行过程中发生错误，会被回滚到事务开始前的状态，就像这个事务从来没有执行过一样
#### 1.1.2.一致性（Consistency）：
###### 一个事务可以封装状态改变（除非它是一个只读的）。事务必须始终保持系统处于一致的状态，不管在任何给定的时间并发事务有多少。<br>也就是说：如果事务是并发多个，系统也必须如同串行事务一样操作。其主要特征是保护性和不变形，以转账案例为例，假设有五个账户，每个账户余额是100元，那么五个账户总额是500元，如果在这个5个账户之间同时发生多个转账，无论并发多少个，这5个账户总额也应该还是500元，这就是保护性和不变形。
#### 1.1.3.隔离性（Isolation）：
####### 隔离状态执行事务，使它们好像是在系统给定的时间内执行的唯一操作。如果有两个事务，运行在相同的时间内，执行相同的功能，事务的隔离性将确保每一事务在系统中认为只有个该事物在使用系统。这种属性有时称为串行化，为了防止事务操作间的混淆，必须串行化或序列化请求，是的在同一时间仅有一个请求用于同一数据。
#### 1.1.4.持久性（Durability）：
###### 在事务完成以后，该事务对数据库所做的更改便持久的保存在数据库中了，并不会被回滚。
#### 1.1.5.总结：
###### 由于一项操作通常会包含许多子操作，而这些自操作可能会因为硬件的损坏或其他因素产生问题，要正确实现ACID并不容易。ACID建议数据库将所有西药更新以及修改的资料一次操作完毕，但实际上并不可行。<br>目前主要有两种方式实现ACID：第一种是Write ahead logging，也就是日志式的方式（现代数据库均基于这种方式）。第二种是Shadow paging。<br>相对于WAL技术，shadow paging技术实现起来比较简单，消除了写日志记录的开销恢复的速度也快（不需要redo和undo）。shadow paging的缺点就是事务提交时要输出多个块，这使得提交的开销很大，而且以块为单位，很难应用到允许多个食物并发的执行的情况--这是它的致命的缺点。<br>WAL的中心思想是对数据文件的修改（它们是表和索引的载体）必须是只能发生在这些修改已经记录了日志之后--也就是说，在日志记录冲刷到永久存储器之后。如果我们准讯这个过程，那么我们就不需要在每次事务提交的时候都把数据也冲刷到磁盘，因为我们知道在出现崩溃的情况下，我们可以用日志来恢复数据库：任何尚未附加到数据也的巨鹿都将先从日志记录中重做（这叫向前滚动恢复，也叫REDO)然后那些未提交的事务做的修改江北从数据也中删除（这叫向后滚动恢复，也叫UNDO)。
### 1.2.数据库隔离级别
###### 不考虑隔离级别，会出现的几种问题：
#### 1.2.1.脏读
###### 脏读是指在一个事务处理过程中读取了另一个未提交的事务中的数据。
#### 1.2.2.不可重复读
###### 不可重复读是指在对于数据库中的某个数据，一个事务范围内多次查询缺返回了多个不同的数据值，这是由于在查询间隔，被另一个事务修改并提交了。
#### 1.2.3.幻读（虚读）
###### 幻读是事务费独立执行是发生的一种现象，例如事务T1对一个表中所有的行的某个数据项做了从“1”修改为“2”的操作，这是事务T2又对这个表中插入了一行数据项，而这个数据项的数值还是为“1”并且提交给数据库。而操作事务T1的用户如果再查看刚刚修改的数据，会发生还有一行没有修改，其实这行是从事务T2中添加的，就好像发生了幻觉一样，这就是幻读。<br>幻读好人不可重复读都是读取了另一条已经提交的事务（这点就脏读不同），所不同的是不可重复读查询的都是同一个数据项，而幻读针对的是一批数据整体（比如数据的个数）。
#### 1.2.4.MySQL四种隔离级别：
###### Serializable（串行化）：可以避免脏读、不可重复读、幻读的发生。
###### Repeatable read（可重复读）：可以避免脏读、不可重复读的发生。
###### Read committed（读已提交）：可避免脏读的发生。
###### Read uncommitted（读未提交）：最低级别，任何情况都无法保证。
###### 对于MySQL而言，事务级别的设置只对Connection有效，不同的Connection可以有不同的隔离级别
### 1.3.MySQL数据库引擎
#### 1.3.1.ISAM
###### ISAM是一个定义明确且经历时间考验的数据表格管理方法，它在设计之时就考虑到数据库被查询次数要远大于更新的次数。因此，ISAM执行读取操作的速度很快，而且不占用大量的内存和存储资源。ISAM的两个主要不足之处在于，它不支持事务，也不能够容错：如果你的硬盘奔溃了，那么数据文件就无法恢复了。如果你正在把SIAM用在关键任务应用程序里，那就必须经常备份你所有的实时数据，通过复制特性，MYSQL能够支持这样的备份应用程序。
#### 1.3.2.MyISAM
###### MyISAM是MySQL的ISAM扩展格式和缺省的数据库引擎。除了提供ISAM里所没有的索引和字段管理的大量功能，MyISAM还使用一种表格锁定的机制，来优化多个并发的读写操作，其代价是你需要经常运行OPTIMIZE TABLE命令，来恢复被更新机制所浪费的空间。MyISAM还有一些有用的扩展，例如用来修复数据库文件的MyISAMCHK工具和用来恢复浪费空间的MyISAMPACK工具。MyISAM强调快读取操作，这可能就是为什么MySQL受到WEB开发如此青睐的原因：在WEB开发中你所进行的大量数据都是读取操作。所以，大多数虚拟主机提供商和INTERNET平台提供商只允许使用MyISAM格式。MyISAM的一个重要缺陷就是不能在损坏后恢复是数据，也不支持事务操作。
#### 1.3.3.HEAP
###### HEAP允许只驻留在内存里的临时表格。驻留在内存里让HEAP要比ISAM和MyISAM还要快，但是它所管理的数据时不稳定的，而且如果在关机之前没有进行保存，那么所有的数据都会丢失。在数据行被删除的时候，HEAP也不会浪费大量的空间。HEAP表格在你需要使用SELECT表达式来选择和操控数据的时候都非常有用。要记住，在用完表格之后就删除表格。
#### 1.3.4.InnoDB 关闭autocommit
###### InnoDB数据库引擎也是造就MySQL灵活性的技术的直接产品，这项技术就是MySQL+API。在使用MySQL的时候，你所面对的每一个挑战，几乎都源于ISAM和MyISAM数据库引擎不支持事务处理，也不支持外键。尽管要比ISAM和MyISAM引擎慢很多，但是InnoDB包括了对事务处理和外键的支持，这两点都是前两个引擎所没有的。如前所述，如果你的设计需要这些特性中的一个或者两个，那你就要被迫使用后HEAP或者InnoDB引擎。
#### 1.3.5.MySQL+ API
###### 可以使用MySQL+API来创建自己的数据库引擎。这个API提供了操作字段，记录，表格，数据库，连接，安全账号的功能，以及建立诸如MySQL这样DBMS所需要的所有其他无数功能。
#### 1.3.6.MyISAM和InnoDB的比较
###### MyISAM的索引和数据时分开的，并且索引是有压缩的，内存的使用率就对应提高不少。能加载更多索引，而InnoDB是索引和数据紧密捆绑的，没有使用压缩从而造成InnoDB比MyISAM体积庞大不小。
###### 从平台角度来说，经常隔1，2个月就会发生应用开发人员不小心update一个表where写的范围不对，导致这个表没法正常用了，这个时候MyISAM的优越性就体现出来了，随便从当天拷贝的压缩包取出对应表的文件，随便放到一个数据库目录下，然后dump成sql再导回到主库，并把对应的binlog补上。如果是Innodb，恐怕不可能有这么快速度，别和我说让Innodb定期用导出xxx.sql机制备份，因为我平台上最小的一个数据库实例的数据量基本都是几十G大小。
###### 从我接触的应用逻辑来说，select count(*) 和order by是最频繁的，大概能占了整个sql总语句的60%以上的操作，而这种操作Innodb其实也是会锁表的，很多人以为Innodb是行级锁，那个只是where对它主键是有效，非主键的都会锁全表的。
###### 还有就是经常有很多应用部门需要我给他们定期某些表的数据，MyISAM的话很方便，只要发给他们对应那表的frm.MYD,MYI的文件，让他们自己在对应版本的数据库启动就行，而Innodb就需要导出xxx.sql了，因为光给别人文件，受字典数据文件的影响，对方是无法使用的。
###### 如果和MyISAM比insert写操作的话，Innodb还达不到MyISAM的写性能，如果是针对基于索引的update操作，虽然MyISAM可能会逊色Innodb,但是那么高并发的写，从库能否追的上也是一个问题，还不如通过多实例分库分表架构来解决。
###### 如果是用MyISAM的话，merge引擎可以大大加快应用部门的开发速度，他们只要对这个merge表做一些selectcount(*)操作，非常适合大项目总量约几亿的rows某一类型(如日志，调查统计)的业务表。
###### 当然Innodb也不是绝对不用，用事务的项目如模拟炒股项目，我就是用Innodb的，活跃用户20多万时候，也是很轻松应付了，因此我个人也是很喜欢Innodb的，只是如果从数据库平台应用出发，我还是会首MyISAM。
###### MyISAM适用于：<br>(1).做很多count的计算；<br>(2).插入不频繁，查询非常频繁；<br>(3).没有事务
###### InnoDB适用于：<br>(1).要求可靠性高，要求事务；<br>(2).表更新和查询都相当频繁，并且表锁定的机会比较大的情况；<br>(3).没有事务
## 2.进程与线程
### 2.1.多线程
#### 2.1.1.进程与线程的区别
###### 进程每个进程都有独立的代码和数据空间（进程上下文），进程间的切换会有较大的开销，一个进程包含1--n个线程（进程是资源分配的最小单位）；
###### 同一进程的线程共享代码和数据空间，每个线程有独立的运行栈和程序计数器，线程切换开销小（线程是cpu调度的最小单位）
###### 线程和进程一样分为五个阶段：创建，就绪，运行，阻塞，终止。
###### 多进程是指操作系统能够同时运行多个任务（程序）；多线程是指同一程序中有多个顺序流在执行
#### 2.1.2.如何启动多线程
##### 继承java.lang.Thread类
##### 实现java.lang.Runnable接口
##### 直接传入一个Runnable接口实例化一个Thread
##### 实现Callable接口，并与Future、线程池结合使用
#### 2.1.3.线程底层初始化过程
#### 2.1.4.Thread和Runnable的区别
###### 如果一个类继承Thread，则不适合资源共享。但是如果实现了Runnable接口的话，则很容易地实现资源共享。
###### Runnable接口适合多个相同的程序代码的线程去处理同一个资源；可以避免java中的单继承的限制；增加程序的健壮性，代码可以被多个线程共享，代码和数据独立；线程池只能放入实现Runable或callable类线程，不能直接放入继承Thread的类
###### main方法其实也是一个线程。线程的执行先后随机，完全看谁先得到CPU的资源
###### java中每次程序至少启动两个线程。一个是main线程，一个是垃圾收集线程。而每一个java命令执行一个类的时候，实际上都会启动一个JMV，每个JVM实际就在操作系统中启动一个进程
#### 2.1.5.线程状态的转换
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/ThreadState.bmp)
#### 2.1.6.线程同步
##### synchronized单独使用
1. 代码块：在多线程环境下，synchronized块中的方法获取了lock实例的monitor，试过实例相同，那么只有一个线程能执行该块内容<br>
```
public class Thread1 implements Runnable {
   Object lock;
   public void run() {  
       synchronized(lock){
         ..do something
       }
   }
}
```
2. 直接用于方法：相当于上面代码中用lock来锁定的效果，实际获取的是Thread1类的monitor。更进一步，如果修饰的是static方法，则锁定该类所有实例<br>
```
public class Thread1 implements Runnable {
   public synchronized void run() {  
        ..do something
   }
}
```
##### synchronized，wait，notyfy
```
public synchronized void produce()
{
	if(this.product >= MAX_PRODUCT)
    {
    	try
        {
        	wait();  
            System.out.println("产品已满,请稍候再生产");
        }
        catch(InterruptedException e)
        {
        	e.printStackTrace();
        }
        return;
  	}
    this.product++;
    System.out.println("生产者生产第" + this.product + "个产品.");
    notifyAll();   //通知等待区的消费者可以取出产品了
}

public synchronized void consume()
{
	if(this.product <= MIN_PRODUCT)
    {
    	try 
        {
        	wait(); 
            System.out.println("缺货,稍候再取");
        } 
        catch (InterruptedException e) 
        {
            e.printStackTrace();
        }
        return;
    }
	System.out.println("消费者取走了第" + this.product + "个产品.");
    this.product--;
    notifyAll();   //通知等待去的生产者可以生产产品了
}
```
##### volatile
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/MTMM.bmp)
###### 多线程的内存模型：main memory（主存）、working memory（线程栈），在处理数据时，线程会把值从主存load到本地站，完成操作后再save回去（volatile关键词的作用：每次针对该变量的操作都激发一次load and save）。
###### 针对多线程使用的变量如果不是volatile或者final修饰的，很有可能产生不可预知的结果（另一个线程修改了这个值，但是之后在某线程看到的是修改 之前的值）。其实道理上讲同一实例的同一属性本身只有一个副本。但是多线程是会缓存值得，本质上，volatile就是不去缓存，直接取值。在线程安全的情况下加volatile会牺牲性能。
#### 2.1.7.线程数据传递
###### 使用同一个Runnable对象，这个Runnable对象中的成员变量可以用作共享数据
###### 将共享数据封装成另外一个对象，然后将这个对象逐一传递给各个Runnable对象，每个线程对共享数据的操作方法也分配到那个对象身上完成，这样容易实现针对数据进行各个操作的互斥和通信。
###### 将Runnable对象作为一个类的内部类，共享数据作为这个类的成员变量，每个线程对共享数据的操作方法也封装在外部类，以便实现对数据的各个操作的同步和互斥，作为内部类的各个Runnable对象调用外部类的这些方法。
### 2.2.多进程
#### 2.2.1.创建进程
##### 使用Runtime的exec()方法
###### 每个Java应用程序都有一个Runtime类实例，使用应用程序能够与其运行的环境相连接。可以通过getRuntime方法获取当前运行时。<br>应用程序不能创建自己的Runtime类实例。但可以通过getRuntime方法获取当前Runtime运行时对象的引用。一旦得到了一个当前的Runtime类对象的引用，就可以调用Runtime对象的方法去控制Java虚拟机的状态和行为。有以下方法：
1. void addShutdownHook(Thread hook)
2. int availableProcessors()
3. Process exec(String command)
4. Process exec(String[] cmdarray)
5. Process exec(String[] cmdarray, String[] envp)
6. Process exec(String[] cmdarray, String[] envp, File dir)
7. Process exec(String command, String[] envp)
8. Process exec(String command, String[] envp, File dir)
9. void exit(int status)
10. long freeMemory()
11. void gc()
12. InputStream getLocalizedInputStream(InputStream in)
13. OutputStream getLocalizedOutputStream(OutputStream out)
14. static Runtime getRuntime()
15. void halt(int status)
16. void load(String filename)
17. void loadLibrary(String libname)
18. long maxMemory()
19. boolean removeShutdownHook(Thread hook)
20. void runFinalization()
21. static void runFinalization()
22. static void runFinalizersOnExit(boolean value)
23. long totlalMemory()
##### 使用ProcessBuilder的start()方法
###### 每个ProcessBuilder实例管理一个进程属性集。start()方法利用这些属性创建一个新的Process实例。start()方法可以从同一实例重复调用，以利用相同的活或相关的属性创建新的子进程。每个进程管理器管理这些进程属性：
1. 命令：是一个字符串列表，它表示调用的外部程序文件及其参数（如果有）。在此，表示有效操作系统命令的字符串列表是依赖于系统的。；例如，每一个总体变量，通常都要成为此列表中的元素，但有一些操作系统，希望程序能自己标记命令行字符串--在这种系统中，Java实现可能需要命令确切的包括这两个元素。
2. 环境：是从变量到值的依赖于系统的映射。初始值是当前进程环境的一个副本(请参阅System.getenv())。
3. 工作目录：默认值是当前进程的当前工作目录，通常根据系统属性user.dir来命名。
4. redirectErrorStream属性。最初，此属性为false，意思是子进程的标准输出和错误输出被发送给两个独立的流，这些流可以通过Process.getInputStream()和Process.getErrorStream()方法来访问。如果将值设置为true，标准错误将与标准输出合并。这使得关联错误消息和相应的输出变得更容易。在此下，合并得数据可从Process.getInputStream()返回的流读取，而从Process.getErrorStream()返回的流读取将直接达到文件尾。
###### 修改进程构建起的属性将影响后续有该对象的start()方法启动的进程，但从不会影响以前启动的进程或Java自身的进程。大多数错误检查有start()方法执行。可以修改对象的状态，但这样start()将会注意，此类不是同步的。如果多个线程同时访问一个ProcessBuilder，而其中至少一个线程从结构上修改了其中一个属性，它必须保持外部同步。
#### 2.2.2.多进程编程实例
#### 2.2.3.进程间通信
## 3.TCP/IP，协议，网络传输，网络安全
### 3.1.基本概念：
###### 应用层，包含http，ftp等协议
###### 传输层（运输层），包含tcp，udp等协议
###### 网路层（IP层），IP协议，负责对数据加上IP地址和其他数据以确定传输的目标
###### 数据链路层（MAC层），加上以太网协议头，进行CRC编码，为IP模块发送和接受IP数据包，为ARP模块发送ARP请求和接受ARP应答，为RARP发送RARP请求和接受RARP应答。链路层上至少支持loopback协议和以太网协议（RFC894和RFC1042）
###### 硬件层次，包含网卡的定义，网线的制式等
###### 端口号是用在TCP、UDP上的一个逻辑号码，并不是一个硬件端口，我们平时说把某某端口封掉，也只是在IP层次把带有这个号码的IP包给过滤掉
#### 3.1.1.IP协议
###### IP协议是TCP/IP的核心，所有的TCP,UDP,IMCP,IGCP的数据都以IP数据格式传输。要注意的是，IP不是可靠的协议，这么说，IP协议没有提供一种数据未传达以后的处理机制。这被认为是上层协议TCP或UDP要做的事情。
###### IP协议头中含八位TTL字段，表示最多能够被路由几次，八位最多路由255次。协议头中还有版本号，现在的IP版本号是4，所以称为IPv4，还有IPv6
###### IP路由选择方式：如果TTL（生命周期）还没有到，首先搜索路由表，如果存在目标IP完全一致的路由器，就将该包发给该路由器；否则搜索路由表，发给同子网的路由器，这需要子网掩码的协助；否则搜索路由表，发给同一网络号的路由器；否则就丢弃这个包。（所以IP协议是不可靠的，它不保证送达）
###### IP地址可以分为 网络号+子网号+主机号
#### 3.1.2.ARP协议
###### 在数据链路层的数据包中会有一个MAC地址头。而每一块以太网卡都有一个唯一的MAC地址。ARP协议的工作就是让数据包能够知道以太网卡的MAC地址，然后将该MAC地址打包到数据链路层包地址头中
###### ARP协议（地址解析协议）是一种解析协议，通过IP地址获得MAC地址。当主机要发送IP包的时候，首先查询自己本地的ARP告诉缓存，如果查询不到IP-MAC键值对，主机就会想网络发送ARP协议广播包。而接受到这个广播的所有主机都会查询自己的IP地址，符合条件，就准备一个包含本主机MAC地址的ARP包发送给ARP广播主机，广播主机拿到ARP包以后就会更新自己的ARP缓存。然后重新准备好数据链路层的数据包发送工作
#### 3.1.3.RARP协议
#### 3.1.4.ICMP协议
###### ICMP协议全称Internet Control Message Protocol，因特网控制报文协议。由于IP协议是不可靠的，为了保证数据的送达工作，ICMP协议在其中扮演了重要的作用，当传送IP数据包发生错误（目标主机不可达，路由不可达，端口不可达等）时，源主机会收到ICMP协议封装的错误信息。
###### ICMP协议数据包由 20byteIP头，8bit错误类型（目前已经定义了14种，可以分为两大类，取值1-127的类型和取值128以上的类型）和8bit代码（类型和代码共同决定ICMP报文的详细类型）和16bit校验和，16bit标识符（标识ICMP的进程，但仅适用于回显请求和应答ICMP报文，对于目标不可达ICMP报文和超时ICMP报文等，该字段的值为0）和16bit的序列号，还可以有若干byte的选项
###### 一般发生错误的包传送时，应该给出ICMP报文，但特殊情况下，为了防止产生ICMP报文的无限传播，不产生错误报文。
###### ICMP有两种类型，一种是查询报文，一种是差错报文。查询报文主要有一下几种用途：ping查询，子网掩码查询（用于无盘工作站在初始化自身的时候初始化子网掩码），时间戳查询（可以用来同步时间）：
|类型|代码|描述|
|-----|-----|-----|
|0|0|ping响应|
|8|0|ping请求|
|9|0|路由器响应|
|10|0|路由器请求|
|13|0|时间戳请求|
|14|0|时间戳响应|
|15|0|信息请求（废弃）|
|16|0|信息响应（废弃）|
|17|0|地址掩码请求|
|18|0|地址掩码响应|
###### ICMP的应用--ping。发送类型为0的ICMP包发请求，收到请求的主机则用类型为8的ICMP包回应。ping还可以看到主机到目标主机路由的机会。ICMP的ping请求没经过一个路由器，路由器会把自己的IP放到该数据包中，而目标主机会把这个IP列表复制到回应用的ICMP数据包中。但这个机制首先于IP头所能记录的路由列表。
###### ICMP的应用--Traceroute，用来侦测主机到目标主机之间所经过路由情况。源主机先发一个TTL=1的UDP包，第一个路由器收到包，TTL变成0，第一个路由器会返回一个主机不可达ICMP数据包给源主机；源主机再发送一个TTL=2的UDP包，一次类推。由于发送的UDP包的端口号大于30000，达到目标主机时，目标主机会返回一个端口不可达的ICMP报文给源主机，源主机根据ICMP报文类型判断UDP包是否到了目标主机。
###### ICMP的应用--改良的Tracerout：由于安全问题，大多数服务器不提供UDP服务或者被防火墙挡掉，所以traceroute很多情况下不可用。有一种基于ICMP实现的traceroute，首先发送一个TTL=1的ping请求，如果收到TTL超时的ICMP，则继续发送TTL=2的ICMP报文，一次类推，直到收到pong响应
###### ICMP的IP重定向报文：当IP包在某个地方转向的时候，都会给源主机发送一个ICMP重定向报文，源主机会据此更新自己的路由表。其中重定向报文只能由路由器发出，只能为主机所用。随着网络通信次数的增多，路由表会越来越完备
#### 3.1.5.静态IP选路
###### 选路是IP层最重要的功能之一。通过路由表来选路。具体见3.1.1
###### ICMP的路由发现报文：在主机引导的时候，一般会在网内发送一个路由请求的ICMP报文，而多个路由器会响应一个路由通告报文。而且路由其本身不定期地在网络内发布路由通告报文。根据这些报文，主机有机会建立自己的路由表。路由器的一份通告中可以通告多个地址，并且给出每个地址的优先等级，表示该IP作为默认路由的等级。路由器一般会在450-600秒时间间隔内发布一次通告，而一个通告报文的寿命是30分钟。而主机引导是时候会每隔三秒发送一次请求报文，一旦接收到有限的通告报文，就停止发送请求报文
#### 3.1.6.动态选路协议
###### 静态选路简要地说就是在配置接口的时候，以默认的方式生成路由表项，并通过route来增加表项，或者通过ICMP报文来更新表项。如果上述三种方法都不能满足，那么我们就使用动态路由
###### RIP协议：选路信息协议
#### 3.1.5.UDP协议
##### 3.1.5.1.简要介绍
###### UDP是传输层协议，和TCP协议处于一个分层中，但与TCP协议不同，UDP协议不提供超时重传，出错重传等功能，是个不可靠的协议
##### 3.1.5.2.UDP协议头
###### UDP端口号：用于区分不同的程序所需要的数据包。例如某个UDP程序A在系统中注册了3000端口，那么以后从外面传进来的目的端口号位3000的UDP包都会交给该程序。端口号理论上有2^16这么多，因为它的长度是16bit
###### UDP检验和：这是要一个可选的选项，并不是所有的系统都对UDP数据包加以检验和（TCP协议必须），但RFC标准中，发送端应该计算校验和。<br>校验和覆盖UDP协议头和数据（IP协议的校验和值覆盖IP数据头）。UDP和TCP都包含一个伪首部，这是为了计算校验和而设置的。伪首部甚至包括IP地址这样的IP协议里面都有的信息，目的是让UDP两次检查数据是否已经正确到达目的地。如果发送端没有打开校验和选项，而接收端计算校验和有差错，那么UDP数据将会被悄悄丢掉，而不产生差错报文
###### UDP长度：UDP可以很长很长，最长为65535字节。但网络传输的时候，一次一般传送不了那么长的协议（涉及到MTU问题），就只好对数据分片，当然，这些对UDP等上级协议是透明的，UDP不需要关心IP协议层对数据如何分片
##### 3.1.5.3.IP分片
###### IP在从上层接到数据以后，要根据IP地址来判断从哪个接口发送数据，并进行MTU查询，如果数据大小超过MTU就进行数据分片。数据分片对上层和下层透明，而数据到达目的地还会被重新组装。在IP头里面，16bit识别号唯一记录一个IP包的ID，具有同一个ID的IP包会被重新组装；而13位偏移则记录了某IP片相对整个包的位置；而这两个表示中间的3bit标志则标示着该分片后面是否还有新的分片。
##### 3.1.5.4.UDP和ARP之间的交互式用
###### 这是不常被人注意到的一个细节，这是针对一些系统地实现来说的。当ARP缓存还是空的时候。UDP在被发送之前一定要发送一个ARP请求来获得目的主机的MAC地址，如果这个UDP的数据包足够大，大到IP层一定要对其进行分片的时候，想象中，该UDP数据包的第一个分片会发出一个ARP查询请求，所有的分片都辉等到这个查询完成以后再发送。事实上是这样吗？<br>结果是，某些系统会让每一个分片都发送一个ARP查询，所有的分片都在等待，但是接受到第一个回应的时候，主机却只发送了最后一个数据片而抛弃了其他，这实在是让人匪夷所思。这样，因为分片的数据不能被及时组装，接受主机将会在一段时间内将永远无法组装的IP数据包抛弃，并且发送组装超时的ICMP报文（其实很多系统不产生这个差错），以保证接受主机自己的接收端缓存不被那些永远得不到组装的分片充满
##### 3.1.5.5.ICMP源站抑制差错
###### 当目标主机的处理速度赶不上数据接收的速度，因为接受主机的IP层缓存会被占满，所以主机就会发出一个“我受不了”的一个ICMP报文
##### 3.1.5.6.UDP服务器设计
###### UDP协议的某些特性将会影响我们的服务器程序设计，大致总结如下：<br>1.关于客户IP和地址：服务器必须有根据客户IP地址和端口号判断数据包是否合法的能力（这似乎要求每一个服务器都要具备）<br>2.关于目的地址：服务器必须要有过滤广播地址的能力<br>3.关于数据输入：通常服务器系统的每一个端口号都会和一块输入缓冲区对应，进来的输入根据先来后到的原则等待服务器的处理，所以难免会出现缓冲区溢出的问题，这种情况下，UDP数据包可能会被丢弃，而应用服务器程序本身并不知道这个问题<br>4.服务器应该限制本地IP地址，就是说它应该可以把自己绑定到某一个网络接口的某一个端口上
#### 3.1.6.广播与多播，IGMP协议
##### 3.1.6.1.单播
###### 单播是说，对特定的主机进行数据传送。例如给某一个主机发送IP数据包。这时候，数据链路层给出的数据头里面是非常具体的目的地址，对于以太网来 说，就是网卡的MAC地址（不是FF-FF-FF-FF-FF-FF这样的地址）。现在的具有路由功能的主机应该可以将单播数据定向转发，而目的主机的网 络接口则可以过滤掉和自己MAC地址不一致的数据
##### 3.1.6.2.广播
###### 广播是主机针对某一个网络上的所有主机发送数据包。这个网络可能是网络，可能是子网，还可能是所有的子网。如果是网络，例如A类网址的广播就是 netid.255.255.255，如果是子网，则是netid.netid.subnetid.255；如果是所有的子网（B类IP）则是则是 netid.netid.255.255。广播所用的MAC地址FF-FF-FF-FF-FF-FF。网络内所有的主机都会收到这个广播数据，网卡只要把 MAC地址为FF-FF-FF-FF-FF-FF的数据交给内核就可以了。一般说来ARP，或者路由协议RIP应该是以广播的形式播发的
##### 3.1.6.3.多播
###### 可以说广播是多播的特例，多播就是给一组特定的主机（多播组）发送数据，这样，数据的播发范围会小一些(实际上播发的范围一点也没有变小)，多播的MAC地址是最高字节的低位为一，例 如01-00-00-00-00-00。多播组的地址是D类IP，规定是224.0.0.0-239.255.255.255。<br>虽然多播比较特殊，但是究其原理，多播的数据还是要通过数据链路层进行MAC地址绑定然后进行发送。所以一个以太网卡在绑定了一个多播IP地址之后，必 定还要绑定一个多播的MAC地址，才能使得其可以像单播那样工作。这个多播的IP和多播MAC地址有一个对应的算法，在书的p133到p134之间。可以看到这个对应不是一一对应的，主机还是要对多播数据进行过滤。<br>个人的看法：广播和多播的性质是一样的，路由器会把数据放到局域网里面，然后网卡对这些数据进行过滤，只拿到自己打算要的数据，比如自己感兴趣的多 播数据，自己感兴趣的组播数据。当一个主机运行了一个处理某一个多播IP的进程的时候，这个进程会给网卡绑定一个虚拟的多播mac地址，并做出来一个多播 ip。这样，网卡就会让带有这个多播mac地址的数据进来，从而实现通信，而那些没有监听这些数据的主机就会把这些数据过滤掉，换句话说，多播，是让主机 的内核轻松了，而网卡，对不起，您就累点吧。
##### 3.1.6.4.IGMP协议
###### IGMP协议即Internet组管理协议（Internet Group Manager Protocol），其作用在让其他所有需要知道自己处于哪个多播组的主机和路由器知道自己的状态。一般多播路由器根本不需要知道某一个多播组里面有多少个主机，而只要知道自己的子网内还有没有处于某个多播组的主机就可以了。主要某个多播组还有一台主机，多播路由器就会把数据传输出去，这样接收方就会通过网卡过滤功能来得到自己想要的数据。为了知道多播组的信息，多播路由器需要定时发送IGMP查询，各个多播组里面的主机要根据查询来回复自己的状态。路由器来决定有几个多播组，自己要对某一个多播组发送什么样的数据。
#### 3.1.7.TCP协议
##### 3.1.7.1.简单介绍
###### TCP和UDP处于同一层--运输层，TCP通过以下简单的工作原理<br>1.应用数据被分隔成TCP认为最合适发送的数据块。这和UDP完全不同，应用程序产生的数据报长度将保持不变。由TCP传递给IP的信息单位称为报文段或段（segment）<br>2.当TCP发出一段后，它自动启动一个定时器，等待目的端确认收到这个报文段。如果超时，则重发这个报文段<br>3.TCP接收端收到数据报后会向发送端发送一个确认，但这个确认不是立即发送，通常将延迟几分之一秒<br>4.TCP将保持他首部和数据的校验和。这是一个端到端的校验和，如果接收端校验和不通过，将抛弃该报文段并不确认收到此报文段（触发超时重发）<br>5.TCP可以接受顺序失序的报文段，并进行重新排序<br>6.TCP提供流量控制。TCP接收端只允许另一端发送接收端能够接纳的数据。这能防止较快主机致使较慢主机的缓冲区溢出
##### 3.1.7.2.DNS系统
###### DNS全称Domain Name System，它负责吧FQDN翻译成一个IP。DNS系统是一个巨大的树，最上方有一个无名树根，下一层arpa，com，cn，edu，us等等。其中arpa是域名反解析树的顶端，如果没有这个反解析树，反向查询时必须要遍历整个数据库，这将带来巨大的负担。DNS服务器有高速缓存功能
###### DNS查询支持TCP和UDP两种协议的查询方式，而且端口都是53。而大多数的查询都是UDP查询。
##### 3.1.7.3.TCP连接的建立--三次握手
###### 客户端首先向服务器申请打开某一个端口（用SYN段等于1的TCP报文）；然后服务器端发送一个ACK+SYN报文通知客户端请求报文收到；然后客户端收到服务端的ACK报文后要再次发出确认报文用来确认收到刚才服务端发出的ACK报文
###### 在建立连接的时候，通信双方要互相确认对方的最大报文长度（MSS），以便通信。一般这个MSS是MTU减去固定IP首部和TCP首部的长度。对于一个以太网，一般可以达到1460字节。当然如果对于非本地IP，这个MSS可能就只有536字节，而且如果中间的传输网络的MSS更加小，这个值还会变得更小
##### 3.1.7.4.TCP连接的终止--四次挥手
###### TCP有一个特别的概念叫做half-close，这个概念是说，TCP的连接是全双工连接，因此在关闭连接的时候，必须关闭传和送两个方向上的连接。主动关闭方发送一个FIN为1的TCP报文，然后被动关闭方返回给主动关闭方一个确认的ACK报文，并且发送一个FIN报文，当主动关闭方回复ACK报文后，连接就结束了
##### 3.1.7.5.TCP状态转换图
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/TCP_STATE1.bmp)
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/TCP_STATE2.bmp)
> ###### MSL为任何IP数据报能够在因特网中存活的最长时间
###### 客户端状态迁移图：CLOSED->SYN_SENT->ESTABLISHED->FIN_WAIT_1->FIN_WAIT_2->TIME_WAIT_CLOSED（没那么简单）
###### 服务端状态迁移图：CLOSED-LISTEN->SYN_RECEIVED->ESTABLISHED->CLOSE_WAIT->LAST_ACK->CLOSED（没那么简单）
###### TIME_WAIT状态的必要性：<br>1.可靠地实现TCP全双工连接的终止<br>2.允许老的重复分节（数据报）在网络中消逝
###### 还存在一种情况会关闭TCP连接，使用RST信号。如果出现异常，主动方发送一个RST信号，即可丢弃TCP缓存区并关闭本方的TCP连接，被动方接收到RST信号后也可以直接丢弃TCP缓存区并关闭TCP连接，连ACK都不用发送
##### 3.1.7.6.TCP的交互数据流
###### 目前建立在TCP协议上的网络协议特别多，比如telnet，ssh，ftp，http等。这些协议可以大致地分为两类，第一种是交互数据类型：例如telnet，ssh，这种类型的协议大多数情况下只做小流量的数据交换，比如说按一下键盘，回显一些文字等；第二种是数据成块类型，例如ftp，这种类型的协议要求TCP能尽量的运载数据，把数据的吞吐量做到最大，并尽可能的提高效率。
###### 捎带ACK的发送方式：这个策略是说，当主机收到远程主机的TCP数据报之后，通常不马上发送ACK，而是等上一个短暂的时间，如果这段时间里面主机还有发送到远程主机的数据报，那么就把这个ACK数据报捎带着发送出去，吧本来两个TCP数据报整合成一个数据报。一般这个等待时间是200ms
###### Nagle算法：当主机A个主机B发送一个数据报并进入等待主机B的ACK数据报的状态时，TCP的输出缓冲区里面只能有一个TCP数据报，并且这个数据报不断的收集后来的数据，整合成一个大的数据报，等到主机B的ACK包一到，就把这些数据一股脑的发送出去
##### 3.1.7.7.TCP的成块数据流
###### 传输数据时ACK的问题：一般来说，发送端发送一个TCP数据报，那么接收端就应该发送一个ACK数据报。但事实上并不是这样，发送端将会连续发送数据尽量填充满对方的缓存区，而接收方对这些数据只要发送一个ACK报来回应就可以了，这就是ACK的累积特性，这个特性大大减少了发送端和接收端的负担
###### 滑动窗口：
###### 滑动窗口的本质上时描述接收方的能接受数据报的大小，发送方根据这个数据来计算自己最多能发送多长的数据。如果发送方收到接收方的窗口大小为0的TCP数据报，那么发送方将停止数据报发送，直到接收方发送窗口大小不为0的数据报到来。发送缓冲区内有四部分数据：已经发送已经确认的数据，已经发送没有确认的数据，可以发送还没发送的数据，不能发送的数据；其中第二种第三种数据在滑动窗口内。
###### 由于窗口大小是接收方通过数据报发送给发送方的。某次发送方接收到0窗口大小的数据报后，会停止发送数据，等待接收方发送窗口大于0的数据报，但这个数据报在网络中丢失。所以这时候发送方在等待接收方的窗口大于0的数据报，而接收方在等待发送方的数据数据报，所以进入了死锁。为了防止死锁，发送方在窗口用完后会开启一个持续定时器，当定时器溢出时，发送一个1字节的探测报文，对方会在此时回应自身的接受窗口大小，如果结果仍然是零窗口报文，则重设持续定时器，继续等待；如果不是零窗口，则开始发送数据；如果没收到怎么办？
###### 滑动窗口有多种协议，停等协议，后退n协议，选择重传协议
###### 配合Nagle算法，先发一个字节，获得窗口后，等数据长度达到窗口一半大小或最大大小时再把数据发出去。接收方发送窗口报文也要等待一段时间，等到窗口空间足够容纳一个报文段或者有缓存空间一半大小时，再通知发送方发送数据
###### 数据拥塞：
###### 网路中窗口大小小于接收方窗口大小时，会导致网络拥塞。
###### 解决方法，慢开始、拥塞控制、快重传、快恢复
###### 1.发送方维持一个拥塞窗口的变量，拥塞窗口和接收方窗口共同决定发送窗口。
###### 2.当主机开始发送数据时，先选择发送一个字节的试探报文
###### 3.当接收到第一个字节的确认后，就发送两个字节
###### 4.当接收到后两个字节的确认后吗，发送四个字节。一次递增2的指数级
###### 5.最后会打到一个提前预设的“慢开始门限”，比如24，即当发送字节数多于24时，如果拥塞窗口大小小于慢开始门限，则继续使用慢开始算法，如果拥塞窗口大于慢开始门限，停止使用慢开始算法，换用了拥塞避免算法，如果等于，则两种算法选一个
###### 6.所谓拥塞避免算法就是每经过一个往返时间RTT就吧发送方的拥塞窗口+1，即让拥塞窗口线性地增大
###### 7.当出现网络拥塞，比如丢包时，将慢开始门限设为原来的一般，然后将拥塞窗口设为1，执行慢开始算法。
###### 8.快重传和快恢复算法是为了减少因为拥塞导致的数据报丢失带来的重传时间，从而避免传递无用的数据到网络，其机制是：
###### 8.1.接收方如果一个包丢失，则当接收到后续的包时继续发送针对该包的ACK确认
###### 8.2.一旦发送方接收到三个一样的确认，就知道该包之后出现了错误，立刻使用快恢复算法重传该包：
###### 8.2.1.慢开始门限减半
###### 8.2.2.拥塞窗口大小设为慢开始门限减半后的数值
###### 8.2.3.执行拥塞避免算法（高起点，线性增长）
##### 3.1.7.8.超时定时器
###### 如何确定超时的时间设置，太长会造成网络利用率不高，太短会造成多次重传，使得网络阻塞
##### 3.1.7.9.持续定时器
###### 见3.1.7.7，用于防止零窗口报文丢失导致的死锁现象
##### 3.1.7.10.保活定时器
###### 保活定时器用于保持TCP只连接不传送数据的半开放连接，并在某种情况下释放这种连接。需要提到一点，如果其中一端崩溃了，重新启动后收到该端“前生”的保活探测，则要发送一个RST数据报文包住另一端结束连接
##### 3.1.7.11.2MSL定时器
###### 见3.1.7.5
## 4.zookeeper
### 4.1.概念
###### Zookeeper是一个分布式的，开放源码的分布式应用程序协调服务，是Google的Chubby一个开源的实现，他是集群的管理者，监视着集群中各个节点的状态，根据节点提交的反馈，进行下一步合理操作，最终，将简单易用的接口和性能高效、功能稳定的系统提供给用户
### 4.2.Zookeeper提供了什么
#### 4.2.1.文件系统
###### 每个子目录想入NameService都被称作为znode，和文件系统一样，我们能够自由的增加、删除znode，在一个znode下增加、删除子znode，唯一的不同在于znode是可以存储数据的
##### 4.2.1.1.znode的类型：
###### PERSISTENT 持久化目录节点，客户端与Zookeeper断开连接后，该节点依旧存在
###### PERSISTENT_SEQUENTIAL 持久化顺序编号目录节点，客户端与Zookeeper断开连接后，该节点依旧存在，只是Zookeeper给该节点进行顺序编号
###### EPHEMERAL 临时目录节点，客户端与Zookeeper断开连接后，该节点被删除
###### EPHEMERAL_SEQUENTIAL 临时顺序编号目录节点，客户端与Zookeeper断开连接后，该节点被删除，只是Zookeeper给该节点进行顺序编号
#### 4.2.2.通知机制
###### 客户端注册监听它关心的目录节点，当目录节点发生变化（数据改变、被删除、子目录节点增加删除）时，Zookeeper会通知客户端
### 4.3.Zookeeper能做什么
#### 4.3.1.命名服务
###### 在Zookeeper的文件系统中创建一个目录，即有唯一的path。在我们使用tborg无法确定上游程序的部署机器时即可与下游程序约定好path，通过path即能互相探索发现
#### 4.3.2.配置管理
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/ZookeeperConfigManager.png)
###### 程序总是需要配置的，如果程序分散部署在多台机器上，要逐个改变配置变得十分困难。现在把这些配置全部放到Zookeeper上去，保存在Zookeeper的某个目录节点中，然后所有相关应用程序对这个目录节点进行监听，一旦配置信息发生变化，每个应用程序就会收到Zookeeper的通知，然后获取新的配置信息应用到系统中就好了
#### 4.3.3.集群管理
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/ZookeeperGroupManager.png)
###### 所谓集群管理无外乎两点：是否有机器退出和加入，选举master
###### 对于第一点，所有机器约定在父目录GroupMembers下创建临时目录节点，然后监听父目录节点的子节点变化消息，一旦有机器挂掉，该机器与Zookeeper的连接断开，其所创建的临时目录节点被删除，所有其他机器都收到通知：某个兄弟目录被删除，于是，所有人都知道：它下线了。新机器的加入也是蕾西，所有机器收到通知：新兄弟目录加入
###### 对于第二点，稍加变更，所有机器创建临时顺序编号目录节点，每次选取编号最小的机器作为master
#### 4.3.4.分布式锁
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/ZookeeperDistributedLock.png)
###### 有了Zookeeper的一致性文件系统，锁的问题变得容易。锁服务可以分为两类，一个是保持独占，另一个是控制时序。
###### 对于第一类，我们将Zookeeper上的znode看作是一把锁，通过createznode的方法来实现。所有的客户端都去创建/distribute_lock节点，最终成功创建的那个客户端也即拥有了这把锁。用完删除掉自己创建的distributed_lock节点就释放出锁
###### 对于第二类，/distributed_lock可以已经预先存在，所有客户端在他下面创建临时顺序编号目录节点，和选master一样， 编号最小的获得锁，用完删除，依次方便
#### 4.3.5.队列管理
###### 两种类型的队列：一种同步队列，当一个队列的成员都聚齐时，这个队列才可用，否则一直等待所有成员到达；另一种是队列按照FIFO方式进行入队和出队操作
###### 对于第一类，在约定的目录下创建临时目录节点，监听节点数目是否是我们要求的数目
###### 对于第二类，和分布式锁服务中的控制时序场景基本原理一致，入队有编号，出队有编号
#### 4.3.6.分布式与数据复制
###### Zookeeper作为一个为集群提供一致的数据服务，自然，他要在所有机器间做数据复制。数据复制有如下好处：<br>1.容错，一个节点出错，不至于让整个系统停止工作，别的节点可以接管他的工作<br>2.提高系统的扩展能力：把负载分布到多个节点上去，或者增加节点来提高系统的负载能力<br>3.提高性能：让客户端本地访问就近的节点他，提高用户访问速度
###### 从客户端读写访问的透明度来看，数据复制集群系统分为下面两种：<br>1.写主：对数据的修改提交给指定的节点。读无此限制，可以读取任何一个节点。这种情况下，客户端需要对读和写进行区别，俗称读写分离<br>2.写任意：对数据的修改可提交给任意的节点，跟读一样。这种情况下，客户端对集群节点的角色与变化透明
###### 对于Zookeeper而言，它采用的是写任意。通过增加机器，他的读吞吐能力和响应能力扩展性非常好，而且，随着机器的增多，吞吐能力肯定下降（这也是他简历observer的原因），而响应能力则取决于具体实现方式，是延迟复制保持最终一直性还是立即复制快速响应
#### 4.3.7.Zookeeper角色描述
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/ZookeeperRoleDescribe.png)
#### 4.3.8.Zookeeper与客户端
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/ZookeeperClient.png)
#### 4.3.9.Zookeeper设计目的
###### 最终一致性：client不论连接到哪个server，展示给它的都是同一个视图，这是Zookeeper的重要性能
###### 可靠性：具有简单、健壮、良好的性能，如果消息被一台服务器接受，那么它将被所有的服务器接受
###### 实时性：Zookeeper保证客户端将在一个时间间隔范围内获得服务器的更新信息，或者服务器失效信息。但由网络延时等原因，Zookeeper不能保证两个客户端能同时得到刚更新的数据，如果需要最新数据，应该在读数据之前调用sync()接口
###### 等待无关：慢的或者失效的client不得干预快速的client的请求，使得每个client都能有效地等待
###### 原子性：更新只能成功或者失败，没有中间状态
###### 顺序性：包括全局有序和偏序两种。全局有序是指如果在一台服务器上消息a在消息b前发布，则在所有服务器上消息a都将在消息b前被发布；偏序是指如果一个消息b在消息a后备同一个发送者发布，a必将排在b前面？？？
#### 4.3.10.Zookeeper工作原理
###### Zookeeper的核心是原子广播，这个机制保证了各个服务器之间的同步。实现这个机制的协议叫做Zab协议。Zab协议有两种模式，它们分别是恢复模式（选主）和广播模式（同步）。当服务启动或者在领导者崩溃后，Zab就进入了恢复模式，当领导者被选举出来，且大多数服务器完成了和leader的状态同步后，恢复模式就结束了，状态同步保证了leader和follower具有相同的系统状态
###### 为了保证食物的顺序一致性，Zookeeper采用了递增的事务id号（zxid）来标识事务。所有的提议（proposal）都在被提出的时候加上了zxid。实现中zxid是一个64位的数字，它高32位是epoch，用来标识leader关系是否改变，每次一个leader被选出来，他都会有一个新的epoch，标识当前属于哪个leader的统治时期。低32位用于递增计数。
#### 4.3.11.Zookeeper下Server的工作状态
###### 有三种状态：LOOKING（当前Server不知道leader是谁，正在搜寻），LEADING（当前Server即为选举出来的leader），FOLLOWER（leader已经选举出来，当前Server与之同步）
#### 4.3.12.Zookeeper选主流程（basic paxos）
###### 当leader崩溃或者leader失去大多数的follower，这时候zk进入恢复模式，恢复模式需要重新选举出一个新的leader，让所有的Server都恢复到一个正确的状态。Zk的选举算法有两种，一种是局域basic paxos实现的，另外一种是基于fast paxos算法实现的。系统默认的选举算法为fast paxos
###### 1.选举线程由当前Server发起选举的线程担任，其主要功能是投票结果进行统计，并选出推荐的Server；<br>2.选举线程首先向所有Server发起一次询问（包括自己）<br>3.选举线程收到回复后，验证是否是自己发起的询问（验证zxid是否一致），然后获取对方的id（myid），并存储到当前询问对象列表中，最后获取对方提议的leader相关信息（id，zxid），并将这些信息存储到档次选举的投票记录表中；<br>4.当收到所有Server回复后，就计算出zxid最大的那个Server，并将这个Server相关信息设置成下一次要投票的Server；<br>5.线程将当前zxid最大的Server设置为当前Server要推荐的Leader，如果此时获胜的Server获得n/2+1的Server票数，设置当前推荐的leader为获胜的Server，将根据获胜的Server相关信息设置自己的状态，否则，继续这个过程，知道leader被选举出来。通过流程分析，我们可以得出：要使Leader获得多数Server的支持，则Server总数必须是奇数2n+1，且存活的Server的数目补的少于n+1，每个Server启动后都会重复以上流程。在恢复模式下，如果是刚才崩溃状态恢复的或者刚启动的server还会从磁盘快照中恢复数据和绘画信息，zk会记录事务日志并定期进行快照，方便在恢复时进行状态恢复。选主的具体流程图如下：
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/ZookeeperBasicPaxos.png)
#### 4.3.13.Zookeeper选主流程（fast paxos）
###### fast paxos流程是在选举过程中，某Server首先向所有Server提议自己要成为leader，当其他Server收到提议后，解决epoch和zxid 的冲突，并接受对方的提议，然后向对方发送接受提议完成的消息，重复这个流程，最后一定能选出leader
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/ZookeeperFastPaxos.png)
#### 4.3.14.Zookeeper同步流程
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/ZookeeperSync.png)
###### 选完leader以后，zk就进入状态同步过程：<br>1.Leader等待server连接<br>2.follower连接leader，将最大的zxid发送个leader<br>3.leader根据follower的zxid确定同步点<br>4.完成同步后通知follower已经成为uptodate状态<br>5.follower收到uptodate消息后，又可以重新接受client的请求进行服务了
#### 4.3.15.Zookeeper工作流程-leader
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/ZookeeperLeaderWork.png)
###### 1.恢复数据<br>2.维持与learner的心跳，接受learner请求并判断learner的请求消息类型<br>3.Learner的消息类型主要有PING消息（Learner的心跳信息）、REQUEST消息（Follower发送的提议信息，包括写请求及同步请求）、ACK消息（Follower的对提议的回复，超过半数Follower通过，则commit该提议）、REVALIDATE消息（用来延长SESSION有效时间），根据不同的消息类型，进行不同的处理
#### 4.3.16.Zookeeper工作流程-follower
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/ZookeeperFollowerWork.png)
###### Follower主要有四个功能：<br>1.向leader发送请求（PING请求、REQUEST请求、ACK消息、REVALIDATE消息）<br>2.接受Leader消息并进行处理<br>3.接受Client的请求，如果为写请求，发送给Leader进行投票<br>4.给Client返回结果
###### Follower的消息循环处理如下几种来自Leader的消息：PING消息（心跳消息）、PROPOSAL消息（Leader发起的提案，要求Follower投票）、COMMIT消息（服务器端最新一次提案的信息）、UPTODATE消息（表明同步完成）、REVALIDATE消息（更具Leader的REVALIDATE结果，关闭revalidate的session还是允许其接受消息）、SYNC消息（返回SYNC结果到客户端，这个消息最初由客户端发起，用来强制得到最新的更新）
## 5.redis
## 6.异常分级
## 7.中间件
## 8.Java集合继承关系图
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JavaGroup1.bmp)
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JavaGroup2.bmp)
###### 数组虽然也可以存储对象，但长度是固定的；集合长度是可变的，数组中可以存储基本数据类型，集合只能存储对象
###### 上述类图中，实线边框的是实现类，比如ArrayList，LinkedList，HashMap等，折线边框的是抽象类，比如AbstractCollection，AbstractList，AbstractMap等，而点线边框的是接口，比如Collection，Iterator，List等
###### Collection是一个集合接口，而Collections是一个包装类，提供了很多静态多态方法，此类就像一个工具类，不能被实例化，服务于Java 的Collection框架
###### 关于Collections类中同步集合的方法，要考虑两点，首先整个操作必须要原子性，Collections只能保证集合类中的方法的线程安全，如果自定义的方法中调用了两次集合方法，并且两处会产生线程安全问题，就需要在自定的方法上加锁，再者自定义方法上加锁必须加到集合上，而不是加到方法上，加到方法上那个锁其实是加载方法所在的类或对象上，而Collections的线程安全方法加的锁是加在集合上的
### 8.1.Iterator接口
###### Iterator接口，这是一个用于遍历集合中元素的接口，主要包含hashNext(),next(),remove()三种方法。他的一个子接口ListIterator在它的基础上又添加了三种方法，分别是add(),previous(),hasPrevious()。也就是说如果实现Iterator接口，那么在遍历集合中元素的时候，只能往后遍历，被遍历后的元素不会再遍历到，通常无序集合实现的都是这个接口，比如HashSet，HashMap；而那些元素有序的集合，实现的一般都是ListIterator接口，实现这个接口的集合可以双向遍历，比如ArrayList<br>Collection依赖于Iterator，是因为Collection的实现类都要实现iterator()函数，返回一个Iterator对象。
### 8.2.Map接口
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JavaGroupMap.bmp)
###### Collection属于单值的操作，是单列集合，Map中的每个元素都是用key-value的形式存储在集合中，是双列集合。Map没有直接取出元素的方法，而是先转成Set集合，再通过迭代获取元素
###### Map是个映射接口，即key-value键值对，Map中的每一个元素包含一个key和一个value，key不能重复，每个key最多只能映射一个值，有以下实现：HashMap、Hashtable、Properties、LinkedHashMap、IdentityHashMap、TreeMap、WeakHashMap、ConcurrentHashMap。有一个抽象类，AbstractMap，它实现了Map接口中大部分的API。
###### Map接口提供三种Collection视图。允许以键集、值集合键值映射关系集的形式查看映射的内容。由于键值的唯一性，所以键集、键值集都是Set，而值集是Collection
###### Map的实现类提供两个“标准的”构造方法，第一个无参构造，用于创建空映射；第二个带有单个Map类型参数的构造方法，用于创建一个与其参数具有相同键值映射关系的新映射。
###### Map中定义了一个接口interface Entry<K, V>，定义了键值对。
#### 8.2.1.AbstractMap
###### AbstractMap是继承于Map的抽象类，它实现了Map中的大部分API。其他Map的实现类可以通过继承AbstractMap来减少重复编码
#### 8.2.2.SortedMap
###### SortedMap是继承与Map的接口。SortedMap中的内容是排序的键值对，排序的方法是自然排序或者通过用户指定的比较器（Comparator）
###### 四种构造函数：无参构造方法，带Comparator参数的构造方法，带Map参数的构造方法（自然排序），带SortedMap类型参数的构造方法
#### 8.2.3.NavigableMap
###### NavigableMap是继承于SortedMap的接口。相比于SortedMap，NavigableMap有一系列的导航方法；如“获取大于/等于某对象的键值对”、“获取小于/等于某对象的键值对”等等
###### 提供操作键值对的方法：lowerEntry，floorEntry，ceilingEntity和higherEntity方法，它们分别返回小于，小于等于，大于等于，大于给定键的键关联的Map.Entry对象；firstEntry，pollFirstEntry，lastEntry，pollLastEntry方法，它们返回和/或移除最小和最大的映射关系（如果存在），否则返回null
###### 提供操作键的方法：lowerKey，floorKey，ceilingKey和higherKey方法，它们分别返回小于，小于等于，大于等于，大于给定键的键
###### 提供获取键集的方法：navigableKeySet、descendingKeySet分别获取正序/反序的键集
###### 提供获取键值对的子集的方法
#### 8.2.4.Dictionary
###### Dictionary是JDK1.0定义的键值对的抽象类，方法全抽象，更像一个接口，但当时还没有接口的概念
#### 8.2.5.HashMap
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JavaGroupHashMap.jpg)
###### HashMap继承于AbstractMap，但没有实现NavigableMap接口，因此HashMap的内容是“不保证次序的键值对”。它还实现了Cloneable，java.io.Serializable接口。它不是线程安全的，它的key和value都可以为null
###### HashMap的实例有两个参数影响其性能：“初始容量”和“加载因子”。容量是哈希表中桶的数量，初始容量只是哈希表在创建是的容量。加载因子是一个阈值，当哈希表充满到这个阈值时，哈希表会扩容（rehash）
###### HashMap有四个构造方法：HashMap();HashMap(int capacity);HashMap(int capacity, float loadFactor);HashMap(Map<? extends K, ? extends V> map)
###### HashMap是一个散列表，通过拉链法解决哈希冲突。它包含几个重要的成员变量：<br>1.table是一个Entry[]数组类型，而Entry实际上是一个单向链表。哈希表的键值对都存储在Entry数组中<br>2.size是HashMap的大小，它是HashMap保存键值对的数量<br>3.threshold是HashMap的阈值，用于判断是否需要调整HashMap的容量（加倍）。threshold=容量*加载因子<br>4.loadFactor就是加载因子<br>5.modCount是用来实现fail-fast机制的
###### HashMap包含三个集合，所以有三种方式的遍历
#### 8.2.6.Hashtable
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JavaGroupHashtable.jpg)
###### Hashtable和HashMap一样，是一个散列表，通过拉链法解决哈希冲突。Hashtable继承自Dictionary，实现了Map，Cloneable，java.io.Serializable接口。Hashtable是同步的，线程安全，且他的key和value不能为null，映射不是有序的
###### Hashtable包含几个重要的成员变量：<br>1.table是一个Entry[]数组类型，而Entry实际上是一个单向链表。哈希表的键值对都存储在Entry数组中<br>2.count是Hashtable的大小，它是Hashtable保存键值对的数量<br>3.threshold是Hashtable的阈值，用于判断是否需要调整Hashtable的容量（加倍）。threshold=容量*加载因子<br>4.loadFactor就是加载因子<br>5.modCount是用来实现fail-fast机制的
###### Hashtable内定义了Enumeration类，同时实现了Enumerator接口和Iterator接口，提供了通过elements()方法遍历Hashtable的接口和通过entrySet()遍历Hashtable的接口
###### Hashtable可以通过键值对集合遍历（entrySet()方法）、键集合遍历（keySet()方法）、值集合遍历（value()方法）、Enumeration遍历键（Enumeration enu = table.keys()）、Enumeration遍历值（Enumeration enu = table.elements()）
#### 8.2.7.TreeMap
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JavaGroupTreeMap.jpg)
###### TreeMap继承于AbstractMap，且实现了NavigableMap接口，因此TreeMap中的内容是“有序的键值对”（通过红黑树实现）
###### TreeMap包含几个重要的成员变量：<br>1.root是红黑树的根节点，他是Entry类型，Entry是红黑树的节点，它包含了红黑树的六个基本组成成分：key、value、left、right、parent、color。Entry是根据key进行排序<br>2.size是红黑树中节点的个数<br>3.comparator是用来红黑树用来进行排序的
###### 
#### 8.2.8.WeakHashMap
###### WeakHashMap继承与AbstractMap，它和HashMap的区别是WeakHashMap的键是弱键，会被GC
#### 8.2.5.LinkedHashMap
### 8.3.Collection接口
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JavaGroupCollection.bmp)
###### public interface Collection<E> extends Iterable<E>,Collection接口的所有子类（直接子类和间接子类）都必须实现两种构造函数：不带参数的构造函数和参数为Collection的构造函数。带参数的构造函数，可以用来转换Collection的类型
###### Collection是一个接口，是高度抽象出来的集合，它包含了集合的基本操作（添加，删除，清空，遍历，是否为空，获取大小，时候包含某元素）和属性。Collection包含了List和Set两大分支。List是一个有序的队列，每一个元素都有它的索引。第一个元素的索引值是0。Set是数学概念中的集合，Set中没有重复的元素
#### 8.3.1.AbstractCollection抽象类
###### AbstractCollection实现了Collection中除iterator()和size()之外的绝大部分方法，可以通过继承AbstractCollection省去重复编码。
#### 8.3.1.List接口
###### public interface List<E> extends Collection<E>{}，List是一个继承于Collection的接口，即List是集合中的一种。List是有序的队列，List中的每个元素都有一个索引；第一个元素的索引值是0，往后的元素的索引值依次+1。与Set不同，List可以存放重复的元素
###### List除了包含Collection中的全部方法接口，由于List是有序队列，它还额外包含自己的API接口，主要有添加，删除，获取和修改指定位置的元素，获取List中的子队列等。
#### 8.3.2.Set接口
###### public interface Set<E> extends Collection<E>{}
###### Set的API和Collection完全一样。Set不能存放重复的内容，靠hashCode()和equals()两个方法判断内容是否重复

#### 8.3.3.Queue接口
###### 队列接口
## 9.String
## 10.数据结构（数组，链表，队列，栈，树，图，堆，散列表，红黑树）
## 11.JVM，GC
## 12.决策树
## 13.操作系统生产者消费者
## 14.模板类和泛型类的区别
###### Java的泛型靠的还是类型擦除，目标代码只会生成一份，牺牲的是运行速度。
###### C++的模板会对针对不同的模板参数静态实例化，目标代码体积会稍大一些，运行速度会快很多。
## 15.Java设计模式
## 16.回调函数
#### 回调函数的概念
##### 最原始的回调函数是C语言中，回调函数实现了底层调用高层函数的功能。高层在调用底层函数时，将一个函数指针传给被调用的函数，当被调用的函数返回时，会调用该函数指针所指的函数，同时将返回值以参数的形式传给这个函数指针对应的函数。这个函数指针所指的函数就是回调函数。所以回调函数的意思就是调用了一个函数后，被调用的函数回过来调用高层的函数
#### java回调函数的一般实现
##### 首先要设计一个接口a，里面可以定义一些方法。类b里面保留一个接口a的属性property-c，并设置property-c的setter方法。类d里实例化类b，并传入一个实现了接口a的引用给类b，在特定的事件驱动下类b会调用传入的实现接口a的对象的方法，实现了回调
# 区块链 找程硕 骆志坤 截图

----------

# 项目总结
### 分层设计 软删除 异常分级
### RBAC权限管理
##### RBAC比较复杂，有用户表，用户组表，角色表，权限表以及这些表之间的关联表。权限表还会管理一大堆网络资源比如页面资源，方法资源，菜单资源等。
##### RBAC第一个修改点：RBAC用户组概念和角色概念有比较大的重复，比如一个网站有管理员和普通用户，那么是不是就可以分为管理员用户组和普通用户组，然后管理员和普通用户又可以作为两种角色，这样的话两个概念我觉得是冗余的。所以我把用户组概念去掉了，然后把用户组的层级概念在角色表中实现，比如最高级的角色是管理员，下属普通用户角色。高层角色的权限有两个部分，一个是所有低层角色的权限，另一个是自身的权限。
##### RBAC对网站资源的定义十分复杂，也不好实现。使用Spring Security来管理所有的网站资源，进行粗粒度的权限管理
##### 最终实现的RBAC：首先有一个用户表，通过关联表关联角色表，一个用户可以有多个角色。角色表中的角色有父子关系，父角色拥有所有子孙角色的权限及父角色自身的权限。角色表通过关联表关联权限表，一个角色可以拥有多个权限，每个权限可以被多个角色拥有。权限表记录权限的名称和权限的描述。修改Spring Security的UserDetailService，自定义如何根据用户名查询到所拥有的权限（也就是权限表中记录的权限名称），并返回结果。并将UserDetailService注入到AuthenticationManager中。然后在控制器中用@Security注解用权限名称标注资源，即这样，通过修改两张关联表，就可以修改用户权限，实现了权限的动态访问。
##### 遇到两个问题：
1. 想要修改权限名称的前缀，本来是ROLE_。一般的修改方法是：修改AccessDecisionManager中的RoleVoter，通过注入rolePrefix属性修改前缀，然后将修改完的AccessDecisionManager注入给对于FilterSecurityInterceptor或者直接在<security:http/>中通过access-decision-manager-ref引入我们修改后的AccessDecisionManager，就实现前缀的修改。但发现这个AccessDecisionManager不是全局的。@secured注解的方法是通过AOP进行方法拦截后再进行权限认证。进过代码跟踪和原码查看，发现是基于cglib动态代理实现的AOP最终会在MethodSecurityInterceptor中实现方法拦截，然后调用MethodSecurityInterceptor中的AccessDecisionManager进行权限验证。MethodSecurityInterceptor中的AccessDecisionManager并不能通过access-decision-manager-ref修改（通过调用栈查看，确实不是同一个），而且，虽然我们可以通过注入的方式将AccessDecisionManager注入给MethodSecurityInterceptor，但我们自定义的MethodSecurityInterceptor无法注入到系统的（找了半天没找到方法），于是乎，对于@secured包括其他几种注解方式进行权限定义就不能修改ROLE_前缀。于是乎在找
到解决方法前，就不改前缀了，全用ROLE_。
2. <security:global-method-security secured-annotations="enabled" />注解添加到spring-security.xml后，并不能拦截请求，并不会进行权限验证。原因在于springsecured对于注解是通过AOP拦截到方法后，调用决策方法的。spring-mvc中管理的bean所在的容器是spring容器的一个子容器，父容器看不到子容器中定义的bean，所以就更不可能拦截这些bean中的方法调用。然后我在controller的方法上进行了@secured注解，controller bean都是由spring-mvc托管的，所以需要把<security:global-method-security secured-annotations="enabled" />注解添加到spring-mvc.xml中，就可以生效了。
### 父类公共属性注入问题
###### 使用Mybatis时，先写了一个抽象的通用的DAO，包括增删查改，然后为每个需要持久化的Bean编写DAO都继承自通用的DAO。然后Mybatis进行数据库操作需要SqlSessionTemplate。如果直接在父类中进行属性注入，为了代码简介优美，希望通过注解让Spring自动注入。如果直接在属性中进行注入，由于抽象父类是不会实例化的，所以Spring无法进行注入

解决了父类公共属性的注入问题：
public abstract class fatherclass{
    private String commonproperty;
    @Autowire
    public void SetCommonproperty(String commonproperty){
         this.commonproperty = commonproperty;
    }
}
public class sonclass{
    public void function(){
         System.out.println(super.commonproperty);
    }
}