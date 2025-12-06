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

- Build a tree $T$, where every node $v \in T$ corresponds to a cell $\Box_v$
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
- Traverse down the tree, iteratively moving to the child node whose quadrant contains 𝑞 (Note that this can be done in 𝑂(1) time)
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

with a **point-location query**, given a quadtree $T$ and a point $q$, we would want to return the canonical square that contains only $q$ (or, on other word - a leaf containing $q$).

We’ll make a unique ID for each node by the following triple:

---

 $$
 id(v) = (l(v), \lfloor x/r \rfloor, \lfloor y/r \rfloor)
 $$
 Where $(x,y)$ is any point in $\Box_v$ and $r$ = $2^{l(v)}$  (i.e., $𝑟$ is the side length of the squares correspond to all the nodes at level $𝑙(𝑣)$). Notice that $𝑙(𝑣)$ represents the depth, $\lfloor x/r \rfloor$ represents the row, and $\lfloor y/r \rfloor$ represents the column.

![w: 100 center](ids.png)

---

# Fast point-location in a quadtree. #
Given a query point 𝑞 and a level $𝑙$ we can compute the ID of the square containing it at $𝑂(1)$ time.
So given a quadtree $T$, a natural algorithm for a point location query would be to store all IDs of nodes in $T$ in a hash-table, and compute the maximal height $h$ of $T$.

Let **QTGetNode($T$, $q$, $d$)** denote the procedure that, in constant time, returns the node $𝒗$ of depth $𝑑$ (i.e., its level is -$d$), where $𝑞$ is any point inside square its corresponding square $\Box_v$.

---
**Binary search for point-location queries**
 We’ll use QTFastPLI($T$, $q$, $l$, $h$), with 𝑚 representing the median between the lowest depth ($𝑙$) and the highest depth ($ℎ$).
- If $𝑣$ = null then we searched too deep, so we’ll recurse with QTFastPLI($T$, $q$, $l$, $m-1$)       
- Else, if $𝑣$ is a leaf we’re done.
- Otherwise – we need to search deeper, so we’ll recurse with QTFastPLI($T$, $q$, $m+1$, $h$)

![bg right 90%](alg.png)

---
## Summing it up ##
Given a quadtree $𝑇$ of size $𝑛$ and of height $ℎ$, one can preprocess it (using
hashing), in linear time, such that one can perform a point-location query in $𝑇$ in $𝑂(𝑙𝑜𝑔(ℎ))$

**The problem:**
$ℎ$ isn’t bounded, so the query can be very inefficient.
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

Before we jump into the proof, first let's describe a construction algorothm for $T$.
- Start with the root (representing the unit square) and keep splitting the node as long as it contains more than one points of $P$. if you create an empy leaf - remove it and store a pointer to this leaf in the parent node.

---

# Proof: #
**Depth:** For any $p, q \in P$ define level $u = \lfloor \lg \|p - q\| \rfloor - 1$.
**claim:** a node $v$ of level $u$ that contains $p$ ,must not contain $q$
**proof:** Let $v$ be a node at level $u$. The side length of its corresponding $\Box_v$ is $2^u$. Indeed,
$$
diam(\square_v) \le \sqrt{2} \cdot 2^u = \sqrt{2} \cdot 2^{\lfloor \lg \|p - q\| \rfloor - 1} \le \frac {\sqrt{2}}{2} 2^{\lg\|p-q\|} = \frac {\sqrt{2}}{2}\|p-q\| < \|p-q\|
$$
So looking at $P$
$$
\min_{p,q \in P, p \neq q} \|p - q\| = \frac{1}{\Phi(P)}\cdot \text {diam} (P) \ge \frac{\frac{1}{2}}{\Phi(P)} = \frac{1}{2\Phi(P)}
$$
So any node of level $l$ = $\lfloor \lg{\frac{1}{2\Phi(P)}}\rfloor - 1 = \lfloor -\lg{\Phi(P)} -1 \rfloor -1 = -\lceil \lg{\Phi(P)}\rceil - 2$ contains at most one point of $P$, so the total depth $\in O(\lg {\Phi(P)})$.

---

**Construction time:** The construction algorithm spends $O(n)$ time at each level of $T$, it follows that the construction time is $O(n\lg{\Phi(P)})$, and this also bounds the size of $T$.

One may realize that such tree $T$ may have a lot of nodes which are of degree 1 (have a single child). For example, if most points lie within a very small cell.

These "chains" of useless nodes don't add information and one should be able to get rid of and get a more compact data structure.![bg right 80%](uncompressed.png)


---

# Construction #

We can replace such a sequence of edges by a single edge.
We will store inside each node $v$ its square $\Box_v$ and its level $l(v)$.

Given a path of vertices that are all of degree one, we will replace
them with a single vertex $v$ that corresponds to the first vertex in this path, and its only child would be the last vertex in this path (which is the first node in this path of degree larger than one).

![bg right 90%](compressed.png)

---

## A compressed quadtree has a linear size ##

