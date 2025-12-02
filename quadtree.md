---
marp: true
theme: default
paginate: true
math: mathjax
---

# Quadtrees – Hierarchical Grids

---

### Background…

Let $P_{map}$ be a planar map. To be more concrete, let $P_{map}$ a partition of the unit square into polygons.
![bg right 90%](planar_map.png)
**Our goal** – We want to preprocess the maps for point location queries, so given a point $p$, quickly find which polygons contain it. There are numerous data-sctructures that can do it, let consider the following data structure.

---

## The Simple Solution - Construction ##

- Build a tree $T$, where every node $𝑣$∈$T$ corresponds to a cell $\Box_v$
- the root corresponds to the unit square.
- Each internal node has four children that correspond to the four equal sized squares formed by splitting $\Box_v$ by horizontal and vertical cuts
- The conflict list of the square $\Box_v$ (i.e., the square associated with $v$) is a list of all the polygons of $P_{map}$ that intersect $𝑣$.

---

**Recursive Construction**
- Start at the root ($𝑣=𝑟𝑜𝑜𝑡_𝑇$)
- Compute the conflict list for the current node from its parent’s list.
- Termination Condition: If the conflict list size is small (e.g., ≤ 9 polygons), stop. This node becomes a leaf.
- Otherwise, split the node into four children and recurse.
**Note**: We only store the actual conflict lists at the leaves to save space, by deleting the parent’s list after processing its children.

---

**Point-Location Query:**
To identify the polygon containing a query point $q$ :
- Start at the root of $T$
- Traverse down the tree, iteratively moving to the child node whose quadrant contains 𝑞 (Note that this can be done in 𝑂(1))
- Terminate upon reaching a leaf.
- Scan the leaf's conflict list to identify the specific polygon containing $q$ (And this takes also $𝑂(1)$ time).

**Complexity Analysis:**
*Worst Case*: If polygons are "long and skinny," they may intersect many squares without being fully contained. This can lead to a tree of unbounded depth and complexity.
*Reasonable Input:* - for reasonable inputs, the complexity of the quadtree will be linear in the size of the input (a proof for fat triangles polygons is shown at the end)

---

# Fast point-location query in a quadtree #

**Definition:** A square is a canonical square if it is contained inside the unit square; it is a cell in a grid $𝐺_𝑟$ and $𝑟$ is a power of two. We will refer to such a grid $𝐺_𝑟$ as a **canonical grid**.

Consider a node $𝑣$ of a quadtree of depth $𝑖$ (the root has depth 0) and its associated square $\Box_v$

The side length of $\Box_v$ is $2^{−𝑖}$, and it is a canonical square inside a canonical grid $𝐺_𝑟$. We will refer to $𝑙(𝑣)=−𝑖$ as the **level** of $𝑣$.

---


We’ll make a unique ID for each node by the following triple:
 $id(v) = (l(v), \lfloor x/r \rfloor, \lfloor y/r \rfloor)$ where $(x,y)$ is any point in $\Box_v$ and $r$ = $2^{l(v)}$  (i.e., $𝑟$ is the side length of the squares correspond to all the nodes at level $𝑙(𝑣)$.)

Notice that $𝑙(𝑣)$ represents the depth, $\lfloor x/r \rfloor$ represents the row, and $\lfloor y/r \rfloor$ represents the column.

![w: 100 center](ids.png)

---

# Fast point-location in a quadtree. #
Given a query point 𝑞 and a level $𝑙$ we can compute the square at level $𝑙$ containing $𝑞$ at $𝑂(1)$ time.
Let **QTGetNode($T$, $q$, $d$)** denote the procedure that, in constant time, returns the node $𝒗$ of depth $𝑑$, where $𝑞$ is any point inside square $\Box_v$
We can create a hash-table storing all the IDs of nodes in the quadtree, (since out tree is already built)

---
Like in Binary search, we’ll use QTFastPLI($T$, $q$, $l$, $h$), 
with 𝑚 representing the median between the upper level ($𝑙$) and the lower level ($ℎ$)
- If $𝑣$ = null then we searched too deep, so we’ll recurse with QTFastPLI($T$, $q$, $l$, $m-1$)       
- Else, if $𝑣$ is a leaf then we’re done.
- Otherwise – we need to search deeper, so we’ll recurse with QTFastPLI($T$, $q$, $m+1$, $h$)

![bg right 90%](alg.png)

---
# So far #
Given a quadtree $𝑇$ of size $𝑛$ and of height $ℎ$, one can preprocess it (using
hashing), in linear time, such that one can perform a point-location query in $𝑇$ in $𝑂(𝑙𝑜𝑔(ℎ))$

**The problem:** $ℎ$ isn’t bounded, so the query can be very inefficient.
For that we’ll use a tricky but very useful “compression” of the quadtree.

---

# Compressed Quadtrees #

Let $P$ be a set of $n$ points in the unit square.
Let $\Phi(P) = \frac{\max_{p,q \in P} \|p - q\|}{\min_{p,q \in P, p \neq q} \|p - q\|}$
























