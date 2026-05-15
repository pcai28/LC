# 695. Max Area of Island

- **695. Max Area of Island**
    - **1. Breadth First Search**
        
        ```python
        class Solution:
            def maxAreaOfIsland(self, grid: List[List[int]]) -> int:
                ROWS, COLS = len(grid), len(grid[0])
                directions = [[0, 1], [0, -1], [1, 0], [-1, 0]]
                maxArea = 0
        
                def bfs(r, c):
                    grid[r][c] = 0
                    count = 1
                    q = deque()
                    q.append((r, c))
                    while q:
                        r, c = q.popleft()
                        for dr, dc in directions:
                            row, col = r + dr, c + dc
                            if (row in range(ROWS)
                                and col in range(COLS)
                                and grid[row][col] == 1
                            ):
                                count += 1
                                grid[row][col] = 0
                                q.append((row, col))
                    return count
        
                for r in range(ROWS):
                    for c in range(COLS):
                        if grid[r][c] == 1:
                            count = bfs(r, c)
                            maxArea = max(maxArea, count)
        
                return maxArea
        ```
        
    - **2. Recursive DFS**
        
        
        ```python
        class Solution:
            def maxAreaOfIsland(self, grid: List[List[int]]) -> int:
                ROWS, COLS = len(grid), len(grid[0])
                directions = [[0, 1], [0, -1], [1, 0], [-1, 0]]
                maxArea = 0
        
                def dfs(r, c):
                    if (r not in range(ROWS)
                        or c not in range(COLS)
                        or grid[r][c] == 0
                    ):
                        return 0
                    
                    grid[r][c] = 0
                    count = 1
        
                    for dr, dc in directions:
                        row, col = r + dr, c + dc
                        count += dfs(row, col)
        
                    return count
        
                for r in range(ROWS):
                    for c in range(COLS):
                        if grid[r][c] == 1:
                            count = dfs(r, c)
                            maxArea = max(maxArea, count)
        
                return maxArea
        ```
        
        核心思想：**每个格子负责返回"以自己为起点能贡献多少面积"**
        
        递归函数的语义 `def dfs(r, c) -> int:` —— 返回值 = 从 (r,c) 出发，这块连通陆地的面积
        
        **三步走**
        
        **① 边界/已访问/终止条件 → 返回 0（不贡献面积）**
        
        **② 标记已访问 + 自己贡献 1**
        
        **③ 向四个方向扩展，累加邻居的贡献**