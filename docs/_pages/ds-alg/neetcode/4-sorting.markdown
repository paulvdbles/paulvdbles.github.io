# Sorting
## 10 - Insertion Sort

### Core Concept
Given: `[2, 3, 4, 1, 6]` → Want: `[1, 2, 3, 4, 6]`

**Key Insight:** Build a sorted array incrementally by inserting each element into its correct position within the already-sorted portion.

### How It Works (Two-Pointer Approach)
1. **Treat first element as "sorted"** (array of size 1 is always sorted)
2. **For each remaining element:**
    - `i` pointer: current element to insert
    - `j` pointer: starts at `i-1` (left neighbor)
    - While `arr[j] > arr[j+1]` and `j >= 0`:
        - Swap them
        - Move `j` left by 1
    - Stop when element finds its correct position

### Step-by-Step Example
```
[2, 3, 4, 1, 6]
 ^sorted

[2, 3, 4, 1, 6]  (3 > 2? No, stay)
 ^^^^sorted

[2, 3, 4, 1, 6]  (4 > 3? No, stay)
 ^^^^^^^sorted

[2, 3, 1, 4, 6]  (1 < 4? Swap)
[2, 1, 3, 4, 6]  (1 < 3? Swap)
[1, 2, 3, 4, 6]  (1 < 2? Swap, j < 0, stop)
 ^^^^^^^^^^sorted

[1, 2, 3, 4, 6]  (6 > 4? No, stay)
 ^^^^^^^^^^^^^sorted ✓
```

### Stability
**Stable algorithm**: Equal elements maintain their original relative order
- Example: `[7a, 3, 7b]` → `[3, 7a, 7b]` (7a still before 7b)
- Why insertion sort is stable: We only swap when `arr[j+1] < arr[j]` (not ≤)

### Complexity Analysis

**Time Complexity:**
- **Best Case O(n)**: Already sorted array
    - Inner while loop never executes (no swaps needed)
    - Just one comparison per element

- **Worst Case O(n²)**: Reverse sorted array
    - Element at position `i` needs `i` swaps to reach front
    - Total operations: 1 + 2 + 3 + ... + (n-1) = n(n-1)/2 = O(n²)

- **Average Case O(n²)**: Random array
    - On average, each element moves halfway through sorted portion
    - Still results in O(n²) operations

**Space Complexity: O(1)**
- In-place sorting (only uses a constant amount of extra variables)
- No additional data structures needed

### Key Implementation Points
```python
def insertionSort(arr):
    for i in range(1, len(arr)):  # Start at index 1
        j = i - 1
        while j >= 0 and arr[j+1] < arr[j]:  # Find insertion point
            arr[j], arr[j+1] = arr[j+1], arr[j]  # Swap
            j -= 1
    return arr
```

### When to Use Insertion Sort
✅ **Good for:**
- Small datasets (< 50 elements)
- Nearly sorted data
- Online algorithms (sorting data as it arrives)
- Simple implementation needed

❌ **Avoid for:**
- Large datasets (use quicksort, mergesort, or heapsort)
- Reverse sorted or highly unsorted data