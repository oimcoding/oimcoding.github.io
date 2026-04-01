# TikTok LeetCode Problems - Solutions Organized by Pattern

---

## SLIDING WINDOW PROBLEMS

---

### Problem 1: 3. Longest Substring Without Repeating Characters - Medium

**Description:** Given a string s, find the length of the longest substring without repeating characters.

**Constraints:**
- 0 <= s.length <= 5 * 10^4
- s consists of English letters, digits, symbols and spaces

**Examples:**
```
Input: s = "abcabcbb"
Output: 3
Explanation: The answer is "abc", with the length of 3.

Input: s = "bbbbb"
Output: 1
Explanation: The answer is "b", with the length of 1.

Input: s = "pwwkew"
Output: 3
Explanation: The answer is "wke", with the length of 3.
```

**Pattern:** Sliding Window + Hash Set

**Time Complexity:** O(n)
**Space Complexity:** O(min(m, n)) where m is charset size

**Solution:**
```python
def lengthOfLongestSubstring(s: str) -> int:
    char_set = set()
    left = 0
    max_length = 0
    
    for right in range(len(s)):
        while s[right] in char_set:
            char_set.remove(s[left])
            left += 1
        char_set.add(s[right])
        max_length = max(max_length, right - left + 1)
    
    return max_length
```

---

### Problem 2: 76. Minimum Window Substring - Hard

**Description:** Given two strings s and t, return the minimum window substring of s such that every character in t is included.

**Constraints:**
- 1 <= s.length, t.length <= 10^5
- s and t consist of uppercase and lowercase English letters

**Examples:**
```
Input: s = "ADOBECODEBANC", t = "ABC"
Output: "BANC"

Input: s = "a", t = "a"
Output: "a"

Input: s = "a", t = "aa"
Output: ""
```

**Pattern:** Sliding Window + Hash Map

**Time Complexity:** O(|s| + |t|)
**Space Complexity:** O(|s| + |t|)

**Solution:**
```python
from collections import Counter

def minWindow(s: str, t: str) -> str:
    if not s or not t or len(t) > len(s):
        return ""
    
    dict_t = Counter(t)
    required = len(dict_t)
    
    left, right = 0, 0
    formed = 0
    window_counts = {}
    
    ans = float("inf"), None, None
    
    while right < len(s):
        char = s[right]
        window_counts[char] = window_counts.get(char, 0) + 1
        
        if char in dict_t and window_counts[char] == dict_t[char]:
            formed += 1
        
        while left <= right and formed == required:
            char = s[left]
            
            if right - left + 1 < ans[0]:
                ans = (right - left + 1, left, right)
            
            window_counts[char] -= 1
            if char in dict_t and window_counts[char] < dict_t[char]:
                formed -= 1
            
            left += 1
        
        right += 1
    
    return "" if ans[0] == float("inf") else s[ans[1]:ans[2] + 1]
```

---

### Problem 3: 560. Subarray Sum Equals K - Medium

**Description:** Given an array of integers nums and an integer k, return the total number of subarrays whose sum equals to k.

**Constraints:**
- 1 <= nums.length <= 2 * 10^4
- -1000 <= nums[i] <= 1000
- -10^7 <= k <= 10^7

**Examples:**
```
Input: nums = [1,1,1], k = 2
Output: 2

Input: nums = [1,2,3], k = 3
Output: 2
```

**Pattern:** Prefix Sum + Hash Map

**Time Complexity:** O(n)
**Space Complexity:** O(n)

**Solution:**
```python
def subarraySum(nums: List[int], k: int) -> int:
    count = 0
    prefix_sum = 0
    prefix_map = {0: 1}
    
    for num in nums:
        prefix_sum += num
        if prefix_sum - k in prefix_map:
            count += prefix_map[prefix_sum - k]
        prefix_map[prefix_sum] = prefix_map.get(prefix_sum, 0) + 1
    
    return count
```

---

### Problem 4: 974. Subarray Sums Divisible by K - Medium

**Description:** Given an integer array nums and an integer k, return the number of non-empty subarrays that have a sum divisible by k.

**Constraints:**
- 1 <= nums.length <= 3 * 10^4
- -10^4 <= nums[i] <= 10^4
- 2 <= k <= 10^4

**Examples:**
```
Input: nums = [4,5,0,-2,-3,1], k = 5
Output: 7

Input: nums = [5], k = 9
Output: 0
```

**Pattern:** Prefix Sum + Hash Map

**Time Complexity:** O(n)
**Space Complexity:** O(k)

**Solution:**
```python
def subarraysDivByK(nums: List[int], k: int) -> int:
    remainder_count = {0: 1}
    prefix_sum = 0
    count = 0
    
    for num in nums:
        prefix_sum += num
        remainder = prefix_sum % k
        
        count += remainder_count.get(remainder, 0)
        remainder_count[remainder] = remainder_count.get(remainder, 0) + 1
    
    return count
```

---

### Problem 5: 2537. Count the Number of Good Subarrays - Medium

**Description:** Given an integer array nums and an integer k, return the number of good subarrays of nums.

**Constraints:**
- 1 <= nums.length <= 10^5
- 1 <= nums[i], k <= 10^9

**Examples:**
```
Input: nums = [1,1,1,1,1], k = 10
Output: 1

Input: nums = [3,1,4,3,2,2,4], k = 2
Output: 4
```

**Pattern:** Sliding Window + Hash Map

**Time Complexity:** O(n)
**Space Complexity:** O(n)

**Solution:**
```python
def countGood(nums: List[int], k: int) -> int:
    from collections import defaultdict
    
    count = defaultdict(int)
    pairs = 0
    result = 0
    left = 0
    
    for right in range(len(nums)):
        pairs += count[nums[right]]
        count[nums[right]] += 1
        
        while pairs >= k:
            result += len(nums) - right
            count[nums[left]] -= 1
            pairs -= count[nums[left]]
            left += 1
    
    return result
```

---

## TWO POINTERS PROBLEMS

---

### Problem 6: 11. Container With Most Water - Medium

**Description:** Find two lines that together with the x-axis form a container that holds the most water.

**Constraints:**
- n == height.length
- 2 <= n <= 10^5
- 0 <= height[i] <= 10^4

**Examples:**
```
Input: height = [1,8,6,2,5,4,8,3,7]
Output: 49

Input: height = [1,1]
Output: 1
```

**Pattern:** Two Pointers

**Time Complexity:** O(n)
**Space Complexity:** O(1)

**Solution:**
```python
def maxArea(height: List[int]) -> int:
    left, right = 0, len(height) - 1
    max_area = 0
    
    while left < right:
        width = right - left
        max_area = max(max_area, min(height[left], height[right]) * width)
        
        if height[left] < height[right]:
            left += 1
        else:
            right -= 1
    
    return max_area
```

---

### Problem 7: 15. 3Sum - Medium

**Description:** Given an integer array nums, return all the triplets that sum to zero.

**Constraints:**
- 3 <= nums.length <= 3000
- -10^5 <= nums[i] <= 10^5

**Examples:**
```
Input: nums = [-1,0,1,2,-1,-4]
Output: [[-1,-1,2],[-1,0,1]]

Input: nums = [0,1,1]
Output: []

Input: nums = [0,0,0]
Output: [[0,0,0]]
```

**Pattern:** Two Pointers + Sorting

**Time Complexity:** O(n²)
**Space Complexity:** O(1) excluding output

**Solution:**
```python
def threeSum(nums: List[int]) -> List[List[int]]:
    nums.sort()
    result = []
    
    for i in range(len(nums) - 2):
        if i > 0 and nums[i] == nums[i - 1]:
            continue
        
        left, right = i + 1, len(nums) - 1
        
        while left < right:
            total = nums[i] + nums[left] + nums[right]
            
            if total < 0:
                left += 1
            elif total > 0:
                right -= 1
            else:
                result.append([nums[i], nums[left], nums[right]])
                
                while left < right and nums[left] == nums[left + 1]:
                    left += 1
                while left < right and nums[right] == nums[right - 1]:
                    right -= 1
                
                left += 1
                right -= 1
    
    return result
```

---

### Problem 8: 33. Search in Rotated Sorted Array - Medium

**Description:** Search for a target value in a rotated sorted array.

**Constraints:**
- 1 <= nums.length <= 5000
- -10^4 <= nums[i] <= 10^4
- All values of nums are unique
- nums is rotated at some pivot
- -10^4 <= target <= 10^4

**Examples:**
```
Input: nums = [4,5,6,7,0,1,2], target = 0
Output: 4

Input: nums = [4,5,6,7,0,1,2], target = 3
Output: -1

Input: nums = [1], target = 0
Output: -1
```

**Pattern:** Binary Search (Modified)

**Time Complexity:** O(log n)
**Space Complexity:** O(1)

**Solution:**
```python
def search(nums: List[int], target: int) -> int:
    left, right = 0, len(nums) - 1
    
    while left <= right:
        mid = (left + right) // 2
        
        if nums[mid] == target:
            return mid
        
        if nums[left] <= nums[mid]:
            if nums[left] <= target < nums[mid]:
                right = mid - 1
            else:
                left = mid + 1
        else:
            if nums[mid] < target <= nums[right]:
                left = mid + 1
            else:
                right = mid - 1
    
    return -1
```

---

### Problem 9: 165. Compare Version Numbers - Medium

**Description:** Compare two version numbers version1 and version2.

**Constraints:**
- 1 <= version1.length, version2.length <= 500
- version1 and version2 only contain digits and '.'
- version1 and version2 are valid version numbers
- All revision numbers are stored as integers without leading zeros

**Examples:**
```
Input: version1 = "1.01", version2 = "1.001"
Output: 0

Input: version1 = "1.0", version2 = "1.0.0"
Output: 0

Input: version1 = "0.1", version2 = "1.1"
Output: -1
```

**Pattern:** Two Pointers / String Parsing

**Time Complexity:** O(max(m, n))
**Space Complexity:** O(m + n)

