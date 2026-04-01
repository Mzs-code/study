# java

## 基础
1. 基础数据类型
   1. byte-8
   2. short-16
   3. char-16
   4. int-32
   5. float-32
   6. long-64
   7. double-64
   8. boolean
      1. 只有2个值 true false 可以用一个bit存储
      2. jvm在编译时转为int 0:false 1:true
2. 包装类型
   1. java是一种面向对象的语言,基本数据类型,在部分场景下不太方便
   2. 基本数据类型都有包装类型,自动装箱,自动拆箱
   3. Integer x = 2; 使用了自动装箱 Integer.valueof(2)
   4. int y = x; 使用了自动拆箱 x.intValue()
   5. 缓冲池
      1. 通过Integer.valueof()创建的值,如果在-128和127直接,则直接返回缓存,上限可以配置
      2. 但通过new Integer(3)每次都会创建一个新的对象
3. String
   1. 被声明为final,不可变,不可以被继承
   2. 通过数组存储,修改操作实际是生成一个新的String
      1. 1.8以及之前
         1. char[],一个字符串占用2个字节
      2. 1.9
         1. byte[]
         2. coder-新增标识字符编码,代表是用哪种编码方式
            1. 0:- LATIN1 编码,一个字符串占用1个字节
            2. 1:- UTF-16 编码,一个字符串占用2个字节
   3. 字符串常量池
      1. 在编译时加入字符串常量池,String a = "123";
      2. String.intern()会将字符串加入常量池
      3. String b = new String("123"),会新建内存空间
      4. 1.7之前在方法区中,之后开始在堆中
   4. 不可变的分析
      1. 保存字符串的数组被final修饰且私有,同时没有提供修改方法
      2. 被final修饰不能被继承,避免了被子类修改
      3. 任何修改都是一个新的字符串
      4. 可以通过反射修改
      5. 优点
         1. 可以缓存hash值
         2. 字符串常量池
         3. 安全性
         4. 线程安全
      6. String StringBuffer StringBuilder 
         1. String不可变, StringBuffer 和 StringBuilder 可变
         2. String不可变线程安全
         3. StringBuffer可变,线程不安全
         4. StringBuilder可变,但内部使用了synchronize,所以线程安全
4. 运算
   1. 参数传递
      1. java中是将参数的值传入方法
      2. 本质是将对对象的引用地址作为参数传入方法
      3. 在方法内修改对象,外部也会修改,因为实际是一个地址的对象
   2. 隐式类型转换
      1. float f = 1.1错误
         1. 1.1字面量是double类型的
         2. float f = 1.1f
      2. 使用 += 或者 ++ 运算符会执行隐式类型转换
         1. short s1 = 1
         2. s1 = s1 + 1显示错误
         3. s1 = s1++ 等价于 s1 = (short) (s1 + 1);
5. 关键字
   1. final
      1. 数据
         1. 基本数据类型,不可修改
         2. 引用类型,本身不可修改,内容可以修改
      2. 方法
         1. 声明方法不能被子类重写
         2. private方法隐式被指定为final
      3. 类-声明不能被继承
   2. static
      1. 静态变量
         1. 在内存中只会有一份
         2. 所有该类的实例数据中该变量只会有一份
      2. 静态方法
         1. 在类加载时候就存在了,不依赖任何实例
         2. 只能访问所属类内的静态变量和方法
      3. 静态代码块
         1. 在类初始化时运行一次
      4. 静态内部类
         1. 不能访问外部非静态的变量和方法
      5. 静态导包
      6. 初始化顺序
         1. 静态变量和代码块优先于实例变量和普通代码块
         2. 初始化顺序取决于在类中的位置
6. Object
   1. equals
      1. 基本类型没有该方法,使用==
   2. hashcode
      1. equals true,hashcode必须一致.反之不一定
      2. 不一致会破坏hashset
   3. toString
      1. 默认返回className@123这种形式,其中@后的值是hashcode的无符号16进制展示
   4. wait
      1. 线程挂起-会释放锁
   5. notify,notifyAll
      1. 必须在 synchronized 块或方法中调用
      2. notify随机唤醒一个线程-通常在只有一个线程时使用
      3. notifyAll唤起所有线程
   6. clone
      1. 类要实现cloneable接口,才能调用clone
      2. 浅拷贝-默认-拷贝后产物和原有对象是同一个对象引用
      3. 深拷贝-不同对象引用
      4. 不建议使用,建议使用构造函数或BeanUtil.copy方法
         1. 默认是浅拷贝
         2. 产物是Object,还需要类型转换
         3. cloneable是一个标记接口,本身没有方法,违反了接口的初衷
         4. 还需要额外处理异常
   7. finalize
      1. 实例在触发GC前会调用一次
      2. 不建议手动执行,影响jvm的GC
   8. getClass
7. 继承
   1. 访问权限修饰符
      1. private、protected 以及 public
   2. 抽象类与接口
      1. 抽象类
         1. 使用 abstract 关键字进行声明
         2. 只能被实例化,不能被抽象
      2. 接口
         1. 是抽象类的延伸
      3. 比较
         1. 继承抽象类需要实现所有方法,接口可自行选择
         2. 接口的字段只能是 static 和 final 类型的，而抽象类的字段没有这种限制
         3. 一个类可以实现多个接口，但是不能继承多个抽象类
         4. 已经实现功能暴露给相关业务类则使用抽象类,比如不同类型的支付相关流程,总体流程相同,部分不同
         5. 接口相对比较灵活
   3. super
      1. 访问父类的构造函数
      2. 访问父类的成员
   4. 重写与重载
      1. 重写(Override)
         1. 子类方法的访问权限必须大于等于父类方法
         2. 子类方法的返回类型必须是父类方法返回类型或为其子类型
         3. 子类方法抛出的异常类型必须是父类抛出异常类型或为其子类型
      2. 重载(Overload)
         1. 方法名和入参决定了方法签名
         2. 返回可以不同
         3. 返回不同,其他不同不能被认为是重载
8. 反射
   1. 核心是java是运行时动态加载生成对应的类或访问对应的方法或属性
   2. 功能
      1. 在运行时判断一个对象的类
      2. 在运行时判断一个类的方法和属性,private也可以
      3. 在运行时生成一个类的对象
      4. 在运行时调用一个类的方法
   3. Class 和 java.lang.reflect
      1. class
      2. java.lang.reflect
         1. Method-可以使用 invoke() 方法调用与 Method 对象关联的方法
         2. Field-可以使用 get() 和 set() 方法读取和修改 Field 对象关联的字段
   4. 用途
      1. 扩展-开发各类通用框架-根据配置动态加载类与方法
      2. IDE-获取类的各类信息,提现在编辑窗口里
      3. 调试器和测试工具
   5. 缺点
      1. 性能开销-涉及到了反射,JVM无法优化
      2. 破坏了安全性,还能获取到private
