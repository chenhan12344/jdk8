# Spring AOP

## 1.什么是 AOP?

在传统的OOP思想中，一切都是面向对象。与面向过程相比，OOP极大地简化了开发的难度，并且使得代码具有更高的组织性。虽然如此，但在OOP中仍会遇到许多问题，例如：在查询数据库的时候一般都需要进行参数的合法性校验，不同类型的参数校验方式可能存在差别。每创建一个方法我们就需要在方法中加入这种重复的代码，虽然可以单独将这部分代码抽离出来作为一个对象的方法，但是依旧不能解决需要在业务逻辑中加入参数校验逻辑的代码，使代码变得臃肿，无法专注于业务逻辑。并且如果校验的逻辑发生变化，可能需要修改所有调用了该校验方法的代码。使得代码之间耦合度高。

AOP的核心想就是将如日志记录、事务处理、性能统计等与业务逻辑无关的代码抽离出来，作为一个切面。在不改变源代码的前提下动态地去添加一些功能，降低了代码的耦合度，提高了代码的重用性，使得代码逻辑更加清晰，让我们能够更加专注于业务。

## 2.Spring AOP的实现方式和原理？

Spring AOP是通过动态代理的方式实现的，如果一个类继承自接口，则Spring会使用JDK动态代理，否则会使用CGLIB动态代理。Spring使用哪一种代理方式是由```DefaultAopProxyFactory```决定的，其代码如下：

```java
public class DefaultAopProxyFactory implements AopProxyFactory, Serializable {

	@Override
	public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
        //config.isOptimize()表示对代理类的生成是否使用策略优化
        //config.isProxyTargetClass()表示是否使用CGLIB的方式创建代理对象，默认为false
		if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
            // 获取目标类
			Class<?> targetClass = config.getTargetClass();
			if (targetClass == null) {
				throw new AopConfigException("TargetSource cannot determine target class: " +
						"Either an interface or a target is required for proxy creation.");
			}
            // 如果目标类是接口或者本身就是一个代理类，则使用JDK动态代理创建代理类
			if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
				return new JdkDynamicAopProxy(config);
			}
            // 如果目标不是接口且不是代理类则使用CGLIB动态代理创建代理类
			return new ObjenesisCglibAopProxy(config);
		}
		else {
            // 若目标类没拥有接口，则使用JDK动态代理创建代理类
			return new JdkDynamicAopProxy(config);
		}
	}
    
	// 用于检测目标类是否存在接口，如果有是否属于SpringProxy的类型
	private boolean hasNoUserSuppliedProxyInterfaces(AdvisedSupport config) {
		Class<?>[] ifcs = config.getProxiedInterfaces();
		return (ifcs.length == 0 || (ifcs.length == 1 && SpringProxy.class.isAssignableFrom(ifcs[0])));
	}

}
```



### 2.1 什么是代理？

所谓代理就是通过一个代理对象来控制对委托对象的访问。比如厂家销售商品，没有代理的时候，顾客只能直接去找厂家购买，而且不同厂家的产品质量可能不一，客户需要自行筛选。而厂家有时希望对不同的客户出售不同的商品，那么对客户进行分类的工作需要厂家自己来做。如果引入一个代理，即经销商，那么厂家只需要负责向经销商供货即可，而客户只需要到经销商购买商品，而不用直接跑去寻找厂家。经销商自己就可以负责对产品质量和客户群体进行筛选，而对客户和厂家产生影响。代理也就是AOP的一个最基本的雏形。

### 2.2 代理好处？

- 可以隐藏委托类的实现
- 可以实现客户和委托类的解耦，在不修改委托类代码的情况下做一些额外的处理

### 2.3 代理实现的方式？（需要弄清楚动态代理的好处，为什么要使用动态代理，而非静态代理）

#### 2.3.1 静态代理

> 静态代理是指代理类在程序运行前就已经存在，一般的做法就是对需要被代理的对象进行一次封装，内部持有一个委托类的引用。
>
> - 优点：实现简单，直接对被代理类的功能进行扩展，不入侵源代码。
> - 缺点：需要为每一个委托类都创建一个代理类，而且如果委托类增加或者减少了功能，那么代理类也需要相应的进行增加或者删除，增加了代码的维护成本。

#### 2.3.2 动态代理

> 动态代理则是在程序运行时动态地根据委托对象来创建代理对象，从而实现对目标对象的代理功能。动态代理主要有JDK动态代理和CGLIB动态代理两种方式。
>
> -  JDK动态代理：
>
> 使用JDK动态代理是通过反射来实现的主要涉：```java.lang.reflect.Proxy```和```java.lang.reflect.InvocationHandler```。其中创建代理对象是通过```Proxy.newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)```来实现的。该方法需要一个对```InvocationHandler```接口的实现，其代码如下：
>
> ```java
> public interface InvocationHandler {
>       /**
>      * @param proxy		目标对象
>      * @param method	代理对象被调用的方法
>      * @param args		代理对象被调用的方法的参数列表
>         */
> 	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable;
> }
> ```
>
> 通过实现上述接口，当代理对象的方法被调用时，就能够获取被调用的方法以及参数，然后在被代理对象的方法执行前后增加一些与业务逻辑无关的功能，如：日志记录、性能监控等。
>
> JDK动态代理的缺陷：使用JDK动态代理要求被代理的类需要实现特定接口，如果该类没有通过接口实现业务，那么也就无法通过JDK进行动态代理。
>
> - CGLIB动态代理：
>
> CGLIB(Code Generation Library)是一个第三方的代码生成库，其创建代理对象的方法是：运行时在内存中动态生成一个子类对象从而实现对目标对象功能的扩展，主要涉及```net.sf.cglib.proxy.MethodInterceptor```和```net.sf.cglib.proxy.Enhancer```。其中 ```MethodInterceptor```的代码如下：
>
> ```java
> public interface MethodInterceptor extends Callback {
>     
>       /**
>      * 拦截方法，用于在目标对象的方法调用的前后增加额外的功能
>         *
>      * @param proxy		目标对象
>      * @param method	代理对象被调用的方法
>      * @param args		代理对象被调用的方法的参数列表
>      * @param proxy		代理对象
>         */
>     public Object intercept(Object proxy, Method method, Object[] args, MethodProxy proxy)
>         throws Throwable;
> }
> ```
>
> 代理的创建是通过创建```Enhancer```的实例，然后通过```setSuperClass```方法将目标类设置为自己的父类，并通过```setCallback```来设置方法调用时的回调，为目标对象方法调用前后增加额外的功能。
>
> CGLIB动态代理的缺陷：相比于JDK动态代理，CGLIB是通过为目标类动态创建子类来实现的，解决了JDK动态代理要求目标类需要通过接口实现业务的局限性。但同时也导致了CGLIB无法对```final```修饰的类进行代理，因为这些类不能够被继承。

三种代理方式的比较：

- 静态代理：通过在代码中显示地定义一个代理类，对目标类进行包装。客户则是调用代理类中被包装过的方法。
- JDK动态代理：通过目标类的接口中的方法名来动态创建具有同名方法的代理类。
- CGLIB动态代理：通过继承目标类来动态创建代理类，然后重写业务方法进行代理。

### 2.4 AOP相关术语

- Aspect（切面）：

  

- Advice（通知/增强）：

- Join Point（连接点）：

- Point Cut（切入点）：

- Weaving（织入）：

