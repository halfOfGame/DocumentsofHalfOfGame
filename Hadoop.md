[TOC]



# MapReduce

## 概念

- **Job(作业)** ：一个MR程序称为一个Job
- **MRAppMaster**（MR任务的主节点）:  一个Job在运行时，会先启动一个进程，这个进程为 MRAppMaster。负责Job中执行状态的监控，容错，和RM申请资源，提交Task等！
- **Task**(任务)：Task是一个进程！负责某项计算！
- **Map**(Map阶段): Map是MapReduce程序运行的第一个阶段！Map阶段的目的是将输入的数据，进行切分。将一个大数据，切分为若干小部分！切分后，每个部分称为1片(split)，每片数据会交给一个Task（进程）进行计算！Task负责是Map阶段程序的计算，称为MapTask!在一个MR程序的Map阶段，会启动N（取决于切片数）个MapTask。每个MapTask是并行运行！
- **Reduce**(Reduce阶段)：Reduce是MapReduce程序运行的第二个阶段(最后一个阶段)！Reduce阶段的目的是将Map阶段，每个MapTask计算后的结果进行合并汇总！得到最终结果！Reduce阶段是可选的！Task负责是Reduce阶段程序的计算，称为ReduceTask!一个Job可以通过设置，启动N个ReduceTask，这些ReduceTask也是并行运行！每个ReduceTask最终都会产生一个结果！

## MapReduce中常用的组件

- **Mapper**:  map阶段核心的处理逻辑
- **Reducer**:  reduce阶段核心的处理逻辑
- **InputFormat**: MapReduce的输入格式，MR程序必须指定一个输入目录，一个输出目录！
	如果是普通文件，可以使用FileInputFormat.
	如果是SequeceFile（hadoop提供的一种文件格式），可以使用SequnceFileInputFormat.
	如果处理的数据在数据库中，需要使用DBInputFormat
	
- **RecordReader**:  记录读取器
				RecordReader负责从输入格式中，读取数据，读取后封装为一组记录(k-v)!
- **OutPutFormat**: MapReduce的输出格式
			OutPutFormat代表MR处理后的结果，要以什么样的文件格式写出！
	将结果写出到一个普通文件中，可以使用FileOutputFormat！
	将结果写出到数据库中，可以使用DBOutPutFormat！
	将结果写出到SequeceFile中，可以使用SequnceFileOutputFormat
- **RecordWriter**: 记录写出器
			RecordWriter将处理的结果以什么样的格式，写出到输出文件中！
- **Partitioner**: 分区器
		分区器，负责在Mapper将数据写出时，将keyout-valueout，为每组keyout-valueout打上标记，进行分区！
	目的： 一个ReduceTask只会处理一个分区的数据！

## MapReduce的数据流程

1. InputFormat调用RecordReader，从输入目录的文件中，读取一组数据，封装为keyin-valuein对象
2. 将封装好的key-value，交给Mapper.map()------>将处理的结果写出 keyout-valueout
3. ReduceTask启动Reducer，使用Reducer.reduce()处理Mapper写出的keyout-valueout，
4. OutPutFormat调用RecordWriter，将Reducer处理后的keyout-valueout写出到文件

## MapReduce运行流程示例

- **需求**：
  统计/hello目录中每个文件的单词数量，
  a-p开头的单词放入到一个结果文件中，
  q-z开头的单词放入到一个结果文件中。
  例如： 
  /hello/a.txt   200M
  hello,hi,hadoop
  hive,hadoop,hive,
  zoo,spark,wow
  zoo,spark,wow
  ...

  /hello/b.txt    100m
  hello,hi,hadoop
  zoo,spark,wow
   ...

- 过程