9. 异常
   1. Throwable可以用来表示可以用做异常抛出的类
   2. Error-jvm抛出的异常
   3. Exception
     1. 受检异常,try catch捕获,并从异常中恢复
     2. 非受检异常,程序运行时错误,导致崩溃无法恢复
10. 泛型
    1.  采用类型擦除的方式来设计泛型,因为要兼容JDK1.5之前的代码
    2.  而c++采用类型替换的方式
    3.  流程
        1.  编译期将类的信息记录到signature中,后续可以通过反射拿到
        2.  编译期将泛型替换为Object,同时在使用处进行类型强行转换
    4.  编译后的代码是没有泛型的
    5.  常见的ArrayList，HashMap等，都是泛型类，而且都是容器类，所以叫泛型容器
    6.  通过上界和多边界对泛型进行约束
    7.  上界-<T extends 类或接口>
    8.  多边界-<T extends 类或接口1 & 接口2 & 接口3...>
11. 注解
    1.  自定义注解
        1.  @interface关键字
        2.  在运行时,使用反射来处理注解
        3.  元注解-用于定义自定义注解
            1.  @Target-指定注解可以应用的目标类型（如方法、字段、类等）
            2.  @Retention-指定注解的保留策略,如RUNTIME（在运行时可见）
    2.  使用场景
        1.  自动生成代码-如Lombok库使用注解来自动生成getter,setter等方法
        2.  配置-Spring框架使用注解来进行依赖注入和配置
        3.  测试-@Mock @Test
        4.  验证-@Validator
12. 不同JDK区分
    1.  JDK 8,11,17是长期支持版本，适合生产环境使用
    2.  1.9 G1
    3.  1.11 ZGC
13. 其他
    1.  java与c++的区别
        1.  面向对象
            1.  java是纯粹面向对象,所有的对象都继承自Object
            2.  c++为了兼容,也支持面向对象,也支持面向过程
        2.  跨平台
            1.  java通过JVM实现跨平台
            2.  c++只支持特点平台
        3.  指针
            1.  java没有指针,引用可以理解为安全指针
            2.  c++有
        4.  垃圾回收
            1.  java自动垃圾回收
            2.  c++手动
        5.  继承
            1.  java不支持多重继承,可以通过实现多个接口
            2.  c++支持
        6.  操作符重载
            1.  java的设计哲学觉得操作符重载会增加代码的复杂度,同时也不直观
            2.  java内置了一些操作符重载
                1.  数字相加
                2.  字符串拼接
                3.  == != += -=
            3.  c++允许
        7.  保留字
            1.  goto
    2.  JRE 与 JDK
        1.  JRE是Java Runtime Environment,java运行环境
        2.  JVM程序,JVM的标准实现与java基本类库
        3.  JDK是Java Development Kit,包括了JRE,java的开发与环境
        4.  包含了编译java的编译器javac
    3.  SPI
        1.  service provider interface
        2.  是java提供的一套交由第三方实现和扩展的接口,一般是框架扩展,插件开发
        3.  核心思想是把一部分决定权交给用户来
        4.  基于接口的编程+策略模式+配置文件来实现动态加载机制
        5.  基于约定在指定位置配置接口实现类,由JDK动态加载和初始化
        6.  如何实现
            1.  定义一个接口以及实现类
            2.  在指定位置配置实现类-resource/META-INF/service
                1.  文件名是接口的全路径名
                2.  内容是实现类的全路径名
            3.  通过serviceLoad加载,再通过反射创建对象加入缓存中
        7.  使用案例
            1. JDBC-JDBC加载不同类型数据库的驱动
            2. DUBBO-对原生SPI进行封装,并加上了filter
            3. Sharing-jdbc的加密方式
               1. 原生之提供了AES和MD52种加密方式
               2. 其他机密方式由用户指定
            4. 日志框架-加载不同供应商的日志实现类
            5. Spring-自动装配
         8. 破坏了双亲委派模型
            1. 将原先由启动加载器加载的类交给了应用加载器
            2. SPI是使用了Thread.currentThread().getContextClassLoader()//线程上下文类加载器
         9. 优点-极大得进行了解耦
         10. 缺点-只能通过遍历加载,即使不需要的也会被加载实例化

## 线程
1. 方法
   1. 实现
      1. 实现 Runnable 接口,实现 Callable 接口,继承 Thread 类
      2. 实现 Runnable 和 Callable 接口的类只能当做一个工作机制,实际还是调用Thread
      3. 但还是比继承Thread好,接口可以实现多个,继承只能一个
   2. 函数
      1. setDaemon()-将一个线程设置为守护线程
         1. 守护线程的生命周期依赖非守护线程(当所有的非守护线程结束时，守护线程会自动终止),适合后台任务
         2. main()是非守护线程
         3. GC 是守护线程
      2. sleep()-休眠当前正在执行的线程
      3. yield()-当前线程会从运行状态转为就绪状态,等待线程调度器重新调度.让出CPU资源
      4. 中断
         1. interrupt()
            1. 中断线程,并不会强制终止线程,而是通过设置中断标志位来提示线程
            2. 如果线程处于阻塞,或有期限的等待或无期限等待时(wait()、sleep()、join()),会先抛出异常,再中断
            3. 但无法中断I/O阻塞与synchronized锁
            4. 具体的中断逻辑需要开发者自己实现
         2. interrupted()
            1. 检查当前线程的中断状态，并清除中断状态
         3. Executor
            1. shutdown()-用于优雅地关闭线程池,等待线程都执行完毕之后再关闭
            2. shutdownNow()-停止接收任务,相当于调用每个线程的 interrupt() 方法
            3. 也可以通过submit()方法提交任务,使用Future作为返回值,调用future.cancel()中断线程
      5. 协作
         1. join()
            1. 让当前线程等待另外一个线程执行完成,自身进入阻塞状态
            2. 使用场景
               1. 主线程等待子线程完成
               2. 线程同步
               3. 资源释放的顺序执行
            3. 注意事项
               1. 等待时中断了,会抛出异常
               2. 超时时间设置
               3. 避免死锁
         2. wait() notify() notifyAll()
            1. 在synchronized同步代码块中使用
            2. wait()挂起线程-释放锁
            3. notify随机唤醒一个线程-通常在只有一个线程时使用
            4. notifyAll唤起所有线程
         3. await() signal() signalAll()
            1. java的java.lang.concurrent包下的condition方法
            2. Condition提供了更细粒度的线程控制,允许一个锁关联多个条件队列
            3. 相比较wait(),await可以指定条件,更加灵活
            4. 比Object下的方法更加清晰,同时也可以避免一些虚假唤醒的问题
   3. 互斥同步
      1. Synchronized
         1. jvm
         2. 不可中断
         3. 非公平
         4. 得益于硬件升级,syn的性能使用自旋锁等机制升级了不少
      2. ReentrantLock
         1. java的java.lang.concurrent包
         2. 可中断
         3. 公平与非公平
         4. 支持多个条件
   4. 状态
      1. new-线程还未开始,还未调用start方法
      2. runnable-可运行状态,线程可能在执行,也可能还在等待CPU资源
      3. running
         1. blocked-线程阻塞等待监视器释放锁
         2. waiting-等待状态-Object,wait(),Thread.join(),LockSupport.pack()
         3. timed_waiting-限时等待-Object,wait(time),Thread.sleep(time),LockSupport.packNanos() packUtil
      4. terminated-终结状态
      5. Monitor 监视锁
         1. synchronized实现依赖 Monitor 监视锁,是一个隐式锁
         2. 每个对象都有一个 Monitor
         3. Monitor 通过锁、等待队列和入口队列实现线程的互斥和协作
         4. 线程想要获取monitor,首先会进入Entry Set队列
         5. 如果线程调用了wait()方法，则会进入Wait Set队列，它会释放monitor锁
