---
title: Leetcode 之 dfs
author: Lilihx
tags:
  - Leetcode
categories:
  - code
date: 2021-03-04 14:12:00
---
## 前言

> Dfs : deep first search。深度优先搜索，主要用于数据结构中的树论或图论之中。与之对应的是：广度优先搜索。『寻路算法』的思想主要是：step forward，不行就go back。下面以LeetCode的题目作为举例。

<!--more-->

## 题目1

[题目链接：](https://leetcode-cn.com/problems/ju-zhen-zhong-de-lu-jing-lcof/)

![截屏2021-03-04 下午1.33.38](http://bj.bcebos.com/ibox-thumbnail98/d7bca9ac9f9a2491041c9f18e85c1bc7?authorization=bce-auth-v1%252Ffbe74140929444858491fbf2b6bc0935%252F2021-03-04T05%253A33%253A44Z%252F1800%252F%252Fe493675c50a019f209817c3923d911a5c835a23b6ef07415a2c91a57468f0a15)

思路：这是一个典型的『寻路』题。下面直接给出代码：

```c++
class Solution {
public:
    bool exist(vector<vector<char>>& board, string word) {
        int rows = board.size();
        int cols = board[0].size();
        for(int i = 0; i < rows; i++) {
            for(int j = 0; j < cols; j++) {
                if(dfs(board, word, i, j, 0)) return true;
            }
        }
        return false;
    }
    bool dfs(vector<vector<char>>& board, string word, int i, int j, int k) {
            int rows = board.size();
        int cols = board[0].size();
        if(i >= rows || i < 0 || j >= cols || j < 0 || board[i][j] != word[k]) return false;
        if(k == word.size() - 1) return true;
        board[i][j] = '\0';
        bool res = dfs(board, word, i + 1, j, k + 1) || dfs(board, word, i - 1, j, k + 1) || 
                      dfs(board, word, i, j + 1, k + 1) || dfs(board, word, i , j - 1, k + 1);
        board[i][j] = word[k];
        return res;
    }
};
```

实现dfs的主要方式就是递归，这里判断的是字符串是否和某一路径匹配。我们用k作为已经匹配的长度。寻路算法需要注意的几点是：

- 某次搜索的结束条件：通常为边界值 + 特定条件。本题中的特定条件是字符串不匹配。
- 状态记录：主要记录某节点的状态值：是否访问过 等。在访问某节点后，需要更新状态值。
- 回退：当当前节点的各方向都不可达时，需要将当前节点的状态回退。通常是 将已访问 改为 未访问。

本题的思路中，巧妙的一点是：没有新开状态数组，而是在原字符串数组中，利用特殊字符来记录状态变化。



另：本题虽然这种方法通过，但是 时空间 表现却极差。![截屏2021-03-04 下午1.49.40](http://bj.bcebos.com/ibox-thumbnail98/a433025c2f0efe2fac7171cbbedf0841?authorization=bce-auth-v1%252Ffbe74140929444858491fbf2b6bc0935%252F2021-03-04T05%253A49%253A45Z%252F1800%252F%252F1291d09016b74a73213d6ae12db083280e5d56cea4e717c93f6f3ccfabb82deb)

** 分析 **

> 上述方法对每个节点都进行了大量重复的遍历，我们的跳出判断拦截率不高。我们在做**字符串模式匹配**的时候，注意到其会利用到字符串本身的特性。

> 此外，我们没有利用子问题的解。我可以提供一种思想：比如我们从后往前遍历，记录后面每个节点可以匹配到的末尾字符串数量。比如子节点i,j 可以匹配最后w各字符，那我们往前探寻的时候就可以充分利用子问题的解，减少递归。



## 题目2：

[题目链接](https://leetcode-cn.com/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof/)

![截屏2021-03-04 下午1.57.35](http://bj.bcebos.com/ibox-thumbnail98/96f2291be8d98a60c35256f15449e3c4?authorization=bce-auth-v1%252Ffbe74140929444858491fbf2b6bc0935%252F2021-03-04T05%253A57%253A40Z%252F1800%252F%252F8cce66d9e7722871567fa9a5965d5d87e68df513c70c4f9d9db29dae7cc699dc)

**分析**

该题目和第一题很类似。典型的dfs问题。需要注意的是：

- 该dfs的跳出判断是：出界 + 当前节点已访问 + 当前节点不合法
- 状态记录：需要新开内存，记录每个节点状态
- 回退：该题的回退你可以理解为：路路可达。路路不可达时，回退当前节点。可达时不需要回退。



** 代码 **

```c++
class Solution {
public:

    int cal_num (int num) {
        int res = 0;
        while(num >= 10) {
            res += (num%10);
            num /= 10;
        }
        res += num;
        return res;
    }

    void dfs(vector<vector<bool>> &table, int width, int height, int k) {
        int row = table.size();
        int col = table[0].size();

        if(width<0 || width>=row || height<0 || height>=col) return;

        else if(cal_num(width) + cal_num(height) > k) return;

        else if(table[width][height]) return;
        table[width][height] = true;

        dfs(table, width-1,height, k);
        dfs(table, width+1,height, k);
        dfs(table, width,height-1, k);
        dfs(table, width,height+1, k);
        
    }

    int movingCount(int m, int n, int k) {
        vector<vector<bool>> table(m);
        for(int i=0; i<m; i++) {
            table[i].resize(n);
            for(int j =0; j<n; j++) {
                table[i][j] = false;
            }
        }
        dfs(table, 0, 0, k);

        int res = 0;
        for(int i=0; i<m; i++) {
            for(int j =0; j<n; j++) {
                if(table[i][j])
                    res ++;
            }
        }
        return res;
    }
};
```



注意：

> 状态数组我用的是bool，bool只能表示两种状态：已访问 未访问。但是本题中还有第三种状态：行列超出范围导致的『节点无效』。关于这个节点无效的处理方式有两种：
>
> - dfs之前：状态方程不用bool，增加第三种状态记录。
> - dfs之中：dfs的时候进行判断，状态方程已然用Bool, 只需两种
>
> 前者bool转int，内存增大；后者判断的次数增大，如果单独写cal_num，调用栈 和 调用次数增加，时间和内存也可能增大。

两种方法的时空间表现如下：

![截屏2021-03-04 下午2.03.48](http://bj.bcebos.com/ibox-thumbnail98/0bfbb5793a71a14d904fcd485acdc771?authorization=bce-auth-v1%252Ffbe74140929444858491fbf2b6bc0935%252F2021-03-04T06%253A03%253A53Z%252F1800%252F%252F66e7b475a646a9c8e1386ba05888f5f9109455d15e3fa9a8ac1b8090382fbbff)