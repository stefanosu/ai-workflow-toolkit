# Mental Models for Problem Solving

Frameworks for approaching new problems systematically.

---

## The UMPIRE Method

A structured approach for coding interviews and new problems.

```
U - Understand the problem
M - Match to known patterns
P - Plan your approach
I - Implement the solution
R - Review your code
E - Evaluate complexity
```

### U - Understand
- Restate the problem in your own words
- Clarify inputs and outputs
- Ask about edge cases
- Write example inputs/outputs

**Questions to ask:**
- What are the input types and ranges?
- What should I return for empty/null input?
- Are there duplicates? Are they sorted?
- What's the expected output for [edge case]?

### M - Match
- Does this match a known pattern?
- Have I solved something similar?
- What data structures fit?

**Pattern matching:**
- Sorted array → Binary Search, Two Pointers
- Optimization → DP, Greedy
- Shortest path → BFS
- All possibilities → Backtracking

### P - Plan
- Write pseudocode before code
- Identify the algorithm
- Walk through with an example
- Verify approach handles edge cases

### I - Implement
- Translate plan to code
- Use meaningful variable names
- Handle edge cases first
- Comment non-obvious parts

### R - Review
- Walk through code with an example
- Check edge cases
- Look for off-by-one errors
- Verify return statements

### E - Evaluate
- Time complexity: How does runtime scale?
- Space complexity: Extra memory used?
- Can you optimize?

---

## Problem Decomposition

Break complex problems into smaller pieces.

### The Approach
```
1. Solve the simplest version first
2. Add one constraint at a time
3. Handle edge cases last
```

### Example: "Find K closest points to origin"

**Simplest version:**
- Find the closest point (K=1)
- Just calculate distances and find minimum

**Add complexity:**
- Now find K closest
- Use a heap to track K smallest

**Handle edges:**
- K larger than array?
- Points at same distance?

---

## Working Backwards

Start from the desired output and reason backward.

### When to Use
- The output format gives hints
- Forward reasoning is stuck
- Optimization problems

### Example: "Minimum operations to reach target"

**Forward thinking:** Try all operations from start
**Backward thinking:** What operation leads to target? Work back to start.

Sometimes the reverse is simpler:
- Instead of "add 1 or multiply by 2 to reach N"
- Think: "subtract 1 or divide by 2 from N to reach 1"

---

## The Brute Force First Principle

Always start with brute force, then optimize.

### Why
1. Ensures you understand the problem
2. Gives a baseline to compare
3. Sometimes brute force is acceptable
4. Optimization is easier with working code

### The Process
```
1. Write brute force solution
2. Identify the bottleneck
3. Apply optimization technique
4. Verify correctness preserved
```

### Common Optimizations
| Brute Force | Optimization |
|-------------|--------------|
| Nested loops O(n²) | Hash map O(n) |
| Check all pairs | Two pointers |
| Recalculate | Sliding window |
| Recompute subproblems | Memoization |
| Linear search | Binary search |

---

## Constraint Analysis

Use constraints to guide solution approach.

### Array Size Hints
| Constraint | Suggests |
|------------|----------|
| n ≤ 10 | O(n!) backtracking OK |
| n ≤ 20 | O(2^n) subset enumeration OK |
| n ≤ 100 | O(n³) OK |
| n ≤ 1000 | O(n²) OK |
| n ≤ 10^5 | O(n log n) or O(n) needed |
| n ≤ 10^6 | O(n) or O(log n) needed |

### Value Range Hints
- Small range (1-100) → Counting sort, array as map
- Large range → Hash map
- Binary values → Bit manipulation

---

## State Machine Thinking

Model problems as state transitions.

### When to Use
- String parsing
- Multi-step processes
- Problems with rules about what follows what

### Template
```
Define states: [S1, S2, S3]
Define transitions: S1 --condition--> S2
Start state: S1
End states: [S3]
```

### Example: Valid Number
```
States: START, INTEGER, DECIMAL, EXPONENT, END
Transitions:
  START --digit--> INTEGER
  START --dot--> DECIMAL
  INTEGER --dot--> DECIMAL
  INTEGER --e--> EXPONENT
  ...
```

---

## Invariant Thinking

Maintain a property that's always true.

### How It Works
1. Define an invariant (property that must hold)
2. Initialize to make invariant true
3. Each step preserves the invariant
4. Invariant guarantees correct final state

### Example: Two Pointers for Sorted Array Sum
```
Invariant: If answer exists, it's between left and right pointers

Initialize: left = 0, right = n-1 (answer in range)

Each step:
- If sum < target: left++ (smaller values excluded, invariant holds)
- If sum > target: right-- (larger values excluded, invariant holds)
- If equal: found answer

Termination: left >= right means no answer exists
```

---

## The Test-Driven Approach

Write tests before coding to clarify requirements.

### Process
```
1. Write test for simplest case
2. Write test for normal case
3. Write test for edge cases
4. Implement to pass tests
5. Refactor
```

### Edge Cases to Always Consider
- Empty input
- Single element
- Two elements
- All same elements
- Already sorted / reverse sorted
- Maximum values
- Negative numbers (if applicable)

---

## Rubber Duck Debugging

Explain your approach out loud.

### The Process
1. State what the code should do
2. Walk through line by line
3. Say what each variable holds
4. Explain why each step is correct

### Why It Works
- Forces sequential thinking
- Exposes assumptions
- Reveals gaps in understanding
- Catches "I assumed this works" bugs

---

## Time-Space Tradeoff

Decide what to trade.

### Options
| Trade | Example |
|-------|---------|
| Space for time | Memoization, hash maps |
| Time for space | Recompute vs store |
| Precompute for query | Build index once, query fast |

### Questions to Ask
- Is memory limited?
- Are queries repeated?
- Can I precompute?
- Is the tradeoff worth it?

---

## The Checklist

Before submitting any solution:

### Correctness
- [ ] Handles empty input?
- [ ] Handles single element?
- [ ] Handles duplicates?
- [ ] Handles negative numbers?
- [ ] Handles maximum values?
- [ ] Returns correct type?

### Code Quality
- [ ] Variable names meaningful?
- [ ] No redundant code?
- [ ] Comments for non-obvious logic?
- [ ] Consistent style?

### Complexity
- [ ] Know the time complexity?
- [ ] Know the space complexity?
- [ ] Is this the best possible?

---

## Learning Loop

How to improve over time.

```
┌─────────────────────────────────────────┐
│                                         │
│   Solve → Review → Identify Gap →       │
│      Study Pattern → Practice →         │
│         Solve Similar Problems          │
│                                         │
└─────────────────────────────────────────┘
```

### After Each Problem
1. What pattern was this?
2. What was my first instinct? Was it right?
3. Where did I get stuck?
4. What would I do differently?

### Retention
- Revisit problems after 1 day, 1 week, 1 month
- Explain solutions without looking at code
- Teach the pattern to someone (or rubber duck)
