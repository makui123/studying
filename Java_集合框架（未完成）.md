##集合框架

  ![](https://images2015.cnblogs.com/blog/472013/201702/472013-20170223204818085-299175911.png)


从类图结构可以了解 java.util包下的2个大类：

　　1、Collecton：可以理解为主要存放的是单个对象

　　2、Map：可以理解为主要存储key-value类型的对象


##Collection类分析
  
  #Collection接口源码
	


	package java.util;
	
	import java.util.function.Predicate;
	import java.util.stream.Stream;
	import java.util.stream.StreamSupport;
	
	
	 *
	 * @author  Josh Bloch
	 * @author  Neal Gafter
	 */
	
	public interface Collection<E> extends Iterable<E> {
	    // Query Operations
	
	  	//返回此集合的大小 小于等于Integer的最大值
	    int size();
	
	    //判断此集合不包含任何元素
	    boolean isEmpty();
	
	 	//判断此集合是否包含此对象
	    boolean contains(Object o);
	
		//遍历集合中的元素
	    Iterator<E> iterator();
	
		//返回此集合中包含的所有元素数组
	    Object[] toArray();
	
		//返回此集合中包含所有元素的数组; 返回的数组的运行时类型是指定数组的运行时类型。
	    <T> T[] toArray(T[] a);
	
	   	//确保此集合包含指定的元素（可选操作）此处不一定是增加 只是确保此元素存在。
	    boolean add(E e);
	
	  	//从该集合中删除指定元素的单个实例（如果存在）（可选操作）。 
	    boolean remove(Object o);
	
	
	  	//如果此集合包含指定 集合中的所有元素，则返回true。 
	    boolean containsAll(Collection<?> c);
	
	 	//确保集合中的所有元素添加到此集合（可选操作） 。 
	    boolean addAll(Collection<? extends E> c);
	
	  	//删除指定集合中包含的所有此集合的元素（可选操作）。 
	    boolean removeAll(Collection<?> c);
	

		//删除该集合中满足给定谓词的所有元素。在迭代期间或由谓词引发的错误或运行时异常被转发给调用者。
		//如果无法从该集合中移除元素，则将不支持操作异常。如果不能删除匹配元素，或者一般不支持移除，则实现可能引发此异常UnsupportedOperationException。
	    default boolean removeIf(Predicate<? super E> filter) {
	        Objects.requireNonNull(filter);
	        boolean removed = false;
	        final Iterator<E> each = iterator();
	        while (each.hasNext()) {
	            if (filter.test(each.next())) {
	                each.remove();
	                removed = true;
	            }
	        }
	        return removed;
	    }
	
	   	//仅保留此集合中包含在指定集合中的元素（可选操作）。 
	    boolean retainAll(Collection<?> c);
	
	  	//从此集合中删除所有元素（可选操作）。 
	    void clear();
	
	
	   	//将指定的对象与此集合进行比较以获得相等性。 
	    boolean equals(Object o);
	
	    //返回此集合的哈希码值。 
	    int hashCode();
	
	   //创建一个Spliterator在这个集合中的元素???
	    @Override
	    default Spliterator<E> spliterator() {
	        return Spliterators.spliterator(this, 0);
	    }
	
	   //返回以此集合作为源的顺序 Stream 。 
	    default Stream<E> stream() {
	        return StreamSupport.stream(spliterator(), false);
	    }
	
	    default Stream<E> parallelStream() {
	        return StreamSupport.stream(spliterator(), true);
	    }
	}


Spliterator是JDK1.8新特性
　Spliterator是一个可分割迭代器(splitable iterator)，可以并行遍历元素，对于并行处理的能力大大增强。


#Spliterator源码

	package java.util;
	
	import java.util.function.Consumer;
	import java.util.function.DoubleConsumer;
	import java.util.function.IntConsumer;
	import java.util.function.LongConsumer;
	
	/*
	 * 可分割的迭代器 	
	 * @see Collection
	 * @since 1.8
	 */
	public interface Spliterator<T> {
	  	//如果剩下的元素存在，执行给定的操作，返回true ; 否则返回false 。
	    boolean tryAdvance(Consumer<? super T> action);
	
	   	//在当前线程中依次执行每个剩余元素的给定操作，直到所有元素都被处理或动作引发异常
	    default void forEachRemaining(Consumer<? super T> action) {
	        do { } while (tryAdvance(action));
	    }
	
	    //如果此分割器可以被分区，返回一个包含元素的Spliter，当从该方法返回时，它不会被该Spliter所覆盖。
	    Spliterator<T> trySplit();
	
	   	////用于估算还剩下多少个元素需要遍历
	    long estimateSize();
	
	  	////当迭代器拥有SIZED特征时，返回剩余元素个数；否则返回-1
	    default long getExactSizeIfKnown() {
	        return (characteristics() & SIZED) == 0 ? -1L : estimateSize();
	    }
	
	  	///返回此Spliterator及其元素的一组特征。
	    int characteristics();
	
	  	//如果Spliterator的 characteristics()包含所有给定的特性,返回 true
	    default boolean hasCharacteristics(int characteristics) {
	        return (characteristics() & characteristics) == characteristics;
	    }
	
	  	//如果Spliterator的list是通过Comparator排序的，则返回Comparator
		//如果Spliterator的list是自然排序的 ，则返回null
		//其他情况下抛错	
	    default Comparator<? super T> getComparator() {
	        throw new IllegalStateException();
	    }
	
	   //特征值表示为元素定义遇到顺序。 
	    public static final int ORDERED    = 0x00000010;
	
	  	//特性值这标志着，对于每对遇到的元件 x, y ， !x.equals(y) 。 
	    public static final int DISTINCT   = 0x00000001;
	
	  	//特征值表示遇到的顺序遵循定义的排序顺序
	    public static final int SORTED     = 0x00000004;
	
	   	//表示在遍历或 estimateSize()之前从 estimateSize()返回的值的特征值表示在没有结构源修改的情况下表示完全遍历将遇到的元素数量的精确计数的有限大小。
	    public static final int SIZED      = 0x00000040;
	
	   //特征值表示源保证遇到的元素不会为 null 。 
	    public static final int NONNULL    = 0x00000100;
	
	  	//特征值表示元素源不能在结构上进行修改; 也就是说，不能添加，替换或删除元素，因此在遍历过程中不会发生这种更改。
	    public static final int IMMUTABLE  = 0x00000400;
	
	   	//特征值表示可以通过多个线程安全同时修改元素源（允许添加，替换和/或删除），而无需外部同步。 
	    public static final int CONCURRENT = 0x00001000;
	
	   	//特征值这标志着从产生的所有Spliterators trySplit()将是既 SIZED和 SUBSIZED 。
	    public static final int SUBSIZED = 0x00004000;
	
	 
	    public interface OfPrimitive<T, T_CONS, T_SPLITR extends Spliterator.OfPrimitive<T, T_CONS, T_SPLITR>>
	            extends Spliterator<T> {
	        @Override
	        T_SPLITR trySplit();
	
	        @SuppressWarnings("overloads")
	        boolean tryAdvance(T_CONS action);
	
	       
	        @SuppressWarnings("overloads")
	        default void forEachRemaining(T_CONS action) {
	            do { } while (tryAdvance(action));
	        }
	    }
	
	   
	    public interface OfInt extends OfPrimitive<Integer, IntConsumer, OfInt> {
	
	        @Override
	        OfInt trySplit();
	
	        @Override
	        boolean tryAdvance(IntConsumer action);
	
	        @Override
	        default void forEachRemaining(IntConsumer action) {
	            do { } while (tryAdvance(action));
	        }
	
	        
	        @Override
	        default boolean tryAdvance(Consumer<? super Integer> action) {
	            if (action instanceof IntConsumer) {
	                return tryAdvance((IntConsumer) action);
	            }
	            else {
	                if (Tripwire.ENABLED)
	                    Tripwire.trip(getClass(),
	                                  "{0} calling Spliterator.OfInt.tryAdvance((IntConsumer) action::accept)");
	                return tryAdvance((IntConsumer) action::accept);
	            }
	        }
	
	       
	        @Override
	        default void forEachRemaining(Consumer<? super Integer> action) {
	            if (action instanceof IntConsumer) {
	                forEachRemaining((IntConsumer) action);
	            }
	            else {
	                if (Tripwire.ENABLED)
	                    Tripwire.trip(getClass(),
	                                  "{0} calling Spliterator.OfInt.forEachRemaining((IntConsumer) action::accept)");
	                forEachRemaining((IntConsumer) action::accept);
	            }
	        }
	    }
	
	    /**
	     * A Spliterator specialized for {@code long} values.
	     * @since 1.8
	     */
	    public interface OfLong extends OfPrimitive<Long, LongConsumer, OfLong> {
	
	        @Override
	        OfLong trySplit();
	
	        @Override
	        boolean tryAdvance(LongConsumer action);
	
	        @Override
	        default void forEachRemaining(LongConsumer action) {
	            do { } while (tryAdvance(action));
	        }
	
	      
	        @Override
	        default boolean tryAdvance(Consumer<? super Long> action) {
	            if (action instanceof LongConsumer) {
	                return tryAdvance((LongConsumer) action);
	            }
	            else {
	                if (Tripwire.ENABLED)
	                    Tripwire.trip(getClass(),
	                                  "{0} calling Spliterator.OfLong.tryAdvance((LongConsumer) action::accept)");
	                return tryAdvance((LongConsumer) action::accept);
	            }
	        }
	
	      
	        @Override
	        default void forEachRemaining(Consumer<? super Long> action) {
	            if (action instanceof LongConsumer) {
	                forEachRemaining((LongConsumer) action);
	            }
	            else {
	                if (Tripwire.ENABLED)
	                    Tripwire.trip(getClass(),
	                                  "{0} calling Spliterator.OfLong.forEachRemaining((LongConsumer) action::accept)");
	                forEachRemaining((LongConsumer) action::accept);
	            }
	        }
	    }
	
	    /**
	     * A Spliterator specialized for {@code double} values.
	     * @since 1.8
	     */
	    public interface OfDouble extends OfPrimitive<Double, DoubleConsumer, OfDouble> {
	
	        @Override
	        OfDouble trySplit();
	
	        @Override
	        boolean tryAdvance(DoubleConsumer action);
	
	        @Override
	        default void forEachRemaining(DoubleConsumer action) {
	            do { } while (tryAdvance(action));
	        }
	
	       
	        @Override
	        default boolean tryAdvance(Consumer<? super Double> action) {
	            if (action instanceof DoubleConsumer) {
	                return tryAdvance((DoubleConsumer) action);
	            }
	            else {
	                if (Tripwire.ENABLED)
	                    Tripwire.trip(getClass(),
	                                  "{0} calling Spliterator.OfDouble.tryAdvance((DoubleConsumer) action::accept)");
	                return tryAdvance((DoubleConsumer) action::accept);
	            }
	        }
	
	       
	        @Override
	        default void forEachRemaining(Consumer<? super Double> action) {
	            if (action instanceof DoubleConsumer) {
	                forEachRemaining((DoubleConsumer) action);
	            }
	            else {
	                if (Tripwire.ENABLED)
	                    Tripwire.trip(getClass(),
	                                  "{0} calling Spliterator.OfDouble.forEachRemaining((DoubleConsumer) action::accept)");
	                forEachRemaining((DoubleConsumer) action::accept);
	            }
	        }
	    }
	}

 