* **Compressed Nodes:** In a compressed quadtree, Every node with a single child (except the root) must have a parent with *degree $\ge 2$* (otherwise, they would be merged).
* **Branching Nodes:** There are at most **$n-1$** nodes with degree $\ge 2$.
(Splitting a set of $n$ points into singletons requires at most $n-1$ splits).
    * Therefore, we can "charge" every compressed node to a branching parent.

**There for, a compressed tree has linear size**
(However, the depth can also be linear in the worst case)

---

**Applications**
Consider the problem of reporting all the points inside a given trianlge.

A regular quadtree might require unbounded space, since the spread of the point set (and therefore the depth) might be arbitrarily large.

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
* Imagine building a 1D compressed quadtree for the set $\{\alpha, \beta\}$.
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

Lemmas

---

# Proof: #
Compute a disk $D$ of radius $r$ containing at least **$n/10$** points, where $r \le 2 \cdot r_{opt}(P, n/10)$

Let $\alpha = 2^{\lfloor \lg{r} \rfloor}$
$\text {diam} (D)=2r \le 4\alpha$ (Because $2^{\lg{r} -1} \le \alpha$ and then $r/2 \le \alpha$)
Consider the grid $G_\alpha$.
The disk $D$ is covered by at most $5×5 = 25$ grid cells of $G_\alpha$,
Then, by pigeon hole principle there must be a cell in $G_\alpha$ that contains at least $n/10/25 = n/250$ points of $P$.

![bg right 90%](pigeon.png)

---

By the previous lemma, no cell of $G_\alpha$ contains more than $5(n/10)$ points = $n/2$ points.

We can compute $G_{\alpha}$ and find the cell $\Box$ with maximal density. We are assured $\Box$ has at least $n/250$ points and the rest of the space has at least $n/2$ points.

## The Algorithm: Recursive Construction & Merging ##

Let $P_{in}$ be the points inside $\square$ and $P_{out}$ the points outside ($\square$), we can proceed with the following recursion:
Independently, build two compressed quadtrees:$\mathcal{T}_{in}$ for $P_{in}$ and $\mathcal{T}_{out}$ for $P_{out}$.
Since all points in $P_{in}$ are inside the cell $\square$, the root of $\mathcal{T}_{in}$ will corresponds exactly to the region $\square$ (Let's call to this root $V_{in}$)
Simililarly, since $P_{out}$ doesn't include points of $\square$, $\mathcal{T}_{out}$ would have an empty leaf representing the region $\square$ (let's call this leaf $V_{out}$). which will point to $V_{in}$.
After the recursion we can "hang" $V_{in}$ on $V_{out}$, merging them into a single node.

---

The bit$_∆(α, β)$ should be used in case we hang $\mathcal{T}_{in}$ on a compressed node of $\mathcal{T}_{out}$.
using the RAM unit model, this can be done in constant time. Note that also $V_{out}$ and (therefore $V_{in}$) may be deleted because of the compression.

The overall construction time:  $T(n) = T(P_{in}) + T(P_{out}) + O(n)$ = $O(n \log n)$."

---
# Balanced Quadtrees #

**Lemma:** Given a list $C$ of $n$ canonical squares lying inside the unit square, one can construct a (minimal) compressed quadtree $T$ such that for any square $c \in C$, there exists a node $v \in T$, such that $\Box_{v}=c$. The construction time is $O(n \log n)$.

**Proof:** For every canonical square $\Box \in C$ put two points in $\Box$ such that they belong to two different subsquares of $\Box$.
Every $\square \in C$ will be represented by an internal node with at least two children, and every node that is not a leaf represents a canocial square from the list.
then trimming away the leaves from the quadtree will leave us with the desired quadtree

---

# Fingering #
Similarly to the point location algorithm from the beginning, we would want to perform a fast algorithm for returning a leaf (which is essentially the smallest canonical square contain it)
In worst case, that would take $\Omega(n)$ (demonstrated in example in previous slides)

**Definition:**
Let $T$ be a tree with $n$ nodes. A separator in $T$ is a node $v$, such that if we remove $v$ from $T$, we remain with a forest, and every tree in the forest has at most $\lceil n/2 \rceil$ vertices.

---

**Lemma.** Every tree has a separator, and it can be computed in linear time.

**Proof.** Let $T$ be a rooted tree. We can compute for every node in the tree the number of nodes in its subtree in linear time.
Let $v$ to be the lowest node in $T$ that its subtree has $\ge \lceil n/2 \rceil$ nodes in it.
We can find this node by a DFS traversal.
clearly such node must exist, since the subtree rooted in the root contains $n$ nodes and every subtree representing a leaf contains $n/2$ roots at most.

Notice that by our DFS algorithm, all the children of $v$ don't contain less than $\lceil n/2 \rceil$ nodes (as they're deeper than $v$). since the tree rooted by $v$ contains more than $\lceil n/2 \rceil$ nodes, removing $v$ will create a forest where every tree contains less than $\lceil n/2 \rceil$ nodes. then $v$ is the separator of $T$.

