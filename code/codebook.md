# Complete LeetCode Problems Guide

## Sliding Window (5 problems)

### 1. Problem #3: Longest Substring Without Repeating Characters
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/longest-substring-without-repeating-characters/

**Problem Description:**
Given a string `s`, find the length of the longest substring without repeating characters. A substring is a contiguous sequence of characters within a string.

**Examples:**
- Input: `s = "abcabcbb"` → Output: `3` (The answer is "abc")
- Input: `s = "bbbbb"` → Output: `1` (The answer is "b")
- Input: `s = "pwwkew"` → Output: `3` (The answer is "wke")

**Simple Explanation:**
Imagine you have a sliding window that moves across the string. Start with two pointers (left and right) both at the beginning. Move the right pointer forward, adding each character to a set. If you find a character that's already in your set (meaning it repeats), move the left pointer forward until that duplicate is removed. Keep track of the maximum window size you've seen.

**Algorithm Steps:**
1. Create an empty set to store characters and two pointers (left = 0, right = 0)
2. Create a variable to track the maximum length found
3. For each character at the right pointer:
   - If the character is already in the set, remove characters from the left until it's gone
   - Add the current character to the set
   - Update the maximum length if current window is larger
   - Move right pointer forward
4. Return the maximum length

**Python Solution:**
```python
def lengthOfLongestSubstring(s: str) -> int:
    char_set = set()
    left = 0
    max_length = 0
    
    for right in range(len(s)):
        # Remove characters from left until no duplicate
        while s[right] in char_set:
            char_set.remove(s[left])
            left += 1
        
        # Add current character
        char_set.add(s[right])
        
        # Update maximum length
        max_length = max(max_length, right - left + 1)
    
    return max_length
```

**Time Complexity:** O(n) where n is the length of the string  
**Space Complexity:** O(min(n, m)) where m is the character set size

---

### 2. Problem #76: Minimum Window Substring
**Difficulty:** Hard  
**URL:** https://leetcode.com/problems/minimum-window-substring/

**Problem Description:**
Given two strings `s` and `t`, return the minimum window substring of `s` such that every character in `t` (including duplicates) is included in the window. If there is no such substring, return an empty string.

**Examples:**
- Input: `s = "ADOBECODEBANC"`, `t = "ABC"` → Output: `"BANC"`
- Input: `s = "a"`, `t = "a"` → Output: `"a"`
- Input: `s = "a"`, `t = "aa"` → Output: `""` (not enough 'a's)

**Simple Explanation:**
You need to find the smallest window in string `s` that contains all characters from string `t`. First, count how many of each character you need from `t`. Then use two pointers to create a window. Expand the window by moving right until you have all required characters. Then try to shrink from the left to make the window smaller while still containing all characters.

**Algorithm Steps:**
1. Count all characters in string `t` using a dictionary
2. Use two pointers (left = 0, right = 0) and a counter for matched characters
3. Expand right pointer and decrease the required count for each character
4. When all characters are matched:
   - Try to shrink from left to find the minimum window
   - Save this window if it's the smallest so far
5. Return the smallest window found

**Python Solution:**
```python
def minWindow(s: str, t: str) -> str:
    if not s or not t:
        return ""
    
    # Count characters needed from t
    t_count = {}
    for char in t:
        t_count[char] = t_count.get(char, 0) + 1
    
    required = len(t_count)  # Number of unique characters needed
    formed = 0  # Number of unique characters formed with desired frequency
    
    window_counts = {}
    left = 0
    min_len = float('inf')
    min_window = ""
    
    for right in range(len(s)):
        char = s[right]
        window_counts[char] = window_counts.get(char, 0) + 1
        
        # Check if current character satisfies the requirement
        if char in t_count and window_counts[char] == t_count[char]:
            formed += 1
        
        # Try to shrink window while it's valid
        while formed == required and left <= right:
            # Update minimum window
            if right - left + 1 < min_len:
                min_len = right - left + 1
                min_window = s[left:right + 1]
            
            # Shrink from left
            char = s[left]
            window_counts[char] -= 1
            if char in t_count and window_counts[char] < t_count[char]:
                formed -= 1
            left += 1
    
    return min_window
```

**Time Complexity:** O(|S| + |T|) where S and T are lengths of strings  
**Space Complexity:** O(|S| + |T|)

---

### 3. Problem #560: Subarray Sum Equals K
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/subarray-sum-equals-k/

**Problem Description:**
Given an array of integers `nums` and an integer `k`, return the total number of subarrays whose sum equals to `k`. A subarray is a contiguous non-empty sequence of elements within an array.

**Examples:**
- Input: `nums = [1,1,1]`, `k = 2` → Output: `2`
- Input: `nums = [1,2,3]`, `k = 3` → Output: `2`

**Simple Explanation:**
Instead of checking every possible subarray (which would be slow), use a clever trick with prefix sums. As you go through the array, keep track of the running sum. At each position, check if there's a previous sum such that (current_sum - previous_sum = k). Use a dictionary to remember all the prefix sums you've seen.

**Algorithm Steps:**
1. Create a dictionary to store prefix sums and their frequencies
2. Initialize with {0: 1} (sum of 0 appears once before the array starts)
3. Keep a running sum as you iterate through the array
4. At each position:
   - Check if (running_sum - k) exists in the dictionary
   - If yes, add its frequency to the count (those are valid subarrays)
   - Add the current running sum to the dictionary
5. Return the total count

**Python Solution:**
```python
def subarraySum(nums: list[int], k: int) -> int:
    count = 0
    current_sum = 0
    sum_freq = {0: 1}  # prefix sum -> frequency
    
    for num in nums:
        current_sum += num
        
        # Check if there's a prefix sum that would give us k
        if current_sum - k in sum_freq:
            count += sum_freq[current_sum - k]
        
        # Add current sum to dictionary
        sum_freq[current_sum] = sum_freq.get(current_sum, 0) + 1
    
    return count
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n)

---

### 4. Problem #974: Subarray Sums Divisible by K
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/subarray-sums-divisible-by-k/

**Problem Description:**
Given an integer array `nums` and an integer `k`, return the number of non-empty subarrays that have a sum divisible by `k`.

**Examples:**
- Input: `nums = [4,5,0,-2,-3,1]`, `k = 5` → Output: `7`

**Simple Explanation:**
This is similar to the previous problem but uses remainders. When you divide the running sum by k, the remainder tells you something important. If two positions have the same remainder, the subarray between them is divisible by k. Keep track of remainders in a dictionary.

**Algorithm Steps:**
1. Create a dictionary to store remainders and their frequencies
2. Initialize with {0: 1} (remainder 0 appears once at the start)
3. Keep a running sum and calculate its remainder when divided by k
4. Important: Make negative remainders positive by adding k
5. At each position, add the frequency of this remainder to the count
6. Update the dictionary with the current remainder

**Python Solution:**
```python
def subarraysDivByK(nums: list[int], k: int) -> int:
    count = 0
    current_sum = 0
    remainder_freq = {0: 1}  # remainder -> frequency
    
    for num in nums:
        current_sum += num
        remainder = current_sum % k
        
        # Handle negative remainders
        if remainder < 0:
            remainder += k
        
        # Add count of subarrays ending here with this remainder
        if remainder in remainder_freq:
            count += remainder_freq[remainder]
        
        # Update frequency
        remainder_freq[remainder] = remainder_freq.get(remainder, 0) + 1
    
    return count
```

**Time Complexity:** O(n)  
**Space Complexity:** O(k)

---

### 5. Problem #2537: Count the Number of Good Subarrays
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/count-the-number-of-good-subarrays/

**Problem Description:**
Given an integer array `nums` and an integer `k`, return the number of good subarrays. A subarray is good if there are at least `k` pairs of equal elements in it. A pair (i, j) is valid if i < j and nums[i] == nums[j].

**Examples:**
- Input: `nums = [1,1,1,1,1]`, `k = 10` → Output: `1`
- Input: `nums = [3,1,4,3,2,2,4]`, `k = 2` → Output: `4`

**Simple Explanation:**
Use a sliding window with two pointers. Count pairs as you add elements to the window. When you have at least k pairs, count all valid subarrays that start from the current left pointer. Then shrink the window from the left and update the pair count.

**Algorithm Steps:**
1. Use two pointers (left and right) and a dictionary to count element frequencies
2. Keep a variable to count pairs in the current window
3. Expand right pointer and add elements:
   - When adding an element, it forms (previous_count) new pairs
4. When pairs >= k:
   - All subarrays from left to any position beyond right are valid
   - Add (n - right) to the count
   - Shrink from left and update pair count
5. Return total count

**Python Solution:**
```python
def countGood(nums: list[int], k: int) -> int:
    count = 0
    pairs = 0
    freq = {}
    left = 0
    n = len(nums)
    
    for right in range(n):
        # Add element to window
        num = nums[right]
        if num in freq:
            pairs += freq[num]  # Forms this many new pairs
        freq[num] = freq.get(num, 0) + 1
        
        # Shrink window while we have enough pairs
        while pairs >= k:
            # All subarrays from left to end are valid
            count += n - right
            
            # Remove leftmost element
            left_num = nums[left]
            freq[left_num] -= 1
            pairs -= freq[left_num]  # Remove this many pairs
            if freq[left_num] == 0:
                del freq[left_num]
            left += 1
    
    return count
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n)

---

## Two Pointers (6 problems)

### 6. Problem #11: Container With Most Water
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/container-with-most-water/

**Problem Description:**
You are given an integer array `height` of length `n`. There are `n` vertical lines drawn such that the two endpoints of the ith line are (i, 0) and (i, height[i]). Find two lines that together with the x-axis form a container that holds the most water.

**Examples:**
- Input: `height = [1,8,6,2,5,4,8,3,7]` → Output: `49`

**Simple Explanation:**
Imagine you have two pointers, one at the start and one at the end of the array. Calculate the water that can be held between them (width × shorter height). Then move the pointer pointing to the shorter line inward, because moving the taller one can't possibly give us more water. Keep track of the maximum water found.

**Algorithm Steps:**
1. Place left pointer at start (0) and right pointer at end (n-1)
2. Calculate water = (right - left) × min(height[left], height[right])
3. Update maximum if current water is larger
4. Move the pointer pointing to the shorter line inward
5. Repeat until pointers meet
6. Return maximum water found

**Python Solution:**
```python
def maxArea(height: list[int]) -> int:
    left = 0
    right = len(height) - 1
    max_water = 0
    
    while left < right:
        # Calculate current water
        width = right - left
        current_water = width * min(height[left], height[right])
        max_water = max(max_water, current_water)
        
        # Move pointer pointing to shorter line
        if height[left] < height[right]:
            left += 1
        else:
            right -= 1
    
    return max_water
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### 7. Problem #15: 3Sum
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/3sum/

**Problem Description:**
Given an integer array `nums`, return all the triplets [nums[i], nums[j], nums[k]] such that i != j, i != k, and j != k, and nums[i] + nums[j] + nums[k] == 0. The solution set must not contain duplicate triplets.

**Examples:**
- Input: `nums = [-1,0,1,2,-1,-4]` → Output: `[[-1,-1,2],[-1,0,1]]`

**Simple Explanation:**
First, sort the array. Then for each number, use two pointers to find two other numbers that sum with it to zero. Skip duplicates to avoid repeating triplets. It's like solving the "two sum" problem for each number in the array.

**Algorithm Steps:**
1. Sort the array
2. Loop through each number (this will be the first number of triplet)
3. Skip duplicate first numbers
4. For remaining array, use two pointers (left and right)
5. Calculate sum of three numbers
6. If sum is zero, add to results and skip duplicates
7. If sum is negative, move left pointer right
8. If sum is positive, move right pointer left
9. Return all unique triplets

**Python Solution:**
```python
def threeSum(nums: list[int]) -> list[list[int]]:
    nums.sort()
    result = []
    n = len(nums)
    
    for i in range(n - 2):
        # Skip duplicate first numbers
        if i > 0 and nums[i] == nums[i - 1]:
            continue
        
        left = i + 1
        right = n - 1
        
        while left < right:
            total = nums[i] + nums[left] + nums[right]
            
            if total == 0:
                result.append([nums[i], nums[left], nums[right]])
                
                # Skip duplicates for second number
                while left < right and nums[left] == nums[left + 1]:
                    left += 1
                # Skip duplicates for third number
                while left < right and nums[right] == nums[right - 1]:
                    right -= 1
                
                left += 1
                right -= 1
            elif total < 0:
                left += 1
            else:
                right -= 1
    
    return result
```

**Time Complexity:** O(n²)  
**Space Complexity:** O(1) excluding the output array

---

### 8. Problem #33: Search in Rotated Sorted Array
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/search-in-rotated-sorted-array/

**Problem Description:**
You are given an integer array `nums` sorted in ascending order (with distinct values), and rotated at some pivot. Given a target value, return its index, or -1 if not found. You must write an algorithm with O(log n) runtime complexity.

**Examples:**
- Input: `nums = [4,5,6,7,0,1,2]`, `target = 0` → Output: `4`
- Input: `nums = [4,5,6,7,0,1,2]`, `target = 3` → Output: `-1`

**Simple Explanation:**
Even though the array is rotated, one half of the array is always sorted. Use binary search, but first check which half is sorted. Then determine if the target is in the sorted half or the other half. This tells you which direction to continue searching.

**Algorithm Steps:**
1. Use binary search with left and right pointers
2. Find the middle element
3. Check if middle element is the target
4. Determine which half (left or right) is sorted
5. Check if target is in the sorted half
6. If yes, search that half; if no, search the other half
7. Repeat until found or pointers cross

**Python Solution:**
```python
def search(nums: list[int], target: int) -> int:
    left, right = 0, len(nums) - 1
    
    while left <= right:
        mid = (left + right) // 2
        
        if nums[mid] == target:
            return mid
        
        # Check which half is sorted
        if nums[left] <= nums[mid]:
            # Left half is sorted
            if nums[left] <= target < nums[mid]:
                right = mid - 1
            else:
                left = mid + 1
        else:
            # Right half is sorted
            if nums[mid] < target <= nums[right]:
                left = mid + 1
            else:
                right = mid - 1
    
    return -1
```

**Time Complexity:** O(log n)  
**Space Complexity:** O(1)

---

### 9. Problem #165: Compare Version Numbers
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/compare-version-numbers/

**Problem Description:**
Given two version numbers, `version1` and `version2`, compare them. Return -1 if version1 < version2, return 1 if version1 > version2, otherwise return 0. Version strings are composed of numeric strings separated by dots. Leading zeros should be ignored.

**Examples:**
- Input: `version1 = "1.01"`, `version2 = "1.001"` → Output: `0`
- Input: `version1 = "1.0"`, `version2 = "1.0.0"` → Output: `0`

**Simple Explanation:**
Split both versions by dots to get individual numbers. Compare them one by one from left to right. If one version has more parts, treat the missing parts as 0. Return the comparison result based on the first difference found.

**Algorithm Steps:**
1. Split both version strings by '.'
2. Use two pointers to compare each revision number
3. For each position, get the number from both versions (or 0 if missing)
4. Convert strings to integers to compare (handles leading zeros)
5. If numbers are different, return -1 or 1 accordingly
6. If all numbers are equal, return 0

**Python Solution:**
```python
def compareVersion(version1: str, version2: str) -> int:
    v1_parts = version1.split('.')
    v2_parts = version2.split('.')
    
    # Make both lists the same length
    max_len = max(len(v1_parts), len(v2_parts))
    
    for i in range(max_len):
        # Get revision number or 0 if doesn't exist
        num1 = int(v1_parts[i]) if i < len(v1_parts) else 0
        num2 = int(v2_parts[i]) if i < len(v2_parts) else 0
        
        if num1 < num2:
            return -1
        elif num1 > num2:
            return 1
    
    return 0
