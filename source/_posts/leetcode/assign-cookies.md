---
title: Assign Cookies
categories: Leetcode
tags:
- Leetcode
- C/C++
- Python
- Algorithm
- 贪心算法
---

# 描述：
[455. Assign Cookies](https://leetcode.com/problems/assign-cookies/)

Assume you are an awesome parent and want to give your children some cookies. But, you should give each child at most one cookie. Each child i has a greed factor gi, which is the minimum size of a cookie that the child will be content with; and each cookie j has a size sj. If sj >= gi, we can assign the cookie j to the child i, and the child i will be content. Your goal is to maximize the number of your content children and output the maximum number.

**Note:**
You may assume the greed factor is always positive. 
You cannot assign more than one cookie to one child. 

**Example 1:** 
```
Input: [1,2,3], [1,1] 

Output: 1 

Explanation: You have 3 children and 2 cookies. The greed factors of 3 children are 1, 2, 3. 

And even though you have 2 cookies, since their size is both 1, you could only make the child whose greed factor is 1 content. 

You need to output 1. 
```

**Example 2:**
```
Input: [1,2], [1,2,3] 

Output: 2 

Explanation: You have 2 children and 3 cookies. The greed factors of 2 children are 1, 2. 

You have 3 cookies and their sizes are big enough to gratify all of the children, 

You need to output 2. 
```

# 题意：
假设你是一位很赞的家长想要给孩子一些饼干。但是，你只能至多给每个孩子一个饼干。孩子i的贪婪因子为gi，意思是他所满意的饼干的最小尺寸；每一个饼干j的尺寸为sj。如果sj >= gi，我们就可以把饼干j分给孩子i，然后孩子i会很满意。你的目标是最大化分到饼干的孩子的个数。

**注意：**

可以假设贪婪因子都是正数。
不可以为一个孩子分配多个饼干。

# 分析：
贪心算法

# 题解：
[\[C/C++\]](https://github.com/lightmen/leetcode/blob/master/c/greedy/assign-cookies.c):
```
int cmp(const void *a, const void *b)
{
    return *(int *)a - *(int *)b;
}

int findContentChildren(int* g, int gSize, int* s, int sSize) {
    int i = gSize - 1;
    int j = sSize - 1;
    int ret = 0;

    qsort(g, gSize, sizeof(int), cmp);
    qsort(s, sSize, sizeof(int), cmp);

    while(i >= 0 && j >= 0){
        if(s[j] >= g[i]){
            j--;
            ret++;
        }
        i--;
    }

    return ret;
}
```

[\[Python\]](https://github.com/lightmen/leetcode/blob/master/python/greedy/assign-cookies.py):
```
class Solution(object):
    def findContentChildren(self, g, s):
        """
        :type g: List[int]
        :type s: List[int]
        :rtype: int
        """
        i, j = len(g) - 1, len(s) - 1
        g, s = sorted(g), sorted(s)
        ret = 0

        while min(i, j) >= 0:
            if s[j] >= g[i]:
                ret += 1
                j -= 1

            i -= 1

        return ret
```

