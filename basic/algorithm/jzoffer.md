#### 剑指 Offer 03. 数组中重复的数字

链接：https://leetcode.cn/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/

> 1. 直接用哈希表 `Set` 可以做。
> 2. 直接原地交换，将值放入对应的索引位置。如果索引位置有相同的值，则返回。（其本质也是`hash`，**推荐**，速度更快）

#### 剑指 Offer 04. 二维数组中的查找

`M` 链接：https://leetcode.cn/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof

> 从右上角开始，向左（当前值大于目标值）和向下（当前值小于目标值）遍历，进行判断。

#### 剑指 Offer 05. 替换空格

链接：https://leetcode.cn/problems/ti-huan-kong-ge-lcof/

> 1. 直接用 `s.replace(" ", "%20");`
> 2. 第一遍查找空格个数，然后扩容字符串数组，然后倒着遍历，遇到空格填充进去。

#### 剑指 Offer 06. 从尾到头打印链表

链接：https://leetcode.cn/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/

> 1. 使用 list，先放到 list，再从末尾取放进数组。
> 2. 使用栈（更慢）。

#### 剑指 Offer 07. 重建二叉树

`M`

> TODO

#### 剑指 Offer 09. 用两个栈实现队列

链接：https://leetcode.cn/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/

> A 栈负责入栈，B 栈为空时，将 A 栈 倒进 B栈，再从B栈弹出

#### 剑指 Offer 10- I. 斐波那契数列

链接：https://leetcode.cn/problems/fei-bo-na-qi-shu-lie-lcof/

> 1. 完全可以递归，但是容易栈溢出。
> 2. 使用两个变量标记 `n-1` 和 `n-2` 的值，循环计算 `n` 的值。

#### 剑指 Offer 10- II. 青蛙跳台阶问题

链接：https://leetcode.cn/problems/qing-wa-tiao-tai-jie-wen-ti-lcof

> 1. 也可以递归，但容易栈溢出。
> 2. 方法同斐波那契数列。

#### 剑指 Offer 12. 矩阵中的路径

`M` 链接：https://leetcode.cn/problems/ju-zhen-zhong-de-lu-jing-lcof

> DFS，常规深搜题目。

#### 剑指 Offer 14- I. 剪绳子

`M` 链接：https://leetcode.cn/problems/jian-sheng-zi-lcof/

> 长度大于 4 时，尽可能多的剪成长度为 3 的段。

#### 剑指 Offer 14- II. 剪绳子 II

`M` 链接：https://leetcode.cn/problems/jian-sheng-zi-ii-lcof/

> 当长度大于 4 时，尽可能多的剪长度为 3 的绳子。多了中间求积是的取模。

#### 剑指 Offer 15. 二进制中1的个数

