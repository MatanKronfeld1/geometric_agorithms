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

Be the 𝒔𝒑𝒓𝒆𝒂𝒅 of $P$, which is the ratio between the 𝑑𝑖𝑎𝑚𝑒𝑡𝑒𝑟 of 𝑃 and the distance between the two closest points in $P$.

---
**Lemma:** Let $P$ be a set of $n$ points contained in the unit square, such that $\text{diam}(P) \ge \frac{1}{2}$, and Let $T$ be a quadtree of $P$ constructed over the unit square, where
no leaf contains more than one point of $P$.

**Then, the complexity of $T$ is bounded by the Spread $\Phi$:**

1.  **Depth:** The maximal depth of $T$ is bounded by $O(\log\Phi)$.
2.  **Construction Time:** $T$ can be constructed in $O(n \log\Phi)$ time.
3.  **Total Size:** The total size (number of nodes) of $T$ is $O(n \log\Phi)$
where $\Phi = \Phi(P)$

---

**Proof:**
First, let's describe a construction algorothm for $T$.
- Start with the root (representing the unit square) and keep splitting the node as long as it contains more than one points of $P$. if you create an empy leaf - remove it and store a pointer to this leaf in the parent node.

**Depth, Construction Time, and Space Bound**
For any $p, q \in P$ define level $u = \lfloor \lg \|p - q\| \rfloor - 1$.
Let $v$ be a node at level $u$. The side length of its corresponding $\Box_v$ is $2^u$. Indeed,$diam(\square_v) \le \sqrt{2} \cdot 2^u = \sqrt{2} \cdot 2^{\lfloor \lg \|p - q\| \rfloor - 1} \le \frac {\sqrt{2}}{2} 2^{\lg\|p-q\|} = \frac {\sqrt{2}}{2}\|p-q\| < \|p-q\|$.

In particular, any node of level $l = -\lceil \lg{\Phi(P)} \rceil - 2$ can contain at most one point of $P$
Since the construction algorithm spends $O(n)$ time at each level of $T$, it follows that the construction time is O(n logΦ), and this also bounds the size of $T$.

---

One may realize that such tree $T$ may have a lot of nodes which are of degree 1 (has a single child).
If a node $v$ has more than one child, then it contains at least two different quadrants $\Box_x, \Box_y$, both contain points of $P$.
Thus, a quadtree might have a lot of "useless" nodes which one should be able to get rid of and get a more compact data structure.![bg right 80%](uncompressed.png)


---

# Construction #

We can replace such a sequence of edges by a single edge.
We will store inside each node $v$ its square $\Box_v$ and its level $l(v)$.

Given a path of vertices that are all of degree one, we will replace
them with a single vertex $v$ that corresponds to the first vertex in this path, and its only child would be the last vertex in this path (which is the first node of degree larger
than one).

![bg right 90%](compressed.png)

---

**Key Insight: the number of compressed nodes is $\in$ $O(n)$**

* **Compressed Nodes:** There are at most $O(n)$ nodes with degree 1.
    * Every single-child node (except the root) must have a parent with **degree $\ge 2$** (otherwise, they would be merged).
* **Branching Nodes:** There are at most **$n-1$** nodes with degree $\ge 2$.
(Splitting a set of $n$ points into singletons requires at most $n-1$ splits).
    * Therefore, we can "charge" every compressed node to a branching parent.

---

**Applications**
Consider the problem of reporting all the points inside a given trianlge.
A regular quadtree might require unbounded space, since the spread of the point set (and therefore the depth) might be arbitrarily large
A compressed quadtree would use only $O(n)$ space
The quadtree shown at the example the demostrates the "worse case" space complexity (the depth is $\in O(n)$ and it looks like a linked list)

![bg right 90%](list.png)

---

# Efficient Construction: The "Bit Twiddling" Requirement

To build Compressed Quadtrees efficiently we implicitly assume the **Unit RAM model**.
This model allows us to perform bit-manipulation on real numbers in $O(1)$ time.

**Definition ($bit_{\Delta}$):**
Let $\alpha, \beta \in [0, 1)$ be two real numbers.
$bit_{\Delta}(\alpha, \beta)$ is the index of the **first bit** after the period in which their binary representations differ.
**Example:**
* $\alpha = 1/4 = 0.01_2$
* $\beta = 3/4 = 0.11_2$
* $bit_{\Delta}(\alpha, \beta) = 1$ (They differ at the 1st bit).

---

**Lemma:**
If one can compute a compressed quadtree of two points in constant time, then one can compute $bit_{\Delta}(\alpha, \beta)$ in constant time.

**Proof**
* Imagine building a 1D compressed quadtree (a trie) for the set $\{\alpha, \beta\}$.
* Since they share a common prefix (the matching bits), the root will be a **compressed node**.
* The length of this compressed edge corresponds exactly to the number of matching bits.
* The level where the node finally splits is exactly the index $bit_{\Delta}(\alpha, \beta)$.

Thus, building the tree quickly **is equivalent** to finding the first different bit quickly.

---

once one has such an operation at hand, it is quite easy to compute a
compressed quadtree efficiently via “linearization”, and we'll see more details later at *dynamic quadtrees*.
However, the resulting algorithm is somewhat counterintuitive. As a first step, we suggest a direct construction algorithm.

## A construction algorithm ##
 Let $P$ be a set of $n$ points in the unit square, with unbounded spread. We are interested in computing the compressed quadtree of $P$.

 **Theorem:** Given a set $P$ of $n$ points in the plane, one can compute a compressed quadtree of $P$ in $O(n log n)$ time.

---

---

# Proof: #
Compute a disk $D$ of radius $r$ containing at least **$n/10$** points, where $r \le 2 \cdot r_{opt}(P, n/10)$, where $r_{opt}(P, n/10)$ is radius of the smallest disk contains $n/10$ points of $P$. This is done in linear time (lemma shown in chapter 1).

Let $\alpha = 2^{\lfloor \lg r \rfloor}$
Consider the grid $G_\alpha$.
Since $diam(D)=2r \le 4\alpha$ (Because $2^{\lg{r} -1} \le \alpha$ and then $r/2 \le \alpha$)
and the disk $D$ is covered by a $5×5 = 25$ grid cells of $G_\alpha$,
then by pigeon hole principle there must be a cell in $G_\alpha$ that contains at least $n/10/25 = n/250$ points of $P$.

---