2. 线程池
   1. 优点
      1. 降低资源消耗-重复使用已经创建的线程,降低创建与销毁的开销
      2. 提高响应速度-任务进入时,不用再等待线程创建
      3. 线程可管理-使用线程池进行统一调优与监控
   2. ThreadPoolExecutor
      1. 核心参数
         1. 核心线程数-corePoolSize-最小可以同时运行的线程数
         2. 最大线程数-maximumPoolSize-当队列已经满了时,可以同时运行的最大线程数
         3. 队列长度-workQueue-当任务进入时,先判断核心线程数是否满了,如何满了,则加入队列
      2. 其他参数
         1. keepAliveTime-超过核心线程数后创建的线程的最大存活时间
         2. unit-时间单位
         3. threadFactory-线程工厂-一般采用默认
         4. handler-拒绝策略-同时运行的线程达到最大线程数,同时队列已经满了时的执行策略
            1. 抛出异常-默认策略
            2. 丢弃
            3. 丢弃最早执行的
            4. 创建新的线程
      3. 推荐使用,而不是使用Executors,队列太长或线程数太多,会oom
         1. FixedThreadPool 和 SingleThreadExecutor-队列长度为Integer.MAX_VALUE
         2. CachedThreadPool 和 ScheduledThreadPool-线程数量为Integer.MAX_VALUE
   3. 流程
      1. 当前线程小于核心线程数,则创建新线程
      2. 当前线程大于核心线程数,则加入队列
      3. 如果队列已经满了,则检查当前显示小于最大线程数,则创建新线程
      4. 执行拒绝策略
   4. 关闭
      1. shutdown(),平滑关闭,等所有线程执行完成
      2. shutdownNow(),立即关闭,尝试中断所有线程,并返回未执行的任务队列
      3. isShutDown()-会调用shutdown(),并返回true
      4. isTerminated()-会调用shutdown(),并返回等待所有任务执行完成,返回true
   5. 实现
      1. Runnable-不会返回结果与异常
      2. Callable-会返回结果与异常
   6. 执行
      1. execute()-用于提交不需要返回值的任务-Runnable
      2. submit()-用于提交需要返回值的任务-Callable
         1. 会返回一个Future对象
         2. 可以通过get()方法获取结果,会阻塞直到任务执行完成
         3. 基于AQS框架
         4. 支持设置超时时间
   7. 配置
      1. cpu密集-N(cpu)+1-在因为某些原因导致线程暂停时,这个额外的线程可以避免cpu时钟周期浪费
      2. I/O密集-2*N(cpu)
   8. 面试
      1. SpringBoot使用的线程池
         1. 将线程实例化后注入容器中
         2. Spring 的 ThreadPoolTaskExecutor类(就是对JDK ThreadPoolExecutor 的一层包装，可以理解为装饰者模式)
      2. 线程池创建后会马上有线程吗?-不会有,但可以调用方法预热全部或一个
      3. 核心线程会被回收吗?-不会,默认策略是false,可以设置为true
3. ThreadLocal
   1. 线程变量,线程附带的一个的结构
   2. 每个线程都关联一个ThreadLocalMap对象,之间是强引用
   3. ThreadLocal的set动作实际是将自己作为key,存储到ThreadLocalMap中,之间是弱引用,ThreadLocal本身不存储值
   4. 由于是弱引用,当线程还未完成但发生了GC时,ThreadLocal会被回收
      1. 导致ThreadLocalMap中出现了为null的key,对应的value无法被访问-出现了内存泄漏
      2. 当使用ThreadLocal是每次都new一个的场景下,会出现oom
   5. 改进-ThreadLocal的get(),set(),remove()的时候都会清除线程ThreadLocalMap里所有key为null的value
   6. 在使用完ThreadLocal后主动调用remove()方法
