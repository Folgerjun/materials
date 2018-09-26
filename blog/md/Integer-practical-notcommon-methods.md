---
title: 整理一些 JDK 中 Integer 实用但不常用的方法
date: 2018-9-26 15:14:30
categories: [开发,总结]
tags: [Java]
---

直接开搞。

## toString
该方法进行了重载，一种是 `toString(int i, int radix)`，另一个是 `toString(int i)`。一个参数的方法就相当于 `toString(int i, 10)`，看代码便知，何况其官网注释也有：
```
    public static String toString(int i, int radix) {
        if (radix < Character.MIN_RADIX || radix > Character.MAX_RADIX)
            radix = 10;

        /* Use the faster version */
        if (radix == 10) {
            return toString(i);
        }
        // int 32位
        char buf[] = new char[33];
        boolean negative = (i < 0);
        int charPos = 32;

        if (!negative) {
            i = -i;
        }
        // 根据进制取余转换
        while (i <= -radix) {
            buf[charPos--] = digits[-(i % radix)];
            i = i / radix;
        }
        buf[charPos] = digits[-i];

        if (negative) {
            buf[--charPos] = '-';
        }

        return new String(buf, charPos, (33 - charPos));
    }
```
**不过该方法需注意：** `If the first argument is negative, the first element of the result is the ASCII minus character '-' ('\u005Cu002D'). If the first argument is not negative, no sign character appears in the result. `例如：
```
Integer.toString(-44, 2)  // -101100
Integer.toBinaryString(-44) // 11111111111111111111111111010100
```
若是负数，用该方法求得的值只是正数前加了个 "-" 。

## toBinaryString
类似的几个方法也一并列出了。 `toBinaryString(int i)` 转二进制方法，`toOctalString(int i)` 转八进制方法，`toHexString(int i)` 转十六进制方法。
```
Integer.toBinaryString(-44) // 11111111111111111111111111010100
Integer.toOctalString(44) // 54
Integer.toHexString(44) // 2c
```

## parseUnsignedInt
与 toString 方法一样进行了重载。 `parseUnsignedInt(String s)` 与 `parseUnsignedInt(String s, int radix)` 这是 JDK 1.8 新增的方法，作用就是将字符串参数解析为第二个参数指定的基数中的无符号整数。
```
Integer.parseUnsignedInt("11111111111111111111111111010100", 2) // -44
Integer.parseUnsignedInt("44", 10) // 44
Integer.parseUnsignedInt("44") // 44
```

## decode
该方法将 String 解码为整数。 接受指定语法的十进制，十六进制和八进制数。源码如下：
```
    public static Integer decode(String nm) throws NumberFormatException {
        int radix = 10;
        int index = 0;
        boolean negative = false;
        Integer result;

        if (nm.length() == 0)
            throw new NumberFormatException("Zero length string");
        char firstChar = nm.charAt(0);
        // Handle sign, if present
        if (firstChar == '-') {
            negative = true;
            index++;
        } else if (firstChar == '+')
            index++;

        // Handle radix specifier, if present
        if (nm.startsWith("0x", index) || nm.startsWith("0X", index)) {
            index += 2;
            radix = 16;
        }
        else if (nm.startsWith("#", index)) {
            index ++;
            radix = 16;
        }
        else if (nm.startsWith("0", index) && nm.length() > 1 + index) { // 0 后面长度要大于 1
            index ++;
            radix = 8;
        }

        if (nm.startsWith("-", index) || nm.startsWith("+", index))
            throw new NumberFormatException("Sign character in wrong position");

        try {
            result = Integer.valueOf(nm.substring(index), radix);
            result = negative ? Integer.valueOf(-result.intValue()) : result;
        } catch (NumberFormatException e) {
            // If number is Integer.MIN_VALUE, we'll end up here. The next line
            // handles this case, and causes any genuine format error to be
            // rethrown.
            String constant = negative ? ("-" + nm.substring(index))
                                       : nm.substring(index);
            result = Integer.valueOf(constant, radix);
        }
        return result;
    }
```
使用测试如下：
```
Integer.decode("0xff") // 255
Integer.decode("#ff") // 255
Integer.decode("-07") // -7
Integer.decode("-071") // -57
```

