##Java反射
#反射基础

	/**
 	* @Auther: makui
 	* @Date: 2018/12/11
 	* @Description: jdk 反射
 	*/
 	public class ReflectDemo {

    public static void main(String[] args) throws InstantiationException,IllegalAccessException, IllegalArgumentException,
            InvocationTargetException,NoSuchMethodException {
        Class c;
        try {
            c = Class.forName("com.mk.coffee.test.jdk.annotation.Nursery");// 会有一个ClassNotFoundException异常
        } catch (ClassNotFoundException e) {
        }
        Class nursery = Nursery.class;   // 任何一个类都有一个隐含的静态成员变量class（知道类名时用）
        Class nursery1 = new Nursery().getClass();
        Nursery n = new Nursery();
        System.out.println(nursery == nursery1); //true 同一个类
        System.out.println(nursery.newInstance() == nursery1.newInstance()); //false 创建由此类对象表示的类的新实例 非单例
        Method  method= Nursery.class.getMethod("create",int.class,String.class,String.class); //返回一个方法对象 第一个参数为方法名 之后的参数为参数类型
        Nursery nursery2 = (Nursery)method.invoke(n,3,"xiaomage","1992-02-22");//在具有指定参数的 方法对象上调用此方法对象表示的底层方法
        //invoke方法 在动态代理中会经常用到
        System.out.println(nursery2);
        Method[] methods = Nursery.class.getMethods(); //public方法数组
        Method[] allMethods = Nursery.class.getDeclaredMethods(); //所有的方法数组
        Field[] fields = Nursery.class.getFields(); //public属性数组
        Field[] allFields = Nursery.class.getDeclaredFields(); //所有的属性数组
        for(Field field : allFields) {
            Validate validate = field.getAnnotation(Validate.class);//字段上是否有此注解
            String name = field.getName();//字段名称
            //.......可以进行一些字段上的校验
        }
        boolean isPrimitive = Nursery.class.isPrimitive(); //判断是否为基本类型
        String name = Nursery.class.getName(); //返回基础类的全名
        String simpleName = Nursery.class.getSimpleName();//返回源代码中给出的基础类的简单名称。
        Validate validate = Nursery.class.getAnnotation(Validate.class); //返回该元素的，如果有这样的注释 ，否则返回null指定类型的注释。
      //  System.out.println(validate.toString());
        Annotation[] annotations = Nursery.class.getAnnotations();//返回此元素上 存在的注释。
        for(Annotation annotation : annotations) {
            System.out.println(annotation.toString());
        }
        ClassLoader classLoader = Nursery.class.getClassLoader(); //获取类的加载器
        boolean isAnnotation = Nursery.class.isAnnotationPresent(Validate.class); //如果此元素上 存在指定类型的注释，则返回true，否则返回false
        AnnotatedType[] annotatedTypes = Nursery.class.getAnnotatedInterfaces(); //返回一个 AnnotatedType对象的数组， AnnotatedType使用类型指定由此 AnnotatedType对象表示的实体的超级 类 。
        Type[] types = Nursery.class.getGenericInterfaces();//返回 Type表示通过由该对象所表示的类或接口直接实现的接口。
        Class<?>[] classes = Nursery.class.getInterfaces();//确定由该对象表示的类或接口实现的接口
        Nursery.class.getGenericSuperclass();//返回 Type表示此所表示的实体（类，接口，基本类型或void）的直接超类。

    }
	}

#JDK 动态代理

	import java.math.BigDecimal;
	
	/**
	* @Auther: makui
	* @Date: 2018/12/11
	* @Description: 接口 房东
	*/
	public interface Fangdong {
	
	/**
	* 出租
	* @param money 钱
	*/
	void rent(BigDecimal money);
	}

	import java.math.BigDecimal;
	
	/**
	 * @Auther: makui
	 * @Date: 2018/12/11
	 * @Description: 中介 实现房东接口
	 */
	public class ZhongJie implements Fangdong {
	    @Override
	    public void rent(BigDecimal money) {
	        System.out.println("我在帮房东卖房，此房" + money +"万元");
	    }
	}


	import java.lang.reflect.InvocationHandler;
	import java.lang.reflect.Method;
	import java.lang.reflect.Proxy;
	import java.math.BigDecimal;
	
	/**
	 * @Auther: makui
	 * @Date: 2018/12/11
	 * @Description: JDK 动态代理
	 * 使用JDK动态代理有一个很大的限制，就是它要求目标类必须实现了对应方法的接口，它只能为接口创建代理实例。
	 * 我们在上文测试类中的Proxy的newProxyInstance方法中可以看到，
	 * 该方法第二个参数便是目标类的接口。如果该类没有实现接口，这就要靠cglib动态代理了。
	 * Java动态代理只能代理接口，要代理类需要使用第三方的CLIGB等类库。
	 */
	public class JDK_Proxy implements InvocationHandler {
	
	    private Object target;
	    JDK_Proxy(Object target) {
	        this.target = target;
	    }
	
	    @Override
	    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
	        System.out.println("调用前=====");
	        method.invoke(target,args);
	        System.out.println("调用后=====");
	        return null;
	    }
	
	    public static void main(String[] args) {
	        Fangdong fangdong = new ZhongJie();
	        JDK_Proxy jdk_proxy = new JDK_Proxy(fangdong);
	        Fangdong fangdong1 = (Fangdong)Proxy.newProxyInstance(fangdong.getClass().getClassLoader(),fangdong.getClass().getInterfaces(),jdk_proxy);
	        fangdong1.rent(BigDecimal.valueOf(300));         
			//输出以下内容：调用前===== 我在帮房东卖房，此房300万元 调用后=====
	    }
	
	}

**注意事项：**  JDK动态代理的代理对象在创建时，需要使用业务实现类所实现的接口作为参数（因为在后面代理方法时需要根据接口内的方法名进行调用）。如果业务实现类是没有实现接口而是直接定义业务方法的话，就无法使用JDK动态代理了。并且，如果业务实现类中新增了接口中没有的方法，这些方法是无法被代理的（因为无法被调用）。