**Solution:**
```python
def compareVersion(version1: str, version2: str) -> int:
    v1_parts = list(map(int, version1.split('.')))
    v2_parts = list(map(int, version2.split('.')))
    
    max_len = max(len(v1_parts), len(v2_parts))
    
    for i in range(max_len):
        v1 = v1_parts[i] if i < len(v1_parts) else 0
        v2 = v2_parts[i] if i < len(v2_parts) else 0
        
        if v1 < v2:
            return -1
        elif v1 > v2:
            return 1
    
    return 0
```

---

### Problem 10: 189. Rotate Array - Medium

**Description:** Given an array, rotate the array to the right by k steps.

**Constraints:**
- 1 <= nums.length <= 10^5
- -2^31 <= nums[i] <= 2^31 - 1
- 0 <= k <= 10^5

**Examples:**
```
Input: nums = [1,2,3,4,5,6,7], k = 3
Output: [5,6,7,1,2,3,4]

Input: nums = [-1,-100,3,99], k = 2
Output: [3,99,-1,-100]
```

**Pattern:** Array Reversal

**Time Complexity:** O(n)
**Space Complexity:** O(1)

**Solution:**
```python
def rotate(nums: List[int], k: int) -> None:
    """Do not return anything, modify nums in-place instead."""
    n = len(nums)
    k = k % n
    
    nums.reverse()
    nums[:k] = reversed(nums[:k])
    nums[k:] = reversed(nums[k:])
```

---

### Problem 11: 678. Valid Parenthesis String - Medium

**Description:** Given a string s containing only '(', ')' and '*', return true if s is valid.

**Constraints:**
- 1 <= s.length <= 100
- s[i] is '(', ')' or '*'

**Examples:**
```
Input: s = "()"
Output: true

Input: s = "(*)"
Output: true

Input: s = "(*))"
Output: true
```

**Pattern:** Greedy / Two Pointers

**Time Complexity:** O(n)
**Space Complexity:** O(1)

**Solution:**
```python
def checkValidString(s: str) -> bool:
    low = high = 0
    
    for char in s:
        if char == '(':
            low += 1
            high += 1
        elif char == ')':
            low = max(0, low - 1)
            high -= 1
        else:  # '*'
            low = max(0, low - 1)
            high += 1
        
        if high < 0:
            return False
    
    return low == 0
```

---

## DEPTH-FIRST SEARCH (DFS) PROBLEMS

---

### Problem 12: 200. Number of Islands - Medium

**Description:** Given an m x n 2D binary grid which represents a map of '1's (land) and '0's (water), return the number of islands.

**Constraints:**
- m == grid.length
- n == grid[i].length
- 1 <= m, n <= 300
- grid[i][j] is '0' or '1'

**Examples:**
```
Input: grid = [
  ["1","1","1","1","0"],
  ["1","1","0","1","0"],
  ["1","1","0","0","0"],
  ["0","0","0","0","0"]
]
Output: 1

Input: grid = [
  ["1","1","0","0","0"],
  ["1","1","0","0","0"],
  ["0","0","1","0","0"],
  ["0","0","0","1","1"]
]
Output: 3
```

**Pattern:** Depth-First Search (DFS)

**Time Complexity:** O(m × n)
**Space Complexity:** O(m × n) for recursion stack

**Solution:**
```python
def numIslands(grid: List[List[str]]) -> int:
    if not grid or not grid[0]:
        return 0
    
    rows, cols = len(grid), len(grid[0])
    count = 0
    
    def dfs(r, c):
        if r < 0 or r >= rows or c < 0 or c >= cols or grid[r][c] == '0':
            return
        grid[r][c] = '0'
        dfs(r + 1, c)
        dfs(r - 1, c)
        dfs(r, c + 1)
        dfs(r, c - 1)
    
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == '1':
                count += 1
                dfs(r, c)
    
    return count
```

---

### Problem 13: 79. Word Search - Medium

**Description:** Given an m x n grid of characters board and a string word, return true if word exists in the grid.

**Constraints:**
- m == board.length
- n = board[i].length
- 1 <= m, n <= 6
- 1 <= word.length <= 15
- board and word consists of only lowercase and uppercase English letters

**Examples:**
```
Input: board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]], word = "ABCCED"
Output: true

Input: board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]], word = "SEE"
Output: true

Input: board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]], word = "ABCB"
Output: false
```

**Pattern:** Backtracking + DFS

**Time Complexity:** O(m × n × 4^L) where L = word length
**Space Complexity:** O(L) for recursion stack

**Solution:**
```python
def exist(board: List[List[str]], word: str) -> bool:
    if not board or not board[0]:
        return False
    
    rows, cols = len(board), len(board[0])
    
    def dfs(r, c, index):
        if index == len(word):
            return True
        
        if r < 0 or r >= rows or c < 0 or c >= cols or board[r][c] != word[index]:
            return False
        
        temp = board[r][c]
        board[r][c] = '#'
        
        found = (dfs(r + 1, c, index + 1) or
                 dfs(r - 1, c, index + 1) or
                 dfs(r, c + 1, index + 1) or
                 dfs(r, c - 1, index + 1))
        
        board[r][c] = temp
        return found
    
    for r in range(rows):
        for c in range(cols):
            if dfs(r, c, 0):
                return True
    
    return False
```

---

### Problem 14: 694. Number of Distinct Islands - Medium

**Description:** Given a 2D grid, count the number of distinct islands.

**Constraints:**
- m == grid.length
- n == grid[i].length
- 1 <= m, n <= 50
- grid[i][j] is either 0 or 1

**Examples:**
```
Input: grid = [[1,1,0,0,0],[1,1,0,0,0],[0,0,0,1,1],[0,0,0,1,1]]
Output: 1

Input: grid = [[1,1,0,1,1],[1,0,0,0,0],[0,0,0,0,1],[1,1,0,1,1]]
Output: 3
```

**Pattern:** DFS + Hash Set

**Time Complexity:** O(m × n)
**Space Complexity:** O(m × n)

**Solution:**
```python
def numDistinctIslands(grid: List[List[int]]) -> int:
    if not grid or not grid[0]:
        return 0
    
    rows, cols = len(grid), len(grid[0])
    islands = set()
    
    def dfs(r, c, r0, c0, shape):
        if r < 0 or r >= rows or c < 0 or c >= cols or grid[r][c] == 0:
            return
        
        grid[r][c] = 0
        shape.append((r - r0, c - c0))
        
        dfs(r + 1, c, r0, c0, shape)
        dfs(r - 1, c, r0, c0, shape)
        dfs(r, c + 1, r0, c0, shape)
        dfs(r, c - 1, r0, c0, shape)
    
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == 1:
                shape = []
                dfs(r, c, r, c, shape)
                islands.add(tuple(shape))
    
    return len(islands)
```

---

### Problem 15: 99. Recover Binary Search Tree - Medium

**Description:** Two elements of a BST are swapped by mistake. Recover the tree without changing its structure.

**Constraints:**
- The number of nodes in the tree is in the range [2, 1000]
- -2^31 <= Node.val <= 2^31 - 1

**Examples:**
```
Input: root = [1,3,null,null,2]
Output: [3,1,null,null,2]

Input: root = [3,1,4,null,null,2]
Output: [2,1,4,null,null,3]
```

**Pattern:** In-order Traversal + Two Pointers

**Time Complexity:** O(n)
**Space Complexity:** O(h) where h is height

**Solution:**
```python
def recoverTree(root: Optional[TreeNode]) -> None:
    """Do not return anything, modify root in-place instead."""
    first = second = prev = None
    
    def inorder(node):
        nonlocal first, second, prev
        if not node:
            return
        
        inorder(node.left)
        
        if prev and node.val < prev.val:
            if not first:
                first = prev
            second = node
        
        prev = node
        inorder(node.right)
    
    inorder(root)
    if first and second:
        first.val, second.val = second.val, first.val
```

---

### Problem 16: 297. Serialize and Deserialize Binary Tree - Hard

**Description:** Design an algorithm to serialize and deserialize a binary tree.

**Constraints:**
- The number of nodes in the tree is in the range [0, 10^4]
- -1000 <= Node.val <= 1000

**Examples:**
```
Input: root = [1,2,3,null,null,4,5]
Output: [1,2,3,null,null,4,5]

Input: root = []
Output: []

Input: root = [1]
Output: [1]
```

**Pattern:** Tree Traversal (DFS) + String Manipulation

**Time Complexity:** O(n) for both serialize and deserialize
**Space Complexity:** O(n)

**Solution:**
```python
class Codec:
    def serialize(self, root):
        """Encodes a tree to a single string."""
        if not root:
            return "null"
        return str(root.val) + "," + self.serialize(root.left) + "," + self.serialize(root.right)
    
    def deserialize(self, data):
        """Decodes your encoded data to tree."""
        def helper(nodes):
            val = next(nodes)
            if val == "null":
                return None
            node = TreeNode(int(val))
            node.left = helper(nodes)
            node.right = helper(nodes)
            return node
        
        return helper(iter(data.split(",")))
```

---

### Problem 17: 399. Evaluate Division - Medium

**Description:** Given equations and values, evaluate queries where each query is a division.

**Constraints:**
- 1 <= equations.length <= 20
- equations[i].length == 2
- 1 <= Ai.length, Bi.length <= 5
- values.length == equations.length
- 0.0 < values[i] <= 20.0
- 1 <= queries.length <= 20
- queries[i].length == 2
- 1 <= Cj.length, Dj.length <= 5
- Ai, Bi, Cj, Dj consist of lowercase English letters and digits

**Examples:**
```
Input: equations = [["a","b"],["b","c"]], values = [2.0,3.0], queries = [["a","c"],["b","a"],["a","e"],["a","a"],["x","x"]]
Output: [6.00000,0.50000,-1.00000,1.00000,-1.00000]
```

**Pattern:** Graph + DFS

**Time Complexity:** O((V + E) × Q) where Q = queries
**Space Complexity:** O(V + E)

**Solution:**
```python
from collections import defaultdict

def calcEquation(equations: List[List[str]], values: List[float], queries: List[List[str]]) -> List[float]:
    graph = defaultdict(dict)
    
    for (a, b), value in zip(equations, values):
        graph[a][b] = value
        graph[b][a] = 1 / value
    
    def dfs(start, end, visited):
        if start not in graph or end not in graph:
            return -1.0
        if start == end:
            return 1.0
        
        visited.add(start)
        
        for neighbor in graph[start]:
            if neighbor not in visited:
                result = dfs(neighbor, end, visited)
                if result != -1.0:
                    return graph[start][neighbor] * result
        
        return -1.0
    
    return [dfs(a, b, set()) for a, b in queries]
```