```

**Time Complexity:** O(max(n, m)) where n and m are number of revisions  
**Space Complexity:** O(n + m)

---

### 10. Problem #189: Rotate Array
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/rotate-array/

**Problem Description:**
Given an integer array `nums`, rotate the array to the right by `k` steps, where `k` is non-negative. Try to do it in-place with O(1) extra space.

**Examples:**
- Input: `nums = [1,2,3,4,5,6,7]`, `k = 3` → Output: `[5,6,7,1,2,3,4]`

**Simple Explanation:**
Use the reversal trick! First reverse the entire array. Then reverse the first k elements. Finally reverse the remaining elements. This effectively rotates the array. Remember to use k % len(nums) to handle cases where k is larger than array length.

**Algorithm Steps:**
1. Calculate k = k % len(nums) to handle large k values
2. Reverse the entire array
3. Reverse the first k elements
4. Reverse the remaining elements from k to end
5. Helper function: reverse a portion of array using two pointers

**Python Solution:**
```python
def rotate(nums: list[int], k: int) -> None:
    n = len(nums)
    k = k % n  # Handle k larger than n
    
    def reverse(start, end):
        while start < end:
            nums[start], nums[end] = nums[end], nums[start]
            start += 1
            end -= 1
    
    # Reverse entire array
    reverse(0, n - 1)
    # Reverse first k elements
    reverse(0, k - 1)
    # Reverse remaining elements
    reverse(k, n - 1)
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### 11. Problem #678: Valid Parenthesis String
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/valid-parenthesis-string/

**Problem Description:**
Given a string `s` containing only three types of characters: '(', ')' and '*', return true if `s` is valid. The '*' character can be treated as a single right parenthesis ')' or a single left parenthesis '(' or an empty string "".

**Examples:**
- Input: `s = "()"` → Output: `true`
- Input: `s = "(*)"` → Output: `true`
- Input: `s = "(*))"` → Output: `true`

**Simple Explanation:**
Keep track of the minimum and maximum possible number of open parentheses as you go through the string. For '(' increase both min and max. For ')' decrease both. For '*' you have options, so decrease min and increase max. If max ever goes negative, it's invalid. At the end, min should be 0 or less.

**Algorithm Steps:**
1. Keep track of min and max possible open parentheses
2. For each character:
   - '(': increase both min and max by 1
   - ')': decrease both min and max by 1
   - '*': decrease min by 1 (treating as ')'), increase max by 1 (treating as '(')
3. If max becomes negative, return false (too many closing)
4. Keep min at 0 or higher (can't have negative open brackets)
5. At end, return true if min is 0

**Python Solution:**
```python
def checkValidString(s: str) -> bool:
    min_open = 0  # Minimum possible open parentheses
    max_open = 0  # Maximum possible open parentheses
    
    for char in s:
        if char == '(':
            min_open += 1
            max_open += 1
        elif char == ')':
            min_open -= 1
            max_open -= 1
        else:  # char == '*'
            min_open -= 1  # Treat as ')'
            max_open += 1  # Treat as '('
        
        if max_open < 0:
            return False  # Too many closing parentheses
        
        min_open = max(0, min_open)  # Can't be negative
    
    return min_open == 0
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

## Depth-First Search (DFS) (9 problems)

### 12. Problem #200: Number of Islands
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/number-of-islands/

**Problem Description:**
Given an m x n 2D binary grid `grid` which represents a map of '1's (land) and '0's (water), return the number of islands. An island is surrounded by water and is formed by connecting adjacent lands horizontally or vertically.

**Examples:**
- Input: `grid = [["1","1","0"],["1","1","0"],["0","0","1"]]` → Output: `2`

**Simple Explanation:**
Go through each cell in the grid. When you find a '1' (land), start a DFS to explore all connected land cells and mark them as visited. Each time you start a new DFS, you've found a new island. Count how many times you start a DFS.

**Algorithm Steps:**
1. Loop through all cells in the grid
2. When you find a '1', increment island count
3. Start DFS from that cell to mark all connected land
4. DFS function:
   - Mark current cell as '0' (visited)
   - Recursively call DFS on all 4 adjacent cells if they are '1'
5. Return total island count

**Python Solution:**
```python
def numIslands(grid: list[list[str]]) -> int:
    if not grid:
        return 0
    
    rows, cols = len(grid), len(grid[0])
    islands = 0
    
    def dfs(r, c):
        # Base cases
        if (r < 0 or r >= rows or c < 0 or c >= cols or grid[r][c] == '0'):
            return
        
        # Mark as visited
        grid[r][c] = '0'
        
        # Explore all 4 directions
        dfs(r + 1, c)  # Down
        dfs(r - 1, c)  # Up
        dfs(r, c + 1)  # Right
        dfs(r, c - 1)  # Left
    
    # Check each cell
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == '1':
                islands += 1
                dfs(r, c)
    
    return islands
```

**Time Complexity:** O(m × n)  
**Space Complexity:** O(m × n) for recursion stack

---

### 13. Problem #79: Word Search
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/word-search/

**Problem Description:**
Given an m x n grid of characters `board` and a string `word`, return true if `word` exists in the grid. The word can be constructed from letters of sequentially adjacent cells, where adjacent cells are horizontally or vertically neighboring. The same letter cell may not be used more than once.

**Examples:**
- Input: `board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]]`, `word = "ABCCED"` → Output: `true`

**Simple Explanation:**
Try starting the search from each cell in the grid. Use DFS to explore paths that match the word. Mark cells as visited while exploring, and unmark them when backtracking (so they can be used in other paths). If you successfully match all characters, return true.

**Algorithm Steps:**
1. Loop through each cell as a potential starting point
2. For each cell, start DFS if it matches the first character
3. DFS function:
   - Check if current position matches current character of word
   - Mark cell as visited (use '#' or similar)
   - If this is the last character, return true
   - Recursively try all 4 adjacent cells for next character
   - Unmark cell (backtrack) before returning
4. Return true if any starting point succeeds

**Python Solution:**
```python
def exist(board: list[list[str]], word: str) -> bool:
    rows, cols = len(board), len(board[0])
    
    def dfs(r, c, index):
        # Found the word
        if index == len(word):
            return True
        
        # Check boundaries and character match
        if (r < 0 or r >= rows or c < 0 or c >= cols or 
            board[r][c] != word[index]):
            return False
        
        # Mark as visited
        temp = board[r][c]
        board[r][c] = '#'
        
        # Explore all 4 directions
        found = (dfs(r + 1, c, index + 1) or
                dfs(r - 1, c, index + 1) or
                dfs(r, c + 1, index + 1) or
                dfs(r, c - 1, index + 1))
        
        # Backtrack
        board[r][c] = temp
        
        return found
    
    # Try starting from each cell
    for r in range(rows):
        for c in range(cols):
            if dfs(r, c, 0):
                return True
    
    return False
```

**Time Complexity:** O(m × n × 4^L) where L is the length of the word  
**Space Complexity:** O(L) for recursion stack

---

### 14. Problem #694: Number of Distinct Islands (Premium)
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/number-of-distinct-islands/

**Problem Description:**
Given a non-empty 2D array `grid` of 0's and 1's, an island is a group of 1's connected horizontally or vertically. You may assume all four edges of the grid are surrounded by water. Count the number of distinct islands. An island is considered to be the same as another if they have the same shape.

**Examples:**
- Input: `grid = [[1,1,0],[0,1,1],[0,0,0],[1,1,1],[0,1,0]]` → Output: `2`

**Simple Explanation:**
Similar to counting islands, but now you need to identify the shape of each island. Record the path you take during DFS as a signature for each island's shape. Use a set to store unique shapes. Two islands with the same path signature have the same shape.

**Algorithm Steps:**
1. Use a set to store unique island shapes
2. For each cell that's a '1', start DFS
3. During DFS, record the direction taken at each step (e.g., 'D' for down, 'U' for up)
4. Add a marker for backtracking (e.g., 'B') to distinguish different shapes
5. Convert the path to a tuple and add to set
6. Return the size of the set

**Python Solution:**
```python
def numDistinctIslands(grid: list[list[int]]) -> int:
    if not grid:
        return 0
    
    rows, cols = len(grid), len(grid[0])
    unique_islands = set()
    
    def dfs(r, c, direction):
        if (r < 0 or r >= rows or c < 0 or c >= cols or grid[r][c] == 0):
            return
        
        grid[r][c] = 0  # Mark as visited
        path.append(direction)
        
        dfs(r + 1, c, 'D')  # Down
        dfs(r - 1, c, 'U')  # Up
        dfs(r, c + 1, 'R')  # Right
        dfs(r, c - 1, 'L')  # Left
        
        path.append('B')  # Backtrack marker
    
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == 1:
                path = []
                dfs(r, c, 'S')  # Start
                unique_islands.add(tuple(path))
    
    return len(unique_islands)
```

**Time Complexity:** O(m × n)  
**Space Complexity:** O(m × n)

---

### 15. Problem #99: Recover Binary Search Tree
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/recover-binary-search-tree/

**Problem Description:**
You are given the root of a binary search tree (BST) where exactly two nodes were swapped by mistake. Recover the tree without changing its structure.

**Examples:**
- Input: `root = [1,3,null,null,2]` → Output: `[3,1,null,null,2]`

**Simple Explanation:**
Do an in-order traversal of the BST. In a correct BST, in-order traversal gives you sorted values. Find the two nodes that are out of order. The first node is where a value is greater than the next value. The second node is the last value that's smaller than the previous value. Swap these two nodes' values.

**Algorithm Steps:**
1. Perform in-order traversal and track previous node
2. Find violations where current value < previous value
3. First violation: mark previous node as first swapped node
4. Any violation: mark current node as second swapped node
5. After traversal, swap the values of the two marked nodes

**Python Solution:**
```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

def recoverTree(root: TreeNode) -> None:
    first = second = prev = None
    
    def inorder(node):
        nonlocal first, second, prev
        
        if not node:
            return
        
        # Traverse left
        inorder(node.left)
        
        # Check for violations
        if prev and node.val < prev.val:
            if not first:
                first = prev  # First violation
            second = node  # Last violation
        
        prev = node
        
        # Traverse right
        inorder(node.right)
    
    inorder(root)
    
    # Swap the values
    if first and second:
        first.val, second.val = second.val, first.val
```

**Time Complexity:** O(n)  
**Space Complexity:** O(h) where h is the height of the tree

---

### 16. Problem #297: Serialize and Deserialize Binary Tree
**Difficulty:** Hard  
**URL:** https://leetcode.com/problems/serialize-and-deserialize-binary-tree/

**Problem Description:**
Design an algorithm to serialize and deserialize a binary tree. Serialization is converting a tree to a string, and deserialization is converting the string back to the original tree structure.

**Examples:**
- Input: `root = [1,2,3,null,null,4,5]` → Serialize to string → Deserialize back to tree

**Simple Explanation:**
For serialization, use pre-order traversal (root, left, right) and represent null nodes with a special marker like "null". Join all values with commas. For deserialization, split the string and use the same pre-order approach to rebuild the tree. Use a queue or iterator to track your position in the list.

**Algorithm Steps:**
Serialize:
1. Do pre-order traversal
2. Add node values to a list (use "null" for null nodes)
3. Join the list into a string with commas

Deserialize:
1. Split the string by commas to get a list
2. Create an iterator or index pointer
3. Build tree using pre-order: read value, create node, build left, build right
4. Return the root

**Python Solution:**
```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

class Codec:
    def serialize(self, root):
        def preorder(node):
            if not node:
                result.append("null")
                return
            result.append(str(node.val))
            preorder(node.left)
            preorder(node.right)
        
        result = []
        preorder(root)
        return ",".join(result)
    
    def deserialize(self, data):
        def build():
            val = next(values)
            if val == "null":
                return None
            node = TreeNode(int(val))
            node.left = build()
            node.right = build()
            return node
        
        values = iter(data.split(","))
        return build()
```

**Time Complexity:** O(n) for both operations  
**Space Complexity:** O(n)

---

### 17. Problem #399: Evaluate Division
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/evaluate-division/

**Problem Description:**
You are given equations in the format A / B = k, and you need to answer queries C / D = ?. Variables that don't appear in equations have no defined value. Return the answers array, and if answer cannot be determined, return -1.0.

**Examples:**
- Input: `equations = [["a","b"],["b","c"]]`, `values = [2.0,3.0]`, `queries = [["a","c"],["b","a"]]`
- Output: `[6.00000,0.50000]`

**Simple Explanation:**
Build a graph where each variable is a node. An edge from A to B with weight k means A/B = k. Also add reverse edge B to A with weight 1/k. For each query, use DFS to find a path from start to end variable, multiplying weights along the path. If no path exists, return -1.

**Algorithm Steps:**
1. Build a graph (dictionary of dictionaries) from equations
2. For each equation A/B = k:
   - Add edge A → B with weight k
   - Add edge B → A with weight 1/k
3. For each query:
   - Use DFS to find path from start to end
   - Multiply weights along the path
   - Return product or -1.0 if no path

**Python Solution:**
```python
def calcEquation(equations: list[list[str]], values: list[float], 
                queries: list[list[str]]) -> list[float]:
    # Build graph
    graph = {}
    for (a, b), value in zip(equations, values):
        if a not in graph:
            graph[a] = {}
        if b not in graph:
            graph[b] = {}
        graph[a][b] = value
        graph[b][a] = 1 / value
    
    def dfs(start, end, visited):
        if start not in graph or end not in graph:
            return -1.0
        if start == end:
            return 1.0
        
        visited.add(start)
        for neighbor, value in graph[start].items():
            if neighbor not in visited:
                result = dfs(neighbor, end, visited)
                if result != -1.0:
                    return value * result
        
        return -1.0
    
    results = []
    for start, end in queries:
        results.append(dfs(start, end, set()))
    
    return results
```

**Time Complexity:** O(Q × (V + E)) where Q is queries, V is variables, E is equations  
**Space Complexity:** O(V + E)

---

### 18. Problem #827: Making A Large Island
**Difficulty:** Hard  
**URL:** https://leetcode.com/problems/making-a-large-island/

**Problem Description:**
You are given an n x n binary matrix `grid`. You can change at most one 0 to be 1. Return the size of the largest island in grid after applying this operation. An island is a 4-directionally connected group of 1s.

**Examples:**
- Input: `grid = [[1,0],[0,1]]` → Output: `3`

**Simple Explanation:**
First, identify all existing islands and assign each a unique ID. Store each island's size. Then check each 0 cell - if you flip it to 1, what unique islands would it connect? Add 1 (for the cell itself) plus the sizes of all adjacent unique islands. Track the maximum size found.

**Algorithm Steps:**
1. Use DFS to identify all islands and give each a unique ID (starting from 2)
2. Store island sizes in a dictionary {island_id: size}
3. For each 0 in the grid:
   - Check all 4 neighbors
   - Collect unique island IDs from neighbors
   - Calculate size = 1 + sum of all unique neighbor island sizes
4. Return the maximum size (or original largest island if no 0s)

**Python Solution:**
```python
def largestIsland(grid: list[list[int]]) -> int:
    n = len(grid)
    island_id = 2  # Start from 2 (0 and 1 are used)
    island_sizes = {0: 0}  # island_id -> size
    
    def dfs(r, c, island_id):
        if r < 0 or r >= n or c < 0 or c >= n or grid[r][c] != 1:
            return 0
        grid[r][c] = island_id
        size = 1
        size += dfs(r + 1, c, island_id)
        size += dfs(r - 1, c, island_id)
        size += dfs(r, c + 1, island_id)
        size += dfs(r, c - 1, island_id)
        return size
    
    # Label all islands
    for r in range(n):
        for c in range(n):
            if grid[r][c] == 1:
                size = dfs(r, c, island_id)
                island_sizes[island_id] = size
                island_id += 1
    
    max_size = max(island_sizes.values()) if island_sizes else 0
    
    # Try flipping each 0
    directions = [(0, 1), (1, 0), (0, -1), (-1, 0)]
    for r in range(n):
        for c in range(n):
            if grid[r][c] == 0:
                seen_islands = set()
                for dr, dc in directions:
                    nr, nc = r + dr, c + dc
                    if 0 <= nr < n and 0 <= nc < n and grid[nr][nc] > 1:
                        seen_islands.add(grid[nr][nc])
                
                size = 1 + sum(island_sizes[iid] for iid in seen_islands)
                max_size = max(max_size, size)
    
    return max_size