- Map阶段(运行MapTask，将一个大的任务切分为若干小任务，处理输出阶段性的结果)
  1. 切片(切分数据)
     /hello/a.txt   200M
     /hello/b.txt    100m
     默认的切分策略是以文件为单位，以文件的块大小(128M)为片大小进行切片！
     split0:/hello/a.txt,0-128M
     split1: /hello/a.txt,128M-200M
     split2: /hello/b.txt,0M-100M
  2. 运行MapTask（进程），每个MapTask负责一片数据
     split0:/hello/a.txt,0-128M--------MapTask1
     split1: /hello/a.txt,128M-200M--------MapTask2
     split2: /hello/b.txt,0M-100M--------MapTask3
  3. 读取数据阶段
     在MR中，所有的数据必须封装为key-value
     MapTask1,2,3都会初始化一个InputFormat（默认TextInputFormat），每个InputFormat对象负责创建一个RecordReader(LineRecordReader)对象，
  RecordReader负责从每个切片的数据中读取数据，封装为key-value.
     LineRecordReader: 将文件中的每一行封装为一个key（offset）-value(当前行的内容)
  举例：
  hello,hi,hadoop----->(0,hello,hi,hadoop)
  hive,hadoop,hive----->(20,hive,hadoop,hive)
  zoo,spark,wow----->(30,zoo,spark,wow)
  zoo,spark,wow----->(40,zoo,spark,wow)
  5. 进入Mapper的map()阶段
     	map()是Map阶段的核心处理逻辑！ 单词统计! map()会循环调用，对输入的每个Key-value都进行处理！
     输入：(0,hello,hi,hadoop)
     输出：(hello,1),(hi,1),(hadoop,1) 
     
     输入：(20,hive,hadoop,hive)
     输出：(hive,1),(hadoop,1),(hive,1) 
     
     输入：(30,zoo,spark,wow)
     输出：(zoo,1),(spark,1),(wow,1) 
        	
     输入：(40,zoo,spark,wow)
     输出：(zoo,1),(spark,1),(wow,1) 

  5. 目前，我们需要启动两个ReduceTask,生成两个结果文件，需要将MapTask输出的记录进行分区(分组，分类)
     在Mapper输出后，调用Partitioner，对Mapper输出的key-value进行分区，分区后也会排序（默认字典顺序排序）
     分区规则：
     a-p开头的单词放入到一个区
     q-z开头的单词放入到另一个区
     MapTask1:
     0号区：  (hadoop,1)，(hadoop,1)，(hello,1),(hi,1),(hive,1),(hive,1)
     1号区：  (spark,1),(spark,1),(wow,1) ，(wow,1),(zoo,1)(zoo,1)

     MapTask2:
     0号区：  。。。
     1号区： ...

     MapTask3:
     0号区：   (hadoop,1),(hello,1),(hi,1),
     1号区： (spark,1),(wow,1),(zoo,1)

- Reduce阶段
  1. copy
     ReduceTask启动后，会启动shuffle线程，从MapTask中拷贝相应分区的数据！
     ReduceTask1: 只负责0号区
     将三个MapTask，生成的0号区数据全部拷贝到ReduceTask所在的机器！
     (hadoop,1)，(hadoop,1)，(hello,1),(hi,1),(hive,1),(hive,1)
     (hadoop,1),(hello,1),(hi,1),
     		
     		
     ReduceTask2: 只负责1号区
     将三个MapTask，生成的1号区数据全部拷贝到ReduceTask所在的机器！
     (spark,1),(spark,1),(wow,1) ，(wow,1),(zoo,1)(zoo,1)
     (spark,1),(wow,1),(zoo,1)
     
  2. sort
     ReduceTask1:	只负责0号区进行排序：
     	(hadoop,1)，(hadoop,1)，(hadoop,1),(hello,1),(hello,1),(hi,1),(hi,1),(hive,1),(hive,1)
     
     ReduceTask2: 只负责1号区进行排序：
     	(spark,1),(spark,1),(spark,1),(wow,1) ，(wow,1),(wow,1),(zoo,1),(zoo,1)(zoo,1)
     
  3. reduce
     	ReduceTask1---->Reducer----->reduce(一次读入一组数据)
      何为一组数据： key相同的为一组数据
      输入： (hadoop,1)，(hadoop,1)，(hadoop,1)
      输出：   (hadoop,3)
      
      输入： (hello,1),(hello,1)
      输出：   (hello,2)
      
      输入： (hi,1),(hi,1)
      输出：  (hi,2)
      
      输入：(hive,1),(hive,1)
      输出： （hive,2）
      
      ReduceTask2---->Reducer----->reduce(一次读入一组数据)
      
      输入： (spark,1),(spark,1),(spark,1)
      输出：   (spark,3)
      
      输入： (wow,1) ，(wow,1),(wow,1)
      输出：   (wow,3)
      
      输入：(zoo,1),(zoo,1)(zoo,1)
      输出：   (zoo,3)
