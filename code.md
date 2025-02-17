# code

## 时间复杂度
1. 时间复杂度是一个函数,它定性描述该算法的运行时间
2. 大O
   1. 大O用来表示上界的,当用它作为算法的最坏情况运行时间的上界
   2. 面试中说道算法的时间复杂度是多少指的都是一般情况
3. 我们说的时间复杂度都是省略常数项系数的,是因为一般情况下都是默认数据规模足够的大
4. O(1)常数阶 < O(logn)对数阶 < O(n)线性阶 < O(n^2)平方阶 < O(n^3)立方阶 < O(2^n)指数阶
5. O(logn)中的log都是忽略底数的
6. 求x的n次方,来逐步分析递归算法的时间复杂度
   1. 并不是采用递归就是O(logn),往往是O(n)
   2. 采用对半处理,可以达到O(logn)
7. 斐波那契数列
   1. ![](https://pic1.imgdb.cn/item/67ac6474d0e0a243d4fe986a.png)
   2. 传统递归 时间复杂度O(2^n)
   3. 优化递归,减少递归次数后 时间复杂度:O(n)
8. ![](https://pic1.imgdb.cn/item/67ac6474d0e0a243d4fe9869.png)

## 空间复杂度
1. 是对一个算法在运行过程中占用内存空间大小的量度

## 数组
1. 数组是存放在连续内存空间上的相同类型数据的集合
2. 二分查找
   1. https://leetcode.cn/problems/binary-search/
3. 移除元素
   1. https://leetcode.cn/problems/remove-element/
   2. 快慢指针-双指针-遍历该序列至多两次
   3. 双指针优化-避免了需要保留的元素的重复赋值操作-遍历该序列至多一次
4. 有序数组的平方
   1. https://leetcode.cn/problems/squares-of-a-sorted-array/
   2. 方法一:先平方再排序Arrays.sort
   3. 方法二:非递增,平方后收尾最大-首尾进行比较-双指针
5. 长度最小的子数组
   1. https://leetcode.cn/problems/minimum-size-subarray-sum/description/
   2. 不能排序后,从后开始
   3. 方法一:2个for循环,时间复杂度n方,但会超时
   4. 方法二:滑动窗口,其实也是双指针
6. 螺旋矩阵
   1. https://leetcode.cn/problems/spiral-matrix-ii/description/
   2. 按照最外层作为一个圈,完成后进行下一个圈
   3. 如果n是奇数,则单独处理中心点
   4. 衍生-https://leetcode.cn/problems/spiral-matrix/submissions
7. 区间和
   1. https://kamacoder.com/problempage.php?pid=1070
   2. 前缀合
8. 开发商购买土地
   1. https://kamacoder.com/problempage.php?pid=1044
   2. 前缀合

## 链表
1. 移除链表元素
   1. https://leetcode.cn/problems/remove-linked-list-elements/description/
   2. 定义一个虚拟头节点
2. 设计链表
   1. https://leetcode.cn/problems/design-linked-list/description/
   2. ListNode定义,构造函数,维护size
   3. 单向链表
   4. 双向链表
3. 翻转链表
   1. https://leetcode.cn/problems/reverse-linked-list/description/
   2. 迭代-需要一个临时节点用于中转
   3. 递归-需要明确返回的条件与两两交换指向
4. 两两交换链表中的节点
   1. https://leetcode.cn/problems/swap-nodes-in-pairs/description/
   2. 迭代法-临时节点和交换需要的第一个节点和第二个节点
   3. 递归
5. 删除链表的倒数第N个节点
   1. https://leetcode.cn/problems/remove-nth-node-from-end-of-list/description/
   2. 遍历2次-第一次获得size,第二步删除元素
   3. 快慢指针-fast先走n+1步,然后和slow一起走到结尾
6. 链表相交
   1. https://leetcode.cn/problems/intersection-of-two-linked-lists-lcci/description/
   2. 方法一-需要注意的是比较的不是val,而是整个节点,相当于val和next,使用哈希Set<ListNode>,先存链表A,再去遍历链表B比较
   3. 方法二-快慢指针-先行移动长链表实现同步移动
   4. 方法三-快慢指针-合并链表-A和B分别移动,结束了则从另外一个的头开始
7. 环形链表II
   1. https://leetcode.cn/problems/linked-list-cycle-ii/description/
   2. 方法一-使用哈希Set存储
   3. 方法二-快慢指针
      1. 快走2步 慢走1步
      2. 如果没有环,则循环结束
      3. 如果有环,快指针追上慢指针
      4. 再从头节点开始,寻找入环点

## 哈希表
1. 有效的字母异位词
   1. https://leetcode.cn/problems/valid-anagram/
   2. 利用现有函数-转数组s.toCharArray()-排序Arrays.sort(str1)-判断相等Arrays.equals(str1, str2)
   3. 根据26个字母通过数组遍历字符串,判断数量是否一致
   4. 解决Unicode-构建一个Map<Character, Integer>
2. 两个数组的交集
   1. https://leetcode.cn/problems/intersection-of-two-arrays/description/
   2. 使用HashSet
3. 快乐数
   1. https://leetcode.cn/problems/happy-number/description/
   2. 通过hash判断该数是否循环了-部分数字会无限循环
   3. 快慢指针-可以转换为判断是否是环形链表
4. 两数之和
   1. https://leetcode.cn/problems/two-sum/description/
   2. 
5. 四数相加II
6. 赎金信
7. 三数之和
8. 四数之和

## 字符串

## 双指针法

## 栈与队列

## 二叉树

## 回溯算法

## 贪心算法

## 动态规划

## 排序
  
 
