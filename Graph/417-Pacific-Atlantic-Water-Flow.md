# 417. Pacific Atlantic Water Flow

- **417. Pacific Atlantic Water Flow**
    - **1. Recursive DFS**
        
        A cell can reach an ocean if water can flow from that cell to the ocean (downhill/flat).
        
        Reverse it: start from the ocean borders and move uphill/flat (to neighbors with height >= current).
        
        If you can climb from the ocean to a cell, then that cell can flow down to that ocean.
        
        So we do 2 DFS runs:
        
        - From all **Pacific border** cells (top row + left column) → mark all reachable cells in `pac`
        - From all **Atlantic border** cells (bottom row + right column) → mark all reachable cells in `atl`
        
        Answer = cells that are in **both** sets.
        
        **Algorithm**
        
        1. Create two visited sets: `pac`, `atl`.
        2. `DFS` rule (reverse flow):
            - From cell `(r, c)`, you may go to a neighbor `(nr, nc)` only if `heights[nr][nc] >= heights[r][c]` (uphill or same).
        3. Run `DFS` from every Pacific border cell, fill `pac`.
        4. Run `DFS` from every Atlantic border cell, fill `atl`.
        5. For every cell in the grid, if it's in both `pac` and `atl`, add it to result.
        6. Return result.
        
        ```python
        class Solution:
            def pacificAtlantic(self, heights: List[List[int]]) -> List[List[int]]:
                ROWS, COLS = len(heights), len(heights[0])
                pac, atl = set(), set()
        
                def dfs(r, c, visit, prevHeight):
                    if ((r, c) in visit or          # visited
                        r < 0 or c < 0 or
                        r == ROWS or c == COLS or   # out of bound
                        heights[r][c] < prevHeight  # lower
                    ):
                        return # if visited or out of bound or lower, do nothing
                    visit.add((r, c)) # otherwise can reach from ocean, add to set
                    dfs(r + 1, c, visit, heights[r][c])
                    dfs(r - 1, c, visit, heights[r][c])
                    dfs(r, c + 1, visit, heights[r][c])
                    dfs(r, c - 1, visit, heights[r][c])
        
                for c in range(COLS):
                    dfs(0, c, pac, heights[0][c])
                    dfs(ROWS - 1, c, atl, heights[ROWS - 1][c])
        
                for r in range(ROWS):
                    dfs(r, 0, pac, heights[r][0])
                    dfs(r, COLS - 1, atl, heights[r][COLS - 1])
        
                res = []
                for r in range(ROWS):
                    for c in range(COLS):
                        if (r, c) in pac and (r, c) in atl:
                            res.append([r, c])
                return res
        ```
        
        <aside>
        🗣️
        
        **1. why reverse** 
        
        "Instead of checking each cell individually whether it can reach both oceans — which is expensive — I'll reverse the direction. 
        
        **2. what the DFS does** 
        
        I'll start from the ocean boundaries and flood inward, following cells that are equal or higher. 
        
        **3. how the two passes combine**
        
        Any cell reachable from the Pacific boundary AND the Atlantic boundary is an answer. I'll run DFS twice, once per ocean, sharing a visited set, so each cell is processed at most once per ocean.”
        
        </aside>
        
        <aside>
        💡
        
        DFS的结构，问自己3个问题：
        
        **Q1: DFS需要知道什么信息？**
        
        - 当前在哪：`(r, c)`
        - 哪些cell已经访问过（避免重复）：`visit`
        - 上一个cell的高度是多少（用来判断能不能走过来）：`prevHeight`
        
        → 这就是为什么这三个是参数
        
        **Q2: 什么时候停下来（base case）？**
        
        - 越界了
        - 已经访问过了
        - 当前cell比prevHeight**低**（逆流走不上去）
        
        **Q3: 什么时候把cell加入集合？**
        
        - 没有触发base case，说明"逆流"能到达这里 → `visit.add((r, c))`
        </aside>
        
        <aside>
        💡
        
        ### Time Complexity: O(M × N)
        
        每个cell最多被加入`pac`或`atl`一次，加入后 再被访问 就直接return。所以两次BFS/DFS（pacific + atlantic）加起来：
        
        - 每个cell最多处理**2次**（一次为pac，一次为atl）
        - 2 × M × N = **O(M × N)**
        
        ---
        
        ### Space Complexity: O(M × N)
        
        三块空间：三项都是O(M × N)，加起来还是 **O(M × N)**。
        
        | 来源 | 大小 |
        | --- | --- |
        | `pac` set | O(M × N) 最坏全部cell都可达 |
        | `atl` set | O(M × N) 同上 |
        | call stack | O(M × N) 最坏DFS递归深度遍历所有cell |
        </aside>