4. 调用OutPutFormat中的RecordWriter将Reducer输出的记录写出
ReduceTask1---->OutPutFormat（默认TextOutPutFormat）------>RecordWriter（LineRecoreWriter）
    LineRecoreWriter将一个key-value以一行写出，key和alue之间使用\t分割
     在输出目录中，生成文件part-r-0000
     hadoop	3
     hello	2
     hi	2
     hive	2
   
     ReduceTask2---->OutPutFormat（默认TextOutPutFormat）------>RecordWriter（LineRecoreWriter）
     LineRecoreWriter将一个key-value以一行写出，key和alue之间使用\t分割
     在输出目录中，生成文件part-r-0001
     spark	3
     wow	3
     zoo	3

## MapReduce的编写

```java
1. Mapper
		MapTask中负责Map阶段核心运算逻辑的类！
		①继承Mapper<KEYIN,VALUEIN,KEYOUT,VALUEOUT>
		②KEYIN,VALUEIN 取决于InputFormat中RecordReader的设置
			KEYOUT,VALUEOUT由自己定义
		③Mapper的运行流程
			由MapTask调用Mapper.run()
			
			run(){
				setUp();
				while(context.nextKeyValue()) { //循环调用RR读取下一组输入的key-value
					map(key,value,context);
				}
				cleanUp();
			}
		④在Mapper的map()中编写核心处理逻辑
				
2. Reducer
		ReduceTask中负责Reduce阶段核心运算逻辑的类！
		①继承Reducer<KEYIN,VALUEIN,KEYOUT,VALUEOUT>
		②KEYIN,VALUEIN 取决于Mapper的输出
			KEYOUT,VALUEOUT由自己定义
		③Reducer的运行流程
			由MapTask调用Reducer.run()
			
			run(){
				setUp();
				while(context.nextKey()) { // 使用迭代器不断迭代key相同的数据
					reduce(key,value,context);
				}
				cleanUp();
			}
		④在Reducer的reduce()中编写核心处理逻辑

3. Job
		①创建Job
			Job job=Job.getInstance(Configuration conf);
			
		②设置Job
			Job.setName("job名");  //设置名称
			Job.setJar("jar名"); |  Job.setJarByClass(类名)  // 设置Job所在的Jar包，只有在YARN上运行时才需要设置
		③配置Job
				设置输入和输出格式，如果不设置使用默认
				设置Mapper,Reducer
				设置Mapper和Reducer的输出类型 // 主要为了方便根据类型获取对应的序列化器
				设置输入和输出目录
		④运行Job
				Job.waitforCompletion(true);
```





## 常见的输入格式

1. **TextInputFormat**
   场景： TextInputFormat常用于输入目录中全部是文本文件！
   
   切片：默认的切片策略
   
   RecordReader:  LineRecordReader(将一行封装为一个key-value),一次处理一行，将一行内容的偏移量作为key，一行内容作为value!
LongWritable key(行的偏移量)
   Text value(行的内容)
   
2. **NlineInputFormat**
   
   场景：适合一行的内容特别多，在Map阶段map()处理的逻辑非常复杂！根据行数自定义切片的大小！

切片： 读取配置中mapreduce.input.lineinputformat.linespermap，默认为1，以文件为单位，切片每此参数行作为1片！

   RecordReader: LineRecordReader(将一行封装为一个key-value),一次处理一行，将一行内容的偏移量作为key，一行内容作为value!
   LongWritable key(行的偏移量)
   Text value(行的内容)

