# 基本知识

----------

## 1.数据库
### 1.1.数据库ACID原则-数据库事务正确执行的四个基本要素
#### 1.1.1.原子性（Atomicity）：
###### 整个事务中的所有操作，要么全部完成，要么全部不完成不可能停滞在中间某个环节。事务在执行过程中发生错误，会被回滚到事务开始前的状态，就像这个事务从来没有执行过一样
#### 1.1.2.一致性（Consistency）：
###### 一个事务可以封装状态改变（除非它是一个只读的）。事务必须始终保持系统处于一致的状态，不管在任何给定的时间并发事务有多少。<br>也就是说：如果事务是并发多个，系统也必须如同串行事务一样操作。其主要特征是保护性和不变形，以转账案例为例，假设有五个账户，每个账户余额是100元，那么五个账户总额是500元，如果在这个5个账户之间同时发生多个转账，无论并发多少个，这5个账户总额也应该还是500元，这就是保护性和不变形。
#### 1.1.3.隔离性（Isolation）：
###### 隔离状态执行事务，使它们好像是在系统给定的时间内执行的唯一操作。如果有两个事务，运行在相同的时间内，执行相同的功能，事务的隔离性将确保每一事务在系统中认为只有个该事物在使用系统。这种属性有时称为串行化，为了防止事务操作间的混淆，必须串行化或序列化请求，是的在同一时间仅有一个请求用于同一数据。
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
###### 幻读是事务费独立执行是发生的一种现象，例如事务T1对一个表中所有的行的某个数据项做了从“1”修改为“2”的操作，这是事务T2又对这个表中插入了一行数据项，而这个数据项的数值还是为“1”并且提交给数据库。而操作事务T1的用户如果再查看刚刚修改的数据，会发生还有一行没有修改，其实这行是从事务T2中添加的，就好像发生了幻觉一样，这就是幻读。<br>幻读好人不可重复读都是读取了另一条已经提交的事务（这点就脏读不同），所不同的是不可重复读查询的都是同一个数据项，而幻读针对的是一批数据整体（比如数据的个数）。避免幻读需要锁表，避免不可重复读只需要锁行，幻读一般是由于插入或者删除操作，不可重复读一般是由于更新操作
#### 1.2.4.MySQL四种隔离级别：
###### Serializable（串行化）：可以避免脏读、不可重复读、幻读的发生。（锁表）
###### Repeatable read（可重复读）：可以避免脏读、不可重复读的发生。（锁行）
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
#### 2.1.8.线程变量 todo
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
###### redis是一个开源的支持网络、可基于内存亦可持久化的日志型、Key-Value数据库。它的value类型相对更多，包括string字符串、list链表、set集合、zset有序集合、hash哈希类型，这些数据类型都支持push/pop、add/remove以及取交集并集和差集等等丰富的操作，而且这些操作都是原子性的。在此基础上，redis支持各种不同方式的排序。redis的数据都缓存在内存中，但会周期性地把数据写入磁盘或者把修改操作写入追加的记录文件，并在此基础上实现了主从同步
## 6.异常分级
## 7.中间件
## 8.Java集合继承关系图
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JavaGroup1.bmp)
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JavaGroup2.bmp)
###### 数组虽然也可以存储对象，但长度是固定的；集合长度是可变的，数组中可以存储基本数据类型，集合只能存储对象
###### 上述类图中，实线边框的是实现类，比如ArrayList，LinkedList，HashMap等，折线边框的是抽象类，比如AbstractCollection，AbstractList，AbstractMap等，而点线边框的是接口，比如Collection，Iterator，List等
###### Collection是一个集合接口，而Collections是一个包装类，提供了很多静态多态方法，此类就像一个工具类，不能被实例化，服务于Java 的Collection框架
###### 关于Collections类中同步集合的方法，要考虑两点，首先整个操作必须要原子性，Collections只能保证集合类中的方法的线程安全，如果自定义的方法中调用了两次集合方法，并且两处会产生线程安全问题，就需要在自定的方法上加锁，再者自定义方法上加锁必须加到集合上，而不是加到方法上，加到方法上那个锁其实是加载方法所在的类或对象上，而Collections的线程安全方法加的锁是加在集合上的
###### Collections.synchronizedMap()的实现原理：todo
### 8.1.Iterator接口
###### Iterator接口，这是一个用于遍历集合中元素的接口，主要包含hashNext(),next(),remove()三种方法。他的一个子接口ListIterator在它的基础上又添加了三种方法，分别是add(),previous(),hasPrevious()。也就是说如果实现Iterator接口，那么在遍历集合中元素的时候，只能往后遍历，被遍历后的元素不会再遍历到，通常无序集合实现的都是这个接口，比如HashSet，HashMap；而那些元素有序的集合，实现的一般都是ListIterator接口，实现这个接口的集合可以双向遍历，比如ArrayList<br>Collection依赖于Iterator，是因为Collection的实现类都要实现iterator()函数，返回一个Iterator对象。
### 8.2.Map接口
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JavaGroupMap.jpg)
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
###### TreeMap继承于AbstractMap，且实现了NavigableMap接口，因此TreeMap中的内容是“有序的键值对”（通过红黑树实现）；实现了Cloneable接口，实现了Serializable接口
###### TreeMap包含几个重要的成员变量：<br>1.root是红黑树的根节点，他是Entry类型，Entry是红黑树的节点，它包含了红黑树的六个基本组成成分：key、value、left、right、parent、color。Entry是根据key进行排序<br>2.size是红黑树中节点的个数<br>3.comparator是用来红黑树用来进行排序的
###### 看源码吧~~~~
#### 8.2.8.WeakHashMap
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JavaGroupWeakHashMap.jpg)
###### WeakHashMap和HashMap一样，是一个散列表，存储键值对，键和值都可以为null。WeakHashMap是不同步的，线程不安全
###### WeakHashMap继承与AbstractMap，它和HashMap的区别是WeakHashMap的键是弱键，会被GC，准确地说，其存储的映射的存在不会阻止GC回收。这就使该键可以被终止，被回收；该键一旦被终止时，它对应的键值对就从映射中有效的移除了。弱键的原理，大致上就是通过WeakReference和ReferenceQueue实现的。WeakHashMap的key是弱键，即是WeakReference类型的；ReferenceQueue是一个队列，它会保存被GC回收的弱键：<br>当某弱键不再被其他对象引用，并被GC回收时，在GC该弱键时，这个弱键也同时会被添加到ReferenceQueue队列中<br>当下一次我们需要操作WeakHashMap时，会先同步table个queue。table中保存了全部键值对，而queue中保存被GC回收的键值对，同步他们，就是删除table中被GC回收的键值对
###### WeakHashMap中有几个重要的成员变量：<br>1.table为一个Entry[]数组类型，用来存储键值对<br>2.size是散列表中保存键值对的数量<br>3.threshold是散列表的阈值，关系到是否需要调整散列表的大小，threshold=容量*loadFactory<br>4.loadFactor为加载因子，关系到是否需要调整散列表的大小<br>5.modCount是用来实现fail-fast机制的<br>6.queue是用来保存已被GC清除的弱引用的键
#### 8.2.5.LinkedHashMap
###### todo
#### 8.2.6.Map总结
##### 8.2.6.1.概括
###### (1)Map是键值对映射的抽象接口<br>(2)AbstractMap实现了Map中的绝大部分函数接口。它减少了Map的实现类的重复编码<br>(3)SortedMap是有序的键值对映射接口<br>(4)NavigableMap是继承与SortedMap的，支持导航函数的接口<br>(5)HashMap是基于拉链法实现的散列表，一般用于单线程程序中；Hashtable也是基于拉链法实现的散列表，一般用于多线程程序中（其实不用）；WeakHashMap也是基于拉链法实现的散列表，也踊跃单线程程序中，但他的见识弱键；TreeMap是有序的散列表，通过红黑树实现，用于单线程
##### 8.2.6.2.HashMap与Hashtable
###### 相同点：HashMap和Hashtable都是存储键值对的散列表，都采用拉链法解决哈希冲突。散列表的结构都是一个Entry数组，而Entry是一个单向链表。添加键值对是，先根据key值计算出哈希值，再计算出数组索引，然后遍历单向链表，将key和链表中的每个节点的key进行对比，若key已经存在，则替换，都在新建键值对节点，将该节点插入到单向链表表头位置。删除键值对，同样先计算key的哈希值，然后计算索引，然后遍历单向链表，如果找到节点，即删除
###### 不同点1：HashMap继承于AbstractMap（在JDK1.2新增的类），实现了Map、Cloneable、java.io.Serializable接口，而Hashtable继承于Dictionary，实现了Map、Cloneable、java.io.Serializable接口。Dictionary是一个抽象类，继承于Object类，没有实现任何接口，是在JDK1.0时引入的。虽然Dictionary也支持添加键值对，获取值，获取大小等基本操作，但它的API比Map少，而且Dictionary一般是通过Enumeration（枚举类）去遍历，Map则是通过Iterator（迭代器）去遍历。由于Hashtable也实现了Map，所以Hashtable既可以用Enumeration遍历，也可以用Iterator遍历
###### 不同点2：Hashtable几乎所有的函数都是同步的，即它是线程安全的，支持多线程；而HashMap函数时非同步的，它不是线程安全的。可以使用Collections.synchronizedMap静态方法，或者直接使用JDK5.0后提供额java.util.concurrent包里的ConcurrentHashMap
###### 不同点3：HashMap的键值都可以为null（key为null的键值对会固定插入到table[0]位置，即HashMap散列表的第一个位置），Hashtable的键值都不可以为null
###### 不同点4：HashMap只支持Iterator遍历，Hashtable同时支持Enumeration和Iterator遍历；两者Iterator遍历方式也不同，HashMap是从前向后遍历table，然后再对单向链表从表头开始遍历，Hashtable是从后向前遍历table，然后再对单向链表从表头开始遍历
###### 不同点5：HashMap的默认容量是16，增加容量时新容量为原容量的两倍（可以减少扩容时的计算，后面详述todo）；Hashtable的默认容量是11，增加容量时新容量为原容量的两倍加一
###### 不同点6：键值对的hash值算法不同，HashMap使用自定义的哈希算法，Hashtable直接使用key的hashCode()
###### 不同点7：Hashtable支持contains(Object value)，而且重写了toString()方法；HashMap不支持contains(Object value)方法（使用containsValue(Object value)），没有重写toString()方法
##### 8.2.6.3.HashMap和WeakHashMap
###### 相同点：都是散列表，都保存键值对，都继承于AbstractMap，并且实现Map，构造函数都一样，默认容量都是16，默认加载因子都是0.75，都允许null作为key和value，都是非同步的
###### 不同点1：HashMap实现了Cloneable和Serializable接口，而WeakHashMap没有
###### 不同点2：WeakHashMap是弱键，它的键是弱引用，而HashMap的键是强引用
### 8.3.Collection接口
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JavaGroupCollection.jpg)
###### public interface Collection<E> extends Iterable<E>,Collection接口的所有子类（直接子类和间接子类）都必须实现两种构造函数：不带参数的构造函数和参数为Collection的构造函数。带参数的构造函数，可以用来转换Collection的类型
###### Collection是一个接口，是高度抽象出来的集合，它包含了集合的基本操作（添加，删除，清空，遍历，是否为空，获取大小，时候包含某元素）和属性。Collection包含了List和Set两大分支。List是一个有序的队列，每一个元素都有它的索引。第一个元素的索引值是0。Set是数学概念中的集合，Set中没有重复的元素
### 8.3.1.List接口
###### public interface List<E> extends Collection<E>{}，List是一个继承于Collection的接口，即List是集合中的一种。List是有序的队列，List中的每个元素都有一个索引；第一个元素的索引值是0，往后的元素的索引值依次+1。与Set不同，List可以存放重复的元素
###### List除了包含Collection中的全部方法接口，由于List是有序队列，它还额外包含自己的API接口，主要有添加，删除，获取和修改指定位置的元素，获取List中的子队列等。
#### 8.3.2.Set接口
###### public interface Set<E> extends Collection<E>{}
###### Set的API和Collection完全一样。Set不能存放重复的内容，靠hashCode()和equals()两个方法判断内容是否重复
#### 8.3.3.AbstractCollection
###### public abstract class AbstractCollection<E> implements Collection<E>{}
###### AbstractCollection实现了Collection中除iterator()和size()之外的绝大部分方法，可以通过继承AbstractCollection省去重复编码。
#### 8.3.4.AbstractList
###### public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E>{}
###### AbstractList是一个继承于AbstractCollection，并且实现List接口的抽象类。它实现了List中除了size()、get(int location)之外的所有方法。AbstractList的作用在于实现了List接口中的大部分方法，从而方便其他类继承实现List。和AbstractCollection相比，AbstractList中，对iterator()接口进行了实现
#### 8.3.5.AbstractSet
###### public abstract class AbstractSet<E> extends AbstractCollection<E> implements Set<E>{}
###### AbstractSet是一个继承于AbstractCollection，并且实现Set接口的抽象类。由于Set接口和Collection接口中的API完全一样，Set没有自己单独的API，所以AbstractSet和AbstractCollection一样，实现了实现了Set中除了size()和iterator()之外的所有方法。
#### 8.3.6.Iterator
###### public interface Iterator<E>{}
###### Iterator是一个接口，他是集合的迭代器。集合可以通过Iterator去遍历集合中的元素。Iterator提供的API接口，包括：是否存在下一个元素，获取下一个元素，删除当前元素。
###### fail-fast:Iterator遍历Collection是fail-fast机制的，即，当某一个线程A通过iterator去遍历某集合的过程中，若其他线程B改变了集合的内容，那么线程A访问集合时就会抛出ConcurrentModificationException异常，产生fail-fast事件。fail-fast会在后面详述
#### 8.3.7 ListIterator
###### public interface ListIterator<E> extends Iterator<E>{}
###### ListIterator是一个继承于Iterator的接口，他是队列迭代器，专门用于遍历List，能提供向前/向后遍历。相比于Iterator，它新增了添加元素，是否存在上一个元素，获取上一个元素等等API接口
### 8.4.List
#### 8.4.1.ArrayList
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JavaGroupArrayList.jpg)
###### ArrayList是一个数组队列，相当于动态数组，与Java中的数组相比，它的容量能动态增大。它继承与AbstractList，实现了List。实现了RandomAccess接口，即提供了随机访问功能。实现了Cloneable接口，覆盖了clone()方法，能被克隆，实现了Serializable接口，即支持序列化。和Vector不同，ArrayList的操作不是线程安全的。
###### ArrayList包含了两个重要的对象elementData和size。<br>elementData是“Object[]类型的数组”,它保存了添加到ArrayList中的元素。实际上elementData是一个动态数组，我们能通过构造函数ArrayList(int initialCapacity)来设置他的初始容量，如果使用没有参数的构造函数，默认的initialCapacity为10。elementData的大小会根据ArrayList的容量增加而动态增长（ensureCapacity()方法）<br>size是动态数组的实际大小
###### ArrList可以通过三种方式访问：随机访问，for循环遍历，通过迭代器访问，三种方式依次效率变低
###### ArrayList提供了两个toArray()函数：<br>Object[] toArray()，会抛出java.lang.ClassCastException异常，是在Object[]转化为其他类型是造成的，因为Java不支持向下转型<br><T> T[] toArray(T[] contents)不会抛出异常
```
//toArray(T[] contents)调用方式一
public static Integer[] vectorToArray(ArrayList<Integer> v){
    Integer[] newText = new Integer[v.size()];
    v.toArray(newText);
    return newText;
}
//toArray(T[] contents)调用方式二，最常用
public static Integer[] vectorToArray(ArrayList<Integer> v){
    Integer[] newText = (Integer[])v.toArray(new Integer[0]);
    return newText;
}
//toArray(T[] contents)调用方式三
public static Integer[] vectorToArray(ArrayList<Integer> v){
    Integer[] newtext = new Integer[v.size];
    Integer[] newStrings = (Integer[])v.toArray(newText);
    return newStrings;
}
```
#### 8.4.2.fail-fast总结
###### fail-fast简介：fail-fast机制是java集合中的一种错误机制，当多个线程对同一个集合的内容进行操作时，有可能就会产生fail-fast事件，如线程A正在通过Iterator遍历集合，线程B改变了集合的内容，那么线程A中就会抛出ConcurrentModificationException，产生fail-fast事件
###### fail-fast原理：通过modCount变量值是否改变来保证在Iterator遍历集合时集合数据是否被改变。无论是add(),remove()还是clear()，都会改变modCount的值
###### fail-fast解决：fail-fast机制是一种错误检测机制，它只能用来被检测错误，因为JDK并不保证fail-fast机制一定会发生。若在多线程环境下使用fail-fast机制的集合，建议使用java.util.concurrent包下的类去取代java.util包下的类。java.util.concurrent包中比如CopyOnWriteArrayList中实现了一个ListIterator，名为CopyOnWriteIterator，在新建这个Iterator时，会将集合中的元素保存在一个新的拷贝数组中，遍历的是拷贝数组
#### 8.4.3.LinkedList
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JavaGroupLinkedList.jpg)
###### public class LinkedList<E> extends AbstractSequentialList<E> implements List<E>, Deque<E>, Cloneable, java.io.Serializable{}
###### AbstractSequentialList实现了get(int index),set(int index, E element),add(int index, E element)和remove(int index)这些函数。这些接口都是随机访问List的；我们若需要通过AbstractSequentialList自己实现一个列表，只需要扩展此类，并提供listIterator()和size()方法的实现即可。若要实现不可修改的列表，则需要实现列表迭代器的hasNext、next、hasPrevious、previous和index方法即可。
###### LinkedList的本质是双向链表，包含两个重要的成员：<br>1.header是双向链表的表头，它是双向链表节点所对应的类Entry的实例。Entry中包含成员变量：previous，next，element，分别表示上一个节点，下一个节点，当前节点所包含的值<br>2.size是双向链表中节点的个数
###### LinkedList既然是双向链表，那么它的顺序访问会非常高效，而随机访问效率比较低。但LinkedList又实现了List接口{也就是实现了get(int index), remove(int index)等根据索引值来获取删除节点的函数}，LinkedList用来将双向链表和索引值联系起来的原理非常简单，是通过一个计数索引值来实现的。例如我们调用get(int index)时，首先会比较index和双向链表长度的1/2，若前者大，则从链表头开始往后查找，直到找到index位置，否则从链表末尾开始向前查找
###### 由于LinkedList实现了Deque接口，而Deque接口定义了在双端队列两端访问元素的方法。提供插入、移除和检查元素的防范。没中方法都存在两种形式：一种形式操作失败时抛出异常，另一种形式返回一个特殊值（null或false，具体取决于操作）
| |第一个元素（头部）|最后一个元素（尾部）|
|------|------|------|
| |抛出异常|特殊值|抛出异常|特殊值|
|插入|addFirst(e)|offerFirst(e)|addLast(e)|offerLast(e)|
|移除|removeFirst()|pollFirst()|removeLast()|pollLast()|
|检查|getFirst()|peekFirst()|getLast()|peekLast()|
###### LinkedList可以作为FIFO的队列，作为队列时，下表的方法等价：
|队列方法|等效方法|
|------|------|
|add(e)|addLast(e)|
|offer(e)|offerLast(e)|
|remove()|removeFirst()|
|poll()|pollFirst()|
|element()|getFirst()|
|peek()|peekFirst()|
###### LinkedList可以作为LIFO的栈，作为栈时，下表方法等价：
|队列方法|等效方法|
|------|------|
|push(e)|addFirst(e)|
|pop()|removeFirst(e)|
|peek()|peekFirst()|
###### LinkedList遍历方式：<br>1.通过迭代器<br>2.随机访问<br>3.for循环遍历<br>4.用pollFirst()遍历<br>5.用pollLast()遍历<br>6.用removeFirst()遍历<br>7.用removeLast()遍历
#### 8.4.4..Vector
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JavaGroupVector.jpg)
###### public class Vector<E> extends AbstractList<E> implements List<E>, RandomAccess, Clonneable, java.io.Serializable{}
###### Vector是矢量队列，是JDK1.0版本添加的类，继承于AbstractList，实现了List（它是一个队列，支持添加，删除，修改，遍历），RandomAccess（支持随机访问），Cloneable（能被克隆）这些接口。是同步的，大多数方法都是多线程安全
###### Vector的数据结构额ArrayList差不多，它包含了三个成员变量：<br>1.elementData是Object[]类型的数组，它保存了添加到Vector中的元素，是一个动态数组，如果没有指定，默认大小为10<br>2.elementCount是动态数组的实际大小<br>3.capacityIncrement是动态数组的增长系数。如果在创建Vector时，指定了capacityIncrement是动态数组的增长系数，则每次增加的大小都是capacityIncrement
###### Vector可以用迭代器遍历、随机访问、for循环访问、Enumeration遍历
#### 8.4.5.Stack
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JavaGroupStack.jpg)
###### public class Stack<E> extends Vector<E>{}
###### Stack是栈，特征是先进后出，Java中的Stack继承于Vector，所以Stack也是通过数组实现，而非链表。当然我们也可以将LinkedList当做栈来使用
###### Stack通过Vector实现，代码非常简单
#### 8.4.6.List总结
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JavaGroupList.jpg)
##### 8.4.6.1.概括
###### List是继承于Collection的接口，代表着有序的队列
###### AbstractList是一个抽象类，继承于AbStractCollection，它实现了List接口中出size()和get(int location)以外的所有方法
###### AbstractSequentialList是一个抽象类，继承于AbstractList，实现了链表中根据index索引值操作链表的全部方法
###### ArrayList，LinkedList，Vector，Stack是List的四个实现类。ArrayList是一个数组队列，相当于动态数组，它由数组实现，随机访问效率高，随机插入、随机删除效率低；LinkedList是一个双向链表，它也可以被当做栈、队列或双端队列进行操作，他的随机访问效率略低，但随机插入、随机删除效率高；Vector是矢量队列，和ArrayList一样，他也是一个动态数组，由数组实现。但ArrayList非线程安全，Vector线程安全；Stack是栈，继承于Vector。
##### 8.4.6.4.Vector和ArrayList比较
###### 相同处：他们都是List，都继承于AbstractList，并实现了List接口；都实现了RandomAccress和Cloneable接口；都通过数组实现，本质上都是动态数组；默认的数组容量都是10；都支持Iterator和ListIterator遍历
###### 不同处1：线程安全不一样：ArrayList是非线程安全，Vector是线程安全
###### 不同处2：对序列化的支持不同，ArrayList支持序列化，而Vector不支持
###### 不同处3：构造函数个数不同，Vector增加了一个可以指定容量增加系数的构造函数
###### 不同处4：容量增加方式不同：ArrayList新容量=(原容量*3)/2+1；Vector容量增长与增长系数有关，若指定增长系数则新容量=原容量+增长系数，若没有指定增长系数，则新容量=原容量*2
###### 不同处5：Enumeration支持Enum遍历，而ArrayList不支持
### 8.5.Set
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JavaGroupSet.jpg)
###### Set的实现类都是基于Map来实现的，比如HashSet是通过HashMap实现的，TreeSet是通过TreeMap实现的
###### Set是继承于Collection的接口，它是一个不允许有重复元素的集合；AbstractSet是一个抽象类，它继承了AbstractCollection，它实现了Set中的绝大部分函数，为Set的实现类提供了便利；HashSet依赖于HashMap，是Set的一个实现类；TreeSet依赖于TreeMap，是Set的另一个实现类
#### 8.5.1..HashSet
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JavaGroupHashSet.jpg)
###### public class HashSet<E> extends AbstractSet<E> implements Set<E>, Cloneable, java.io.Serializable{}
###### HashSet是一个没有重复元素的集合，允许使用null元素，但不保证元素的顺序，是非同步的。HashSet的Iterator是fail-fast的
###### HashSet中有一个成员变量map，是HashMap类型，HashSet操作函数，实际上都是通过map实现的。HashMap保存的是键值对，而HashSet只需要保存key即可，所以该map中的value都等于PRESENT = new Object();
###### HashSet的遍历：<br>1.通过迭代器遍历：set.iterator();<br>2.通过for-each遍历：先使用toArray函数获得数组，然后用foreach遍历数组
#### 8.5.2..TreeSet
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JavaGroupTreeSet.jpg)
###### public class TreeSet<E> extends AbstractSet<E> implements NavigableSet<E>, Cloneable, java.io.Serializable{}
###### TreeSet是一个有序的没有重复元素的集合，不允许使用null元素，是非同步的。TreeSet的Iterator是fail-fast的
###### TreeSet是基于TreeMap实现的，TreeSet有一个重要的成员变量m，是NavigableMap类型的，而m实际上是TreeMap的实例
###### TreeSet的遍历：<br>1.Iterator顺序遍历<br>2.Iterator逆序遍历<br>3.toArray foreach遍历
#### 8.5.3.Iterator和Enumeration的区别
###### Enumeration只有两个函数接口(hasMoreElements(), nextElement())，只能对读取集合的数据，而不能对数据进行修改；Iterator有三个函数接口(hasNext(), next(), remove())，除了能读取数据之外，还能对数据进行删除
###### Iterator支持fail-fast机制，而Enumeration不支持。<br>1.Enumeration是JDK1.0中加入的，Enumeration存在的目的就是为它们提供遍历接口。Enumeration本身并不支持同步，而在Vector、Hashtable实现Enumeration时，添加了同步；<br>2.Iterator是JDK1.2才添加的接口，它也是为了HashMap、ArrayList等集合提供遍历接口。Iterator是支持fail-fast机制。
## 9.String，StringBuffer，StringBuilder
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/String.jpg)
### 9.1.String和CharSequence详解
###### String是java中的字符串，它继承于CharSequence。CharSequence是一个接口，它只包括length(),charAt(int index),subSequence(int start, int end)这几个接口。除了String实现了CharSequence之外，StringBuilder和StringBuffer也实现了CharSequence。String，StringBuilder和StringBuffer本质上都是通过字符数组实现的
###### CharSequence十分简单
```java
package java.lang;
public interface CharSequence {
    int length();
    char charAt(int index);
    CharSequence subSequence(int start, int end);
    public String toString();
}
```
###### String本质是一个字符序列，是通过字符数组实现的，有很多API；String是final修饰的（public final class String），String中的字符数组也是final修饰的（private final char value[]），所以String是一个字符串常量，每次改变String，其实都生成了一个新的String对象
### 9.2.AbStractStringBuilder详解
###### AbstractStringBuilder是核心，StringBuilder和StringBuffer很多方法的实现是调用父类的方法完成的。
### 9.3.StringBuilder详解
###### StringBuilder是一个可变的字符序列，继承于AbStractStringBuilder，实现了CharSequence接口，StringBuilder是线程不安全的
### 9.4.StringBuffer详解
###### StringBuffer是一个可变的字符序列，继承于AbStractStringBuilder，实现了CharSequence接口，StringBuffer是线程不安全的
### 9.5.StringBuilder和StringBuffer区别
###### StringBuilder 和 StringBuffer都是可变的字符序列。它们都继承于AbstractStringBuilder，实现了CharSequence接口。但是，StringBuilder是非线程安全的，而StringBuffer是线程安全的。
## 10.数据结构
### 10.1.散列表
###### 散列表也成为哈希表，可以根据key直接访问数据。通过散列函数（哈希函数）根据key计算出数据存放的位置，存放数据的数组叫散列表
#### 10.1.1.hash冲突
###### 即不同的key计算出来的hash值一致，解决哈希冲突的方法主要有一下几种。下面哈希函数表示为H(key);
##### 10.1.1.1.开放寻址法
###### 如果H(key)冲突，则求得一个地址序列：
```
H0,H1,H2,...,Hs  1 <= s <= m-1，
其中： Hi = H(key),            i = 0
       Hi=(H(key)+di) mod m    i=1,2,...,s
其中： m是表的长度
       di可以有三种取法：
            线性探测再散列：di = 1,2,3,...,m-1
            平方探测再散列：di = 1^2,-1^2,2^2,-2^2,3^2,-3^2,...,k^2,-k^2，k<=m/2
            随机探测再散列：di是一组伪随机数列      
当产生哈希冲突时，依次使用H1,H2,...,Hs作为位置，直到找到一个空单元或者查遍全表                 
```
##### 10.1.1.2.再散列法
###### 再散列法就是同事构造多个不同的哈希函数，当发生哈希冲突时，使用下一个哈希函数，直到冲突不再产生或哈希函数耗尽，这种方法不易产生聚集，但增加了计算时间
##### 10.1.1.3.拉链法
###### 相同hash值的数据保存在一个链表中
##### 10.1.1.4.建立一个公共溢出区
###### 无话可说
### 10.2.堆，堆排序
###### 堆是一个完全二叉树，分为最小堆和最大堆，最小堆的定义就是所有父节点都比子节点要小，最大堆则相反。
###### 堆有一下种操作：上浮，下沉，插入（push，加入到堆底，然后进行上浮操作），弹出（pop，将堆顶与堆底交换，然后删除堆底，对堆顶进行下沉操作），堆排序
#### 10.2.1.堆排序算法，最大堆可以产生逆序，最小堆可以产生顺序。
##### 10.2.1.1.构建最大堆
###### 分为两层循环，外循环从最后一个节点的父节点开始，每次循环节点索引减一，一直循环到1，这个循环是从下往上的，每循环完一次，保证以该节点为根节点的子树是一个最大堆。至于怎么保证，则由内层循环实现。内存循环是从上往下的（其实就是下沉操作），由于外层循环是从下往上的，所以保证了该节点的左右子树都是最大堆，所以比较节点和两个子树的根节点也就是节点的两个孩子的大小，如果节点最大，则结束循环，如果不是，则将最大的孩子和节点互换位置，将以最大孩子原位置作为根节点的子树重复当前操作，直到树底。
```
//构建最大堆
template<class T>
void maxHeap<T>::buildBigTree(T* theHeap, int theSize)
{
	delete[] heap; //删除堆中原先的元素
	heap = theHeap; //指针赋值，将参数中数组的元素赋值给堆
	heapSize = theSize; //元素个数赋值
 
	//执行循环，从最后一个节点的父节点开始，逐个向上，外层循环
	for(int root = heapSize/2; root >= 1; --root)	
	{
		T theElement = heap[root]; //保存外层循环的当前结点，每次都可以理解为给这个值找位置。
 
		//内层循环开始，给theElement找位置，将大的上移，大节点原先的位置变成空位置。
		int currentNode = root;	//始终是空位置的下标，也是child下标的父节点位置
		int child = 2*root;	//当前结点左孩子下标
		while(child <= heapSize)
		{
			if(child < heapSize && heap[child+1] > heap[child])
				++child;	//令child指向值较大的孩子
 
			//如果theElement比两个孩子都大，那么就证明找到位置
			if(theElement > heap[child])
				break;
 
			//如果没有找到，将大的节点放在空位置上，大节点原先的位置变为空位置
			heap[currentNode] = heap[child];
			current = child;
			//更新孩子下标，继续向下寻找位置
			child *= 2;	
		}
	}
	//找到位置，将theElement放在空位置上
	heap[currentNode] = theElement;
}
```
##### 10.2.1.2.堆排序
###### 以逆序排序为例，首先构建一次最大堆，然后将堆顶元素和堆底元素互换，这时候，堆底元素就是有序区，其他元素为无序区；然后再构造一次最大堆，将堆顶元素和堆底倒数第二个元素互换，这时候堆底元素和堆底第二个元素都为有序区，其他元素为无序区；以此类推，知道所有元素都为有序区
### 10.3.红黑树
#### 10.3.1.简介
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/RBTree.jpg)
###### 红黑树是一种特殊的二叉查找树，红黑树的每个节点都有存储位表示节点的颜色，可以是红或黑。其特点是：<br>1.每个节点或者是黑的，或者是红的；<br>2.根节点必然是黑色的；<br>3.每个叶子节点（NIL或者null）是黑色的；<br>4.如果一个节点是红色的，则它的子节点必须是黑色的；<br>5.从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点（需要确保没有一条路径会比其他路径长出两倍。因而，红黑树是相对接近平衡的二叉树）
#### 10.3.2.基本操作
##### 10.3.2.1.左旋
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/RBTreeLeftRotate1.jpg)
###### 红黑树的基本操作时添加、删除。在对红黑树进行添加或删除之后，都会用到旋转方法。为什么呢？道理很简单，添加或删除红黑树中的节点之后，红黑树就发生了变化，可能不满足红黑树的五条性质，也就不再是一棵红黑树。而通过旋转，可以重新成为红黑树
###### 如上图，对x进行左旋意味着将x变成一个左节点。如果一棵树在左旋前是一个二叉查找树，那么在左旋后依然是一个二叉查找树
##### 10.3.2.2.右旋
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/RBTreeRightRotate1.jpg)
###### 如上图，对x进行右旋意味着将x变成一个右节点。如果一棵树在右旋前是一个二叉查找树，那么在右旋后依然是一个二叉查找树
##### 10.3.2.3.添加
###### todo
##### 10.3.2.4.删除
###### todo
### 10.4.图（有向图，无向图）
###### 图是一种由定点组成的抽象网络，网络中的各定点可以通过边实现彼此的连接，表示两定点有关联。注意上面图定义中的两个关键字，由此得到我们最基础的两个概念，顶点和边。
###### 图可以分为无向图和有向图，根据边是否有向来确定。
###### 边的权重也是一个非常核心的概念，每条边都有与之对应的值。边的权重可以是0，正数或者负数
#### 10.4.1.树是一种很特别的图：任意两点之间都连通，并且没有环的图。树有自身的特点和概念：节点，枝，根，叶，度（一个节点拥有的子树数称为节点的度），层（选定根以后，每个点离根的距离，可以将树中的点分为多个层级），深度/高度（一个树的最大层级数）（叶子也有高度），双亲，孩子，兄弟，祖先，后代，森林（很多棵树的集合称为森林）
#### 10.4.2.图的遍历
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/map.png)
##### 10.4.2.1.深度优先搜索（DFS）
###### 深度优先就是从北京出发，随便找一个相连的城市，如济南，作为访问的城市，然后从济南出发，随便找一个与济南相连的城市，比如南京，再从南京出发，随便找一个与南京相连的城市，如上海，一直找到头，直到找不到城市，再往回找。因为一条路“走到黑”，所以叫深度优先
###### 深度优先搜索一般用栈实现：<br>1.把起始点放入栈，进行标记<br>2.重复下述两个动作，直到栈为空<br>    2.1.从栈中访问栈顶的点（不取出）<br>    2.2.如果此点有领接的且尚未遍历的点，则选择一个点进行标记并放入栈，否则将这个点从栈弹出
##### 10.4.2.2.广度优先搜索（BFS）
###### 广度优先就是从北京出发，先访问那些直接与北京相连的城市，比如天津、沈阳、包头、太原、郑州、济南等；然后再访问那些城市和这些已访问过的城市相连，如长春与沈阳相连，武汉与郑州相连，南京与济南相连等。而后再访问与长春、武汉、南京等相连的城市，如哈尔滨、上海等，直到把所有的城市都访问一遍为止
###### 广度优先搜索一般用队列实现：<br>1.把起始点放入队列，进行标记<br>2.取出队列头（不放回），找出所有的领接点，进行标记并放入队列中<br>3.重复2，直到队列为空
#### 10.4.3.图的领接表表示方法
###### 领接表的核心思想就是针对每个顶点设置一个邻居表
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/MapLJB.png)
```
//上图的领接表可以表示为（错误）
N = [   {b, c, d, e, f},    #a的邻居表
        {c, e},             #b的邻居表
        {d},                #c的邻居表
        {e},                #d的邻居表
        {f},                #e的邻居表
        {c, g, h},          #f的邻居表
        {f, h},             #g的邻居表
        {f, g}]             #h的邻居表
```
#### 10.4.4.图的领接矩阵表示方法
###### 领接矩阵的核心思想是针对每个顶点设置一个表，这个表包含了所有顶点，通过true/false来表示是否是邻居顶点
```
//上图的邻接矩阵可以表示为（错误）
N = [   {0,1,1,1,1,1,0,0},  #a的邻居表
        {0,0,1,0,1,0,0,0},  #b的邻居表
        {0,0,0,1,0,0,0,0},  #c的邻居表
        {0,0,0,0,1,0,0,0},  #d的邻居表
        {0,0,0,0,0,1,0,0},  #e的邻居表
        {0,0,1,0,0,0,1,1},  #f的邻居表
        {0,0,0,0,0,1,0,1},  #g的邻居表
        {0,0,0,0,0,1,1,0}]  #h的邻居表
```
#### 10.4.5.最短路径
##### 10.4.5.1.DFS或BFS（单源最短路径）
###### 从起始开始访问所有的DFS路径或者BFS路径，到达终点的路径有多条，取其中路径权值最短的一条则为最短路径
```
//DFS
//0是路，1是墙 
#include<iostream>
#include<algorithm>
using namespace std;
int x1,y1,x2,y2;//起点坐标终点坐标
int c=999999; //步数 
int dir[4][2]={-1,0,1,0,0,-1,0,1};
int mg[5][5]={0,0,1,1,1,
              0,0,1,1,1,
              0,1,0,1,1,
              1,0,1,1,1,
              0,0,0,0,0
			 };
void dfs(int x,int y,int t)
{
	int i,xx,yy;
	if (x==x2&&y==y2)//到达终点 
	{
		c=min(c,t);//取最小值 
		return ;
	}
	mg[x][y]=1;//将走过的路设1，以免下次又以为这是一条路，又走回来 
	for (i=0;i<4;i++)
	{
		xx=x+dir[i][0];
		yy=y+dir[i][1];//生成新的方向坐标 
		if (xx<0||xx>=5||yy<0||yy>=5||mg[xx][yy]==1)//超出地图，或者为墙,则要重新换个方向走 
		 continue;
		dfs(xx,yy,t+1);//步数+1，以xx,yy为新的坐标，来进行对下次的方向进行选择又生成新的xxyy 
		mg[xx][yy]=0;//这一步是当走到终点了，或者是有一个xxyy坐标上下左右都不能走，则要将刚走过的路(之前设为墙了)恢复成路 
	}
}
int main()
{
	cin>>x1>>y1>>x2>>y2;
	dfs(x1,y1,0);
	cout<<c;
	return 0;
}
```
```
//BFS
#include<iostream>
#include<algorithm>
#include<queue>
using namespace std;
struct st
{
   int x,y,c=0;//行列坐标xy，c为步数	
};
queue<struct st> xz;//创建一个队列xy 
int dir[4][2]={-1,0,1,0,0,-1,0,1};
int mg[5][5]={0,0,1,1,1,
              0,0,0,1,1,
              0,1,0,1,1,
              0,0,0,1,1,
              0,0,0,0,0
			 };
int bfs(struct st s,struct st e)
{
	struct st t;
	int i;
	xz.push(s);//先将起点坐标的信息入队 
	mg[s.x][s.y]=1;
	while (!xz.empty())//队列为空，说明到达终点，完毕 
	{
		s=xz.front(); //获取队头元素 
		xz.pop();//删除后队列为空 
		if (s.x==e.x&&s.y==e.y)//到达终点，返回步数c 
		  return s.c;
		for (i=0;i<4;i++)
		{
			t.x=s.x+dir[i][0];
			t.y=s.y+dir[i][1];
			if (t.x<0||t.x>=5||t.y<0||t.y>=5||mg[t.x][t.y])//不能走 
			 continue;
			 t.c=s.c+1;  //步数加1 
			 mg[t.x][t.y]=1;//走过的路赋1 
			 xz.push(t);// 将新的坐标和步数信息t入队 
		}
	}
}
int main()
{
	struct st s,e;
	cin>>s.x>>s.y>>e.x>>e.y;
	cout<<bfs(s,e)<<endl;
	return 0;
}

```
##### 10.4.5.2.弗洛伊德算法（多源最短路径）
###### Floyd算法是一个经典的动态规划算法。用通俗的语言来描述的话，首先我们的目标是寻找从点i到点j的最短路径。从动态规划的角度看问题，我们需要为这个目标重新做一个诠释。<br>从任意节点i到任意节点j的最短路径不外乎2种可能，一是直接从i到j，二是从i经过若干个节点k到j。所以，我们假设Dis(i,j)为节点u到节点v的最短路径的距离，对于每一个节点k，我们检查Dis(i,k) + Dis(k,j) < Dis(i,j)是否成立，如果成立，证明从i到k再到j的路径比i直接到j的路径短，我们便设置Dis(i,j) = Dis(i,k) + Dis(k,j)，这样一来，当我们遍历完所有节点k，Dis(i,j)中记录的便是i到j的最短路径的距离
###### (1)用一个数组来存储任意两个点之间的距离，注意，这里可以是有向的，也可以是无向的。当任意两点之间不允许经过第三个点时，这些城市之间最短路程就是初始路程。 
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/Floyd1.jpg)
###### (2)如果现在只允许经过1号顶点，求任意两点之间的最短路程，只需判断e[i][1]+e[1][j]是否比e[i][j]要小即可。e[i][j]表示的是从i号顶点到j号顶点之间的路程。e[i][1]+e[1][j]表示的是从i号顶点先到1号顶点，再从1号顶点到j号顶点的路程之和。其中i是1~n循环，j也是1~n循环
```
for(i=1;i<=n;i++) {   
    for(j=1;j<=n;j++) {   
        if ( e[i][j] > e[i][1]+e[1][j] )   
            e[i][j] = e[i][1]+e[1][j];   
    }   
} 
```
###### 只允许经过1号顶点的情况下，最短路径更新为：
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/Floyd2.jpg)
###### (3)接下来继续求在只允许通过1和2号顶点的情况下，任意两点之间的最短距离。我们需要在只允许经过1号顶点时任意两点的最短路程的结果下，再判断如果经过2号顶点是否可以使得i号顶点到j号顶点之间的路程变得更短。即判断e[i][2]+e[2][j]是否比e[i][j]要小
```
//经过1号顶点   
for(i=1;i<=n;i++)   
    for(j=1;j<=n;j++)   
        if (e[i][j] > e[i][1]+e[1][j])  e[i][j]=e[i][1]+e[1][j];   
//经过2号顶点   
for(i=1;i<=n;i++)   
    for(j=1;j<=n;j++)   
        if (e[i][j] > e[i][2]+e[2][j])  e[i][j]=e[i][2]+e[2][j];
```
###### 只允许经过1号2号顶点的情况下，最短路径更新为：
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/Floyd3.jpg)
###### (4)以此类推，如果允许通过所有顶点，求最短路径：
```
for(k=1;k<=n;k++)   
    for(i=1;i<=n;i++)   
         for(j=1;j<=n;j++)   
             if(e[i][j]>e[i][k]+e[k][j])   
                 e[i][j]=e[i][k]+e[k][j]; 
```
##### 10.4.5.3.迪杰斯特拉算法（单源最短路径）
###### 算法的特点是以起始点为中心向外层层扩展，直到扩展到终点未知（不能存在负权边）
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/Dijkstra1.png)
###### 求图中1到2,3,4,5,6号顶点的最短距离。使用二维数组e来存储顶点之间的最短距离，初始值如下
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/Dijkstra2.png)
###### 从一号点出发，作为第一个操作点；选择离操作点index最近的节点，然后根据e[1][index]的值来更新与index直接相连并且没有被访问过的节点的e值;重复上述过程，直到操作点是终点；
```
int main() {
    int n,m,s,t;//分别是节点数、边的条数、起点、终点
    while(cin>>n>>m>>s>>t) {
        vector<vector<int>> edge(n+1,vector<int>(n+1,0));//邻接矩阵
        vector<int> dis(n+1,0);//从起点出发的最短路径，其实这个数组可以省去，dis[i]=edge[0][i]
        vector<int> book(n+1,0);//某结点已经被访问过

        for(int i=1;i<=n;i++)//初始化邻接矩阵
            for(int j=1;j<=n;j++)
                if(i!=j) edge[i][j]=INT_MAX;

        int u,v,length;
        for(int i=0;i<m;i++) {//读入每条边，完善邻接矩阵
            cin>>u>>v>>length;
            if(length<edge[u][v]) {//如果当前的边长比已有的短，则更新邻接矩阵
                edge[u][v]=length;
                edge[v][u]=length;
            }
        }
        for(int i=1;i<=n;i++)//初始化dis数组
            dis[i]=edge[s][i];
        book[s]=1;//把起点先标记一下

        //算法核心！！！先确定没访问过的节点中，离起点最近的，然后松弛
        for(int i=1;i<=n;i++) {
            int min=INT_MAX;
            int index=0;
            for(int j=1;j<=n;j++) {
                if(book[j]==0 && dis[j]<min) {
                    min=dis[j];
                    index=j;
                }
            }
            book[index]=1;//标记这个刚刚被确定了的节点
            if(index==t) break;//如果已经到终点了，就直接结束

            for(int i=1;i<=n;i++) {//从刚被确定的节点出发，松弛剩下的节点
                if(book[i]==0 && edge[index][i]<INT_MAX && dis[i] > dis[index]+edge[index][i]) 
                    dis[i] = dis[index]+edge[index][i];
            }

        }
        cout<<dis[t]<<endl;
    }
    return 0;
}
```
###### 如果我们需要将那条最短路径记录下来，就要用到一个新数组pre[]，在更新最短路径时，将当前节点的前一个节点记录下来，这样，当最终确定最短路径后，从后往前依次查看pre数组，就可以得到到达当前节点的最短路径了
##### 10.4.5.4.Bellman-Ford算法（单源最短路径，可以存在负权边）
###### Bellman-Ford算法是从DJ算法引申出来的，它可以解决带有负权边的最短路径问题。值得注意的是，DJ算法和下面的Floyd算法是基于邻接矩阵的，而Bellman-Ford算法是基于邻接表，从边的角度考量的
###### 对所有的边进行n-1次松弛操作。如果图中存在最短路径（即不存在负权回路），那么最短路径所包含的边最多为n-1条，也就是不可能包含回路。因为如果存在正回路，该路径就不是最短的，而如果存在负回路，就压根就不存在所谓的最短路径
###### 这里，和用邻接矩阵表示图的方式不同，采用了u、v、w三个矩阵存储边的信息。例如第i条边（有向边）存储在u[i]、v[i]、w[i]中，表示从顶点u[i]到v[i]这条边（u[i]->v[i]）的权值为w[i]。 之所以把dis[i]初始化为inf，可以理解成，初始时：从1号顶点通过0条边到达其他顶点的路径为正无穷
```
int main() {
    int n,m,s,t;//分别是节点数、边的条数、起点、终点
    while(cin>>n>>m>>s>>t) {
        vector<int> u(2*m+1,0);//某条边的起点
        vector<int> v(2*m+1,0);//某条边的终点
        vector<int> length(2*m+1,0);//某条边的长度
        vector<int> dis(n+1,0);//从起点出发的最短路径
        int num=0;
        for(int i=1;i<=m;i++) {//读入每条边
            num++;
            cin>>u[num]>>v[num]>>length[num];
            swap(u[num],v[num]);
        }
        for(int i=num+1;i<=2*num;i++) { //因为是无向图，所以每条边再复制一次，端点交换
            u[i]=v[i-num];
            v[i]=u[i-num];
            length[i]=length[i-num];
        }
        num=2*num;

        for(int i=1;i<=n;i++)//初始化dis数组
            dis[i]=INT_MAX;
        dis[s]=0;

        for(int k=1;k<n;k++)
            for(int j=1;j<=num;j++) {
                if(dis[u[j]]<INT_MAX && dis[v[j]] > dis[u[j]]+length[j])
                    dis[v[j]] = dis[u[j]]+length[j];
            }

        cout<<dis[t]<<endl;
    }
    return 0;
}
```
###### 对所有m条边进行（n-1）轮松弛，第1轮的效果是得到从1号顶点“只经过一条边”到达其余各顶点的最短路径长度，第2轮则是得到从1号顶点“经过最多两条边”到达其余各顶点的最短路径长度，第k轮则是得到从1号顶点“经过最多k条边”到达其余各顶点的最短路径长度。
###### 首先指出，图的任意一条最短路径既不能包含负权回路，也不会包含正权回路，因此它最多包含|v|-1条边。<br>其次，从源点s可达的所有顶点如果 存在最短路径，则这些最短路径构成一个以s为根的最短路径树。Bellman-Ford算法的迭代松弛操作，实际上就是按顶点距离s的层次，逐层生成这棵最短路径树的过程。<br>在对每条边进行1遍松弛的时候，生成了从s出发，层次至多为1的那些树枝。也就是说，找到了与s至多有1条边相联的那些顶点的最短路径；对每条边进行第2遍松弛的时候，生成了第2层次的树枝，就是说找到了经过2条边相连的那些顶点的最短路径……。因为最短路径最多只包含|v|-1条边，所以，只需要循环|v|-1 次。<br>每实施一次松弛操作，最短路径树上就会有一层顶点达到其最短距离，此后这层顶点的最短距离值就会一直保持不变，不再受后续松弛操作的影响。（但是，每次还要判断松弛，这里浪费了大量的时间，这就是Bellman-Ford算法效率底下的原因，也正是SPFA优化的所在）。
##### 10.4.5.5.SPFA算法
###### 是Bellman-Ford算法的改进，用一个队列来进行维护。初始时将源加入队列。每次从队列中取出一个元素，并对所有与他相邻的点进行松弛，若某个相邻的点松弛成功，则将其入队。直到队列为空时算法结束；这个算法，简单的说就是队列优化的bellman-ford，利用了每个点不会更新次数太多的特点发明的此算法
###### 设立一个队列q用来保存待优化的结点，优化时每次取出队首结点u，并且用u点当前的最短路径估计值对离开u点所指向的结点v进行松弛操作，如果v点的最短路径估计值有所调整，且v点不在当前的队列中，就将v点放入队尾。这样不断从队列中取出结点来进行松弛操作，直至队列空为止。 <br>松弛操作的原理是著名的定理：“三角形两边之和大于第三边”，在信息学中我们叫它三角不等式。所谓对结点i,j进行松弛，就是判定是否dis[j]>dis[i]+w[i,j]，如果该式成立则将dis[j]减小到dis[i]+w[i,j]，否则不动。 
```
int main() {
    int n,m,s,t;//分别是节点数、边的条数、起点、终点

    while(cin>>n>>m>>s>>t) {
        vector<vector<int>> edge(n+1,vector<int>(n+1,0));//邻接矩阵
        vector<int> dis(n+1,INT_MAX);//从起点出发的最短路径
        vector<int> book(n+1,0);//某结点在队列中
        queue<int> q;

        for(int i=1;i<=n;i++)//初始化邻接矩阵
            for(int j=1;j<=n;j++)
                if(i!=j) edge[i][j]=INT_MAX;

        int u,v,length;
        for(int i=0;i<m;i++) {//读入每条边，完善邻接矩阵
            cin>>u>>v>>length;
            if(length<edge[u][v]) {//如果当前的边长比已有的短，则更新邻接矩阵
                edge[u][v]=length;
                edge[v][u]=length;
            }
        }

        dis[s]=0;
        book[s]=1;//把起点先放入队列
        q.push(s);

        int top;
        while(!q.empty()) {//如果队列非空
            top=q.front();//取出队首元素
            q.pop();
            book[top]=0;//释放队首结点，因为这节点可能下次用来松弛其它节点，重新入队

            for(int i=1;i<=n;i++) {
                if(edge[top][i]!=INT_MAX && dis[i]>dis[top]+edge[top][i]) {
                    dis[i]= dis[top]+edge[top][i]; //更新最短路径
                    if(book[i]==0) { //如果扩展结点i不在队列中，入队
                        book[i]=1;
                        q.push(i);
                    }
                }
            }

        }
        cout<<dis[t]<<endl;
    }
    return 0;
```
###### SPFA在形式上和广度优先搜索非常类似，不同的是BFS中一个点出了队列就不可能重新进入队列，但是SPFA中一个点可能在出队列之后再次被放入队列，也就是一个点改进过其它的点之后，过了一段时间可能本身被改进(重新入队)，于是再次用来改进其它的点，这样反复迭代下去
#### 10.4.6.连通性（求割边，割点等）
## 11.JVM，GC
### 11.1.基本原理
#### 11.1.1.Java程序从编译到运行的过程
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMCodeBuilding.png)
#### 11.1.2.JVM的内部结构
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMStructure.png)
###### 上图是JVM的内部结构，class文件被jvm装在以后，经过jvm内存空间分配，最终是由执行引擎完成class文件的执行，当然这个过程还有其他角色模块的协助配合才能让一个java程序成功的运行，下面就详细介绍这些模板，他们也是后面学习jvm最重要的部分
###### JVM的内存空间包含：方法区、java堆、java栈、本地方法栈。<br>方法区是各个线程共享的区域，存放类信息、常量、静态变量；<br>java堆是各个线程共享的区域，我们的类的实例就放在这个区域，可以想象你的一个系统会产生很多实例，因此java堆的空间也是最大的。如果java堆空间不足，程序会报出OOM异常；<br>java栈时每个线程私有的区域，它的生命周期与线程相同，一个线程对应一个java栈，每执行一个方法就会往栈中压入一个元素，这个元素叫栈帧，而栈帧中包括了方法中的局部变量、用于存放中间状态值的操作栈，这里面有很多细节，后续展开。如果Java栈空间不足，程序会抛出StackOverflowError异常。递归容易发生栈溢出<br.本地方法栈角色和java栈类似，只不过它是用来表示执行本地方法的，本地方法栈存放的方法调用本地方法接口，最终调用本地方法库，实现与操作系统、硬件交互的目的；<br>PC寄存器用来控制程序指令的执行顺序，线程独占；<br>执行引擎当然就是根据PV寄存器调配的指令顺序，依次执行程序指令
### 11.2.内存模型、可见性、指令重排序
#### 11.2.1.内存模型
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMThreadComm1.png)
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMThreadComm2.png)
###### 上图1中，我们把java堆称为主存，而每一个线程都有自己的私有内存空间，称为工作内存
###### java线程要想另外一个线程进行通信，线程首先需要更新自己的工作内存，然后将工作内存的变化刷新到主存中。线程2从主存更新自己的工作内存，然后从自己的工作内存读取的数据就是线程1更新的数据。而主存与工作内存之间的交互，则需要Java内存模型（JMM）来管理，如上图2
###### 将线程1需要将一个更新后的变量值刷新到主存中时，需要进过两个步骤：<br>1.工作内存执行store操作；<br>2.主内存执行write操作；<br>当线程2需要从主内存中读取变量的最新值时，同样需要经过两个步骤：<br>1.主内存执行read操作，将变量值从主存中读取出来；<br>2.工作内存执行load操作，将读取出来的变量值更新到本地内存的副本；
#### 11.2.2.可见性
###### Java中有一个关键字volatile，它有什么用呢？这个答案其实就在上述java线程间通信机制中，我们想象一下下，由于工作内存这个中间层的出现，线程1和线程2必然存在延迟的问题，例如线程1在工作内存中更新了变量，但还没有刷新到主内存，而此时线程2获取到的变量值就是未更新的变量值，又或者线程1成功将变量更新到主内存，但线程2依然使用自己工作内存中的变量值，同样会出问题。不管出现哪种情况都可以导致线程间的通信不能达到预期的目的。例如以下例子：
```
boolean stop = false;
//线程1
while(!stop){
    doSommeThing();
}
//线程2
stop = false;
```
###### 这个经典的例子表示线程2通过修改stop的值，控制线程1中断，但在真实环境中可能会出现意想不到的结果，线程2在执行之后，线程1并没有立刻中断甚至一直不会中断。出现这种现象的原因就是线程2对线程1的变量更新无法第一时间获取到
###### 但这一切等到volatile出现后，再也不是问题，volatile保证两件事：<br>1.线程1工作内存中的变量更新会强制立即写入到主内存<br>2.线程2工作内存中的变量会强制立即实现，这使得线程2必须去主内存中获取最新的变量值<br>所以volatile保证了变量的可变性，因为线程1对变量的修改能第一时间让线程2可见
#### 11.2.3.指令重排序，
###### 关于指令的排序先看一段代码
```
int a = 0;
boolean flag = false;
//线程1
public void writer(){
    a = 1;
    flag = true;
}
//线程2
public void reader(){
    if(flag){
        int i = a+1;
        ......
    }
}
```
###### 线程1依次执行a=1,,flag = true;线程2判断flag=true后，设置i=a+1，根据代码语义，我们可能会推断此时i的值等于2，因为线程2在判断flag=true时，线程1已经执行了a=1；所以i的值等于a+1=2；但真实情况却并不一定如此，引起此问题的原因是线程1内部的两条语句a=1;flag=true；可能被重新排序执行，如图：
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMCodeReSort1.png)
###### 这就是指令重排序的简单演示，两个赋值语句尽管他们的代码吮吸是一前一后，但真正执行时却不一定按照代码顺序执行。代码重排序有一套严格的指令重排序规则，下图显示了一个java程序从编译到执行会经历那些重排序：
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMCodeReSort2.png)
###### 这个流程中，第一步属于编译器的重排序，编译器重排序会按JMM的规范严格执行，换言之编译器重排序一般不会对程序的正确逻辑造成印象。第二、三步属于处理器重排序，处理器重排序JMM就不好管理了，但它会要求java编译器在生成指令时加入内存屏障，内存屏障的作用就是把不能重排序的指令保护起来，那么处理器在遇到内存屏障保护的指令时就不会对它进行重排序。
###### JMM中有三种情况，任意改变一个代码的吮吸，结果都会大不相同，对于这样的逻辑代码，是不会被重排序的。注意这里是指单线程中不会被重排序，如果在多线程环境下，还是会产生逻辑问题，例如一开始举的例子
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMCodeReSort3.png)
### 11.3.配置参数
###### JVM配置参数分为三类参数：<br>1.跟踪参数<br>2.堆分配参数<br>3.栈分配参数<br>这三类三数分别用于跟踪监控JVM状态，分配堆内存以及分配栈内存
#### 11.3.1.跟踪参数
###### 跟踪参数用于跟踪监控JVM，往往被开发人员用于JVM调优以及故障排查。
##### 11.3.1.1.当发生GC时，打印GC简要信息
###### -XX:+PrintGC或-verbose:gc参数；这两个配置参数效果是一样的，都是在发生GC时打印出简要的信息，例如执行代码：
```
public static void main(String[] args){
    byte[] bytes = null;
    for(int i = 0; i < 100; i++){
        bytes = new byte[1*1024*1024];
    }
}
```
###### 这个程序连续创建了100个1M的数组对象，使用-XX:+PrintGC或-verbose:gc参数执行该程序，即可查看GC情况：
> [GC (Allocation Failure) 32686K->1648K(123904K), 0.0007230 secs]
> [GC (Allocation Failure) 34034K->1600K(123904K), 0.0009652 secs]
> [GC (Allocation Failure) 33980K->1632K(123904K), 0.0005306 secs]
###### 我们可以看到程序执行了三次GC（minor GC），这三次GC都是新生代GC，因为这个程序每次创建新的数组对象，都会把新的对象赋给bytes变量，而老的对象没有任意对象引用它，老对象会变得不可达。32686K表示回收前，对象占用空间。1648K表示回收后，对象占用空间。123904K表示还有多少空间可用。0.0007230secs表示这次垃圾回收花的时间。
##### 11.3.1.2.打印GC的详细信息以及堆使用详细信息
###### -XX:+PrintGCDetail参数：
###### 还是使用上面的代码，获得如下结果：
> [GC (Allocation Failure) [PSYoungGen: 32686K->1656K(37888K)] 32686K->1664K(123904K), 0.0342788 secs] [Times: user=0.00 sys=0.00, real=0.03 secs]
> [GC (Allocation Failure) [PSYoungGen: 34042K->1624K(70656K)] 34050K->1632K(156672K), 0.0013466 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
> Heap
> PSYoungGen total 70656K, used 43118K [0x00000000d6100000, 0x00000000dab00000, 0x0000000100000000)
> eden space 65536K, 63% used [0x00000000d6100000,0x00000000d8985ac8,0x00000000da100000)
> from space 5120K, 31% used [0x00000000da600000,0x00000000da796020,0x00000000dab00000)
> to space 5120K, 0% used [0x00000000da100000,0x00000000da100000,0x00000000da600000)
> ParOldGen total 86016K, used 8K [0x0000000082200000, 0x0000000087600000, 0x00000000d6100000)
> object space 86016K, 0% used [0x0000000082200000,0x0000000082202000,0x0000000087600000)
> Metaspace used 2669K, capacity 4486K, committed 4864K, reserved 1056768K
> class space used 288K, capacity 386K, committed 512K, reserved 1048576K
###### 我们看到除了打印GC信息之外，还显示是了堆使用情况，堆分为新生代、老年代、元空间。注意这里没有永久区了，永久区在java8已经移除，原来放在永久区的常量、字符串静态变量都移到了元空间，并使用本地内存。<br>新生代当中又分为伊甸区（eden）和幸存区（from和to），从上面打印的内容可以看到新生代总大小为70656K，使用了43118K，细心的同学的可能会发现eden+from+to=65536K+5120K+5120K=75776 并不等于总大小70656K，这是为什么呢？这是因为新生代的垃圾回收算法是采用复制算法，简单的说就是在from和to之间来回复制（复制过程中再把不可达的对象回收掉），所以必须保证其中一个区是空的，这样才能有预留空间存放复制过来的数据，所以新生代的总大小其实等于eden+from（或to）=65536K+5120K=70656k。
##### 11.3.1.3.使用外部文件记录GC的日志
###### -Xloggc:log/gc.log
###### 这是一个非常有用的参数，它可以把GC的日志记录到外部文件中，这在生产环境进行故障排查时尤为重要，当java程序出现OOM时，总希望看到当时垃圾回收的情况，通过这个参数就可以把GC的日志记录下来，便于排查问题，当然也可以做日常JVM监控
##### 11.3.1.4.监控类的加载
###### -XX:+TraceClassLoading；使用这个参数可以监控java程序加载的类
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMArgs1.png)
#### 11.3.2.堆分配参数
###### 可以用以下代码打印出目前内存使用的情况：
```
public static void main(String[] args){
  System.out.println("最大堆：" + Runtime.getRuntime().maxMemory()/1024/1024 + "M");
  System.out.println("空闲堆：" + Runtime.getRuntime().freeMemory()/1024/1024 + "M");
  System.out.println("总的堆：" + Runtime.getRuntime().totalMemory()/1024/1024 + "M");
}
```
###### -Xmx参数：指定堆的最大大小，表示java程序最大能够使用的内存大小，如果超过这个大小，那么java汇报oom错误，空闲堆表示程序已经分配的内存大小减去已经使用的内存大小，而总的堆大小表示目前程序已经配置到多少内存大小，一般而言程序一启动，会按照-Xmx5m先分配5M的空间，这时总的堆大小就是5M
###### -Xms参数：指定堆的初始大小
###### -Xmn参数：设置年轻代的大小。例如：-Xmx20m -Xms5m -Xmn2m -XX:+PrintGCDetails，得到如下结果：
> 最大堆：19.5M
> 空闲堆：4.720428466796875M
> 总的堆：5.5M 
> Heap
> PSYoungGen total 1536K, used 819K [0x00000000ffe00000, 0x0000000100000000, 0x0000000100000000)
> eden space 1024K, 79% used [0x00000000ffe00000,0x00000000ffeccc80,0x00000000fff00000)
> from space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
> to space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
> ParOldGen total 4096K, used 0K [0x00000000fec00000, 0x00000000ff000000, 0x00000000ffe00000)
> object space 4096K, 0% used [0x00000000fec00000,0x00000000fec00000,0x00000000ff000000)
> Metaspace used 2723K, capacity 4486K, committed 4864K, reserved 1056768K
> class space used 293K, capacity 386K, committed 512K, reserved 1048576K
###### 可以看到新生代总大小：eden+from+to=1024K+512K+512K=2M，和-Xmn相对应
###### -NewRatio：设置新生代（eden+from+to）和老年代（不包含永久区）的比值，比如NewRatio=4，则新生代：老年代=1:4
###### -SurvivorRatio：设置Eden和Survivor区的大小比值，比如SurvivorRatio=8，则from:to:Eden=1:1:8
###### -XX:+HeapDumpOnOutOfMemoryError、-XX:+HeapDumpPath这两个参数可以实现在发生OOM异常时把堆栈信息打印到外部文件
#### 11.3.3.永久区分配参数
###### -XX:PermSize参数：设置永久区的初始空间大小
###### -XX:MaxPermSize参数：设置永久区的最大空间大小
###### 他们表示一个系统可以容纳多少个类型，一般空间比较小。在java1.8以后，永久区被移到元数据区，使用本地内存，所以这两个参数也不建议再使用
#### 11.3.4.栈分配参数
###### -Xss参数：设置每个线程的堆栈大小，通常只有几百K，决定了函数调用的深度，每个线程都有自己独立的栈空间。如果函数调用太深，超过了栈的大小，则会抛出java.lang.StackOverflowError，通常我们遇到这种错误，不是去调整-Xss参数，而是应该去调查函数调用太深的原因
### 11.4.垃圾回收算法
#### 11.4.1.stop the world
###### stop the world（STW）会在执行某一个垃圾回收算法的时候产生，JVM为了执行垃圾回收，会暂停java应用程序的执行，等垃圾回收完成后，再继续运行。如果你使用JMeter测试过java程序，你可能会发现在测试过程中，java程序有不规则的停顿现象，其实这就是stop the world，停顿的时候JVM是在做垃圾回收。所以尽可能减少STW的时间就是我们优化JVM的主要目标
#### 11.4.2.引用计数法
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMGCReferenceCount1.png)
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMGCReferenceCount2.png)
###### 引用计数法顾名思义，就是对一个对象被引用的次数进行计数，当增加一个引用，引用计数就加一，减少一个引用，引用计数就减一
###### 引用计数算法原理非常简单，是最原始的回收算法， 但是java中没有使用这种算法，原因有两个：一是频繁的计数影响性能，二是无法处理循环引用的问题（例如Teacher对象中引用了Student对象，Student对象中又引用了Teacher对象，这种情况下，对象将永远无法被回收）
#### 11.4.3.标记清除法
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMGCMarkClean.png)
###### 标记清楚算法是很多垃圾回收算法的基础，简单来说有两个步骤：标记（遍历所有的GC Root，并将从GC Root可达的对象设置为存活对象），清除（遍历堆中所有对象，将北邮被标记的对象清除）
###### 注意上图中灰色的对象，因为从GC Root遍历不到它们（尽管它们本身有引用关系，但从GC Root无法遍历到它们），因此它们没有被标记为存活对象，在清除过程中将会被回收。
###### 需要注意的是，为了让java程序暂停等待以保存在标记清楚过程中， 不会有新的对象产生，所以这个算法还是会产生STW
###### 1.设计大量的内存遍历工作，执行性能较低，STW时间较长，java程序吞吐量降低<br>2.对象呗清楚之后，被清除的对象留下内存的空缺位置，造成内存不连续，空间浪费 
#### 11.4.4.标记压缩
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMGCMarkCompress.png)
###### 标记压缩算法就是在标记清除算法的基础上，增加压缩过程，在进行完标记清除之后，对内存空间进行压缩，节省内存空间，解决了标记清除算法内存不连续的问题
###### 标记压缩算法也会产生STW，不能喝java程序并发执行，在压缩过程中一些对象内存地址会发生改变，java程序只能等待压缩完成后才能继续
#### 11.4.5.复制算法
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMGCCopy1.png)
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMGCCopy2.png)
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMGCCopy3.png)
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMGCCopy4.png)
###### 复制算法简单来说就是把内存一分为二，但只使用其中一份，在垃圾回收时，将正在使用的那份内存中存活的对象复制到另一份中，最后将正在使用的内存空间的对象清除，完成垃圾回收
###### 复制算法相对于标记压缩算法来说更简洁高效，但它的缺点就是不适合用于存活对象多的情况，因为那样需要复制的对象很多，复制性能较差，所以复制算法往往用于内存空间中新生代的垃圾回收。另一个缺点是内存空间占用成本高，因为它基于两份内存空间做对象复制，在非垃圾回收的周期内，只用到了一份内存空间
### 11.5.垃圾回收器
###### JVM中并不是单纯的使用特定的垃圾回收算法，而是使用一种叫垃圾回收器的东西，垃圾回收器可以看成是一系列算法的不同组合，在不同的场景使用合适的垃圾回收器，才能起到事半功倍的效果
#### 11.5.1.堆内存回顾
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMHeapStructure.png)
###### java堆内存结构包括：新生代和老年代，其中新生代由一个伊甸区和两个幸存区组成，两个幸存区的大小相同，完全对称，没有任何差别。我们把它们成为S0区和S1区，也可以称为from区和to区<br>JVM的垃圾回收主要针对以上堆空间的垃圾回收，当然其实也会针对元数据区（永久区）进行垃圾回收，在此我们主要介绍对堆空间的垃圾回收
#### 11.5.2.串行收集器
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMGCSerialColletion.png)
###### 串行收集器就是使用单线程进行垃圾回收，对新生代的回收使用复制算法，对老年代使用标记压缩算法。串行垃圾收集器是最古老最稳定的收集器，尽管他是串行回收，回收时间较长，但其稳定性是由于其他及回收器的，综合来说是个不错的选择。要使用串行回收器，可以在启动配置时加上一下参数：-XX:+UserSerialGC
#### 11.5.3.并行回收器
##### 11.5.3.1.ParNew回收器
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMGCParNewColletion.png)
###### 这个回收器只针对新生代进行并发回收，在老年代依然使用串行回收。回收算法依然和串行回收一样，同样会产生STW。在多核条件下，他的性能显然由于串行回收器，如果需要使用这种回收器，可以在启动参数中配置：-XX:+UserParNewGC，如果要进一步指定并发的线程数，可以配置一下参数：-XX:ParallelGCThreads
##### 11.5.3.2.Parallel回收器
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMGCParallelColletion.png)
###### 依然是并行回收器，但这种回收器有两种配置：<br>一种类似于ParNew：在新生代使用并行回收，在老年代使用串行回收。它与ParNew的不同在于它在设计目标上更重视吞吐量，可以认为在相同条件下它比ParNew更优。要使用这种回收器，可以在启动程序时配置：-XX:UserParallelGC<br>>Parallel回收器的另外一种配置则不同于ParNew，对于新生代和老年代均使用并行回收，要使用这种回收器可以在启动程序中配置：-XX:UserParallelOldGC
#### 11.5.4.CMS回收器
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMGCCMS.png)
###### CMS回收器：Concurrent Mark Sweep，并发标记清楚。并发表示它可以与应用程序并发执行，交替执行；标记清楚表示这种回收器不是使用标记压缩算法，这和前面介绍的串行回收器和并行回收器有所不同。需要注意的是CMS回收器是一种针对老年代的回收器，不对新生代产生作用。这种回收器的优点在于减少了应用程序停顿的时间，因为它不需要应用程序暂停等待垃圾回收，要使用这种垃圾回收器可以在启动参数中配置：-XX:UserConcMarkSweepGC
##### 11.5.4.1.CMS回收器的运行机制：
###### 初始标记（STW）：进行可达性分析，标记GC Root能够直接关联的对象
###### 并发标记：是主要的标记过程，进行GC Root Tracing，在第一阶段被暂停的线程重新开始运行，有第一阶段标记过得对象出发，所有可达的对象都在本阶段中标记
###### 并发预清理：做的工作还是标记，与下一步重新标记功能类似。因为重新标记需要STW，因此冲标记的工作尽可能多的在并发阶段完成来减少STW的时间。此阶段标记从新生代晋升的对象、新分配到老年代的对象以及在并发阶段被修改了的对象 
###### 重新标记（STW）：暂停所有用户线程，重新扫描堆中的对象，进行可达性分析，标记活着的对象，由于前面的基础，此阶段的工作量被大大减轻，停顿时间因此也会减少。注意这个阶段是多线程的
###### 并发清理（和用户线程一起）：用户线程被重新激活，同时清理那些无效的对象基于标记结果，直接清理对象
###### 重置：CMS清除内部状态，为下次回收做准备
##### 11.5.4.2.并发预清理阶段介绍：JVMGCCMSbfyql1
###### 此阶段比较复杂，从初学者容易忽略或者说不理解的地方抛出一个问题：如何确定老年代的对象是活着的：答案很简单，通过GC Root Tracing可到达的对象就是活着的：
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMGCCMSbfyql1.png)
###### 如果老年代进行GC是，如何确保上图中Current Obj标记为活着的，答案是必须扫描新生代来确保，这也是为什么CMS虽然是老年代的gc，但仍要扫描新生代的原因。那么问题来了，全量扫描新生代和老年代很慢，而CMS号称是停顿时间最短的GC，解决方法就是：
###### 新生代机制：新生代垃圾回收完剩下的对象全是存活的，并且存活的对象很少。如果在扫描新生代之前进行一次Minor GC，情况是不是就变得好很多。CMS有两个参数：CMSScheduleRemarkEdenSizeThreshold、CMSScheduleRemarkEdenPenetration，默认分别为2M、50%，意思就是预清理后，如果eden空间使用超过2M是，启动可中断的并发预清理，直到eden空闲使用率达到50%时中断，进入remark阶段。如果能在可终止的预清理阶段发生一次Minor GC，那就万事大吉了，但可终止的预清理要执行多长时间来保证发生一次Minor GC，这是没法保证的，Gc由JVM自动调度的，CMS的参数CMSMaxAbortablePrecleanTime，默认为5秒，表示只要到了5秒，不管有没有发生Minor GC，都会终止次阶段，进入remark。CMS另一个参数CMSScavengeBeforeRemark参数能够使remark前强制进行一次Minor GC，这样做利弊都有，利是减少了remark阶段的停顿，弊是Minor GC后紧跟一个remark pause，如此一来，停顿时间也很久。为了尽可能减少remark阶段的STW时间，预清理阶段会尽可能多做一些事情，比如remark的rescan阶段是多线程的，为了便于多线程扫描，预清理阶段会将新生代块，每个块中存放着多个对象，这样remark阶段接不需要从头开始识别每个对象的起始位置，遗憾的是，这种办法仍然是建立在发生了Minor GC的条件下的
###### 老年代机制：老年代机制与一个叫CARD TABLE密不可分。CMS将老年代的空间分成大小为512bytes的块，card table中的每个元素对应着一个块，并发标记时，如果某个对象的引用发生了变化，就标记该对象所在的块为dirty card，并发预处理阶段就会重新扫描该块，将该对象引用的对象标识为可达。
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMGCCMSbfyql2.png)
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMGCCMSbfyql3.png)
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMGCCMSbfyql4.png)
###### card table还有其他作用：进行Minor GC时，如果有老年代引用新生代，怎么识别，可以将有老年代引用新生代对应的card标记为相应的值
##### 11.5.4.3.优缺点：
###### 优点：显而易见，减少了应用程序的停顿时间，让回收线程和用户线程可以并发执行
###### 缺点：<br>1.多线程回收会抢占CPU资源，这可能会造成用户线程执行效率下降<br>2.并发清理阶段，用户线程还在运行，这段时间就可能产生新的垃圾，这些称为浮动垃圾，只能等到下次清理才能被清理<br>由于垃圾回收阶段用户线程仍在执行，必须预留出内存空间给用户线程使用。因此不能像其他回收器那样，等待老年代满了再进行GC。CMS提供了CMSInitiatingOccupancyFraction参数来设置老年代空间使用百分比，达到百分比就进行垃圾回收。这个参数设置太小，会导致频繁GC；设置太大，会导致用户线程使用空间不足，发生Concurrent Mode Failure错误，这是虚拟机就会启动备案，使用Serial old收集器重新对老年代进行垃圾回收，如此一来，停顿时间变得更长。CMS其实还有一个动态检查机制，会根据历史记录，预测老年代还需要多久填满以及进行一次回收所需要的时间。这个特性可以使用参数UseCMSInitiatingOccupancyOnly来关闭<br>3.使用标记清理算法会造成大量的空间碎片，给大对象分配带来麻烦。CMS的解决方案是使用UseCMSCompactAtFullCollection参数（默认开启），在顶不住要进行Full GC是开启内存碎片整理，这个过程需要STW，会导致停顿时间变长。虚拟机还提供了另外一个参数CMSFullFCsBeforeCompaction，用于设置执行多少次不压缩的Full GC后，跟着来一次带压缩1T
#### 11.5.5.G1回收器
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMGCG11.jpg)
###### G1回收器是jdk1.7以后推出的回收器，以关注延迟为目标、服务端用用的垃圾回收器，试图取代CMS回收器。虽然G1也有类似CMS的手机动作：初始标记、并发标记、重新标记、清除、转移回收，并且也以一个串行收集器做担保机制。
###### G1回收器的设计理念：<br>1.设计原则是首先收集尽可能多的垃圾。因此G1并不会等内存耗尽（串行回收器、并行回收器）或者快耗尽（CMS回收器）的时候开始垃圾手机，而是在内部采用启发式算法，在老年代找出具有高收集收益的分区进行收集。同时G1可以根据用户设置的暂停时间目标自动调整年轻代和总堆的大小，暂停目标越短年轻代空间越小，总空间就越大<br>2.G1采用内存分区的思想，将内存划分为一个个相等大小的内存分区，回收时则以分区为单位进行回收，存活的对象复制到另一个空闲分区中。由于都是以相等大小的分区为单位进行操作，因此G1天然就是一种压缩方案<br>3.G1虽然也是分代收集器，但整个内存分区不存在物理上的年轻代和老年代的却别，也不需要完全独立的survivor堆做复制转呗。G1只有逻辑上的分代概念，或者说每个分区都可能随G1的运行在不同代之间前后切换<br>4.G1收集都是STW的，但年轻代和老年代的收集界限比较模糊，采用了混合收集的方式。即每次收集既可能只收集年轻代分区，也可能在收集年轻代的同时，包含部分老年代分区，这样即使堆内存很大时，也可以限制收集范围，从而降低停顿
##### 11.5.5.1.G1内存模型
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMGCG12.jpg)
###### 分区Region：G1采用了分区的思路，将整个堆空间分成若干个大小相等的内存区域，每次分配对象空间将逐段地使用内存。因此，在堆的使用上，G1并不不要求对象的存储一定是物理上连续的，只要逻辑上连续即可；每个分区也不会确定地为某个代服务，可以按需在年轻代和老年代之间雀环。启动时可以通过参数-XX:G1HeapRegionSize=n可指定分区大小（1MB-32Mb，且必须是2的幂），默认将整堆划分成2048个分区
###### 卡片Card：在每个分区内部被分成了若干个大小为512Byte卡片，标识堆内存最小可用粒度所有分区的卡片将会记录在全局卡片表中，分配的对象会占用物理上连续的若干个卡片，当查找对分区内对象的引用时便可通过记录卡片来查找该引用对象。每次对内存的回收，都是对指定分区的卡片进行处理
###### 堆Heap：G1同样可以通过-Xms/-Xmx来指定堆空间的大小。当发生年轻代收集或混合收集时，通过计算GC与应用的耗费时间比，自动调整堆空间大小。如果GC频率过高，则通过增加堆尺寸，来减少GC频率，相应地GC占用的时间也随之降低；目标参数-XX:GCTimeRatio即为GC与应用的耗费时间比，G1默认为9，而CMS默认为99，因为CMS的设计原则是耗费在GC上的时间尽可能的少。另外，当空间不足，如对象空间分配或转移失败时，G1会首先尝试增加堆空间，如果扩容失败，则发起担保的Full GC。Full GC后，堆尺寸计算结果也会调整堆空间。
###### 分代模型
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMGCG13.jpg)
###### 分代Generation：分代垃圾手机可以将关注点集中在最近被分配的对象上，而无需整堆扫描，避免长命名对象的拷贝，同时独立手机有助于降低响应时间。虽然分区使得内存分配不再要求紧凑的内存空间，但G1依然使用分代的思想。与其他垃圾收集器类似，G1将内存在逻辑上划分为年轻代和老年代，其中年轻代又划分为Eden空间和Survivor空间。但年轻代空间并不是固定不变的，当现有年轻代分区占满时，JVM会分配新的空间分区加入到年轻代空间。<br>整个年轻代内存会在初始空间-XX:G1NewSizePercent（默认整堆5%）与最大空间-XX:G1MaxNewSizePercent（默认60%）之间动态变化，且由参数目标暂停时间-XX:MaxGCPauseMillis（默认200ms）、需要扩缩容的大小以及分区的已记忆集合（RSet）计算得到。当然，G1依然可以设置固定的年轻代大小（参数-XX:NewRatio、-Xmn），但同时暂停目标将失去意义。
###### 本地分配缓冲：。。。。。。。。。。。。。。不总结了（https://blog.csdn.net/coderlius/article/details/79272773）
### 11.6.类加载器原理
###### 我们知道我们编写的java代码，会经过编译器编译成字节码文件（class文件），再把字节码文件装在到JVM中，映射到各个内存区域中，我们的程序就可以在内存中运行了。
#### 11.6.1.类装载流程：
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMLjzq.png)
###### 1.加载：加载是类状态的第一步，首先通过class文件的路径读取到二进制流，并解析二进制流将里面的元数据（类型、常量等）载入到方法区，在java堆中生成对应的java.lang.Class对象
###### 2.连接：连接过程又分为三步，验证、准备、解析<br>2.1.验证：验证的主要目的就是判断class文件的合法性，比如class文件一定是以0xCAFEBABE开头的，另外对版本号也会做验证，例如如果使用Java1.8编译后的class文件要在java1.6的虚拟机上运行，因为版本的问题就会验证不通过。除此之外还会对元数据、字节码进行验证，具体的验证过程就复杂的多了，可以专门查看相关的资料去来了解<br>2.2.准备过程就是分配内存，给来的一些字段设置初始值，例如public static int v=1;这段代码在准备阶段v的值就会被初始化为0，只有到后面类初始化阶段时才会被设置为1.但是对于static final常量，在准备阶段就会被设置成指定的值<br>2.3.解析过程就是将符号引用替换为直接引用，例如某个类继承java.lang.Object，原来的符号引用记录的是java.lang.Object这个符号，凭借这个符号并不能找到java.lang.Object这个对象在哪里，而直接引用就是要找到java.lang.Object所在的内存地址，建立直接引用关系，这样就方便查询到具体对象
###### 3.初始化：初始化过程，主要包括执行类构造方法，static变量赋值语句，static{}语句块，需要注意的是如果一个子类进行初始化，那么它会实现初始化其父类，保证父类在子类之前被初始化。所以其实在java中初始化一个类，那么必然先初始化java.lang.Object，因为所有的java类都继承自java.lang.Object
#### 11.6.2.类加载器
###### 类加载器是类加载过程中的主角。类加载器ClassLoader它是一个抽象类，ClassLoader的具体实例负责把java字节码读取到JVM中，ClassLoader还可以定制以满足不同字节码流的加载方式，比如从网络加载、从文件加载。ClassLoader的负责整个类装载流程中“加载”阶段。
##### 11.6.2.1.系统中的ClassLoader
###### BootStrap ClassLoader（启动ClassLoader）、Extension ClassLoader（扩展ClassLoader）、App ClassLoader（应用ClassLoader）、Custom ClassLoader（自定义ClassLoader）。每个ClassLoader都有另外一个ClassLoader作为父ClassLoader（BootStrap ClassLoader除外）
##### 11.6.2.2.ClassLoader加载机制（双亲模式）
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/JVMClassLoader.jpg)
###### 自下向上检查类是否被加载，一般情况下，首先从App ClassLoader中调用findLoadedClass方法查看是否已经加载，如果没有加载，则会交给父类，Extension ClassLoader去查看是否加载，还没加载，则再调用其父类，BootStrap ClassLoader查看是否已经加载，如果仍然没有，自定向下尝试加载类，从BootStrap ClassLoader到App ClassLoader一次尝试加载。<br>值得注意的是，即使两个类来源于相同的class文件，如果使用不同的类加载器加载，加载后的对象是完全不同的，这个不同反映在对象的equals()、isAssignableFrom()、isInstance()等方法的返回结果，也包括了使用instanceof关键字对对象所属关系的判定结果
##### 11.6.2.3.双亲模式问题
###### 我们把上一节中，自下向上的检查类是否被加载的过程称为双亲模式。双亲模式存在一个问题，就是父ClassLoader无法加载底层ClassLoader的类<br>比如：javax.xml.parsers包中定义了xml解析的类接口，而Service Provider Interface（SPI）位于rt.jar中，换句话说，接口在BootStrap ClassLoader中，而SPI的实现类在App ClassLoader中，这样BootStrap ClassLoader就无法接在SPI的实现类。<br>解决办法：JDK中提供了一个方法：Thread.setContextClassLoader()，泳衣解决顶层ClassLoader无法访问底层ClassLoader的类的问题，基本思想是，在顶层ClassLoader中，传入底层ClassLoader的实例
##### 11.6.2.4.双亲模式的破坏
###### 双亲模式是默认的模式，但不是必须这么做；比如Tomcat的WebappClassLoader就会先加载自己的Class，找不到再委托parent；OSGI的ClassLoader形成网状结构，根据需要自由加载Class
## 12.排序
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/Sort.png)
### 12.1.冒泡排序
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/BubbleSort.gif)
```
function bubbleSort(arr) {
    var len = arr.length;
    for (var i = 0; i < len - 1; i++) {
        for (var j = 0; j < len - 1 - i; j++) {
            if (arr[j] > arr[j+1]) {        // 相邻元素两两对比
                var temp = arr[j+1];        // 元素交换
                arr[j+1] = arr[j];
                arr[j] = temp;
            }
        }
    }
    return arr;
}
```
### 12.2.选择排序
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/SelectionSort.gif)
```
function selectionSort(arr) {
    var len = arr.length;
    var minIndex, temp;
    for (var i = 0; i < len - 1; i++) {
        minIndex = i;
        for (var j = i + 1; j < len; j++) {
            if (arr[j] < arr[minIndex]) {     // 寻找最小的数
                minIndex = j;                 // 将最小数的索引保存
            }
        }
        temp = arr[i];
        arr[i] = arr[minIndex];
        arr[minIndex] = temp;
    }
    return arr;
} 
```
### 12.3.插入排序
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/InsertionSort.gif)
```
function insertionSort(arr) {
    var len = arr.length;
    var preIndex, current;
    for (var i = 1; i < len; i++) {
        preIndex = i - 1;
        current = arr[i];
        while (preIndex >= 0 && arr[preIndex] > current) {
            arr[preIndex + 1] = arr[preIndex];
            preIndex--;
        }
        arr[preIndex + 1] = current;
    }
    return arr;
}
```
###### 插入排序插入时，如果比较操作的代价比交换操作大的话，可以采用二分查找法来减少比较操作的次数，我们成为二分插入排序
### 12.4.希尔排序
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/ShellSort.gif)
```
function shellSort(arr) {
    var len = arr.length,
        temp,
        gap = 1;
    while (gap < len / 3) {          // 动态定义间隔序列
        gap = gap * 3 + 1;
    }
    for (gap; gap > 0; gap = Math.floor(gap / 3)) {
        for (var i = gap; i < len; i++) {
            temp = arr[i];
            for (var j = i-gap; j > 0 && arr[j]> temp; j-=gap) {
                arr[j + gap] = arr[j];
            }
            arr[j + gap] = temp;
        }
    }
    return arr;
}
```
### 12.4.归并排序
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/MergeSort.gif)
```
function mergeSort(arr) {  // 采用自上而下的递归方法
    var len = arr.length;
    if (len < 2) {
        return arr;
    }
    var middle = Math.floor(len / 2),
        left = arr.slice(0, middle),
        right = arr.slice(middle);
    return merge(mergeSort(left), mergeSort(right));
}
 
function merge(left, right) {
    var result = [];
 
    while (left.length>0 && right.length>0) {
        if (left[0] <= right[0]) {
            result.push(left.shift());
        } else {
            result.push(right.shift());
        }
    }
 
    while (left.length)
        result.push(left.shift());
 
    while (right.length)
        result.push(right.shift());
 
    return result;
}
```
### 12.5.堆排序
###### 参见10.2
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/HeapSort.gif)
```
var len;    // 因为声明的多个函数都需要数据长度，所以把len设置成为全局变量
 
function buildMaxHeap(arr) {   // 建立大顶堆
    len = arr.length;
    for (var i = Math.floor(len/2); i >= 0; i--) {
        heapify(arr, i);
    }
}
 
function heapify(arr, i) {     // 堆调整
    var left = 2 * i + 1,
        right = 2 * i + 2,
        largest = i;
 
    if (left < len && arr[left] > arr[largest]) {
        largest = left;
    }
 
    if (right < len && arr[right] > arr[largest]) {
        largest = right;
    }
 
    if (largest != i) {
        swap(arr, i, largest);
        heapify(arr, largest);
    }
}
 
function swap(arr, i, j) {
    var temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}
 
function heapSort(arr) {
    buildMaxHeap(arr);
 
    for (var i = arr.length - 1; i > 0; i--) {
        swap(arr, 0, i);
        len--;
        heapify(arr, 0);
    }
    return arr;
}
```
### 12.6.快速排序
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/QuickSort.gif)
###### 快速排序的基本思想：通过一趟排序将待排记录分隔成独立的两部分，其中一部分记录的关键字均比另一部分的关键字小，则可分别对这两部分记录继续进行排序，以达到整个序列有序。
###### 算法描述：快速排序使用分治法把一个串（list）分成两个子串（sub-lists）。具体算法描述如下：<br>1.从数列中挑出一个元素，称为“基准”(pivot)<br>2.重新排序数列，所有元素比基准值小的摆放在基准前面，大的摆放在后面，在这个分区退出之后，该基准就处于数列的中间位置。这个称为分区操作<br>3.递归的，把小于基准值元素的子数列和大于基准值元素的子数列排序
```
function quickSort(arr, left, right) {
    var len = arr.length,
        partitionIndex,
        left = typeof left != 'number' ? 0 : left,
        right = typeof right != 'number' ? len - 1 : right;
 
    if (left < right) {
        partitionIndex = partition(arr, left, right);
        quickSort(arr, left, partitionIndex-1);
        quickSort(arr, partitionIndex+1, right);
    }
    return arr;
}
 
function partition(arr, left ,right) {     // 分区操作
    var pivot = left,                      // 设定基准值（pivot）
        index = pivot + 1;
    for (var i = index; i <= right; i++) {
        if (arr[i] < arr[pivot]) {
            swap(arr, i, index);
            index++;
        }       
    }
    swap(arr, pivot, index - 1);
    return index-1;
}
 
function swap(arr, i, j) {
    var temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}
```
### 12.7.计数排序
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/CountingSort.gif)
```
function countingSort(arr, maxValue) {
    var bucket = new Array(maxValue + 1),
        sortedIndex = 0;
        arrLen = arr.length,
        bucketLen = maxValue + 1;
 
    for (var i = 0; i < arrLen; i++) {
        if (!bucket[arr[i]]) {
            bucket[arr[i]] = 0;
        }
        bucket[arr[i]]++;
    }
 
    for (var j = 0; j < bucketLen; j++) {
        while(bucket[j] > 0) {
            arr[sortedIndex++] = j;
            bucket[j]--;
        }
    }
 
    return arr;
}
```
### 12.8.桶排序
##### 桶排序是计数排序的升级版。它利用了函数的映射关系，高效与否的关键就在于这个映射函数的确定。桶排序 (Bucket sort)的工作的原理：假设输入数据服从均匀分布，将数据分到有限数量的桶里，每个桶再分别排序（有可能再使用别的排序算法或是以递归方式继续使用桶排序进行排）
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/BucketSort.png)
```
function bucketSort(arr, bucketSize) {
    if (arr.length === 0) {
      return arr;
    }
 
    var i;
    var minValue = arr[0];
    var maxValue = arr[0];
    for (i = 1; i < arr.length; i++) {
      if (arr[i] < minValue) {
          minValue = arr[i];                // 输入数据的最小值
      } else if (arr[i] > maxValue) {
          maxValue = arr[i];                // 输入数据的最大值
      }
    }
 
    // 桶的初始化
    var DEFAULT_BUCKET_SIZE = 5;            // 设置桶的默认数量为5
    bucketSize = bucketSize || DEFAULT_BUCKET_SIZE;
    var bucketCount = Math.floor((maxValue - minValue) / bucketSize) + 1;  
    var buckets = new Array(bucketCount);
    for (i = 0; i < buckets.length; i++) {
        buckets[i] = [];
    }
 
    // 利用映射函数将数据分配到各个桶中
    for (i = 0; i < arr.length; i++) {
        buckets[Math.floor((arr[i] - minValue) / bucketSize)].push(arr[i]);
    }
 
    arr.length = 0;
    for (i = 0; i < buckets.length; i++) {
        insertionSort(buckets[i]);                      // 对每个桶进行排序，这里使用了插入排序
        for (var j = 0; j < buckets[i].length; j++) {
            arr.push(buckets[i][j]);                     
        }
    }
 
    return arr;
}
```
### 12.7.最低位优先基数排序
![avatar](https://raw.githubusercontent.com/AssassinGQ/MarkDownHub/master/RadixSort.gif)
```
// LSD Radix Sort
var counter = [];
function radixSort(arr, maxDigit) {
    var mod = 10;
    var dev = 1;
    for (var i = 0; i < maxDigit; i++, dev *= 10, mod *= 10) {
        for(var j = 0; j < arr.length; j++) {
            var bucket = parseInt((arr[j] % mod) / dev);
            if(counter[bucket]==null) {
                counter[bucket] = [];
            }
            counter[bucket].push(arr[j]);
        }
        var pos = 0;
        for(var j = 0; j < counter.length; j++) {
            var value = null;
            if(counter[j]!=null) {
                while ((value = counter[j].shift()) != null) {
                      arr[pos++] = value;
                }
          }
        }
    }
    return arr;
}
```
## 13.操作系统生产者消费者
### 13.1.背景
###### 生产者生产数据到缓冲区，消费者从缓冲区中取数据；如果缓冲区满了，则生产者线程阻塞；如果缓冲区空了，则消费者线程阻塞
### 13.2.实现方式：
#### 13.2.1.实现方式1：synchronized、wait和notify
###### 参考2.1.6节
#### 13.2.2.实现方式2：lock和condition的await、signalAll、signal
###### ReentrantLock类会维护一个队列，称为Release队列
###### Condition类会维护一个队列，称为Condition队列，通过await操作的线程会进入这个队列中
###### todo
```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class LockCondition{
    public static void main(String[] args){
        Lock lock = new ReentrantLock();
        Condition producerCondition = lock.newCondition();
        Condition consumerCondition = lock.newCondition();
        Rescource rescource = new Rescource(lock, producerCondition, consumerCondition);
        
        ProducerThread producer = new ProducerThread(rescource);
        ConsumerThread consumer1 = new ConsumerThread(rescource);
        ConsumerThread consumer2 = new ConsumerThread(rescource);
        ConsumerThread consumer3 = new ConsumerThread(rescource);
        
        producer.start();
        consumer1.start();
        consumer2.start();
        consumer3.start();
    }
    
    //消费者线程
    class ConsumerThread extends Thread{
        private Resource resource;
        public ConsumerThread(Resource resource){
            this.resource = resource;
        }
        public void run(){
            while(true){
                try{
                    Thread.sleep((long)(1000*Math.random()));
                }catch(InterruptedException e){
                    e.printStackTrace();
                }
                resource.remove();
            }
        }
    }
    
    //生产者线程
    class ProducerThread extends Thread{
        private Resource resource;
        public ProducerThread(Resource resource){
            this.resource = resource;
        }
        public void run(){
             while(true){
                try{
                    Thread.sleep((long)(1000*Math.random()));
                }catch(InterruptedException e){
                    e.printStackTrace();
                }
                resource.add(); 
             }
        }
                                
    }
    
    class Rescource{
        private int num = 0;//当前资源数量
        private int size = 10;//资源池中允许存放的资源数目
        private Lock lock;
        private Condition producerCondition;
        private Condition consumerCondition;
        public Resource(Lock lock, Condition producerCondition, Condition consumerCondition){
            this.lock = lock;
            this.producerCondition = producerCondition;
            this.consumerCondition = consumerCondition;
        }
        
        //向资源池中添加资源
        public void add(){
            lock.lock();//请求锁
            try{
                if(num < size){
                    num++;
                    System.out.println(Thread.currentThread().getName() + "生产一件资源，当前资源池有" + num + "个");
                    //唤醒等待的消费者
                    consumerCondition.signalAll();//通知持有该锁的除当前线程外的所有await的线程，唤醒
                    //consumerCondition.signal();//随机通知持有该锁的除当前线程外的某一个await的线程，唤醒
                }else{
                    //让生产者线程等待
                    try{
                        producerCondition.await();//让调用这个方法的线程进入等待状态
                        System.out.println(Thread.currentThread().getName() + "线程进入等待");
                    }catch(InterruptedException e){
                        e.printStackTrace();
                    }
                }
            }finally{
                lock.unlock();//释放锁
            }
        }
        
        //从资源池中取走资源
        public void remove(){
            lock.lock();
            try{
                if(num > 0){
                    num--;
                    System.out.println("消费者" + Thread.currentThread().getName() + "消耗一件资源，" + "当前资源池有" + num + "个");
                    producerCondition.signalAll();//唤醒等待的生产者
                }else{
                    try{
                        //让消费者线程进入等待
                        consumerCondition.await();
                        System.out.println(Thread.currentThread().getName() + "线程进入等待");
                    }catch(InterruptedException e){
                        e.printStackTrace();
                    }
                }
            }finally{
                lock.unlock();
            }
        }
    }
}
```
#### 13.2.3.实现方式3：BlockingQueue
###### 其本质其实是方式2 todo
```java
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;

//使用阻塞队列BlockingQueue解决生产者消费者
public class BlockingQueueConsumerProducer {
    public static void main(String[] args) {
        Resource3 resource = new Resource3();
        //生产者线程
        ProducerThread3 p = new ProducerThread3(resource);
        //多个消费者
        ConsumerThread3 c1 = new ConsumerThread3(resource);
        ConsumerThread3 c2 = new ConsumerThread3(resource);
        ConsumerThread3 c3 = new ConsumerThread3(resource);
 
        p.start();
        c1.start();
        c2.start();
        c3.start();
    }
}
//消费者线程
class ConsumerThread3 extends Thread {
    private Resource3 resource3;
 
    public ConsumerThread3(Resource3 resource) {
        this.resource3 = resource;
        //setName("消费者");
    }
 
    public void run() {
        while (true) {
            try {
                Thread.sleep((long) (1000 * Math.random()));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            resource3.remove();
        }
    }
}
//生产者线程
class ProducerThread3 extends Thread{
    private Resource3 resource3;
    public ProducerThread3(Resource3 resource) {
        this.resource3 = resource;
        //setName("生产者");
    }
 
    public void run() {
        while (true) {
            try {
                Thread.sleep((long) (1000 * Math.random()));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            resource3.add();
        }
    }
}
class Resource3{
    private BlockingQueue resourceQueue = new LinkedBlockingQueue(10);
    /**
     * 向资源池中添加资源
     */
    public void add(){
        try {
            resourceQueue.put(1);
            System.out.println("生产者" + Thread.currentThread().getName()
                    + "生产一件资源," + "当前资源池有" + resourceQueue.size() + 
                    "个资源");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    /**
     * 向资源池中移除资源
     */
    public void remove(){
        try {
            resourceQueue.take();
            System.out.println("消费者" + Thread.currentThread().getName() + 
                    "消耗一件资源," + "当前资源池有" + resourceQueue.size() 
                    + "个资源");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
## 14.模板类和泛型类的区别
###### Java的泛型靠的还是类型擦除，目标代码只会生成一份，牺牲的是运行速度。
###### C++的模板会对针对不同的模板参数静态实例化，目标代码体积会稍大一些，运行速度会快很多。
## 15.Java设计模式
###### 分为三大类：创建型模式（工厂方法模式、抽象工厂模式、单例模式、建造者），结构型模式（适配器模式、装饰器模式、代理模式、外观模式、桥接模式、组合模式、享元模式），行为型模式（策略模式、模板方法模式、观察者模式、迭代子模式、责任链模式、命令模式、备忘录模式、状态模式、访问者模式、中介者模式、解释器模式）
###### 设计模式的六大原则：<br>1.开闭原则：对扩展开放，对修改关闭<br>2.里氏代换原则：只有当衍生类可以替换掉基类，软件单位的而功能不受到影响，基类才能真正被复用，而衍生类也能够在基类的基础上增加新的行为<br>3.依赖倒转原则：这个是开闭原则的基础，对接口编程，依赖于抽象而不依赖于具体<br>4.接口隔离原则：使用多个隔离的接口来降低耦合度<br>5.迪米特法则（最少知道法则）：一个实体应当尽量少的与其他实体之间发生相互作用，使得系统功能模块相对独立<br>6.合成复用原则：原则是尽量使用合成/聚合的方式，而不是使用继承。继承实际上破坏了类的封装性，超类的方法可能会被子类修改
### 15.1.[工厂方法模式](http://www.cnblogs.com/java-my-life/archive/2012/03/28/2418836.html)
### 15.2.[抽象工厂模式](http://www.cnblogs.com/java-my-life/archive/2012/03/25/2416227.html)
### 15.3.[单例模式](http://www.cnblogs.com/java-my-life/archive/2012/03/31/2425631.html)
### 15.4.[建造者模式](http://www.cnblogs.com/java-my-life/archive/2012/04/07/2433939.html)
### 15.5.[原型模式](http://www.cnblogs.com/java-my-life/archive/2012/04/11/2439387.html)
### 15.6.[适配器模式](http://www.cnblogs.com/java-my-life/archive/2012/04/13/2442795.html)
### 15.7.[装饰器模式](http://www.cnblogs.com/java-my-life/archive/2012/04/20/2455726.html)
### 15.8.[代理模式](http://www.cnblogs.com/java-my-life/archive/2012/04/23/2466712.html)
### 15.9.[外观模式](http://blog.csdn.net/jason0539/article/details/22775311)
### 15.10.[桥接模式](http://blog.csdn.net/jason0539/article/details/22568865)
### 15.11.[组合模式](http://blog.csdn.net/jason0539/article/details/22642281)
### 15.12.[享元模式](http://www.cnblogs.com/java-my-life/archive/2012/04/26/2468499.html)
### 15.13.[策略模式](http://www.cnblogs.com/java-my-life/archive/2012/05/10/2491891.html)
### 15.14.[模板方法模式](http://www.cnblogs.com/java-my-life/archive/2012/05/14/2495235.html)
### 15.15.[观察者模式](http://www.cnblogs.com/java-my-life/archive/2012/05/16/2502279.html)
### 15.16.[迭代子模式](http://www.cnblogs.com/java-my-life/archive/2012/05/22/2511506.html)
### 15.17.[责任链模式](http://blog.csdn.net/zhouyong0/article/details/7909456)
### 15.18.[命令模式](http://www.cnblogs.com/java-my-life/archive/2012/06/01/2526972.html)
### 15.19.[备忘录模式](http://www.cnblogs.com/java-my-life/archive/2012/06/06/2534942.html)
### 15.20.[状态模式](http://www.cnblogs.com/java-my-life/archive/2012/06/08/2538146.html)
### 15.21.[访问者模式](http://www.cnblogs.com/java-my-life/archive/2012/06/14/2545381.html)
### 15.22.[中介者模式](http://blog.csdn.net/chenhuade85/article/details/8141831)
### 15.23.[解释器模式](http://www.cnblogs.com/java-my-life/archive/2012/06/19/2552617.html)
## 16.Java反射
###### Java反射机制，通俗来说，就是在运行状态中，我们可以更具类的部分信息，来还原类的全部信息。这里的类的部分信息，可以是类名或累的对象等信息，累的全部信息，是指累的属性、方法、继承关系和注解等内容。比如假设对于类ReflectionTest.java，我们知道的唯一信息是它的类名：com.example.Reflection，这时我们可以通过反射知道它的其他信息
### 16.1.获取Class对象的方法
###### Class.forName("类名字符串")(注意类名必须是全称，包名+类名)
###### 类名.class：Person.class
###### 实例对象.getClass()
###### 类名字符串.getClass()
### 16.2.获取其他信息
###### 获取构造函数
###### 获取成员方法
###### 获取成员变量
###### 获取其他信息（注解、父类、接口、类名、全类名、是不是某种类）
## 17.Java引用
### 17.1.Java引用介绍：
###### Java从1.2版本开始，引入了四种引用，这四种引用的级别由高到低依次为：强引用>软引用>弱引用>虚引用
##### 17.1.1.强引用
###### 强引用是最普遍的引用。如果一个对象具有强引用，那垃圾回收期绝不会回收它。当内存空间不足，Java虚拟机宁愿排除OOM错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足的问题
##### 17.1.2.软引用
```
Object obj = new Object();
SoftReference<Object> sf = new SoftReference<Object>(obj);
sf.get();//获得引用，有时返回为空
```
###### 如果一个对象值具有软引用，则内存空间足够是，GC不会回收它；如果内存空间不足时，就会回收这些对象的内存。只要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可以用来实现内存敏感的高速缓存；软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收器回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列中
##### 17.1.3.弱引用
```
Object obj = new Object();
WeakReference<Object> wf = new WeakReference<Object>(obj);
wf.get();//获得引用，有时返回为空
wf.isEnQueued();//返回是否被垃圾回收器标记为即将回收的垃圾
```
###### 弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期，在第二次垃圾回收的时候回收。在垃圾回收器线程扫描它锁管辖的内存区域的过程中， 一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收他的内存。不过，由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象。弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收器回收，Java虚拟机就会把这个弱引用加入到与之关联的引用队列中
##### 17.1.4.虚引用
```
Object obj = new Object();
PhantomReference<Object> pf = new PhantomReference<Object>(obj);
pf.get();//获得引用,永远返回null
pf.isEnQueued();//返回是否被垃圾回收器标记为即将回收的垃圾
```
###### 虚引用顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收；<br>虚引用主要用来跟踪垃圾回收器回收的活动。虚引用与软引用和弱引用的一个区别在于：虚引用必须和引用队列（ReferenceQueue）联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中
### 17.2.各种引用的内存回收测试
###### finalize()函数时JVM回收内存时执行的，但JVM并不保证在回收内存时一定会调用finalize()。MyDate类重写finalize()函数，在函数中打印出信息，就可以一定程度上追踪内存回收
```java
import java.util.Date;
public class MyDate extends Date {
    public MyDate() {}
    // 覆盖finalize()方法
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("obj [Date: " + this.getTime() + "] is gc");
    }
    public String toString() {
        return "Date: " + this.getTime();
    }
}
```
###### ReferenceTest类定义了一个drainMemory()方法，消耗大量内存，以此来引发JVM回收内存
```java
public class ReferenceTest {
    public ReferenceTest() {}
    // 消耗大量内存
    public static void drainMemory() {
        String[] array = new String[1024 * 10];
        for(int i = 0; i < 1024 * 10; i++) {
            for(int j = 'a'; j <= 'z'; j++) {
                array[i] += (char)j;
            }
        }
    }
}
```
#### 17.2.1.清除对象
```java
public class NoGarbageRetrieve {
    public static void main(String[] args) {
        MyDate date = new MyDate();
        date = null;
    }
}
```
> 无任何输出
###### date虽然设为null，但由于JVM没有执行垃圾回收操作，MyDate的finalize()方法没有被运行
#### 17.2.2.显式调用垃圾回收
```java
public class ExplicitGarbageRetrieve {
    public static void main(String[] args) {
        MyDate date = new MyDate();
        date = null;
        System.gc();
    }
}
```
> obj [Date: 1372137067328] is gc
###### 调用System.gc()，使用JVM运行垃圾回收，MyDate的finalize()方法被运行
#### 17.2.3.隐式调用垃圾回收
```java
public class ImplicitGarbageRetrieve {
    public static void main(String[] args) {
        MyDate date = new MyDate();
        date = null;
        ReferenceTest.drainMemory();
    }
}
```
> obj [Date: 1372137067328] is gc
###### 虽然没有显式调用垃圾回收方法，但是由于运行了消耗大量内存的方法，触发JVM进行垃圾回收
#### 17.2.4.总结
###### JVM的垃圾回收机制，在内存充足的情况下，除非你显式调用System.gc()，否则它不会进行垃圾回收；在内存不足的情况下，垃圾回收将自动进行
|级别|什么时候被GC|用途|生存时间|
|------|------|------|------|
|强引用|从来不会|对象的一般状态|JVM停止运行时终止|
|软引用|在内存不足时|对象简单？缓存|内存不足时终止|
|弱引用|在垃圾回收时|对象缓存|GC运行后终止|
|虚引用|Unknown|Unknown|Unknown|
## 18.回调函数
### 18.1.回调函数的概念
###### 最原始的回调函数是C语言中，回调函数实现了底层调用高层函数的功能。高层在调用底层函数时，将一个函数指针传给被调用的函数，当被调用的函数返回时，会调用该函数指针所指的函数，同时将返回值以参数的形式传给这个函数指针对应的函数。这个函数指针所指的函数就是回调函数。所以回调函数的意思就是调用了一个函数后，被调用的函数回过来调用高层的函数
### 18.2.java回调函数的一般实现
###### 首先要设计一个接口a，里面可以定义一些方法。类b里面保留一个接口a的属性property-c，并设置property-c的setter方法。类d里实例化类b，并传入一个实现了接口a的引用给类b，在特定的事件驱动下类b会调用传入的实现接口a的对象的方法，实现了回调
## 19.dubbo
# 区块链 找程硕 骆志坤 截图 剑指offer

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