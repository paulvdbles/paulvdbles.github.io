# Arrays
## 1 - RAM
Data structure: way of structuring data efficiently inside of RAM

RAM: where all our variables are stored
- RAM is measured in bytes
- 1 byte = 8 bits
- Bit = position can store digit of 0 or 1
- RAM can store values at address

Example: an array of integers  1, 3, 5
How can we store integer 1? Store as byte: 00000001
Integer can be represented as 4 bytes (32 bits) in memory


Array is always contiguous (nothing between 1, 3, 5)

The address increments 4 every time: each value takes 4 bytes to store

```
Array of integers                    RAM  
┌───┬───┬───┐                  ┌───┬───┬───┬───┬───┐
│ 1 │ 3 │ 5 │            Value │   │ 1 │ 3 │ 5 │   │
└───┴───┴───┘                  ├───┼───┼───┼───┼───┤
                       Address │   │$0 │$4 │$8 │   │
                               └───┴───┴───┴───┴───┘
```

## 2 - Static Arrays
Most common operations on data structure: reading and writing

Read first value? go to first address

We use indexes to access value: 
first value always 0, next 1, 2

Index 0 points to first address
We can map all indexes to address

Read: O(1)
In the worst case it's an instant operation, it happens at constant time

Static array is fixed size
Suppose we want to add a 4th value to 3 size array
We don't know if the next spot in RAM is available to us: could be another array stored there
Array needs to be contiguous

Python offers dynamic array as default

Write: O(1)

Insert value in the end is always efficient
Insert value in the middle or beginning? Not efficient
Overwrite is easy, but shifting not: takes more than 1 operation
In the worst case move all values in the array: O(N)

n = variable, here length of array

We don't want to go over every possible scenario, generalize to worst case

Removing value at index: also in the worst case move all elements

- Read or write i-th element: O(1)
- Insert / remove end: O(1)
- Insert / remove middle: O(n)