#剖析JDK动态代理
返回指定接口的代理类的实例，该接口将方法调用分派给指定的调用处理程序。

	public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
        throws IllegalArgumentException
	{
        Objects.requireNonNull(h); //判断invocationHandler是否为空 为空抛出空指针异常

        final Class<?>[] intfs = interfaces.clone();  // 克隆所有的接口。clone:创建并返回此对象的副本。 “复制”的精确含义可能取决于对象的类。 一般的意图是，对于任何对象x ，表达式： x.clone() != x将是真实的
        final SecurityManager sm = System.getSecurityManager(); //获得Java安全管理器
        if (sm != null) { //如果java安全管理器不为null 
            checkProxyAccess(Reflection.getCallerClass(), loader, intfs); //**知识点1**
        }

        /*
         * Look up or generate the designated proxy class.
         */
 		//产生代理类 ***知识点2***
        Class<?> cl = getProxyClass0(loader, intfs);

        /*
         * Invoke its constructor with the designated invocation handler.
         */
        try {
            if (sm != null) {
				//检测新的代理类的代理权限
                checkNewProxyPermission(Reflection.getCallerClass(), cl);
            }
			
			//得到代理类对象的构造函数，这个构造函数的参数由constructorParams指定
			// //参数constructorParames为常量值：private static final Class<?>[] constructorParams = { InvocationHandler.class };

            final Constructor<?> cons = cl.getConstructor(constructorParams);
            final InvocationHandler ih = h;
            if (!Modifier.isPublic(cl.getModifiers())) { //返回此类或接口的Java语言修饰符，以整数编码。判断是否为public
				//根据当前有效的安全策略来决定是否允许或拒绝对关键系统资源的访问， 
                AccessController.doPrivileged(new PrivilegedAction<Void>() { 
                    public Void run() {
                        cons.setAccessible(true);
                        return null;
                    }
                });
            }
			//这里生成代理对象，传入的参数new Object[]{h}后面讲
			// 使用此 Constructor对象表示的构造函数，使用指定的初始化参数来创建和初始化构造函数的声明类的新实例。 
            return cons.newInstance(new Object[]{h});
        } catch (IllegalAccessException|InstantiationException e) {
            throw new InternalError(e.toString(), e);
        } catch (InvocationTargetException e) {
            Throwable t = e.getCause();
            if (t instanceof RuntimeException) {
                throw (RuntimeException) t;
            } else {
                throw new InternalError(t.toString(), t);
            }
        } catch (NoSuchMethodException e) {
            throw new InternalError(e.toString(), e);
        }
    }

 
点1： 校验加载器权限

	private static void checkProxyAccess(Class<?> caller,
                                         ClassLoader loader,
                                         Class<?>... interfaces)
    {
        SecurityManager sm = System.getSecurityManager(); 
        if (sm != null) {
            ClassLoader ccl = caller.getClassLoader(); //类加载器
			//加载器都不为null，校验权限
            if (VM.isSystemDomainLoader(loader) && !VM.isSystemDomainLoader(ccl)) {
                sm.checkPermission(SecurityConstants.GET_CLASSLOADER_PERMISSION);
            }
            ReflectUtil.checkProxyPackageAccess(ccl, interfaces);
        }
    }

点2：产生代理类  查询（在缓存中已经有）或生成指定的代理类的class对象。

		private static Class<?> getProxyClass0(ClassLoader loader,
                                           Class<?>... interfaces) {
		//接口数不得超过65535个,否则抛异常
        if (interfaces.length > 65535) {
            throw new IllegalArgumentException("interface limit exceeded");
        }

        // If the proxy class defined by the given loader implementing
        // the given interfaces exists, this will simply return the cached copy;
        // otherwise, it will create the proxy class via the ProxyClassFactory
		//***知识点3、5****代理类缓存，如果缓存中有代理类了直接返回，否则将由ProxyClassFactory创建代理类
        return proxyClassCache.get(loader, interfaces);
    }

点3：代理类缓存，如果缓存中有代理类了直接返回，否则将由ProxyClassFactory创建代理类
  Proxy类中的 静态的成员变量
 		
		private static final WeakCache<ClassLoader, Class<?>[], Class<?>> proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory());
		  //意思是：如果代理类被指定的类加载器loader定义了，并实现了给定的接口interfaces，
       //那么就返回缓存的代理类对象，否则使用ProxyClassFactory创建代理类。
		//***知识点4***
		proxyClassCache.get(loader, interfaces);

点4：WeakCache 缓存
	//K代表key的类型，P代表参数的类型，V代表value的类型。
	// WeakCache<ClassLoader, Class<?>[], Class<?>>  proxyClassCache  说明proxyClassCache存的值是Class<?>对象，正是我们需要的代理类对象。
		
	final class WeakCache<K, P, V> {
	
	   private final ReferenceQueue<K> refQueue
	       = new ReferenceQueue<>();
	   // the key type is Object for supporting null key
	   private final ConcurrentMap<Object, ConcurrentMap<Object, Supplier<V>>> map
	       = new ConcurrentHashMap<>();
	   private final ConcurrentMap<Supplier<V>, Boolean> reverseMap
	       = new ConcurrentHashMap<>();
	   private final BiFunction<K, P, ?> subKeyFactory;
	   private final BiFunction<K, P, V> valueFactory;
	
	 
	   public WeakCache(BiFunction<K, P, ?> subKeyFactory,
	                    BiFunction<K, P, V> valueFactory) {
	       this.subKeyFactory = Objects.requireNonNull(subKeyFactory);
	       this.valueFactory = Objects.requireNonNull(valueFactory);
	   }

	}
其中map变量是实现缓存的核心变量，他是一个双重的Map结构: (key, sub-key) -> value。其中key是传进来的Classloader进行包装的
对象，sub-key是由WeakCache构造函数传人的KeyFactory()生成的。value就是产生代理类的对象，
是由WeakCache构造函数传人的ProxyClassFactory()生成的。 如下：

	private static final WeakCache<ClassLoader, Class<?>[], Class<?>> proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory());
	