4. AQS
   1. Abstract Queued Synchronizer-抽象队列同步器
   2. 是java并发包下的一个核心框架,用于构建锁和其他同步工具（如 ReentrantLock、Semaphore、CountDownLatch 等）
   3. 提供了一种基于 FIFO(先进先出) 等待队列的同步机制,结合cas-非阻塞的乐观锁
   4. 核心思想
      1. 通过一个整型的 state 变量表示同步状态,state 的具体含义由子类定义
         1. ReentrantLock中表示锁的持有次数
         2. Semaphore、CountDownLatch中表示剩余的许可数
         3. 一个 volatile 修饰的整型变量
         4. 0:没有线程持有 1:有线程持有
      2. 通过一个 FIFO 队列管理等待线程
         1. 当线程获取同步状态失败时,会加入队列
         2. 当同步状态释放时,会从队列中唤醒线程
         3. 每个节点（Node）表示一个等待线程
   5. 核心方法
      1. 独占模式（Exclusive Mode）
         1. 同一时刻只有一个线程可以获取同步状态
         2. acquire(int arg)：获取同步状态
         3. release(int arg)：释放同步状态
         4. ReentrantLock
      2. 共享模式（Shared Mode）
         1. 同一时刻可以有多个线程获取同步状态
         2. acquireShared(int arg)：获取共享同步状态
         3. releaseShared(int arg)：释放共享同步状态
         4. Semaphore CountDownLatch CyclicBarrier
   6. 应用
      1. 线程并不是一定安全,更多的是对线程间的协调处理,
      2. ReentrantLock-可重入锁
         1. 同一个线程可以多次加锁
      3. Semaphore-信号量-控制并发线程数,确保同时访问资源的线程数量
         1. 只是控制并发总数,并不保证对于同一资源的竞争
      4. CountDownLatch-倒计时-用来控制一个或者多个线程等待多个线程
         1. 假设定义了一个计数cnt=2
         2. 第一个线程await(),进入等待状态
         3. 第二个线程执行完成,调用countDown(),cnt减一,等于1
         4. 第三个线程执行完成,调用countDown(),cnt减一,等于0
         5. cnt=0,第一个线程被唤醒
      5. CyclicBarrier-循环屏障-用来控制多个线程互相等待，只有当多个线程都到达时，这些线程才会继续执行
         1. 线程执行 await() 方法之后计数器会减 1，并进行等待，直到计数器为 0
         2. 通过调用 reset() 方法可以循环使用
      6. ReentrantReadWriteLock-可重入读写锁
         1. 分别维护了一个读锁,一个写锁-WriteLock为独占锁-ReadLock为共享锁
         2. 读写锁只使用一个state共享变量-高16位代表读锁-低16位代表写锁
         3. 支持锁降级-保持住当前拥有的写锁,再获取到读锁,再释放写锁的过程-为了保证可见性
         4. 读锁只要没有写锁占用且没有超过最大数量,都可以进行尝试获取读锁
         5. 使用ThreadLocal记录当前线程的加锁次数情况
         6. 饥饿问题
            1. 写请求也要排队
            2. 不管是公平锁还是非公平锁,在有读锁的情况下,都不能保证写锁一定能获取到
            3. 1.8中新增的改进读写锁-StampedLock
      7. StampedLock
         1. JDK 1.8引入的对可重入读写锁的升级,不可重入
         2. 通过“戳记（Stamp）”来管理锁的状态
         3. 性能优于 ReentrantReadWriteLock
         4. 原有的独占写锁,共享读锁,乐观读锁(通过 validate(stamp) 方法验证数据是否有效)
      8. 其他
         1. FutureTask-用于表示异步计算结果-实现了 Future 接口和 Runnable 接口
            1. 使用 AQS 来管理任务状态和线程阻塞
         2. BlockingQueue-用于实现线程安全的阻塞队列
            1. 使用 AQS 的独占锁（如 ReentrantLock）来实现线程安全
            2. 使用 AQS 的条件变量（如 Condition）来实现线程的阻塞和唤醒
         3. ForkJoin-并行计算框架，主要用于分治算法和并行任务处理
            1. 并发处理,将大任务拆分为小任务
            2. 工作窃取算法（Work-Stealing Algorithm）来实现任务的并行执行,提高 CPU 的利用率
            3. 每个线程都维护了一个双端队列,用来存储需要执行的任务
            4. 工作窃取算法允许空闲的线程从其它线程的双端队列中窃取一个任务来执行
            5. 窃取的任务必须是最晚的任务,避免和队列所属线程发生竞争
5. 线程安全
   1. 具体查看JVM部分中
6. 面试
   1. 单核可以实现多线程吗?
      1. cpu密集型-如果单个任务执行超过10秒,则不适合,因为切换上下文也要开销
      2. I/O密集型-适合,cpu会分配时间片来并发任务

## IO/NIO/AIO
1. BIO (Blocking I/O)同步阻塞
   1. 基于流
   2. socket.accept()、socket.read()、socket.write()三个主要函数都是同步阻塞的
   3. 单个线程处理时,系统是阻塞的,但此时cpu资源已经释放了
   4. 通常使用线程池实现,为了更好的利用多核和避免开销
      1. 在连接数少的情况下(单机1000),效果较好
      2. 线程池是一个天然的漏洞机制
   5. 在更高的连接,支持不好
      1. 线程本身占用内存资源
      2. 上下文开销太大
      3. 阻塞模式,在网络不稳定的情况下可能同时收到过大请求
2. NIO (NO Blocking I/O)同步非阻塞
   1. 允许应用程序在等待 I/O 操作完成时继续执行其他任务，而不必阻塞等待
   2. 基于buffer
   3. 在等待就绪状态的时候是非阻塞-cpu不使用
   4. 真正的I/O操作是阻塞的,但速度非常快,属于memory copy
      1. NIO底层由操作系统的epoll实现,但未屏蔽系统差异,可能存在cpu100%占用bug
   5. 非阻塞-通过selector选择器来向操作系统询问
      1. NIO的读写函数会马上返回
      2. 一个连接不能读写时,socket.read()或者socket.write()返回0
      3. 同时在Selector选择器上注册标记位记录下来
      4. 然后切换到其它就绪的连接（channel）继续进行读写
   6. 多路复用机制-依赖操作系统支持
      1. 操作系统可以同时扫描同一个端口上不同网络连接的事件
      2. 同一个端口可以处理多种协议
      3. 为不同的多路复用IO技术(操作系统不同)创建一个统一的抽象组，并且为不同的操作系统进行具体的实现
   7. 支持事件驱动模型
      1. 读就绪、写就绪、有新连接到来
      2. 最简单的Reactor模式：注册所有感兴趣的事件处理器，单线程轮询选择就绪事件，执行事件处理器
   8. 抽象
      1. Selector-选择器
         1. 用于监控多个 Channel 的状态（如连接就绪、读就绪、写就绪）
         2. 通过 Selector，一个线程可以管理多个 Channel，实现高效的 I/O 多路复用
         3. 事件订阅和Channel管理
         4. 轮询代理-应用程序不直接和操作系统轮询,而是通过选择器代理
      2. channel-通道
         1. 应用程序和操作系统交互的通道-双向读写
         2. 流的读写是单向的
         3. 无论读写，通道只能和Buffer交互
         4. 因为 Buffer，通道可以异步地读写
      3. buffer-缓冲区
         1. 在通道上交换的数据块
         2. 从通道进行数据读取:创建一个缓冲区,然后请求通道读取数据
         3. 从通道进行数据写入:创建一个缓冲区,填充数据,并要求通道写入数据
   9. 流程
      1.  将 Channel 设置为非阻塞模式
      2.  将 Channel 注册到 Selector 上，并指定感兴趣的事件（如 OP_READ、OP_WRITE、OP_CONNECT）
      3.  通过 Selector.select() 方法轮询已注册的 Channel，检查是否有事件就绪
      4.  如果有事件就绪，处理相应的 I/O 操作
   10. 是面向缓冲,基于通道的I/O操作方法-NIO 通过Channel（通道）进行读写(buffer)
   11. 高级函数-零拷贝
      1. 缓冲区（Buffer）分为堆内存（HeapBuffer）和堆外内存（DirectBuffer）
      2. 为了避免堆内内存被GC回收
   12. 使用场景
       1.  redis-单线程并结合多路复用机制
       2.  dubbo-维护了一个map,将请求号作为key,future方法作为value,结合NIO+长连接
   13. 总结
       1.  事件驱动模型
       2.  避免多线程
       3.  单线程处理多任务
       4.  非阻塞I/O，I/O读写不再阻塞，而是返回0
       5.  基于block的传输，通常比基于流的传输更高效
       6.  更高级的IO函数，zero-copy
       7.  IO多路复用大大提高了Java网络应用的可伸缩性和实用性
