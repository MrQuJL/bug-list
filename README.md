# work-journey

## 每日一记

1. 不要轻易删除数据库中数据，与该表相关联的业务极有可能收到影响，例：某程序员误删了用户表中某用户的记录，但在订单表中仍然存储着该用户的订单记录，导致查询订单时，从订单表中获取用户的详细信息时报空指针异常。

2. 不要为了写优雅的代码轻易使用极易出现空指针的链式调用写法，导致后台报空指针时排查困难。

    反例：

    ```
    order.setMsg(userMap.get(order.userId()).getUserInfo());
    ```

    若从userMap中获取了null，则调用getUserInfo方法时就会报空指针异常，此类小错看起来可笑，但是却很常见。

3. 在实现一个需求的时候，需要事先与前台约定好数据的格式，不要一意孤行。

4. 重写Comparator的compare方法时，第一个值大于第二个值，返回1表示升序，-1降序；第一个值小于第二个值，返回1表示降序，返回-1表示升序。

5. 慎用复制粘贴：

    反例：

    ```
    int s1 = o1.getShopCount().intValue();
    int s2 = o1.getShopCount().intValue();
    ```

    这种错误IDE不会报错，很难发现。

6. 在向session中缓存数据的时候，对应的实体类必须实现Serializable接口，否则会导致序列化失败

7. 查询商品列表的同时计算每件商品的昨日销量，过去7天销量，过去30天销量需要频繁查询订单明细表中的数据，造成性能低下。

    > 解决：一次性将过去30天内已经付款的订单加载到内存，在内存里计算。（此种方式仅限于订单量较小的情况）

8. 使用```@GeneratedValue```注解的字段，必须是自增长的字段，比如int，如果是varchar类型，不要使用该注解，否则在用jpa的save方法保存对象的时候会报错：

    Filed id dosen't have a default value

9. 使用Java8的parallelStream（并行流）可以实现并行化流操作，但是要想达到最大的效果，流中元素的个数必须很大，CPU的核心数必须很多。

10. 在使用Java8的StreamAPI来进行编码时可以使用 ```peek``` 方法来进行断点调试，例如在下方代码的 ```peek``` 那一行点上断点，然后以```debug``` 模式运行就可以断点调试，也建议使用下面那样的调用方式（调用的方法排成一列，这样就方便断点调试）：

    ```java
    public class LambdaPeek {

        public static void main(String[] args) {
            List<String> list = Stream.of("ABC", "DEF", "Ghi")
                                      .peek(e -> System.out.println(e.toLowerCase()))
                                      .collect(Collectors.toList());
        }
    }
    ```

11. 什么是web1.0，web2.0，web3.0？

    web1.0：Yahoo，AOL，新浪，搜狐等。**关注点是是网络的连接**。
    web2.0：博客，互动论坛，今日头条，拼多多，大众点评，社交网络等。**关注点是信息的互动**。
    web3.0：Steemit ，趣头条，区块链等。**关注点是利益共享，激励**。

    Web 1.0是**B2C**模式，由商家直接提供服务，是单向的行为，受中心化控制。

    Web 2.0则是**C2B**，中心相对变弱，用户的影响力增强，平台规模急速扩大，用户受制于平台控制。

    Web 3.0相当于**C2C**，在这里，每一个C都可以更加顺畅、更加简单、更低成本地跟其他C产生交易，完全不依赖于平台，不受平台掌控，共同分享整个平台的收益。 

12. Navicat中表的记录一页只显示1000条，多出的记录在下一页，但是不会主动显示有第二页，坑。

13. @Transactional注解需要放到所有注解的最上面才起作用。

14. jpa的Repository接口的抽象方法上也是可以添加@Transactional注解的。

15. Spring的@Scheduled注解可以用来跑定时任务。

16. 创建订单的时候减**商品**的库存，只有在发货的时候才减**仓库商品**的库存。

17. 如果商品已经发货了，就不可以取消订单了。

18. 所以取消订单的时候也只是加的商品的库存，不需要加仓库商品的库存。

