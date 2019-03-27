### 1、为什么mybatis的mapper没有实现类(原理探究)
	https://blog.csdn.net/puhaiyang/article/details/77418012
	因为 它采用了：Java动态代理实现接口

### 2、Mybatis中Mapper映射文件详解	
	映射文件是以<mapper>作为根节点，在根节点中支持9个元素，分别为
	insert、update、delete、select（增删改查）;
	cache、cache-ref、resultMap、parameterMap、sql。


### 3、Spring加载流程
	1.监听器加载spring
	2.加载配置文件
	3.工厂生产实例化对象
	4.放入ServletContext


### 5、Spring事务的传播属性 （propagation）
	1) REQUIRED--支持当前事务，如果当前没有事务，就新建一个事务，默认属性 
	2）SUPPORTS--支持当前事务，如果当前没有事务，就以非事务方式执行。
	3）MANDATORY--支持当前事务，如果当前没有事务，就抛出异常。
	4）REQUIRES_NEW--新建事务，如果当前存在事务，把当前事务挂起。
	5）NOT_SUPPORTED--以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
	6）NEVER--以非事务方式执行，如果当前存在事务，则抛出异常。
	7）NESTED -- 新建事务，与父事务一块提交

	理解Nested的关键是savepoint。他与PROPAGATION.REQUIRES_NEW的区别是，PROPAGATION_REQUIRES_NEW另起一个事务，将会与他的父事务相互独立，而Nested的事务和他的父事务是相依的，他的提交是要等和他的父事务一块提交的。也就是说，如果父事务最后回滚，他也要回滚的。而Nested事务的好处是他有一个savepoint。
	
### 6、Spring如何管理事务的
	Spring事务管理主要包括3个接口
	1）PlatformTransactionManager：事务管理器，Spring事务管理的核心接口，用于平台相关事务的管理，包括commit 事务的提交；rollback 事务的回滚；getTransaction 事务状态的获取三种方法
	2）TransactionDefinition：事务定义信息，是用来定义事务相关的属性，给事务管理器PlatformTransactionManager使用的
	3）TransactionStatus：事务具体运行状态，

### 7、Spring怎么配置事务
	1.基于XML的事务配置。2.基于注解方式的事务配置。

### 8、Springmvc spring 区别
	springmvc是表现层框架，核心是DispatcherServlet控制器，HandlerMapper负责解析路径、接受参数，封装数据，HandlerAdapter负责适配后端控制器
	spring是业务层框架，沟通表现层和持久层，主要作用是控制反转和切面编程

	