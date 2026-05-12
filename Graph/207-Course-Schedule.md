# 207. Course Schedule

- **207. Course Schedule**
    - **1. Cycle Detection (DFS)**
        
        
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