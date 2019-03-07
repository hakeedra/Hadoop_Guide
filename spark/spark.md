# Spark

#### spark中stage的划分

- DAG(Directed Acyclic Graph)叫做有向无环图，原始的RDD通过一系列的转换就就形成了DAG，根据RDD之间的依赖关系的不同将DAG划分成不同的Stage，对于窄依赖，partition的转换处理在Stage中完成计算。对于宽依赖，由于有Shuffle的存在，只能在parent RDD处理完成后，才能开始接下来的计算，因此宽依赖是划分Stage的依据。

#### yarn cluster 与client 的区别

- yarn-cluster和yarn-client模式的区别其实就是Application Master进程的区别，yarn-cluster模式下，driver运行在AM(Application Master)中，它负责向YARN申请资源，并监督作业的运行状况。当用户提交了作业之后，就可以关掉Client，作业会继续在YARN上运行。然而yarn-cluster模式不适合运行交互类型的作业。而yarn-client模式下，Application Master仅仅向YARN请求executor，client会和请求的container通信来调度他们工作，也就是说Client不能离开

- cluster
  Spark Driver首先作为一个ApplicationMaster在YARN集群中启动，客户端提交给ResourceManager的每一个job都会在集群的worker节点上分配一个唯一的ApplicationMaster，由该ApplicationMaster管理全生命周期的应用。因为Driver程序在YARN中运行，所以事先不用启动Spark Master/Client，应用的运行结果不能在客户端显示（可以在history server中查看），所以最好将结果保存在HDFS而非stdout输出，客户端的终端显示的是作为YARN的job的简单运行状况。

  步骤如下：

  1. 由client向ResourceManager提交请求，并上传jar到HDFS上
     这期间包括四个步骤：
     ​	a). 连接到RM
     ​	b). 从RM ASM（ApplicationsManager ）中获得metric、queue和resource等信息。
     ​	c). upload app jar and spark-assembly jar
     ​	d). 设置运行环境和container上下文（launch-container.sh等脚本)

  2. ResouceManager向NodeManager申请资源，创建Spark ApplicationMaster（每个SparkContext都有一个ApplicationMaster）

  3. NodeManager启动Spark App Master，并向ResourceManager AsM注册

  4. Spark ApplicationMaster从HDFS中找到jar文件，启动DAGscheduler和YARN Cluster Scheduler

  5. ResourceManager向ResourceManager AsM注册申请container资源（INFO YarnClientImpl: Submitted application）

  6. ResourceManager通知NodeManager分配Container，这时可以收到来自ASM关于container的报告。（每个container的对应一个executor）

  7. Spark ApplicationMaster直接和container（executor）进行交互，完成这个分布式任务。

     需要注意的是：
     ​	a). Spark中的localdir会被yarn.nodemanager.local-dirs替换
     ​	b). 允许失败的节点数(spark.yarn.max.worker.failures)为executor数量的两倍数量，最小为3.
     ​	c). SPARK_YARN_USER_ENV传递给spark进程的环境变量
     ​	d). 传递给app的参数应该通过–args指定。

- client

  YarnClientClusterScheduler查看对应类的文件，在yarn-client模式下，Driver运行在Client上，通过ApplicationMaster向RM获取资源。本地Driver负责与所有的executor container进行交互，并将最后的结果汇总。结束掉终端，相当于kill掉这个spark应用。一般来说，如果运行的结果仅仅返回到terminal上时需要配置这个。

  客户端的Driver将应用提交给Yarn后，Yarn会先后启动ApplicationMaster和executor，另外ApplicationMaster和executor都 是装载在container里运行，container默认的内存是1G，ApplicationMaster分配的内存是driver- memory，executor分配的内存是executor-memory。同时，因为Driver在客户端，所以程序的运行结果可以在客户端显 示，Driver以进程名为SparkSubmit的形式存在。

#### yarn调度器

- FIFO 调度器：先进先出

- Capacity 调度器：容器调度器，可以将它理解为一个资源队列

  - 层次化的队列设计，这种层次化的队列设计保证了子队列可以使用父队列的全部资源。这样通过层次化的管理可以更容易分配和限制资源的使用。
  - 容量，给队列设置一个容量(资源占比)，确保每个队列不会占用集群的全部资源。
  - 安全，每个队列都有严格的访问控制。用户只能向自己的队列提交任务，不能修改或者访问其他队列的任务。
  - 弹性分配，可以将空闲资源分配给任何队列。当多个队列出现竞争的时候，则会按照比例进行平衡。
  - 多租户租用，通过队列的容量限制，多个用户可以共享同一个集群，colleagues 保证每个队列分配到自己的容量，并且提高利用率。
  - 可操作性，Yarn支持动态修改容量、权限等的分配，这些可以在运行时直接修改。还提供管理员界面，来显示当前的队列状态。管理员可以在运行时添加队列；但是不能删除队列。管理员还可以在运行时暂停某个队列，这样可以保证当前队列在执行期间不会接收其他任务。如果一个队列被设置成了stopped，那么就不能向他或者子队列提交任务
  - 基于资源的调度，以协调不同资源需求的应用程序，比如内存、CPU、磁盘等等

- Fair 调度器：公平调度器

  Fair 调度器是一种队列资源分配方式，在整个时间线上，所有的 Job 平分资源。默认情况下，Fair 调度器只是对内存资源做公平的调度和分配。当集群中只有一个任务在运行时，那么此任务会占用集群的全部资源。当有其他的任务提交后，那些释放的资源将会被分配给新的 Job，所以每个任务最终都能获取几乎一样多的资源。	

#### spark shuffle优化

​	Shuffle阶段需要将数据写入磁盘，这其中涉及大量的读写文件操作和文件传输操作，因此对节点的系统IO有比较大的影响，因此可通过调整参数，减少shuffle阶段的文件数和IO读写次数来提高性能，具体参数如下：

-  **spark.shuffle.manager**

  设置Spark任务的shuffleManage模式，1.2以上版本的默认方式是sort,即shuffle write阶段会进行排序，每个executor上生成的文件会合并成两个文件（包含一个索引文件）

- **spark.shuffle.sort.bypassMergeThreshold** 

  设置启用bypass机制的阈值（默认为200），若Shuffle Read阶段的task数小于等于该值，则Shuffle Write阶段启用bypass机制

- **spark.shuffle.file.buffer** （默认32M） 

  设置启用bypass机制的阈值（默认为200），若Shuffle Read阶段的task数小于等于该值，则Shuffle Write阶段启用bypass机制

- **spark.shuffle.io.maxRetries** （默认3次） 

  设置Shuffle Read阶段fetches数据时的重试次数，若shuffle阶段的数据量很大，可以适当调大一些

- shuffle 操作的时候可以用 combiner 压缩数据,减少 IO 的消耗

#### 宽依赖 窄依赖

- 窄依赖指的是每一个父RDD的Partition最多被子RDD的一个Partition使用 

  ​	map、filter、union等操作会产生窄依赖

- 宽依赖指的是多个子RDD的Partition会依赖同一个父RDD的Partition 

  ​	groupByKey、reduceByKey、sortByKey等操作会产生宽依赖，会产生shuffle

- join操作有两种情况：如果两个RDD在进行join操作时，一个RDD的partition仅仅和另一个RDD中已知个数的Partition进行join，那么这种类型的join操作就是窄依赖，例如图1中左半部分的join操作(join with inputsco-partitioned)；其它情况的join操作就是宽依赖,例如图1中右半部分的join操作(join with inputsnot co-partitioned)，由于是需要父RDD的所有partition进行join的转换，这就涉及到了shuffle，因此这种类型的join操作也是宽依赖。



