3. AIO(Async I/O) 异步非阻塞
   1. 基于事件和回调机制实现-JDK7引入
   2. 基于操作系统实现
      1. Windows-IOCP
      2. Linux-使用epoll模拟异步

## 设计模式
1. 解决问题的方案,学习现有的设计模式可以做到经验复用.拥有设计模式词汇,在沟通时就能用更少的词汇来讨论
2. https://github.com/CyC2018/CS-Notes/blob/master/notes
3. 六大原则
   1. 开闭原则-一个软件实体应当对扩展开放,对修改关闭.即软件实体应尽量在不修改原有代码的情况下进行扩展
   2. 里氏替换原则-所有引用基类对象的地方能够透明地使用其子类的对象
   3. 依赖倒置原则-抽象不应该依赖于具体类，具体类应当依赖于抽象.换言之,要针对接口编程,而不是针对实现编程
   4. 单一职责原则-一个类只负责一个功能领域中的相应职责
   5. 迪米特法则(最少知道原则)-一个软件实体应当尽可能少地与其他实体发生相互作用
   6. 接口分离原则-使用多个专门的接口,而不使用单一的总接口,即客户端不应该依赖那些它不需要的接口
   7. 合成复用原则(六大之外的)-尽量使用对象组合，而不是继承来达到复用的目的