```

**Time Complexity:** O(n²)  
**Space Complexity:** O(n²)

---

### 19. Problem #886: Possible Bipartition
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/possible-bipartition/

**Problem Description:**
Given a set of n people (numbered 1 to n), we need to split them into two groups of any size. Given an array `dislikes` where dislikes[i] = [a, b] means person a and person b should not be in the same group. Return true if it's possible to split everyone into two groups this way.

**Examples:**
- Input: `n = 4`, `dislikes = [[1,2],[1,3],[2,4]]` → Output: `true`

**Simple Explanation:**
This is a graph coloring problem. Build a graph where people who dislike each other are connected. Try to color the graph with 2 colors such that no adjacent nodes have the same color. Use DFS to color nodes - if you encounter a conflict (two dislikes with same color), return false.

**Algorithm Steps:**
1. Build adjacency list from dislikes
2. Create a color array (0 = uncolored, 1 = group 1, -1 = group 2)
3. For each uncolored person, start DFS with color 1
4. DFS function:
   - Color current person
   - Try to color all neighbors with opposite color
   - If neighbor already has same color, return false
5. Return true if all people can be colored

**Python Solution:**
```python
def possibleBipartition(n: int, dislikes: list[list[int]]) -> bool:
    # Build adjacency list
    graph = [[] for _ in range(n + 1)]
    for a, b in dislikes:
        graph[a].append(b)
        graph[b].append(a)
    
    color = [0] * (n + 1)  # 0 = uncolored, 1 and -1 are two groups
    
    def dfs(person, c):
        color[person] = c
        for neighbor in graph[person]:
            if color[neighbor] == c:
                return False  # Same group as person
            if color[neighbor] == 0 and not dfs(neighbor, -c):
                return False
        return True
    
    # Check each component
    for person in range(1, n + 1):
        if color[person] == 0:
            if not dfs(person, 1):
                return False
    
    return True
```

**Time Complexity:** O(V + E) where V is people and E is dislikes  
**Space Complexity:** O(V + E)

---

### 20. Problem #934: Shortest Bridge
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/shortest-bridge/

**Problem Description:**
You are given an n x n binary matrix `grid` where 1 represents land and 0 represents water. There are exactly two islands in the grid. Return the smallest number of 0s you must flip to connect the two islands.

**Examples:**
- Input: `grid = [[0,1],[1,0]]` → Output: `1`

**Simple Explanation:**
First, use DFS to find all cells of the first island and add them to a queue. Then use BFS starting from all cells of the first island simultaneously. Expand outward level by level (flipping 0s). When you reach a 1 from the second island, return the number of steps taken.

**Algorithm Steps:**
1. Find the first island using DFS, add all its cells to a queue
2. Mark all first island cells as visited (e.g., change to 2)
3. Use BFS from the queue:
   - For each cell, check all 4 neighbors
   - If neighbor is water (0), add to queue and mark as visited
   - If neighbor is land (1), you've reached the second island - return distance
   - Increment distance after each level
4. Return the distance

**Python Solution:**
```python
from collections import deque

def shortestBridge(grid: list[list[int]]) -> int:
    n = len(grid)
    directions = [(0, 1), (1, 0), (0, -1), (-1, 0)]
    
    def dfs(r, c):
        if r < 0 or r >= n or c < 0 or c >= n or grid[r][c] != 1:
            return
        grid[r][c] = 2  # Mark as visited
        queue.append((r, c))
        for dr, dc in directions:
            dfs(r + dr, c + dc)
    
    # Find first island and add to queue
    queue = deque()
    found = False
    for r in range(n):
        if found:
            break
        for c in range(n):
            if grid[r][c] == 1:
                dfs(r, c)
                found = True
                break
    
    # BFS to find shortest path to second island
    steps = 0
    while queue:
        for _ in range(len(queue)):
            r, c = queue.popleft()
            for dr, dc in directions:
                nr, nc = r + dr, c + dc
                if 0 <= nr < n and 0 <= nc < n:
                    if grid[nr][nc] == 1:
                        return steps
                    elif grid[nr][nc] == 0:
                        grid[nr][nc] = 2
                        queue.append((nr, nc))
        steps += 1
    
    return steps
```

**Time Complexity:** O(n²)  
**Space Complexity:** O(n²)

---

## Backtracking (3 problems)

### 21. Problem #22: Generate Parentheses
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/generate-parentheses/

**Problem Description:**
Given `n` pairs of parentheses, write a function to generate all combinations of well-formed parentheses.

**Examples:**
- Input: `n = 3` → Output: `["((()))","(()())","(())()","()(())","()()()"]`
- Input: `n = 1` → Output: `["()"]`

**Simple Explanation:**
Use backtracking to build valid combinations. At each step, you can add an opening parenthesis if you haven't used all n, or a closing parenthesis if it won't make the string invalid (must have more opening than closing at any point). Build the string character by character until you have 2*n characters.

**Algorithm Steps:**
1. Create a result list and start backtracking with an empty string
2. Backtracking function takes: current string, count of open parens, count of close parens
3. If string length is 2*n, add to results
4. If open count < n, try adding '(' and recurse
5. If close count < open count, try adding ')' and recurse
6. Return all valid combinations

**Python Solution:**
```python
def generateParenthesis(n: int) -> list[str]:
    result = []
    
    def backtrack(current, open_count, close_count):
        # Base case: we have a valid combination
        if len(current) == 2 * n:
            result.append(current)
            return
        
        # Add opening parenthesis if we can
        if open_count < n:
            backtrack(current + '(', open_count + 1, close_count)
        
        # Add closing parenthesis if it's valid
        if close_count < open_count:
            backtrack(current + ')', open_count, close_count + 1)
    
    backtrack('', 0, 0)
    return result
```

**Time Complexity:** O(4^n / √n) - Catalan number  
**Space Complexity:** O(n) for recursion depth

---

### 22. Problem #47: Permutations II
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/permutations-ii/

**Problem Description:**
Given a collection of numbers `nums` that might contain duplicates, return all possible unique permutations in any order.

**Examples:**
- Input: `nums = [1,1,2]` → Output: `[[1,1,2],[1,2,1],[2,1,1]]`
- Input: `nums = [1,2,3]` → Output: `[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]`

**Simple Explanation:**
Sort the array first so duplicates are together. Use backtracking to build permutations. Use a boolean array to track which numbers are used. Skip duplicates by checking: if current number equals previous AND previous is not used, skip it (this avoids creating duplicate permutations).

**Algorithm Steps:**
1. Sort the input array
2. Create a used boolean array
3. Backtracking function:
   - If current permutation length equals input length, add to results
   - For each index:
     - Skip if already used
     - Skip if equals previous number AND previous is not used (duplicate handling)
     - Mark as used, add to current permutation
     - Recurse
     - Backtrack: remove from permutation, mark as unused
4. Return all unique permutations

**Python Solution:**
```python
def permuteUnique(nums: list[int]) -> list[list[int]]:
    nums.sort()  # Sort to group duplicates
    result = []
    used = [False] * len(nums)
    
    def backtrack(current):
        if len(current) == len(nums):
            result.append(current[:])
            return
        
        for i in range(len(nums)):
            # Skip if already used
            if used[i]:
                continue
            
            # Skip duplicates: if current equals previous and previous not used
            if i > 0 and nums[i] == nums[i - 1] and not used[i - 1]:
                continue
            
            # Choose
            used[i] = True
            current.append(nums[i])
            
            # Explore
            backtrack(current)
            
            # Unchoose
            current.pop()
            used[i] = False
    
    backtrack([])
    return result
```

**Time Complexity:** O(n! × n)  
**Space Complexity:** O(n)

---

### 23. Problem #212: Word Search II
**Difficulty:** Hard  
**URL:** https://leetcode.com/problems/word-search-ii/

**Problem Description:**
Given an m x n board of characters and a list of strings `words`, return all words on the board. Each word must be constructed from letters of sequentially adjacent cells, where adjacent cells are horizontally or vertically neighboring. The same letter cell may not be used more than once in a word.

**Examples:**
- Input: `board = [["o","a","a","n"],["e","t","a","e"],["i","h","k","r"],["i","f","l","v"]]`, `words = ["oath","pea","eat","rain"]`
- Output: `["eat","oath"]`

**Simple Explanation:**
Build a Trie (prefix tree) from all words to search efficiently. For each cell in the board, start DFS. As you explore, follow the Trie. If you reach a word ending in the Trie, add it to results. Mark cells as visited during DFS and unmark when backtracking. This is much faster than searching for each word separately.

**Algorithm Steps:**
1. Build a Trie from all words in the list
2. For each cell in the board:
   - Start DFS if the cell's letter is in the Trie root
3. DFS function:
   - If current Trie node marks end of word, add word to results
   - Mark cell as visited (save original char, replace with '#')
   - Try all 4 directions
   - Only continue DFS if next letter is in current Trie node's children
   - Restore cell value (backtrack)
4. Return all found words

**Python Solution:**
```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.word = None

def findWords(board: list[list[str]], words: list[str]) -> list[str]:
    # Build Trie
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
        
        # Found a word
        if next_node.word:
            result.append(next_node.word)
            next_node.word = None  # Avoid duplicates
        
        # Mark as visited
        board[r][c] = '#'
        
        # Explore all 4 directions
        for dr, dc in [(0, 1), (1, 0), (0, -1), (-1, 0)]:
            nr, nc = r + dr, c + dc
            if 0 <= nr < rows and 0 <= nc < cols and board[nr][nc] != '#':
                dfs(nr, nc, next_node)
        
        # Restore
        board[r][c] = char
    
    # Start DFS from each cell
    for r in range(rows):
        for c in range(cols):
            if board[r][c] in root.children:
                dfs(r, c, root)
    
    return result
```

**Time Complexity:** O(m × n × 4^L) where L is max word length  
**Space Complexity:** O(N) where N is total characters in all words

---

## Dynamic Programming (DP) (11 problems)

### 24. Problem #322: Coin Change
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/coin-change/

**Problem Description:**
You are given an integer array `coins` representing coins of different denominations and an integer `amount` representing a total amount of money. Return the fewest number of coins needed to make up that amount. If that amount cannot be made up, return -1.

**Examples:**
- Input: `coins = [1,2,5]`, `amount = 11` → Output: `3` (11 = 5 + 5 + 1)
- Input: `coins = [2]`, `amount = 3` → Output: `-1`

**Simple Explanation:**
Create an array where dp[i] represents the minimum coins needed to make amount i. For each amount from 1 to target, try using each coin and take the minimum. Think of it as: "For amount X, what's the minimum between: using coin C and adding 1 to dp[X-C]?"

**Algorithm Steps:**
1. Create dp array of size (amount + 1), initialize all to infinity except dp[0] = 0
2. For each amount from 1 to target amount:
   - For each coin:
     - If coin <= amount:
       - dp[amount] = min(dp[amount], dp[amount - coin] + 1)
3. Return dp[target] if it's not infinity, otherwise -1

**Python Solution:**
```python
def coinChange(coins: list[int], amount: int) -> int:
    # dp[i] = minimum coins needed for amount i
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0  # 0 coins needed for amount 0
    
    for i in range(1, amount + 1):
        for coin in coins:
            if coin <= i:
                dp[i] = min(dp[i], dp[i - coin] + 1)
    
    return dp[amount] if dp[amount] != float('inf') else -1
```

**Time Complexity:** O(amount × number of coins)  
**Space Complexity:** O(amount)

---

### 25. Problem #72: Edit Distance
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/edit-distance/

**Problem Description:**
Given two strings `word1` and `word2`, return the minimum number of operations required to convert word1 to word2. You can insert, delete, or replace a character.

**Examples:**
- Input: `word1 = "horse"`, `word2 = "ros"` → Output: `3` (horse -> rorse -> rose -> ros)
- Input: `word1 = "intention"`, `word2 = "execution"` → Output: `5`

**Simple Explanation:**
Create a 2D table where dp[i][j] represents the edit distance between the first i characters of word1 and first j characters of word2. If characters match, copy the diagonal value. If they don't match, take the minimum of three operations (insert, delete, replace) and add 1.

**Algorithm Steps:**
1. Create a 2D array dp of size (len1+1) × (len2+1)
2. Initialize first row and column (converting from/to empty string)
3. For each cell dp[i][j]:
   - If characters match: dp[i][j] = dp[i-1][j-1]
   - Else: dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])
     - dp[i-1][j] + 1: delete from word1
     - dp[i][j-1] + 1: insert into word1
     - dp[i-1][j-1] + 1: replace
4. Return dp[len1][len2]

**Python Solution:**
```python
def minDistance(word1: str, word2: str) -> int:
    m, n = len(word1), len(word2)
    
    # Create DP table
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    # Initialize base cases
    for i in range(m + 1):
        dp[i][0] = i  # Delete all characters
    for j in range(n + 1):
        dp[0][j] = j  # Insert all characters
    
    # Fill the table
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if word1[i - 1] == word2[j - 1]:
                dp[i][j] = dp[i - 1][j - 1]  # No operation needed
            else:
                dp[i][j] = 1 + min(
                    dp[i - 1][j],      # Delete
                    dp[i][j - 1],      # Insert
                    dp[i - 1][j - 1]   # Replace
                )
    
    return dp[m][n]
```

**Time Complexity:** O(m × n)  
**Space Complexity:** O(m × n)

---

### 26. Problem #300: Longest Increasing Subsequence
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/longest-increasing-subsequence/

**Problem Description:**
Given an integer array `nums`, return the length of the longest strictly increasing subsequence. A subsequence is an array that can be derived from another array by deleting some or no elements without changing the order of the remaining elements.

**Examples:**
- Input: `nums = [10,9,2,5,3,7,101,18]` → Output: `4` (The LIS is [2,3,7,101] or [2,3,7,18])
- Input: `nums = [0,1,0,3,2,3]` → Output: `4`

**Simple Explanation:**
For each number, find the longest increasing subsequence ending at that number. To do this, look at all previous numbers that are smaller, and take the best one. dp[i] = 1 + max(dp[j] for all j < i where nums[j] < nums[i]).

**Algorithm Steps:**
1. Create dp array where dp[i] = length of LIS ending at index i
2. Initialize all values to 1 (each number is a subsequence of length 1)
3. For each position i:
   - Look at all previous positions j (j < i)
   - If nums[j] < nums[i], we can extend that subsequence
   - dp[i] = max(dp[i], dp[j] + 1)
4. Return the maximum value in dp array

**Python Solution:**
```python
def lengthOfLIS(nums: list[int]) -> int:
    if not nums:
        return 0
    
    n = len(nums)
    dp = [1] * n  # Each number is a subsequence of length 1
    
    for i in range(1, n):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    
    return max(dp)
