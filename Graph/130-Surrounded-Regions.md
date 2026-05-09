# 130. Surrounded Regions

- **130. Surrounded Regions**
    - **1. Recursive DFS**
        
        <aside>
        💡
        
        **Reverse thinking** —— Capture surrounded regions → Capture everything except unsurrounded regions
        
        **A region is unsurrounded if there is ‘O’ cell in that region connected to the boarder.** ——Scan through every boarder cell, look for any ‘O’s, then run dfs on it, give it tmp value, change all ‘O’s to ‘T’s. 
        
        Why do we change it to a tmp value? —— To mark it as a unsurrounded region.
        
        Then do a double nested loop, iterate every single row, then find and change ‘O’ to ‘X’, because now we know for sure that anytime we see an ‘O’, it’s definitely part of a surrounded region.
        
        Last, change ‘T’ back to ‘O’
        
        </aside>
        
        <aside>
        🗣️
        
        Instead of trying to find surrounded regions directly, I’ll find the regions that cannot be surrounded: any O connected to the border. 
        
        I’ll use DFS starting from the border cells. The key observation is that an `"O"` should **not** be flipped if it is connected to any border `"O"`. 
        
        So first, I’ll traverse from all border `"O"` cells and temporarily mark all connected `"O"` cells as `"T"`. 
        
        After that, I’ll scan the whole board: any remaining `"O"` must be fully surrounded, so I flip it to `"X"`;  and I restore any `"T"` back to `"O"`.
        
        ---
        
        The time complexity is `O(m * n)` because Each cell is checked only a constant number of times, and each `"O"` cell is processed by DFS at most once. The extra space is `O(m * n)` in the worst case because of the DFS recursion stack, if the whole board is connected `"O"` cells.
        
        </aside>
        
        ```python
        class Solution:
            def solve(self, board: List[List[str]]) -> None:
                """
                Do not return anything, modify board in-place instead.
                """
                ROWS, COLS = len(board), len(board[0])
                directions = [[0, 1], [0, -1], [1, 0], [-1, 0]]
        
                def dfs(r, c):
                    if (r < 0 or c < 0
                        or r == ROWS or c == COLS
                        or board[r][c] != "O"
                    ):
                        return
        
                    board[r][c] = "T"
                    for dr, dc in directions:
                        dfs(r + dr, c + dc)
        
                for c in range(COLS):
                    if board[0][c] == "O": dfs(0, c)
                    if board[ROWS-1][c] == "O": dfs(ROWS-1, c)
                
                for r in range(ROWS):
                    if board[r][0] == "O": dfs(r, 0)
                    if board[r][COLS-1] == "O": dfs(r, COLS-1)
        
                for r in range(ROWS):
                    for c in range(COLS):
                        if board[r][c] == "O": 
                            board[r][c] = "X"
                        elif board[r][c] == "T": 
                            board[r][c] = "O"
        ```
        
        <aside>
        💡
        
        这题其实 `visited` 可以不要，因为你已经把访问过的 `"O"` 改成 `"T"` 了，`board[r][c] != "O"` 本身就能防止重复访问。
        
        ---
        
        `elif` 更清楚，在语义上表达的是：
        
        > 这是同一组分类判断，只会进入其中一个分支。
        > 
        
        > 如果这个格子是 `"O"`，把它变成 `"X"`；
        > 
        > 
        > **否则 如果**它是 `"T"`，把它变回 `"O"`。
        > 
        
        这里关键是 **“否则”**：一个 cell 不可能同时既是 `"O"` 又是 `"T"`。
        
        `"O"` 和 `"T"` 是两种互斥情况，我只需要处理其中一种。
        
        ---
        
        两个 `if` 的语义更像：
        
        > 先检查它是不是 `"O"`；
        > 
        > 
        > 然后不管刚才发生了什么，再检查它是不是 `"T"`。
        > 
        
        虽然这题里不会出错，但两个 `if` 是：这两个判断是**独立的**，可能都需要检查。
        
        </aside>
        
    - **2. Breadth First Search**
        
        
        ```python
        class Solution:
            def solve(self, board: List[List[str]]) -> None:
                """
                Do not return anything, modify board in-place instead.
                """
                ROWS, COLS = len(board), len(board[0])
                directions = [[0, 1], [0, -1], [1, 0], [-1, 0]]
                
                def bfs():
                    q = deque()
                    for r in range(ROWS):
                        for c in range(COLS):
                            if (r == 0 or r == ROWS - 1 or
                                c == 0 or c == COLS - 1) and board[r][c] == "O":
                                q.append((r, c))
                                board[r][c] = "T"
                    while q:
                        r, c = q.popleft()
                        for dr, dc in directions:
                            row, col = r + dr, c + dc
                            if (0 <= row < ROWS and 
                                0 <= col < COLS and 
                                board[row][col] == "O"
                            ):
                                q.append((row, col))
                                board[row][col] = "T"
        
                bfs()
                for r in range(ROWS):
                    for c in range(COLS):
                        if board[r][c] == "O":
                            board[r][c] = "X"
                        elif board[r][c] == "T":
                            board[r][c] = "O"
        ```
        
        <aside>
        🗣️
        
        I’ll use multi-source BFS starting from the border cells. 
        
        The key observation is that any O connected to the border cannot be captured, because it is not fully surrounded by X. So instead of trying to directly find surrounded regions, I’ll first find all safe O’s. I’ll push every border O into a queue, then run BFS to mark every reachable O’s as a temporary value, T. 
        
        After BFS finishes, I’ll scan the whole board. Any O that is still left must be surrounded, so I flip it to X. Any T was border-connected and safe, so I restore it back to O. 
        
        </aside>
        
        <aside>
        💡
        
        you should mark the cell as `"T"` **when you add it to the queue**, not when you pop it. Otherwise the same `"O"` can be appended multiple times by different neighbors.
        
        </aside>