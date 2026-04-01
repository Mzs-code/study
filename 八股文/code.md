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
   2. 使用hash避免重复查找
5. 四数相加II
   1. https://leetcode.cn/problems/4sum-ii/
   2. 将4数相加转换为2数相加-使用map存储
6. 赎金信
   1. https://leetcode.cn/problems/ransom-note/description/
   2. 使用map
   3. 也可以属于26个字母的数组
7. 三数之和
   1. https://leetcode.cn/problems/3sum/description/
   2. 先排序,然后使用快慢指针,同时需要注意判重
8. 四数之和
   1. https://leetcode.cn/problems/4sum/
   2. 先排序,再使用快慢指针
   3. 和三数之和一样,只是多了一个for
   4. 第一个for是第一个数字
   5. 第二个for是第二个数字
   6. 剩下的是left和right
   7. target是负数时 需要2个条件都满足才能剪枝
   8. nums[i] > target && nums[i] >= 0

## 字符串
1. 反转字符串
   1. https://leetcode.cn/problems/reverse-string/description/
   2. 收尾交换,逐步缩进
2. 反转字符串II
   1. https://leetcode.cn/problems/reverse-string-ii/description/
   2. for循环时候使用i += 2 * k
   3. 再定义start = i end = min(i + k - 1, s.length() - 1) 交换
3. 替换数字
   1. https://kamacoder.com/problempage.php?pid=1064
   2. 先计算计算数字个数,创建新的长度的数组
   3. 原有数组复制
   4. 从后往前判断,因为从前往后的会,每次都要重新调整后续数组的顺序,时间复杂度会变为n的2次方
   5. 部分方法,判断是否是数字Character.isDigit()
4. 翻转字符串里的单词
   1. https://leetcode.cn/problems/reverse-words-in-a-string/description/
   2. 利用现有函数
      1. 去除首尾空格 s.trim()
      2. 去除中间多余空格,并转为集合 Arrays.asList(s.split("\\s+"))
      3. 翻转 Collections.reverse(list);
      4. 拼接 String.join(" ", list)
   3. 自行实现
      1. 去除首尾空格-整个翻转-单个翻转
      2. StringBuilder有一个setCharAt方法(index, value)
   4. 使用双端队列 可以直接遍历后塞入头部
      1. Deque<String> d = new ArrayDeque<>() d.offsetFirst(sb.toString()) sb.setLength(0)
   5. 关联题-https://leetcode.cn/problems/length-of-last-word/description/
      1. 从尾部开始关联
5. 右旋字符串 
   1. https://kamacoder.com/problempage.php?pid=1065
   2. 无论是左旋还是右旋,都可以通过多次翻转完成
   3. 也可以用substring切片
   4. 先整体翻转,再前后单独翻转
   5. 关联题-https://leetcode.cn/problems/zuo-xuan-zhuan-zi-fu-chuan-lcof/description/
6. 实现 strStr()
   1. https://leetcode.cn/problems/find-the-index-of-the-first-occurrence-in-a-string/description/
   2. 基于滑动窗口-不断匹配-不匹配则删除头部
      1. 使用StringBuilder的append与deleteCharAt(0)-时间复杂度O((m - n) * n)
      2. 直接使用数组比较--时间复杂度O(m * n)
   3. KMP-时间复杂度O(m + n)
      1. 通过构建next数组存储相同的前缀和的位置
      2. 在匹配失败时候,可以跳过一些不可能匹配的case,加速匹配效率
      3. 可以在原有字符串前加" ",来让下标从1开始
      4. 先构建next[], 相等则让j++,同时在next[]中记录下来
      5. 不相等,则让j往前找相同的
      6. 匹配时也是一样,不匹配时往前找最近的相同前缀
      7. 如果j的长度相等,则说明已经完全匹配,结果为i - m;
7. 重复的子字符串
   1. https://leetcode.cn/problems/repeated-substring-pattern/description/
   2. 枚举法-时间复杂度：O(n平方)
      1. 如果一个长度为n的字符串s可以由它的一个长度为n1的子串s1重复多次构成
      2. 那么n肯定是n1的倍数
      3. s1是s的前缀
      4. s[i]=s[i−n1]
   3. 利用contains方法
      1. s = s + s;
      2. 去除首尾
      3. 判断contains
   4. KMP
      1. 还是构建next数组
      2. 判断条件(next[length] > 0 && length % (length - next[length]) == 0)

## 双指针法

## 栈与队列
1. 用栈实现队列
   1. https://leetcode.cn/problems/implement-queue-using-stacks/description/
   2. 模拟题
   3. 通过定义2个栈,分别用于入栈和出栈
   4. 出栈如果为空,则从入栈中转移到出栈中
2. 用队列实现栈
   1. https://leetcode.cn/problems/implement-stack-using-queues/
   2. 需要注意的Deque接口的几个方法
      1. 队列
         1. offer-新增元素在最后一个
         2. poll-取出第一个元素,并删除
         3. peek-取出第一个元素,不删除
      2. 栈
         1. push-新增元素在第一个
         2. pop-取出第一个元素,并删除
3. 有效的括号
   1. https://leetcode.cn/problems/valid-parentheses/description/
   2. 除了完全对称的case,还有()[]{}
   3. 需要构建一个map用于映射对应符号
4. 删除字符串中的所有相邻重复项
   1. https://leetcode.cn/problems/remove-all-adjacent-duplicates-in-string/description/
5. 逆波兰表达式求值
   1. https://leetcode.cn/problems/evaluate-reverse-polish-notation/description/
   2. leetcode 内置jdk的问题，不能使用==判断字符串是否相等
   3. 逆波兰表达式由波兰的逻辑学家卢卡西维兹提出.逆波兰表达式的特点是：没有括号，运算符总是放在和它相关的操作数之后
   4. 遇到数字则入栈；遇到算符则取出栈顶两个数字进行计算，并将结果压入栈中
6. 滑动窗口最大值
   1. https://leetcode.cn/problems/sliding-window-maximum/description/
   2. 单调队列
   3. 队列内不用存储窗口内的所有元素,只需要维护有可能成为最大的元素就行
      1. poll-移除元素-如果当前值是最大值才移除
      2. add-添加元素-从末尾开始比较
      3. peek-最大元素
7. 前 K 个高频元素
   1. https://leetcode.cn/problems/top-k-frequent-elements/description/
   2. 先使用map统计
   3. 再注入到优先级队列中-定义时约定了排序方式
   4. 再取k个
   5. 小顶堆-从小到大
   6. 大顶堆-从大到小
   7. 方法二-也可以频次放在数组内,然后倒序获取

## 二叉树

## 回溯算法

## 贪心算法

## 动态规划

## 排序
  
 
