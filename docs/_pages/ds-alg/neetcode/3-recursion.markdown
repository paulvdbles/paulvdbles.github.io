# Recursion
##  8 - Factorial (one-branch recursion)
Factorial: n! = n × (n-1) × (n-2) × ... × 1
Example: 5! = 5 × 4 × 3 × 2 × 1 = 120

The Recursive Insight
- 5! = 5 × 4!
- 4! = 4 × 3!
- ...keeps breaking down...
- 1! = 1 (base case - where we stop)

Recursive Version
```
def factorial(n):
    if n <= 1:      # Base case: stop here!
        return 1
    return n * factorial(n - 1)  # Recursive call
```

Iterative Version
```
def factorial(n):
    result = 1
    while n > 1:
        result = result * n
        n -= 1
    return result
```

- Time: O(n) - both versions
- Space: Recursive uses O(n) memory (call stack), iterative uses O(1)
- Trade-off: Recursion is cleaner to read here but uses more memory

##  9 - Fibonacci sequence (two-branch recursion)
Formula: F(n) = F(n-1) + F(n-2)  

Base cases:
- F(0) = 0
- F(1) = 1

Sequence: 0, 1, 1, 2, 3, 5, 8, 13, 21...

Each number = sum of previous two

Recursive Version (Simple but Slow)
```python
def fib(n):
    if n <= 1:      # Base cases: F(0)=0, F(1)=1
        return n
    return fib(n-1) + fib(n-2)  # Two recursive calls!
```

Why It's Inefficient
- Each call spawns TWO more calls = creates a tree
- Example: F(5) calls F(4) and F(3), F(4) calls F(3) and F(2)...
- We recalculate the same values multiple times
- Time Complexity: O(2^n) - exponential (terrible)
- Space Complexity: O(n) - max depth of call stack

The Tree Visual
        F(5)
       /    \
    F(4)    F(3)
    /  \    /  \
  F(3) F(2) F(2) F(1)
  ...continues...

Quick Comparison
- Iterative loop: O(n) time, O(1) space - much better
- This recursive: O(2^n) time - gets unusable fast
- Lesson: Not all problems suit recursion

Note: For real code, use iteration or memoization. This is just for understanding recursion.

