产生sub-key的KeyFactory代码如下，这个我们不去深究，只要知道他是根据传入的ClassLoader和接口类生成sub-key即可。
	
	private static final class KeyFactory
	       implements BiFunction<ClassLoader, Class<?>[], Object>
	   	{
	       @Override
	       public Object apply(ClassLoader classLoader, Class<?>[] interfaces) {
	           switch (interfaces.length) {
	               case 1: return new Key1(interfaces[0]); // the most frequent
	               case 2: return new Key2(interfaces[0], interfaces[1]);  
	               case 0: return key0;
	               default: return new KeyX(interfaces);
	           }
	       }
	   }
	
通过sub-key拿到一个Supplier<Class<?>>对象，然后调用这个对象的get方法，最终得到代理类的Class对象。


知识点5： proxyClassCache.get(loader, interfaces);
	
	//K和P就是WeakCache定义中的泛型，key是类加载器，parameter是接口类数组
	public V get(K key, P parameter) {
	       //检查parameter不为空
	       Objects.requireNonNull(parameter);
	        //清除无效的缓存
	       expungeStaleEntries();
	       // cacheKey就是(key, sub-key) -> value里的一级key，
	       Object cacheKey = CacheKey.valueOf(key, refQueue);
	
	       // lazily install the 2nd level valuesMap for the particular cacheKey
	       //根据一级key得到 ConcurrentMap<Object, Supplier<V>>对象。如果之前不存在，则新建一个ConcurrentMap<Object, Supplier<V>>和cacheKey（一级key）一起放到map中。
	        ConcurrentMap<Object, Supplier<V>> valuesMap = map.get(cacheKey);
	       if (valuesMap == null) {
	           ConcurrentMap<Object, Supplier<V>> oldValuesMap
	               = map.putIfAbsent(cacheKey,
	                                 valuesMap = new ConcurrentHashMap<>());
	           if (oldValuesMap != null) {
	               valuesMap = oldValuesMap;
	           }
	       }
	
	       // create subKey and retrieve the possible Supplier<V> stored by that
	       // subKey from valuesMap
	       //这部分就是调用生成sub-key的代码，上面我们已经看过怎么生成的了
	       Object subKey = Objects.requireNonNull(subKeyFactory.apply(key, parameter));
	       //通过sub-key得到supplier
	       Supplier<V> supplier = valuesMap.get(subKey);
	       //supplier实际上就是这个factory
	       Factory factory = null;
	
	       while (true) {
	           //如果缓存里有supplier ，那就直接通过get方法，得到代理类对象，返回，就结束了，一会儿分析get方法。
	            if (supplier != null) {
	               // supplier might be a Factory or a CacheValue<V> instance
	               V value = supplier.get();
	               if (value != null) {
	                   return value;
	               }
	           }
	           // else no supplier in cache
	           // or a supplier that returned null (could be a cleared CacheValue
	           // or a Factory that wasn't successful in installing the CacheValue)
	           // lazily construct a Factory
	           //下面的所有代码目的就是：如果缓存中没有supplier，则创建一个Factory对象，把factory对象在多线程的环境下安全的赋给supplier。
	            //因为是在while（true）中，赋值成功后又回到上面去调get方法，返回才结束。
	           if (factory == null) {
	               factory = new Factory(key, parameter, subKey, valuesMap);
	           }
	
	           if (supplier == null) {
	               supplier = valuesMap.putIfAbsent(subKey, factory);
	               if (supplier == null) {
	                   // successfully installed Factory
	                   supplier = factory;
	               }
	               // else retry with winning supplier
	           } else {
	               if (valuesMap.replace(subKey, supplier, factory)) {
	                   // successfully replaced
	                   // cleared CacheEntry / unsuccessful Factory
	                   // with our Factory
	                   supplier = factory;
	               } else {
	                   // retry with current supplier
						//***知识点6*** factory.get(key)
	                   supplier = valuesMap.get(subKey);
	               }
	           }
	       }
	   }

	
知识点6: Factory类中的get方法。

	public synchronized V get() { // serialize access
	    // re-check
	    Supplier<V> supplier = valuesMap.get(subKey);
	    /重新检查得到的supplier是不是当前对象
	           if (supplier != this) {
	               // something changed while we were waiting:
	               // might be that we were replaced by a CacheValue
	               // or were removed because of failure ->
	               // return null to signal WeakCache.get() to retry
	               // the loop
	               return null;
	           }
	           // else still us (supplier == this)
	 
	           // create new value
	           V value = null;
	           try {
	                //代理类就是在这个位置调用valueFactory生成的
	                //valueFactory就是我们传入的 new ProxyClassFactory()
	               //一会我们分析ProxyClassFactory()的apply方法
					//知识点7 ： 来到ProxyClassFactory的apply方法，代理类就是在这里生成的。
	               value = Objects.requireNonNull(valueFactory.apply(key, parameter));
	           } finally {
	               if (value == null) { // remove us on failure
	                   valuesMap.remove(subKey, this);
	               }
	           }
	           // the only path to reach here is with non-null value
	           assert value != null;
	 
	           // wrap value with CacheValue (WeakReference)
	           //把value包装成弱引用
	           CacheValue<V> cacheValue = new CacheValue<>(value);
	 
	           // put into reverseMap
	           // reverseMap是用来实现缓存的有效性
	           reverseMap.put(cacheValue, Boolean.TRUE);
	 
	           // try replacing us with CacheValue (this should always succeed)
	           if (!valuesMap.replace(subKey, this, cacheValue)) {
	               throw new AssertionError("Should not reach here");
	           }
	 
	           // successfully replaced us with new CacheValue -> return the value
	           // wrapped by it
	           return value;
	       }
	   }


