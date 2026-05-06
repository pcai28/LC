# 133. Clone Graph

- **133. Clone Graph**
    - **1. Recursive DFS**
        
        <aside>
        🗣️
        
        **1. Identify the core challenge**
        
        > "Since this is a graph, and not a tree, there can be cycles — so a naive DFS would loop infinitely."
        > 
        
        **2. State your solution to that challenge**
        
        > "I'll use a hashmap to map each original node to its clone, which doubles as a visited set to prevent revisiting."
        > 
        
        **3. State the DFS logic**
        
        > "Then I'll DFS through the graph — for each node, I create its clone, then recursively clone all its neighbors and attach them."
        > 
        </aside>
        
        ```python
        """
        # Definition for a Node.
        class Node:
            def __init__(self, val = 0, neighbors = None):
                self.val = val
                self.neighbors = neighbors if neighbors is not None else []
        """
        
        from typing import Optional
        class Solution:
            def cloneGraph(self, node: Optional['Node']) -> Optional['Node']:
                oldToNew = {}
        
                def dfs(node):
                    if node in oldToNew:
                        return oldToNew[node]
        
                    copy = Node(node.val)
                    oldToNew[node] = copy
                    for nei in node.neighbors:
                        copy.neighbors.append(dfs(nei))
        
                    return copy
        
                return dfs(node) if node else None
        ```
        
        <aside>
        💡
        
        树的 DFS 克隆很简单，但图有环，如果朴素递归会**无限循环**。
        
        > **关键洞察**：需要一个机制来标记"这个节点我已经克隆过了"。
        > 
        
        这就自然引出了 `oldToNew` 这张哈希表，它同时承担两个职责：
        
        1. **visited 标记**（防止无限递归）
        2. **旧节点 → 新节点的映射**（让别的节点能找到已克隆的副本）
        
        ---
        
        明确 `dfs(node)` 的含义：
        
        > 给我一个旧节点，返回它对应的**新节点**（如果还没克隆就克隆，如果已克隆直接返回）
        > 
        
        ---
        
        **"先克隆自己并注册，再递归邻居"** — 注册这一步既是 visited 标记，也是给其他节点返回副本的桥梁，两用合一是这题设计最精妙的地方。
        
        ---
        
        本质上，**树是无环图的特例**，pre-order DFS 就是 图 DFS 去掉 visited 检查后的简化版。
        
        |  | 树 Pre-order | 图 DFS（这题） |
        | --- | --- | --- |
        | 处理当前节点 | ✓ 先处理 | ✓ 先处理 |
        | 递归子结构 | 递归 children | 递归 neighbors |
        | 防重复 | 不需要（树无环） | 需要 `oldToNew` 作为 visited |
        </aside>
        
    - **2. Breadth First Search**
        
        
        ```python
        """
        # Definition for a Node.
        class Node:
            def __init__(self, val = 0, neighbors = None):
                self.val = val
                self.neighbors = neighbors if neighbors is not None else []
        """
        
        from typing import Optional
        class Solution:
            def cloneGraph(self, node: Optional['Node']) -> Optional['Node']:
                if not node:
                    return None
        
                oldToNew = {}
                oldToNew[node] = Node(node.val)
                q = deque([node])
                while q:
                    cur = q.popleft()
                    for nei in cur.neighbors:
                        if nei not in oldToNew:
                            oldToNew[nei] = Node(nei.val)
                            q.append(nei)
                        oldToNew[cur].neighbors.append(oldToNew[nei])
                
                return oldToNew[node]
        ```
        
        <aside>
        🗣️
        
        **1. Identify the core challenge**
        
        > "Since this is a graph with potential cycles, I need to track which nodes I've already cloned to avoid revisiting."
        > 
        
        **2. State your data structure**
        
        > "I'll use a hashmap from original to clone — it serves as both a visited set and lets me look up the cloned version of any node when I need to wire up neighbors."
        > 
        
        **3. State the BFS logic**
        
        > "I'll BFS level by level. For each node I pop, I go through its neighbors — if a neighbor hasn't been cloned yet, I clone it and enqueue it. Either way, I always connect the current clone to the neighbor's clone."
        > 
        </aside>
        
        <aside>
        💡
        
        BFS 从起点开始，但起点没有"父节点"来触发它的克隆。
        
        所以需要在循环外 手动克隆 起点，并放入队列
        
        </aside>