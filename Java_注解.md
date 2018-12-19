#注解
JDK1.5之后内部提供的三个注解

       @Deprecated 意思是“废弃的，过时的”

       @Override 意思是“重写、覆盖”

       @SuppressWarnings 意思是“压缩警告”
##注解的定义
  注解通过 @interface 关键字进行定义。
	
 public @interface TestAnnotation {
	
 }
##注解的应用
上面创建了一个注解，那么注解的的使用方法是什么呢。

@TestAnnotation 

public class Test {

}

##元注解
元注解是什么意思呢？

元注解是可以注解到注解上的注解，或者说元注解是一种基本注解，但是它能够应用到其它的注解上面。

如果难于理解的话，你可以这样理解。元注解也是一张标签，但是它是一张特殊的标签，它的作用和目的就是给其他普通的标签进行解释说明的。

元标签有 @Retention、@Documented、@Target、@Inherited、@Repeatable 5 种。

@Retention
Retention 的英文意为保留期的意思。当 @Retention 应用到一个注解上的时候，它解释说明了这个注解的的存活时间。

它的取值如下： 

- RetentionPolicy.SOURCE 注解只在源码阶段保留，在编译器进行编译时它将被丢弃忽视。 

- RetentionPolicy.CLASS 注解只被保留到编译进行的时候，它并不会被加载到 JVM 中。
  
- RetentionPolicy.RUNTIME 注解可以保留到程序运行的时候，它会被加载进入到 JVM 中，所以在程序运行时可以获取到它们。

我们可以这样的方式来加深理解，@Retention 去给一张标签解释的时候，它指定了这张标签张贴的时间。@Retention 相当于给一张标签上面盖了一张时间戳，时间戳指明了标签张贴的时间周期。

@Retention(RetentionPolicy.RUNTIME)

public @interface TestAnnotation {
}

上面的代码中，我们指定 TestAnnotation 可以在程序运行周期被获取到，因此它的生命周期非常的长。

@Documented

顾名思义，这个元注解肯定是和文档有关。它的作用是能够将注解中的元素包含到 Javadoc 中去。

@Target

Target 是目标的意思，@Target 指定了注解运用的地方。

你可以这样理解，当一个注解被 @Target 注解时，这个注解就被限定了运用的场景。

类比到标签，原本标签是你想张贴到哪个地方就到哪个地方，但是因为 @Target 的存在，它张贴的地方就非常具体了，比如只能张贴到方法上、类上、方法参数上等等。@Target 有下面的取值

ElementType.ANNOTATION_TYPE 可以给一个注解进行注解

ElementType.CONSTRUCTOR 可以给构造方法进行注解

ElementType.FIELD 可以给属性进行注解

ElementType.LOCAL_VARIABLE 可以给局部变量进行注解

ElementType.METHOD 可以给方法进行注解

ElementType.PACKAGE 可以给一个包进行注解

ElementType.PARAMETER 可以给一个方法内的参数进行注解

ElementType.TYPE 可以给一个类型进行注解，比如类、接口、枚举

@Inherited

Inherited 是继承的意思，但是它并不是说注解本身可以继承，而是说如果一个超类被 @Inherited 注解过的注解进行注解的话，那么如果它的子类没有被任何注解应用的话，那么这个子类就继承了超类的注解。 
说的比较抽象。代码来解释。

@Inherited

@Retention(RetentionPolicy.RUNTIME)

@interface Test {}


@Test
public class A {}


public class B extends A {}

注解 Test 被 @Inherited 修饰，之后类 A 被 Test 注解，类 B 继承 A,类 B 也拥有 Test 这个注解。

可以这样理解：

老子非常有钱，所以人们给他贴了一张标签叫做富豪。

老子的儿子长大后，只要没有和老子断绝父子关系，虽然别人没有给他贴标签，但是他自然也是富豪。

老子的孙子长大了，自然也是富豪。

这就是人们口中戏称的富一代，富二代，富三代。虽然叫法不同，好像好多个标签，但其实事情的本质也就是他们有一张共同的标签，也就是老子身上的那张富豪的标签。

@Repeatable

Repeatable 自然是可重复的意思。@Repeatable 是 Java 1.8 才加进来的，所以算是一个新的特性。

什么样的注解会多次应用呢？通常是注解的值可以同时取多个。

举个例子，一个人他既是程序员又是产品经理,同时他还是个画家。


@interface Persons {
    Person[]  value();
}


@Repeatable(Persons.class)
@interface Person{
    String role default "";
}


