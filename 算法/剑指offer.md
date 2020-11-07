## 剑指offer

#### 64

求1 + ... + n，不能使用循环语句和判断语句

**题解：**

使用递归

```java
public int sumNums(int n) {
    if(n == 1) return 1;
    return sumNums(n-1) + n;
}
```

​	

#### 44

数字以0123456789101112131415…的格式序列化到一个字符序列中。在这个序列中，第5位（从下标0开始计数）是5，第13位是1，第19位是4

**题解：**

最容易想到的方法就是用一个StringBuilder一直append，然后charAt；但是这种方法空间复杂度太高了，数字一大就内存溢出。

* 将每一位称为 数位 ，记为 n ；

* 将 10,11,12,⋯ 称为 数字 ，记为 num；

* 数字 1010 是一个两位数，称此数字的 位数 为 22 ，记为 digit；

* 每 digit 位数的起始数字（即：1, 10, 100，⋯），记为 start

![](../img/剑指offer/44.png)



```java
public int findNthDigit(int n) {
        if(n == 0) return 0;
        int digit = 1;       // 位数
        long start = 1;      // 用long类型，n的范围[0, 2^32)
        long cnt = 9;		 // 用long类型
        while(n > cnt) {
            n -= cnt;
            digit++;
            start *= 10;
            cnt = 9 * start * digit;
        }
        long num = start + (n - 1) / digit;	   // 起始值 + 偏移量;  偏移量：[(n-1)/digit]，找到对应的值
        return Long.toString(num).charAt((n - 1) % digit) - '0';  // 根据(n-1)%digit找到该值中对应的位置
    }
```