```

**Time Complexity:** O(n²)  
**Space Complexity:** O(n)

---

### 27. Problem #887: Super Egg Drop
**Difficulty:** Hard  
**URL:** https://leetcode.com/problems/super-egg-drop/

**Problem Description:**
You are given `k` identical eggs and a building with `n` floors. You want to know the minimum number of moves needed to determine with certainty what the highest floor is from which an egg can be dropped without breaking. An egg that survives a fall can be used again.

**Examples:**
- Input: `k = 1`, `n = 2` → Output: `2`
- Input: `k = 2`, `n = 6` → Output: `3`

**Simple Explanation:**
This is a tricky problem. Instead of asking "how many moves for k eggs and n floors?", flip it: "with k eggs and m moves, how many floors can we check?". Use DP: dp[m][k] = dp[m-1][k-1] + dp[m-1][k] + 1. This represents: with one move, either egg breaks (check dp[m-1][k-1] floors below) or doesn't break (check dp[m-1][k] floors above), plus current floor.

**Algorithm Steps:**
1. Think inversely: dp[moves][eggs] = max floors we can check
2. Initialize: with 0 moves or 0 eggs, we can check 0 floors
3. For each number of moves m and eggs k:
   - dp[m][k] = dp[m-1][k-1] + dp[m-1][k] + 1
   - Egg breaks: can check dp[m-1][k-1] floors below
   - Egg doesn't break: can check dp[m-1][k] floors above
   - Plus 1 for current floor
4. Find minimum m where dp[m][k] >= n

**Python Solution:**
```python
def superEggDrop(k: int, n: int) -> int:
    # dp[m][k] = max floors we can check with m moves and k eggs
    dp = [[0] * (k + 1) for _ in range(n + 1)]
    
    for m in range(1, n + 1):
        for eggs in range(1, k + 1):
            # One move: either breaks (check below) or doesn't (check above)
            dp[m][eggs] = dp[m - 1][eggs - 1] + dp[m - 1][eggs] + 1
        
        # If we can check n floors with m moves, return m
        if dp[m][k] >= n:
            return m
    
    return n
```

**Time Complexity:** O(k × n)  
**Space Complexity:** O(k × n)

---

### 28. Problem #32: Longest Valid Parentheses
**Difficulty:** Hard  
**URL:** https://leetcode.com/problems/longest-valid-parentheses/

**Problem Description:**
Given a string containing just the characters '(' and ')', return the length of the longest valid (well-formed) parentheses substring.

**Examples:**
- Input: `s = "(()"` → Output: `2` (The substring "()" is the longest valid)
- Input: `s = ")()())"` → Output: `4` (The substring "()()" is the longest valid)

**Simple Explanation:**
Use dynamic programming where dp[i] represents the length of the longest valid parentheses ending at index i. When you see ')', check if it can form a valid pair. If previous character is '(', add 2 plus any valid substring before it. If previous character is ')', check if there's a matching '(' before that valid substring.

**Algorithm Steps:**
1. Create dp array of size n, initialized to 0
2. Iterate through string starting from index 1
3. When character is ')':
   - If previous is '(': dp[i] = dp[i-2] + 2 (direct pair)
   - If previous is ')' and there's a matching '(':
     - Find the matching '(' position: j = i - dp[i-1] - 1
     - If s[j] == '(': dp[i] = dp[i-1] + 2 + dp[j-1]
4. Return max value in dp

**Python Solution:**
```python
def longestValidParentheses(s: str) -> int:
    if not s:
        return 0
    
    n = len(s)
    dp = [0] * n
    max_len = 0
    
    for i in range(1, n):
        if s[i] == ')':
            if s[i - 1] == '(':
                # Case: ...()
                dp[i] = (dp[i - 2] if i >= 2 else 0) + 2
            elif i - dp[i - 1] > 0 and s[i - dp[i - 1] - 1] == '(':
                # Case: ...))
                dp[i] = dp[i - 1] + 2 + (dp[i - dp[i - 1] - 2] if i - dp[i - 1] >= 2 else 0)
            
            max_len = max(max_len, dp[i])
    
    return max_len
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n)

---

### 29. Problem #5: Longest Palindromic Substring
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/longest-palindromic-substring/

**Problem Description:**
Given a string `s`, return the longest palindromic substring in `s`. A palindrome is a string that reads the same forward and backward.

**Examples:**
- Input: `s = "babad"` → Output: `"bab"` (Note: "aba" is also valid)
- Input: `s = "cbbd"` → Output: `"bb"`

**Simple Explanation:**
Expand around each possible center. A palindrome can have either one center (odd length like "aba") or two centers (even length like "abba"). For each position, try expanding outward as long as characters match. Keep track of the longest palindrome found.

**Algorithm Steps:**
1. For each position i in the string:
   - Expand around single center (odd length palindromes)
   - Expand around double center i and i+1 (even length palindromes)
2. Helper function to expand:
   - While left and right characters match and within bounds
   - Expand outward (left--, right++)
   - Return the palindrome length
3. Keep track of the longest palindrome's start and length
4. Return substring using start and length

**Python Solution:**
```python
def longestPalindrome(s: str) -> str:
    if not s:
        return ""
    
    start = 0
    max_len = 0
    
    def expand_around_center(left, right):
        while left >= 0 and right < len(s) and s[left] == s[right]:
            left -= 1
            right += 1
        return right - left - 1  # Length of palindrome
    
    for i in range(len(s)):
        # Odd length palindromes (single center)
        len1 = expand_around_center(i, i)
        # Even length palindromes (double center)
        len2 = expand_around_center(i, i + 1)
        
        current_len = max(len1, len2)
        if current_len > max_len:
            max_len = current_len
            start = i - (current_len - 1) // 2
    
    return s[start:start + max_len]
```

**Time Complexity:** O(n²)  
**Space Complexity:** O(1)

---

### 30. Problem #70: Climbing Stairs
**Difficulty:** Easy  
**URL:** https://leetcode.com/problems/climbing-stairs/

**Problem Description:**
You are climbing a staircase with `n` steps. Each time you can either climb 1 or 2 steps. In how many distinct ways can you climb to the top?

**Examples:**
- Input: `n = 2` → Output: `2` (1+1 or 2)
- Input: `n = 3` → Output: `3` (1+1+1, 1+2, or 2+1)

**Simple Explanation:**
This is a Fibonacci problem in disguise! To reach step n, you can come from step n-1 (taking 1 step) or from step n-2 (taking 2 steps). So ways[n] = ways[n-1] + ways[n-2]. Base cases: 1 way to reach step 1, 2 ways to reach step 2.

**Algorithm Steps:**
1. Handle base cases: n=1 returns 1, n=2 returns 2
2. Create dp array or use two variables
3. For each step from 3 to n:
   - dp[i] = dp[i-1] + dp[i-2]
4. Return dp[n]

**Python Solution:**
```python
def climbStairs(n: int) -> int:
    if n <= 2:
        return n
    
    # Only need to track last two values
    prev2 = 1  # Ways to reach step 1
    prev1 = 2  # Ways to reach step 2
    
    for i in range(3, n + 1):
        current = prev1 + prev2
        prev2 = prev1
        prev1 = current
    
    return prev1
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### 31. Problem #123: Best Time to Buy and Sell Stock III
**Difficulty:** Hard  
**URL:** https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/

**Problem Description:**
You are given an array `prices` where prices[i] is the price of a given stock on the ith day. Find the maximum profit you can achieve. You may complete at most two transactions. Note: You may not engage in multiple transactions simultaneously (i.e., you must sell before you buy again).

**Examples:**
- Input: `prices = [3,3,5,0,0,3,1,4]` → Output: `6` (Buy on day 4, sell on day 6 = 3. Buy on day 7, sell on day 8 = 3. Total = 6)
- Input: `prices = [1,2,3,4,5]` → Output: `4` (Buy on day 1, sell on day 5)

**Simple Explanation:**
Track four states: after first buy, after first sell, after second buy, after second sell. For each day, update these states. First buy: minimum price so far. First sell: maximum profit from one transaction. Second buy: best we can do after buying again. Second sell: best we can do after selling again.

**Algorithm Steps:**
1. Initialize four variables:
   - buy1 = -prices[0] (cost of first buy)
   - sell1 = 0 (profit after first sell)
   - buy2 = -prices[0] (cost of second buy)
   - sell2 = 0 (profit after second sell)
2. For each price:
   - buy1 = max(buy1, -price) (minimize first buy cost)
   - sell1 = max(sell1, buy1 + price) (maximize after first sell)
   - buy2 = max(buy2, sell1 - price) (best position after second buy)
   - sell2 = max(sell2, buy2 + price) (maximize after second sell)
3. Return sell2

**Python Solution:**
```python
def maxProfit(prices: list[int]) -> int:
    if not prices:
        return 0
    
    buy1 = buy2 = float('-inf')
    sell1 = sell2 = 0
    
    for price in prices:
        buy1 = max(buy1, -price)
        sell1 = max(sell1, buy1 + price)
        buy2 = max(buy2, sell1 - price)
        sell2 = max(sell2, buy2 + price)
    
    return sell2
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### 32. Problem #124: Binary Tree Maximum Path Sum
**Difficulty:** Hard  
**URL:** https://leetcode.com/problems/binary-tree-maximum-path-sum/

**Problem Description:**
A path in a binary tree is a sequence of nodes where each pair of adjacent nodes has an edge connecting them. A node can only appear once in a path. Given the root of a binary tree, return the maximum path sum of any non-empty path.

**Examples:**
- Input: `root = [1,2,3]` → Output: `6` (The path is 2 -> 1 -> 3)
- Input: `root = [-10,9,20,null,null,15,7]` → Output: `42` (The path is 15 -> 20 -> 7)

**Simple Explanation:**
Use recursion. For each node, calculate the maximum path sum that passes through it. This includes the node value plus the maximum gain from left and right children (but only if they're positive). Update the global maximum. Return to parent only the maximum single-branch path (node + max of left or right).

**Algorithm Steps:**
1. Keep a global variable for maximum path sum
2. Recursive function for each node:
   - Return 0 if node is null
   - Recursively get max gain from left and right children
   - Ignore negative gains (max with 0)
   - Calculate path through this node: node.val + left_gain + right_gain
   - Update global maximum
   - Return to parent: node.val + max(left_gain, right_gain)
3. Return global maximum

**Python Solution:**
```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

def maxPathSum(root: TreeNode) -> int:
    max_sum = float('-inf')
    
    def max_gain(node):
        nonlocal max_sum
        
        if not node:
            return 0
        
        # Max gain from left and right (ignore if negative)
        left_gain = max(max_gain(node.left), 0)
        right_gain = max(max_gain(node.right), 0)
        
        # Path through current node
        current_path = node.val + left_gain + right_gain
        
        # Update global maximum
        max_sum = max(max_sum, current_path)
        
        # Return max path going through this node (single branch)
        return node.val + max(left_gain, right_gain)
    
    max_gain(root)
    return max_sum
```

**Time Complexity:** O(n)  
**Space Complexity:** O(h) where h is tree height

---

### 33. Problem #139: Word Break
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/word-break/

**Problem Description:**
Given a string `s` and a dictionary of strings `wordDict`, return true if `s` can be segmented into a space-separated sequence of one or more dictionary words. Note that the same word may be reused multiple times.

**Examples:**
- Input: `s = "leetcode"`, `wordDict = ["leet","code"]` → Output: `true`
- Input: `s = "applepenapple"`, `wordDict = ["apple","pen"]` → Output: `true`
- Input: `s = "catsandog"`, `wordDict = ["cats","dog","sand","and","cat"]` → Output: `false`

**Simple Explanation:**
Use DP where dp[i] means the first i characters can be segmented. For each position, check all possible words ending at that position. If a word matches and the part before it can be segmented (dp[start] is true), then dp[i] is true.

**Algorithm Steps:**
1. Create dp array of size n+1, initialize dp[0] = True (empty string)
2. Convert wordDict to set for O(1) lookup
3. For each position i from 1 to n:
   - For each position j from 0 to i:
     - If dp[j] is True and s[j:i] is in wordDict:
       - Set dp[i] = True and break
4. Return dp[n]

**Python Solution:**
```python
def wordBreak(s: str, wordDict: list[str]) -> bool:
    n = len(s)
    word_set = set(wordDict)
    dp = [False] * (n + 1)
    dp[0] = True  # Empty string
    
    for i in range(1, n + 1):
        for j in range(i):
            # Check if s[0:j] can be segmented and s[j:i] is a word
            if dp[j] and s[j:i] in word_set:
                dp[i] = True
                break
    
    return dp[n]
```

**Time Complexity:** O(n² × m) where m is average word length  
**Space Complexity:** O(n)

---

### 34. Problem #128: Longest Consecutive Sequence
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/longest-consecutive-sequence/

**Problem Description:**
Given an unsorted array of integers `nums`, return the length of the longest consecutive elements sequence. You must write an algorithm that runs in O(n) time.

**Examples:**
- Input: `nums = [100,4,200,1,3,2]` → Output: `4` (The sequence is [1,2,3,4])
- Input: `nums = [0,3,7,2,5,8,4,6,0,1]` → Output: `9`

**Simple Explanation:**
Put all numbers in a set for O(1) lookup. For each number, check if it's the start of a sequence (no number before it exists). If yes, count how long the sequence is by checking num+1, num+2, etc. Keep track of the maximum length found.

**Algorithm Steps:**
1. Convert array to set
2. Initialize max_length = 0
3. For each number in the set:
   - Check if number-1 exists (if yes, skip - not a sequence start)
   - If it's a sequence start, count the length:
     - While number+1, number+2, etc. exist, increment length
   - Update max_length
4. Return max_length

**Python Solution:**
```python
def longestConsecutive(nums: list[int]) -> int:
    if not nums:
        return 0
    
    num_set = set(nums)
    max_length = 0
    
    for num in num_set:
        # Only start counting if this is the beginning of a sequence
        if num - 1 not in num_set:
            current_num = num
            current_length = 1
            
            # Count consecutive numbers
            while current_num + 1 in num_set:
                current_num += 1
                current_length += 1
            
            max_length = max(max_length, current_length)
    
    return max_length
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n)

---

## Stack (6 problems)

### 35. Problem #20: Valid Parentheses
**Difficulty:** Easy  
**URL:** https://leetcode.com/problems/valid-parentheses/

**Problem Description:**
Given a string `s` containing just the characters '(', ')', '{', '}', '[' and ']', determine if the input string is valid. An input string is valid if: open brackets are closed by the same type of brackets and open brackets are closed in the correct order.

**Examples:**
- Input: `s = "()"` → Output: `true`
- Input: `s = "()[]{}"` → Output: `true`
- Input: `s = "(]"` → Output: `false`

**Simple Explanation:**
Use a stack. When you see an opening bracket, push it onto the stack. When you see a closing bracket, check if the top of the stack has the matching opening bracket. If yes, pop it. If the stack is empty or doesn't match, return false. At the end, the stack should be empty.

**Algorithm Steps:**
1. Create an empty stack
2. Create a mapping of closing to opening brackets
3. For each character:
   - If it's a closing bracket:
     - Check if stack is empty or top doesn't match -> return False
     - Otherwise pop from stack
   - If it's an opening bracket:
     - Push to stack
4. Return True if stack is empty

**Python Solution:**
```python
def isValid(s: str) -> bool:
    stack = []
    mapping = {')': '(', '}': '{', ']': '['}
    
    for char in s:
        if char in mapping:
            # Closing bracket
            if not stack or stack[-1] != mapping[char]:
                return False
            stack.pop()
        else:
            # Opening bracket
            stack.append(char)
    
    return len(stack) == 0
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n)

---

### 36. Problem #71: Simplify Path
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/simplify-path/

**Problem Description:**
Given a string `path`, which is an absolute path (starting with '/') to a file or directory in a Unix-style file system, convert it to the simplified canonical path. Return the simplified path.

**Examples:**
- Input: `path = "/home/"` → Output: `"/home"`
- Input: `path = "/../"` → Output: `"/"`
- Input: `path = "/home//foo/"` → Output: `"/home/foo"`

**Simple Explanation:**
Split the path by '/'. Use a stack to track directories. For each part: if it's '..' and stack is not empty, pop (go up one directory). If it's '.' or empty, skip it. Otherwise, push to stack. Finally, join the stack with '/' to create the simplified path.