---

### Problem 18: 827. Making A Large Island - Hard

**Description:** You are given an n x n binary matrix grid. Change at most one 0 to a 1 and return the size of the largest island.

**Constraints:**
- n == grid.length
- n == grid[i].length
- 1 <= n <= 500
- grid[i][j] is either 0 or 1

**Examples:**
```
Input: grid = [[1,0],[0,1]]
Output: 3

Input: grid = [[1,1],[1,0]]
Output: 4

Input: grid = [[1,1],[1,1]]
Output: 4
```

**Pattern:** DFS + Hash Map

**Time Complexity:** O(n²)
**Space Complexity:** O(n²)

**Solution:**
```python
def largestIsland(grid: List[List[int]]) -> int:
    n = len(grid)
    island_id = 2
    island_sizes = {0: 0}
    
    def dfs(r, c, island_id):
        if r < 0 or r >= n or c < 0 or c >= n or grid[r][c] != 1:
            return 0
        grid[r][c] = island_id
        size = 1
        for dr, dc in [(0, 1), (0, -1), (1, 0), (-1, 0)]:
            size += dfs(r + dr, c + dc, island_id)
        return size
    
    for r in range(n):
        for c in range(n):
            if grid[r][c] == 1:
                island_sizes[island_id] = dfs(r, c, island_id)
                island_id += 1
    
    max_size = max(island_sizes.values()) if island_sizes else 0
    
    for r in range(n):
        for c in range(n):
            if grid[r][c] == 0:
                neighbors = set()
                for dr, dc in [(0, 1), (0, -1), (1, 0), (-1, 0)]:
                    nr, nc = r + dr, c + dc
                    if 0 <= nr < n and 0 <= nc < n:
                        neighbors.add(grid[nr][nc])
                
                size = 1 + sum(island_sizes[island_id] for island_id in neighbors)
                max_size = max(max_size, size)
    
    return max_size
```

---

### Problem 19: 886. Possible Bipartition - Medium

**Description:** Given a set of people and a list of pairs of people who dislike each other, check if it's possible to split everyone into two groups.

**Constraints:**
- 1 <= n <= 2000
- 0 <= dislikes.length <= 10^4
- dislikes[i].length == 2
- 1 <= dislikes[i][j] <= n
- dislikes[i][0] < dislikes[i][1]
- There are no repeated edges

**Examples:**
```
Input: n = 4, dislikes = [[1,2],[1,3],[2,4]]
Output: true

Input: n = 3, dislikes = [[1,2],[1,3],[2,3]]
Output: false
```

**Pattern:** Graph Coloring + DFS

**Time Complexity:** O(V + E)
**Space Complexity:** O(V + E)

**Solution:**
```python
from collections import defaultdict

def possibleBipartition(n: int, dislikes: List[List[int]]) -> bool:
    graph = defaultdict(list)
    for a, b in dislikes:
        graph[a].append(b)
        graph[b].append(a)
    
    color = {}
    
    def dfs(node, c):
        if node in color:
            return color[node] == c
        
        color[node] = c
        
        for neighbor in graph[node]:
            if not dfs(neighbor, 1 - c):
                return False
        
        return True
    
    for node in range(1, n + 1):
        if node not in color:
            if not dfs(node, 0):
                return False
    
    return True
```

---

### Problem 20: 934. Shortest Bridge - Medium

**Description:** You are given an n x n binary matrix grid where 1 represents land and 0 represents water. Find the smallest number of 0's you must flip to connect the two islands.

**Constraints:**
- n == grid.length == grid[i].length
- 2 <= n <= 100
- grid[i][j] is either 0 or 1
- There are exactly two islands in grid

**Examples:**
```
Input: grid = [[0,1],[1,0]]
Output: 1

Input: grid = [[0,1,0],[0,0,0],[0,0,1]]
Output: 2

Input: grid = [[1,1,1,1,1],[1,0,0,0,1],[1,0,1,0,1],[1,0,0,0,1],[1,1,1,1,1]]
Output: 1
```

**Pattern:** DFS + BFS

**Time Complexity:** O(n²)
**Space Complexity:** O(n²)

**Solution:**
```python
from collections import deque

def shortestBridge(grid: List[List[int]]) -> int:
    n = len(grid)
    queue = deque()
    found = False
    
    def dfs(r, c):
        if r < 0 or r >= n or c < 0 or c >= n or grid[r][c] != 1:
            return
        grid[r][c] = 2
        queue.append((r, c))
        dfs(r + 1, c)
        dfs(r - 1, c)
        dfs(r, c + 1)
        dfs(r, c - 1)
    
    for r in range(n):
        if found:
            break
        for c in range(n):
            if grid[r][c] == 1:
                dfs(r, c)
                found = True
                break
    
    steps = 0
    while queue:
        for _ in range(len(queue)):
            r, c = queue.popleft()
            for dr, dc in [(0, 1), (0, -1), (1, 0), (-1, 0)]:
                nr, nc = r + dr, c + dc
                if 0 <= nr < n and 0 <= nc < n:
                    if grid[nr][nc] == 1:
                        return steps
                    if grid[nr][nc] == 0:
                        grid[nr][nc] = 2
                        queue.append((nr, nc))
        steps += 1
    
    return -1
```

---

## BACKTRACKING PROBLEMS

---

### Problem 21: 22. Generate Parentheses - Medium

**Description:** Generate all combinations of well-formed parentheses given n pairs.

**Constraints:**
- 1 <= n <= 8

**Examples:**
```
Input: n = 3
Output: ["((()))","(()())","(())()","()(())","()()()"]

Input: n = 1
Output: ["()"]
```

**Pattern:** Backtracking

**Time Complexity:** O(4^n / √n) - Catalan number
**Space Complexity:** O(n) for recursion stack

**Solution:**
```python
def generateParenthesis(n: int) -> List[str]:
    result = []
    
    def backtrack(current, open_count, close_count):
        if len(current) == 2 * n:
            result.append(current)
            return
        
        if open_count < n:
            backtrack(current + '(', open_count + 1, close_count)
        
        if close_count < open_count:
            backtrack(current + ')', open_count, close_count + 1)
    
    backtrack('', 0, 0)
    return result
```

---

### Problem 22: 47. Permutations II - Medium

**Description:** Given a collection of numbers that might contain duplicates, return all possible unique permutations.

**Constraints:**
- 1 <= nums.length <= 8
- -10 <= nums[i] <= 10

**Examples:**
```
Input: nums = [1,1,2]
Output: [[1,1,2],[1,2,1],[2,1,1]]

Input: nums = [1,2,3]
Output: [[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
```

**Pattern:** Backtracking + Sorting

**Time Complexity:** O(n! × n)
**Space Complexity:** O(n)

**Solution:**
```python
def permuteUnique(nums: List[int]) -> List[List[int]]:
    result = []
    nums.sort()
    
    def backtrack(path, used):
        if len(path) == len(nums):
            result.append(path[:])
            return
        
        for i in range(len(nums)):
            if used[i]:
                continue
            if i > 0 and nums[i] == nums[i - 1] and not used[i - 1]:
                continue
            
            path.append(nums[i])
            used[i] = True
            backtrack(path, used)
            path.pop()
            used[i] = False
    
    backtrack([], [False] * len(nums))
    return result
```

---

### Problem 23: 212. Word Search II - Hard

**Description:** Given an m x n board of characters and a list of strings words, return all words on the board.

**Constraints:**
- m == board.length
- n == board[i].length
- 1 <= m, n <= 12
- board[i][j] is a lowercase English letter
- 1 <= words.length <= 3 * 10^4
- 1 <= words[i].length <= 10
- words[i] consists of lowercase English letters
- All strings of words are unique

**Examples:**
```
Input: board = [["o","a","a","n"],["e","t","a","e"],["i","h","k","r"],["i","f","l","v"]], 
       words = ["oath","pea","eat","rain"]
Output: ["eat","oath"]

Input: board = [["a","b"],["c","d"]], words = ["abcb"]
Output: []
```

**Pattern:** Trie + Backtracking + DFS

**Time Complexity:** O(m × n × 4^L) where L = max word length
**Space Complexity:** O(w × L) where w = number of words

**Solution:**
```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.word = None

def findWords(board: List[List[str]], words: List[str]) -> List[str]:
    root = TrieNode()
    
    for word in words:
        node = root
        for char in word:
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
        node.word = word
    
    rows, cols = len(board), len(board[0])
    result = []
    
    def dfs(r, c, node):
        char = board[r][c]
        if char not in node.children:
            return
        
        next_node = node.children[char]
        if next_node.word:
            result.append(next_node.word)
            next_node.word = None
        
        board[r][c] = '#'
        
        for dr, dc in [(0, 1), (0, -1), (1, 0), (-1, 0)]:
            nr, nc = r + dr, c + dc
            if 0 <= nr < rows and 0 <= nc < cols and board[nr][nc] != '#':
                dfs(nr, nc, next_node)
        
        board[r][c] = char
    
    for r in range(rows):
        for c in range(cols):
            dfs(r, c, root)
    
    return result
```

---

## DYNAMIC PROGRAMMING (DP) PROBLEMS

---

### Problem 24: 322. Coin Change - Medium

**Description:** Given an integer array coins and an integer amount, return the fewest number of coins needed to make up that amount.

**Constraints:**
- 1 <= coins.length <= 12
- 1 <= coins[i] <= 2^31 - 1
- 0 <= amount <= 10^4

**Examples:**
```
Input: coins = [1,2,5], amount = 11
Output: 3
Explanation: 11 = 5 + 5 + 1

Input: coins = [2], amount = 3
Output: -1

Input: coins = [1], amount = 0
Output: 0
```

**Pattern:** Dynamic Programming (Bottom-Up)

**Time Complexity:** O(amount × n) where n = number of coins
**Space Complexity:** O(amount)

