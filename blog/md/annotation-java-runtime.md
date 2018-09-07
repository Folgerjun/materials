---
title: Java 运行时（RUNTIME）注解详解
date: 2018-9-7 10:29:41
categories: [开发,总结]
tags: [Java,反射,注解]
---

> 参考博文：[Java注解解析-运行时注解详解(RUNTIME)](https://blog.csdn.net/jsonChumpKlutz/article/details/81747839#commentBox)
> 
> 个人博客：[DoubleFJ の Blog](http://putop.top/2018/09/07/annotation-java-runtime/)

整理测试后并附上完整代码

---

## 注解定义 
注解（Annotation），也叫元数据。一种代码级别的说明。它是JDK1.5及以后版本引入的一个特性，与类、接口、枚举是在同一个层次。它可以声明在包、类、字段、方法、局部变量、方法参数等的前面，用来对这些元素进行说明 。如果要对于元数据的作用进行分类，还没有明确的定义，不过我们可以根据它所起的作用，注解不会改变编译器的编译方式，也不会改变虚拟机指令执行的顺序，它更可以理解为是一种特殊的注释，本身不会起到任何作用，需要工具方法或者编译器本身读取注解的内容继而控制进行某种操作。大致可分为三类：

- 编写文档：通过代码里标识的元数据生成文档。
- 代码分析：通过代码里标识的元数据对代码进行分析。
- 编译检查：通过代码里标识的元数据让编译器能实现基本的编译检查。

## 注解用途
因为注解可以在代码编译期间帮我们完成一些复杂的准备工作，所以我们可以利用注解去完成我们的一些准备工作。可以在编译期间获取到注解中的内容以便之后的数据处理，完全可以写好逻辑代码就等着编译时将值传入。

## 注解详解
Java JDK 中包含了三个注解分别为 @Override（校验格式），@Deprecated：（标记过时的方法或者类），@SuppressWarnnings（注解主要用于抑制编译器警告）等等。JDK 1.8 之后有新增了一些注解像 @FunctionalInterface()这样的,对于每个注解的具体使用细节这里不再论述。我们来看一下 @Override 的源码。
```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```
通过源代码的阅读我们可以看出生命注解的方式为 @interface，每个注解都需要不少于一个的元注解的修饰，这里的元注解其实就是修饰注解的注解，可以理解成最小的注解单位吧。下面详细的看下每个注释注解的意义吧：

### @Target
说明了 Annotation 所修饰的对象范围,也就是我们这个注解是用在那个对象上面的：Annotation 可被用于 packages、types（类、接口、枚举、Annotation 类型）、类型成员（方法、构造方法、成员变量、枚举值）、方法参数和本地变量（如循环变量、catch参数）。在 Annotation 类型的声明中使用了target可更加明晰其修饰的目标。以下属性是多选状态，我们可以定义多个注解作用域，比如：
```
@Target({ElementType.METHOD,ElementType.FIELD})，单个的使用 @Target(ElementType.FIELD)。    
（1）.CONSTRUCTOR：构造方法声明。
（2）.FIELD：用于描述域也就是类属性之类的，字段声明（包括枚举常量）。
（3）.LOCAL_VARIABLE：用于描述局部变量。
（4）.METHOD：用于描述方法。
（5）.PACKAGE：包声明。
（6）.PARAMETER：参数声明。
（7）.TYPE：类、接口（包括注释类型）或枚举声明 。
（8）.ANNOTATION_TYPE：注释类型声明，只能用于注释注解。
```
**官方解释：指示注释类型所适用的程序元素的种类。如果注释类型声明中不存在 Target 元注释，则声明的类型可以用在任一程序元素上。如果存在这样的元注释，则编译器强制实施指定的使用限制。** 例如，此元注释指示该声明类型是其自身，即元注释类型。它只能用在注释类型声明上:
```
@Target(ElementType.ANNOTATION_TYPE)
public @interface MetaAnnotationType {
}
```
此元注释指示该声明类型只可作为复杂注释类型声明中的成员类型使用。它不能直接用于注释：
```
@Target({}) 
public @interface MemberType {
             ...
}
```
这是一个**编译时错误**，它表明一个 ElementType 常量在 Target 注释中出现了不只一次。例如，以下元注释是非法的：
```
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.FIELD})
public @interface Bogus {
             ...
}
```

### @Retention
定义了该 Annotation 被保留的时间长短：某些 Annotation 仅出现在源代码中，而被编译器丢弃；而另一些却被编译在 class 文件中；编译在 class 文件中的 Annotation 可能会被虚拟机忽略，而另一些在 class 被装载时将被读取（请注意并不影响 class 的执行，因为 Annotation 与 class 在使用上是被分离的）。使用这个 meta-Annotation 可以对 Annotation 的“生命周期”限制。来源于 java.lang.annotation.RetentionPolicy 的枚举类型值： 
```
（1）.SOURCE:在源文件中有效（即源文件保留）编译成class文件将舍弃该注解。 
（2）.CLASS:在class文件中有效（即class保留） 编译成dex文件将舍弃该注解。 
（3）.RUNTIME:在运行时有效（即运行时保留） 运行时可见。 
```
**也就是说注解处理器能处理这三类的注解,我们通过反射的话只能处理 RUNTIME 类型的注解。**

**官方解释：指示注释类型的注释要保留多久。如果注释类型声明中不存在 Retention 注释，则保留策略默认为 RetentionPolicy.CLASS。只有元注释类型直接用于注释时，Target 元注释才有效。如果元注释类型用作另一种注释类型的成员，则无效。**

### @Documented
指示某一类型的注释将通过 javadoc 和类似的默认工具进行文档化。应使用此类型来注释这些类型的声明：其注释会影响由其客户端注释的元素的使用。如果类型声明是用 Documented 来注释的，则其注释将成为注释元素的公共 API 的一部。Documented 是一个标记注解，没有成员。

### @Inherited
元注解是一个标记注解，@Inherited 阐述了某个被标注的类型是**被继承的**。如果一个使用了 @Inherited 修饰的 annotation 类型被用于一个 class ，则这个 annotation 将被用于该 class 的子类。 注意：@Inherited annotation 类型是被标注过的 class 的子类所继承。类并不从它所实现的接口继承 annotation，方法并不从它所重载的方法继承 annotation。当 @Inherited annotation 类型标注的 annotation 的 Retention 是 RetentionPolicy.RUNTIME，则反射 API 增强了这种继承性。如果我们使用 java.lang.reflect 去查询一个 @Inherited annotation 类型的 annotation 时，反射代码检查将展开工作：检查 class 和其父类，直到发现指定的 annotation 类型被发现，或者到达类继承结构的顶层。

**官方解释：指示注释类型被自动继承。如果在注释类型声明中存在 Inherited 元注释，并且用户在某一类声明中查询该注释类型，同时该类声明中没有此类型的注释，则将在该类的超类中自动查询该注释类型。此过程会重复进行，直到找到此类型的注释或到达了该类层次结构的顶层 (Object) 为止。如果没有超类具有该类型的注释，则查询将指示当前类没有这样的注释。** 

***注意，如果使用注释类型注释类以外的任何事物，此元注释类型都是无效的。还要注意，此元注释仅促成从超类继承注释；对已实现接口的注释无效。***

### @Repeatable
Repeatable可重复性，JDK 1.8 新特性，其实就是把标注的注解放到该元注解所属的注解容器里面。以下是一个完整 Demo ：

**MyTag.java** ： 自定义注解
```
@Target({ ElementType.METHOD, ElementType.FIELD })
@Retention(RetentionPolicy.CLASS)
@Repeatable(MyCar.class) // 注解可重复使用 将 MyTag 作为 MyCar 中 value 的值，即放入了 MyCar 注解容器中
public @interface MyTag {

    // default 后为其默认值
    String name() default "";

    int size() default 0;
}
```

**MyCar.java** ： MyTag 的注解容器
```
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyCar {
    MyTag[] value(); // 注解里面属性的返回值是 Tag 注解的数组，即 MyTag 注解容器
}
```

**Car.java** ： 测试实体类
```
public class Car {

    private String name;

    private int size;

    public Car(String name, int size) {
        this.name = name;
        this.size = size;
    }

    // 省略了 set get

    @Override
    public String toString() {
        return "Car [name=" + name + ", size=" + size + "]";
    }

}
```

**AnnotationCar.java** ： 最关键的注解处理类
```
/**
 * Car 注解处理类
 * 
 * @author ffj
 *
 */
public class AnnotationCar {

    private AnnotationCar() {
    }

    private static volatile AnnotationCar annotationCar;

    public static AnnotationCar instance() {
        // 单例 双重检查
        if (annotationCar == null) {
            synchronized (AnnotationCar.class) {
                if (annotationCar == null) {
                    annotationCar = new AnnotationCar();
                }
            }
        }
        return annotationCar;
    }

    public void inject(Object o) {
        Class<?> aClass = o.getClass();
        Field[] declaredFields = aClass.getDeclaredFields(); // 获取所有声明的字段
        for (Field field : declaredFields) {
            if (field.getName().equals("car")) {
                Annotation[] annotations = field.getAnnotations();
                for (Annotation annotation : annotations) { // 注解的对象类型
                    Class<? extends Annotation> className = annotation.annotationType();
                    System.out.println("className :" + className);
                }
                MyCar annotation = field.getAnnotation(MyCar.class); // MyCar 类型输出
                MyTag[] tags = annotation.value();
                for (MyTag tag : tags) {
                    System.out.println("name :" + tag.name());
                    System.out.println("size :" + tag.size());
                    try {
                        field.setAccessible(true); // 类中的成员变量为 private,故必须进行此操作
                        field.set(o, new Car(tag.name(), tag.size())); // 重新赋值对象
                        System.out.println("注解对象为 ：" + field.get(o).toString());
                    } catch (IllegalArgumentException e) {
                        e.printStackTrace();
                    } catch (IllegalAccessException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }
}
```

**AnnotationTest.java** ： 测试运行类
```
public class AnnotationTest {

    @MyTag(name = "野马", size = 222)
    @MyTag(name = "兰博基尼", size = 333)
    Car car;

    public void printAnno() {
        AnnotationCar.instance().inject(this);
    }

    public static void main(String[] args) {
        new AnnotationTest().printAnno();

    }
}
```

**最终运行结果便是：**
```
className :interface com.tonglei.test.MyCar
name :野马
size :222
注解对象为 ：Car [name=野马, size=222]
name :兰博基尼
size :333
注解对象为 ：Car [name=兰博基尼, size=333]
```

## 总结
不知这样可否清晰。这里运行时注解就是在程序编译时扫描到类下的字段上的注解，就可以知道该字段上的元注解的类型，进而将注解中元素的值得到进行你自己的业务操作。这个 Demo 是利用了 @Repeatable 注解，不用该注解直接用元注解 RUNTIME 类型也是一样的，只要注解类逻辑稍微修改即可。结合这个可以更好地理解了反射和注解以及 class 的注入。