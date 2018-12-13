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
