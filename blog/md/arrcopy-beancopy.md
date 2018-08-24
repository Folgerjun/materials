---
title: Java中的数组拷贝以及对象Bean拷贝
date: 2018-8-24 14:12:26
categories: [开发,总结]
tags: [Java]
---

## 数组拷贝方式
直接先贴出测试代码：

**Student ：**
```
package com.tonglei.test;

/**
 * 学生实体测试类
 * 
 * @author ffj
 *
 */
public class Student {

    private int age;
    private int height;
    private String sex;

    public Student(int age, int height, String sex) {
        this.age = age;
        this.height = height;
        this.sex = sex;
    }

    // 省略set、get方法

    @Override
    public String toString() {

        return "Student :[" + this.age + " ," + this.height + " ," + this.sex + "]";
    }
}

```

**运行测试 ：**
```
public static void main(String[] args) {

        Student[] stu = new Student[3];
        stu[0] = new Student(11, 110, "男");
        stu[1] = new Student(12, 120, "女");
        stu[2] = new Student(13, 130, "嬲");
        System.out.println("stu length :" + stu.length);
        System.out.println("stu :" + Arrays.toString(stu));
        System.out.println("stu address :" + stu);

        System.out.println("<------------------------------------->");

        // Arrays.copyOf 原数组 新数组长度
        Student[] arrayCopyStu = Arrays.copyOf(stu, stu.length + 1);
        arrayCopyStu[stu.length] = new Student(14, 140, "奻");
        System.out.println("arrayCopyStu length :" + arrayCopyStu.length);
        System.out.println("arrayCopyStu :" + Arrays.toString(arrayCopyStu));
        System.out.println("arrayCopyStu address :" + arrayCopyStu);

        System.out.println("<------------------------------------->");

        // System.arraycopy 原数组 原数组拷贝起始地址 目标数组 目标数组拷贝起始地址 拷贝长度
        Student[] systemCopyStu = new Student[4];
        System.arraycopy(stu, 0, systemCopyStu, 0, stu.length);
        System.out.println("systemCopyStu length :" + systemCopyStu.length);
        System.out.println("systemCopyStu :" + Arrays.toString(systemCopyStu));
        System.out.println("systemCopyStu address :" + systemCopyStu);

        System.out.println("<------------------------------------->");
        System.out.println("<---------改变了原数组第一个对象的age------->");
        System.out.println("<------------------------------------->");

        // 改变原数组的数据
        stu[0].setAge(99);
        System.out.println("stu :" + Arrays.toString(stu));
        System.out.println("arrayCopyStu :" + Arrays.toString(arrayCopyStu));
        System.out.println("systemCopyStu :" + Arrays.toString(systemCopyStu));

        /**
         * 总结：Arrays.copyOf 和 System.arraycopy 都可将结果生成一个新数组，
         * 不过两者的区别在于，Arrays.copyOf()不仅仅只是拷贝数组中的元素，在拷贝元素时，会创建一个新的数组对象。而System.arrayCopy只拷贝已经存在数组元素。
         * Arrays.copyOf()的源码中可知其底层还是调用了System.arrayCopyOf()方法
         * 当修改了原数组中对象的属性时目标数组中也随之改变，故两者都是地址引用，其中元素指向的还是原数组的地址
         */
    }

```

**测试运行结果 ：**
```
stu length :3
stu :[Student :[11 ,110 ,男], Student :[12 ,120 ,女], Student :[13 ,130 ,嬲]]
stu address :[Lcom.tonglei.test.Student;@70dea4e
<------------------------------------->
arrayCopyStu length :4
arrayCopyStu :[Student :[11 ,110 ,男], Student :[12 ,120 ,女], Student :[13 ,130 ,嬲], Student :[14 ,140 ,奻]]
arrayCopyStu address :[Lcom.tonglei.test.Student;@5c647e05
<------------------------------------->
systemCopyStu length :4
systemCopyStu :[Student :[11 ,110 ,男], Student :[12 ,120 ,女], Student :[13 ,130 ,嬲], null]
systemCopyStu address :[Lcom.tonglei.test.Student;@33909752
<------------------------------------->
<---------改变了原数组第一个对象的age------->
<------------------------------------->
stu :[Student :[99 ,110 ,男], Student :[12 ,120 ,女], Student :[13 ,130 ,嬲]]
arrayCopyStu :[Student :[99 ,110 ,男], Student :[12 ,120 ,女], Student :[13 ,130 ,嬲], Student :[14 ,140 ,奻]]
systemCopyStu :[Student :[99 ,110 ,男], Student :[12 ,120 ,女], Student :[13 ,130 ,嬲], null]
```