**Solution:**
```python
def coinChange(coins: List[int], amount: int) -> int:
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0
    
    for i in range(1, amount + 1):
        for coin in coins:
            if i >= coin:
                dp[i] = min(dp[i], dp[i - coin] + 1)
    
    return dp[amount] if dp[amount] != float('inf') else -1
```

---

### Problem 25: 72. Edit Distance - Medium

**Description:** Given two strings word1 and word2, return the minimum number of operations required to convert word1 to word2.

**Constraints:**
- 0 <= word1.length, word2.length <= 500
- word1 and word2 consist of lowercase English letters

**Examples:**
```
Input: word1 = "horse", word2 = "ros"
Output: 3

Input: word1 = "intention", word2 = "execution"
Output: 5
```

**Pattern:** Dynamic Programming (2D DP)

**Time Complexity:** O(m × n)
**Space Complexity:** O(m × n)

**Solution:**
```python
def minDistance(word1: str, word2: str) -> int:
    m, n = len(word1), len(word2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    for i in range(m + 1):
        dp[i][0] = i
    for j in range(n + 1):
        dp[0][j] = j
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if word1[i - 1] == word2[j - 1]:
                dp[i][j] = dp[i - 1][j - 1]
            else:
                dp[i][j] = 1 + min(dp[i - 1][j], dp[i][j - 1], dp[i - 1][j - 1])
    
    return dp[m][n]
```

---

### Problem 26: 300. Longest Increasing Subsequence - Medium

**Description:** Given an integer array nums, return the length of the longest strictly increasing subsequence.

**Constraints:**
- 1 <= nums.length <= 2500
- -10^4 <= nums[i] <= 10^4

**Examples:**
```
Input: nums = [10,9,2,5,3,7,101,18]
Output: 4

Input: nums = [0,1,0,3,2,3]
Output: 4

Input: nums = [7,7,7,7,7,7,7]
Output: 1
```

**Pattern:** Dynamic Programming / Binary Search

**Time Complexity:** O(n²) for DP, O(n log n) for binary search
**Space Complexity:** O(n)

**Solution:**
```python
# DP Solution
def lengthOfLIS(nums: List[int]) -> int:
    if not nums:
        return 0
    
    dp = [1] * len(nums)
    
    for i in range(1, len(nums)):
        for j in range(i):
            if nums[i] > nums[j]:
                dp[i] = max(dp[i], dp[j] + 1)
    
    return max(dp)

# Optimized Binary Search Solution
import bisect

def lengthOfLIS_optimized(nums: List[int]) -> int:
    tails = []
    
    for num in nums:
        pos = bisect.bisect_left(tails, num)
        if pos == len(tails):
            tails.append(num)
        else:
            tails[pos] = num
    
    return len(tails)
```

---

### Problem 27: 887. Super Egg Drop - Hard

**Description:** Given k eggs and n floors, find the minimum number of moves needed to determine with certainty what the highest floor is from which an egg can be dropped without breaking.

**Constraints:**
- 1 <= k <= 100
- 1 <= n <= 10^4

**Examples:**
```
Input: k = 1, n = 2
Output: 2

Input: k = 2, n = 6
Output: 3

Input: k = 3, n = 14
Output: 4
```

**Pattern:** Dynamic Programming + Binary Search

**Time Complexity:** O(k × n log n)
**Space Complexity:** O(k × n)

**Solution:**
```python
def superEggDrop(k: int, n: int) -> int:
    memo = {}
    
    def dp(k, n):
        if (k, n) in memo:
            return memo[(k, n)]
        
        if n == 0:
            return 0
        if k == 1:
            return n
        
        left, right = 1, n
        result = n
        
        while left <= right:
            mid = (left + right) // 2
            broken = dp(k - 1, mid - 1)
            not_broken = dp(k, n - mid)
            
            worst = 1 + max(broken, not_broken)
            
            if broken > not_broken:
                right = mid - 1
            else:
                left = mid + 1
            
            result = min(result, worst)
        
        memo[(k, n)] = result
        return result
    
    return dp(k, n)
```

---

### Problem 28: 32. Longest Valid Parentheses - Hard

**Description:** Given a string containing just '(' and ')', find the length of the longest valid parentheses substring.

**Constraints:**
- 0 <= s.length <= 3 * 10^4
- s[i] is '(', or ')'

**Examples:**
```
Input: s = "(()"
Output: 2

Input: s = ")()())"
Output: 4

Input: s = ""
Output: 0
```

**Pattern:** Stack / Dynamic Programming

**Time Complexity:** O(n)
**Space Complexity:** O(n)

**Solution:**
```python
def longestValidParentheses(s: str) -> int:
    stack = [-1]
    max_length = 0
    
    for i, char in enumerate(s):
        if char == '(':
            stack.append(i)
        else:
            stack.pop()
            if not stack:
                stack.append(i)
            else:
                max_length = max(max_length, i - stack[-1])
    
    return max_length
```

---

### Problem 29: 5. Longest Palindromic Substring - Medium

**Description:** Given a string s, return the longest palindromic substring in s.

**Constraints:**
- 1 <= s.length <= 1000
- s consist of only digits and English letters

**Examples:**
```
Input: s = "babad"
Output: "bab"

Input: s = "cbbd"
Output: "bb"
```

**Pattern:** Expand Around Center / Dynamic Programming

**Time Complexity:** O(n²)
**Space Complexity:** O(1)

**Solution:**
```python
def longestPalindrome(s: str) -> str:
    def expand_around_center(left, right):
        while left >= 0 and right < len(s) and s[left] == s[right]:
            left -= 1
            right += 1
        return right - left - 1
    
    start, end = 0, 0
    
    for i in range(len(s)):
        len1 = expand_around_center(i, i)
        len2 = expand_around_center(i, i + 1)
        max_len = max(len1, len2)
        
        if max_len > end - start:
            start = i - (max_len - 1) // 2
            end = i + max_len // 2
    
    return s[start:end + 1]
```

---

### Problem 30: 70. Climbing Stairs - Easy

**Description:** You are climbing a staircase with n steps. Each time you can climb 1 or 2 steps. In how many distinct ways can you climb to the top?

**Constraints:**
- 1 <= n <= 45

**Examples:**
```
Input: n = 2
Output: 2

Input: n = 3
Output: 3
```

**Pattern:** Dynamic Programming / Fibonacci

**Time Complexity:** O(n)
**Space Complexity:** O(1)

**Solution:**
```python
def climbStairs(n: int) -> int:
    if n <= 2:
        return n
    
    prev, curr = 1, 2
    
    for i in range(3, n + 1):
        prev, curr = curr, prev + curr
    
    return curr
```

---

### Problem 31: 123. Best Time to Buy and Sell Stock III - Hard

**Description:** Design an algorithm to find the maximum profit with at most two transactions.

**Constraints:**
- 1 <= prices.length <= 10^5
- 0 <= prices[i] <= 10^5

**Examples:**
```
Input: prices = [3,3,5,0,0,3,1,4]
Output: 6

Input: prices = [1,2,3,4,5]
Output: 4

Input: prices = [7,6,4,3,1]
Output: 0
```

**Pattern:** Dynamic Programming (State Machine)

**Time Complexity:** O(n)
**Space Complexity:** O(1)

**Solution:**
```python
def maxProfit(prices: List[int]) -> int:
    buy1 = buy2 = float('-inf')
    sell1 = sell2 = 0
    
    for price in prices:
        buy1 = max(buy1, -price)
        sell1 = max(sell1, buy1 + price)
        buy2 = max(buy2, sell1 - price)
        sell2 = max(sell2, buy2 + price)
    
    return sell2
```

---

### Problem 32: 124. Binary Tree Maximum Path Sum - Hard

**Description:** Given a non-empty binary tree, find the maximum path sum.

**Constraints:**
- The number of nodes in the tree is in the range [1, 3 * 10^4]
- -1000 <= Node.val <= 1000

**Examples:**
```
Input: root = [1,2,3]
Output: 6

Input: root = [-10,9,20,null,null,15,7]
Output: 42
```

**Pattern:** DFS + Post-order Traversal

**Time Complexity:** O(n)
**Space Complexity:** O(h) where h is height

**Solution:**
```python
def maxPathSum(root: Optional[TreeNode]) -> int:
    max_sum = float('-inf')
    
    def dfs(node):
        nonlocal max_sum
        if not node:
            return 0
        
        left = max(0, dfs(node.left))
        right = max(0, dfs(node.right))
        
        max_sum = max(max_sum, node.val + left + right)
        
        return node.val + max(left, right)
    
    dfs(root)
    return max_sum
```

---

### Problem 33: 139. Word Break - Medium

**Description:** Given a string s and a dictionary of strings wordDict, return true if s can be segmented into dictionary words.

**Constraints:**
- 1 <= s.length <= 300
- 1 <= wordDict.length <= 1000
- 1 <= wordDict[i].length <= 20
- s and wordDict[i] consist of only lowercase English letters
- All strings of wordDict are unique

**Examples:**
```
Input: s = "leetcode", wordDict = ["leet","code"]
Output: true

Input: s = "applepenapple", wordDict = ["apple","pen"]
Output: true

Input: s = "catsandog", wordDict = ["cats","dog","sand","and","cat"]
Output: false
```

**Pattern:** Dynamic Programming

**Time Complexity:** O(n² × m)
**Space Complexity:** O(n)

**Solution:**
```python
def wordBreak(s: str, wordDict: List[str]) -> bool:
    word_set = set(wordDict)
    dp = [False] * (len(s) + 1)
    dp[0] = True
    
    for i in range(1, len(s) + 1):
        for j in range(i):
            if dp[j] and s[j:i] in word_set:
                dp[i] = True
                break
    
    return dp[len(s)]
```

---

### Problem 34: 128. Longest Consecutive Sequence - Medium

**Description:** Given an unsorted array of integers, find the length of the longest consecutive elements sequence.

**Constraints:**
- 0 <= nums.length <= 10^5
- -10^9 <= nums[i] <= 10^9

**Examples:**
```
Input: nums = [100,4,200,1,3,2]
Output: 4

Input: nums = [0,3,7,2,5,8,4,6,0,1]
Output: 9
```