19. SpringBoot启动默认是需要向的DataSourceAutoConfiguration类注入dataSource的，如果工程中没有与dataSource相关的配置，当创建dataSource bean时，就会因为缺少相关信息报如下错误：

	Cannot determine embedded database driver class for database type NONE

	解决：

	> 在application.yml中添加如下配置:

		spring:
		  datasource:
			driver-class-name: com.mysql.jdbc.Driver
			username: admin
			password: 123456
			url: jdbc:mysql://127.0.0.1:3306/blog?useUnicode=true&characterEncoding=utf-8&useSSL=false

20. 大型网站核心架构要素：

	1. 可用性：网站 7*24 是否全时段可用？
	2. 性能：网站对请求的响应是否迅速？超出负载的情况下呢？
	3. 伸缩性：向集群中不断加入服务器是否可以缓解不断上升的**用户并发访问压力**和**不断增长的数据存储需求**？
	4. 扩展性：网站增加新的业务功能模块时，是否可以保持对现有业务的透明？
	5. 安全性：针对现存和潜在的各种攻击，是否有可靠的应对策略？

21. 日志表有时候也需要设计成具有主从关系的。例：记录商品的非正常出入库记录，一次记录中可能操作了多个商品，这时就要把日志表设计成具有主从关系的。主表记录该次记录相同的信息，从表记录单个商品的详细信息。

22. 使用fastjson解析json格式的数组：

	```
	List<OrderDetail> orderDetailList = JSONArray.parseArray(items, OrderDetail.class);
	```

23. 取消订单只返回商品的库存，不返回仓库商品的库存，因为订单可以取消的话，就说明还没有发货，而只有发货的商品才会减仓库商品的库存，所以在取消订单的时候仓库商品的库存是没有变化的。

24. ```tar -xvf file.tar``` : 解压tar包

25. ```tar -zxvf file.tar.gz``` : 解压tar.gz包

26. ```tar -jxvf file.tar.bz2``` : 解压tar.bz2包

27. ```tar -Zxvf file.tar.Z``` : 解压tar.Z包

28. ```rpm -ivh tree-1.6.0-10.el7.x86_64.rpm``` : 安装tree命令，其中i表示安装，v表示显示安装过程，h表示显示进度

29. ```tree -d -L 2``` : 以树形结构查看当前路径下的文件夹，深度为两层
	```
	.
	└── jdk1.8.0_144
		├── bin
		├── db
		│   ├── bin
		│   └── lib
		├── include
		│   └── linux
		├── jre
		│   ├── bin
		│   ├── lib
		│   └── plugin
		├── lib
		│   ├── amd64
		│   ├── missioncontrol
		│   └── visualvm
		└── man
			├── ja -> ja_JP.UTF-8
			├── ja_JP.UTF-8
			└── man1
	```
30. Linux清屏快捷键：Ctrl+L

31. JDK自带的jps命令可以用来查看Java进程的PID。

32. 使用```kill -3 PID```命令可以查看Thread Dump信息，用来分析死锁和性能问题。

33. 使用JPA的save方法存储对象时，要注意存储对象数据类型的一致.

	反例:
	@Entity
	@DynamicUpdate
	public class OrderMaster {
		...
	}

	public class OrderDto extends OrderMaster {
		private List<OrderDetail> orderDetailList;
	}

	OrderMaster orderMaster = findOne(orderId); // 注意：该方法返回OrderDto类型，但是IDE不会保存

	orderMasterRepository.save(orderMaster); // 执行的时候就会报如下错误：

	Unknown entity: com.xxx.xxx.dto.OrderDto

34. Integer类型不能使用```@NotEmpty```注解进行校验

35. String类型可以使用```@NotEmpty```也可以使用```@NotNull```注解进行校验