知识点7：来到ProxyClassFactory的apply方法，代理类就是在这里生成的。

	//这里的BiFunction<T, U, R>是个函数式接口，可以理解为用T，U两种类型做参数，得到R类型的返回值
	private static final class ProxyClassFactory
	       implements BiFunction<ClassLoader, Class<?>[], Class<?>>
	   {
	       // prefix for all proxy class names
	       //所有代理类名字的前缀
	       private static final String proxyClassNamePrefix = "$Proxy";
	       
	       // next number to use for generation of unique proxy class names
	       //用于生成代理类名字的计数器
	       private static final AtomicLong nextUniqueNumber = new AtomicLong();
	 
	       @Override
	       public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {
	             
	           Map<Class<?>, Boolean> interfaceSet = new IdentityHashMap<>(interfaces.length);
	            //验证代理接口，可不看
	           for (Class<?> intf : interfaces) {
	               /*
	                * Verify that the class loader resolves the name of this
	                * interface to the same Class object.
	                */
	               Class<?> interfaceClass = null;
	               try {
	                   interfaceClass = Class.forName(intf.getName(), false, loader);
	               } catch (ClassNotFoundException e) {
	               }
	               if (interfaceClass != intf) {
	                   throw new IllegalArgumentException(
	                       intf + " is not visible from class loader");
	               }
	               /*
	                * Verify that the Class object actually represents an
	                * interface.
	                */
	               if (!interfaceClass.isInterface()) {
	                   throw new IllegalArgumentException(
	                       interfaceClass.getName() + " is not an interface");
	               }
	               /*
	                * Verify that this interface is not a duplicate.
	                */
	               if (interfaceSet.put(interfaceClass, Boolean.TRUE) != null) {
	                   throw new IllegalArgumentException(
	                       "repeated interface: " + interfaceClass.getName());
	               }
	           }
	           //生成的代理类的包名 
	           String proxyPkg = null;     // package to define proxy class in
	           //代理类访问控制符: public ,final
	           int accessFlags = Modifier.PUBLIC | Modifier.FINAL;
	 
	           /*
	            * Record the package of a non-public proxy interface so that the
	            * proxy class will be defined in the same package.  Verify that
	            * all non-public proxy interfaces are in the same package.
	            */
	           //验证所有非公共的接口在同一个包内；公共的就无需处理
	           //生成包名和类名的逻辑，包名默认是com.sun.proxy，类名默认是$Proxy 加上一个自增的整数值
	            //如果被代理类是 non-public proxy interface ，则用和被代理类接口一样的包名
	           for (Class<?> intf : interfaces) {
	               int flags = intf.getModifiers();
	               if (!Modifier.isPublic(flags)) {
	                   accessFlags = Modifier.FINAL;
	                   String name = intf.getName();
	                   int n = name.lastIndexOf('.');
	                   String pkg = ((n == -1) ? "" : name.substring(0, n + 1));
	                   if (proxyPkg == null) {
	                       proxyPkg = pkg;
	                   } else if (!pkg.equals(proxyPkg)) {
	                       throw new IllegalArgumentException(
	                           "non-public interfaces from different packages");
	                   }
	               }
	           }
	 
	           if (proxyPkg == null) {
	               // if no non-public proxy interfaces, use com.sun.proxy package
	               proxyPkg = ReflectUtil.PROXY_PACKAGE + ".";
	           }
	 
	           /*
	            * Choose a name for the proxy class to generate.
	            */
	           long num = nextUniqueNumber.getAndIncrement();
	           //代理类的完全限定名，如com.sun.proxy.$Proxy0.calss
	           String proxyName = proxyPkg + proxyClassNamePrefix + num;
	 
	           /*
	            * Generate the specified proxy class.
	            */
	           //核心部分，生成代理类的字节码
	           byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
	               proxyName, interfaces, accessFlags);
	           try {
	               //把代理类加载到JVM中，至此动态代理过程基本结束了
	               return defineClass0(loader, proxyName,
	                                   proxyClassFile, 0, proxyClassFile.length);
	           } catch (ClassFormatError e) {
	               /*
	                * A ClassFormatError here means that (barring bugs in the
	                * proxy class generation code) there was some other
	                * invalid aspect of the arguments supplied to the proxy
	                * class creation (such as virtual machine limitations
	                * exceeded).
	                */
	               throw new IllegalArgumentException(e.toString());
	           }
	       }
	   }
	
#延伸：JDK生成的动态代理字节码是什么，于是我们将字节码保存到磁盘上的class文件中。

		public static void main(String[] args) {
		    Fangdong fangdong = new ZhongJie();
		    JDK_Proxy jdk_proxy = new JDK_Proxy(fangdong);
		    Fangdong fangdong1 = (Fangdong) Proxy.newProxyInstance(fangdong.getClass().getClassLoader(), fangdong.getClass().getInterfaces(), jdk_proxy);
		    fangdong1.rent(BigDecimal.valueOf(300));
		    //调用前=====
		    //我在帮房东卖房，此房300万元
		    //调用后=====
		    byte[] proxy = ProxyGenerator.generateProxyClass("com.mk.test.jdk.reflect.$ProxyFangDong", ZhongJie.class.getInterfaces());
		    FileOutputStream out = null;
		    String path = "C:/Users/dell/Desktop/$ProxyFangDong.class";
		    try {
		        out = new FileOutputStream(path);
		        out.write(proxy);
		        out.flush();
		    } catch (Exception e) {
		        e.printStackTrace();
		    } finally {
		        try {
		            out.close();
		        } catch (IOException e) {
		            e.printStackTrace();
		        }
		    }
		}

