#  Backtracking vs Subarray Cheat Sheet

Use this as a quick reference when solving array problems.

---

## 🧭 Quick Decision Guide

|  Problem Asks For...                                  |  Technique              |  Tools / Strategy                          |
|---------------------------------------------------------|---------------------------|----------------------------------------------|
| All combinations (e.g. choose `k` out of `n`)           | **Backtracking**          | `start` index + recursive choice             |
| All subsets (power set, no duplicates)                  | **Backtracking**          | Sort + skip duplicates                       |
| All permutations (all orderings)                        | **Backtracking**          | `used[]` array or swapping                   |
| Partitioning string (e.g. palindrome partitioning)      | **Backtracking**          | DFS on substrings                            |
| Generate all valid expressions / strings (e.g. `()`)    | **Backtracking**          | Recursive rules with constraints             |
| Maze / Sudoku / N-Queens                                | **Backtracking**          | Try moves → check → backtrack                |
| Longest/shortest subarray that satisfies condition      | **Sliding Window**        | Expand/shrink window                         |
| All contiguous subarrays                                | **Nested Loops**          | `for i` + `for j`                            |
| Subarray sum queries (multiple)                         | **Prefix Sum**            | `prefix[j] - prefix[i]`                      |
| Maximum subarray sum                                    | **Kadane’s Algorithm**    | `cur = max(num, cur+num)`                   |
| Max/min in window of size `k`                           | **Monotonic Queue (Deque)**| Maintain max/min in O(n)                    |
| Count subarrays with given sum                          | **Prefix Sum + HashMap**  | Cumulative sum → count with hashmap          |
| Find subarray length satisfying a condition             | **Binary Search on Answer**| Combine with prefix / sliding window        |

---

##  Core Technique Templates

###  Backtracking (Subsets, Combinations, etc.)
```python
def backtrack(start, path):
    if condition:
        res.append(path[:])
    for i in range(start, len(nums)):
        if i > start and nums[i] == nums[i-1]:  # Skip duplicates
            continue
        path.append(nums[i])
        backtrack(i + 1, path)
        path.pop()
```

---

###  Sliding Window (Variable Size)
```python
start = 0
for end in range(len(nums)):
    window_sum += nums[end]
    while window_sum >= target:
        min_len = min(min_len, end - start + 1)
        window_sum -= nums[start]
        start += 1
```

---

###  Prefix Sum Array
```python
prefix = [0]
for num in nums:
    prefix.append(prefix[-1] + num)
# subarray sum = prefix[j] - prefix[i]
```

---

###  Kadane’s Algorithm (Max Subarray Sum)
```python
cur = max_sum = nums[0]
for num in nums[1:]:
    cur = max(num, cur + num)
    max_sum = max(max_sum, cur)
```

---