**Algorithm Steps:**
1. Split path by '/'
2. Create a stack for directory names
3. For each part:
   - If '..': pop from stack if not empty (go up)
   - If '.' or empty: skip
   - Otherwise: push to stack
4. Join stack with '/' and add leading '/'
5. Return simplified path (or '/' if empty)

**Python Solution:**
```python
def simplifyPath(path: str) -> str:
    stack = []
    parts = path.split('/')
    
    for part in parts:
        if part == '..':
            if stack:
                stack.pop()
        elif part and part != '.':
            stack.append(part)
    
    return '/' + '/'.join(stack)
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n)

---

### 37. Problem #224: Basic Calculator
**Difficulty:** Hard  
**URL:** https://leetcode.com/problems/basic-calculator/

**Problem Description:**
Given a string `s` representing a valid expression, implement a basic calculator to evaluate it. The expression string may contain open '(' and closing ')' parentheses, the plus '+' or minus '-' sign, non-negative integers and empty spaces.

**Examples:**
- Input: `s = "1 + 1"` → Output: `2`
- Input: `s = " 2-1 + 2 "` → Output: `3`
- Input: `s = "(1+(4+5+2)-3)+(6+8)"` → Output: `23`

**Simple Explanation:**
Use a stack to handle parentheses. Keep track of the current number, current result, and sign. When you see a digit, build the number. When you see an operator or parenthesis, apply the current number with its sign. For '(', push current result and sign onto stack. For ')', pop from stack and apply.

**Algorithm Steps:**
1. Initialize: result = 0, number = 0, sign = 1, stack = []
2. For each character:
   - If digit: build the number
   - If '+': add number to result, reset number, set sign = 1
   - If '-': add number to result, reset number, set sign = -1
   - If '(': push result and sign to stack, reset result and sign
   - If ')': add number to result, pop from stack and apply
3. Add any remaining number to result
4. Return result

**Python Solution:**
```python
def calculate(s: str) -> int:
    stack = []
    number = 0
    result = 0
    sign = 1  # 1 for positive, -1 for negative
    
    for char in s:
        if char.isdigit():
            number = number * 10 + int(char)
        elif char == '+':
            result += sign * number
            number = 0
            sign = 1
        elif char == '-':
            result += sign * number
            number = 0
            sign = -1
        elif char == '(':
            # Push current result and sign
            stack.append(result)
            stack.append(sign)
            result = 0
            sign = 1
        elif char == ')':
            result += sign * number
            number = 0
            # Pop sign and previous result
            result *= stack.pop()  # sign
            result += stack.pop()  # previous result
    
    result += sign * number
    return result
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n)

---

### 38. Problem #394: Decode String
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/decode-string/

**Problem Description:**
Given an encoded string, return its decoded string. The encoding rule is: k[encoded_string], where the encoded_string inside the brackets is repeated exactly k times. You may assume the input string is always valid.

**Examples:**
- Input: `s = "3[a]2[bc]"` → Output: `"aaabcbc"`
- Input: `s = "3[a2[c]]"` → Output: `"accaccacc"`
- Input: `s = "2[abc]3[cd]ef"` → Output: `"abcabccdcdcdef"`

**Simple Explanation:**
Use a stack. When you see a number, start building it. When you see '[', push the current string and number to stack. When you see ']', pop the number and previous string, repeat the current string that many times, and append to previous string. For letters, just add to current string.

**Algorithm Steps:**
1. Create stack, current_string = "", current_num = 0
2. For each character:
   - If digit: build current_num
   - If '[': push (current_string, current_num) to stack, reset both
   - If ']': pop (prev_string, num), current_string = prev_string + num * current_string
   - If letter: add to current_string
3. Return current_string

**Python Solution:**
```python
def decodeString(s: str) -> str:
    stack = []
    current_string = ""
    current_num = 0
    
    for char in s:
        if char.isdigit():
            current_num = current_num * 10 + int(char)
        elif char == '[':
            # Push current state to stack
            stack.append((current_string, current_num))
            current_string = ""
            current_num = 0
        elif char == ']':
            # Pop and decode
            prev_string, num = stack.pop()
            current_string = prev_string + num * current_string
        else:
            # Regular letter
            current_string += char
    
    return current_string
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n)

---

### 39. Problem #735: Asteroid Collision
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/asteroid-collision/

**Problem Description:**
Given an array `asteroids` of integers representing asteroids in a row, for each asteroid, the absolute value represents its size, and the sign represents its direction (positive right, negative left). Find out the state of the asteroids after all collisions. If two meet, the smaller one explodes. If both are the same size, both explode.

**Examples:**
- Input: `asteroids = [5,10,-5]` → Output: `[5,10]`
- Input: `asteroids = [8,-8]` → Output: `[]`
- Input: `asteroids = [10,2,-5]` → Output: `[10]`

**Simple Explanation:**
Use a stack. Process asteroids left to right. If current asteroid is moving right (positive), push to stack. If moving left (negative), it might collide with right-moving asteroids in the stack. Keep popping from stack until the current asteroid is destroyed, destroys all right-moving ones, or no collision happens.

**Algorithm Steps:**
1. Create a stack
2. For each asteroid:
   - If positive (moving right), push to stack
   - If negative (moving left):
     - While stack has positive asteroids (collision):
       - If top is smaller, pop and continue
       - If top equals abs(asteroid), pop and break (both destroyed)
       - If top is larger, break (current destroyed)
     - If no collision or all popped, push current asteroid
3. Return stack

**Python Solution:**
```python
def asteroidCollision(asteroids: list[int]) -> list[int]:
    stack = []
    
    for asteroid in asteroids:
        while stack and asteroid < 0 < stack[-1]:
            # Collision: right-moving (stack top) vs left-moving (asteroid)
            if stack[-1] < -asteroid:
                stack.pop()  # Right-moving destroyed
                continue
            elif stack[-1] == -asteroid:
                stack.pop()  # Both destroyed
            break  # Current asteroid destroyed or no more collisions
        else:
            stack.append(asteroid)
    
    return stack
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n)

---

### 40. Problem #772: Basic Calculator III (Premium)
**Difficulty:** Hard  
**URL:** https://leetcode.com/problems/basic-calculator-iii/

**Problem Description:**
Implement a basic calculator to evaluate a simple expression string. The expression string contains only non-negative integers, '+', '-', '*', '/' operators, open '(' and closing ')' parentheses and empty spaces. The integer division should truncate toward zero.

**Examples:**
- Input: `s = "1+1"` → Output: `2`
- Input: `s = "6-4/2"` → Output: `4`
- Input: `s = "2*(5+5*2)/3+(6/2+8)"` → Output: `21`

**Simple Explanation:**
Use a stack to evaluate expressions with proper operator precedence. Process multiplication and division immediately. For addition and subtraction, push to stack and evaluate later. Handle parentheses using recursion or another stack. Keep track of the current number, operation, and use the stack for intermediate results.

**Algorithm Steps:**
1. Use recursion or stack to handle parentheses
2. For each segment:
   - Track current number, last operation, and stack
   - When seeing operator or end:
     - Apply last operation: +/- push to stack, */ calculate immediately
   - Update last operation
3. Sum all values in stack for final result

**Python Solution:**
```python
def calculate(s: str) -> int:
    def helper(i):
        stack = []
        num = 0
        operation = '+'
        
        while i < len(s):
            char = s[i]
            
            if char.isdigit():
                num = num * 10 + int(char)
            elif char == '(':
                # Recursively evaluate parentheses
                num, i = helper(i + 1)
            
            if char in '+-*/' or i == len(s) - 1 or char == ')':
                if operation == '+':
                    stack.append(num)
                elif operation == '-':
                    stack.append(-num)
                elif operation == '*':
                    stack.append(stack.pop() * num)
                elif operation == '/':
                    stack.append(int(stack.pop() / num))
                
                if char == ')':
                    return sum(stack), i
                
                operation = char
                num = 0
            
            i += 1
        
        return sum(stack), i
    
    result, _ = helper(0)
    return result
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n)

---

## Heap (Priority Queue) (4 problems)

### 41. Problem #347: Top K Frequent Elements
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/top-k-frequent-elements/

**Problem Description:**
Given an integer array `nums` and an integer `k`, return the k most frequent elements. You may return the answer in any order.

**Examples:**
- Input: `nums = [1,1,1,2,2,3]`, `k = 2` → Output: `[1,2]`
- Input: `nums = [1]`, `k = 1` → Output: `[1]`

**Simple Explanation:**
Count the frequency of each element using a dictionary. Then use a min-heap of size k to keep track of the k most frequent elements. If heap size exceeds k, remove the element with lowest frequency. Alternatively, use a max-heap with all elements and extract k times.

**Algorithm Steps:**
1. Count frequency of each element using dictionary
2. Method 1 - Min heap of size k:
   - For each unique element:
     - Add (frequency, element) to heap
     - If heap size > k, remove minimum
   - Return all elements in heap
3. Method 2 - Bucket sort (O(n)):
   - Create buckets where bucket[i] contains elements with frequency i
   - Iterate from highest frequency down, collect k elements

**Python Solution:**
```python
import heapq
from collections import Counter

def topKFrequent(nums: list[int], k: int) -> list[int]:
    # Count frequencies
    count = Counter(nums)
    
    # Use heap to find k most frequent
    # Python heapq is min-heap, so use negative frequencies for max-heap
    return heapq.nlargest(k, count.keys(), key=count.get)

# Alternative: Using bucket sort for O(n)
def topKFrequent_bucket(nums: list[int], k: int) -> list[int]:
    count = Counter(nums)
    # Create buckets: bucket[i] = elements with frequency i
    buckets = [[] for _ in range(len(nums) + 1)]
    
    for num, freq in count.items():
        buckets[freq].append(num)
    
    result = []
    # Iterate from highest frequency to lowest
    for i in range(len(buckets) - 1, 0, -1):
        for num in buckets[i]:
            result.append(num)
            if len(result) == k:
                return result
    
    return result
```

**Time Complexity:** O(n log k) with heap, O(n) with bucket sort  
**Space Complexity:** O(n)

---

### 42. Problem #23: Merge k Sorted Lists
**Difficulty:** Hard  
**URL:** https://leetcode.com/problems/merge-k-sorted-lists/

**Problem Description:**
You are given an array of k linked-lists `lists`, each linked-list is sorted in ascending order. Merge all the linked-lists into one sorted linked-list and return it.

**Examples:**
- Input: `lists = [[1,4,5],[1,3,4],[2,6]]` → Output: `[1,1,2,3,4,4,5,6]`
- Input: `lists = []` → Output: `[]`

**Simple Explanation:**
Use a min-heap to always extract the smallest element across all lists. Initially, add the first node from each list to the heap. When you extract a node, add its next node to the heap. Keep doing this until all nodes are processed. Use the node's value for heap comparison.

**Algorithm Steps:**
1. Create a min-heap
2. Add first node from each list to heap (with value as priority)
3. Create a dummy node for result
4. While heap is not empty:
   - Extract minimum node
   - Add to result list
   - If extracted node has a next node, add it to heap
5. Return result list

**Python Solution:**
```python
import heapq

class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

def mergeKLists(lists: list[ListNode]) -> ListNode:
    heap = []
    
    # Add first node from each list
    for i, node in enumerate(lists):
        if node:
            # Use index as tiebreaker for nodes with same value
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

**Time Complexity:** O(N log k) where N is total nodes and k is number of lists  
**Space Complexity:** O(k) for the heap

---

### 43. Problem #253: Meeting Rooms II (Premium)
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/meeting-rooms-ii/

**Problem Description:**
Given an array of meeting time intervals where intervals[i] = [start_i, end_i], return the minimum number of conference rooms required.

**Examples:**
- Input: `intervals = [[0,30],[5,10],[15,20]]` → Output: `2`
- Input: `intervals = [[7,10],[2,4]]` → Output: `1`

**Simple Explanation:**
Sort meetings by start time. Use a min-heap to track end times of ongoing meetings. When a new meeting starts, check if any meeting has ended (minimum end time in heap). If yes, remove it from heap. Add current meeting's end time to heap. The size of the heap represents rooms needed at any time.

**Algorithm Steps:**
1. Sort intervals by start time
2. Create a min-heap for end times
3. For each meeting:
   - While heap is not empty and minimum end time <= current start:
     - Remove from heap (room becomes free)
   - Add current meeting's end time to heap
   - Update maximum heap size seen
4. Return maximum heap size

**Python Solution:**
```python
import heapq

def minMeetingRooms(intervals: list[list[int]]) -> int:
    if not intervals:
        return 0
    
    # Sort by start time
    intervals.sort(key=lambda x: x[0])
    
    # Min-heap for end times
    heap = []
    heapq.heappush(heap, intervals[0][1])
    
    for i in range(1, len(intervals)):
        # If earliest meeting has ended, remove it
        if heap[0] <= intervals[i][0]:
            heapq.heappop(heap)
        
        # Add current meeting's end time
        heapq.heappush(heap, intervals[i][1])
    
    # Heap size is number of rooms needed
    return len(heap)
```

**Time Complexity:** O(n log n)  
**Space Complexity:** O(n)

---

### 44. Problem #2402: Meeting Rooms III
**Difficulty:** Hard  
**URL:** https://leetcode.com/problems/meeting-rooms-iii/

**Problem Description:**
You are given an integer `n` and a 2D array `meetings` where meetings[i] = [start_i, end_i] means a meeting from start to end. All values are in the range [0, 10^6]. Return the room number used most often. If there is a tie, return the room with the smallest number.

**Examples:**
- Input: `n = 2`, `meetings = [[0,10],[1,5],[2,7],[3,4]]` → Output: `0`
- Input: `n = 3`, `meetings = [[1,20],[2,10],[3,5],[4,9],[6,8]]` → Output: `1`

**Simple Explanation:**
Use two heaps: one for available rooms and one for occupied rooms (with end times). Sort meetings by start time. For each meeting, free up rooms that have ended. If no room is available, wait for the earliest one to free up and delay the meeting. Track which room is used most.

**Algorithm Steps:**
1. Sort meetings by start time
2. Create min-heap for available rooms (0 to n-1)
3. Create min-heap for occupied rooms (end_time, room_number)
4. Track room usage count
5. For each meeting:
   - Free rooms that ended before or at start time
   - If no available room, pop earliest ending room and delay meeting
   - Assign room and update count
6. Return room with highest count (lowest number in ties)

**Python Solution:**
```python
import heapq

def mostBooked(n: int, meetings: list[list[int]]) -> int:
    # Sort meetings by start time
    meetings.sort()
    
    # Heap for available rooms
    available = list(range(n))
    heapq.heapify(available)
    
    # Heap for occupied rooms: (end_time, room_number)
    occupied = []
    
    # Track room usage
    room_count = [0] * n
    
    for start, end in meetings:
        # Free up rooms that have ended
        while occupied and occupied[0][0] <= start:
            _, room = heapq.heappop(occupied)
            heapq.heappush(available, room)
        
        if available:
            # Room is available
            room = heapq.heappop(available)
            heapq.heappush(occupied, (end, room))
        else:
            # No room available, use earliest ending room
            earliest_end, room = heapq.heappop(occupied)
            # Meeting is delayed
            new_end = earliest_end + (end - start)
            heapq.heappush(occupied, (new_end, room))
        
        room_count[room] += 1
    
    # Find room used most often (smallest number in ties)
    max_count = max(room_count)
    return room_count.index(max_count)
```

**Time Complexity:** O(m log n) where m is meetings and n is rooms  
**Space Complexity:** O(n)

---

## Binary Search (3 problems)

### 45. Problem #69: Sqrt(x)
**Difficulty:** Easy  
**URL:** https://leetcode.com/problems/sqrtx/

