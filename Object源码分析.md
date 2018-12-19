##Object 源码分析

#Object源码
  
	

	package java.lang;
	
	/**
	 * Class {@code Object} is the root of the class hierarchy.
	 * @author  unascribed
	 * @see     java.lang.Class  默认是所有类的父类
	 * @since   JDK1.0
		Object类是类层次结构的根类。Object类是每一个类的超类。所有对象，包括数组，都实现了这个类的方法。 
		Object类属于java.lang包，所有类都直接或间接继承Object类，在Jdk1.6版本中Object类共有11个方法。 
		Object类中有很多native方法，也称为本地方法，具体是用C（C++）在动态库中实现的，然后通过JNI调用。
	 */

	public class Object {

	//本地方法，对象初始化时自动调用此方法
    private static native void registerNatives();
    static {
        registerNatives();
    }

 	// 1. 返回这个Object的运行时Class对象。此对象也被用于该类的static synchronized方法的监视器对象。
    public final native Class<?> getClass();
	
    // 2. 返回这个对象的哈希值，默认是返回该对象的内存地址。重写此方法可以提高哈希结构的集合的性能，例如HashMap。
    public native int hashCode();

   	// 3. 对象比较方法，判断当前对象与其它对象是否相等，默认是比较两个对象的地址是否相同。
    // 一般重写equals方法时会同时重写hashCode方法
    public boolean equals(Object obj) {
        return (this == obj);
    }
 	// 4.对象克隆方法用于对象的复制，支持复制的类必须要实现Cloneable接口，如果没有实现Cloneable接口，执行clone方法会产生异常。 
    protected native Object clone() throws CloneNotSupportedException;

    // 5. 返回该对象的字符串表示形式，默认是使用类名+16进制的hashcode值(内存地址)，重写此方法可以提高类的可读性。
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }

  	// 6. 线程唤醒(单个)，唤醒在此对象监视器上等待的单个线程。可用于线程间相互协作通信。
    public final native void notify();

	// 7. 线程唤醒(所有)，唤醒在此对象监视器上等待的所有线程。
    public final native void notifyAll();

	// 8. 线程等待，本地方法。timeout表示要等待的最长时间（以毫秒为单位）。 
    public final native void wait(long timeout) throws InterruptedException;
	
  	// 9. 线程等待，支持毫微秒修正值，进行四舍五入
    public final void wait(long timeout, int nanos) throws InterruptedException {
        if (timeout < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

        if (nanos > 0) {
            timeout++;
        }

        wait(timeout);
    }

    // 10. 线程等待，与notify方法是相对的方法，wait与nofity两个方法实现线程通信，这两个方法必须在同一个监控器对象的同步代码块中使用。
    public final void wait() throws InterruptedException {
        wait(0);
    }


	 // 11. 类似析构函数，当垃圾回收器确定不存在对该对象的更多引用时，由对象的垃圾回收器调用此方法，用于释放资源。
    // finalized方法在由GC调用，并且还允许一次再生(标记为finalized后该对象在这个线程中又可达了)，可能会拖慢GC速度导致频繁fullGC，甚至直接OOM。
    // finalize的执行是不确定的，既不确定由哪个线程执行，也不确定执行的顺序。
    // 由于finalize方法可能影响GC，因此可以使用PhantomReference虚引用来替代finalize的功能。
    // effective java告诉我们，最好的做法是提供close()方法，并且告知上层应用在不需要该对象时一掉要调用这类接口。

    protected void finalize() throws Throwable { }
	}






#Object 方法示例

	import lombok.Getter;
	import lombok.Setter;
	
	/**
	 * @Auther: makui
	 * @Date: 2018/12/14
	 * @Description:
	 */
	@Setter
	@Getter
	public class Clone_Demo implements Cloneable{
    private String name;
    public static void main(String[] args) {
        Clone_Demo c = new Clone_Demo();
        c.setName("ma");
        Clone_Demo c1 = new Clone_Demo();
        c1.setName("ma");
        try {
            Clone_Demo d = (Clone_Demo)c.clone(); //如果没有实现cloneable接口，调用clone()方法，会抛出java.lang.CloneNotSupportedException
            System.out.println(d.getClass() == c.getClass()); // 克隆的两个class类型相同true
            System.out.println("克隆属性"+ d.getName());  //克隆属性ma
            System.out.println("克隆对象是否相等"+ (d==c)); //克隆对象是否相等false
			System.out.println("克隆的两个对象是否equal相同" + c.equals(d));  //克隆的两个对象是否equal相同 false
            System.out.println("克隆的两个对象是否hashCode相同" + (c.hashCode() == d.hashCode()));  //克隆的两个对象是否hashCode相同 false
            System.out.println(c==c1); //false  equal 不只是比较两个对象内容是否相同 也比较两个对象的内存地址
            System.out.println(c.hashCode() == c1.hashCode()); //false 默认是返回该对象的内存地址
            System.out.println(c.toString()); //com.mk.coffee.test.jdk.resource.Clone_Demo@53bd815b  默认是返回该对象的内存地址  重写此方法可以提高类的可读性。
            //c.wait(); // 没有只有当前对象的锁，使用wait 方法会 抛出IllegalMonitorStateException
			//c.notify();// 没有只有当前对象的锁，使用wait 方法会 抛出IllegalMonitorStateException
            //c.wait(1000);// 没有只有当前对象的锁，使用wait 方法会 抛出IllegalMonitorStateException
           //System.out.println(c.getName());
           //c.notify();// 没有只有当前对象的锁，使用wait 方法会 抛出IllegalMonitorStateException
            System.out.println("111");
            synchronized (c){   //下面代码会执行 因为会获得这个当前对象锁  上面没有使用synchronized（）修饰的 都会抛出异常
                System.out.println("获取当前锁 并等待10S");
                c.wait(10000);
                System.out.println("已等待10S");
                System.out.println("111111");
            }
            c.finalize(); // finalize方法不会抛出Exception，而是Throwable  （关于Exception和Throwable之后详细了解）
            System.out.println("在调用finalize之后,尝试去获取他的属性"+c.getName()); //ma  finalize执行 并不保证一定会垃圾回收处理 所以一般不会使用finalize方法去垃圾回收（此处可以详细了解JVM 垃圾回收机制）
        } catch (CloneNotSupportedException | InterruptedException e) {
            e.printStackTrace();
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }

    }
	}


##Object 总结

	Object是所有类的基类，需要了解其所有的方法 才能更好的学习Java更深层的东西。 务必学习Object。