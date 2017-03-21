---
title: Add Binary
categories: Leetcode
tags:
- Leetcode
- C/C++
- Python
- Algorithm
---

# 描述
[67. Add Binary](https://leetcode.com/problems/add-binary/)

Given two binary strings, return their sum (also a binary string).
 
For example,
a = "11"
b = "1"
Return "100".

# 题意
 略
 
# 分析
略

# 题解
[\[C/C++\]](https://github.com/lightmen/leetcode/blob/master/c/string/add-binary.c):
```
char* addBinary(char* a, char* b) {
    char *ret;
    int la,lb,lr;
    int i,j,k;
    int carry = 0;
    int value;

    la = strlen(a);
    lb = strlen(b);
    lr = (la > lb ? la : lb) + 2;

    ret = (char *)malloc(sizeof(char) * lr);
    ret[lr-1] = 0;
    k = lr-2;
    i = la - 1;
    j = lb - 1;
    while(i >= 0 && j >= 0){
        value = (a[i--] - '0') + (b[j--] - '0')  + carry;
        carry = value / 2;
        value %= 2;
        ret[k--] = '0' + value;
    }

    if(j >= 0){
        i = j;
        a = b;
    }

    while(i >= 0){
        value = (a[i--] - '0') + carry;
        carry = value / 2;
        value %= 2;
        ret[k--] = '0' + value;
    }

    if(carry)
        ret[k--] = '1';

    return ret + k + 1;
}
```

[\[Python\]](https://github.com/lightmen/leetcode/blob/master/python/string/add-binary.py):
```
class Solution(object):
    def addBinary(self, a, b):
        """
        :type a: str
        :type b: str
        :rtype: str
        """
        indexa = len(a) - 1
        indexb = len(b) - 1
        res = ''
        carry = 0
        while indexa >= 0 or indexb >= 0:
            x = int(a[indexa]) if indexa >= 0 else 0
            y = int(b[indexb]) if indexb >= 0 else 0
            value = x + y + carry
            carry = value // 2
            value = value % 2
            res = str(value) + res
            indexa, indexb = indexa - 1, indexb - 1

        if carry == 1:
            res = '1' + res

        return res
```