**Problem Description:**
Given a non-negative integer `x`, return the square root of `x` rounded down to the nearest integer. The returned integer should be non-negative as well. You must not use any built-in exponent function or operator.

**Examples:**
- Input: `x = 4` → Output: `2`
- Input: `x = 8` → Output: `2` (sqrt(8) = 2.828...)

**Simple Explanation:**
Use binary search between 0 and x. For each middle value, check if its square is less than, equal to, or greater than x. If mid² ≤ x, it could be the answer, so search the right half for a better answer. If mid² > x, search the left half.

**Algorithm Steps:**
1. Handle edge cases: if x < 2, return x
2. Initialize left = 0, right = x // 2
3. While left <= right:
   - Calculate mid
   - If mid² equals x, return mid
   - If mid² < x, save mid as potential answer, search right (left = mid + 1)
   - If mid² > x, search left (right = mid - 1)
4. Return the saved answer

**Python Solution:**
```python
def mySqrt(x: int) -> int:
    if x < 2:
        return x
    
    left, right = 0, x // 2
    
    while left <= right:
        mid = (left + right) // 2
        squared = mid * mid
        
        if squared == x:
            return mid
        elif squared < x:
            left = mid + 1
        else:
            right = mid - 1
    
    # Right pointer will be at the floor of sqrt(x)
    return right
```

**Time Complexity:** O(log n)  
**Space Complexity:** O(1)

---

### 46. Problem #153: Find Minimum in Rotated Sorted Array
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/

**Problem Description:**
Suppose an array of length n sorted in ascending order is rotated between 1 and n times. Given the rotated array `nums` of unique elements, return the minimum element. You must write an algorithm that runs in O(log n) time.

**Examples:**
- Input: `nums = [3,4,5,1,2]` → Output: `1`
- Input: `nums = [4,5,6,7,0,1,2]` → Output: `0`
- Input: `nums = [11,13,15,17]` → Output: `11`

**Simple Explanation:**
Use binary search. The minimum element is where the rotation happened. Compare the middle element with the rightmost element. If mid > right, the minimum is in the right half. If mid < right, the minimum is in the left half (including mid). Keep narrowing until you find it.

**Algorithm Steps:**
1. Initialize left = 0, right = len(nums) - 1
2. While left < right:
   - Calculate mid
   - If nums[mid] > nums[right]:
     - Minimum is in right half: left = mid + 1
   - Else:
     - Minimum is in left half: right = mid
3. Return nums[left]

**Python Solution:**
```python
def findMin(nums: list[int]) -> int:
    left, right = 0, len(nums) - 1
    
    while left < right:
        mid = (left + right) // 2
        
        if nums[mid] > nums[right]:
            # Minimum is in the right half
            left = mid + 1
        else:
            # Minimum is in the left half (including mid)
            right = mid
    
    return nums[left]
```

**Time Complexity:** O(log n)  
**Space Complexity:** O(1)

---

### 47. Problem #1044: Longest Duplicate Substring
**Difficulty:** Hard  
**URL:** https://leetcode.com/problems/longest-duplicate-substring/

**Problem Description:**
Given a string `s`, consider all duplicated substrings (contiguous substrings of `s` that occur 2 or more times). Return any duplicated substring that has the longest possible length. If `s` does not have a duplicate substring, return "".

**Examples:**
- Input: `s = "banana"` → Output: `"ana"`
- Input: `s = "abcd"` → Output: `""`

**Simple Explanation:**
Use binary search on the length of substring combined with rolling hash (Rabin-Karp) to check for duplicates efficiently. Binary search finds the maximum length where a duplicate exists. For each length, use rolling hash to check all substrings of that length for duplicates in O(n) time.

**Algorithm Steps:**
1. Binary search on substring length (left = 1, right = len(s) - 1)
2. For each length mid:
   - Use rolling hash to check if any substring of length mid appears twice
   - If duplicate found, save it and search for longer (left = mid + 1)
   - If no duplicate, search for shorter (right = mid - 1)
3. Return the longest duplicate found

**Python Solution:**
```python
def longestDupSubstring(s: str) -> str:
    def search(length):
        # Rabin-Karp rolling hash
        base = 26
        mod = 2**63 - 1
        
        # Compute hash for first substring
        h = 0
        for i in range(length):
            h = (h * base + ord(s[i])) % mod
        
        seen = {h}
        # Highest power of base for length
        power = pow(base, length, mod)
        
        # Rolling hash for remaining substrings
        for start in range(1, len(s) - length + 1):
            # Remove leftmost character and add rightmost
            h = (h * base - ord(s[start - 1]) * power + ord(s[start + length - 1])) % mod
            if h in seen:
                return start  # Found duplicate
            seen.add(h)
        
        return -1
    
    # Binary search on length
    left, right = 1, len(s)
    result = ""
    
    while left <= right:
        mid = (left + right) // 2
        start = search(mid)
        
        if start != -1:
            result = s[start:start + mid]
            left = mid + 1
        else:
            right = mid - 1
    
    return result
```

**Time Complexity:** O(n log n)  
**Space Complexity:** O(n)

---

## Hash Map / Hash Set (4 problems)

### 48. Problem #146: LRU Cache
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/lru-cache/

**Problem Description:**
Design a data structure that follows Least Recently Used (LRU) cache constraints. Implement `LRUCache` class with methods: `get(key)` returns the value if key exists, otherwise -1. `put(key, value)` updates or inserts the key-value pair. If capacity is reached, evict the least recently used key before inserting a new item.

**Examples:**
```
LRUCache cache = new LRUCache(2);
cache.put(1, 1);
cache.put(2, 2);
cache.get(1);       // returns 1
cache.put(3, 3);    // evicts key 2
cache.get(2);       // returns -1
```

**Simple Explanation:**
Use a combination of a hash map and a doubly linked list. The hash map provides O(1) access to nodes. The doubly linked list maintains order of use (most recent at head, least recent at tail). When accessing or adding a node, move it to the head. When capacity is full, remove the tail node.

**Algorithm Steps:**
1. Create a doubly linked list node class with key, value, prev, and next
2. Use a hash map to store key -> node mapping
3. Maintain head and tail dummy nodes
4. For get(key):
   - If key exists, move node to head and return value
   - Otherwise return -1
5. For put(key, value):
   - If key exists, update value and move to head
   - If key doesn't exist:
     - If at capacity, remove tail node from list and map
     - Create new node, add to head and map

**Python Solution:**
```python
class Node:
    def __init__(self, key=0, value=0):
        self.key = key
        self.value = value
        self.prev = None
        self.next = None

class LRUCache:
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.cache = {}  # key -> node
        # Dummy head and tail
        self.head = Node()
        self.tail = Node()
        self.head.next = self.tail
        self.tail.prev = self.head
    
    def _remove(self, node):
        """Remove node from list"""
        prev_node = node.prev
        next_node = node.next
        prev_node.next = next_node
        next_node.prev = prev_node
    
    def _add_to_head(self, node):
        """Add node right after head"""
        node.prev = self.head
        node.next = self.head.next
        self.head.next.prev = node
        self.head.next = node
    
    def get(self, key: int) -> int:
        if key in self.cache:
            node = self.cache[key]
            # Move to head (most recently used)
            self._remove(node)
            self._add_to_head(node)
            return node.value
        return -1
    
    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            # Update existing node
            node = self.cache[key]
            node.value = value
            self._remove(node)
            self._add_to_head(node)
        else:
            # Add new node
            if len(self.cache) >= self.capacity:
                # Remove least recently used (tail.prev)
                lru = self.tail.prev
                self._remove(lru)
                del self.cache[lru.key]
            
            # Add new node to head
            node = Node(key, value)
            self.cache[key] = node
            self._add_to_head(node)
```

**Time Complexity:** O(1) for both operations  
**Space Complexity:** O(capacity)

---

### 49. Problem #49: Group Anagrams
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/group-anagrams/

**Problem Description:**
Given an array of strings `strs`, group the anagrams together. You can return the answer in any order. An anagram is a word formed by rearranging the letters of another word, using all original letters exactly once.

**Examples:**
- Input: `strs = ["eat","tea","tan","ate","nat","bat"]`
- Output: `[["bat"],["nat","tan"],["ate","eat","tea"]]`

**Simple Explanation:**
Anagrams have the same characters, just in different order. Sort each word's characters to create a key. Words with the same sorted key are anagrams. Use a dictionary where the key is the sorted string and the value is a list of words with that pattern.

**Algorithm Steps:**
1. Create a dictionary to store anagram groups
2. For each word:
   - Sort the characters to create a key
   - Add the word to the list for that key
3. Return all values from the dictionary

**Python Solution:**
```python
from collections import defaultdict

def groupAnagrams(strs: list[str]) -> list[list[str]]:
    anagrams = defaultdict(list)
    
    for word in strs:
        # Sort characters to create key
        key = ''.join(sorted(word))
        anagrams[key].append(word)
    
    return list(anagrams.values())

# Alternative: Using character count as key
def groupAnagrams_count(strs: list[str]) -> list[list[str]]:
    anagrams = defaultdict(list)
    
    for word in strs:
        # Count characters (a-z)
        count = [0] * 26
        for char in word:
            count[ord(char) - ord('a')] += 1
        # Use tuple of counts as key
        key = tuple(count)
        anagrams[key].append(word)
    
    return list(anagrams.values())
```

**Time Complexity:** O(n × k log k) where n is number of words and k is max word length  
**Space Complexity:** O(n × k)

---

### 50. Problem #560: Subarray Sum Equals K (Already covered in Sliding Window)
This problem is already covered as problem #3 in the Sliding Window section.

---

### 51. Problem #128: Longest Consecutive Sequence (Already covered in DP)
This problem is already covered as problem #34 in the Dynamic Programming section.

---

## Graph Algorithms (4 problems)

### 52. Problem #207: Course Schedule
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/course-schedule/

**Problem Description:**
There are `n` courses labeled from 0 to n-1. You are given an array `prerequisites` where prerequisites[i] = [a, b] indicates you must take course b before course a. Return true if you can finish all courses, false otherwise.

**Examples:**
- Input: `numCourses = 2`, `prerequisites = [[1,0]]` → Output: `true`
- Input: `numCourses = 2`, `prerequisites = [[1,0],[0,1]]` → Output: `false` (circular dependency)

**Simple Explanation:**
This is a cycle detection problem in a directed graph. Build a graph from prerequisites. Use DFS to detect cycles. Mark nodes with three states: unvisited (0), visiting (1), and visited (2). If you encounter a node that's currently being visited (1), there's a cycle.

**Algorithm Steps:**
1. Build adjacency list from prerequisites
2. Create a state array (0 = unvisited, 1 = visiting, 2 = visited)
3. For each course, if unvisited, start DFS
4. DFS function:
   - Mark as visiting (1)
   - For each neighbor:
     - If visiting (1), return False (cycle found)
     - If unvisited (0), recursively check
   - Mark as visited (2)
   - Return True
5. Return True if no cycles found

**Python Solution:**
```python
def canFinish(numCourses: int, prerequisites: list[list[int]]) -> bool:
    # Build adjacency list
    graph = [[] for _ in range(numCourses)]
    for course, prereq in prerequisites:
        graph[course].append(prereq)
    
    # 0 = unvisited, 1 = visiting, 2 = visited
    state = [0] * numCourses
    
    def has_cycle(course):
        if state[course] == 1:
            return True  # Cycle detected
        if state[course] == 2:
            return False  # Already checked
        
        state[course] = 1  # Mark as visiting
        
        for prereq in graph[course]:
            if has_cycle(prereq):
                return True
        
        state[course] = 2  # Mark as visited
        return False
    
    for course in range(numCourses):
        if state[course] == 0:
            if has_cycle(course):
                return False
    
    return True
```

**Time Complexity:** O(V + E) where V is courses and E is prerequisites  
**Space Complexity:** O(V + E)

---

### 53. Problem #210: Course Schedule II
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/course-schedule-ii/

**Problem Description:**
Return the ordering of courses you should take to finish all courses. If there are many valid answers, return any of them. If it's impossible to finish all courses, return an empty array.

**Examples:**
- Input: `numCourses = 2`, `prerequisites = [[1,0]]` → Output: `[0,1]`
- Input: `numCourses = 4`, `prerequisites = [[1,0],[2,0],[3,1],[3,2]]` → Output: `[0,2,1,3]` or `[0,1,2,3]`

**Simple Explanation:**
This is topological sorting. Use DFS with post-order traversal (add to result after visiting all neighbors) or use Kahn's algorithm with in-degrees. We'll use DFS: after checking all prerequisites of a course, add it to the result. Reverse the result at the end.

**Algorithm Steps:**
1. Build adjacency list from prerequisites
2. Use DFS with state tracking (same as Course Schedule I)
3. After checking all prerequisites, add course to result
4. If cycle is detected, return empty array
5. Reverse the result before returning (post-order)

**Python Solution:**
```python
def findOrder(numCourses: int, prerequisites: list[list[int]]) -> list[int]:
    # Build adjacency list
    graph = [[] for _ in range(numCourses)]
    for course, prereq in prerequisites:
        graph[course].append(prereq)
    
    # 0 = unvisited, 1 = visiting, 2 = visited
    state = [0] * numCourses
    result = []
    
    def dfs(course):
        if state[course] == 1:
            return False  # Cycle detected
        if state[course] == 2:
            return True  # Already processed
        
        state[course] = 1  # Mark as visiting
        
        for prereq in graph[course]:
            if not dfs(prereq):
                return False
        
        state[course] = 2  # Mark as visited
        result.append(course)  # Add to result after processing prerequisites
        return True
    
    for course in range(numCourses):
        if state[course] == 0:
            if not dfs(course):
                return []
    
    return result  # Already in correct order due to post-order traversal
```

**Time Complexity:** O(V + E)  
**Space Complexity:** O(V + E)

---

### 54. Problem #127: Word Ladder
**Difficulty:** Hard  
**URL:** https://leetcode.com/problems/word-ladder/

**Problem Description:**
Given two words `beginWord` and `endWord`, and a dictionary `wordList`, return the number of words in the shortest transformation sequence from beginWord to endWord. Each transformed word must exist in the word list, and you can only change one letter at a time.

**Examples:**
- Input: `beginWord = "hit"`, `endWord = "cog"`, `wordList = ["hot","dot","dog","lot","log","cog"]`
- Output: `5` (Explanation: "hit" -> "hot" -> "dot" -> "dog" -> "cog")

**Simple Explanation:**
Use BFS to find the shortest path. Start from beginWord. For each word, try changing each letter to all 26 letters. If the resulting word is in the dictionary and not visited, add it to the queue. Count the levels of BFS - that's your answer.

**Algorithm Steps:**
1. Check if endWord is in wordList, if not return 0
2. Create a queue with (beginWord, 1) and a visited set
3. BFS:
   - For each word, try changing each position to a-z
   - If resulting word equals endWord, return length
   - If resulting word in wordList and not visited:
     - Add to queue with length + 1
     - Mark as visited
4. If endWord not reached, return 0

**Python Solution:**
```python
from collections import deque

def ladderLength(beginWord: str, endWord: str, wordList: list[str]) -> int:
    if endWord not in wordList:
        return 0
    
    word_set = set(wordList)
    queue = deque([(beginWord, 1)])
    visited = {beginWord}
    
    while queue:
        word, length = queue.popleft()
        
        # Try changing each letter
        for i in range(len(word)):
            for char in 'abcdefghijklmnopqrstuvwxyz':
                next_word = word[:i] + char + word[i+1:]
                
                if next_word == endWord:
                    return length + 1
                
                if next_word in word_set and next_word not in visited:
                    visited.add(next_word)
                    queue.append((next_word, length + 1))
    
    return 0
```

