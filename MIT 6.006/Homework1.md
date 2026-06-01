Problem 1-1: Asymptotic Ordering

For each part, compare growth rates using standard rules:

Useful facts
loglogn≪logn≪(logn)^c≪n^ϵ≪n^c≪c^n≪n!≪n^n
Stirling:
n!≈sqrt(2πn)(n/e)^n
Constants don't matter in asymptotic notation.

How to solve

For every pair:

Rewrite using log identities.
Remove constant factors.
Compare dominant terms.
Group functions that are Θ(⋅) of each other using braces.

Problem 1-2: Using Sequence Operations

You have:

insert_at(i,x) = O(logn)
delete_at(i) = O(logn)

Target: O(klogn).

(a) reverse(D,i,k)

Idea:

Delete the k items.
Store them in an array.
Insert them back in reverse order.

(b) move(D,i,k,j)

Idea:

Delete the block into an array.
Adjust j if the block came before it.
Reinsert.

Problem 1-3: Binder Bookmarks

This is the interesting data-structure design problem.

The intended solution is usually:

Doubly linked list of pages.
Bookmarks are pointers into the list.
Why?

Need:

shift_mark ±1   -> O(1)
move_page       -> O(1)

which strongly suggests pointer manipulation rather than arrays.

Build
Store pages in a doubly linked list.
Each bookmark stores a pointer to the node immediately after the bookmark.
place_mark

Walk to index i and store a pointer.

Runtime:

O(n)
shift_mark
bookmark = bookmark.next

or

bookmark = bookmark.prev

Runtime:

O(1)
move_page

Remove one node and splice it elsewhere.

All pointer updates are constant time.

Problem 1-4: Doubly Linked List

For the written portion, focus on pointer changes.

insert_first(x)
Set x.next to the current head.
Set x.prev to None.
If the list is nonempty, update the old head's prev pointer.
Update head to x.
If the list was empty, also update tail to x.
insert_last(x)

Symmetric.

delete_first()
Store old head.
Move head to head.next.
Update head.prev.
Return removed node.
delete_last()

Symmetric.

Part (b): Remove interval [x
1
	​

,x
2
	​

]

Let

a = x1.prev
b = x2.next

Reconnect:

a.next = b
b.prev = a

The removed segment becomes:

head = x1
tail = x2

All pointer updates are constant.

Runtime:

O(1)
Part (c): Splice L2 after x

Suppose

y = x.next

Connect:

x -> L2.head
L2.tail -> y

Fix the four surrounding pointers.

Empty L2 afterward.

Runtime:

O(1)
Python Template for Problem 1-4

The exact code depends on the template MIT provided. If you upload the Python starter file, I can show exactly what functions to fill in.

Typically the methods look like:

def insert_first(self, x):
    x.prev = None
    x.next = self.head

    if self.head:
        self.head.prev = x
    else:
        self.tail = x

    self.head = x
def insert_last(self, x):
    x.next = None
    x.prev = self.tail

    if self.tail:
        self.tail.next = x
    else:
        self.head = x

    self.tail = x
def delete_first(self):
    if self.head is None:
        return None

    x = self.head
    self.head = x.next

    if self.head:
        self.head.prev = None
    else:
        self.tail = None

    return x
def delete_last(self):
    if self.tail is None:
        return None

    x = self.tail
    self.tail = x.prev

    if self.tail:
        self.tail.next = None
    else:
        self.head = None

    return x