---
construction algorithm for a balanced quadtree:

Find a separator $v \in T$ and create a root node $f_{v}$ for $T'$ which has a pointer to $v$; now recursively build a finger tree for each tree of $T \setminus \{v\}$.
Hang the resulting finger trees on $f_{v}$. The resulting tree is the required finger tree $T'$.

Given a query point $q$, we traverse $T'$, where at node $f_{v} \in T'$, we check whether the query point $q \in \Box_{v}$, where $v$ is the corresponding node of $T$. If $q \notin \Box_{v}$, we continue the search into the child of $f_{v}$ which corresponds to the connected component outside, that was hung on $f_{v}$. Otherwise, we continue into the child that contains $q$: naturally, we have to check out the $O(1)$ children of $v$ to decide which one we should continue the search into. This takes constant time per node. As for the depth for the finger tree $T'$, observe $D(n) \le 1 + D(\lceil n/2 \rceil) = O(\log n)$.

Thus, a point-location query in $T'$ takes logarithmic time.

---

Theorem: Given a compressed quadtree $\widehat{T}$ of size $n$, one can preprocess it, in time $O(n \log n)$ such that given a query point $q$, one can return the lowest node in $\widehat{T}$ whose region contains $q$ in $O(\log n)$ time.
Proof. We just need to bound the preprocessing time. Each level we scan all the nodes in $O(n)$ and we might have depth of up to $\lg{(n)}$ before reaching to leaves (as all the trees in the forest has maximal $\lceil n/2 \rceil$ size)
then we get the preprocessing time is in $O(n\log{n})$

---

Z order and Q order

Here we show how to store compressed quadtrees using any data-structure for ordered sets. The idea is to define an order on the compressed quadtree nodes and then store the compressed nodes in a data-structure for ordered sets. We show how to perform basic operations using this representation. In particular, we show how to perform insertions and deletions. This provides us with a simple implementation of dynamic compressed quadtrees. We start our discussion with regular quadtrees and later on discuss how to handle compressed quadtrees.

---

# Q-order and Z-Order #

Consider a regular quadtree $T$ and a DFS traversal of $T$, where the DFS always traverse the children of a node in the same relative order.

Comparing Canonical Squares ($\Box$):
- The order is determined by the visit time during the traversal.
- $\Box \prec \widehat{\Box}$ if the traversal visits $\Box$ before $\widehat{\Box}$.

Extending to Points: 
- If a point $p$ is inside a square $\Box$, then $\Box < p$. otherwise, $p$ lies in other cell, $\overline{\Box}$.
Then $\Box \prec p$ $\iff$ $\Box \prec \overline{\Box}$ .
- If two points lie in different cells, then $p \prec q$ $\iff$ $\Box_{p} \prec \Box_{q}$. 

---

## Z-order ##

When the ordering is restricted only to points by the DFS order of a given quadtree, we'll call this order **Z-order**.

Formally, this order is induced by a specific bijection

$$
z: [0, 1) \to [0, 1)^{2}
$$

where given a real number $\alpha \in [0,1)$, with the binary expansion $\alpha=0.x_{1}x_{2}x_{3}...$, the Z-order mapping of $\alpha$ is the point
$$
z(\alpha)=(0.x_{2}x_{4}x_{6}..., 0.x_{1}x_{3}x_{5}...)
$$

---


# The connection betwwen Z-order and DFS order #

Let $p \in [0,1)^{2}$ and think of a point-location query in an infinite quadtree for $p$.

In the top level of the quadtree we have four options to continue the search:

- If $y(p) \in [0,1/2)$ then the first bit in the encoding is 0
- If $y(p) \in [1/2,1)$, the first bit output is 1.

Similar encoding of the second bit represents $x(p)$, and so we solved the query for the first level.

This search cotinue to the next level, and the $2i+1$ and $2i+2$ bits in the encoding of $p$ represent the $i$-th bits of $y(p)$ and $x(p)$.

---

Let $enc(p)$ denote the number in the range $[0, 1)$ encoded by this process.

For any two points $p, q \in [0,1)^{2}$, we have that $p \prec q$ if and only if $enc(p) < enc(q)$ (as previously demonstrated)

**We need to be able to order any two given cells/points according to $\prec$ quickly.**

**Definition:** For any two points $p, q \in [0,1]^{2}$ let $lca(p,q)$ denote the smallest canonical square that contains both $p$ and $q$.
Then
- its level $l =  1 - \min(bit_{\Delta}(x_p, x_q), bit_{\Delta}(y_{p},y_{q}))$
- its side length is $2^l$

Let $\Delta=2^{l}$, and let $x^{\prime}=\Delta\lfloor x_p/\Delta\rfloor$ and $y^{\prime}=\Delta\lfloor y_p/\Delta\rfloor$. ($x^{\prime}$ and $y^{\prime}$  represent where the cell start), Then:
$$
lca(p,q)=[x^{\prime},x^{\prime}+\Delta) \times [y^{\prime},y^{\prime}+\Delta)
$$