$ProxyFangDong.class 字节码文件如下：
		
		//放入idea反编译
		// Source code recreated from a .class file by IntelliJ IDEA
		// (powered by Fernflower decompiler)
		//
		
		package com.mk.test.jdk.reflect;
		
		import com.mk.coffee.test.jdk.reflect.Fangdong;
		import java.lang.reflect.InvocationHandler;
		import java.lang.reflect.Method;
		import java.lang.reflect.Proxy;
		import java.lang.reflect.UndeclaredThrowableException;
		import java.math.BigDecimal;
		
		public final class $ProxyFangDong extends Proxy implements Fangdong {
		    private static Method m1;
		    private static Method m2;
		    private static Method m3;
		    private static Method m0;
		
		    public $ProxyFangDong(InvocationHandler var1) throws  {
		        super(var1);
		    }
		
		    public final boolean equals(Object var1) throws  {
		        try {
		            return (Boolean)super.h.invoke(this, m1, new Object[]{var1});
		        } catch (RuntimeException | Error var3) {
		            throw var3;
		        } catch (Throwable var4) {
		            throw new UndeclaredThrowableException(var4);
		        }
		    }
		
		    public final String toString() throws  {
		        try {
		            return (String)super.h.invoke(this, m2, (Object[])null);
		        } catch (RuntimeException | Error var2) {
		            throw var2;
		        } catch (Throwable var3) {
		            throw new UndeclaredThrowableException(var3);
		        }
		    }
		
		    public final void rent(BigDecimal var1) throws  {
		        try {
		            super.h.invoke(this, m3, new Object[]{var1});
		        } catch (RuntimeException | Error var3) {
		            throw var3;
		        } catch (Throwable var4) {
		            throw new UndeclaredThrowableException(var4);
		        }
		    }
		
		    public final int hashCode() throws  {
		        try {
		            return (Integer)super.h.invoke(this, m0, (Object[])null);
		        } catch (RuntimeException | Error var2) {
		            throw var2;
		        } catch (Throwable var3) {
		            throw new UndeclaredThrowableException(var3);
		        }
		    }
		
		    static {
		        try {
		            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
		            m2 = Class.forName("java.lang.Object").getMethod("toString");
		            m3 = Class.forName("com.mk.coffee.test.jdk.reflect.Fangdong").getMethod("rent", Class.forName("java.math.BigDecimal"));
		            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
		        } catch (NoSuchMethodException var2) {
		            throw new NoSuchMethodError(var2.getMessage());
		        } catch (ClassNotFoundException var3) {
		            throw new NoClassDefFoundError(var3.getMessage());
		        }
		    }
		}

生成了Object类的三个方法：toString，hashCode，equals。还有我们需要被代理的方法 rent方法。


 
##


#CGLIB 动态代理 （必须导入 cglib jar包）
 
类 Car 有一个方法

	/**
	 * @Auther: makui
	 * @Date: 2018/12/13
	 * @Description: cglib 动态代理 代理类不是接口
	 */
	public class Car {
	    public void run(int speed) {
	        System.out.println("正在以时速"+speed +"行驶");
	    }
	
	}


具体代理的方法： CGLIB_Proxy类 必须实现MethodInterceptor接口 

	import org.springframework.cglib.proxy.Enhancer;
	import org.springframework.cglib.proxy.MethodInterceptor;
	import org.springframework.cglib.proxy.MethodProxy;
	
	import java.lang.reflect.Method;
	
	/**
	 * @Auther: makui
	 * @Date: 2018/12/13
	 * @Description: cglib 动态代理
	 */
	public class CGLIB_Proxy implements MethodInterceptor {
   		//代理类class文件存入本地磁盘
        System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, "C:/Users/dell/Desktop/");
	    //通过Enhancer 创建代理对象
		//知识点***1***
	    private Enhancer enhancer = new Enhancer();
	
	    //通过Class对象获取代理对象
	    public Object getProxy(Class c){
	        //设置创建子类的类
			//知识点***2***	
	        enhancer.setSuperclass(c);
			//知识点***3***	
	        enhancer.setCallback(this);
			//知识点***4***	
	        return enhancer.create();
	    }
		
		//知识点***5***	
	    @Override
	    public Object intercept(Object obj, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
	        System.out.println("日志开始...");
	        //代理类调用父类的方法
			//知识点***6***	
	        methodProxy.invokeSuper(obj, args);
	        System.out.println("日志结束...");
	        return null;
	    }
	
	    public static void main(String[] args) {
	
	        CGLIB_Proxy cglib_proxy = new CGLIB_Proxy();
	        Car car = (Car)cglib_proxy.getProxy(Car.class);
	        car.run(500);
	    }
	}


知识点1：Enhancer类 

知识点2：enhancer.setSuperclass(c)

	  public void setSuperclass(Class superclass) {
	        if (superclass != null && superclass.isInterface()) {
	            this.setInterfaces(new Class[]{superclass});
	        } else if (superclass != null && superclass.equals(Object.class)) {
	            this.superclass = null;
	        } else {
	            this.superclass = superclass;
	        }
	
	    }
	
知识点3： enhancer.setCallback(this);

	public void setCallback(Callback callback) {
        this.setCallbacks(new Callback[]{callback});
    }

    public void setCallbacks(Callback[] callbacks) {
        if (callbacks != null && callbacks.length == 0) {
            throw new IllegalArgumentException("Array cannot be empty");
        } else {
            this.callbacks = callbacks;
        }
    }

	
知识点4：  return enhancer.create();

	 public Object create() {
	        this.classOnly = false;
	        this.argumentTypes = null;
	        return this.createHelper();
	    }

	private Object createHelper() {
        this.preValidate();
        Object key = KEY_FACTORY.newInstance(this.superclass != null ? this.superclass.getName() : null, ReflectUtils.getNames(this.interfaces), this.filter == ALL_ZERO ? null : new WeakCacheKey(this.filter), this.callbackTypes, this.useFactory, this.interceptDuringConstruction, this.serialVersionUID);
        this.currentKey = key;
        Object result = super.create(key);
        return result;
    }


	    protected Object create(Object key) {
        try {
            ClassLoader loader = this.getClassLoader();
            Map<ClassLoader, AbstractClassGenerator.ClassLoaderData> cache = CACHE;
            AbstractClassGenerator.ClassLoaderData data = (AbstractClassGenerator.ClassLoaderData)cache.get(loader);
            if (data == null) {
                Class var5 = AbstractClassGenerator.class;
                synchronized(AbstractClassGenerator.class) {
                    cache = CACHE;
                    data = (AbstractClassGenerator.ClassLoaderData)cache.get(loader);
                    if (data == null) {
                        Map<ClassLoader, AbstractClassGenerator.ClassLoaderData> newCache = new WeakHashMap(cache);
                        data = new AbstractClassGenerator.ClassLoaderData(loader);
                        newCache.put(loader, data);
                        CACHE = newCache;
                    }
                }
            }

            this.key = key;
            Object obj = data.get(this, this.getUseCache());
            return obj instanceof Class ? this.firstInstance((Class)obj) : this.nextInstance(obj);
        } catch (RuntimeException var9) {
            throw var9;
        } catch (Error var10) {
            throw var10;
        } catch (Exception var11) {
            throw new CodeGenerationException(var11);
        }
    }


生成三个代理类：final修饰的方法，不会被子类继承 所以不会被代理类代理