**Pattern:** Hash Set

**Time Complexity:** O(n)
**Space Complexity:** O(n)

**Solution:**
```python
def longestConsecutive(nums: List[int]) -> int:
    num_set = set(nums)
    longest = 0
    
    for num in num_set:
        if num - 1 not in num_set:
            current_num = num
            current_length = 1
            
            while current_num + 1 in num_set:
                current_num += 1
                current_length += 1
            
            longest = max(longest, current_length)
    
    return longest
```

---

## STACK PROBLEMS

---

### Problem 35: 20. Valid Parentheses - Easy

**Description:** Given a string s containing just the characters '(', ')', '{', '}', '[' and ']', determine if the input string is valid.

**Constraints:**
- 1 <= s.length <= 10^4
- s consists of parentheses only '()[]{}'

**Examples:**
```
Input: s = "()"
Output: true

Input: s = "()[]{}"
Output: true

Input: s = "(]"
Output: false
```

**Pattern:** Stack

**Time Complexity:** O(n)
**Space Complexity:** O(n)

**Solution:**
```python
def isValid(s: str) -> bool:
    stack = []
    mapping = {')': '(', '}': '{', ']': '['}
    
    for char in s:
        if char in mapping:
            top = stack.pop() if stack else '#'
            if mapping[char] != top:
                return False
        else:
            stack.append(char)
    
    return not stack
```

---

### Problem 36: 71. Simplify Path - Medium

**Description:** Given an absolute path for a Unix-style file system, simplify it.

**Constraints:**
- 1 <= path.length <= 3000
- path consists of English letters, digits, period '.', slash '/' or '_'
- path is a valid absolute Unix path

**Examples:**
```
Input: path = "/home/"
Output: "/home"

Input: path = "/../"
Output: "/"

Input: path = "/home//foo/"
Output: "/home/foo"
```

**Pattern:** Stack

**Time Complexity:** O(n)
**Space Complexity:** O(n)

**Solution:**
```python
def simplifyPath(path: str) -> str:
    stack = []
    parts = path.split('/')
    
    for part in parts:
        if part == '..' and stack:
            stack.pop()
        elif part and part != '.' and part != '..':
            stack.append(part)
    
    return '/' + '/'.join(stack)
```

---

### Problem 37: 224. Basic Calculator - Hard

**Description:** Implement a basic calculator to evaluate expression with +, -, (, ) and spaces.

**Constraints:**
- 1 <= s.length <= 3 * 10^5
- s consists of digits, '+', '-', '(', ')', and ' '
- s represents a valid expression
- '+' is not used as a unary operation
- '-' could be used as a unary operation
- There will be no two consecutive operators in the input
- Every number and running calculation will fit in a signed 32-bit integer

**Examples:**
```
Input: s = "1 + 1"
Output: 2

Input: s = " 2-1 + 2 "
Output: 3

Input: s = "(1+(4+5+2)-3)+(6+8)"
Output: 23
```

**Pattern:** Stack

**Time Complexity:** O(n)
**Space Complexity:** O(n)

**Solution:**
```python
def calculate(s: str) -> int:
    stack = []
    num = 0
    sign = 1
    result = 0
    
    for char in s:
        if char.isdigit():
            num = num * 10 + int(char)
        elif char == '+':
            result += sign * num
            num = 0
            sign = 1
        elif char == '-':
            result += sign * num
            num = 0
            sign = -1
        elif char == '(':
            stack.append(result)
            stack.append(sign)
            result = 0
            sign = 1
        elif char == ')':
            result += sign * num
            num = 0
            result *= stack.pop()
            result += stack.pop()
    
    result += sign * num
    return result
```

---

### Problem 38: 394. Decode String - Medium

**Description:** Decode string with pattern k[encoded_string].

**Constraints:**
- 1 <= s.length <= 30
- s consists of lowercase English letters, digits, and square brackets '[]'
- s is guaranteed to be a valid input
- All the integers in s are in the range [1, 300]

**Examples:**
```
Input: s = "3[a]2[bc]"
Output: "aaabcbc"

Input: s = "3[a2[c]]"
Output: "accaccacc"

Input: s = "2[abc]3[cd]ef"
Output: "abcabccdcdcdef"
```

**Pattern:** Stack

**Time Complexity:** O(n × k)
**Space Complexity:** O(n)

**Solution:**
```python
def decodeString(s: str) -> str:
    stack = []
    current_num = 0
    current_str = ""
    
    for char in s:
        if char.isdigit():
            current_num = current_num * 10 + int(char)
        elif char == '[':
            stack.append(current_str)
            stack.append(current_num)
            current_str = ""
            current_num = 0
        elif char == ']':
            num = stack.pop()
            prev_str = stack.pop()
            current_str = prev_str + current_str * num
        else:
            current_str += char
    
    return current_str
```

---

### Problem 39: 735. Asteroid Collision - Medium

**Description:** Given an array of asteroids, find out the state after all collisions.

**Constraints:**
- 2 <= asteroids.length <= 10^4
- -1000 <= asteroids[i] <= 1000
- asteroids[i] != 0

**Examples:**
```
Input: asteroids = [5,10,-5]
Output: [5,10]

Input: asteroids = [8,-8]
Output: []

Input: asteroids = [10,2,-5]
Output: [10]
```

**Pattern:** Stack

**Time Complexity:** O(n)
**Space Complexity:** O(n)

**Solution:**
```python
def asteroidCollision(asteroids: List[int]) -> List[int]:
    stack = []
    
    for asteroid in asteroids:
        while stack and asteroid < 0 < stack[-1]:
            if stack[-1] < -asteroid:
                stack.pop()
                continue
            elif stack[-1] == -asteroid:
                stack.pop()
            break
        else:
            stack.append(asteroid)
    
    return stack
```

---

### Problem 40: 772. Basic Calculator III - Hard

**Description:** Calculator with +, -, *, /, (, ) operations.

**Constraints:**
- 1 <= s <= 10^4
- s consists of digits, '+', '-', '*', '/', '(', and ')'
- s is a valid expression

**Examples:**
```
Input: s = "1 + 1"
Output: 2

Input: s = " 6-4 / 2 "
Output: 4

Input: s = "2*(5+5*2)/3+(6/2+8)"
Output: 21
```

**Pattern:** Stack + Recursion

**Time Complexity:** O(n)
**Space Complexity:** O(n)

**Solution:**
```python
def calculate(s: str) -> int:
    def helper(s, index):
        stack = []
        num = 0
        operation = '+'
        
        while index < len(s):
            char = s[index]
            
            if char.isdigit():
                num = num * 10 + int(char)
            elif char == '(':
                num, index = helper(s, index + 1)
            
            if char in '+-*/)' or index == len(s) - 1:
                if operation == '+':
                    stack.append(num)
                elif operation == '-':
                    stack.append(-num)
                elif operation == '*':
                    stack.append(stack.pop() * num)
                elif operation == '/':
                    stack.append(int(stack.pop() / num))
                
                if char == ')':
                    return sum(stack), index
                
                operation = char
                num = 0
            
            index += 1
        
        return sum(stack), index
    
    return helper(s, 0)[0]
```

---

## HEAP (PRIORITY QUEUE) PROBLEMS

---

### Problem 41: 347. Top K Frequent Elements - Medium

**Description:** Return the k most frequent elements.

**Constraints:**
- 1 <= nums.length <= 10^5
- -10^4 <= nums[i] <= 10^4
- k is in the range [1, the number of unique elements in the array]
- The answer is guaranteed to be unique

**Examples:**
```
Input: nums = [1,1,1,2,2,3], k = 2
Output: [1,2]

Input: nums = [1], k = 1
Output: [1]
```

**Pattern:** Hash Map + Heap

**Time Complexity:** O(n log k)
**Space Complexity:** O(n)

**Solution:**
```python
from collections import Counter
import heapq

def topKFrequent(nums: List[int], k: int) -> List[int]:
    count = Counter(nums)
    return heapq.nlargest(k, count.keys(), key=count.get)
```

---

### Problem 42: 23. Merge k Sorted Lists - Hard

**Description:** Merge k sorted linked lists into one sorted list.

**Constraints:**
- k == lists.length
- 0 <= k <= 10^4
- 0 <= lists[i].length <= 500
- -10^4 <= lists[i][j] <= 10^4
- lists[i] is sorted in ascending order
- Sum of lists[i].length will not exceed 10^4

**Examples:**
```
Input: lists = [[1,4,5],[1,3,4],[2,6]]
Output: [1,1,2,3,4,4,5,6]

Input: lists = []
Output: []

Input: lists = [[]]
Output: []
```

**Pattern:** Heap (Min Heap)

**Time Complexity:** O(N log k) where N = total nodes
**Space Complexity:** O(k)

**Solution:**
```python
import heapq

def mergeKLists(lists: List[Optional[ListNode]]) -> Optional[ListNode]:
    heap = []
    
    for i, node in enumerate(lists):
        if node:
            heapq.heappush(heap, (node.val, i, node))
    
    dummy = ListNode(0)
    current = dummy
    
    while heap:
        val, i, node = heapq.heappop(heap)
        current.next = node
        current = current.next
        
        if node.next:
            heapq.heappush(heap, (node.next.val, i, node.next))
    
    return dummy.next
```

---

### Problem 43: 253. Meeting Rooms II - Medium

**Description:** Find minimum conference rooms required.

**Constraints:**
- 1 <= intervals.length <= 10^4
- 0 <= starti < endi <= 10^6

**Examples:**
```
Input: intervals = [[0,30],[5,10],[15,20]]
Output: 2

Input: intervals = [[7,10],[2,4]]
Output: 1
```

**Pattern:** Heap (Min Heap)

**Time Complexity:** O(n log n)
**Space Complexity:** O(n)

**Solution:**
```python
import heapq

def minMeetingRooms(intervals: List[List[int]]) -> int:
    if not intervals:
        return 0
    
    intervals.sort(key=lambda x: x[0])
    heap = []
    
    heapq.heappush(heap, intervals[0][1])
    
    for i in range(1, len(intervals)):
        if intervals[i][0] >= heap[0]:
            heapq.heappop(heap)
        heapq.heappush(heap, intervals[i][1])
    
    return len(heap)
```