36. 某个Controler上已经添加了 ```@Controller``` 注解，若方法要返回json格式的字符串，需要在方法上额外添加 ```@ResponseBody``` 注解，如果不添加则数据仍会返回，但是 SpringMVC 会同时转发到一个请求名.html的url去，导致404。

	带来的思考：由于项目中的某个Controller里面的方法过多，笔者已经忘记了这个Controller里面的方法默认返回的是视图名，而不是json格式的数据，为了方便，就顺手在这个Controller中加了个方法来返回json数据，导致获取到数据的同时，又跳转到了一个 ```请求名.html``` 的请求，导致 404。以后一定要看准Controller上的注解再添加方法。

37. Linux 的 tmp 目录特点：重启后里面的文件会丢失，所以 Hadoop 默认的数据保存位置一定要在 core-site.xml 中修改。

38. 在搭建Hadoop全分布式环境的时候，一定要用一个全新的hadoop jar包，不要用之前做伪分布式格式化之后的hadoop，容易出现的问题：

	从机上的DataNode进程处于运行状态，但是从网页上查看，却显示live Nodes数量为0

	一旦出现这种问题，不要尝试删除hadoop保存数据的tmp目录然后重新格式化，没用。最简单的方法，删掉hadoop，重装。

39. Controller层如果要做多次写操作，需要封装到Service层，调用Service层方法，Service层出现异常可以回滚。

40. HDFS 客户端与 NameNode 的通信使用的是代理对象以及 RPC 协议。

41. idea导出jar包时需要将META-INF目录的输出位置指定为resources目录，否则，MAINFEST.MF文件导不出来。

42. 某个字段有默认值的话，最好在 Java 实体类中设置属性的默认值，数据库设置的默认值有时候不管用。

43. 使用 JPA 的 save 方法时报 could not read a hi value。

	解决办法：把数据库实体类的注解@GeneratedValue改成@GeneratedValue(strategy = GenerationType.IDENTITY) ，试了试真的可以

	@GeneratedValue(strategy = GenerationType.IDENTITY)的意思是把Hibernate提供的主键生成策略设置为identity （即自增）

44. 使用 POSTMAN 时，测试 post 请求需要将参数放到 body 里面，否则请求到不了 Controller。

45. 使用 hadoop 的 FileSplit 时，要导入 org.apache.hadoop.mapreduce.lib.input.FileSplit 该包，不要导成这个包下的：org.apache.hadoop.mapred。

46. 无法通过公网ip直接获取阿里云ECS服务器上的zookeeper的地址。
	
	解决：单独购买HBase服务器，配置ip白名单。

47. VMware配置的 ip 地址可能会发生变化，在使用多台虚拟机模拟 hadoop 的全分布环境时，要时刻检查虚拟机的 ip 地址，ip 地址发生变化后，要马上修改 /etc/hosts 文件的 DNS 映射。

48. 向 HBase 中存储数据时，要时刻监控 Region 的情况，当一个 Region 的容量过大时，需要及时手动的对 Regiion 进行平衡分裂，避免线上被动分裂占用带宽导致网络不可用。

49. 有关字符串的操作，比如replace将返回一个新的字符串，原来的字符串不会改变。（ps：背面试题的时候背挺6，真正写起代码就忘了）

50. POI中的XWPFDocument表示整个word文档，XWPFTable表示文档中的表格（可能有多个），XWPFTableRow表示表格中的行（可能有多个），XWPFTableCell表示一行中的单元格（可能有多个），XWPFParagraph表示段落（以回车符结尾区分），XWPFRun表示段落中的文本（一个字被封装成一个XWPFRun）。

51. 慎用自动导包功能，极有可能导入了错误的类，但是IDE没报错，编译也没报错，但是一运行就抛各种奇怪的异常。