4. 创建型
   1. 单例模式Singleton
      1. 确保一个类只有一个实例,并提供该实例的全局访问点
      2. doubleCheck机制-使用volatile修饰变量,使用synchronized确保只有一个线程进入代码块
      3. [![](https://s21.ax1x.com/2025/01/12/pEPdbSe.png)](https://imgse.com/i/pEPdbSe)
   2. 简单工厂Simple Factory
      1. 在创建一个对象时,不对外暴露细节
      2. 让具体是哪个子类实现类,交给工厂完成
      3. [![](https://s21.ax1x.com/2025/01/12/pEPdLyd.png)](https://imgse.com/i/pEPdLyd)
   3. 工厂模式Factory Method
      1. "定义了一个创建对象的接口"，但由子类决定要实例化哪个类。工厂方法把实例化操作推迟到子类
      2. 在简单工厂中，创建对象的是另一个类，而在工厂方法中,是由子类来创建对象
      3. [![](https://s21.ax1x.com/2025/01/12/pEPdOOA.png)](https://imgse.com/i/pEPdOOA)
   4. 抽象工厂Abstract Factory
      1. 提供一个接口，用于创建 相关的对象家族
      2. 抽象工厂模式用到了工厂模式来创建单一对象
      3. [![](https://s21.ax1x.com/2025/01/12/pEPdvwt.png)](https://imgse.com/i/pEPdvwt)
   5. 生成器Builder
      1. 封装一个对象的构造过程，并允许按步骤构造
      2. [![](https://s21.ax1x.com/2025/01/12/pEPwFyj.png)](https://imgse.com/i/pEPwFyj)
   6. 原型模式Prototype
      1. 使用原型实例指定要创建对象的类型，通过复制这个原型来创建新对象
      2. [![](https://s21.ax1x.com/2025/01/12/pEPwVwq.png)](https://imgse.com/i/pEPwVwq)
5. 行为型
   1. 责任链Chain Of Responsibility
      1. Handler：定义处理请求的接口，并且实现后继链（successor）
      2. 将这些对象连成一条链，并沿着这条链发送该请求，直到有一个对象处理它为止
      3. [![](https://s21.ax1x.com/2025/01/12/pEPwmkV.png)](https://imgse.com/i/pEPwmkV)
      4. [![](https://s21.ax1x.com/2025/01/12/pEPwnYT.png)](https://imgse.com/i/pEPwnYT)
   2. 命令Command
      1. 将命令封装成对象中
      2. 使用命令来参数化其它对象,将命令放入队列中进行排队,将命令的操作记录到日志中,支持可撤销的操作
      3. Command：命令
      4. Receiver：命令接收者，也就是命令真正的执行者
      5. Invoker：通过它来调用命令
      6. Client：可以设置命令与命令的接收者
      7. java.lang.Runnable Netflix Hystrix javax.swing.Action
      8. ![](https://i.miji.bid/2025/01/12/9648d2b1c1518b540e052db7caadef3d.png)
      9. ![](https://i.miji.bid/2025/01/12/bd671e26aeda798fa627d95ed7574ada.png)
      10. ![](https://i.miji.bid/2025/01/12/27c68fdbe3fe67a1d5fdb24026a3527a.png)
   3. 解释器Interpreter
      1. 为语言创建解释器，通常由语言的语法和语法分析来定义
      2. TerminalExpression：终结符表达式，每个终结符都需要一个 TerminalExpression
      3. Context：上下文，包含解释器之外的一些全局信息
      4. java.util.Pattern
      5. ![](https://i.miji.bid/2025/01/12/89d9f4e2d0a1db58c57285bba16fe9a1.png)
      6. ![](https://i.miji.bid/2025/01/12/a413d0fde2ccf09c180f4b3ad03e13d4.png)
      7. ![](https://i.miji.bid/2025/01/12/272faf702d9147563b116bdec2801ee1.png)
   4. 迭代器Iterator
      1. 提供一种顺序访问聚合对象元素的方法，并且不暴露聚合对象的内部表示
      2. Aggregate 是聚合类，其中 createIterator() 方法可以产生一个 Iterator
      3. Iterator 主要定义了 hasNext() 和 next() 方法
      4. Client 组合了 Aggregate，为了迭代遍历 Aggregate，也需要组合 Iterator
      5. java.util.Iterator,java.util.Enumeration
      6. [![](https://s21.ax1x.com/2025/01/12/pEP0tDs.png)](https://imgse.com/i/pEP0tDs)
      7. [![](https://s21.ax1x.com/2025/01/12/pEP0Nbn.png)](https://imgse.com/i/pEP0Nbn)
   5. 中介者Mediator
      1. 集中相关对象之间复杂的沟通和控制方式
      2. Mediator：中介者，定义一个接口用于与各同事（Colleague）对象通信
      3. Colleague：同事，相关对象
      4. 使用中介者模式可以将复杂的依赖结构变成星形结构
      5. All scheduleXXX() methods of java.util.Timer
      6. java.util.concurrent.Executor#execute()
      7. submit() and invokeXXX() methods of java.util.concurrent.ExecutorService
      8. scheduleXXX() methods of java.util.concurrent.ScheduledExecutorService
      9. java.lang.reflect.Method#invoke()
      10. ![](https://pic1.imgdb.cn/item/67837d96d0e0a243d4f39db9.png)
      11. ![](https://pic1.imgdb.cn/item/67837d97d0e0a243d4f39dba.png)
      12. ![](https://pic1.imgdb.cn/item/67837d97d0e0a243d4f39dbb.png)
   6. 备忘录Memento
      1. 在不违反封装的情况下获得对象的内部状态，从而在需要时可以将对象恢复到最初状态
      2. Originator：原始对象
      3. Caretaker：负责保存好备忘录
      4. Memento：备忘录，存储原始对象的状态。备忘录实际上有两个接口，一个是提供给 Caretaker 的窄接口：它只能将备忘录传递给其它对象；一个是提供给 Originator 的宽接口，允许它访问到先前状态所需的所有数据。理想情况是只允许 Originator 访问本备忘录的内部状态
      5. 举例:计算器暂存上一次的结算结果
      6. java.io.Serializable
   7. 观察者Observer
      1. 定义对象之间的一对多依赖，当一个对象状态改变时，它的所有依赖都会收到通知并且自动更新状态
      2. 主题（Subject）是被观察的对象，而其所有依赖者（Observer）称为观察者
      3. 主题（Subject）具有注册和移除观察者、并通知所有观察者的功能，主题是通过维护一张观察者列表来实现这些操作的
      4. 观察者（Observer）的注册功能需要调用主题的 registerObserver() 方法。
   8. 状态State
      1. 允许对象在内部状态改变时改变它的行为，对象看起来好像修改了它所属的类
   9. 策略Strategy
      1. Strategy 接口定义了一个算法族，它们都实现了 behavior() 方法
      2. Context 是使用到该算法族的类，其中的 doSomething() 方法会调用 behavior()
      3. setStrategy(Strategy) 方法可以动态地改变 strategy 对象，也就是说能动态地改变 Context 所使用的算法。
      4. 与状态模式的比较
          1. 都是能够动态改变对象的行为
          2. 状态模式是通过状态转移来改变 Context 所组合的 State 对象
          3. 而策略模式是主动根据Context的决策来改变组合的 Strategy 对象
          4. 策略模式主要是用来封装一组可以互相替代的算法族，并且可以根据需要动态地去替换 Context 使用的算法
          5. ![](https://pic1.imgdb.cn/item/67837fadd0e0a243d4f39e3e.jpg)
   10. 模板方法Template Method
       1. 定义算法框架，并将一些步骤的实现延迟到子类
       2. 通过模板方法，子类可以重新定义算法的某些步骤，而不用改变算法的结构
       3. ![](https://pic1.imgdb.cn/item/67838006d0e0a243d4f39e55.png)
       4. ![](https://pic1.imgdb.cn/item/67838007d0e0a243d4f39e56.png)
       5. ![](https://pic1.imgdb.cn/item/67838007d0e0a243d4f39e57.png) 
   11. 访问者Visitor
       1.  为一个对象结构（比如组合结构）增加新能力
       2.  Visitor：访问者，为每一个 ConcreteElement 声明一个 visit 操作
       3.  ConcreteVisitor：具体访问者，存储遍历过程中的累计结果
       4.  ObjectStructure：对象结构，可以是组合结构，或者是一个集合
   12. 空对象Null
       1. 使用什么都不做的空对象来代替 NULL
6. 结构型
   1. 适配器Adapter
      1. 把一个类接口转换成另一个用户需要的接口
      2. java.util.Arrays#asList()
      3. java.util.Collections#list()
      4. java.util.Collections#enumeration()
      5. javax.xml.bind.annotation.adapters.XMLAdapter
      6. ![](https://pic1.imgdb.cn/item/67838216d0e0a243d4f39eb0.png)
      7. ![](https://pic1.imgdb.cn/item/67838216d0e0a243d4f39eb2.png)
   2. 桥接Bridge
      1. 将抽象与实现分离开来，使它们可以独立变化
      2. Abstraction：定义抽象类的接口
      3. Implementor：定义实现类接口
      4. JDBC
      5. ![](https://pic1.imgdb.cn/item/678382cad0e0a243d4f39edc.png)
      6. ![](https://pic1.imgdb.cn/item/678382cbd0e0a243d4f39ee0.png)
      7. ![](https://pic1.imgdb.cn/item/678382cbd0e0a243d4f39ee3.png)
      8. ![](https://pic1.imgdb.cn/item/678382ccd0e0a243d4f39ee6.png)
   3. 组合Composite
      1. 将对象组合成树形结构来表示“整体/部分”层次关系，允许用户以相同的方式处理单独对象和组合对象
      2. javax.swing.JComponent#add(Component)
      3. java.awt.Container#add(Component)
      4. java.util.Map#putAll(Map)
      5. java.util.List#addAll(Collection)
      6. java.util.Set#addAll(Collection)
      7. ![](https://pic1.imgdb.cn/item/678383cad0e0a243d4f39f63.png)
   4. 装饰Decorator
      1. 为对象动态添加功能
      2. 装饰者（Decorator）和具体组件（ConcreteComponent）都继承自组件（Component）
      3. 所谓装饰，就是把这个装饰者套在被装饰者之上，从而动态扩展被装饰者的功能
      4. 通过调用被装饰者的方法实现,保留了被装饰者的功能
      5. ![](https://pic1.imgdb.cn/item/6783848cd0e0a243d4f39fc9.png)
      6. ![](https://pic1.imgdb.cn/item/6783848dd0e0a243d4f39fcb.png)
   5. 外观Facade
      1. 提供了一个统一的接口，用来访问子系统中的一群接口，从而让子系统更容易使用
      2. ![](https://pic1.imgdb.cn/item/678384fed0e0a243d4f39ff5.jpg)
   6. 享元Flyweight
      1. 利用共享的方式来支持大量细粒度的对象，这些对象一部分内部状态是相同的
      2. Flyweight：享元对象
      3. IntrinsicState：内部状态，享元对象共享内部状态
      4. ExtrinsicState：外部状态，每个享元对象的外部状态不同
      5. Java 利用缓存来加速大量小对象的访问时间
      6. ![](https://pic1.imgdb.cn/item/67838609d0e0a243d4f3a05a.png)
      7. ![](https://pic1.imgdb.cn/item/67838609d0e0a243d4f3a05c.jpg)
   7. 代理Proxy
      1. 控制对其它对象的访问
      2. 模拟了图片延迟加载的情况下使用与图片大小相等的临时内容去替换原始图片，直到图片加载完成才将图片显示出来
      3. ava.lang.reflect.Proxy
      4. RMI
      5. ![](https://pic1.imgdb.cn/item/6783873cd0e0a243d4f3a0f0.png)
      6. ![](https://pic1.imgdb.cn/item/6783873cd0e0a243d4f3a0f1.png)
      7. ![](https://pic1.imgdb.cn/item/6783873dd0e0a243d4f3a0f2.png)
7. 面试
   1. java I/O中的设计模式
      1. 装饰者模式-字节流
         1. 通过组合替代继承来扩展原始类的功能
         2. 在一些继承关系比较复杂的场景（IO 这一场景各种类的继承关系就比较复杂）更加实用
         3. 装饰器模式的核心-FilterInputStream （对应输入流）和FilterOutputStream（对应输出流）
         4. BufferedInputStream（字节缓冲输入流）来增强 FileInputStream 的功能
         5. ![](https://pic1.imgdb.cn/item/67838801d0e0a243d4f3a13c.png)
         6. 同时装饰器模式很重要的一个特征，那就是可以对原始类嵌套使用多个装饰器
         7. BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(new FileOutputStream(fileName), "UTF-8"));
      2. 适配器模式-字符流和字节流-FutrueTask
         1. // InputStreamReader 是适配器，FileInputStream 是被适配的类
         2. InputStreamReader isr = new InputStreamReader(new FileInputStream(fileName), "UTF-8");
         3. // BufferedReader 增强 InputStreamReader 的功能（装饰器模式）
         4. BufferedReader bufferedReader = new BufferedReader(isr);
         5. FutrueTask 类使用了适配器模式
            1. Executors 的内部类 RunnableAdapter 实现属于适配器，用于将 Runnable 适配成 Callable
      3. 装饰者模式与适配器模式的区别
         1. 装饰者模式更注重于增强
         2. 适配器模式更注重于让接口不兼容而不能交互的类可以一起工作
      4. 工厂模式
         1. Files 类的 newInputStream 方法用于创建 InputStream 对象（简单工厂）
         2. Paths 类的 get 方法创建 Path 对象
      5. 观察者模式
         1. NIO 中的文件目录监听服务使用到了观察者模式
         2. 基于 WatchService 接口和 Watchable 接口
         3. WatchService 属于观察者，Watchable 属于被观察者
         4. Watchable 接口定义了一个用于将对象注册到 WatchService（监控服务） 并绑定监听事件的方法 register
         5. WatchService 内部是通过一个 daemon thread（守护线程）采用定期轮询的方式来检测文件的变化
   2. Spring中的设计模式
      1. 工厂模式
         1. 使用工厂模式可以通过 BeanFactory 或 ApplicationContext 创建 bean 对象
      2. 单例模式
         1. Spring 中 bean 的默认作用域就是 singleton(单例)的
         2. 优化GC,同时每个bean只会被创建一次,后续从缓存中获取
         3. 其他方式创建bean
            1. prototype : 每次请求都会创建一个新的 bean 实例
            2. request : 每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP request内有效
            3. session : 每一次HTTP请求都会产生一个新的 bean，该bean仅在当前 HTTP session 内有效
      3. 代理模式
         1. AOP-面向切面编程-运行时增强
         2. 动态代理-Spring AOP 使用 JDK 动态代理或 CGLIB 动态代理
         3. 静态代理-AspectJ 的编译时织入
      4. 模版方法
         1. Spring 中 jdbcTemplate、hibernateTemplate 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式
      5. 观察者模式
         1. Spring 事件驱动模型
         2. 定义一个事件: 实现一个继承自 ApplicationEvent，并且写相应的构造函数
         3. 定义一个事件监听者：实现 ApplicationListener 接口，重写 onApplicationEvent() 方法
         4. 使用事件发布者发布消息: 可以通过 ApplicationEventPublisher 的 publishEvent() 方法发布消息
      6. 适配器模式
         1. Spring MVC-Spring MVC 中的 Controller 种类众多，不同类型的 Controller 通过不同的方法来对请求进行处理
         2. Spring AOP 的增强或通知(Advice)使用到了适配器模式，与之相关的接口是AdvisorAdapter-每个类型Advice（通知）都有对应的拦截器
      7. 装饰者模式
         1. Spring 中配置 DataSource 的时候，DataSource 可能是不同的数据库和数据源

## 集合
1. 集合框架
   1. Collection
      1. List
         1. ArrayList
         2. LinkedList
         3. Vector
      2. Set
         1. HashSet
         2. TreeSet
   2. Map
      1. HashMap
      2. TreeMap
      3. HashTable
2. 数组与集合的区别
   1. 长度
      1. 数组固定
         1. 效率最高的存储和随机访问对象的引用序列-用一个连续内存块来存储
      2. 集合可变
   2. 存储
      1. 数组存储同样类型的数据-可以是基本数据类型,也可以是引用类型
      2. 集合可以存储不同类型-只可以是引用类型
3. set
   1. 元素无序,无重复
   2. HsahSet-线程不安全
      1. 通过hashcode决定位置
      2. 通过eqauls判断是否加入
      3. 如果2个元素hashcode相同,eqauls不同,会使用链式结构存储-会导致性能下降
      4. 允许存入null,但只能有一个null值
   3. LinkedHashSet
      1. 使用链表维护元素的次序
      2. 遍历性能较好,但需要额外维护顺序
   4. SortedSet-TreeSet-线程不安全
      1. 用于排序
      2. 红黑树
      3. 自然排序与定制排序,自然排序使用的是Object的compareTo方法
   5. EnumSet
      1. 有序
      2. 通过枚举类的定义顺序排序
   6. 线程安全-使用Collections工具类的synchronizedSortedSet
4. list
   1. ArrayList
      1. 封装了一个动态增长,允许再分配的Object[]数组-基于数组的线性表
      2. 随机访问性能好
      3. 扩容
         1. new时候可以预估size,减少扩容消耗
         2. 到达临界点扩容1.5倍-原有长度+数组长度>>1(等于原有长度/2)
      4. Arrays.asList()
         1. 返回的是Arrays的内部类,并不是ArrayList对象
         2. 没有变长的特性,不可变-所以不要进行修改操作
      5. subList
         1. 返回是原对象的一个视图-修改会在原对象上生效
         2. 可以用subList进行局部清除-subList(100, 200).clear()
      6. 快速失败机制-fail-fast
         1. java.util下的包都是快速失败的
         2. 多线程操作一个list,可能会抛出concurrent modification exception
         3. 迭代器内部维护了一个modCount,add remove等操作都会触发修改
         4. 一个线程hasNext(),会再调用一次check,如果此时,modCount数据不一致,则抛出异常
         5. 解决
            1. 在做操作的地方加上sync或使用Collections.synchronizedList
            2. 使用CopyOnWriteArrayList
               1. 修改操作会进行一次copy,在copy的数据上进行修改-因为创建大量对象-会有性能损耗
               2. 安全失败-fail-safe-使用的是集合的 快照
               3. java.util.concurrent包下的都是安全失败
   2. LinkedList
      1. 实现了list接口
      2. 实现了Deque接口-可以当做双端队列
      3. 基于链的线性表
      4. 双向链表
   3. Vector-线程安全
      1. 一个古老的集合,和ArrayList在用法上几乎完全相同
      2. Stack-Vector的一个子类,和栈类似,后进先出-LIFO
5. Queue
   1. 用于模拟"队列"这种数据结构(先进先出 FIFO)
   2. PriorityQueue-按照队列元素的大小进行重新排序
6. Deque
   1. 代表一个"双端队列",可以同时从两端来添加,删除元素
   2. ArrayDeque-一个基于数组的双端队列
7. map
   1. 与set和list的关系
      1. java是先实现了map,然后包装了一个value都是null(set)
      2. map中的key的实现方式和set一致
      3. map中的value的实现和存储方式和list类似
   2. HashMap-线程不安全
      1. 内部定义了一个hash表数组-Entry[] table-元素的hash转换为数组的索引
      2. key相等的判断-hash值一致-equals也要为true
      3. 扩容
         1. 初始容量是 16,扩展因子是0.75
         2. 当元素达到总大小的75%,进行resize
         3. new一个2倍大小的table,再从旧table中把数据以hash规则刷新到新table
         4. 先插入元素再判断扩容-JDK 8+
      4. 缺陷
         1. 环形结构-JDK1.7以及之前
            1. 多线程put时,定位到同一个通
            2. 桶内是空的,多线程都插入到链表的头部-头插法
            3. 导致链表节点的next指向错误,扩容结果是一个环形结构
            4. Entry的next节点永远不为空
         2. 死循环
            1. 相邻的Entry扩容后还在想要位置,会出现死循环
            2. 因为新链表的顺序跟旧的链表是完全相反的
         3. JDK 1.8进行了修复,用 head 和 tail 来保证链表的顺序和之前一样-尾插法
         4. 元素丢失-多线程put,扩容后丢失元素
      5. 链表与红黑树
         1. 当链表长度超过一定阈值（默认是 8）时，链表会转换为红黑树,定位和查找效率更高,也不会产生环形
         2. 修改为红黑树之后查询效率直接提高到了 O(logn)
         3. 红黑树-Red-Black-Tree R-B Tree
         4. 近似平衡-查询和插入性能较好-旋转次数也少,3次足以
         5. 维护成本比平衡二叉树低-AVL树
         6. 为什么设定是超过8时转换为红黑树
            1. 泊松分布-通过实验和统计得到
               1. 在 HashMap 的设计中,链表长度超过 8 的概率非常低-6 亿分之一
               2. 因此大部分情况下不会转换为红黑树
            2. 性能权衡
               1. 链表长度较小时,链表性能优于红黑树
            3. 其他
               1. 如果数组长度小于 64，优先进行扩容而不是转换为红黑树
               2. 当红黑树的节点数量减少到一定程度时，HashMap 会将红黑树退化为链表。退化阈值为 6-这是为了避免频繁转换
   3. LinkedHashMap
      1. 使用双向链表来维护key-value对的次序
      2. 和插入顺序一致
   4. Hashtable-线程安全
      1. 使用synchronized来保证线程安全-多线程时效率不高
      2. key和value都不可为null
      3. Properties-Hashtable的子类-键和值必须是字符串-适合配置文件管理
      4. 基本不使用-推荐ConcurrentHashMap
   5. SortedMap
      1. 与TreeSet类似
      2. 红黑树-每个key-value对即作为红黑树的一个节点
      3. TreeMap可以保证所有的key-value对处于有序状态
      4. 自然排序,定制排序
      5. 判断两个key相等的标准
         1. 两个key通过compareTo()方法返回0
         2. equals()放回true
   6. WeakHashMap
      1. key只保留了对实际对象的弱引用
   7. IdentityHashMap
      1. 当且仅当两个key严格相等(key1 == key2)时，IdentityHashMap才认为两个key相等
   8. EnumMap
      1. 创建EnumMap时必须显式或隐式指定它对应的枚举类
   9. 效率
       1. HashMap > Hashtable,Hashtable使用synchronized来保证线程安全
       2. HashMap > TreeMap,TreeMap底层采用红黑树来管理key-value对,需要保证有序
8. ConcurrentHashMap
   1. 不允许用 null 作为键和值
   2. JDK 1.7
      1. 数组 + 链表
      2. 使用锁分段技术
      3. 容器中有多把锁,每一个锁只锁一部分数据
      4. 多线程访问不同数据段的数据时,线程之间不存在锁竞争
      5. 用 HashEntry 对象的不变性来降低执行读操作的线程在遍历链表期间对加锁的需求
      6. 通过对同一个 Volatile 变量的写 / 读访问，协调不同线程间读 / 写操作的内存可见性
      7. Segment[]-充当锁的角色，每个 Segment 对象守护整个散列映射表的若干个桶-默认16个segment
         1. HashEntry[]-每个桶是由若干个 HashEntry 对象链接起来的链表
      8. 分段扩容
   3. JDK 1.8
      1. 数组 + 链表 + 红黑树
      2. CAS + synchronized 来保证并发安全性-直接锁HashEntry(桶),减少锁冲突-HashEntry被锁的概率不大,使用syn解锁效率更高
      3. 如果数量大于 TREEIFY_THRESHOLD=8 则要转换为红黑树,与HashMap一致
      4. 支持动态扩容，扩容时不会阻塞所有操作-通过多线程协作完成

## 概念
1. 低代码
2. DDD
   1. Domain-Driven Design 领域驱动设计
   2. 通过深入理解业务领域，将领域知识融入到软件设计和实现中，从而构建出高质量的、可维护的复杂系统
   3. 核心概念是通用语言和限界上下文
   4. 通用语言指的是在一个业务领域内通用（但不是在更大的领域内也完全通用的）的概念、术语等语言
   5. 限界上下文指的是相邻通用语言之间“翻译”的边界，比如前台业务的用户可能要变成后台清算的客户
3. 中台
   1. 要能比较明确的区分中台和平台-中台是支持多个前台业务且具备业务属性的共性能力组织
   2. 建设
      1. 一个中台组织结构-中台组织关键要懂业务和承担业务职责
      2. 一套支撑技术
         1. 微服务技术
         2. DevOps技术
         3. 云原生技术
         4. 分布式事务技术
      3. 在线业务中台方法论
         1. 技术架构的方法论-微服务、网关、REST API及语义化版本控制、六边形架构
         2. 流程层面的方法论-DevOps、敏捷项目管理
         3. 业务架构的方法论-领域驱动设计(DDD)
4. DSL
5. 分布式id