Car$$EnhancerByCGLIB$$9359a76e$$FastClassByCGLIB$$60f1a780.class

Car$$EnhancerByCGLIB$$9359a76e.class

Car$$FastClassByCGLIB$$8eb0d649.class

	//Car$$EnhancerByCGLIB$$9359a76e.class 源码
	
	// Source code recreated from a .class file by IntelliJ IDEA
	// (powered by Fernflower decompiler)
	//
	
	package com.mk.coffee.test.jdk.reflect;
	
	import java.lang.reflect.Method;
	import org.springframework.cglib.core.ReflectUtils;
	import org.springframework.cglib.core.Signature;
	import org.springframework.cglib.proxy.Callback;
	import org.springframework.cglib.proxy.Factory;
	import org.springframework.cglib.proxy.MethodInterceptor;
	import org.springframework.cglib.proxy.MethodProxy;
	
	public class Car$$EnhancerByCGLIB$$9359a76e extends Car implements Factory {
    private boolean CGLIB$BOUND;
    public static Object CGLIB$FACTORY_DATA;
    private static final ThreadLocal CGLIB$THREAD_CALLBACKS;
    private static final Callback[] CGLIB$STATIC_CALLBACKS;
    private MethodInterceptor CGLIB$CALLBACK_0; //拦截器
    private static Object CGLIB$CALLBACK_FILTER;  
    private static final Method CGLIB$run$0$Method; //被代理方法
    private static final MethodProxy CGLIB$run$0$Proxy; //代理方法
    private static final Object[] CGLIB$emptyArgs;
    private static final Method CGLIB$equals$1$Method;
    private static final MethodProxy CGLIB$equals$1$Proxy;
    private static final Method CGLIB$toString$2$Method;
    private static final MethodProxy CGLIB$toString$2$Proxy;
    private static final Method CGLIB$hashCode$3$Method;
    private static final MethodProxy CGLIB$hashCode$3$Proxy;
    private static final Method CGLIB$clone$4$Method;
    private static final MethodProxy CGLIB$clone$4$Proxy;

    static void CGLIB$STATICHOOK1() {
        CGLIB$THREAD_CALLBACKS = new ThreadLocal();
        CGLIB$emptyArgs = new Object[0];
        Class var0 = Class.forName("com.mk.coffee.test.jdk.reflect.Car$$EnhancerByCGLIB$$9359a76e"); //代理类
        Class var1;  //被代理类
        Method[] var10000 = ReflectUtils.findMethods(new String[]{"equals", "(Ljava/lang/Object;)Z", "toString", "()Ljava/lang/String;", "hashCode", "()I", "clone", "()Ljava/lang/Object;"}, (var1 = Class.forName("java.lang.Object")).getDeclaredMethods());
        CGLIB$equals$1$Method = var10000[0];
        CGLIB$equals$1$Proxy = MethodProxy.create(var1, var0, "(Ljava/lang/Object;)Z", "equals", "CGLIB$equals$1");
        CGLIB$toString$2$Method = var10000[1];
        CGLIB$toString$2$Proxy = MethodProxy.create(var1, var0, "()Ljava/lang/String;", "toString", "CGLIB$toString$2");
        CGLIB$hashCode$3$Method = var10000[2];
        CGLIB$hashCode$3$Proxy = MethodProxy.create(var1, var0, "()I", "hashCode", "CGLIB$hashCode$3");
        CGLIB$clone$4$Method = var10000[3];
        CGLIB$clone$4$Proxy = MethodProxy.create(var1, var0, "()Ljava/lang/Object;", "clone", "CGLIB$clone$4");
        CGLIB$run$0$Method = ReflectUtils.findMethods(new String[]{"run", "(I)V"}, (var1 = Class.forName("com.mk.coffee.test.jdk.reflect.Car")).getDeclaredMethods())[0];
        CGLIB$run$0$Proxy = MethodProxy.create(var1, var0, "(I)V", "run", "CGLIB$run$0");
    }
	
	//代理方法
    final void CGLIB$run$0(int var1) {
        super.run(var1);
    }
	
	//被代理方法
    public final void run(int var1) {
        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
        if (this.CGLIB$CALLBACK_0 == null) {
            CGLIB$BIND_CALLBACKS(this);
            var10000 = this.CGLIB$CALLBACK_0;
        }

        if (var10000 != null) {
			// //调用拦截器
            var10000.intercept(this, CGLIB$run$0$Method, new Object[]{new Integer(var1)}, CGLIB$run$0$Proxy);
        } else {
            super.run(var1);
        }
    }

    final boolean CGLIB$equals$1(Object var1) {
        return super.equals(var1);
    }

    public final boolean equals(Object var1) {
        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
        if (this.CGLIB$CALLBACK_0 == null) {
            CGLIB$BIND_CALLBACKS(this);
            var10000 = this.CGLIB$CALLBACK_0;
        }

        if (var10000 != null) {
            Object var2 = var10000.intercept(this, CGLIB$equals$1$Method, new Object[]{var1}, CGLIB$equals$1$Proxy);
            return var2 == null ? false : (Boolean)var2;
        } else {
            return super.equals(var1);
        }
    }

    final String CGLIB$toString$2() {
        return super.toString();
    }

    public final String toString() {
        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
        if (this.CGLIB$CALLBACK_0 == null) {
            CGLIB$BIND_CALLBACKS(this);
            var10000 = this.CGLIB$CALLBACK_0;
        }

        return var10000 != null ? (String)var10000.intercept(this, CGLIB$toString$2$Method, CGLIB$emptyArgs, CGLIB$toString$2$Proxy) : super.toString();
    }

    final int CGLIB$hashCode$3() {
        return super.hashCode();
    }

    public final int hashCode() {
        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
        if (this.CGLIB$CALLBACK_0 == null) {
            CGLIB$BIND_CALLBACKS(this);
            var10000 = this.CGLIB$CALLBACK_0;
        }

        if (var10000 != null) {
            Object var1 = var10000.intercept(this, CGLIB$hashCode$3$Method, CGLIB$emptyArgs, CGLIB$hashCode$3$Proxy);
            return var1 == null ? 0 : ((Number)var1).intValue();
        } else {
            return super.hashCode();
        }
    }

    final Object CGLIB$clone$4() throws CloneNotSupportedException {
        return super.clone();
    }

    protected final Object clone() throws CloneNotSupportedException {
        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
        if (this.CGLIB$CALLBACK_0 == null) {
            CGLIB$BIND_CALLBACKS(this);
            var10000 = this.CGLIB$CALLBACK_0;
        }

        return var10000 != null ? var10000.intercept(this, CGLIB$clone$4$Method, CGLIB$emptyArgs, CGLIB$clone$4$Proxy) : super.clone();
    }

    public static MethodProxy CGLIB$findMethodProxy(Signature var0) {
        String var10000 = var0.toString();
        switch(var10000.hashCode()) {
        case -508378822:
            if (var10000.equals("clone()Ljava/lang/Object;")) {
                return CGLIB$clone$4$Proxy;
            }
            break;
        case 1548665657:
            if (var10000.equals("run(I)V")) {
                return CGLIB$run$0$Proxy;
            }
            break;
        case 1826985398:
            if (var10000.equals("equals(Ljava/lang/Object;)Z")) {
                return CGLIB$equals$1$Proxy;
            }
            break;
        case 1913648695:
            if (var10000.equals("toString()Ljava/lang/String;")) {
                return CGLIB$toString$2$Proxy;
            }
            break;
        case 1984935277:
            if (var10000.equals("hashCode()I")) {
                return CGLIB$hashCode$3$Proxy;
            }
        }

        return null;
    }

    public Car$$EnhancerByCGLIB$$9359a76e() {
        CGLIB$BIND_CALLBACKS(this);
    }

    public static void CGLIB$SET_THREAD_CALLBACKS(Callback[] var0) {
        CGLIB$THREAD_CALLBACKS.set(var0);
    }

    public static void CGLIB$SET_STATIC_CALLBACKS(Callback[] var0) {
        CGLIB$STATIC_CALLBACKS = var0;
    }

    private static final void CGLIB$BIND_CALLBACKS(Object var0) {
        Car$$EnhancerByCGLIB$$9359a76e var1 = (Car$$EnhancerByCGLIB$$9359a76e)var0;
        if (!var1.CGLIB$BOUND) {
            var1.CGLIB$BOUND = true;
            Object var10000 = CGLIB$THREAD_CALLBACKS.get();
            if (var10000 == null) {
                var10000 = CGLIB$STATIC_CALLBACKS;
                if (CGLIB$STATIC_CALLBACKS == null) {
                    return;
                }
            }

            var1.CGLIB$CALLBACK_0 = (MethodInterceptor)((Callback[])var10000)[0];
        }

    }

    public Object newInstance(Callback[] var1) {
        CGLIB$SET_THREAD_CALLBACKS(var1);
        Car$$EnhancerByCGLIB$$9359a76e var10000 = new Car$$EnhancerByCGLIB$$9359a76e();
        CGLIB$SET_THREAD_CALLBACKS((Callback[])null);
        return var10000;
    }

    public Object newInstance(Callback var1) {
        CGLIB$SET_THREAD_CALLBACKS(new Callback[]{var1});
        Car$$EnhancerByCGLIB$$9359a76e var10000 = new Car$$EnhancerByCGLIB$$9359a76e();
        CGLIB$SET_THREAD_CALLBACKS((Callback[])null);
        return var10000;
    }

    public Object newInstance(Class[] var1, Object[] var2, Callback[] var3) {
        CGLIB$SET_THREAD_CALLBACKS(var3);
        Car$$EnhancerByCGLIB$$9359a76e var10000 = new Car$$EnhancerByCGLIB$$9359a76e;
        switch(var1.length) {
        case 0:
            var10000.<init>();
            CGLIB$SET_THREAD_CALLBACKS((Callback[])null);
            return var10000;
        default:
            throw new IllegalArgumentException("Constructor not found");
        }
    }

    public Callback getCallback(int var1) {
        CGLIB$BIND_CALLBACKS(this);
        MethodInterceptor var10000;
        switch(var1) {
        case 0:
            var10000 = this.CGLIB$CALLBACK_0;
            break;
        default:
            var10000 = null;
        }

        return var10000;
    }

    public void setCallback(int var1, Callback var2) {
        switch(var1) {
        case 0:
            this.CGLIB$CALLBACK_0 = (MethodInterceptor)var2;
        default:
        }
    }

    public Callback[] getCallbacks() {
        CGLIB$BIND_CALLBACKS(this);
        return new Callback[]{this.CGLIB$CALLBACK_0};
    }

    public void setCallbacks(Callback[] var1) {
        this.CGLIB$CALLBACK_0 = (MethodInterceptor)var1[0];
    }

    static {
        CGLIB$STATICHOOK1();
    }
	}

	// Car$$FastClassByCGLIB$$8eb0d649 源码 继承FastClass
	public class Car$$FastClassByCGLIB$$8eb0d649 extends FastClass {
    public Car$$FastClassByCGLIB$$8eb0d649(Class var1) {
        super(var1);
    }
	//......


调用过程：代理对象调用this.run()方法->调用拦截器->methodProxy.invokeSuper->CGLIB$run$0->被代理对象run方法

#拦截器MethodInterceptor中就是由MethodProxy的invokeSuper方法调用代理方法的，MethodProxy非常关键，我们分析一下它具体做了什么。

1.创建MethodProxy

	public class MethodProxy {
	    private Signature sig1;
	    private Signature sig2;
	    private MethodProxy.CreateInfo createInfo;
	    private final Object initLock = new Object();
	    private volatile MethodProxy.FastClassInfo fastClassInfo;
	    //c1:被代理对象Class
	    //c2:代理对象Class
	    //desc：入参类型
	    //name1:被代理方法名
	    //name2:代理方法名
	    public static MethodProxy create(Class c1, Class c2, String desc, String name1, String name2) {
	        MethodProxy proxy = new MethodProxy();
	        proxy.sig1 = new Signature(name1, desc);//被代理方法签名
	        proxy.sig2 = new Signature(name2, desc);//代理方法签名
	        proxy.createInfo = new MethodProxy.CreateInfo(c1, c2);
	        return proxy;
	    }
	
	private static class CreateInfo {
	    Class c1;
	    Class c2;
	    NamingPolicy namingPolicy;
	    GeneratorStrategy strategy;
	    boolean attemptLoad;
	
	    public CreateInfo(Class c1, Class c2) {
	        this.c1 = c1;
	        this.c2 = c2;
	        AbstractClassGenerator fromEnhancer = AbstractClassGenerator.getCurrent();
	        if(fromEnhancer != null) {
	            this.namingPolicy = fromEnhancer.getNamingPolicy();
	            this.strategy = fromEnhancer.getStrategy();
	            this.attemptLoad = fromEnhancer.getAttemptLoad();
	        }
	
	    }
	}

2.invokeSuper调用
	
	public Object invokeSuper(Object obj, Object[] args) throws Throwable {
	        try {
	            this.init();
	            MethodProxy.FastClassInfo fci = this.fastClassInfo;
	            return fci.f2.invoke(fci.i2, obj, args);
	        } catch (InvocationTargetException var4) {
	            throw var4.getTargetException();
	        }
	    }
	
	private static class FastClassInfo {
	    FastClass f1;//被代理类FastClass
	    FastClass f2;//代理类FastClass
	    int i1; //被代理类的方法签名(index)
	    int i2;//代理类的方法签名
	
	    private FastClassInfo() {
	    }
	}

上面代码调用过程就是获取到代理类对应的FastClass，

#FastClass机制

Cglib动态代理执行代理方法效率之所以比JDK的高是因为Cglib采用了FastClass机制，它的原理简单来说就是：为代理类和被代理类各生成一个Class，这个Class会为代理类或被代理类的方法分配一个index(int类型)。

这个index当做一个入参，FastClass就可以直接定位要调用的方法直接进行调用，这样省去了反射调用，所以调用效率比JDK动态代理通过反射调用高。下面我们反编译一个FastClass看看：


	//根据方法签名获取index
 	public int getIndex(Signature var1) {
      String var10000 = var1.toString();
      switch(var10000.hashCode()) {
      case -2077043409:
         if(var10000.equals("getPerson(Ljava/lang/String;)Lcom/demo/pojo/Person;")) {
            return 21;
         }
         break;
      case -2055565910:
         if(var10000.equals("CGLIB$SET_THREAD_CALLBACKS([Lnet/sf/cglib/proxy/Callback;)V")) {
            return 12;
         }
         break;
      case -1902447170:
         if(var10000.equals("setPerson()V")) {
            return 7;
         }
	         break;
	   //省略部分代码.....
	　
	 //根据index直接定位执行方法
	 public Object invoke(int var1, Object var2, Object[] var3) throws InvocationTargetException {
      eaaaed75 var10000 = (eaaaed75)var2;
      int var10001 = var1;

      try {
         switch(var10001) {
         case 0:
            return new Boolean(var10000.equals(var3[0]));
         case 1:
            return var10000.toString();
         case 2:
            return new Integer(var10000.hashCode());
         case 3:
            return var10000.newInstance((Class[])var3[0], (Object[])var3[1], (Callback[])var3[2]);
         case 4:
            return var10000.newInstance((Callback)var3[0]);
         case 5:
            return var10000.newInstance((Callback[])var3[0]);
         case 6:
            var10000.setCallback(((Number)var3[0]).intValue(), (Callback)var3[1]);
            return null;
         case 7:
            var10000.setPerson();
            return null;
         case 8:
            var10000.setCallbacks((Callback[])var3[0]);
            return null;
         case 9:
            return var10000.getCallback(((Number)var3[0]).intValue());
         case 10:
            return var10000.getCallbacks();
         case 11:
            eaaaed75.CGLIB$SET_STATIC_CALLBACKS((Callback[])var3[0]);
            return null;
         case 12:
            eaaaed75.CGLIB$SET_THREAD_CALLBACKS((Callback[])var3[0]);
            return null;
         case 13:
            return eaaaed75.CGLIB$findMethodProxy((Signature)var3[0]);
         case 14:
            return var10000.CGLIB$toString$3();
         case 15:
            return new Boolean(var10000.CGLIB$equals$2(var3[0]));
         case 16:
            return var10000.CGLIB$clone$5();
         case 17:
            return new Integer(var10000.CGLIB$hashCode$4());
         case 18:
            var10000.CGLIB$finalize$1();
            return null;
         case 19:
            var10000.CGLIB$setPerson$0();
            return null;
        //省略部分代码....
      } catch (Throwable var4) {
         throw new InvocationTargetException(var4);
      }

      throw new IllegalArgumentException("Cannot find matching method/constructor");
   }

 FastClass并不是跟代理类一块生成的，而是在第一次执行MethodProxy invoke/invokeSuper时生成的并放在了缓存中。

	//MethodProxy invoke/invokeSuper都调用了init()
	private void init() {
        if(this.fastClassInfo == null) {
            Object var1 = this.initLock;
            synchronized(this.initLock) {
                if(this.fastClassInfo == null) {
                    MethodProxy.CreateInfo ci = this.createInfo;
                    MethodProxy.FastClassInfo fci = new MethodProxy.FastClassInfo();
                    fci.f1 = helper(ci, ci.c1);//如果缓存中就取出，没有就生成新的FastClass
                    fci.f2 = helper(ci, ci.c2);
                    fci.i1 = fci.f1.getIndex(this.sig1);//获取方法的index
                    fci.i2 = fci.f2.getIndex(this.sig2);
                    this.fastClassInfo = fci;
                    this.createInfo = null;
                }
            }
        }

    }**







##总结 （JDK动态代理和Gglib动态代理的区别）
至此，动态代理的原理我们就基本搞清楚了，代码细节有兴趣可以再研究下。

最后我们总结一下JDK动态代理和Gglib动态代理的区别：

1.JDK动态代理是实现了被代理对象的接口，Cglib是继承了被代理对象。

2.JDK和Cglib都是在运行期生成字节码，JDK是直接写Class字节码，Cglib使用ASM框架写Class字节码，Cglib代理实现更复杂，生成代理类比JDK效率低。

3.JDK调用代理方法，是通过反射机制调用，Cglib是通过FastClass机制直接调用方法，Cglib执行效率更高。