52. 使用 POI 的 XWPFRun 的 addPicture 方法时，一定要注意后面两个参数要使用 Units.toEMU 来转换一下宽度和高度，否则图片会一直不显示（搞了两天才搞出来）。

	例：
	```
	XWPFRun run = paragraph.createRun();
	FileInputStream fis = new FileInputStream(new File("D:/usr/local/java/jar/chmail_template/barcode/" + e.getValue() + ".png"));
	run.addPicture(
			fis,	// 图片的位置
			Document.PICTURE_TYPE_PNG, // 图片的类型
			e.getValue() + ".png", // 图片的名字
			Units.toEMU(200), // !!!切记，这里一定要使用 Units.toEMU 来转换长宽高，否则图片无法显示
			Units.toEMU(50) // !!!切记，这里一定要使用 Units.toEMU 来转换长宽高，否则图片无法显示
	);
	fis.close();
	```

53. Hive的分区表分的是目录，桶表分的是文件。

54. 服务器上导出数据报表的时候，要为服务器设置字体库，否则中文字体出不来。

55. HDFS联盟的作用是缓存更多的元信息以及实现负载均衡，Zookeeper的HA的作用是为了实现高可用。

56. Memcached中存储的数据只能放在内存中，不能持久化，Redis中的数据可以持久化：RDB，AOF。Memcached集群（官方提供的版本）的各个节点之间是不能互相通信的，一个日本工程师改良了官方的版本，使 Memcached 集群的各个节点之间可以互相通信进行主主复制。

57. **一条来自真实的生产环境的经验**：在本地测试Hadoop的JavaAPI，是直接与NameNode通信，比方说上传文件，先与NameNode通信获取DataNode的地址，然后本地直接与DataNode通信，如果是使用虚拟机测试一点问题没有。但是，在真实的生产环境的话，返回的DataNode的地址是内网地址，而本地直接访问返回的内网地址，访问的是我们的内网地址，无法访问DataNode，导致上传的文件大小为0.解决：Configuration配置返回DataNode的主机名，然后在本地配置主机名和ip地址的映射，开启服务器上的50010端口，本地就可以正常上传文件。

58. 局域网的ip地址段：

	* 10.0.0.0 - 10.255.255.255
	* 172.16.0.0 - 172.31.255.255
	* 192.168.0.0 - 192.168.255.255

59. **一条来自真实的生产环境的经验**：服务器A和服务器B启动Redis服务，服务器C启动Twemproxy的nutcracker（redis的代理分片，在读取后端的两台redis服务器中的缓存时，提供负载均衡服务），在C上执行redis-cli -p 22121命令来访问后端的两台Redis服务器时发现连接被拒绝。原因：代理分片想要读取A和B上的Redis中的数据，需要访问6379端口，阿里云服务器的安全组入方向的配置中没有开放6379端口。解决：配置安全组规则，开放6379端口。

60. **一条来自真实的生产环境的经验**：只能通过代理分片get后端redis的数据，不能写。

61. 分别测试通过代理分片服务器和直接访问redis服务器这两种方式，测试服务器每秒能够处理的请求数：

	root@secret:~/training/redis# bin/redis-benchmark -h redis111 -p 6379 -c 100 -n 10000 -t get -q
	GET: 1003.61 requests per second

	root@secret:~/training/redis# bin/redis-benchmark -h redis112 -p 6379 -c 100 -n 10000 -t get -q
	GET: 1138.69 requests per second

	root@secret:~/training/redis# bin/redis-benchmark -h localhost -p 22121 -c 100 -n 10000 -t get -q
	GET: 4885.20 requests per second

	前两次测试是直接访问的redis服务器，平均每秒能处理1000个请求
	
	最后一次是通过代理分片访问的redis服务器，平均每秒能处理近5000请求
	