---

### Problem 44: 2402. Meeting Rooms III - Hard

**Description:** Find room that held most meetings.

**Constraints:**
- 1 <= n <= 100
- 1 <= meetings.length <= 10^5
- meetings[i].length == 2
- 0 <= starti < endi <= 5 * 10^5
- All the values of starti are unique

**Examples:**
```
Input: n = 2, meetings = [[0,10],[1,5],[2,7],[3,4]]
Output: 0

Input: n = 3, meetings = [[1,20],[2,10],[3,5],[4,9],[6,8]]
Output: 1
```

**Pattern:** Heap (Priority Queue)

**Time Complexity:** O(m log n)
**Space Complexity:** O(n)

**Solution:**
```python
import heapq

def mostBooked(n: int, meetings: List[List[int]]) -> int:
    meetings.sort()
    available = list(range(n))
    heapq.heapify(available)
    occupied = []
    room_count = [0] * n
    
    for start, end in meetings:
        while occupied and occupied[0][0] <= start:
            _, room = heapq.heappop(occupied)
            heapq.heappush(available, room)
        
        if available:
            room = heapq.heappop(available)
            heapq.heappush(occupied, (end, room))
        else:
            earliest_end, room = heapq.heappop(occupied)
            heapq.heappush(occupied, (earliest_end + end - start, room))
        
        room_count[room] += 1
    
    return room_count.index(max(room_count))
```

---

## BINARY SEARCH PROBLEMS

---

### Problem 45: 69. Sqrt(x) - Easy

**Description:** Compute and return the square root of x.

**Constraints:**
- 0 <= x <= 2^31 - 1

**Examples:**
```
Input: x = 4
Output: 2

Input: x = 8
Output: 2
```

**Pattern:** Binary Search

**Time Complexity:** O(log x)
**Space Complexity:** O(1)

**Solution:**
```python
def mySqrt(x: int) -> int:
    if x < 2:
        return x
    
    left, right = 1, x // 2
    
    while left <= right:
        mid = (left + right) // 2
        square = mid * mid
        
        if square == x:
            return mid
        elif square < x:
            left = mid + 1
        else:
            right = mid - 1
    
    return right
```

---

### Problem 46: 153. Find Minimum in Rotated Sorted Array - Medium

**Description:** Find minimum element in rotated sorted array.

**Constraints:**
- n == nums.length
- 1 <= n <= 5000
- -5000 <= nums[i] <= 5000
- All integers of nums are unique
- nums is sorted and rotated between 1 and n times

**Examples:**
```
Input: nums = [3,4,5,1,2]
Output: 1

Input: nums = [4,5,6,7,0,1,2]
Output: 0

Input: nums = [11,13,15,17]
Output: 11
```

**Pattern:** Binary Search

**Time Complexity:** O(log n)
**Space Complexity:** O(1)

**Solution:**
```python
def findMin(nums: List[int]) -> int:
    left, right = 0, len(nums) - 1
    
    while left < right:
        mid = (left + right) // 2
        
        if nums[mid] > nums[right]:
            left = mid + 1
        else:
            right = mid
    
    return nums[left]
```

---

### Problem 47: 1044. Longest Duplicate Substring - Hard

**Description:** Return any duplicated substring that has the longest possible length.

**Constraints:**
- 2 <= s.length <= 3 * 10^4
- s consists of lowercase English letters

**Examples:**
```
Input: s = "banana"
Output: "ana"

Input: s = "abcd"
Output: ""
```

**Pattern:** Binary Search + Rolling Hash

**Time Complexity:** O(n log n)
**Space Complexity:** O(n)

**Solution:**
```python
def longestDupSubstring(s: str) -> str:
    def search(length):
        seen = set()
        for i in range(len(s) - length + 1):
            substring = s[i:i + length]
            if substring in seen:
                return substring
            seen.add(substring)
        return None
    
    left, right = 1, len(s)
    result = ""
    
    while left <= right:
        mid = (left + right) // 2
        dup = search(mid)
        if dup:
            result = dup
            left = mid + 1
        else:
            right = mid - 1
    
    return result
```

---

## HASH MAP / HASH SET PROBLEMS

---

### Problem 48: 146. LRU Cache - Medium

**Description:** Design LRU cache.

**Constraints:**
- 1 <= capacity <= 3000
- 0 <= key <= 10^4
- 0 <= value <= 10^5
- At most 2 * 10^5 calls will be made to get and put

**Examples:**
```
Input:
["LRUCache", "put", "put", "get", "put", "get", "put", "get", "get", "get"]
[[2], [1, 1], [2, 2], [1], [3, 3], [2], [4, 4], [1], [3], [4]]
Output:
[null, null, null, 1, null, -1, null, -1, 3, 4]
```

**Pattern:** Hash Map + Doubly Linked List (or OrderedDict)

**Time Complexity:** O(1) for get and put
**Space Complexity:** O(capacity)

**Solution:**
```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity: int):
        self.cache = OrderedDict()
        self.capacity = capacity
    
    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1
        self.cache.move_to_end(key)
        return self.cache[key]
    
    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.capacity:
            self.cache.popitem(last=False)
```

---

### Problem 49: 49. Group Anagrams - Medium

**Description:** Group anagrams together.

**Constraints:**
- 1 <= strs.length <= 10^4
- 0 <= strs[i].length <= 100
- strs[i] consists of lowercase English letters

**Examples:**
```
Input: strs = ["eat","tea","tan","ate","nat","bat"]
Output: [["bat"],["nat","tan"],["ate","eat","tea"]]

Input: strs = [""]
Output: [[""]]

Input: strs = ["a"]
Output: [["a"]]
```

**Pattern:** Hash Map + Sorting

**Time Complexity:** O(n × k log k)
**Space Complexity:** O(n × k)

**Solution:**
```python
from collections import defaultdict

def groupAnagrams(strs: List[str]) -> List[List[str]]:
    anagrams = defaultdict(list)
    
    for s in strs:
        key = ''.join(sorted(s))
        anagrams[key].append(s)
    
    return list(anagrams.values())
```

---

## GRAPH ALGORITHMS PROBLEMS

---

### Problem 50: 207. Course Schedule - Medium

**Description:** Determine if you can finish all courses.

**Constraints:**
- 1 <= numCourses <= 2000
- 0 <= prerequisites.length <= 5000
- prerequisites[i].length == 2
- 0 <= ai, bi < numCourses
- All the pairs prerequisites[i] are unique

**Examples:**
```
Input: numCourses = 2, prerequisites = [[1,0]]
Output: true

Input: numCourses = 2, prerequisites = [[1,0],[0,1]]
Output: false
```

**Pattern:** Topological Sort / Cycle Detection

**Time Complexity:** O(V + E)
**Space Complexity:** O(V + E)

**Solution:**
```python
from collections import defaultdict, deque

def canFinish(numCourses: int, prerequisites: List[List[int]]) -> bool:
    graph = defaultdict(list)
    in_degree = [0] * numCourses
    
    for course, prereq in prerequisites:
        graph[prereq].append(course)
        in_degree[course] += 1
    
    queue = deque([i for i in range(numCourses) if in_degree[i] == 0])
    count = 0
    
    while queue:
        course = queue.popleft()
        count += 1
        
        for next_course in graph[course]:
            in_degree[next_course] -= 1
            if in_degree[next_course] == 0:
                queue.append(next_course)
    
    return count == numCourses
```

---

### Problem 51: 210. Course Schedule II - Medium

**Description:** Return course ordering.

**Constraints:**
- 1 <= numCourses <= 2000
- 0 <= prerequisites.length <= numCourses * (numCourses - 1)
- prerequisites[i].length == 2
- 0 <= ai, bi < numCourses
- ai != bi
- All the pairs [ai, bi] are distinct

**Examples:**
```
Input: numCourses = 2, prerequisites = [[1,0]]
Output: [0,1]

Input: numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]
Output: [0,2,1,3] or [0,1,2,3]
```

**Pattern:** Topological Sort

**Time Complexity:** O(V + E)
**Space Complexity:** O(V + E)

**Solution:**
```python
from collections import defaultdict, deque

def findOrder(numCourses: int, prerequisites: List[List[int]]) -> List[int]:
    graph = defaultdict(list)
    in_degree = [0] * numCourses
    
    for course, prereq in prerequisites:
        graph[prereq].append(course)
        in_degree[course] += 1
    
    queue = deque([i for i in range(numCourses) if in_degree[i] == 0])
    result = []
    
    while queue:
        course = queue.popleft()
        result.append(course)
        
        for next_course in graph[course]:
            in_degree[next_course] -= 1
            if in_degree[next_course] == 0:
                queue.append(next_course)
    
    return result if len(result) == numCourses else []
```

---

### Problem 52: 127. Word Ladder - Hard

**Description:** Find shortest transformation sequence.

**Constraints:**
- 1 <= beginWord.length <= 10
- endWord.length == beginWord.length
- 1 <= wordList.length <= 5000
- wordList[i].length == beginWord.length
- beginWord, endWord, and wordList[i] consist of lowercase English letters
- beginWord != endWord
- All strings in wordList are unique

**Examples:**
```
Input: beginWord = "hit", endWord = "cog", wordList = ["hot","dot","dog","lot","log","cog"]
Output: 5

Input: beginWord = "hit", endWord = "cog", wordList = ["hot","dot","dog","lot","log"]
Output: 0
```

**Pattern:** BFS

**Time Complexity:** O(M² × N)
**Space Complexity:** O(M × N)

**Solution:**
```python
from collections import deque

def ladderLength(beginWord: str, endWord: str, wordList: List[str]) -> int:
    if endWord not in wordList:
        return 0
    
    word_set = set(wordList)
    queue = deque([(beginWord, 1)])
    
    while queue:
        word, level = queue.popleft()
        
        if word == endWord:
            return level
        
        for i in range(len(word)):
            for c in 'abcdefghijklmnopqrstuvwxyz':
                next_word = word[:i] + c + word[i + 1:]
                if next_word in word_set:
                    word_set.remove(next_word)
                    queue.append((next_word, level + 1))
    
    return 0
```