### Arrays.copyOf
`Arrays.copyOf`方法返回一个新数组，不仅仅只是拷贝原数组中的元素会创建一个新的数组对象

**上述测试结果 ：**
```
<------------------------------------->
arrayCopyStu length :4
arrayCopyStu :[Student :[11 ,110 ,男], Student :[12 ,120 ,女], Student :[13 ,130 ,嬲], Student :[14 ,140 ,奻]]
arrayCopyStu address :[Lcom.tonglei.test.Student;@5c647e05
<------------------------------------->
```
可以看出，copy原数组中元素并扩容了一长度，同时`arrayCopyStu[stu.length] = new Student(14, 140, "奻");`对新增元素赋值，从而打印出的便是上述内容。

### System.arraycopy
`System.arraycopy(Object src, int srcPos, Object dest, int destPos, int length)`方法只拷贝已经存在数组元素，参数依次为原数组、原数组拷贝起始地址、目标数组、目标数组拷贝起始地址、拷贝长度。

**上述测试结果 ：**
```
<------------------------------------->
systemCopyStu length :4
systemCopyStu :[Student :[11 ,110 ,男], Student :[12 ,120 ,女], Student :[13 ,130 ,嬲], null]
systemCopyStu address :[Lcom.tonglei.test.Student;@33909752
<------------------------------------->
```
可见，新数组只从原数组中拷贝了存在的指定个数元素并可以指定拷贝到目标数组中。
从`Arrays.copyOf()`的源码中可知其底层还是调用了`System.arrayCopyOf()`方法。

### 进而探究
两种均可对数组进行copy，但是`Arrays.copyOf()`和`System.arrayCopyOf()`两种方法所copy生成的新的数组对象中的元素对象到底是新的还是依旧指向原先的呢？**来一探究竟！**

然而细心的同学已经心领神会..测试代码早早贴在了上面！
```
System.out.println("<------------------------------------->");
System.out.println("<---------改变了原数组第一个对象的age------->");
System.out.println("<------------------------------------->");

// 改变原数组的数据
stu[0].setAge(99);
System.out.println("stu :" + Arrays.toString(stu));
System.out.println("arrayCopyStu :" + Arrays.toString(arrayCopyStu));
System.out.println("systemCopyStu :" + Arrays.toString(systemCopyStu));
```
这段代码我改变了原数组中第一个stu对象元素中的age属性值，我将其改为了99。

**结果显示为 ：**
```
<------------------------------------->
<---------改变了原数组第一个对象的age------->
<------------------------------------->
stu :[Student :[99 ,110 ,男], Student :[12 ,120 ,女], Student :[13 ,130 ,嬲]]
arrayCopyStu :[Student :[99 ,110 ,男], Student :[12 ,120 ,女], Student :[13 ,130 ,嬲], Student :[14 ,140 ,奻]]
systemCopyStu :[Student :[99 ,110 ,男], Student :[12 ,120 ,女], Student :[13 ,130 ,嬲], null]
```
显而易见了，不管是原数组还是两个copy的数组，其值均由原先的11变成了99。

***总结：当修改了原数组中对象的属性时目标数组中也随之改变，故两者都是地址引用，其中元素指向的还是原数组的地址。***

## 对象拷贝方式
测试类还是上面的`Student`，只不过我又新增了一个`Teacher`类来方便测试，其结构与`Student`一致。