3. **KeyValueTextInputFormat**
    作用： 针对文本文件！使用分割字符，将每一行分割为key和value!

  如果没有找到分隔符，当前行的内容作为key，value为空串!
  默认分隔符为\t，可以通过参数mapreduce.input.keyvaluelinerecordreader.key.value.separator指定！
  切片：默认的切片策略
  RR ：  KeyValueLineRecordReader(将一行封装为一个key-value)
  Text key(行的分隔符之前的部分内容)
  Text value(行的分隔符之后的部分内容)

4. **ConbineTextInputFormat**
     作用： 改变了传统的切片方式！将多个小文件，划分到一个切片中！
       适合小文件过多的场景！
       RecordReader: LineRecordReader(将一行封装为一个key-value),一次处理一行，将一行内容的偏移量作为key，一行内容作为value!
       LongWritable key(行的偏移量)
       Text value(行的内容)
       切片：  先确定片的最大值maxSize，maxSize通过参数mapreduce.input.fileinputformat.split.maxsize设置！
       流程：
       a. 以文件为单位，将每个文件划分为若干part
       ①判断文件的待切部分的大小 <= maxSize,整个待切部分作为1part
       ②maxsize < 文件的待切部分的大小 <= 2* maxSize,将整个待切部分均分为2part
       ③文件的待切部分的大小 > 2* maxSize,先切去maxSize大小，作为1部分，剩余待切部分继续判断！
       举例： maxSize=2048
       t1.txt 4.38KB
       part1(t1.txt,0,2048)
       part2(t1.txt,2048,1219)
       part3(t1.txt.3xxx,1219)

       t2.txt 4.18KB
       part4(t2.txt,0,2048)
       part5(t2.txt,2048,1116)
       part6(t2.txt,3xxx,1116)

       t3.txt  2.71kb
       part7(t3.txt,0 ,13xxx)
       part8(t3.txt,13 ,27xx)

       t4.txt  5.04kb
       part9(t4.txt,0,2048)
       part10(t4.txt,2048,1519)
       part11(t4.txt.3xxx,1519)

b. 将之前切分的若干part进行累加，累加后一旦累加的大小超过 maxSize，这些作为1片！

## Job提交流程

1. 提交之前的准备阶段
   - 检查输出目录是否合法
   - 为Job设置很多属性(用户，ip,主机名..)
   - 使用InputFormat对输入目录中的文件进行切片，设置Job运行的mapTask的数量为切片的数量
   - 在Job的作业目录生成Job执行的关键文件
      job.split (job的切片对象)
      job.splitmetainfo(job切片的属性信息)
      job.xml(job所有的配置)
   - 正式提交Job
2. 本地模式
		在提交Job后，创建LocalJobRunner.Job.Job对象，启动线程！
	在LocalJobRunner.Job.Job启动的线程中，使用线程池，用多线程的形式模拟MapTask和ReduceTask的多进程运行！
	执行Map,调用线程池，根据Map的切片信息，创建若干MapTaskRunable线程，在线程池上运行多个线程！
	MapTaskRunable------>MapTask--------->Mapper--------->Mapper.run()------->Mapper.map()
			
	Map阶段运行完后，执行Reduce,调用线程池，根据Job设置的ReduceTask的数量，
			创建若干ReduceTaskRunable线程，在线程池上运行多个线程！
			
	ReduceTaskRunable------->ReduceTask------>Reducer----->Reducer.run()------>Reducer.reduce()

3. YARN上运行
在提交Job后，创建MRAppMaster进程！
	由MRAppMaster，和RM申请，申请启动多个MapTask,多个ReduceTask
	Container------>MapTask--------->Mapper--------->Mapper.run()------->Mapper.map()
	Container------->ReduceTask------>Reducer----->Reducer.run()------>Reducer.reduce()
## MapReduce总结

**Map阶段**(MapTask)：

1. 切片(Split)
2. 读取数据，封装为输入的k-v(Read)
3. 交给Mapper处理(Map)
4. 分区和排序(sort)

**Reduce阶段**(ReduceTask):  
1. 拷贝分区数据(copy)
2. 排序(sort)
3. 合并(reduce)
4. 写出(write)