**Time Complexity:** O(M² × N) where M is word length and N is word list size  
**Space Complexity:** O(M × N)

---

### 55. Problem #399: Evaluate Division (Already covered in DFS)
This problem is already covered as problem #17 in the DFS section.

---

## String Manipulation (6 problems)

### 56. Problem #68: Text Justification
**Difficulty:** Hard  
**URL:** https://leetcode.com/problems/text-justification/

**Problem Description:**
Given an array of strings `words` and a width `maxWidth`, format the text such that each line has exactly maxWidth characters and is fully justified. Pack as many words as possible in each line. Pad extra spaces uniformly between words. For the last line, it should be left-justified with no extra space between words.

**Examples:**
- Input: `words = ["This", "is", "an", "example", "of", "text", "justification."]`, `maxWidth = 16`
- Output:
```
[
   "This    is    an",
   "example  of text",
   "justification.  "
]
```

**Simple Explanation:**
Greedy approach: pack as many words as fit in maxWidth. For each line, calculate how many spaces needed, and distribute them evenly (more spaces go to left slots when spaces don't divide evenly). Last line is special: left-justified with single spaces between words and padded with spaces at the end.

**Algorithm Steps:**
1. Group words into lines (greedy: fit as many as possible)
2. For each line (except last):
   - Calculate total spaces needed
   - Distribute evenly between words (extra spaces go left)
3. For last line:
   - Join words with single spaces
   - Pad right with spaces
4. Return all lines

**Python Solution:**
```python
def fullJustify(words: list[str], maxWidth: int) -> list[str]:
    result = []
    current_line = []
    current_length = 0
    
    for word in words:
        # Check if word fits in current line (with spaces)
        if current_length + len(word) + len(current_line) > maxWidth:
            # Justify current line
            result.append(justify_line(current_line, maxWidth, False))
            current_line = []
            current_length = 0
        
        current_line.append(word)
        current_length += len(word)
    
    # Last line (left-justified)
    result.append(justify_line(current_line, maxWidth, True))
    return result

def justify_line(words, maxWidth, is_last):
    if is_last or len(words) == 1:
        # Left-justify
        line = ' '.join(words)
        return line + ' ' * (maxWidth - len(line))
    
    # Full justify
    total_chars = sum(len(word) for word in words)
    total_spaces = maxWidth - total_chars
    gaps = len(words) - 1
    
    space_per_gap = total_spaces // gaps
    extra_spaces = total_spaces % gaps
    
    line = ''
    for i, word in enumerate(words[:-1]):
        line += word
        line += ' ' * space_per_gap
        if i < extra_spaces:
            line += ' '
    line += words[-1]
    
    return line
```

**Time Complexity:** O(n × maxWidth) where n is number of words  
**Space Complexity:** O(maxWidth)

---

### 57. Problem #43: Multiply Strings
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/multiply-strings/

**Problem Description:**
Given two non-negative integers `num1` and `num2` represented as strings, return the product of num1 and num2, also represented as a string. You must not use any built-in BigInteger library or convert the inputs to integer directly.

**Examples:**
- Input: `num1 = "2"`, `num2 = "3"` → Output: `"6"`
- Input: `num1 = "123"`, `num2 = "456"` → Output: `"56088"`

**Simple Explanation:**
Simulate multiplication like you do on paper. Create a result array to store digits. For each digit in num2, multiply it with every digit in num1, and add to the appropriate position in the result array. Handle carries. Convert the result array back to string.

**Algorithm Steps:**
1. Handle edge cases (if either is "0", return "0")
2. Create result array of size len(num1) + len(num2)
3. Iterate num1 from right to left (i)
4. Iterate num2 from right to left (j)
5. Multiply digits and add to result[i + j + 1]
6. Handle carry to result[i + j]
7. Convert result array to string (skip leading zeros)

**Python Solution:**
```python
def multiply(num1: str, num2: str) -> str:
    if num1 == "0" or num2 == "0":
        return "0"
    
    m, n = len(num1), len(num2)
    result = [0] * (m + n)
    
    # Multiply each digit
    for i in range(m - 1, -1, -1):
        for j in range(n - 1, -1, -1):
            digit1 = int(num1[i])
            digit2 = int(num2[j])
            
            # Multiply and add to appropriate position
            product = digit1 * digit2
            position = i + j + 1
            total = product + result[position]
            
            result[position] = total % 10
            result[i + j] += total // 10
    
    # Convert to string (skip leading zeros)
    result_str = ''.join(map(str, result))
    return result_str.lstrip('0') or '0'
```

**Time Complexity:** O(m × n)  
**Space Complexity:** O(m + n)

---

### 58. Problem #151: Reverse Words in a String
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/reverse-words-in-a-string/

**Problem Description:**
Given an input string `s`, reverse the order of the words. A word is defined as a sequence of non-space characters. The words in `s` will be separated by at least one space. Return a string of the words in reverse order concatenated by a single space.

**Examples:**
- Input: `s = "the sky is blue"` → Output: `"blue is sky the"`
- Input: `s = "  hello world  "` → Output: `"world hello"`
- Input: `s = "a good   example"` → Output: `"example good a"`

**Simple Explanation:**
Split the string by spaces (this automatically handles multiple spaces). Filter out empty strings. Reverse the list of words. Join them with a single space.

**Algorithm Steps:**
1. Split string by spaces
2. Filter out empty strings
3. Reverse the list
4. Join with single space

**Python Solution:**
```python
def reverseWords(s: str) -> str:
    # Split by spaces and filter empty strings
    words = s.split()
    # Reverse and join
    return ' '.join(reversed(words))

# Alternative: without using reversed()
def reverseWords_manual(s: str) -> str:
    words = s.split()
    left, right = 0, len(words) - 1
    
    while left < right:
        words[left], words[right] = words[right], words[left]
        left += 1
        right -= 1
    
    return ' '.join(words)
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n)

---

### 59. Problem #179: Largest Number
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/largest-number/

**Problem Description:**
Given a list of non-negative integers `nums`, arrange them such that they form the largest number and return it. Since the result may be very large, you need to return a string instead of an integer.

**Examples:**
- Input: `nums = [10,2]` → Output: `"210"`
- Input: `nums = [3,30,34,5,9]` → Output: `"9534330"`

**Simple Explanation:**
The key insight is to sort numbers using a custom comparator. For two numbers a and b, compare which concatenation is larger: ab or ba. Sort all numbers based on this comparison in descending order. Concatenate them to form the result. Handle the edge case where all numbers are 0.

**Algorithm Steps:**
1. Convert all numbers to strings
2. Sort using custom comparator:
   - Compare str(a) + str(b) vs str(b) + str(a)
   - Put the combination that gives larger value first
3. Concatenate sorted strings
4. Handle edge case: if result starts with "0", return "0"

**Python Solution:**
```python
from functools import cmp_to_key

def largestNumber(nums: list[int]) -> str:
    # Convert to strings
    nums_str = list(map(str, nums))
    
    # Custom comparator
    def compare(a, b):
        if a + b > b + a:
            return -1  # a should come first
        elif a + b < b + a:
            return 1   # b should come first
        else:
            return 0   # equal
    
    # Sort using custom comparator
    nums_str.sort(key=cmp_to_key(compare))
    
    # Concatenate
    result = ''.join(nums_str)
    
    # Handle edge case: all zeros
    return '0' if result[0] == '0' else result
```

**Time Complexity:** O(n log n × k) where k is average number length  
**Space Complexity:** O(n × k)

---

### 60. Problem #271: Encode and Decode Strings (Premium)
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/encode-and-decode-strings/

**Problem Description:**
Design an algorithm to encode a list of strings to a string. The encoded string is then sent over the network and decoded back to the original list of strings.

**Examples:**
- Input: `["Hello","World"]` → Encode to string → Decode back to `["Hello","World"]`

**Simple Explanation:**
Use length prefixes. For each string, store its length followed by a delimiter (like '#'), then the string itself. To decode, read the length, skip the delimiter, read that many characters, and repeat.

**Algorithm Steps:**
Encode:
1. For each string:
   - Append length + '#' + string to result
2. Return result

Decode:
1. Initialize position = 0
2. While position < length:
   - Read number until '#'
   - Read next 'number' characters as string
   - Add to result list
3. Return list

**Python Solution:**
```python
class Codec:
    def encode(self, strs: list[str]) -> str:
        """Encodes a list of strings to a single string."""
        encoded = ""
        for s in strs:
            encoded += str(len(s)) + "#" + s
        return encoded
    
    def decode(self, s: str) -> list[str]:
        """Decodes a single string to a list of strings."""
        result = []
        i = 0
        
        while i < len(s):
            # Find the delimiter
            j = i
            while s[j] != '#':
                j += 1
            
            # Get the length
            length = int(s[i:j])
            # Get the string
            string = s[j + 1:j + 1 + length]
            result.append(string)
            # Move to next encoded string
            i = j + 1 + length
        
        return result
```

**Time Complexity:** O(n) for both operations where n is total characters  
**Space Complexity:** O(n)

---

### 61. Problem #316: Remove Duplicate Letters
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/remove-duplicate-letters/

**Problem Description:**
Given a string `s`, remove duplicate letters so that every letter appears once and only once. You must make sure your result is the smallest in lexicographical order among all possible results.

**Examples:**
- Input: `s = "bcabc"` → Output: `"abc"`
- Input: `s = "cbacdcbc"` → Output: `"acdb"`

**Simple Explanation:**
Use a stack and greedy approach. For each character, if it's already in the stack, skip it. Otherwise, pop characters from the stack that are greater than current character AND appear later in the string. Then add current character. Track which characters are in the stack and count remaining occurrences.

**Algorithm Steps:**
1. Count occurrences of each character
2. Use a stack and a set to track characters in stack
3. For each character:
   - Decrease its remaining count
   - If already in stack, skip
   - While stack top > current char AND stack top appears later:
     - Pop from stack and set
   - Add current char to stack and set
4. Return stack as string

**Python Solution:**
```python
def removeDuplicateLetters(s: str) -> str:
    # Count occurrences
    count = {}
    for char in s:
        count[char] = count.get(char, 0) + 1
    
    stack = []
    in_stack = set()
    
    for char in s:
        count[char] -= 1
        
        if char in in_stack:
            continue
        
        # Remove larger characters that appear later
        while stack and stack[-1] > char and count[stack[-1]] > 0:
            removed = stack.pop()
            in_stack.remove(removed)
        
        stack.append(char)
        in_stack.add(char)
    
    return ''.join(stack)
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1) since alphabet is fixed size

---

## Sorting & Greedy (1 problem)

### 62. Problem #56: Merge Intervals
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/merge-intervals/

**Problem Description:**
Given an array of `intervals` where intervals[i] = [start_i, end_i], merge all overlapping intervals and return an array of the non-overlapping intervals that cover all the intervals in the input.

**Examples:**
- Input: `intervals = [[1,3],[2,6],[8,10],[15,18]]` → Output: `[[1,6],[8,10],[15,18]]`
- Input: `intervals = [[1,4],[4,5]]` → Output: `[[1,5]]`

**Simple Explanation:**
Sort intervals by start time. Use a result list to store merged intervals. For each interval, check if it overlaps with the last interval in the result. If yes, merge them by updating the end time. If no, add it as a new interval to the result.

**Algorithm Steps:**
1. Sort intervals by start time
2. Initialize result with first interval
3. For each remaining interval:
   - If it overlaps with last interval in result (start <= last_end):
     - Merge by updating last_end to max(last_end, current_end)
   - Else:
     - Add as new interval to result
4. Return result

**Python Solution:**
```python
def merge(intervals: list[list[int]]) -> list[list[int]]:
    if not intervals:
        return []
    
    # Sort by start time
    intervals.sort(key=lambda x: x[0])
    
    result = [intervals[0]]
    
    for i in range(1, len(intervals)):
        current_start, current_end = intervals[i]
        last_start, last_end = result[-1]
        
        if current_start <= last_end:
            # Overlapping - merge
            result[-1][1] = max(last_end, current_end)
        else:
            # Non-overlapping - add new interval
            result.append(intervals[i])
    
    return result
```

**Time Complexity:** O(n log n) due to sorting  
**Space Complexity:** O(n) for the result

---

## Tree Traversal (3 problems)

### 63. Problem #105: Construct Binary Tree from Preorder and Inorder Traversal
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/

**Problem Description:**
Given two integer arrays `preorder` and `inorder` where preorder is the preorder traversal and inorder is the inorder traversal of the same binary tree, construct and return the binary tree.

**Examples:**
- Input: `preorder = [3,9,20,15,7]`, `inorder = [9,3,15,20,7]` → Output: `[3,9,20,null,null,15,7]`

**Simple Explanation:**
The first element in preorder is always the root. Find this root in inorder array - elements to its left are the left subtree, elements to its right are the right subtree. Recursively build left and right subtrees. Use a hash map to quickly find root positions in inorder.

**Algorithm Steps:**
1. Create hash map: inorder value -> index
2. Recursive function with ranges:
   - Base case: if ranges are invalid, return None
   - First element of preorder range is root
   - Find root index in inorder
   - Calculate left subtree size
   - Recursively build left subtree (adjust preorder and inorder ranges)
   - Recursively build right subtree (adjust preorder and inorder ranges)
3. Return root

**Python Solution:**
```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

def buildTree(preorder: list[int], inorder: list[int]) -> TreeNode:
    # Create hash map for inorder indices
    inorder_map = {val: i for i, val in enumerate(inorder)}
    
    def build(pre_start, pre_end, in_start, in_end):
        if pre_start > pre_end:
            return None
        
        # Root is first element in preorder range
        root_val = preorder[pre_start]
        root = TreeNode(root_val)
        
        # Find root in inorder
        root_index = inorder_map[root_val]
        left_size = root_index - in_start
        
        # Build left subtree
        root.left = build(
            pre_start + 1,
            pre_start + left_size,
            in_start,
            root_index - 1
        )
        
        # Build right subtree
        root.right = build(
            pre_start + left_size + 1,
            pre_end,
            root_index + 1,
            in_end
        )
        
        return root
    
    return build(0, len(preorder) - 1, 0, len(inorder) - 1)
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n)

---

### 64. Problem #113: Path Sum II
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/path-sum-ii/

**Problem Description:**
Given the root of a binary tree and an integer `targetSum`, return all root-to-leaf paths where the sum of the node values equals targetSum. Each path should be returned as a list of node values, not node references. A leaf is a node with no children.

**Examples:**
- Input: `root = [5,4,8,11,null,13,4,7,2,null,null,5,1]`, `targetSum = 22`
- Output: `[[5,4,11,2],[5,8,4,5]]`

**Simple Explanation:**
Use DFS with backtracking. Keep track of the current path and remaining sum. When you reach a leaf node, check if remaining sum equals the leaf's value. If yes, add current path to results. Backtrack by removing the node when returning from recursion.

**Algorithm Steps:**
1. Create result list and current path list
2. DFS function:
   - Add current node to path
   - If leaf node and sum matches, add path copy to results
   - Recursively call for left child (subtract node value from target)
   - Recursively call for right child (subtract node value from target)
   - Remove current node from path (backtrack)
3. Return all paths

**Python Solution:**
```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

def pathSum(root: TreeNode, targetSum: int) -> list[list[int]]:
    result = []
    
    def dfs(node, remaining, path):
        if not node:
            return
        
        # Add current node to path
        path.append(node.val)
        
        # Check if it's a leaf and sum matches
        if not node.left and not node.right and remaining == node.val:
            result.append(path[:])  # Add copy of path
        
        # Explore children
        dfs(node.left, remaining - node.val, path)
        dfs(node.right, remaining - node.val, path)
        
        # Backtrack
        path.pop()
    
    dfs(root, targetSum, [])
    return result
