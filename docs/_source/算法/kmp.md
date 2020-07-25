
## 题目链接

[28\. 实现 strStr()](https://leetcode-cn.com/problems/implement-strstr/)

## 题目描述

Difficulty: **简单**


实现  函数。

给定一个 haystack 字符串和一个 needle 字符串，在 haystack 字符串中找出 needle 字符串出现的第一个位置 (从0开始)。如果不存在，则返回  **-1**。

**示例 1:**

```
输入: haystack = "hello", needle = "ll"
输出: 2
```

**示例 2:**

```
输入: haystack = "aaaaa", needle = "bba"
输出: -1
```

**说明:**

当 `needle` 是空字符串时，我们应当返回什么值呢？这是一个在面试中很好的问题。

对于本题而言，当 `needle` 是空字符串时我们应当返回 0 。这与C语言的  以及 Java的  定义相符。


## Solution


```java
private int[] getNext(String str) {
        int len = str.length();
        int[] next = new int[len];
        next[0] = -1;
        int j = 0, k = -1;
        while (j < len - 1) {
            // 新移动的位置能不能匹配上
            if (k == -1 || str.charAt(j) == str.charAt(k)) {
                // 新的位置，str.charAt(j) 和 str.charAt(k) 是下一轮要判断的
                j++;
                k++;
                // k “前面”是公共前缀；
                // 如果外界的 haystack 匹配不上，回到 k 的位置继续查看能不能匹配 （这里的 j 和 k 都是新位置的 j 和 kz）
                next[j] = k;
            } else {
                // 看看公共前缀 自己有没有公共前缀
                k = next[k];
            }
        }
        return next;
    }

    public int strStr(String haystack, String needle) {
        if(needle.isEmpty()) return 0;

        //源串
        int i = 0;
        //字串
        int j = 0;
        int len1 = haystack.length();
        int len2 = needle.length();
        int[] next = getNext(needle);
        while (i < len1 && j < len2) {
            if (j == -1 || haystack.charAt(i) == needle.charAt(j)) {
                i++;
                j++;
            } else {
                // 获取下一次匹配的位置
                // 变化的是 next 数组上的指针
                j = next[j];
            }
        }
        if (j == len2) {
            return i - j;
        }
        return -1;
    }
```