**Teacher :**
```
package com.tonglei.test;

/**
 * 教师实体测试类
 * 
 * @author ffj
 *
 */
public class Teacher {

    private int age;
    private int height;
    private String sex;

    public Teacher() {
    }

    public Teacher(int age, int height, String sex) {
        this.age = age;
        this.height = height;
        this.sex = sex;
    }

    // 省略了set、get方法

    @Override
    public String toString() {

        return "Teacher :[" + this.age + " ," + this.height + " ," + this.sex + "]";
    }
}
```

**测试的初始代码 ：**
```
Student stu = new Student(18, 180, "男");
Teacher tea = new Teacher();
```

### SpringBeanUtils
`BeanUtils.copyProperties(Object source, Object target) `该方法为spring中方法，故需要导入相应jar包。参数依次为：源数组、目标数组。（方法间参数有差异，需注意！）

**测试代码 ：**
```
BeanUtils.copyProperties(stu, tea);
System.out.println(tea);
```
在运行测试之前，我先将原先`Teacher`中的`age`字段属性类型稍稍修改了下。改为了`private String age`，结果为`Could not copy properties from source to target; nested exception is java.lang.IllegalArgumentException`，抛了个异常错误。而后我将其类型改回，输出`Teacher :[18 ,180 ,男]`。**故：拷贝的目标数组与源数组中的元素对象其属性名称一样，类型就必须一致，否则会报错**

### commonsBeanUtils
该包下有两个copy方法：`BeanUtils.copyProperties(Object dest, Object orig)`和`PropertyUtils.copyProperties(Object dest, Object orig)`，同样使用其方法前需要导入对应`org.apache.commons`jar包。

#### BeanUtils.copyProperties
`BeanUtils.copyProperties(Object dest, Object orig)`其中参数分别为：目标数组、源数组。（对了，跟spring中方法参数顺序不一样）

**测试代码 ：**
```
System.out.println(stu);
org.apache.commons.beanutils.BeanUtils.copyProperties(tea, stu);
System.out.println(tea);
```
还是老步骤：在运行测试之前，我先将原先`Teacher`中的`age`字段属性类型稍稍修改了下。改为了`private String age`，结果显示为：
```
Student :[18 ,180 ,男]
Teacher :[18 ,180 ,男]
```
我再将`Teacher`中的`sex`字段改为了`private int sex`，结果显示为：
```
Student :[18 ,180 ,男]
Teacher :[18 ,180 ,0]
```
**由此可知：拷贝的目标数组与源数组中的元素对象其属性名称一样，类型不一致则会强转该值，若是强转不了就为初始值**

#### PropertyUtils.copyProperties
`PropertyUtils.copyProperties(Object dest, Object orig)`其中参数分别为：目标数组、源数组。

**测试代码 ：**
```
System.out.println(stu);
PropertyUtils.copyProperties(tea, stu);
System.out.println(tea);
```
一样，在运行测试之前，我先将原先`Teacher`中的`age`字段属性类型稍稍修改了下。改为了`private String age`，结果显示为：`Cannot invoke com.tonglei.test.Teacher.setAge on bean class 'class com.tonglei.test.Teacher' - argument type mismatch - had objects of type "java.lang.Integer" but expected signature "java.lang.String"`，报错抛出类型不匹配异常信息，再将类型改回，输出：
```
Student :[18 ,180 ,男]
Teacher :[18 ,180 ,男]
```
**故：拷贝的目标数组与源数组中的元素对象其属性名称一样，类型就必须一致，否则会报错（与SpringBeanUtils差不多不过报错信息更为详细，纯粹这次简单测试个人体会）**

## 参考博文
- [关于Java 拷贝数组方法 Arrays.copyOf() 是地址传递还是值传递](https://blog.csdn.net/qq_27093465/article/details/54970538)
- [Java对象拷贝(BeanUtil.copyProperties 方法)](https://blog.csdn.net/weixin_38399962/article/details/79664598)
- [Java开发中beancopy比较](https://www.cnblogs.com/fanguangdexiaoyuer/p/8358720.html)
- [Java对象间属性值的复制-Spring的BeanUtil](https://blog.csdn.net/zsx157326/article/details/77693220)

---
具体方法具体场景各自选择。 END.