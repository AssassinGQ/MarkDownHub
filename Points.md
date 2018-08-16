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
## 2.2.进程
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
###### 传输层，包含tcp，udp等协议
###### 网路层，IP协议，负责对数据加上IP地址和其他数据以确定传输的目标
###### 数据链路层，加上以太网协议头，进行CRC编码，为IP模块发送和接受IP数据包，为ARP模块发送ARP请求和接受ARP应答，为RARP发送RARP请求和接受RARP应答。链路层上至少支持loopback协议和以太网协议（RFC894和RFC1042）
###### 硬件层次，包含网卡的定义，网线的制式等
###### 端口号是用在TCP、UDP上的一个逻辑号码，并不是一个硬件端口，我们平时说吧某某端口封掉，也只是在IP层次把带有这个号码的IP包给过滤掉
###### ARP协议
## 4.zookeeper
## 5.redis
## 6.异常分级
## 7.中间件
## 8.Map
### 8.1.Map接口
### 8.2.HashMap
### 8.3.Hashtable
### 8.4.SortedMap接口
### 8.5.TreeMap
### 8.6.LinkedHashMap
## 9.List
## 10.Set
## 11.Queue
## 12.Collection
## 13.String
## 14.数据结构（数组，链表，队列，栈，树，图，堆，散列表，红黑树）
## 15.JVM，GC
## 16.决策树
## 17.操作系统生产者消费者
## 18.模板类和泛型类的区别
###### Java的泛型靠的还是类型擦除，目标代码只会生成一份，牺牲的是运行速度。
###### C++的模板会对针对不同的模板参数静态实例化，目标代码体积会稍大一些，运行速度会快很多。
## 19.Java设计模式
## 20.回调函数
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