@Person(role="artist")

@Person(role="coder")

@Person(role="PM")

public class SuperMan{

}

注意上面的代码，@Repeatable 注解了 Person。而 @Repeatable 后面括号中的类相当于一个容器注解。

什么是容器注解呢？就是用来存放其它注解的地方。它本身也是一个注解。

我们再看看代码中的相关容器注解。

@interface Persons {

    Person[]  value();
}

按照规定，它里面必须要有一个 value 的属性，属性类型是一个被 @Repeatable 注解过的注解数组，注意它是数组。

如果不好理解的话，可以这样理解。Persons 是一张总的标签，上面贴满了 Person 这种同类型但内容不一样的标签。把 Persons 给一个 SuperMan 贴上，相当于同时给他贴了程序员、产品经理、画家的标签。

我们可能对于 @Person(role=”PM”) 括号里面的内容感兴趣，它其实就是给 Person 这个注解的 role 属性赋值为 PM ，大家不明白正常，马上就讲到注解的属性这一块。
##
#案例：

```java

import java.lang.annotation.ElementType;

import java.lang.annotation.Retention;

import java.lang.annotation.RetentionPolicy;

import java.lang.annotation.Target;

/**
 * @Auther: makui
 * @Date: 2018/12/11
 * @Description: 注解开发1
 */

@Target({ElementType.FIELD, ElementType.METHOD})

@Retention(RetentionPolicy.RUNTIME)

public @interface Validate {

    int min() default 1;

    int max() default 7;

    String reg() default "^(19|20)\\d{2}-(1[0-2]|0?[1-9])-(0?[1-9]|[1-2][0-9]|3[0-1])$";

    boolean NotNull() default true;
}


/**
 * @Auther: makui
 * @Date: 2018/12/11
 * @Description: 幼儿园
 */

@Data

public class Nursery {

    @Validate(min = 1,max = 6)
    public int age ;

    @Validate(NotNull = true)
    public String name;

    @Validate(NotNull = true, reg = "^(19|20)\\d{2}-(1[0-2]|0?[1-9])-(0?[1-9]|[1-2][0-9]|3[0-1])$")
    public String birthday;

    public static void main(String[] args) {
        Nursery nursery = new Nursery();
        nursery.setAge(6);
        nursery.setBirthday("2018-01-01");
        nursery.setName("马");
        boolean isTrue = NurseryCheck.checkNursery(nursery);
        System.out.println(isTrue);
    }
}


/**
 * @Auther: makui
 * @Date: 2018/12/11
 * @Description: 幼儿检测
 */

public class NurseryCheck {

    public static boolean checkNursery(Nursery nursery){
        if(nursery == null) {
            System.out.println("##校验对象为空##");
            return false;
        }
        // 获取Nursery类的所有属性（如果使用getFields，就无法获取到private的属性）
        Field[] fields = Nursery.class.getDeclaredFields();
        for(Field field : fields) {
            // 如果属性有注解，就进行校验
            if (field.isAnnotationPresent(Validate.class)) {
                Validate validate = field.getAnnotation(Validate.class);
                if ("name".equals(field.getName())){
                    if(StringUtils.isEmpty(nursery.getName())) {
                        if(validate.NotNull()){
                            System.out.println("##校验对象的名称不可为空，名称未通过校验##");
                            return false;
                        }else {
                            System.out.println("##校验对象的名称可为空,名称通过校验##");
                        }

                    }else {
                        System.out.println("##校验对象的名称不为空,名称通过校验##");
                    }

                }
                if("age".equals(field.getName())) {
                    if(validate.min() > nursery.getAge() || validate.max() < nursery.getAge()) {
                        System.out.println("##校验对象的年龄，年龄未通过校验 小于最小年龄1岁或者大于最大年龄##"+validate.max());
                        return false;
                    }else {
                        System.out.println("##校验对象的年龄，年龄通过校验##");
                    }
                }

                if("birthday".equals(field.getName())) {
                    if(!StringUtils.isEmpty(nursery.getBirthday())) {
                        if (!nursery.getBirthday().matches(validate.reg())) {
                            System.out.println("##校验对象的生日，生日未通过校验 生日格式不正确##");
                            return false;
                        } else {
                            System.out.println("##校验对象的生日，生日通过校验##");
                        }
                    }else {
                        System.out.println("##校验对象的生日，生日未通过校验 生日格式为空##");
                        return false;
                    }
                }

            }
        }
        return true;
    }
}

```