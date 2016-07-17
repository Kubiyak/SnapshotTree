SnapshotTree
============

Tree-like structure supporting multi-reader snapshot iteration with partial support for multiple writers.  Stored records must be copy constructible and comparable.

Note: After some consideration, I've decided to work on a snapshot vector/deque
implementation before returning to this project.

Usage
=====

In the main thread:<br>
1.  Create a SnapshotTree object.<br>
2.  Start a number of worker threads each receiving a reference to the tree object.<br>
3.  Wait for threads to complete.<br>

In the worker threads:<br>
1.  Obtain a handle for tree object access.  Handles are read or write.  Read handles open snapshots on the data which can be used to view the state of the tree at the time the handle was created indefinitely.  Write handles can be used to read records but do not create a consistent snapshot and incur locking overhead.<br>
2.  Access the tree ( either read or write ops )<br>
3.  Leave the scope in which the handle object exists in.  This is important for reclamation of resources used in read snaphsots and tree structure optimization operations.<br>

Handles
=======

There are a fixed number of handles which can be active at any time.  This is a configurable parameter but ideally there should not be more than a handful of handles open per thread at any time.  

Tree Ops
========

Update and remove are the supported write operations.  Find and iteration are supported for the read ops.  


Implementation
==============

Immutability is an important property of the implementation.  The idea is to use a list structure to store updates in when they are first made and to gradually incorporate data from the list structure into a sorted set like structure.

Dependencies
============

The implementation is heavily dependent on c++1y features.  The arena allocator from the ArenaAlloc project ( see Kubiyak/ArenaAlloc on github ) is used for memory allocation.  For convenience a recent version of arenaalloc.h from ArenaAlloc is included in this project.