```

**Time Complexity:** O(n)  
**Space Complexity:** O(h) where h is tree height

---

### 65. Problem #426: Convert Binary Search Tree to Sorted Doubly Linked List (Premium)
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/convert-binary-search-tree-to-sorted-doubly-linked-list/

**Problem Description:**
Convert a Binary Search Tree to a sorted Circular Doubly-Linked List in place. The left and right pointers in nodes should be used as previous and next pointers for the doubly linked list. The list should be arranged in the same order as an in-order traversal of the BST.

**Examples:**
- Input: `root = [4,2,5,1,3]` → Output: Circular linked list in order 1<->2<->3<->4<->5<->1

**Simple Explanation:**
Do an in-order traversal of the BST. Keep track of the previous node. For each node, connect it with the previous node bidirectionally. Track the first and last nodes to make the list circular.

**Algorithm Steps:**
1. Keep track of first and last nodes
2. In-order traversal:
   - Process left subtree
   - Connect current node:
     - If first node, mark it
     - Link with last node bidirectionally
     - Update last node
   - Process right subtree
3. Connect first and last to make circular
4. Return first node

**Python Solution:**
```python
class Node:
    def __init__(self, val, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

def treeToDoublyList(root: Node) -> Node:
    if not root:
        return None
    
    first = None
    last = None
    
    def inorder(node):
        nonlocal first, last
        
        if not node:
            return
        
        # Process left subtree
        inorder(node.left)
        
        # Process current node
        if last:
            # Link with previous node
            last.right = node
            node.left = last
        else:
            # First node
            first = node
        
        last = node
        
        # Process right subtree
        inorder(node.right)
    
    inorder(root)
    
    # Make circular
    first.left = last
    last.right = first
    
    return first
```

**Time Complexity:** O(n)  
**Space Complexity:** O(h) for recursion stack

---

## Design / Data Structure (2 problems)

### 66. Problem #706: Design HashMap
**Difficulty:** Easy  
**URL:** https://leetcode.com/problems/design-hashmap/

**Problem Description:**
Design a HashMap without using any built-in hash table libraries. Implement: `put(key, value)`, `get(key)` returns the value or -1 if not found, `remove(key)` removes the mapping.

**Examples:**
```
MyHashMap hashMap = new MyHashMap();
hashMap.put(1, 1);
hashMap.put(2, 2);
hashMap.get(1);    // returns 1
hashMap.get(3);    // returns -1
hashMap.put(2, 1); // update
hashMap.get(2);    // returns 1
hashMap.remove(2);
hashMap.get(2);    // returns -1
```

**Simple Explanation:**
Use an array of buckets (linked lists). Hash the key to find which bucket it belongs to. In each bucket, store key-value pairs. Handle collisions using chaining (multiple entries in same bucket). Use a simple hash function like key % bucket_size.

**Algorithm Steps:**
1. Create an array of buckets (size = 1000 or similar)
2. Each bucket is a list of [key, value] pairs
3. Hash function: key % size
4. put(key, value):
   - Find bucket using hash
   - Search bucket for key
   - If found, update value; else append [key, value]
5. get(key):
   - Find bucket
   - Search for key, return value or -1
6. remove(key):
   - Find bucket
   - Search and remove [key, value] pair

**Python Solution:**
```python
class MyHashMap:
    def __init__(self):
        self.size = 1000
        self.buckets = [[] for _ in range(self.size)]
    
    def _hash(self, key):
        return key % self.size
    
    def put(self, key: int, value: int) -> None:
        bucket_index = self._hash(key)
        bucket = self.buckets[bucket_index]
        
        # Check if key exists
        for i, (k, v) in enumerate(bucket):
            if k == key:
                bucket[i] = [key, value]  # Update
                return
        
        # Key doesn't exist, add it
        bucket.append([key, value])
    
    def get(self, key: int) -> int:
        bucket_index = self._hash(key)
        bucket = self.buckets[bucket_index]
        
        for k, v in bucket:
            if k == key:
                return v
        
        return -1
    
    def remove(self, key: int) -> None:
        bucket_index = self._hash(key)
        bucket = self.buckets[bucket_index]
        
        for i, (k, v) in enumerate(bucket):
            if k == key:
                bucket.pop(i)
                return
```

**Time Complexity:** O(n/k) average, O(n) worst case (n = elements, k = buckets)  
**Space Complexity:** O(k + n)

---

### 67. Problem #2502: Design Memory Allocator
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/design-memory-allocator/

**Problem Description:**
Design a memory allocator with n units of memory. Implement: `allocate(size, mID)` allocates size consecutive free units and assigns them to mID. Returns starting position or -1 if impossible. `free(mID)` frees all units allocated to mID and returns count of freed units.

**Examples:**
```
Allocator allocator = new Allocator(10);
allocator.allocate(1, 1); // returns 0
allocator.allocate(1, 2); // returns 1
allocator.allocate(1, 3); // returns 2
allocator.free(2);        // returns 1
allocator.allocate(3, 4); // returns 3
```

**Simple Explanation:**
Use an array to represent memory. Each cell stores the mID of the owner or 0 if free. For allocate, find a consecutive block of free cells. Mark them with mID. For free, find all cells with that mID and mark them as 0.

**Algorithm Steps:**
1. Create memory array of size n, initialized to 0
2. allocate(size, mID):
   - Scan memory for consecutive free cells of given size
   - If found, mark them with mID and return start position
   - If not found, return -1
3. free(mID):
   - Count and free all cells with this mID
   - Return count

**Python Solution:**
```python
class Allocator:
    def __init__(self, n: int):
        self.memory = [0] * n
        self.n = n
    
    def allocate(self, size: int, mID: int) -> int:
        # Find consecutive free blocks
        consecutive = 0
        start = 0
        
        for i in range(self.n):
            if self.memory[i] == 0:
                if consecutive == 0:
                    start = i
                consecutive += 1
                
                if consecutive == size:
                    # Found enough space, allocate it
                    for j in range(start, start + size):
                        self.memory[j] = mID
                    return start
            else:
                consecutive = 0
        
        return -1  # Not enough space
    
    def free(self, mID: int) -> int:
        count = 0
        for i in range(self.n):
            if self.memory[i] == mID:
                self.memory[i] = 0
                count += 1
        return count
```

**Time Complexity:** O(n) for both operations  
**Space Complexity:** O(n)

---

## Array Manipulation (4 problems)

### 68. Problem #2021: Brightest Position on Street (Premium)
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/brightest-position-on-street/

**Problem Description:**
You are given an array `lights` where lights[i] = [position, range] means there is a street light at position that illuminates the area from [position - range, position + range] (inclusive). Return the brightest position on the street. If there are multiple, return the smallest.

**Examples:**
- Input: `lights = [[-3,2],[1,2],[3,3]]` → Output: `-1`

**Simple Explanation:**
Use the sweep line algorithm. Create events for when lights turn on and off. Sort all events by position. Process events left to right, tracking current brightness. When brightness increases beyond maximum, update the brightest position.

**Algorithm Steps:**
1. Create events: (position, +1 for light start, -1 for light end)
2. For each light at pos with range:
   - Add event (pos - range, +1)
   - Add event (pos + range + 1, -1)
3. Sort events by position
4. Process events, tracking current brightness
5. Update brightest position when brightness exceeds max
6. Return brightest position

**Python Solution:**
```python
def brightestPosition(lights: list[list[int]]) -> int:
    events = []
    
    # Create events
    for pos, range_val in lights:
        events.append((pos - range_val, 1))      # Light starts
        events.append((pos + range_val + 1, -1))  # Light ends
    
    # Sort events
    events.sort()
    
    max_brightness = 0
    current_brightness = 0
    brightest_pos = 0
    
    for pos, change in events:
        current_brightness += change
        
        if current_brightness > max_brightness:
            max_brightness = current_brightness
            brightest_pos = pos
    
    return brightest_pos
```

**Time Complexity:** O(n log n)  
**Space Complexity:** O(n)

---

### 69. Problem #581: Shortest Unsorted Continuous Subarray
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/shortest-unsorted-continuous-subarray/

**Problem Description:**
Given an integer array `nums`, find one continuous subarray that if you only sort this subarray in ascending order, then the whole array will be sorted. Return the length of this shortest subarray.

**Examples:**
- Input: `nums = [2,6,4,8,10,9,15]` → Output: `5` (need to sort [6,4,8,10,9])
- Input: `nums = [1,2,3,4]` → Output: `0`

**Simple Explanation:**
Find the leftmost and rightmost elements that are out of place. From left, find where the increasing sequence breaks. From right, find where the decreasing sequence breaks. The elements between these boundaries need to be sorted.

**Algorithm Steps:**
1. Find the minimum and maximum elements that are out of order
2. Scan from left to find where nums[i] > min_out_of_order
3. Scan from right to find where nums[i] < max_out_of_order
4. The subarray between these positions needs sorting
5. Return the length

**Python Solution:**
```python
def findUnsortedSubarray(nums: list[int]) -> int:
    n = len(nums)
    
    # Find rightmost element that's out of order
    max_val = float('-inf')
    end = -1
    for i in range(n):
        if nums[i] < max_val:
            end = i
        max_val = max(max_val, nums[i])
    
    # Find leftmost element that's out of order
    min_val = float('inf')
    start = 0
    for i in range(n - 1, -1, -1):
        if nums[i] > min_val:
            start = i
        min_val = min(min_val, nums[i])
    
    # If end is still -1, array is already sorted
    return end - start + 1 if end != -1 else 0
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

---

### 70. Problem #959: Regions Cut By Slashes
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/regions-cut-by-slashes/

**Problem Description:**
In a grid of size n x n, each cell has a slash ('/'), backslash ('\\'), or blank space (' '). Count the number of regions formed.

**Examples:**
- Input: `grid = [" /","/ "]` → Output: `2`s
- Input: `grid = ["/\\","\\/"]` → Output: `4`

**Simple Explanation:**
Divide each cell into 4 triangular parts (top, right, bottom, left). Use Union-Find to connect parts that are not separated by slashes. Parts in the same union form one region. Count the number of unions.

**Algorithm Steps:**
1. Create Union-Find structure with n × n × 4 elements
2. For each cell:
   - Connect parts within cell based on slash type
   - Connect with adjacent cells (right and bottom)
3. Count number of distinct components
4. Return count

**Python Solution:**
```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.components = n
    
    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    
    def union(self, x, y):
        root_x, root_y = self.find(x), self.find(y)
        if root_x != root_y:
            self.parent[root_x] = root_y
            self.components -= 1

def regionsBySlashes(grid: list[str]) -> int:
    n = len(grid)
    uf = UnionFind(n * n * 4)
    
    for r in range(n):
        for c in range(n):
            # Each cell has 4 parts: 0=top, 1=right, 2=bottom, 3=left
            base = 4 * (r * n + c)
            
            # Connect parts within this cell
            if grid[r][c] == ' ':
                # No slash: all parts connected
                uf.union(base + 0, base + 1)
                uf.union(base + 1, base + 2)
                uf.union(base + 2, base + 3)
            elif grid[r][c] == '/':
                # /: connect top-left and bottom-right
                uf.union(base + 0, base + 3)
                uf.union(base + 1, base + 2)
            else:  # '\\'
                # \: connect top-right and bottom-left
                uf.union(base + 0, base + 1)
                uf.union(base + 2, base + 3)
            
            # Connect with right neighbor
            if c + 1 < n:
                uf.union(base + 1, base + 4 + 3)
            
            # Connect with bottom neighbor
            if r + 1 < n:
                uf.union(base + 2, base + 4 * n + 0)
    
    return uf.components
```

**Time Complexity:** O(n² × α(n)) where α is inverse Ackermann  
**Space Complexity:** O(n²)

---

### 71. Problem #3071: Minimum Operations to Write the Letter Y on a Grid
**Difficulty:** Medium  
**URL:** https://leetcode.com/problems/minimum-operations-to-write-the-letter-y-on-a-grid/

**Problem Description:**
You are given an n × n grid where each cell has a value 0, 1, or 2. You need to change the grid so cells forming the letter Y have one value and all other cells have a different value. Return minimum operations needed.

**Examples:**
- Input: `grid = [[1,2,2],[1,1,0],[0,1,0]]` → Output: `3`

**Simple Explanation:**
Identify which cells form the Y pattern. These are the diagonals meeting at center and the vertical line from center to bottom. Count frequency of each value (0, 1, 2) in Y cells and non-Y cells. Try all combinations of values for Y and non-Y, calculate operations needed for each.

**Algorithm Steps:**
1. Identify Y cell positions
2. Count value frequencies in Y cells and non-Y cells
3. Try all 9 combinations (3 values for Y × 3 values for non-Y)
4. For each combination:
   - Calculate operations = (Y cells that need change) + (non-Y cells that need change)
5. Return minimum operations

**Python Solution:**
```python
def minimumOperationsToWriteY(grid: list[list[int]]) -> int:
    n = len(grid)
    mid = n // 2
    
    # Identify Y cells
    y_cells = set()
    
    # Top-left to center diagonal
    for i in range(mid + 1):
        y_cells.add((i, i))
    
    # Top-right to center diagonal
    for i in range(mid + 1):
        y_cells.add((i, n - 1 - i))
    
    # Vertical line from center to bottom
    for i in range(mid, n):
        y_cells.add((i, mid))
    
    # Count frequencies
    y_count = [0, 0, 0]
    non_y_count = [0, 0, 0]
    
    for r in range(n):
        for c in range(n):
            if (r, c) in y_cells:
                y_count[grid[r][c]] += 1
            else:
                non_y_count[grid[r][c]] += 1
    
    # Try all combinations
    min_ops = float('inf')
    for y_val in range(3):
        for non_y_val in range(3):
            if y_val != non_y_val:
                # Operations needed
                y_ops = sum(y_count) - y_count[y_val]
                non_y_ops = sum(non_y_count) - non_y_count[non_y_val]
                min_ops = min(min_ops, y_ops + non_y_ops)
    
    return min_ops
```

**Time Complexity:** O(n²)  
**Space Complexity:** O(n²)

---

## Additional Problem (Custom)

### 72. Maximum Square Area in Buildings
**Difficulty:** Medium  
**Similar to:** https://leetcode.com/problems/largest-rectangle-in-histogram/ (Problem #84)

**Problem Description:**
Given an array of building heights, find the maximum area of a square that can fit between buildings. The square's width is constrained by consecutive buildings, and its height cannot exceed the minimum building height in that range.

**Examples:**
- Input: `heights = [2,1,5,6,2,3]` → Output: `4` (square 2×2 between positions 3-4)
- Input: `heights = [2,4]` → Output: `4` (square 2×2)

**Simple Explanation:**
This is similar to largest rectangle in histogram. Use a monotonic stack to track buildings. For each building, pop taller buildings from stack and calculate possible square areas. A square has equal width and height, so area = min(width, height)².

**Algorithm Steps:**
1. Create a stack to store indices
2. For each building:
   - While stack is not empty and current height < stack top height:
     - Pop index and calculate width
     - Height = popped building's height
     - Square side = min(width, height)
     - Update maximum square area
   - Push current index
3. Process remaining buildings in stack
4. Return maximum area

**Python Solution:**
```python
def maxSquareArea(heights: list[int]) -> int:
    stack = []
    max_area = 0
    
    for i, height in enumerate(heights):
        while stack and heights[stack[-1]] > height:
            h_index = stack.pop()
            h = heights[h_index]
            # Width is distance to current position
            width = i if not stack else i - stack[-1] - 1
            # Square side is minimum of width and height
            side = min(width, h)
            max_area = max(max_area, side * side)
        
        stack.append(i)
    
    # Process remaining buildings
    while stack:
        h_index = stack.pop()
        h = heights[h_index]
        width = len(heights) if not stack else len(heights) - stack[-1] - 1
        side = min(width, h)
        max_area = max(max_area, side * side)
    
    return max_area
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n)