## highestOneBit
该方法返回一个 int 值，该值最多只有一位，位于指定 int 值中最高位（“最左侧”）1 的位置。 如果指定的值在其二进制补码表示中没有一位，即，如果它等于零，则返回零。
```
Integer.highestOneBit(44) // 32
```
44 对应的二进制为 0010 1100，只选中其最左侧的 “1” 那就是 0010 0000，也就是 2<sup>5</sup> = 32

## lowestOneBit
该方法返回一个 int 值，该值最多只有一位，位于指定 int 值中最低位（“最右侧”）1 的位置。 如果指定的值在其二进制补码表示中没有一位，即，如果它等于零，则返回零。
```
Integer.lowestOneBit(44) // 4
```
44 对应的二进制为 0010 1100，只选中其最右侧的 “1” 那就是 0000 0100，也就是 2<sup>2</sup> = 4

## numberOfLeadingZeros
该方法计算首部零的个数。
```
    /**
     * 首先在 jvm 中一个 int 类型的数据占 4 个字节，共 32 位，其实就相当于一个长度为 32 的数组。
     *
     * 那我们要计算首部 0 的个数，就是从左边第一个位开始累加 0 的个数，直到遇到一个非零值。
     */
    public static int numberOfLeadingZeros(int i) {
        // HD, Figure 5-6
        if (i == 0)
            return 32;
        int n = 1;
        // 下面的代码就是定位从左边开始第一个非零值的位置，在定位过程中顺便累加从左边开始 0 的个数
        // 将 i 无符号右移 16 位后，有二种情况；
        //   情况1. i=0,则第一个非零值位于低 16 位，i 至少有 16 个 0，同时将 i 左移 16 位（把低 16 位移到原高 16 位的位置，这样情况 1 和情况 2 就能统一后续的判断方式）
        //   情况2. i!=0,则第一个非零值位于高 16 位，后续在高 16 位中继续判断
        // 这个思路就是二分查找，首先把32位的数分为高低 16 位，如果非零值位于高 16 位，后续再将高 16 位继续二分为高低 8 位，一直二分到集合中只有 1 个元素
        if (i >>> 16 == 0) { n += 16; i <<= 16; }
        // 判断第一个非零值是否位于高 8 位
        if (i >>> 24 == 0) { n +=  8; i <<=  8; }
        // 判断第一个非零值是否位于高 4 位
        if (i >>> 28 == 0) { n +=  4; i <<=  4; }
        // 判断第一个非零值是否位于高 2 位
        if (i >>> 30 == 0) { n +=  2; i <<=  2; }
        n -= i >>> 31;
        return n;
    }
```
测试看看：
```
Integer.numberOfLeadingZeros(44) // 26
```
int 4 个字节，一个字节八位，所以有 32 位。44 对应完整二进制就是 0000 0000 0000 0000 0000 0000 0010 1100。所以从左边开始数起共有 26 个零。

## numberOfTrailingZeros
返回指定 int 值的二进制补码表达式中最低位（“最右侧”）1 之后的零位数。
```
Integer.numberOfTrailingZeros(44) // 2
```
44 对应二进制 0010 1100。其最右侧 “1” 之后的零的个数就是 2。

## bitCount
返回指定 int 值的二进制补码中 1 的个数。
```
Integer.bitCount(44) // 3
Integer.bitCount(-44) // 28
```
44 对应的二进制补码为 0000 0000 0000 0000 0000 0000 0010 1100。1 有 3 个。<br>
-44 对应的二进制补码为 1111 1111 1111 1111 1111 1111 1101 0100。1 有 28 个。