---
title: Java小知识点整理（二）
date: 2018-07-04 14:55:24
categories: [开发,总结]
tags: [Java]
---
有一段时间没有整理了，今天整理一下最近看JDK1.8源码发现的几个不错的function。For Java.

---

## Base64加解密
```
    /**
     * Base64加解密
     * 
     * @throws UnsupportedEncodingException
     */
    static void Base64Test() throws UnsupportedEncodingException {
        String str = "Base64加密测试";
        byte[] enByt = Base64.getEncoder().encode(str.getBytes());
        String enStr = new String(enByt, "utf-8");
        System.out.println("加密后 ：" + enStr);

        byte[] decByt = Base64.getDecoder().decode(enStr);
        String decStr = new String(decByt, "utf-8");
        System.out.println("解密后 ：" + decStr);
    }
```
JDK1.8中util里新增了Base64文件，给我们提供了便捷。上述代码结果为：
```
加密后 ：QmFzZTY05Yqg5a+G5rWL6K+V
解密后 ：Base64加密测试
```

## rotate
在看Collections源码时发现的，觉得很妙就记录了下来。
```
private static <T> void rotate1(List<T> list, int distance) {
        int size = list.size();     // 获取集合大小
        if (size == 0)      // 数量为0直接返回
            return;
        distance = distance % size;     // 取模
        if (distance < 0)
            distance += size;       // 保证不为负数
        if (distance == 0)
            return;
        for (int cycleStart = 0, nMoved = 0; nMoved != size; cycleStart++) {
            T displaced = list.get(cycleStart);
            int i = cycleStart;
            do {
                i += distance;      // 依次往后递推
                if (i >= size)
                    i -= size;      // 一轮赋值后回到原处
                displaced = list.set(i, displaced);     // 赋值并返回旧值
                nMoved ++;      // 重新赋值一个数加一
            } while (i != cycleStart);      // 直到回到初值
        }
    }
```
看到这个rotate1那肯定其中还有个2咯。
```
    private static void rotate2(List<?> list, int distance) {       // {1，2，3，4}  2
        int size = list.size();     // 4
        if (size == 0)
            return;
        int mid =  -distance % size;
        if (mid < 0)
            mid += size;        // 2
        if (mid == 0)
            return;

        reverse(list.subList(0, mid));      // {2，1，3，4}
        reverse(list.subList(mid, size));       // {2，1，4，3}
        reverse(list);      // {3，4，1，2}
    }
```
这两个方法效果一样，妙不可言啊，一起分享。

## tableSizeFor
这个是HashMap源码中的一个方法，用来获取一个大于等于该数的二次幂数同时要小于等于给定的最小的二次幂数。
```
    /**
     * 返回给定目标容量的2大小的幂
     */
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;       // 无符号右移，忽略符号位，空位都以0补齐
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

## 插入排序 insertion sort
这个是在DualPivotQuicksort源码中看到抽取出来的，写法很简洁，精炼。
```
int[] arr = {5,1,7,2,4};
for(int i = 0,j = i; i < arr.length - 1; j = ++i) {
    // 保持i 和 j 相等
    int a = arr[i + 1];
    while(a < arr[j]) { // 直到找到没有比它还小的值
        // 如果小于前一个数则将前一个数赋值给它
        arr[j + 1] = arr[j];
        if(j-- == 0) {
            // 执行j-- 若j=0则退出循环
            break;
        }
    }
    arr[j + 1] = a; // 将较小的值赋值
}
System.out.println(Arrays.toString(arr));
```
输出结果为：
```
[1,2,4,5,7]
```

---
End.