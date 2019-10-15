# LeetCode51  ==hard==  N皇后 经典回溯算法

题目描述:*n* 皇后问题研究的是如何将 *n* 个皇后放置在 *n*×*n* 的棋盘上，并且使皇后彼此之间不能相互攻击。攻击指的是在同一行、同一列以及对角线上发动攻击。

![avatar](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/12/8-queens.png)

<center>以上为8皇后的一种解法</center>

给定一个整数 n，返回所有不同的 n 皇后问题的解决方案。

每一种解法包含一个明确的 n 皇后问题的棋子放置方案，该方案中 'Q' 和 '.' 分别代表了皇后和空位。

```
示例:

输入: 4
输出: [
 [".Q..",  // 解法 1
  "...Q",
  "Q...",
  "..Q."],

 ["..Q.",  // 解法 2
  "Q...",
  "...Q",
  ".Q.."]
]
解释: 4 皇后问题存在两个不同的解法。
```

* 对于此类问题，可以用经典的回溯算法。思路就是把问题的解空间转化成了树或者图的结构表示，然后使用深度优先搜索策略进行遍历，遍历的过程中记录和寻找所有可行解或最优解。
* 方法
  * 从根节点出发搜索解空间树
  * 至解空间树某一节点时，先利用==剪枝函数==判断该节点是否可行
    * 如果不可行，跳过对该节点为根的子树的搜索，逐层向其祖节点**回溯**
    * 如果可行，进入该子树，按==深度优先搜索==策略搜索
* 剪枝函数有两类
  * 使用*约束函数*，剪去不满足约束条件的路径
  * 使用*界限函数*，剪去不能得到最优解的路径

## N皇后解法一

* 设计常规的剪枝函数，新建一个N*N的数组，初始化为0，分别对行、列、对角去判断。

```c++
class Solution1 {
public:
    vector<vector<int>> array;
    vector<int> col;
    vector<vector<int>> res;
    bool check(int n, int k, int j) {
        for (int i = 0; i < n; i++) {//检查行列冲突
            if (array[i][j] == 1) {
                return false;
            }
        }
        for (int i = k - 1, m = j - 1; i >= 0 && m >= 0; i--, m--) {//检查左对角线
            if (array[i][m] == 1) {
                return false;
            }
        }
        for (int i = k - 1, m = j + 1; i >= 0 && m < n; i--, m++) {//检查右对角线
            if (array[i][m] == 1) {
                return false;
            }
        }
        return true;
    }
    void dfs(int i, int n) {
        if (i == n) {
            res.push_back(col);
            return;
        }
        for (int m = 0; m < n; m++) {
            if (check(n, i, m)) {
                array[i][m] = 1;
                col.push_back(m);
                dfs(i + 1, n);
                array[i][m] = 0;//回溯
                col.pop_back();
            }
        }
    }
    vector<vector<string>> solveNQueens(int n) {
        array = vector<vector<int>> (n, vector<int> (n, 0));//判断矩阵初始化
        dfs(0, n);
        vector<vector<string>> result;
        //按照一定格式输出
        for (auto& v : res) {
            vector<string> s;
            for (auto& c : v) {
                string t(n, '.');
                t[c] = 'Q';
                s.push_back(t);
            }
            result.push_back(s);
        }
        return result;
    }
};
```

## N皇后解法二

* 对于所有的主对角线有 ==行号 + 列号 = 常数(n-1)==，对于所有的次对角线有 ==行号 - 列号 = 常数(0)==.

  这可以让我们标记已经在攻击范围下的对角线并且检查一个方格 (行号, 列号) 是否处在攻击位置。

```c++
class Solution {
public:
    bool check(int n, vector<int>& cols) {
        if (cols.size() <= 1)
            return true;
        int row = cols.size() - 1;
        int col = cols.back();//新加入的列号
        for (int r = 0; r < row; r++) {
            int c = cols[r];//r行对应的列号c
            if (c == col || abs(c - col) == abs(r - row))//列号冲突或者对角线冲突
                return false;
        }
        return true;
    }

    void dfs(int n, vector<int>& cols, vector<vector<int>>& res) {
        if (!check(n, cols))
            return;
        if (cols.size() == n) {
            res.push_back(cols);
            return;
        }
        for (int i = 0; i < n; i++) {
            cols.push_back(i);
            dfs(n, cols, res);
            cols.pop_back();

        }
    }

    vector<vector<string>> solveNQueens(int n) {
        vector<int> cols;
        vector<vector<int>> all;
        dfs(n, cols, all);
        vector<vector<string>> res;
        //按照一定格式输出
        for (auto& v : all) {
            vector<string> s;
            for (auto& c : v) {
                string t(n, '.');
                t[c] = 'Q';
                s.push_back(t);
            }
            res.push_back(s);
        }
        return res;
    }
};
```

## 总结

两种方法都通过LeetCode案例，执行时间和内存使用相差不大。分别为12ms、10.8MB以及12ms、10.5MB。个人觉得第一种方法比较容易想到，同时符合经典的回溯算法的范式。而第二种方法属于进阶的方法，减少了代码量。

但两者都是属于回溯算法。这说明回溯算法确实对于解决一些诸如**N皇后**、**迷宫问题**、**数独问题**、**全排列问题**非常有效。