# Algorithm Patterns

Common patterns for solving coding problems. Recognize the pattern, apply the template.

---

## Pattern Recognition

| If You See... | Consider... |
|---------------|-------------|
| "Top K" or "K largest/smallest" | Heap |
| "Sorted array" | Binary Search, Two Pointers |
| "All permutations/subsets" | Backtracking |
| "Tree/graph traversal" | BFS/DFS |
| "Optimal substructure" | Dynamic Programming |
| "Contiguous subarray" | Sliding Window |
| "Linked list cycle" | Fast/Slow Pointers |
| "String matching" | Two Pointers, Hashing |
| "Intervals" | Sorting + Greedy |
| "Matrix" | BFS/DFS, DP |

---

## Two Pointers

### When to Use
- Sorted array problems
- Finding pairs with a target sum
- Removing duplicates in-place
- Palindrome checking

### Template
```python
def two_pointers(arr):
    left, right = 0, len(arr) - 1

    while left < right:
        # Process current pair
        current = arr[left] + arr[right]

        if current == target:
            return [left, right]
        elif current < target:
            left += 1
        else:
            right -= 1

    return None
```

### Variations
- **Same direction**: Fast/slow for cycle detection
- **Opposite direction**: Converging for two-sum
- **Three pointers**: 3Sum problem

---

## Sliding Window

### When to Use
- Contiguous subarray/substring problems
- "Maximum/minimum of size K"
- "Longest substring with property X"

### Fixed Size Template
```python
def fixed_window(arr, k):
    window_sum = sum(arr[:k])
    max_sum = window_sum

    for i in range(k, len(arr)):
        window_sum += arr[i] - arr[i - k]  # Slide: add new, remove old
        max_sum = max(max_sum, window_sum)

    return max_sum
```

### Variable Size Template
```python
def variable_window(s):
    left = 0
    window = {}  # or set, counter, etc.
    result = 0

    for right in range(len(s)):
        # Expand: add s[right] to window
        window[s[right]] = window.get(s[right], 0) + 1

        # Contract: while window invalid
        while not valid(window):
            window[s[left]] -= 1
            if window[s[left]] == 0:
                del window[s[left]]
            left += 1

        # Update result
        result = max(result, right - left + 1)

    return result
```

---

## Binary Search

### When to Use
- Sorted array search
- Finding boundaries (first/last occurrence)
- Search space reduction
- "Minimum/maximum that satisfies condition"

### Standard Template
```python
def binary_search(arr, target):
    left, right = 0, len(arr) - 1

    while left <= right:
        mid = left + (right - left) // 2

        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1

    return -1
```

### Left Boundary (First Occurrence)
```python
def left_boundary(arr, target):
    left, right = 0, len(arr)

    while left < right:
        mid = left + (right - left) // 2

        if arr[mid] < target:
            left = mid + 1
        else:
            right = mid

    return left
```

### Search Space Template
```python
def binary_search_condition(lo, hi, condition):
    while lo < hi:
        mid = lo + (hi - lo) // 2

        if condition(mid):
            hi = mid
        else:
            lo = mid + 1

    return lo
```

---

## BFS (Breadth-First Search)

### When to Use
- Shortest path in unweighted graph
- Level-order traversal
- Finding nearest/minimum steps

### Template
```python
from collections import deque

def bfs(graph, start):
    queue = deque([start])
    visited = {start}
    level = 0

    while queue:
        level_size = len(queue)

        for _ in range(level_size):
            node = queue.popleft()

            # Process node

            for neighbor in graph[node]:
                if neighbor not in visited:
                    visited.add(neighbor)
                    queue.append(neighbor)

        level += 1

    return level
```

---

## DFS (Depth-First Search)

### When to Use
- Path finding
- Cycle detection
- Connected components
- Tree traversals

### Recursive Template
```python
def dfs(graph, node, visited):
    if node in visited:
        return

    visited.add(node)

    # Process node (pre-order)

    for neighbor in graph[node]:
        dfs(graph, neighbor, visited)

    # Process node (post-order)
```