62. 在执行 `bin/redis-trib.rb create --replicas 1 192.168.157.111:6379 192.168.157.111:6380 192.168.157.111:6381 192.168.157.111:6382 192.168.157.111:6383 192.168.157.111:6384` 命令启动 redis cluster 时如果出现如下错误：

	/var/lib/gems/2.3.0/gems/redis-3.0.5/lib/redis/client.rb:85:in `call': ERR Slot 0 is already busy (Redis
	from /var/lib/gems/2.3.0/gems/redis-3.0.5/lib/redis.rb:2288:in `block in method_missing'
	from /var/lib/gems/2.3.0/gems/redis-3.0.5/lib/redis.rb:36:in `block in synchronize'
	from /usr/lib/ruby/2.3.0/monitor.rb:214:in `mon_synchronize'
	from /var/lib/gems/2.3.0/gems/redis-3.0.5/lib/redis.rb:36:in `synchronize'
	from /var/lib/gems/2.3.0/gems/redis-3.0.5/lib/redis.rb:2287:in `method_missing'
	from bin/redis-trib.rb:205:in `flush_node_config'
	from bin/redis-trib.rb:667:in `block in flush_nodes_config'
	from bin/redis-trib.rb:666:in `each'
	from bin/redis-trib.rb:666:in `flush_nodes_config'
	from bin/redis-trib.rb:1007:in `create_cluster_cmd'
	from bin/redis-trib.rb:1388:in `<main>'
	
	说明redis集群中的每个节点在搭建集群之前的旧数据和配置信息没有清理干净。解决：用 redis-cli 登录到每一个节点，执行 flushall 和 cluster reset命令，然后重新启动集群就可以了。
	
63. 在配置 redis 集群部署的时候，每个节点的配置文件(redis.conf)中的 pidfile不要忘记修改。

64. **一条来自真实的生产环境的经验**：启动RedisCluster的时候要用内网的ip地址启动，不要使用localhost，127.0.0.1启动，否则本地调试使用JavaAPI无法访问（本地访问要使用公网地址，阿里云服务器会将公网ip映射到对应的内网机器上）。

65. 在使用BigDecimal的setScale方法设置精度的时候，要添加舍入模式，否则小数点位数过多的时候会抛出异常。

	money.setScale(2, BigDecimal.ROUND_HALF_DOWN)

66. 在运行storm程序的时候如果提示找不到某个类，一个简单的方法是可以把对应的jar包放到storm的lib目录下，因为storm启动的时候会去加载lib目录下的jar。

67. mybatis在进行多条件查询的时候，支持的写法：Parameter 'arg5' not found. Available parameters are [arg3, arg2, param5, arg4, arg1, arg0, param3, param4, param1, param2]

	例：arg0, arg1, arg2, arg3, arg4
	例：param1, param2, param3, param4, param5

68. mybatis与springboot整合之后，需要在接口定义的的参数前加@Param("xxx")注解，xxx即为mapper.xml里对应sql中的参数名，不加注解的话会报：There is no getter for property named 'xxxxx' in 'class java.lang.String'

69. 301表示永久重定向，302表示临时重定向，并且在实际应用中：重定向的那个请求的方法会修改为GET

70. 使用POI进行word转pdf的时候一定要设置整个word的文字的颜色为黑色，否则在转换的过程中代码会因为无法decode颜色的代码而抛出NumberFormatException

71. 在进行LIKE模糊查询的时候，如果某个字段是NULL，则没有办法通过 LIKE '%%' 查询到。

