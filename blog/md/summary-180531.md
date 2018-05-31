---
title: Java中一些小知识点
date: 2018-5-31 18:04:46
categories: [开发,总结]
tags: [Java]
---
今天来总结一些平时不起眼的东西。For Java.

---
## for循环进行遍历删除元素
```
ArrayList<String> list = new ArrayList(Arrays.asList("a", "b", "c", "d"));
        System.out.println("before :" + list);
        // 在这个方法中有一个严重的错误。当一个元素被删除时，列表的大小缩小并且下标变化，
        // 所以当你想要在一个循环中用下标删除多个元素的时候，它并不会正常的生效。
        for (int i = 0; i < list.size(); i++) {
            String s = list.get(i);
            System.out.println("s :" + s);
            if ("a".equals(s)) {
                list.remove(s);
            }
            if ("b".equals(s)) {
                list.remove(s);
            }
        }
        System.out.println("after :" + list);
``` 
上述代码结果不尽人意：

```
before :[a, b, c, d]
s :a
s :c
s :d
after :[b, c, d]
```
这正是因为注释中说的那样，list.remove()操作完的时候list就已经改变了大小以及下标，故当"a"删除后list变成了`["b","c","d"]`,然而下标变成了1，这时候取值就是变成了"c",所以"b"这个元素就自然跳过了。
**故我们在进行遍历删除list中元素时要使用Iterator迭代器来操作**

## 变量声明
今天自己写了几行简单测试了下：
##### 1.
```
System.out.println("----------------start------------------");
        int num = 100000;
        long start = System.currentTimeMillis();
        for (int i = 0; i < 9999999; i++) {
            num += 20000;
        }
        long end = System.currentTimeMillis();
        System.out.println("时间差  >> :" + (end - start));

```
结果为：
```
----------------start------------------
时间差  >> :2
```
##### 2.
```
System.out.println("----------------start------------------");
        Integer num = 100000;
        long start = System.currentTimeMillis();
        for (int i = 0; i < 9999999; i++) {
            num += 20000;
        }
        long end = System.currentTimeMillis();
        System.out.println("时间差  >> :" + (end - start));

```
结果为：
```
----------------start------------------
时间差  >> :20
```
结果差了整10倍。想必数据一大自动拆装箱也挺累的吧。**故之后变量要多次基本操作声明尽量不用包装类**

## String相关
- intern()
```
String str = "sss";
String str1 = new String("sss");
String str2 = str1;

System.out.println("str2.intern() == str1 >>" + (str2.intern() == str1));
System.out.println("str2.intern() == str3 >>" + (str2.intern() == str));
System.out.println("str2.intern().equals(str1) >>" + str2.intern().equals(str1));
System.out.println("str2 == str1 >>" + (str2 == str1));
System.out.println("str2.equals(str1) >>" + str2.equals(str1));
```
结果：
```
str2.intern() == str1 >>false
str2.intern() == str3 >>true
str2.intern().equals(str1) >>true
str2 == str1 >>true
str2.equals(str1) >>true
```
我个人理解是，str2.intern()即是str1.intern()指向的故是常量池中的"sss",常量池中的"sss"就是str,str1指向一个String对象，而equals比较大小就不说了。这[里面](https://www.zhihu.com/question/28916657)说得就详细多了。

- new String
```
char[] c = { '1', '2', '3', '4', '5', '6', '7', '8' };
String strC = new String(c, 2, 3);
System.out.println("strC >>" + strC);

int[] arrInt = { 1, 2, 3, 4, 5, 6, 7, 8 };
String strI = new String(arrInt, 2, 3);
System.out.println("strI >>" + strI);
```
结果：
```
strC >>345
strI >>
```
从下标2位置（包含）开始截取，数量为3

- join
```
String joinS = String.join(":", "e", "r", "y");
System.out.println("joinS >>" + joinS);
```
结果：
```
joinS >>e:r:y
```
我们走进join源码看一下：
```
public static String join(CharSequence delimiter, CharSequence... elements) {
        Objects.requireNonNull(delimiter);
        Objects.requireNonNull(elements);
        // Number of elements not likely worth Arrays.stream overhead.
        StringJoiner joiner = new StringJoiner(delimiter);
        for (CharSequence cs: elements) {
            joiner.add(cs);
        }
        return joiner.toString();
    }
```
发现了StringJoiner

- StringJoiner

所以也可以这么用：
```
StringJoiner joiner = new StringJoiner(",");
joiner.add("a").add("b").add("c");
System.out.println(joiner.toString());
```
结果：
```
a,b,c
```

---
Today End.