### Iterative Template
```python
def dfs_iterative(graph, start):
    stack = [start]
    visited = set()

    while stack:
        node = stack.pop()

        if node in visited:
            continue

        visited.add(node)

        # Process node

        for neighbor in graph[node]:
            if neighbor not in visited:
                stack.append(neighbor)
```

---

## Backtracking

### When to Use
- Generate all permutations/combinations
- Solve puzzles (Sudoku, N-Queens)
- Path finding with constraints

### Template
```python
def backtrack(path, choices):
    if is_solution(path):
        result.append(path[:])  # Copy!
        return

    for choice in choices:
        if is_valid(choice, path):
            path.append(choice)          # Make choice
            backtrack(path, choices)      # Recurse
            path.pop()                    # Undo choice
```

### Permutations Example
```python
def permutations(nums):
    result = []

    def backtrack(path, remaining):
        if not remaining:
            result.append(path[:])
            return

        for i in range(len(remaining)):
            path.append(remaining[i])
            backtrack(path, remaining[:i] + remaining[i+1:])
            path.pop()

    backtrack([], nums)
    return result
```

---

## Dynamic Programming

### When to Use
- Optimal substructure (optimal solution from optimal sub-solutions)
- Overlapping subproblems
- "Count ways", "Min/max cost", "Is it possible"

### Approach
1. Define state: What variables define a subproblem?
2. Define recurrence: How does current state relate to previous?
3. Define base case: What are the trivial cases?
4. Define order: Which subproblems solve first?

### Top-Down Template (Memoization)
```python
def dp_topdown(n):
    memo = {}

    def helper(state):
        if state in memo:
            return memo[state]

        if is_base_case(state):
            return base_value

        result = combine(helper(prev_state1), helper(prev_state2))
        memo[state] = result
        return result

    return helper(initial_state)
```

### Bottom-Up Template (Tabulation)
```python
def dp_bottomup(n):
    dp = [0] * (n + 1)

    # Base cases
    dp[0] = base_value_0
    dp[1] = base_value_1

    # Fill table
    for i in range(2, n + 1):
        dp[i] = combine(dp[i-1], dp[i-2])

    return dp[n]
```

### Common DP Problems
- Fibonacci: `dp[i] = dp[i-1] + dp[i-2]`
- Climbing stairs: Same as Fibonacci
- Coin change: `dp[amount] = min(dp[amount - coin] + 1)`
- Longest subsequence: 2D table
- Knapsack: Include/exclude decision

---

## Heap / Priority Queue

### When to Use
- Top K elements
- Kth largest/smallest
- Merge K sorted lists
- Continuous median

### Template
```python
import heapq

# Min heap (default)
heap = []
heapq.heappush(heap, value)
smallest = heapq.heappop(heap)

# Max heap (negate values)
heapq.heappush(heap, -value)
largest = -heapq.heappop(heap)

# K largest elements
k_largest = heapq.nlargest(k, arr)
k_smallest = heapq.nsmallest(k, arr)
```

### Top K Pattern
```python
def top_k(arr, k):
    heap = []

    for num in arr:
        heapq.heappush(heap, num)
        if len(heap) > k:
            heapq.heappop(heap)  # Remove smallest

    return heap  # K largest remain
```

---

## Intervals

### When to Use
- Merge overlapping intervals
- Insert interval
- Meeting rooms
- Interval scheduling

### Template
```python
def merge_intervals(intervals):
    intervals.sort(key=lambda x: x[0])  # Sort by start
    result = []

    for interval in intervals:
        if result and interval[0] <= result[-1][1]:
            # Overlapping: merge
            result[-1][1] = max(result[-1][1], interval[1])
        else:
            result.append(interval)

    return result
```

---

## Quick Reference

### Time Complexity Goals
| Problem Type | Target |
|--------------|--------|
| Array search (sorted) | O(log n) |
| Array traversal | O(n) |
| Sorting | O(n log n) |
| Substring problems | O(n) or O(n²) |
| Graph traversal | O(V + E) |
| DP | O(n) to O(n²) |

### Space Complexity
- In-place preferred when possible
- O(n) for auxiliary structures
- Recursion adds O(h) stack space (h = depth)
