# 🧠 Dynamic Programming (DP) Learnings

---

## 📌 How to Identify a DP Problem

Ask yourself:
- Are there **multiple choices** at each step?
- Are we trying to compute an **optimal result** (e.g., `min`, `max`, `longest`, `shortest`, etc.)?
- Are there **overlapping subproblems** that repeat?

If **yes**, it’s likely a DP problem.

---

## 🧠 What is Dynamic Programming?

At its core, DP is:
- **Recursion** → break the problem into smaller subproblems.
- **Memoization** → store solutions to subproblems to avoid recomputation.

---

## 🛠️ How to Write DP Code

1. **Define the base case**  
   - Start from the smallest **valid input** where the result is obvious.

2. **Create a choice diagram**  
   - Visualize the choices you can make at each step.
   - Helps derive the recursive relation.

3. **Write the recurrence relation**  
   - Use the choices to build the recursive function.

4. **Dry-run the logic**  
   - Use example inputs to manually test edge cases and validate the base case.

---

## 🧾 Memoization Strategy

Instead of recalculating every time:
- **Store the result** of each recursive call.
- **Before calling the function**, check if it’s already stored.

---

## 📐 How to Determine the Shape of the DP Table

- Look at the parameters of your recursive function.
- The **number of changing parameters** determines the dimensions of the storage (e.g., 1D, 2D, etc.).
- For example:
  - `helper(i)` → 1D `dp[i]`
  - `helper(i, j)` → 2D `dp[i][j]`

---

### ✅ Summary

> A well-formed recurrence and memoization structure = 70% of the problem solved.

Once the recursion + memoization pattern is clear, coding becomes mechanical.
