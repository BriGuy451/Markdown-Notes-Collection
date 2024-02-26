# Heap

A heap is a binary tree with 3 main characteristics:

1. **It's complete, filled in from left to right, last row need not be full.**
2. **Implemented with Array (Usually)**
3. **Each node satisfies heap condition (node must be larger or equal to children)**

The common use for a heap is to be used as a Priority Queue, with insertion and deletion times of O(logN), better than insertion time for the Ordered Array implementation. Can also be used to implement a sorting algorithm (Heapsort).

The heap can not be traversed easily like the BST, because of the organizing principle (heap condition) is not as strong/strict as the BST's organizing principle. So the heap is considered **weakly ordered**

## Heap Array Indices
For a node at index x
| Relation | Pattern |
| ----------- | ----------- |
| Parent | (x - 1) / 2 |
| Left Child | (2 * x) + 1 |
| Right Child | (2 * x) + 2 |

## Removal **O(logN)**
The process for removing a node from the heap.
1. Remove the root.
2. Move last node to the root.
3. Trickle down the new root.

Trickle Down is the process of taking an unordered node and comparing it to its children and moving it until it is below a larger node and above a smaller node.

```python
def trickleDown(index):
  largerChild = None
  top = heapArray[index] # save root
  while index < currentSize/2: # while node has at least one child
    leftChild = (2 * index) + 1
    rightChild = (2 * index) + 2

    # find larger child
    if rightChild < currentSize and heapArray[leftChild] < heapArray[rightChild]: 
      largerChild = rightChild
    else:
      largerChild = leftChild

    # top >= largerChild?
    if top >= heapArray[largerChild]:
      break
    
    heapArray[index] = heapArray[largerChild] # Shift child up
    index = largerChild # go down
  
  heapArray[index] = top # index <-- initial root
```

## Insertion **O(logN)**
Insertion is simpler than removal, the steps to insert are:
1. Place value in first open position
2. Trickle Up the node in that

Trickling Up is similar to the process of trickling down, but the node is compared only to its parent to see if it is larger than it and moved upward if need be.

```python
def trickleUp(index):
  parent = (index - 1) / 2
  bottom = heapArray[index]

  while index > 0 and heapArray[parent] < bottom:
    heapArray[index] = heapArray[parent] # move node down
    index = parent # move index up
    parent = (parent - 1) / 2
  
  heapArray[index] = bottom
```

## Heapsort **O(N*logN)**
The heap data structure can be used as a simple and very efficient sorting algorithm called heapsort. The main idea is to insert the unordered items into a heap using the normal insert() routine and repeat application of the remove() routine which will then remove the items in sorted order.