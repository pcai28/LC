# 207. Course Schedule

- **207. Course Schedule**
    - **1. Cycle Detection (DFS)**
        
        <aside>
        💡
        
        ### Step 1: 理解题意 → 转化为图问题
        
        题目说：要完成课程 A，必须先完成课程 B。这天然就是一个**依赖关系**。
        
        问题变成：**能否完成所有课程？** = **依赖关系中有没有循环依赖（环）？**
        
        > 如果 A 依赖 B，B 依赖 A → 永远无法开始 → 返回 False
        > 
        
        所以核心问题变成：**这个有向图是否有环（Cycle Detection）？**
        
        ---
        
        ### Step 2: 边的方向为什么是 course → prerequisite？
        
        这是最容易混淆的地方，关键在于**你想用 DFS 检测什么**。
        
        DFS 的逻辑是：**"从课程 A 出发，沿着依赖链走，能不能回到 A？"**
        
        `canFinish(A) 需要先 canFinish(B)
        canFinish(B) 需要先 canFinish(C)
        canFinish(C) 需要先 canFinish(A)  ← 回到了 A，成环！`
        
        这条"查询路径"天然就是 **A → B → C → A**，即：
        
        `edge 方向 = "我依赖谁" = course → prerequisite`
        
        如果反向建边（prerequisite → course），DFS 走的是"我是谁的前置"，就**无法自然地检测 调用栈上的环**。
        
        ---
        
        ### Step 3: 环检测的核心难点 → 需要区分两种"访问过"
        
        | 状态 | 含义 | 用途 |
        | --- | --- | --- |
        | **在当前 DFS 路径上** | 还没回溯，再次遇到 = 有环 | 检测环 |
        | **已经完全处理完** | 确认无环，可以安全跳过 | 剪枝优化 |
        
        代码里用了一个巧妙的**双重标记**：
        
        `visitSet          # = "当前路径上的节点"（进入时加，回溯时删）
        preMap[crs] = []  # = "已确认无环"（处理完后清空 作为 memo）`
        
        ---
        
        </aside>
        
        ```python
        class Solution:
            def canFinish(self, numCourses: int, prerequisites: List[List[int]]) -> bool:
                # map each course to prereq list
                preMap = { i:[] for i in range(numCourses)}
                # preMap = defaultdict(list)
                for crs, pre in prerequisites:
                    preMap[crs].append(pre)
                
                # visitSet = all crs along the curr DFS path
                visitSet = set()
                def dfs(crs):
                    if crs in visitSet:
                        return False
                    if preMap[crs] == []:
                        return True
        
                    visitSet.add(crs)
                    for pre in preMap[crs]:
                        if not dfs(pre): return False
                    # if we find 1 crs cant be completed, 
                    # we can imdtly return false for the entire function
                    # otherwise all prereq can be completed
                    visitSet.remove(crs)
                    # remove it so that other paths can successfully revisit it
                    preMap[crs] = []
                    # if we have to run dfs on it again, 
                    # we can execute the second base case, return true imdtly
                    # we dont have to repeat running dfs on its neighbors
                    return True
                
                for crs in range(numCourses):
                    if not dfs(crs): return False
                return True
                # in case the graph is not fully connected
                # E.g.: two separate graphs that are not connected
                # 1 -> 2
                # 3 -> 4     
        ```
        
        <aside>
        💡
        
        Each course is a node, and each prerequisite is a **directed edge**. 
        
        You can finish all courses **only if there is no cycle** in this directed graph.
        
        **A cycle means:** 
        
        - Course A needs B
        - B needs C
        - C needs A —— So you’re stuck forever.
        
        **We use DFS with cycle detection:** 
        
        - While doing DFS, keep track of courses in the **current recursion path**.
        - If we visit a course already in the current path → **cycle found**.
        - If a course has no prerequisites left, it’s safe.
        </aside>
        
    - **2. Topological Sort (Kahn's Algorithm)**
        
        <aside>
        💡
        
        ### Step 1: 换一个角度思考问题
        
        DFS 的思路是：**"从一个课程出发，追它的依赖链，看有没有环"**
        
        Kahn's 的思路完全不同：**"模拟真实的选课过程"**
        
        > 现实中你会怎么做？
        > 
        > - 先找**没有任何前置要求**的课，上完
        > - 上完之后，某些课的前置被满足了，加入可选列表
        > - 重复，直到选完所有课（成功）或卡住（有环）
        
        **"卡住"= 有环** — 环里的课互相等待，永远无法开始。
        
        ---
        
        ### Step 2: 边的方向应该反过来
        
        这里是 Kahn's 和 DFS **最关键的区别**。
        
        DFS 的问题是：**"我依赖谁？"** → `course → prerequisite`
        
        Kahn's 的问题是：**"我解锁了谁？"** → `prerequisite → course`
        
        `DFS 视角（追依赖）:         Kahn's 视角（解锁）:
         A → B → C                  C → B → A
        "A 依赖 B 依赖 C"           "学完 C，解锁 B，再解锁 A"`
        
        为什么 Kahn's 需要反向？因为算法的核心操作是：
        
        > 当一个前置课程被"完成"时，需要立刻找到**所有依赖它的课程**，把它们的等待计数减一。
        > 
        
        这个操作需要从 `prerequisite` 快速找到 `course`，所以边必须是 `prerequisite → course`。
        
        ---
        
        ### Step 3: 引入 indegree（入度）
        
        模拟选课过程需要知道：**每门课还有几个前置没完成？**
        
        这就是**入度（indegree）**：
        
        `prerequisites = [[1,0],[2,0],[3,1],[3,2]]
        
        含义：
          课1 需要 课0      课0 → 课1
          课2 需要 课0      课0 → 课2
          课3 需要 课1      课1 → 课3
          课3 需要 课2      课2 → 课3
        indegree:
          课0: 0  ← 没有任何前置，可以立刻开始！
          课1: 1
          课2: 1
          课3: 2`
        
        **indegree = 0 的课 = 当前可以上的课**
        
        </aside>
        
        ```python
        class Solution:
            def canFinish(self, numCourses: int, prerequisites: List[List[int]]) -> bool:
                # pre -> crs
                adj = [[] for i in range(numCourses)]
                indegree = [0] * numCourses
                for crs, pre in prerequisites:
                    adj[pre].append(crs)
                    indegree[crs] += 1
        
                q = deque()
                for c in range(numCourses):
                    if indegree[c] == 0:
                        q.append(c)
        
                finish = 0
                while q:
                    c = q.popleft()
                    finish += 1
                    for nei in adj[c]:
                        indegree[nei] -= 1
                        if indegree[nei] == 0:
                            q.append(nei)
                            
                return finish == numCourses
        ```
        
        <aside>
        💡
        
        If a course has no prerequisites, it can be taken immediately.
        
        Kahn's Algorithm repeatedly takes courses that have **zero prerequisites**.
        
        When we finish a course, we remove its dependency effect from other courses.
        
        - If all courses can be taken this way - **no cycle**, return `true`
        - If some courses are never taken - **cycle exists**, return `false`
        
        **Algorithm**
        
        - Build a graph and compute `indegree` (number of prerequisites) for each course.
        - Add all courses with `indegree = 0` into a queue.
        - While the queue is not empty:
            - Remove a course from the queue.
            - finish += 1.
            - Reduce the `indegree` of its dependent courses.
            - If any dependent course reaches `indegree = 0`, add it to the queue.
        - After processing:
            - If finished courses == total courses - return `true`
            - Else - return `false`
        </aside>