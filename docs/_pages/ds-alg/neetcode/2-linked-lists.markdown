# Linked Lists
## 5 - Singly Linked Lists
Linked list is made of list nodes

List node has:
- value of list node
- link to next node (or null)

listnode1.next = listnode2

How it works under the hood: list node 2 has address in memory, we use this for pointer
In most languages we don't need address/pointer: just reference to object

How to loop through linked list?
cur = ListNode1
while (cur != null)
    cur = cur.next

Going through entire list: O(n)

Linked list is similar to array:
- We keep track of head and tail of linked list. If head and tail point to the same you know size is 1
- What if we want to add a new node to the list?
  - add to the end: use tail pointer, O(1)
  - add to start: use head pointer, O(1)
  - Remove node from linked list: always O(1) (we don't have to shift everything over like array)
  - if we have pointer to previous node, we set the pointer of this node to the next node of the node we want to remove
  - if we want to delete element somewhere in the middle (we have no pointer) we first need to go through the list, then it takes O(n)

## 6 - Doubly Linked Lists
We have two pointers

List node contains
- previous node
- value
- next node

First node has no previous node: set to null
Indicates beginning of list

Head and tail pointer

Add new node to end? O(1)
- Take next pointer of current tail and assign to new node
- Set previous pointer of new node to current tail
- Update tail

Remove a node, remove last O(1):
- Assume we only have pointer to tail, we dont need another pointer to previous node, tail already has it
- Go to previous node, point its next to null

Stacks can also be implemented using linked lists: it fits te requirements

Downside of linked lists: we can't arbitrary access first second third element
Accessing random element: O(n)


| Operation | Arrays - Big-O Time | LinkedLists - Big-O Time |
|-----------|-------------------|------------------------|
| Access i-th Element | O(1) | O(n) |
| Insert / Remove end | O(1) | O(1) |
| Insert / Remove middle | O(n) | O(1) |

Note: For linked lists, the O(1) insertion/removal in the middle assumes you already have a reference to the node. If you need to find the position first, it would be O(n).

## 7 - Queues
Similar to stack but FIFO

Operations:
- Enqueue: add to end O(1)
- Dequeue: remove from beginning O(1)

We can implement this with linked lists. Use head tail pointers
Add list node to start
Add another? current.next 
Remove? update current to current.next

Dequeue is very efficient, cant do that with arrays

You can technically implement queues with arrays but its complicated