72. 在使用mybatis进行模糊查询时，针对实际查询的字段一般这么写：LIKE CONCAT('%', #{content}, '%')

73. 针对多字段模糊查询的时候，可以使用MySql的CONCAT_WS('',column1,column2,column3,..)带有分隔符的拼接函数，来拼接多个字段，这样可以避免因一个字段为NULL而导致查询结果为空

74. 在对Spark的RDD进行操作的时候尽量针对每一个分区进行操作，可以避免多个分区共用一个对象而造成的序列化问题，比如：将RDD中的数据保存到数据库。

75. 在Linux上通过rpm的方式安装mysql，安装rpm软件包的时候要加上 --force --nodeps 避免依赖旧的软件包

	例：rpm -ivh mysql-community-libs-5.7.19-1.el7.x86_64.rpm --force --nodeps

76. 手动释放Linux内存：

	1. 执行 sync 命令将 Linux 缓存的数据写到硬盘上，确保数据的完整性
	
	2. 执行 echo 1 > /proc/sys/vm/drop_caches 修改 drop_caches 文件的值为1
	
	3. drop_caches的值可以是0-3之间的数字，代表不同的含义：
		
		0：不释放
		1：释放页缓存
		2：释放dentries和inodes
		3：释放所有缓存

77. Spark的检查点设置为HDFS目录时，本地调试的时候也需要返回datanode的主机名，然后配置hosts映射来访问datanode，相应的创建StreamingContext的时候就需要使用如下接口以便于传入Hadoop的Configuration，详见：(https://blog.csdn.net/a909301740/article/details/85334980)：

	def getOrCreate(checkpointPath: String, creatingFunc: () => StreamingContext, hadoopConf: Configuration, createOnError: Boolean): StreamingContext

78. kafka创建topic时报错：org.apache.zookeeper.KeeperException$NoNodeException: KeeperErrorCode = NoNode for /brokers/ids

	原因：需要先启动kafka，再创建topic
	bin/kafka-server-start.sh config/server.properties &

79. 
	启动服务：systemctl start service_name
	关闭服务：systemctl stop service_name
	开机自启服务：systemctl enable service_name
	开机不自启服务：systemctl disable service_name

80. Linux切换用户：su 不更换home目录，不更改环境变量；su - 更换home目录，更改环境变量

81. 在RedHat/CentOS/Fedora上安装httpd服务的方式为：yum install httpd
	在Ubuntu/Debian 上安装apache2(相当于上面的httpd)服务的方式为apt-get install apache2

82. 使用 cat 命令查看文件时，若文件过大一般后面会跟一个管道符号+more来分屏显示：hdfs dfs -cat /output/data.txt | more

83. Flume source 类型:
	* Avro Source
	* Exec Source
	* Spooling Directory
	* Tairdir
	* Kafka Source

84. 同一个SparkContext对象可以用来创建多个StreamingContext对象，但是后一个StreamingContext对象必须等前一个StreamingContext对象停止才可以创建。

85. 根据物流单号导出订单的时候导出失败，原因：那一批订单里面有一个售后订单，同一个订单可能产生多个售后订单，为了区分这些订单再这些订单的后面追加"_1,_2,_3,..."来区分，结果导致导出订单时，无法识别"_"符号报错，将"_"替换成字母X解决

86. 设置Flume(pull模式)的sink类型为 a1.sinks.k1.type = org.apache.spark.streaming.flume.sink.SparkSink 时，sink的hostname要设置为云服务器的内网ip,不要写127.0.0.1，否则外部的SparkStreaming程序无法与之建立连接

87. 通过Flume的pull模式集成SparkStreaming时，flume的lib目录下要把spark的所有jar包，以及spark-streaming-flume-sink jar包添加上

88. kafka的高层次消费者API的优点是使用方便，不需要处理偏移量；简单消费者API的优点是可以自己控制偏移量。

89. 启动hive之前先启动HDFS。

90. 启动kafka之前先启动zookeeper。

91. 启动hdfs发现没有namenode的一个原因：没有格式化namenode：hdfs namenode -format

92. 通过 jdbc 操作hive，所需的 jar 包如下：

	```
	<dependency>
		<groupId>org.apache.hive</groupId>
		<artifactId>hive-jdbc</artifactId>
		<version>2.3.0</version>
	</dependency>
	```

93. 编写 hive 的 UDF，所需的 jar 包如下：

	```
	<dependency>
		<groupId>org.apache.hive</groupId>
		<artifactId>hive-exec</artifactId>
		<version>2.3.0</version>
	</dependency>
	```

94. mybatis 在使用 if，when 这样的判断标签时，里面的字符串需要调用.toString()方法，否则会报诡异的NumberFormatException

	```xml
	<if test="deliveryNumber != ''.toString()">
		<choose>
			<when test="deliveryNumber == '空'.toString()">
				AND A.delivery_number IS NULL OR A.delivery_number = ''
			</when>
			<otherwise>
				AND A.delivery_number LIKE CONCAT('%', #{deliveryNumber}, '%')
			</otherwise>
		</choose>
	</if>
	```