---

## STRING MANIPULATION PROBLEMS

---

### Problem 53: 68. Text Justification - Hard

**Description:** Format text with full justification.

**Constraints:**
- 1 <= words.length <= 300
- 1 <= words[i].length <= 20
- words[i] consists of only English letters and symbols
- 1 <= maxWidth <= 100
- words[i].length <= maxWidth

**Examples:**
```
Input: words = ["This", "is", "an", "example", "of", "text", "justification."], maxWidth = 16
Output:
[
   "This    is    an",
   "example  of text",
   "justification.  "
]
```

**Pattern:** String Manipulation + Greedy

**Time Complexity:** O(n)
**Space Complexity:** O(n)

**Solution:**
```python
def fullJustify(words: List[str], maxWidth: int) -> List[str]:
    result = []
    current_line = []
    current_length = 0
    
    for word in words:
        if current_length + len(word) + len(current_line) > maxWidth:
            for i in range(maxWidth - current_length):
                current_line[i % (len(current_line) - 1 or 1)] += ' '
            result.append(''.join(current_line))
            current_line = []
            current_length = 0
        
        current_line.append(word)
        current_length += len(word)
    
    result.append(' '.join(current_line).ljust(maxWidth))
    return result
```

---

### Problem 54: 43. Multiply Strings - Medium

**Description:** Multiply two numbers represented as strings.

**Constraints:**
- 1 <= num1.length, num2.length <= 200
- num1 and num2 consist of digits only
- Both num1 and num2 do not contain any leading zero, except the number 0 itself

**Examples:**
```
Input: num1 = "2", num2 = "3"
Output: "6"

Input: num1 = "123", num2 = "456"
Output: "56088"
```

**Pattern:** Math Simulation

**Time Complexity:** O(m × n)
**Space Complexity:** O(m + n)

**Solution:**
```python
def multiply(num1: str, num2: str) -> str:
    if num1 == "0" or num2 == "0":
        return "0"
    
    m, n = len(num1), len(num2)
    result = [0] * (m + n)
    
    for i in range(m - 1, -1, -1):
        for j in range(n - 1, -1, -1):
            mul = int(num1[i]) * int(num2[j])
            p1, p2 = i + j, i + j + 1
            total = mul + result[p2]
            
            result[p2] = total % 10
            result[p1] += total // 10
    
    result_str = ''.join(map(str, result))
    return result_str.lstrip('0') or '0'
```

---

### Problem 55: 151. Reverse Words in a String - Medium

**Description:** Reverse the order of words.

**Constraints:**
- 1 <= s.length <= 10^4
- s contains English letters (upper-case and lower-case), digits, and spaces ' '
- There is at least one word in s

**Examples:**
```
Input: s = "the sky is blue"
Output: "blue is sky the"

Input: s = "  hello world  "
Output: "world hello"

Input: s = "a good   example"
Output: "example good a"
```

**Pattern:** String Manipulation

**Time Complexity:** O(n)
**Space Complexity:** O(n)

**Solution:**
```python
def reverseWords(s: str) -> str:
    return ' '.join(s.split()[::-1])
```

---

### Problem 56: 179. Largest Number - Medium

**Description:** Arrange numbers to form largest number.

**Constraints:**
- 1 <= nums.length <= 100
- 0 <= nums[i] <= 10^9

**Examples:**
```
Input: nums = [10,2]
Output: "210"

Input: nums = [3,30,34,5,9]
Output: "9534330"
```

**Pattern:** Sorting with Custom Comparator

**Time Complexity:** O(n log n)
**Space Complexity:** O(n)

**Solution:**
```python
from functools import cmp_to_key

def largestNumber(nums: List[int]) -> str:
    nums_str = list(map(str, nums))
    
    def compare(x, y):
        if x + y > y + x:
            return -1
        elif x + y < y + x:
            return 1
        else:
            return 0
    
    nums_str.sort(key=cmp_to_key(compare))
    result = ''.join(nums_str)
    
    return '0' if result[0] == '0' else result
```

---

### Problem 57: 271. Encode and Decode Strings - Medium

**Description:** Design encoding/decoding algorithm.

**Constraints:**
- 0 <= strs.length < 200
- 0 <= strs[i].length < 200
- strs[i] contains any possible characters out of 256 valid ASCII characters

**Examples:**
```
Input: strs = ["Hello","World"]
Output: ["Hello","World"]

Input: strs = [""]
Output: [""]
```

**Pattern:** String Encoding

**Time Complexity:** O(n)
**Space Complexity:** O(n)

**Solution:**
```python
class Codec:
    def encode(self, strs: List[str]) -> str:
        encoded = ""
        for s in strs:
            encoded += str(len(s)) + "#" + s
        return encoded
    
    def decode(self, s: str) -> List[str]:
        decoded = []
        i = 0
        
        while i < len(s):
            j = i
            while s[j] != '#':
                j += 1
            
            length = int(s[i:j])
            decoded.append(s[j + 1:j + 1 + length])
            i = j + 1 + length
        
        return decoded
```

---

### Problem 58: 316. Remove Duplicate Letters - Medium

**Description:** Remove duplicates keeping lexicographically smallest.

**Constraints:**
- 1 <= s.length <= 10^4
- s consists of lowercase English letters

**Examples:**
```
Input: s = "bcabc"
Output: "abc"

Input: s = "cbacdcbc"
Output: "acdb"
```

**Pattern:** Stack + Greedy

**Time Complexity:** O(n)
**Space Complexity:** O(1)

**Solution:**
```python
def removeDuplicateLetters(s: str) -> str:
    stack = []
    seen = set()
    last_occurrence = {char: i for i, char in enumerate(s)}
    
    for i, char in enumerate(s):
        if char not in seen:
            while stack and char < stack[-1] and i < last_occurrence[stack[-1]]:
                seen.remove(stack.pop())
            
            seen.add(char)
            stack.append(char)
    
    return ''.join(stack)
```

---

## SORTING & GREEDY PROBLEMS

---

### Problem 59: 56. Merge Intervals - Medium

**Description:** Merge overlapping intervals.

**Constraints:**
- 1 <= intervals.length <= 10^4
- intervals[i].length == 2
- 0 <= starti <= endi <= 10^4

**Examples:**
```
Input: intervals = [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]

Input: intervals = [[1,4],[4,5]]
Output: [[1,5]]
```

**Pattern:** Sorting + Greedy

**Time Complexity:** O(n log n)
**Space Complexity:** O(n)

**Solution:**
```python
def merge(intervals: List[List[int]]) -> List[List[int]]:
    intervals.sort(key=lambda x: x[0])
    merged = [intervals[0]]
    
    for start, end in intervals[1:]:
        if start <= merged[-1][1]:
            merged[-1][1] = max(merged[-1][1], end)
        else:
            merged.append([start, end])
    
    return merged
```

---

## TREE TRAVERSAL PROBLEMS

---

### Problem 60: 105. Construct Binary Tree from Preorder and Inorder - Medium

**Description:** Build tree from traversals.

**Constraints:**
- 1 <= preorder.length <= 3000
- inorder.length == preorder.length
- -3000 <= preorder[i], inorder[i] <= 3000
- preorder and inorder consist of unique values
- Each value of inorder also appears in preorder
- preorder is guaranteed to be the preorder traversal of the tree
- inorder is guaranteed to be the inorder traversal of the tree

**Examples:**
```
Input: preorder = [3,9,20,15,7], inorder = [9,3,15,20,7]
Output: [3,9,20,null,null,15,7]

Input: preorder = [-1], inorder = [-1]
Output: [-1]
```

**Pattern:** Recursion + Hash Map

**Time Complexity:** O(n)
**Space Complexity:** O(n)

**Solution:**
```python
def buildTree(preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
    inorder_map = {val: i for i, val in enumerate(inorder)}
    
    def helper(pre_left, pre_right, in_left, in_right):
        if pre_left > pre_right:
            return None
        
        root_val = preorder[pre_left]
        root = TreeNode(root_val)
        in_root = inorder_map[root_val]
        left_size = in_root - in_left
        
        root.left = helper(pre_left + 1, pre_left + left_size, 
                          in_left, in_root - 1)
        root.right = helper(pre_left + left_size + 1, pre_right, 
                           in_root + 1, in_right)
        
        return root
    
    return helper(0, len(preorder) - 1, 0, len(inorder) - 1)
```

---

### Problem 61: 113. Path Sum II - Medium

**Description:** Find all root-to-leaf paths with target sum.

**Constraints:**
- The number of nodes in the tree is in the range [0, 5000]
- -1000 <= Node.val <= 1000
- -1000 <= targetSum <= 1000

**Examples:**
```
Input: root = [5,4,8,11,null,13,4,7,2,null,null,5,1], targetSum = 22
Output: [[5,4,11,2],[5,8,4,5]]

Input: root = [1,2,3], targetSum = 5
Output: []

Input: root = [1,2], targetSum = 0
Output: []
```

**Pattern:** Backtracking + DFS

**Time Complexity:** O(n)
**Space Complexity:** O(h) where h is height

**Solution:**
```python
def pathSum(root: Optional[TreeNode], targetSum: int) -> List[List[int]]:
    if not root:
        return []
    
    result = []
    
    def dfs(node, path, remaining):
        if not node:
            return
        
        path.append(node.val)
        
        if not node.left and not node.right and remaining == node.val:
            result.append(path[:])
        
        dfs(node.left, path, remaining - node.val)
        dfs(node.right, path, remaining - node.val)
        
        path.pop()
    
    dfs(root, [], targetSum)
    return result
```

---

### Problem 62: 426. Convert BST to Sorted Doubly Linked List - Medium

**Description:** Convert BST to circular doubly linked list.

**Constraints:**
- The number of nodes in the tree is in the range [0, 2000]
- -1000 <= Node.val <= 1000
- All Node.val are unique

**Examples:**
```
Input: root = [4,2,5,1,3]
Output: [1,2,3,4,5]
```

**Pattern:** In-order Traversal

**Time Complexity:** O(n)
**Space Complexity:** O(h)

