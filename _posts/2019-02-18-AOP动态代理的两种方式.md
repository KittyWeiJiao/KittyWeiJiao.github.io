### 1、Spring AOP的实现原理
	在运行时，动态地将代码切入到类的指定方法、指定位置上的编程思想就是面向切面的编程

	动态代理：JDK、cglib

	AOP 代理其实是由 AOP 框架动态生成的一个对象，该对象可作为目标对象使用。
	AOP 代理包含了目标对象的全部方法，但 AOP 代理中的方法与目标对象的方法存在差异：AOP 方法在特定切入点添加了增强处理，并回调了目标对象的方法。

### 2、描述动态代理的几种实现方式，分别说出相应的优缺点

	AOP可以算作是代理模式的一个典型应用

	1、jdk动态代理
		实现原理：java内部反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理
		应用前提：目标类要实现接口
	2、cglib动态代理（字节码）
		底层使用ASM开源包，对代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。
		即使代理类没有实现任何接口也可以实现动态代理功能

优缺点

	1.JDK动态代理是实现了被代理对象的接口，Cglib是继承了被代理对象。
	2.JDK和Cglib都是在运行期生成字节码，JDK是直接写Class字节码，Cglib使用ASM框架写Class字节码，Cglib代理实现更复杂，生成代理类比JDK效率低。
	3.JDK调用代理方法，是通过反射机制调用，Cglib是通过FastClass机制直接调用方法，Cglib执行效率更高。

	CGLIB的运行速度要远远快于JDK的Proxy动态代理：
	但是CGLib在创建代理对象时所花费的时间却比JDK多得多
	对于单例的对象，因为无需频繁创建对象，用CGLib合适，反之，使用JDK方式要更为合适一些。
	同时，由于CGLib由于是采用动态创建子类的方法，对于final方法，无法进行代理

### 3、为什么CGlib方式可以对接口实现代理。
	
	CGLIB是通过生成java 字节码从而动态的产生代理对象
	CGLib代理的原理是通过二进制流生成一个动态class，该class继承被代理类以实现动态代理。那么就提供了一个代理无实现类的接口的可能

	// claxx: 被代理类或接口的Class文件
	public Object getInstance(Class claxx) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(claxx);
        // 回调方法
        enhancer.setCallback(this);
        // 创建代理对象
        return enhancer.create();
    }

### 4、实例
##### JDK动态代理

	public interface UserService {
	    public String getName(int id);
	
	    public Integer getAge(int id);
	}

	public class UserServiceImpl implements UserService {
	    @Override
	    public String getName(int name) {
	        System.out.println("-----getName------" + name);
	        return "kitty";
	    }
	
	    @Override
	    public Integer getAge(int age) {
	        System.out.println("-----getAge------" + age);
	        return 20;
	    }
	}

	public class MyInvocationHandler implements InvocationHandler {
	    private Object target;
	
	    public MyInvocationHandler(Object target) {
	        this.target = target;
	    }
	
	    public MyInvocationHandler() {
	    }
	
	    @Override
	    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
	        if("getName".equals(method.getName())){
	            System.out.println("++++++before " + method.getName() + "++++++");
	            Object result = method.invoke(target, args);
	            System.out.println("++++++after " + method.getName() + "++++++");
	            return result;
	        }else {
	            Object result = method.invoke(target, args);
	            return result;
	        }
	    }
	}

	public static void main(String[] args) {
        UserService userService = new UserServiceImpl();
        InvocationHandler invocationHandler = new MyInvocationHandler(userService);
        UserService userServiceProxy = (UserService) Proxy.newProxyInstance(
                userService.getClass().getClassLoader(), 
                userService.getClass().getInterfaces(), 
                invocationHandler);
        System.out.println(userServiceProxy.getName(5));
        System.out.println(userServiceProxy.getAge(1));
    }


##### cglib动态代理

	public class CglibProxy implements MethodInterceptor {
	    private Object target;//需要代理的目标对象
	
	    @Override
	    public Object intercept(Object o, Method method, Object[] arr, MethodProxy methodProxy) throws Throwable {
	        return method.invoke(target, arr);
	    }
	
	    //定义获取代理对象方法
	    public Object getCglibProxy(Object objectTarget){
	        //为目标对象target赋值
	        this.target = objectTarget;
	        Enhancer enhancer = new Enhancer();
	        //设置父类,因为Cglib是针对指定的类生成一个子类，所以需要指定父类
	        enhancer.setSuperclass(objectTarget.getClass());
	        enhancer.setCallback(this);// 设置回调
	        Object result = enhancer.create();//创建并返回代理对象
	        return result;
	    }
	}


### 5、aop:aspect与aop:advisor的区别

##### <aop:advisor>的实现方式  （需要实现Advice接口）  大多用于事务管理。 

	// 1.定义通知
	public class SleepHelper implements MethodBeforeAdvice,AfterReturningAdvice{
	    @Override
	    public void before(Method arg0, Object[] arg1, Object arg2)
	            throws Throwable {
	        System.out.println("睡觉前要脱衣服！");
	    }
	 
	    @Override
	    public void afterReturning(Object arg0, Method arg1, Object[] arg2,
	            Object arg3) throws Throwable {
	        System.out.println("起床后要穿衣服！");
	    }
	}

	// 2.aop配置
	<bean id="sleepHelper" class="com.ghs.aop.SleepHelper"></bean>
 
	<aop:config>
	    <aop:pointcut expression="execution(* *.sleep(..))" id="sleepPointcut"/>
	    <aop:advisor advice-ref="sleepHelper" pointcut-ref="sleepPointcut"/>
	</aop:config>

	
##### <aop:aspect>的实现方式    大多用于日志，缓存
  
	//1.定义切面
	public class SleepHelperAspect{
	    public void beforeSleep(){
	        System.out.println("睡觉前要脱衣服！");
	    }
	 
	    public void afterSleep(){
	        System.out.println("起床后要穿衣服！");
	    }
	}

	
	//2.aop配置
	<bean id="sleepHelperAspect" class="com.ghs.aop.SleepHelperAspect"></bean>
	 
	<aop:config>
	    <aop:pointcut expression="execution(* *.sleep(..))" id="sleepPointcut"/>
	    <aop:aspect ref="sleepHelperAspect">
	        <!--前置通知-->
	        <aop:before method="beforeSleep" pointcut-ref="sleepPointcut"/>
	        <!--后置通知-->
	        <aop:after method="afterSleep" pointcut-ref="sleepPointcut"/>
	    </aop:aspect>
	</aop:config>
