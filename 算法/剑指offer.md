### 剑指offer

****

#### 64

> 求1 + ... + n，不能使用循环语句和判断语句
>
> **题解：**
>
> 使用递归
>
> ```java
> public int sumNums(int n) {
>     if(n == 1) return 1;
>     return sumNums(n-1) + n;
> }
> ```
>

​				

​				

#### 44

>数字以0123456789101112131415…的格式序列化到一个字符序列中。在这个序列中，第5位（从下标0开始计数）是5，第13位是1，第19位是4
>
>**题解：**
>
>最容易想到的方法就是用一个StringBuilder一直append，然后charAt；但是这种方法空间复杂度太高了，数字一大就内存溢出。
>
>* 将每一位称为 数位 ，记为 n ；
>
>* 将 10,11,12,⋯ 称为 数字 ，记为 num；
>
>* 数字 1010 是一个两位数，称此数字的 位数 为 22 ，记为 digit；
>
>* 每 digit 位数的起始数字（即：1, 10, 100，⋯），记为 start
>
><img src="https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201124234643.png" style="zoom:67%;" />
>
>```java
>public int findNthDigit(int n) {
>        if(n == 0) return 0;
>        int digit = 1;       // 位数
>        long start = 1;      // 用long类型，n的范围[0, 2^32)
>        long cnt = 9;		 // 用long类型
>        while(n > cnt) {
>            n -= cnt;
>            digit++;
>            start *= 10;
>            cnt = 9 * start * digit;
>        }
>        long num = start + (n - 1) / digit;	   // 起始值 + 偏移量;  偏移量：[(n-1)/digit]，找到对应的值
>        return Long.toString(num).charAt((n - 1) % digit) - '0';  // 根据(n-1)%digit找到该值中对应的位置
>    }
>```
>

​				

​				

#### 48

>最长不包含重复字符的子字符串
>
>**题解：**
>
>滑动窗口，其实滑动窗口就是定义两个指针i，j；一定条件下i++，另一条件下j++；
>
>* j指针往右遍历；用一个set集合保存序列
>* 当j指针指向的元素不存在于set集合中，就一直往set集合中添加元素；同时更新maxLength的值，maxLength就是存在过的最长子序列；
>* 如果set集合中存在j指向的元素，就删除i指针指向的元素。直到set集合不存在j指向的元素；
>
>```java
>public int lengthOfLongestSubstring(String s) {
>    int i = 0, j = 0;
>    char[] arr = s.toCharArray();
>    Set<Character> set = new HashSet<>();
>    int maxLength = 0;
>
>    while(j < arr.length) {
>        if(!set.contains(arr[j])) {
>            set.add(arr[j]);
>            maxLength = Math.max(maxLength, set.size());
>        } else {
>            while(set.contains(arr[j])) {
>                set.remove(arr[i]);
>                i++;
>            }
>            set.add(arr[j]);
>        }
>        j++;
>    }
>    return maxLength;
>}
>```
>

​			

​			

**28**

>判断二叉树是否对称
>
>**题解：**
>
>递归的写法，我们以两个节点举例。可以看看节点对称需要什么条件；
>
>* 这两个节点都为空，则肯定是对称的
>
>* 两个节点只有一个为空，则肯定不对称
>
>* 两个节点的值不相等，则肯定不对称；
>
>  ```java
>  public boolean isSymmetric(TreeNode root) {
>      
>  }
>  
>  public boolean helper(TreeNode left, TreeNode right) {
>      if(left == null && right == null) 
>          return true;
>      // 经过上面的if判断之后能保证，两个节点至少有一个不为空
>      if(left == null ^ right == null)	// 异或
>          return false;
>      
>      // 注意：这里一定只能判断两个节点是否不相等，不能判断是否相等，如果判断是相等的会阻止return里面的递归
>      if(left.val != right.val) 
>          return false;		
>  
>      return helper(root.left, root.right) && helper(root.right, root.left);
>  }
>  ```
>
>  ​	
>
>BFS
>
>* BFS的代码看起来回更加苏福
>
>  ```java
>  public boolean isSymmetric(TreeNode root) {
>      if(root == null) return true;
>      Queue<TreeNode> queue = new LinkedList<>();
>      queue.add(root.left);
>      queue.add(root.right);
>      while(!queue.isEmpty()) {
>          TreeNode left = queue.poll();
>          TreeNode right = queue.poll();
>          if(left == null && right == null)
>              continue;
>          if(left == null ^ right == null)
>              return false;
>          if(left.val != right.val)
>              return false;
>          queue.add(left.left);
>          queue.add(right.right);
>          queue.add(left.right);
>          queue.add(right.left);
>      }
>      return true;
>  }
>  ```
>
>  









**没搞懂的题：**

- [ ] Leetcode108
- [ ] Leetcode617