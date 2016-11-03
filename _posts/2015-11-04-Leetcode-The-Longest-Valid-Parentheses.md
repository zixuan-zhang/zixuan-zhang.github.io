---
layout: post
title: LeetCode: The Longest Valid Parentheses
categories: [LeetCode,算法]
tags: [LeetCode,算法]
description: "Blog of Thought"
---

### 题目
> Given a string containing just the characters ‘(‘ and ‘)’, find the length of the longest valid(well-formed) parentheses substring.
For “(()”, the longest valid parentheses substring is “()”, which has length=2.
> Another example is “)()())”, where the longest valid parentheses substring is “()()”, which has length = 4

### 翻译
给一个只包含 ‘(‘ 和 ‘)’两个字符的字符串，找到最长有合法的括号对。并把长度求出来。

### 分析
首先一个问题，什么样的字符串括号对才是合法的？先看几个例子：

* “(”        NO
* “((”       NO
* “()”      Yes
* “()(”     NO
* “())”     NO

从上面的例子来看，合法的括号对必须包含偶数个字符，且左括号和右括号的数量是一样的。再看一组例子：
* “)(”      NO
* “()”      YES

所以，还要满足， 以任意字符处结尾，都必须满足左括号的数量大于等于右括号的数量。因此，合法的字符应该满足以下条件：
1. 字符串必须包含偶数个字符
2. 左括号和右括号的数量相同
3. 以任意字符处结尾的子字符串中，左括号的数量大于等于右括号的数量。这个条件很重要。

### 解法一 暴力法：

分析到这，很容易就想到可以用暴力法。切割子串，然后判断这个子串是不是合法的括号对。切割子串的时间复杂度是O(n^2)，判断子串是否是合法的需要O(n)，因此此方法的时间复杂度是O(n^3)。然后可以在此基础上进行优化：优先切割较长的字符串，子串长度由n到2，然后判断子串是否是合法的，如果是合法的，就不用继续切割了。这种优化方法可以在一定程度上提高效率，但是时间复杂度在理论上没有变化。

### 解法二 动态规划
有没有更好的方法呢？看到最长合法括号对，很容易想到可不可以用动态规划的方法去做这道题。这里不详细介绍动态规划的思想，如果对动态规划不了解，可以先仔细读一下动态规划：从新手到专家。**动态规划算法通常是基于一个递推公式及一个或多个初始状态，当前子问题的解将由上次子问题的解推出。** 那，在这个问题中，这个状态应该是什么呢？我们可以在用一个例子进行分析。当没有思路的时候，最好的方法就是用一个实例来探索一下。我们看这个字符串。
* “()(()”

我们从字符串的begin到end进行遍历，首先第一个字符'(‘，此时肯定不是合法的，再遍历第二个字符’)’，诶，此时从开始到第二个字符是一个合法的括号对了。所以，我们还可以推出一个结论：**合法的括号对都是以’)’结尾的。** 并且此时，左括号的数量和右括号的数量一样。我们怎么知道左括号和右括号的数量一样呢？我们可以用一个数组left[]来标记。left[i]的意思是说以第i个字符结尾的子串的剩余左括号的数量。在这个例子中left[1]=1，因为只有一个'(‘，left[2]=1，左括号和右括号抵消了。

好，回到动态规划的核心问题，状态是什么？其实，很容易想到，**F(i)可以是第i个字符结尾的最大合法括号对的对数**（不是长度，是长度的一般）。到了这里，我们已经迈出了一大步，现在从头开始分析下，递推公式应该是什么？下面以以”()(())”为例。
1. 初始化 left[0] = 0, F[0] = 0
2. 第一个字符。left[1] = 1, F[1] = 0（因为以这个字符结尾的子串是非法的）
3. 第二个字符。遇到’)’要注意啦~ left[2] = left[1] – 1 = 0， F[2] = F[1] + 1。到这里意思是说，第二个字符的子串的最大合法括号对是1.到这里来看这个公式还是对的
4. 第三个字符。left[3] = left[2] + 1 = 1, F[3] = 0
5. 第四个字符。left[4] = left[3] + 1 = 2, F[4] = 0
6. 第五个字符。left[5] = left[4] – 1 = 1, F[5] = F[4] + 1 = 1
7. 第六个字符。left[6] = left[5] – 1 = 0, F[6] = F[5] + 1 = 2

到了这里，好像不对啊！F[6]应该等于3才对！问题出在哪呢？我们把最开始的”()”丢掉了！那怎么记录这个合法的”()”呢？问题出在这，看这两个字符串 “()(()”和 “()(())”当我们遇到第三个字符的时候，并不知道后面是合法的子串”(())”还是非法的子串”(()”。如果是非法的话，从第三个字符处就已经截断，前面的”()”就和后面的串没有关系了；如果是合法的话，就需要把前面的“()”加上。因此，我们必须保留”()”。怎样保留呢？再用一个数组表示前面有多少合法的子串吗？我们再看一个更复杂的例子“()(()()”。如果用数组存储前面有多少合法的子串的话，第三个字符处应该存储前面有一个合法的，但第四个字符处存啥？第六个呢？这个太难表示了。怎么办呢？**我们使用栈！**

**当第i+1字符是'(‘的时候，把F[i]压入栈，表示前面有F[i]个合法的括号对，当遇到’)’的时候弹栈加上前面的合法的括号对数。** 我们看一下这种方法是否可行。我们使用”()(()()”这个例子。

index | left数组 | F状态 | stack状态
--- | --- | --- | ---
初始化  | left[0] = 0 | F[0] = 0 | stack.clear()
index = 1 | left[1] = left[0] + 1 = 1 | F[1] = 0 | stack.push(F[0]), [0]
index = 2 | left[2] = left[1] – 1 = 0 | F[2] = F[1] + 1 + s.top() = 1 | stack.pop(), []
index = 3 | left[3] = left[2] + 1 = 1 | F[3] = 0 | stack.push(F[2]), [1]
index = 4 | left[4] = left[3] + 1 = 2 | F[4] = 0 | stack.push(F[3]), [1, 0]
index = 5 | left[5] = left[4] – 1 = 1 | F[5] = F[4] + 1 + stack.top() = 1 | stack.pop(), [1]
index = 6 | left[6] = left[5] + 1 = 2 | F[6] = 0 | stack.push(F[5]), [1, 1]
index = 7 | left[7] = left[6] – 1 = 1 | F[7] = F[6] + 1 + stack.top() = 2 | stack.pop(), [1]

结束。最后遍历F数组，找出最大的括号对数。从这个例子中可以看出这个方法是可行的，可以解决上面那个问题。但是还有一个小问题。我们看这个例子：“())()”，第三个字符是’)’，此时它前面已经没有左括号了，可以肯定的是，这个字符将截断前面和后面。因此后面的状态值与前面一点关系都没有，因此需要清空left， F 和栈。
好，上面就是我们一步一步分析的过程。**下面总结一下：**
1. 此方法可以用动态规划来做，时间复杂度是O(n)
2. 需要借用栈来保存前面的可能用到的合法的最大括号对。从上面的例子中，我们可以看到到最后栈中还存有数据，说明这个是没有用到的。
3. 状态F[i]是第i个字符结尾的最大合法括号对的对数。
4. left[i]是第i个字符前剩余左括号的数量。
5. 递推公式有些复杂，直接上伪代码。

### 伪代码
```C++
func longestValidParentheses(string s):
    length = len(s) //字符串长度
    left[length+1] //存储剩余左括号的数量
    F[length+1] //状态函数
    stack
    left[0] = F[0] = 0 //初始化
    for i from 0 to length-1:
        if ('(' == s[i])
            left[i+1] = left[i] + 1
            stack.push(F[i]) //保存前面的合法字符串对数
            F[i+1] = 0
        else if (')' == s[i])
            if (left[i] > 0) //开始收割
                left[i+1] = left[i] + 1
                F[i+1] = F[i] + 1 + stack.top()
                stack.pop()
            else //已经截断，后面的子串与前面子串已无联系
                left[i+1] = 0
                F[i+1] = 0
                stack.clear()
    return 2*max(F)
```

### 完整代码

```java
#include <iostream>
#include <string>
#include <stack>

using namespace std;

class Solution
{
public:
    int longestValidParentheses1(string s)
    {
        int len = s.length();
        if (len < 2)
            return 0;
        int* left = new int[len+1];
        int* max = new int[len+1];
        left[0] = 0;
        max[0] = 0;
        stack<int> stk;
        for (int i = 0; i < len; ++i)
        {
            if (s[i] == '(')
            {
                left[i+1] = left[i] + 1;
                stk.push(max[i]);
                max[i+1] = 0;
            }
            else
            {
                if (left[i] > 0)
                {
                    left[i+1] = left[i] - 1;
                    max[i+1] = max[i] + 1 + stk.top();
                    stk.pop();
                }
                else
                {
                    left[i+1] = 0;
                    max[i+1] = 0;
                    while (!stk.empty())
                        stk.pop();
                }

            }
        }

        int result = max[0];
        for (int i = 1; i < len + 1; ++i)
            if (result < max[i])
                result = max[i];
        return result * 2;
    }
};

int main()
{
    Solution s;
    //string str = ")())()(()()))";
    //string str = ")()(()))()";
    //string str = "))(()((";
    //string str = "()(()";
    string str = ")()(()(())(())()";
    int result = s.longestValidParentheses1(str);
    cout<<result<<endl;
    return 0;
}
```