**Solution:**
```python
def treeToDoublyList(root: 'Node') -> 'Node':
    if not root:
        return None
    
    first = last = None
    
    def inorder(node):
        nonlocal first, last
        if not node:
            return
        
        inorder(node.left)
        
        if last:
            last.right = node
            node.left = last
        else:
            first = node
        
        last = node
        
        inorder(node.right)
    
    inorder(root)
    
    first.left = last
    last.right = first
    
    return first
```

---

## DESIGN / DATA STRUCTURE PROBLEMS

---

### Problem 63: 706. Design HashMap - Easy

**Description:** Implement hash map from scratch.

**Constraints:**
- 0 <= key, value <= 10^6
- At most 10^4 calls will be made to put, get, and remove

**Examples:**
```
Input:
["MyHashMap", "put", "put", "get", "get", "put", "get", "remove", "get"]
[[], [1, 1], [2, 2], [1], [3], [2, 1], [2], [2], [2]]
Output:
[null, null, null, 1, -1, null, 1, null, -1]
```

**Pattern:** Hash Table + Chaining

**Time Complexity:** O(1) average
**Space Complexity:** O(n)

**Solution:**
```python
class MyHashMap:
    def __init__(self):
        self.size = 1000
        self.buckets = [[] for _ in range(self.size)]
    
    def _hash(self, key: int) -> int:
        return key % self.size
    
    def put(self, key: int, value: int) -> None:
        bucket = self.buckets[self._hash(key)]
        for i, (k, v) in enumerate(bucket):
            if k == key:
                bucket[i] = (key, value)
                return
        bucket.append((key, value))
    
    def get(self, key: int) -> int:
        bucket = self.buckets[self._hash(key)]
        for k, v in bucket:
            if k == key:
                return v
        return -1
    
    def remove(self, key: int) -> None:
        bucket = self.buckets[self._hash(key)]
        for i, (k, v) in enumerate(bucket):
            if k == key:
                del bucket[i]
                return
```

---

### Problem 64: 2502. Design Memory Allocator - Medium

**Description:** Design memory allocation system.

**Constraints:**
- 1 <= n, size, mID <= 1000
- At most 1000 calls will be made to allocate and free

**Examples:**
```
Input:
["Allocator", "allocate", "allocate", "allocate", "free", "allocate"]
[[10], [1, 1], [1, 2], [1, 3], [2], [3, 4]]
Output:
[null, 0, 1, 2, 1, 3]
```

**Pattern:** Array Simulation

**Time Complexity:** O(n) per operation
**Space Complexity:** O(n)

**Solution:**
```python
class Allocator:
    def __init__(self, n: int):
        self.memory = [0] * n
        self.n = n
    
    def allocate(self, size: int, mID: int) -> int:
        count = 0
        for i in range(self.n):
            if self.memory[i] == 0:
                count += 1
                if count == size:
                    for j in range(i - size + 1, i + 1):
                        self.memory[j] = mID
                    return i - size + 1
            else:
                count = 0
        return -1
    
    def free(self, mID: int) -> int:
        count = 0
        for i in range(self.n):
            if self.memory[i] == mID:
                self.memory[i] = 0
                count += 1
        return count
```

---

## ARRAY MANIPULATION PROBLEMS

---

### Problem 65: 2021. Brightest Position on Street - Medium

**Description:** Find position with maximum brightness.

**Constraints:**
- 1 <= lights.length <= 10^5
- lights[i].length == 2
- -10^8 <= positioni <= 10^8
- 0 <= rangei <= 10^8

**Examples:**
```
Input: lights = [[-3,2],[1,2],[3,3]]
Output: -1

Input: lights = [[1,0],[0,1]]
Output: 0
```

**Pattern:** Line Sweep

**Time Complexity:** O(n log n)
**Space Complexity:** O(n)

**Solution:**
```python
def brightestPosition(lights: List[List[int]]) -> int:
    events = []
    
    for i, radius in lights:
        events.append((i - radius, 1))
        events.append((i + radius + 1, -1))
    
    events.sort()
    
    max_brightness = 0
    current_brightness = 0
    result = 0
    
    for pos, change in events:
        current_brightness += change
        if current_brightness > max_brightness:
            max_brightness = current_brightness
            result = pos
    
    return result
```

---

### Problem 66: 581. Shortest Unsorted Continuous Subarray - Medium

**Description:** Find shortest subarray to sort.

**Constraints:**
- 1 <= nums.length <= 10^4
- -10^5 <= nums[i] <= 10^5

**Examples:**
```
Input: nums = [2,6,4,8,10,9,15]
Output: 5

Input: nums = [1,2,3,4]
Output: 0

Input: nums = [1]
Output: 0
```

**Pattern:** Sorting / Two Pointers

**Time Complexity:** O(n log n)
**Space Complexity:** O(n)

**Solution:**
```python
def findUnsortedSubarray(nums: List[int]) -> int:
    sorted_nums = sorted(nums)
    start, end = len(nums), 0
    
    for i in range(len(nums)):
        if nums[i] != sorted_nums[i]:
            start = min(start, i)
            end = max(end, i)
    
    return end - start + 1 if end >= start else 0
```

---

### Problem 67: 959. Regions Cut By Slashes - Medium

**Description:** Count regions in grid with slashes.

**Constraints:**
- n == grid.length == grid[i].length
- 1 <= n <= 30
- grid[i][j] is either '/', '\', or ' '

**Examples:**
```
Input: grid = [" /","/ "]
Output: 2

Input: grid = [" /","  "]
Output: 1

Input: grid = ["/\\","\\/"]
Output: 5
```

**Pattern:** Union Find

**Time Complexity:** O(n²)
**Space Complexity:** O(n²)

**Solution:**
```python
def regionsBySlashes(grid: List[str]) -> int:
    n = len(grid)
    parent = list(range(4 * n * n))
    
    def find(x):
        if parent[x] != x:
            parent[x] = find(parent[x])
        return parent[x]
    
    def union(x, y):
        parent[find(x)] = find(y)
    
    for r in range(n):
        for c in range(n):
            base = 4 * (r * n + c)
            char = grid[r][c]
            
            if char == '/':
                union(base + 0, base + 3)
                union(base + 1, base + 2)
            elif char == '\\':
                union(base + 0, base + 1)
                union(base + 2, base + 3)
            else:
                union(base + 0, base + 1)
                union(base + 1, base + 2)
                union(base + 2, base + 3)
            
            if r > 0:
                union(base + 0, 4 * ((r - 1) * n + c) + 2)
            if c > 0:
                union(base + 3, 4 * (r * n + c - 1) + 1)
    
    return sum(find(x) == x for x in range(4 * n * n))
```

---

### Problem 68: 3071. Minimum Operations to Write Y - Medium

**Description:** Count operations for grid pattern.

**Constraints:**
- n == grid.length == grid[i].length
- 3 <= n <= 49
- n is odd
- 0 <= grid[i][j] <= 2

**Examples:**
```
Input: grid = [[1,2,2],[1,1,0],[0,1,0]]
Output: 3

Input: grid = [[0,1,0,1,0],[2,1,0,1,2],[2,2,2,0,1],[2,2,2,2,2],[2,1,2,2,2]]
Output: 12
```

**Pattern:** Array Manipulation + Counting

**Time Complexity:** O(n²)
**Space Complexity:** O(1)

**Solution:**
```python
def minimumOperationsToWriteY(grid: List[List[int]]) -> int:
    n = len(grid)
    y_cells = set()
    
    mid = n // 2
    for i in range(mid + 1):
        y_cells.add((i, i))
        y_cells.add((i, n - 1 - i))
    
    for i in range(mid, n):
        y_cells.add((i, mid))
    
    from collections import Counter
    y_counter = Counter()
    non_y_counter = Counter()
    
    for i in range(n):
        for j in range(n):
            if (i, j) in y_cells:
                y_counter[grid[i][j]] += 1
            else:
                non_y_counter[grid[i][j]] += 1
    
    min_ops = float('inf')
    for y_val in range(3):
        for non_y_val in range(3):
            if y_val != non_y_val:
                ops = (len(y_cells) - y_counter[y_val]) + \
                      (n * n - len(y_cells) - non_y_counter[non_y_val])
                min_ops = min(min_ops, ops)
    
    return min_ops
```

---

## ADDITIONAL PROBLEM

---

### Problem 69: Maximum Square Area in Buildings - Medium

**Description:** Given an array of building heights, return the maximum area of square that can fit in the buildings.

**Constraints:**
- 1 <= heights.length <= 10^5
- 0 <= heights[i] <= 10^4

**Examples:**
```
Input: heights = [2,1,5,6,2,3]
Output: 4
Explanation: Square of side 2 fits between indices 3-4

Input: heights = [2,4]
Output: 4
Explanation: Square of side 2 can fit

Input: heights = [1,1,1,1]
Output: 4
Explanation: Square of side 2 (area 4) fits in 4 consecutive buildings of height 1
```

**Pattern:** Stack / Monotonic Stack

**Time Complexity:** O(n)
**Space Complexity:** O(n)

**Solution:**
```python
def maxSquareArea(heights: List[int]) -> int:
    """
    Find maximum square area that can fit in buildings.
    Similar to Largest Rectangle in Histogram.
    """
    stack = []
    max_area = 0
    
    for i, h in enumerate(heights):
        start = i
        while stack and stack[-1][1] > h:
            index, height = stack.pop()
            width = i - index
            # For square: side = min(height, width)
            side = min(height, width)
            max_area = max(max_area, side * side)
            start = index
        
        stack.append((start, h))
    
    # Process remaining buildings
    for i, h in stack:
        width = len(heights) - i
        side = min(h, width)
        max_area = max(max_area, side * side)
    
    return max_area
```

---

## Summary

**Total Problems:** 69

**Pattern Distribution:**
- Sliding Window: 5
- Two Pointers: 6
- DFS: 9
- Backtracking: 3
- Dynamic Programming: 11
- Stack: 6
- Heap: 4
- Binary Search: 3
- Hash Map/Set: 4
- Graph: 4
- String: 6
- Sorting/Greedy: 1
- Tree Traversal: 3
- Design: 2
- Array: 4
- Custom: 1

Good luck with your TikTok OA on Friday! 🚀
