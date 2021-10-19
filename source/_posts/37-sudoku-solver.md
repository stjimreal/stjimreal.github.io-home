---
title: 37-sudoku-solver
date: 2021-10-19 14:46:13
tags: [data structure & algorithms, leetcode, backtracking]
catagories: 
    - [data structure & algorithms, search schemes]
---

# [回溯法] 37：解数独

> 力扣链接：[sudoku-solver](https://leetcode-cn.com/problems/sudoku-solver/)
## 题目描述

编写一个程序，通过填充空格来解决数独问题。

数独的解法需 遵循如下规则：

+ 数字 1-9 在每一行只能出现一次。 
+ 数字 1-9 在每一列只能出现一次。 
+ 数字 1-9 在每一个以粗实线分隔的 3x3 宫内只能出现一次。（请参考示例图） 
+ 数独部分空格内已填入了数字，空白格用 '.' 表示。 

## 输入输出描述

board.length == 9
board[i].length == 9
board[i][j] 是一位数字 `char` 或者 '.'
题目数据 保证 输入数独仅有一个解

例如：

```
[
    ["5","3",".",".","7",".",".",".","."],
    ["6",".",".","1","9","5",".",".","."],
    [".","9","8",".",".",".",".","6","."],
    ["8",".",".",".","6",".",".",".","3"],
    ["4",".",".","8",".","3",".",".","1"],
    ["7",".",".",".","2",".",".",".","6"],
    [".","6",".",".",".",".","2","8","."],
    [".",".",".","4","1","9",".",".","5"],
    [".",".",".",".","8",".",".","7","9"]
]
```

## 思路

### 扫描一次——获取所有的带入点位置；获取现有的每一行的候选数字集，每一列的候选数字集，每一块的候选数字集。

以上述题干为例：
```python
[
    {"1","2","4","6","8","9"}, 
    {"2","3","4","7","8"}, 
    {"1","2","3","4","5","7"}, ...]
``` 
为行候选数字集。

```python
[
    {"1","2","3","9"}, 
    {"1","2","4","5","7","8"}, 
    {"1","2","3","4","5","6","7","9"}, ...]
```
为列候选数字集。

```python
[
    [
        {"1","2","4","7"},
        {"2","3","4","6","8"},
        {"1","2","3","4","5","7","8","9"}
    ], 
    [
        ...
    ],
    [
        ...
    ]
]
```
为块候选数字集

### dfs搜索——pop_back所有点位置，获取`(x,y)`，对其枚举'1'-'9'，如果在所有的候选数字集中，则在候选集修改该位置，则dfs继续搜索，如果在递归dfs搜索中找到结果则修改状态位`flag = True`，返回。如果找到`flag == True`，则跳出枚举；如果未找到`flag == True`则回溯候选集位置，继续枚举。


## 代码

### C/C++

```c++
class Solution {
public:
    const static int N = 9;
    bool flag;
    int row_map[N][N], col_map[N][N], grid_map[3][3][N];
    vector<pair<int, int>> pos;
    void dfs(vector<vector<char>>& board) {
        if (pos.empty()) {
            flag = true;
            return;
        }
        auto [x, y] = pos.back();
        pos.pop_back();
        for (int digit = 0; digit < 9; ++digit) {
            if (row_map[x][digit] || col_map[y][digit] || grid_map[x/3][y/3][digit]) {
                continue;
            }
            row_map[x][digit] = col_map[y][digit] = grid_map[x/3][y/3][digit] = true;
            board[x][y] = digit + 1 + '0';
            dfs(board);
            if (flag) {
                break;
            } else {
                row_map[x][digit] = col_map[y][digit] = grid_map[x/3][y/3][digit] = false;
            }
        }
        if (!flag) {
            pos.push_back(pair{x,y});
        }
    }


    void solveSudoku(vector<vector<char>>& board) {
        bzero(row_map, sizeof(int) * N * N);
        bzero(col_map, sizeof(int) * N * N);
        bzero(grid_map, sizeof(int) * N * N);
        flag = false;
        for (int i = 0; i < N; ++i) {
            for (int j = 0; j < N; ++j) {
                if (board[i][j] == '.') {
                    pos.push_back(pair{i, j});
                } else {
                    auto val = board[i][j] - '0' - 1;
                    row_map[i][val] = col_map[j][val] = grid_map[i/3][j/3][val] = true;
                }
            }
        }
        dfs(board);

    }
};
```

### Python

```python
class Solution(object):
    
    def solveSudoku(self, board):
        """
        :type board: List[List[str]]
        :rtype: None Do not return anything, modify board in-place instead.
        """
        flag = False
        N = 9    
        row_map = [[False] * N for _ in range(9)]
        col_map = [[False] * N for _ in range(9)] * N
        grid_map= [[[False] * N for _a in range(3)] for _b in range(3)]
        pos = []

        for i in range(9):
            for j in range(9):
                if board[i][j] == '.':
                    pos.append((i,j))
                else:
                    dig = int(board[i][j]) - 1
                    row_map[i][dig] = True
                    col_map[j][dig] = True
                    grid_map[i//3][j//3][dig] = True

        def dfs(board):
            nonlocal flag
            if len(pos) == 0:
                flag = True
                return
            x, y = pos.pop()
            for digit in range(9):
                if row_map[x][digit] or \
                    col_map[y][digit] or \
                    grid_map[x//3][y//3][digit]:
                    continue
                row_map[x][digit] = \
                col_map[y][digit] = \
                grid_map[x//3][y//3][digit] = True
                board[x][y] = str(digit + 1)
                dfs(board)
                row_map[x][digit] = \
                col_map[y][digit] = \
                grid_map[x//3][y//3][digit] = False
                if flag:
                    break
            if not flag:
                pos.append((x,y))
        
        
        dfs(board)
        print(board)
        
        
if __name__ == '__main__':
    Solution().solveSudoku([
    ["5","3",".",".","7",".",".",".","."],
    ["6",".",".","1","9","5",".",".","."],
    [".","9","8",".",".",".",".","6","."],
    ["8",".",".",".","6",".",".",".","3"],
    ["4",".",".","8",".","3",".",".","1"],
    ["7",".",".",".","2",".",".",".","6"],
    [".","6",".",".",".",".","2","8","."],
    [".",".",".","4","1","9",".",".","5"],
    [".",".",".",".","8",".",".","7","9"]
    ])
```