链接：[https://leetcode.cn/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/](https://leetcode.cn/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/)

```java
i = i - ((i >>> 1) & 0x55555555);
i = (i & 0x33333333) + ((i >>> 2) & 0x33333333);
i = (i + (i >>> 4)) & 0x0f0f0f0f;
i = i + (i >>> 8);
i = i + (i >>> 16);
return i & 0x3f;
```

#### 剑指 Offer 16. 数值的整数次方

`M` 链接：https://leetcode.cn/problems/shu-zhi-de-zheng-shu-ci-fang-lcof/

> 使用**快速乘**，如下。

```java
double quickMul(double x, long n) {
    double ans = 1.0;
    double x2 = x;
    while (n > 0) {
        if (n % 2 == 1) {
            ans *= x2;
        }
        x2 *= x2;
        n /= 2;
    }
    return ans;
}
```

#### 剑指 Offer 17. 打印从1到最大的n位数

链接：[https://leetcode.cn/problems/da-yin-cong-1dao-zui-da-de-nwei-shu-lcof/](https://leetcode.cn/problems/da-yin-cong-1dao-zui-da-de-nwei-shu-lcof/)

> 使用**快速幂**，如下。然后至最大长度为 `n` 的数字，有 `n-1` 个。

```java
int quickPow(int a, int b){
    int ans = 1;
    while(b > 0){
        if((b & 1) == 1){
            ans = ans * a;
        }
        a = a * a;
        b = b >> 1;
    }
    return ans;
}
```

#### 剑指 Offer 18. 删除链表的节点

链接：https://leetcode.cn/problems/shan-chu-lian-biao-de-jie-dian-lcof

> 简单删除即可。





#### 剑指 Offer 20. 表示数值的字符串

`M` 链接：https://leetcode.cn/problems/biao-shi-shu-zhi-de-zi-fu-chuan-lcof/

> 遍历模拟，使用三个标记，记录是否出现num、dot、E|e，边界条件进行判断。

#### 剑指 Offer 21. 调整数组顺序使奇数位于偶数前面

链接：https://leetcode.cn/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/

> 双指针。



#### 剑指 Offer 24. 反转链表

链接：[https://leetcode.cn/problems/fan-zhuan-lian-biao-lcof/](https://leetcode.cn/problems/fan-zhuan-lian-biao-lcof/)

> 当链表长度为 0 1 2 时，单独处理
>
> 当链表长度大于 2 时，定义三个指针：pre, cur, next，pre负责指向反转后的链表的新的头结点，cur负责待反转的链表的第一个值，next负责指向cur的下一个待移动的指针（需要保证next不为空，才能 next = next.next）

#### 剑指 Offer 25. 合并两个排序的链表

链接：https://leetcode.cn/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/

> 设置一个辅助的头结点，将两个链表追加至新的头结点上。最后返回头结点的next。

#### 剑指 Offer 26. 树的子结构

`M` 链接：https://leetcode.cn/problems/shu-de-zi-jie-gou-lcof/

> DFS 即可。

#### 剑指 Offer 27. 二叉树的镜像

链接：https://leetcode.cn/problems/er-cha-shu-de-jing-xiang-lcof/

> 递归即可。

#### 剑指 Offer 28. 对称的二叉树

链接：https://leetcode.cn/problems/dui-cheng-de-er-cha-shu-lcof/

> DFS 即可。









#### 剑指 Offer 30. 包含min函数的栈

链接：[https://leetcode.cn/problems/bao-han-minhan-shu-de-zhan-lcof/](https://leetcode.cn/problems/bao-han-minhan-shu-de-zhan-lcof/)

> 题目要求调用 min、push 及 pop 的时间复杂度都是 O(1)，那就可以通过空间换时间的思路。
>
> 通过两个量：stack和min，在stack中push每个值之前，压入历史数据的最小值min，再放入新值x、更新最小值标记min。





#### 剑指 Offer 31. 栈的压入、弹出序列

`M` 链接：https://leetcode.cn/problems/zhan-de-ya-ru-dan-chu-xu-lie-lcof/

> 模拟题，两个指针，一个栈模拟，将push元素入栈。如果栈顶元素与pop元素一致，则循环弹出。

#### 剑指 Offer 32 - I. 从上到下打印二叉树

`M` 链接：https://leetcode.cn/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/

> 数的层序遍历。使用队列存放节点。队列不为空，则取出队头的点，然后将该点的左右孩子入队列。

#### 剑指 Offer 32 - II. 从上到下打印二叉树 II

链接：https://leetcode.cn/problems/cong-shang-dao-xia-da-yin-er-cha-shu-ii-lcof/

> DFS 即可。

#### 剑指 Offer 32 - III. 从上到下打印二叉树 III

`M` 链接：https://leetcode.cn/problems/cong-shang-dao-xia-da-yin-er-cha-shu-iii-lcof/

> Z 字型打印。DFS 即可，用一个 deep 变量保存深度。List中，奇数行头插，偶数行尾插。





#### 剑指 Offer 34. 二叉树中和为某一值的路径

`M` 链接：https://leetcode.cn/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/

> DFS，从根节点到叶子节点即可。

#### 剑指 Offer 35. 复杂链表的复制

`M` 链接：https://leetcode.cn/problems/fu-za-lian-biao-de-fu-zhi-lcof/

> 递归。使用 Map 保存 旧节点-新节点， 重复创建与递归，回溯时创建 random 指针。





#### 剑指 Offer 38. 字符串的排列

`M` 链接：https://leetcode.cn/problems/zi-fu-chuan-de-pai-lie-lcof/

> DFS。通过一个标记，记录每次深搜时，已经被访问的位置。使用set去重。

#### 剑指 Offer 39. 数组中出现次数超过一半的数字

链接：https://leetcode.cn/problems/shu-zu-zhong-chu-xian-ci-shu-chao-guo-yi-ban-de-shu-zi-lcof/

> 摩尔投票法： **票数正负抵消** 。两个变量，当前被选举者，以及被选举者的票数。相同则票数自增，否则自减。票数为0则替换被选举者。

#### 剑指 Offer 40. 最小的k个数

链接：https://leetcode.cn/problems/zui-xiao-de-kge-shu-lcof/

> 常规思路是先排序，然后取前 K 个数。
>
> 当然也可以用大顶堆。
>
> 最优解是使用快排的思想。



#### 剑指 Offer 42. 连续子数组的最大和

链接：https://leetcode.cn/problems/lian-xu-zi-shu-zu-de-zui-da-he-lcof/

> 常规动态规划问题。





#### 剑指 Offer 44. 数字序列中某一位的数字

`M` 链接：https://leetcode.cn/problems/shu-zi-xu-lie-zhong-mou-yi-wei-de-shu-zi-lcof/

> 找规律。可以从1开始找，记录一下每次循环的数字位数，一共多少位等数据，然后再计算所在的数字，以及第几位，即可。



#### 剑指 Offer 45. 把数组排成最小的数

`M` 链接：https://leetcode.cn/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/

> 将数字数组转换成字符串数组，然后以 `(a + b).compareTo(b + a)` 条件排序。





#### 剑指 Offer 47. 礼物的最大价值

`M` 链接：https://leetcode.cn/problems/li-wu-de-zui-da-jie-zhi-lcof/

> 常规 DP 问题。(i,j) 的礼物价值，选左、上的最大者，再求和。

#### 剑指 Offer 48. 最长不含重复字符的子字符串

`M` 链接：[https://leetcode.cn/problems/zui-chang-bu-han-zhong-fu-zi-fu-de-zi-zi-fu-chuan-lcof/](https://leetcode.cn/problems/zui-chang-bu-han-zhong-fu-zi-fu-de-zi-zi-fu-chuan-lcof/)

> 滑动窗口，用set维护一个不重复的窗口，双指针方法，使用一个集合存放左右指针之间的字符。如果当前字符在集合中，则左指针需要右移，直至set中不存在该字符，然后加入当前字符，再计算l 和 r 之间的长度。

#### 剑指 Offer 49. 丑数

`M` 链接：https://leetcode.cn/problems/chou-shu-lcof/

> dp问题。一个丑数通过另一个丑数 * 2 3 5 得到，使用三个指针维护2、3、5的位置， 每次都选取最小的一个。

#### 剑指 Offer 50. 第一个只出现一次的字符

链接：https://leetcode.cn/problems/di-yi-ge-zhi-chu-xian-yi-ci-de-zi-fu-lcof/

> 暴力，两次遍历即可。





#### 剑指 Offer 52. 两个链表的第一个公共节点

链接：https://leetcode.cn/problems/liang-ge-lian-biao-de-di-yi-ge-gong-gong-jie-dian-lcof/

> 两个结点不断的去对方的轨迹中寻找对方的身影，只要二人有交集，就终会相遇。
>
> 人话：只要两个指针不相等，就一直循环，到尾巴后，切换至另一个链表。

#### 剑指 Offer 53 - I. 在排序数组中查找数字 I

链接：https://leetcode.cn/problems/zai-pai-xu-shu-zu-zhong-cha-zhao-shu-zi-lcof/

> 二分。注意边界。

#### 剑指 Offer 53 - II. 0～n-1中缺失的数字

链接：https://leetcode.cn/problems/que-shi-de-shu-zi-lcof/

> 二分，找到不存在的值的坐标。





#### 剑指 Offer 55 - I. 二叉树的深度

链接：https://leetcode.cn/problems/er-cha-shu-de-shen-du-lcof/

> 常规DFS。









#### 剑指 Offer 56 - I. 数组中数字出现的次数

`M` 链接：[https://leetcode.cn/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/](https://leetcode.cn/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/)

> 需要遍历3次：
>
> 第一次，找到所有的两个数异或后的值z
>
> 第二次，找到其中某个值（x）的特征位，
>
> 第三次，再 与，和特征位相与后是0，说明是x，否则是y，再与所有的值 异或

#### 剑指 Offer 56 - II. 数组中数字出现的次数 II

`M` 链接：[https://leetcode.cn/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-ii-lcof](https://leetcode.cn/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-ii-lcof)

> 位运算的思想。通过一个32位长的数组，记录所有数的二进制中，各个位的1 的个数。然后各个数值除以三取余，余数即为多的那个数的1。

#### 剑指 Offer 57. 和为s的两个数字

链接：https://leetcode.cn/problems/he-wei-sde-liang-ge-shu-zi-lcof/

> 双指针问题。等于目标值返回，小于目标值左侧自增，大于目标值右侧自减。

#### 剑指 Offer 57 - II. 和为s的连续正数序列

链接：https://leetcode.cn/problems/he-wei-sde-lian-xu-zheng-shu-xu-lie-lcof/

> 可以用双指针， 也可以用数学等差数列。

#### 剑指 Offer 58 - I. 翻转单词顺序

链接：

> 可以用双指针。
>
> 也可以用正则表达式，将其去掉首位空格后，按空格分割：`s.trim().split("\s+");`





#### 剑指 Offer 61. 扑克牌中的顺子

链接：https://leetcode.cn/problems/bu-ke-pai-zhong-de-shun-zi-lcof/

> 最大的牌减去最小的牌，插值不能大于等于5，同时不允许有重复（0 除外）。

#### 剑指 Offer 62. 圆圈中最后剩下的数字

链接：https://leetcode.cn/problems/yuan-quan-zhong-zui-hou-sheng-xia-de-shu-zi-lcof/

> **约瑟夫环**。公式：`f(N,M)=(f(N−1,M)+M)%N`，其中`N`是队列的人数，报`M`的人出队。
>
> + `f(1, M) = 0`; 即只有1 个人时，直接胜出，下标为 0
> + `f(2, M) = ( f(1,M) + M ) % 2` ;
> + `f(3, M) = ( f(2,M) + M ) % 3` ;
> + ……

#### 剑指 Offer 63. 股票的最大利润

`M` 链接：[https://leetcode.cn/problems/gu-piao-de-zui-da-li-run-lcof/](https://leetcode.cn/problems/gu-piao-de-zui-da-li-run-lcof/)

> 遍历数组，维护两个变量：min 和 ans，如果当前值小于min，写则更新min，否则，更新ans（当前值减去min，即为最大差）





#### 剑指 Offer 66. 构建乘积数组

`M` 链接：https://leetcode.cn/problems/gou-jian-cheng-ji-shu-zu-lcof/

> 由于不能用乘法，就分成两个三角形：右上三角和左下三角，分开累乘。

