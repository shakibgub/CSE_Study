# COMPSCI 224 Advanced Algorithms — Lecture Notes

## Lecture 1: Word-RAM Model, Predecessor Problem, van Emde Boas Trees, X-fast/Y-fast Tries

**Instructor:** Jelani Nelson  
**নোটের ভাষা:** বাংলা ব্যাখ্যা + গুরুত্বপূর্ণ technical term English  
**মূল থিম:** algorithm-এর efficiency শুধু algorithm-এর উপর নয়, বরং **computation model**-এর উপরও নির্ভর করে। Comparison model-এ sorting-এর জন্য \(\Omega(n\log n)\) lower bound থাকলেও word-RAM model-এ bit operations ও word-level arithmetic ব্যবহার করে predecessor data structure এবং sorting দ্রুত করা যায়।

---

# 1. Lecture-এর মূল বিষয়, লক্ষ্য, এবং বড় ধারণা

এই lecture মূলত তিনটি বড় ধারণা প্রতিষ্ঠা করে।

1. **Model matters.**  
   আমরা সাধারণ algorithms course-এ running time বলতে প্রায়ই “number of steps” বুঝি, কিন্তু step বলতে কী বোঝাচ্ছি তা model-dependent. Comparison sorting model-এ algorithm শুধু item pair compare করতে পারে; word-RAM model-এ integer arithmetic, bitwise operations, shifts ইত্যাদি constant time-এ করা যায়। তাই একই problem ভিন্ন model-এ ভিন্ন complexity পেতে পারে।

2. **Predecessor problem sorting-এর সঙ্গে গভীরভাবে সম্পর্কিত।**  
   একটি set \(S\)-এর মধ্যে query \(z\)-এর strictly smaller সবচেয়ে বড় element খুঁজে বের করা হলো predecessor query. Dynamic predecessor data structure দিয়ে numbers insert করে বারবার predecessor নিয়ে sorted order বের করা যায়। তাই predecessor faster হলে sorting-ও faster হতে পারে—কিন্তু model গুরুত্বপূর্ণ।

3. **van Emde Boas tree \((vEB)\) universe recursion ব্যবহার করে \(O(\log\log u)\) query/update দেয়।**  
   যদি universe size \(u=2^w\), তাহলে \(O(\log\log u)=O(\log w)\). vEB tree universe-কে \(\sqrt u\) cluster-এ ভাঙে, প্রতিটি cluster-এর universe size \(\sqrt u\). summary structure non-empty cluster track করে। Naive implementation \(\Theta(u)\) space নেয়, কিন্তু hash table/dictionary দিয়ে only non-empty clusters store করলে space \(\Theta(n)\)-এ নামানো যায়।

পরবর্তী অংশে lectureটি বইয়ের অধ্যায়ের মতো সাজানো হলো।

---

# 2. Course context: Advanced Algorithms-এ কী আলাদা?

লেকচারার course-এর শুরুতে বলেন যে পূর্বের algorithms course-এ অনেক standard topics দেখা হলেও algorithms field অনেক বড়। এই course-এর লক্ষ্য হলো:

- algorithm analyze করার ক্ষমতা বাড়ানো;
- নতুন algorithm তৈরি করার কৌশল শেখা;
- running time ও memory-এর বাইরে অন্যান্য efficiency measure বা computational model বোঝা;
- written, theory-focused reasoning-এ দক্ষ হওয়া।

এই lecture সেই লক্ষ্যকে শুরু করে একটি simple কিন্তু গভীর উদাহরণ দিয়ে: **sorting lower bound সত্য, কিন্তু শুধু comparison model-এ।** বাস্তব কম্পিউটার integer word নিয়ে কাজ করে, তাই word-RAM model-এ আমরা আরও শক্তিশালী operations পাই।

---

# 3. Sorting lower bound: কোন model-এ সত্য?

## 3.1 Intuition

অনেকেই জানে “sorting-এর lower bound \(\Omega(n\log n)\)”. কিন্তু এই statement অসম্পূর্ণ। সঠিক statement হলো:

> Comparison-based sorting-এ \(n\) distinct item sort করতে \(\Omega(n\log n)\) comparisons লাগে।

যদি algorithm শুধু compare করতে পারে—যেমন “\(a_i<a_j\)?”—তাহলে decision tree argument থেকে এই lower bound আসে। কিন্তু বাস্তব machine integer-এর bit pattern নিয়ে কাজ করে; bitwise AND, XOR, shift, arithmetic ইত্যাদি করা যায়। তাই word-RAM model-এ comparison lower bound সরাসরি প্রযোজ্য নয়।

## 3.2 Formal explanation: comparison sorting lower bound

Comparison sorting algorithm-কে একটি decision tree হিসেবে দেখা যায়।

- প্রতিটি internal node একটি comparison।
- প্রতিটি leaf একটি possible sorted permutation।
- \(n\) distinct item-এর possible ordering সংখ্যা \(n!\)।
- decision tree-এর leaf অন্তত \(n!\) হওয়া দরকার।
- height \(h\) হলে leaf সংখ্যা সর্বোচ্চ \(2^h\)। তাই:

\[
2^h \ge n! \quad\Rightarrow\quad h\ge \log_2(n!) = \Omega(n\log n).
\]

## 3.3 Why it matters

এই lower bound algorithm-এর intrinsic সত্য নয়; এটি model-specific. Word-RAM model-এ আমরা integers-এর bits inspect/manipulate করতে পারি, তাই কখনও \(n\log n\)-এর চেয়ে দ্রুত integer sorting সম্ভব। Lecture-এ পরে উল্লেখ করা হয়েছে:

- dynamic fusion trees দিয়ে \(O(n\sqrt{\log n})\) sorting;
- deterministic \(O(n\log\log n)\) integer sorting;
- randomized expected \(O(n\sqrt{\log\log n})\) sorting;
- linear-time integer sorting word-RAM model-এ possible কি না তা open question হিসেবে উল্লেখ করা হয়েছে।

## 3.4 Common pitfalls

- “Sorting cannot be faster than \(n\log n\)” বলা ভুল, যদি model specify না করা হয়।
- Radix sort, counting sort, word-RAM integer sorting—এগুলো comparison lower bound ভাঙে না; কারণ এগুলো comparison-only model-এ কাজ করছে না।
- Input integer universe size, word size, memory model—এসব complexity-তে গুরুত্বপূর্ণ।

---

# 4. Predecessor Problem

## 4.1 Intuition

একটি sorted dictionary কল্পনা করো। Query \(z\) দিলে তুমি জানতে চাও \(z\)-এর ঠিক আগের element কোনটি—অর্থাৎ set-এর মধ্যে \(z\)-এর চেয়ে ছোট সবচেয়ে বড় element।

যেমন:

\[
S=\{2,5,9,13\}.
\]

তাহলে:

- \(\operatorname{pred}(10)=9\)
- \(\operatorname{pred}(13)=9\), যদি predecessor strictly smaller ধরা হয়
- \(\operatorname{pred}(2)=\text{NIL}\)
- \(\operatorname{pred}(100)=13\)

## 4.2 Formal Definition

একটি ordered universe \(U=\{0,1,\dots,u-1\}\)-এর subset \(S\subseteq U\) given. Query \(z\in U\)-এর predecessor হলো:

\[
\operatorname{pred}_S(z)=\max\{x\in S: x<z\}.
\]

যদি এমন কোনো \(x\) না থাকে, return করা হবে `NIL`।

## 4.3 Static vs Dynamic Predecessor

### Static predecessor

- Set \(S\) preprocessing-এর সময় fixed.
- Query support করতে হবে।
- Insert/delete নেই।

### Dynamic predecessor

- Query ছাড়াও update support করতে হবে:
  - `Insert(x)`
  - `Delete(x)`
- Set \(S\) সময়ের সঙ্গে বদলায়।

## 4.4 Classical solutions

### Static: sorted array + binary search

- Store \(S\) sorted array-তে।
- Query \(z\)-এর জন্য binary search করে \(z\)-এর lower bound বের করো।
- Time: \(O(\log n)\)
- Space: \(O(n)\)

### Dynamic: balanced BST

- Red-black tree, AVL tree ইত্যাদি।
- Query/update: \(O(\log n)\)
- Space: \(O(n)\)

## 4.5 Why it matters

Predecessor একটি fundamental data-structural problem. Integer sorting, routing tables, databases, ordered dictionaries, memory allocators, computational geometry—অনেক জায়গায় predecessor-like operation দেখা যায়। এই lecture-এ predecessor ব্যবহার করা হয়েছে দেখানোর জন্য যে word-RAM model-এ \(O(\log n)\)-এর চেয়ে দ্রুত ordered search সম্ভব।

## 4.6 Common pitfalls

- Predecessor strict না non-strict—এটি specify করতে হয়। Lecture-এ definition strict: \(x<z\)।
- Static predecessor দিয়ে sorting করতে গেলে preprocessing time গুরুত্বপূর্ণ। Dynamic predecessor দিয়ে empty থেকে insert করলে preprocessing issue নেই।
- Universe size \(u\) এবং number of stored elements \(n\) আলাদা parameter।

---

# 5. Dynamic Predecessor দিয়ে Sorting

## 5.1 Problem statement

Given \(n\) distinct integers, sort them in increasing or decreasing order using a dynamic predecessor data structure.

## 5.2 Input and output

**Input:** distinct integers \(a_1,a_2,\dots,a_n\).  
**Output:** sorted sequence.

## 5.3 High-level idea

যদি dynamic predecessor data structure দ্রুত হয়, তাহলে:

1. সব number insert করো।
2. maximum element থেকে শুরু করো।
3. বারবার predecessor নিয়ে descending sorted order বের করো।

## 5.4 Step-by-step explanation

1. Linear scan করে maximum \(m\) খুঁজে বের করো।
2. Empty predecessor data structure initialize করো।
3. সব \(a_i\) insert করো।
4. Output \(m\)।
5. Repeatedly set \(m\leftarrow\operatorname{pred}(m)\) এবং output করো, যতক্ষণ না `NIL` হয়।

## 5.5 Pseudocode

```text
Sort-With-Dynamic-Predecessor(A[1..n]):
    D <- empty dynamic predecessor structure
    maxVal <- A[1]

    for x in A:
        if x > maxVal:
            maxVal <- x
        D.Insert(x)

    result <- empty list
    x <- maxVal
    while x != NIL:
        append x to result
        x <- D.Predecessor(x)

    reverse(result)      // if increasing order is required
    return result
```

## 5.6 Correctness intuition

যেহেতু predecessor strictly smaller সবচেয়ে বড় element দেয়, maximum থেকে predecessor chain follow করলে elements decreasing order-এ পাওয়া যায়:

\[
\max S,\ \operatorname{pred}(\max S),\ \operatorname{pred}^{(2)}(\max S),\dots
\]

Set finite, distinct, এবং প্রতিবার strictly smaller element পাওয়া যায়, তাই সব elements exactly once visit হবে। Reverse করলে increasing sorted order পাওয়া যায়।

## 5.7 Complexity

ধরি:

- Insert time: \(T_u\)
- Predecessor query time: \(T_q\)

তাহলে sorting time:

\[
O(nT_u+nT_q+n).
\]

যদি dynamic predecessor query/update \(O(\sqrt{\log n})\) হয়, তাহলে sorting \(O(n\sqrt{\log n})\) পাওয়া যায়। Lecture-এ এই implication dynamic fusion trees context-এ উল্লেখ করা হয়েছে।

## 5.8 Edge cases

- Duplicate থাকলে strictly predecessor chain duplicate হারিয়ে ফেলবে। সমাধান: value-এর সঙ্গে tie-breaking index pair \((value,i)\) store করা।
- Empty input হলে empty output।
- Static data structure হলে preprocessing time sorting-এর জন্য bottleneck হতে পারে। Lecture-এ বলা হয়েছে static fusion tree polynomial preprocessing নিলে sorting-এ directly use করা যায় না।

---

# 6. Word-RAM Model

## 6.1 Intuition

Word-RAM model বাস্তব machine-এর একটি idealized abstraction। এখানে data word আকারে stored, এবং এক word-এ fit করা integer-এর উপর arithmetic/bit operations constant time-এ করা যায়।

Comparison model-এ algorithm শুধু compare করে। Word-RAM model-এ algorithm integer-এর bit-level representation exploit করতে পারে।

## 6.2 Formal model

- Word size: \(w\) bits.
- Universe:

\[
U=\{0,1,\dots,2^w-1\},\qquad u=|U|=2^w.
\]

- Input items are integers fitting in one word.
- Pointers fit in one word.
- Since a data structure storing \(n\) items normally needs at least \(\Omega(n)\) memory cells, a pointer to memory requires at least \(\log n\) bits. তাই সাধারণ assumption:

\[
w\ge \log n.
\]

## 6.3 Constant-time operations

Given word-sized integers \(x,y\), word-RAM model usually allows constant time:

- integer arithmetic: \(+,-,\times,\lfloor x/y\rfloor\) where applicable;
- bitwise operations: NOT, XOR, OR, AND;
- shifts: left/right shift by a constant or by another word-sized amount;
- memory access by word-sized pointer.

Multiplication নিয়ে lecture-এ nuance এসেছে: product two words লাগতে পারে; overflow হলে model variant অনুযায়ী lower bits পাওয়া যায়। Algorithms usually specify করে কী দরকার।

## 6.4 Why it matters

vEB tree-তে \(x\)-কে high bits এবং low bits-এ split করতে হবে। Word-RAM-এ masking এবং shifting দিয়ে এটি constant time-এ করা যায়। Comparison model-এ এমন bit manipulation allowed নয়।

## 6.5 Formula connections

যেহেতু \(u=2^w\), তাই:

\[
\log u=w,
\qquad
\log\log u=\log w.
\]

vEB tree-এর \(O(\log\log u)\) time তাই \(O(\log w)\)।

## 6.6 Common pitfalls

- \(n\) এবং \(u\) এক নয়। \(u\) universe size; \(n\) stored item count।
- Word-RAM lower/upper bounds usually \(w\), \(n\), এবং \(u\)-এর relation-এর উপর নির্ভর করে।
- Constant-time multiplication সব word-RAM variant-এ একইভাবে defined নয়; theoretical papers model specify করে।

---

# 7. Main bounds in the lecture

## 7.1 van Emde Boas trees

Naive vEB tree:

- Query: \(O(\log\log u)=O(\log w)\)
- Update: \(O(\log\log u)=O(\log w)\)
- Space: \(\Theta(u)\)

Hash-table/dictionary improvement:

- Query/update: same asymptotic bound, assuming constant-time dictionary operations;
- Space: \(\Theta(n)\), randomized or expected/high-probability depending on dictionary implementation.

## 7.2 X-fast and Y-fast tries

X-fast trie:

- Query: \(O(\log w)=O(\log\log u)\)
- Space: \(O(nw)\)
- Update can cost \(O(w)\)

Y-fast trie:

- Uses indirection/bucketing.
- Query: \(O(\log w)\)
- Space: \(O(n)\)
- Update: amortized expected \(O(\log w)\)-type bound in the standard randomized version.

## 7.3 Fusion trees

Lecture-এ fusion trees-এর detailed construction পরবর্তী lecture-এর জন্য রেখে দেয়া হয়। এই lecture-এ bound বলা হয়েছে:

- Static query time:

\[
O(\log_w n)=O\left(\frac{\log n}{\log w}\right).
\]

- Space: \(O(n)\)
- Dynamic version possible but more complicated.

## 7.4 Combining vEB/Y-fast and fusion trees

আমরা better data structure choose করতে পারি:

\[
T(n,w)=O\left(\min\left\{\log w,\frac{\log n}{\log w}\right\}\right).
\]

এই expression সর্বোচ্চ হয় যখন দুই term approximately equal:

\[
\log w = \frac{\log n}{\log w}
\quad\Longrightarrow\quad
(\log w)^2=\log n.
\]

তাই:

\[
\min\left\{\log w,\frac{\log n}{\log w}\right\}
\le \sqrt{\log n}.
\]

অর্থাৎ near-linear-space predecessor data structure দিয়ে query/update \(O(\sqrt{\log n})\)-type bound পাওয়া যায়, যদি appropriate dynamic structures use করা যায়।

## 7.5 Consequence for sorting

Dynamic predecessor data structure যদি \(O(\sqrt{\log n})\) query/update দেয়, তাহলে dynamic predecessor দিয়ে sorting:

\[
O(n\sqrt{\log n}).
\]

Lecture-এ আরও দ্রুত integer sorting bounds উল্লেখ করা হয়, যেমন \(O(n\log\log n)\) deterministic এবং \(O(n\sqrt{\log\log n})\) expected randomized; কিন্তু সেগুলো এই lecture-এ develop করা হয়নি।

---

# 8. van Emde Boas Tree: Core Idea

## 8.1 Intuition

Binary search \(n\)-এর উপর divide করে, তাই \(O(\log n)\). vEB tree universe size \(u\)-এর উপর divide করে, কিন্তু half না করে **square root decomposition** করে:

- Universe \(u\) কে \(\sqrt u\) clusters-এ ভাগ করা হয়।
- প্রতিটি cluster-এর size \(\sqrt u\)।
- একটি summary structure বলে কোন cluster non-empty।

যেহেতু recursive universe size \(u\to\sqrt u\), recursion depth:

\[
u,\sqrt u,u^{1/4},u^{1/8},\dots
\]

যখন universe constant হয়, number of levels:

\[
O(\log\log u).
\]

এটাই vEB-এর speed-এর মূল।

## 8.2 Assumption about universe size

সহজ exposition-এর জন্য ধরে নেওয়া হয়:

\[
u=2^{2^k}
\]

কোনো integer \(k\)-এর জন্য। তাহলে বারবার square root নিলে integer universe পাওয়া যায়। বাস্তবে arbitrary \(u\) হলে next power বা rounded high/low split ব্যবহার করা যায়। এই rounding asymptotic bound বদলায় না।

## 8.3 Splitting an element into high and low parts

Let:

\[
b=\sqrt u.
\]

প্রতিটি \(x\in\{0,\dots,u-1\}\) uniquely লেখা যায়:

\[
x = c\cdot b+i,
\]

যেখানে:

\[
c=\left\lfloor \frac{x}{b}\right\rfloor,
\qquad
 i=x\bmod b,
\qquad
0\le c,i<b.
\]

এখানে:

- \(c=\operatorname{high}(x)\): cluster ID;
- \(i=\operatorname{low}(x)\): position inside cluster;
- \(\operatorname{index}(c,i)=c\sqrt u+i\): global value reconstruct করে।

যদি \(u=2^w\), তাহলে \(x\)-এর binary representation-এর left half bits হলো \(c\), right half bits হলো \(i\)। Word-RAM-এ shifts/masks দিয়ে এগুলো constant time-এ পাওয়া যায়।

## 8.4 Data structure fields

A vEB structure \(V\) over universe size \(u\) stores:

1. `V.min`: set-এর minimum element, যদি non-empty হয়;
2. `V.max`: set-এর maximum element;
3. `V.cluster[c]`: cluster \(c\)-এর জন্য recursive vEB over universe \(\sqrt u\);
4. `V.summary`: কোন cluster non-empty তা store করা vEB over universe \(\sqrt u\)।

Naive implementation-এ `cluster` একটি array of length \(\sqrt u\)। Later linear-space version-এ এটি dictionary/hash table।

## 8.5 Crucial invariant: minimum is special

Lecture-এর একটি গুরুত্বপূর্ণ trick:

> `V.min` element-টি তার natural cluster-এর মধ্যে আবার recursively store করা হয় না।

অর্থাৎ structure-এর global minimum একটি buffer field-এ থাকে। এটি insertion time \(O(\log\log u)\) রাখতে critical। যদি minimum-ও normal element হিসেবে recursively store করা হতো, insertion কখনও summary এবং cluster—দুটিতেই nontrivial recursive call করতে পারত, leading to \(O(\log u)=O(w)\)।

## 8.6 Invariants

For a non-empty vEB \(V\):

- `V.min` is the smallest stored key.
- `V.max` is the largest stored key.
- Every stored key except `V.min` is represented recursively in exactly one cluster.
- Cluster \(c\) is non-empty iff \(c\) is stored in `V.summary`.
- If key \(x\neq V.min\) has \(c=\operatorname{high}(x)\), \(i=\operatorname{low}(x)\), then \(i\) is stored in `V.cluster[c]`.

---

# 9. Algorithm: vEB Predecessor

## 9.1 Problem statement

Given a vEB tree \(V\) storing set \(S\subseteq\{0,
\dots,u-1\}\) and query \(x\), return:

\[
\operatorname{pred}_S(x)=\max\{y\in S:y<x\}.
\]

## 9.2 Input and output

**Input:** vEB structure \(V\), word-sized integer \(x\).  
**Output:** predecessor key or `NIL`.

## 9.3 High-level idea

1. যদি \(x\) সব stored element-এর চেয়ে বড় হয়, answer `V.max`।
2. যদি \(x\le V.min\), predecessor নেই।
3. Otherwise \(x\)-কে cluster ID \(c\) এবং internal index \(i\)-তে split করো।
4. যদি same cluster-এ \(i\)-এর predecessor থাকে, recursively cluster-এর মধ্যে খুঁজো।
5. না হলে previous non-empty cluster খুঁজতে `summary`-তে predecessor query করো।
6. সেই cluster-এর maximum নিয়ে global key reconstruct করো।
7. যদি previous non-empty cluster না থাকে, answer হতে পারে `V.min`।

## 9.4 Pseudocode

```text
VEB-Predecessor(V, x):
    if V is empty:
        return NIL

    if x <= V.min:
        return NIL

    if x > V.max:
        return V.max

    if V.u == 2:
        // universe {0,1}; previous cases handle most possibilities
        if x == 1 and V.min == 0:
            return 0
        else:
            return NIL

    c <- high(x)
    i <- low(x)

    C <- V.cluster[c]

    if C is non-empty and i > C.min:
        j <- VEB-Predecessor(C, i)
        return index(c, j)

    else:
        cPrev <- VEB-Predecessor(V.summary, c)

        if cPrev == NIL:
            // No non-empty cluster before c.
            // Since x > V.min, the global minimum is the predecessor.
            return V.min
        else:
            j <- V.cluster[cPrev].max
            return index(cPrev, j)
```

## 9.5 Why checking `C.min` is enough

Same cluster-এ predecessor আছে কি না জানতে full recursive query করার আগে শুধু cluster minimum check করলেই হয়। যদি \(i>C.min\), তাহলে cluster-এর মধ্যে \(i\)-এর চেয়ে ছোট কিছু আছে, so predecessor same cluster-এ। যদি \(i\le C.min\), same cluster-এ কোনো smaller element নেই; answer previous non-empty cluster বা global min।

## 9.6 Correctness intuition

Predecessor of \(x\) either:

1. same cluster \(c\)-এ থাকে, অথবা
2. কোনো cluster \(<c\)-এ থাকে, অথবা
3. global minimum `V.min` হয়।

Same cluster-এ smaller element থাকলে সেটিই best, কারণ same cluster-এর সব element previous cluster-এর সব element-এর চেয়ে বড়। Same cluster empty বা all elements \(\ge i\) হলে answer অবশ্যই largest non-empty cluster before \(c\)-এর maximum; যদি এমন cluster না থাকে, only possible smaller element হলো `V.min`।

## 9.7 Detailed proof sketch

**Claim.** `VEB-Predecessor(V,x)` correctly returns \(\operatorname{pred}_S(x)\)।

**Proof by induction on universe size \(u\).**

Base case \(u=2\): set contains elements from \(\{0,1\}\)। Direct checks return correct predecessor.

Inductive step: assume algorithm works for universe size \(\sqrt u\)। For universe \(u\):

- If \(V\) empty or \(x\le V.min\), no stored key smaller than \(x\), so `NIL` correct.
- If \(x>V.max\), all stored keys are smaller than \(x\), largest is `V.max`.
- Otherwise split \(x\) into \((c,i)\)।
  - If cluster \(c\) has minimum less than \(i\), then there exists smaller key in same cluster. Any key in same cluster is larger than every key in cluster \(<c\), so predecessor must be same cluster. By induction recursive call returns correct low part.
  - If no smaller key in same cluster, predecessor cannot be in cluster \(c\). The best cluster is largest non-empty cluster \(<c\), exactly `Predecessor(summary,c)`. By induction summary returns correct cluster ID. Its maximum gives best key inside that cluster. If no such cluster, the only stored item not represented in clusters that may be smaller is `V.min`, and since \(x>V.min\), return `V.min`.

Thus all cases are correct.

## 9.8 Time complexity

Each predecessor operation makes at most one recursive call on universe \(\sqrt u\): either into a cluster or into the summary.

\[
T(u)=T(\sqrt u)+O(1).
\]

Let \(u=2^w\). Then \(\sqrt u=2^{w/2}\). Define \(R(w)=T(2^w)\). Then:

\[
R(w)=R(w/2)+O(1)=O(\log w)=O(\log\log u).
\]

## 9.9 Space complexity

Predecessor itself uses recursion depth \(O(\log\log u)\), so call stack space \(O(\log\log u)\) if implemented recursively. Data structure space discussed later.

## 9.10 Edge cases

- Empty structure: return `NIL`.
- \(x\le V.min\): return `NIL`.
- \(x>V.max\): return `V.max`.
- Query equals a stored key: predecessor must be strictly smaller; not itself.
- Cluster missing in hash-table version: treat as empty.
- Universe not perfect square: use adjusted high/low with upper/lower square root.

## 9.11 Worked example

Let \(u=16\), \(\sqrt u=4\), and:

\[
S=\{2,3,4,8,9,12\}.
\]

Clusters by high/low:

- Cluster 0: values 0--3 → \(2,3\), low values \(2,3\)
- Cluster 1: values 4--7 → \(4\), low value \(0\)
- Cluster 2: values 8--11 → \(8,9\), low values \(0,1\)
- Cluster 3: values 12--15 → \(12\), low value \(0\)

Suppose query \(x=10\). Then:

\[
high(10)=2,
\qquad
low(10)=2.
\]

Cluster 2 has lows \(0,1\), and \(2>C.min=0\), so predecessor is in same cluster. Recursive predecessor of low 2 in cluster 2 is 1, so answer:

\[
index(2,1)=2\cdot4+1=9.
\]

Suppose query \(x=8\). Then high=2, low=0. Same cluster has no low smaller than 0. Previous non-empty cluster before 2 is cluster 1, whose max low is 0, so answer:

\[
index(1,0)=4.
\]

---

# 10. Algorithm: vEB Insert

## 10.1 Problem statement

Dynamic vEB structure-এ নতুন key \(x\) insert করতে হবে, invariants maintain করে।

## 10.2 Input and output

**Input:** vEB structure \(V\), key \(x\).  
**Output:** updated structure storing \(S\cup\{x\}\)।

## 10.3 High-level idea

Minimum element special buffer হিসেবে stored. তাই insert-এর শুরুতে:

- যদি structure empty হয়, `min=max=x`.
- যদি \(x<V.min\), তাহলে \(x\) এবং `V.min` swap করা হয়। নতুন global min buffer-এ থাকে; পুরনো min-কে recursive structure-এ insert করা হয়।
- এরপর \(x\) not global minimum, তাই তাকে তার cluster-এ insert করা যায়।
- যদি cluster previously empty হয়, cluster ID summary-তে insert করতে হবে।

## 10.4 Pseudocode

```text
VEB-Insert(V, x):
    if V is empty:
        V.min <- x
        V.max <- x
        return

    if x == V.min or x already exists:
        // set semantics: ignore duplicates
        return

    if x < V.min:
        swap x and V.min
        // Now x is the old minimum, which must be inserted recursively.

    if x > V.max:
        V.max <- x

    if V.u == 2:
        return

    c <- high(x)
    i <- low(x)

    if V.cluster[c] is empty:
        VEB-Insert(V.summary, c)
        create V.cluster[c] if needed
        VEB-Insert(V.cluster[c], i)
    else:
        VEB-Insert(V.cluster[c], i)
```

Implementation note: অনেক textbook implementation-এ empty cluster-এ `i` insert করার বদলে সরাসরি `cluster[c].min=cluster[c].max=i` করা হয়; এতে summary insert-এর পর cluster insert immediate base-like operation হয়। Lecture-এ মূল point ছিল: minimum special হওয়ায় দুটি expensive recursive calls simultaneously হয় না।

## 10.5 Correctness intuition

- `V.min` always global minimum থাকে। যদি নতুন \(x\) তার চেয়ে ছোট হয়, swap করে invariant restore করা হয়।
- নতুন \(x\) যদি global minimum না হয়, তাহলে high/low অনুযায়ী correct cluster-এ stored হয়।
- Cluster empty হলে summary-তে cluster ID insert হয়, তাই future predecessor query non-empty cluster খুঁজতে পারে।
- `V.max` update করলে large query দ্রুত answer করতে পারে।

## 10.6 Detailed proof sketch

**Claim.** Insert operation-এর পরে vEB invariants preserved থাকে।

**Proof.** Cases:

1. Empty tree হলে min=max=x, সব invariant trivially true।
2. \(x<V.min\) হলে swap-এর পরে new `V.min` is true global minimum. Old minimum আর global minimum নয়, তাই recursive representation দরকার; algorithm তাকে insert করে।
3. \(x\ge V.min\) হলে global min unchanged. If \(x>V.max\), max update correct.
4. Non-min key \(x\) exactly one cluster \(c=high(x)\)-এ low value \(i\) হিসেবে stored। Cluster empty হলে summary-তে \(c\) insert করা হয়; otherwise summary already contains \(c\)। By induction recursive insertion cluster/summary invariants maintain করে।

Thus all invariants hold.

## 10.7 Time complexity

Naively দেখতে মনে হতে পারে:

\[
T(u)\le 2T(\sqrt u)+O(1),
\]

কারণ cluster empty হলে summary-তে insert এবং cluster-এ insert—দুটি recursive call আছে। এই recurrence দিলে:

\[
T(2^w)\le 2T(2^{w/2})+O(1)
\Rightarrow T=O(w)=O(\log u),
\]

যা desired নয়।

কিন্তু এটি overly pessimistic. Cluster empty হলে cluster-এ `i` insert করা essentially empty-tree insertion; সেটি immediate return করে। তাই একই level-এ দুইটি nontrivial recursive call হয় না। Effective recurrence:

\[
T(u)=T(\sqrt u)+O(1)=O(\log\log u)=O(\log w).
\]

## 10.8 Why storing min separately matters

যদি minimum recursively stored হতো, তাহলে empty/non-empty cluster transitions-এ summary এবং cluster উভয়েই nontrivial recursive updates লাগতে পারত। That gives \(2T(\sqrt u)\)-type recurrence and \(O(\log u)\) time. Min-buffer trick ensures at most one deep recursion per level.

## 10.9 Space complexity of operation

Recursive call stack: \(O(\log\log u)\). Data structure space below।

## 10.10 Edge cases

- Duplicate set semantics: either ignore duplicates or store counts/list.
- Insert new minimum: swap mandatory.
- Insert new maximum: update max before recursion.
- Base universe \(u=2\): no further high/low decomposition needed.
- In hash-table version, cluster object is allocated lazily.

## 10.11 Worked example

Let \(u=16\). Insert sequence: \(8,3,9\).

1. Insert 8:
   - tree empty → `min=max=8`.
   - 8 is not recursively stored.

2. Insert 3:
   - \(3<min=8\), swap → `min=3`, now recursively insert old min \(8\).
   - \(high(8)=2, low(8)=0\).
   - cluster 2 empty → insert 2 into summary; insert 0 into cluster 2.
   - `max=8`.

3. Insert 9:
   - \(9>max=8\), update `max=9`.
   - \(high(9)=2, low(9)=1\).
   - cluster 2 non-empty → insert low 1 into cluster 2.

Now global min is 3, cluster 2 stores lows \(0,1\), representing values 8 and 9.

---

# 11. vEB Space Analysis

## 11.1 Naive array implementation

Each vEB node over universe \(u\) has:

- \(\sqrt u\) clusters, each a vEB over universe \(\sqrt u\);
- one summary vEB over universe \(\sqrt u\);
- min/max fields and metadata.

A more precise recurrence:

\[
S(u)=(\sqrt u+1)S(\sqrt u)+O(\sqrt u).
\]

Lecture-এর board recurrence simplified form-এ আলোচনা করে conclusion দেয়:

\[
S(u)=\Theta(u).
\]

## 11.2 Proof intuition

প্রথম level-এ \(\sqrt u\) cluster allocate করা হয়, even if empty. পরের level-এ প্রতিটি cluster আবার \(u^{1/4}\) subclusters allocate করে। Total allocated structure universe-এর possible positions cover করে। তাই stored item count \(n\) small হলেও space universe size \(u\)-এর proportional।

## 11.3 Why \(\Theta(u)\) is bad

64-bit machine হলে:

\[
u=2^{64}.
\]

\(2^{64}\) space infeasible, even if set-এ মাত্র \(n=1000\) element থাকে। তাই universe-independent or near-linear in \(n\) space চাই।

## 11.4 Common pitfall

vEB query/update fast হলেও naive version practical নয় due to \(u\)-space. The important improvement is **do not allocate empty clusters**।

---

# 12. Linear Space vEB via Dictionary/Hashing

## 12.1 Intuition

Naive vEB-এর waste হলো empty clusters. যদি কোনো cluster empty হয়, তার জন্য recursive structure allocate করার দরকার নেই। তাই array of \(\sqrt u\) clusters-এর বদলে dictionary ব্যবহার করি:

\[
\text{clusterID} \longmapsto \text{pointer to non-empty cluster}.
\]

## 12.2 Dictionary problem

### Formal definition

A dictionary stores key-value pairs \((k,v)\), where key and value fit in word.

Operations:

- `Query(k)`: return value associated with \(k\), or `NULL` if absent.
- `Insert(k,v)`: associate \(v\) with \(k\).
- Dynamic version also supports deletion.

### Required dictionary performance

Lecture-এ hashing/dictionary black box হিসেবে নেওয়া হয়েছে:

- Linear space.
- Constant worst-case query possible.
- Constant expected insertion/deletion possible; high-probability variantsও possible.

## 12.3 vEB with dictionary

Replace:

```text
V.cluster[0 .. sqrt(u)-1]
```

by:

```text
V.clusters: dictionary mapping cluster ID c -> pointer to cluster c
```

Now only non-empty clusters exist physically.

## 12.4 Space charging argument

প্রতিটি non-empty cluster dictionary-তে একটি entry তৈরি করে:

- key: cluster ID \(c\)
- value: pointer to cluster \(c\)

এই entry-র constant number of words cost cluster \(c\)-এর minimum element-এর উপর charge করা যায়। কারণ cluster non-empty হলে তার একটি minimum আছে।

Lecture-এর key observation:

- প্রতিটি element কোনো না কোনো recursive structure-এ minimum হিসেবে stored হয়।
- parent dictionary-এর cluster pointer cost সেই minimum element-কে charge করা যায়।
- প্রতিটি such minimum একটি parent level-এ একবারই charged হয়।

তাই total dictionary entries and allocated clusters stored elements-এর linear number দ্বারা bounded। Thus:

\[
S=\Theta(n)
\]

up to dictionary overhead.

## 12.5 Time impact

Dictionary lookup/insert যদি expected \(O(1)\) বা worst-case query \(O(1)\) হয়, তাহলে vEB recurrence unchanged:

\[
T(u)=T(\sqrt u)+O(1)=O(\log\log u).
\]

## 12.6 Why randomization appears

Hashing সাধারণত randomization ব্যবহার করে expected or high-probability constant time guarantee দেয়। তাই lecture-এ বলা হয়েছে naive \(u\)-space vEB linear space করা যায় randomization সহ। Deterministic linear-space dynamic dictionary-এর exact status lecture-এ বিস্তারিত করা হয়নি।

## 12.7 Common pitfalls

- Summary structure নিজেও vEB; সেটিকেও sparse/dictionary style implement করতে হয়।
- Cluster missing মানে cluster empty, but summary must stay consistent.
- Hash table expected time হলে whole data structure update time expected/amortized হতে পারে, depending on dictionary.

---

# 13. X-fast Trie: Another route to \(O(\log\log u)\)

Lecture-এর শেষ অংশে vEB-এর alternative historical viewpoint হিসেবে X-fast/Y-fast tries sketch করা হয়।

## 13.1 Starting point: bit array of universe

If we do not care about space, predecessor can be represented by bit array \(B[0..u-1]\):

\[
B[i]=
\begin{cases}
1 & \text{if } i\in S,\\
0 & \text{otherwise.}
\end{cases}
\]

Predecessor query naively scan left from \(x-1\), worst-case \(O(u)\)।

## 13.2 Add a binary tree with OR values

Build a perfect binary tree over leaves \(0,
\dots,u-1\). Each internal node stores OR of its children:

\[
node.bit = left.bit \lor right.bit.
\]

Thus a node bit is 1 iff its subtree contains at least one stored element।

Also store all present elements (leaves with bit 1) in a doubly linked list in sorted order।

## 13.3 Predecessor query intuition

For a query leaf \(x\):

- If \(x\in S\), predecessor is previous node in the linked list.
- If \(x\notin S\), go upward until finding the first ancestor whose bit is 1.

On the path from leaf to root, bits are monotone:

\[
0,0,0,\dots,1,1,\dots,1.
\]

Once a subtree contains a stored element, all higher ancestors also contain one।

Suppose the first 1 ancestor is \(a\), and the child on the path to \(x\) is \(p\). Since \(p\) had bit 0, the 1 must come from sibling subtree.

- If \(p\) is the right child, sibling is left subtree, whose elements are all \(<x\). Return maximum element in that sibling subtree.
- If \(p\) is the left child, sibling is right subtree, whose minimum is successor of \(x\). Then predecessor is previous element in doubly linked list.

## 13.4 From \(O(\log u)\) to \(O(\log\log u)\) using binary search

Tree height is:

\[
\log u = w.
\]

Going upward one-by-one costs \(O(w)\). But path bits are monotone, so binary search finds first 1 in:

\[
O(\log w)=O(\log\log u).
\]

## 13.5 How can binary search access ancestors quickly?

If tree is stored as array like a binary heap:

- root at index 0;
- node \(v\)'s left child: \(2v+1\);
- right child: \(2v+2\).

Then ancestors can be computed arithmetically. Conceptually, moving \(k\) levels up corresponds to dropping \(k\) low-order path bits, achievable by division by \(2^k\), i.e., right shift. Exact formula depends on whether using 0-indexed heap indices or 1-indexed representation, but word-RAM supports this in \(O(1)\).

Alternative: each node could store pointers to \(2^j\)-th ancestors, but that increases space.

## 13.6 X-fast space reduction: store only 1-nodes

The full OR tree has \(O(u)\) nodes. To reduce space, only nodes whose bit is 1 are stored in hash tables.

For each level \(\ell\), maintain a dictionary containing positions of 1-nodes at that level.

To test whether a node is 1 during binary search:

- lookup node ID in level dictionary;
- hit means bit 1;
- miss means bit 0.

At each level, at most \(n\) nodes can be 1. There are \(w=\log u\) levels. So:

\[
S=O(nw).
\]

This structure is called an **X-fast trie**.

## 13.7 X-fast complexities

- Query: \(O(\log w)=O(\log\log u)\)
- Space: \(O(nw)\)
- Update: can cost \(O(w)\), because inserting a leaf may require setting/updating one node on every level.

## 13.8 Common pitfalls

- X-fast trie is not yet linear space; it has an extra factor \(w\)।
- Query time fast because of binary search over levels, not because tree height shrank.
- Hash tables store prefixes / node positions representing occupied subtrees, not the original keys only.

---

# 14. Y-fast Trie: Indirection to remove the \(w\) factor

## 14.1 Intuition

X-fast trie has good query time but bad space \(O(nw)\). Trick: do not put all \(n\) elements into X-fast trie. Put only about \(n/w\) representatives. Each representative stands for a bucket or “super item” of \(\Theta(w)\) real elements.

This technique is called **indirection**.

## 14.2 Structure

Partition sorted set \(S\) into buckets:

\[
B_1,B_2,\dots,B_m,
\]

where:

\[
|B_j|=\Theta(w),
\qquad
m=\Theta(n/w).
\]

For each bucket, choose a representative, e.g., maximum or minimum. Store representatives in an X-fast trie. Store the elements inside each bucket in a balanced BST.

## 14.3 Space analysis

X-fast trie on \(m=n/w\) representatives uses:

\[
O(mw)=O((n/w)w)=O(n)
\]

space. Buckets store all elements once:

\[
O(n).
\]

Total:

\[
O(n).
\]

## 14.4 Query algorithm

To answer predecessor \(x\):

1. Use X-fast trie over representatives to find the bucket whose range may contain \(x\) or its predecessor.
2. Search inside the corresponding balanced BST bucket.
3. If needed, check adjacent bucket boundary.

## 14.5 Query time

X-fast search:

\[
O(\log w).
\]

Balanced BST inside bucket of size \(O(w)\):

\[
O(\log w).
\]

Total:

\[
O(\log w)=O(\log\log u).
\]

## 14.6 Update idea

Insert \(x\):

1. Find appropriate bucket using top X-fast trie.
2. Insert into bucket BST: \(O(\log w)\).
3. If bucket size exceeds \(2w\), split it into two buckets of size about \(w\), choose representatives, update X-fast trie.

Because split happens only after \(\Theta(w)\) insertions into a bucket, the expensive top-level update is amortized over many operations. Lecture only sketches this part; full Y-fast trie details require careful randomized bucket management.

## 14.7 Why it matters

Indirection is a widely used data-structure technique:

- expensive structure on fewer representatives;
- cheap local structures on small buckets;
- rebuild/split buckets occasionally;
- amortize expensive global maintenance.

Y-fast trie is a canonical example where indirection removes an unwanted multiplicative factor in space and update time.

## 14.8 Common pitfalls

- Bucket size must be controlled: too large → local search slow; too small → too many representatives.
- Top-level X-fast stores representatives, not every element.
- Updates are amortized/sketched; a fully rigorous implementation is more subtle than the lecture sketch.

---

# 15. Theorem-style summary and proofs

## Theorem 1: vEB predecessor query time

### Statement

For universe size \(u\), vEB predecessor query runs in:

\[
O(\log\log u)
\]

time, assuming word-RAM constant-time high/low/index operations and constant-time cluster access.

### Assumptions

- \(u=2^{2^k}\) or rounded decomposition used.
- Cluster access is \(O(1)\): array or dictionary with constant-time lookup.
- Recursive invariant maintained.

### Proof intuition

Each query reduces universe from \(u\) to \(\sqrt u\), and does only constant work outside recursion.

### Detailed proof

The recurrence:

\[
T(u)=T(\sqrt u)+O(1).
\]

Let \(u=2^w\). Then:

\[
T(2^w)=T(2^{w/2})+O(1).
\]

Define \(R(w)=T(2^w)\). Then:

\[
R(w)=R(w/2)+O(1).
\]

After \(k\) levels, word-size parameter becomes \(w/2^k\). Stop when constant:

\[
\frac{w}{2^k}=O(1)
\Rightarrow
k=O(\log w).
\]

Thus:

\[
T(u)=O(\log w)=O(\log\log u).
\]

### Consequence

If \(u=2^w\), query time is \(O(\log w)\), sublogarithmic in universe size.

---

## Theorem 2: vEB insertion time with min-buffer trick

### Statement

vEB insertion runs in \(O(\log\log u)\) time.

### Assumptions

- Minimum is stored only in `V.min`, not recursively.
- Empty cluster insertion is constant at that recursive level or causes only shallow recursion.
- Dictionary/array access is constant time.

### Proof intuition

Although insertion may appear to update both summary and cluster, only one of those recursions is nontrivial due to the special minimum handling.

### Detailed proof

At each level:

- If cluster already non-empty, recurse into cluster only.
- If cluster empty, insert cluster ID into summary. Then inserting low value into empty cluster immediately sets min/max and returns.

Thus each level has one deep recursive call:

\[
T(u)=T(\sqrt u)+O(1)=O(\log\log u).
\]

### Consequence

vEB supports dynamic predecessor with fast updates, not just static queries.

---

## Theorem 3: Naive vEB space

### Statement

Naive array-based vEB tree uses \(\Theta(u)\) space.

### Assumptions

- Every cluster array entry is allocated recursively, regardless of whether it stores elements.
- Universe decomposition by square roots.

### Proof intuition

The recursive cluster arrays cover the entire universe tree, so space scales with universe size, not number of stored elements.

### Detailed proof sketch

Recurrence:

\[
S(u)=(\sqrt u+1)S(\sqrt u)+O(\sqrt u).
\]

This solves to \(\Theta(u)\). Intuitively, at recursive levels the number of possible subproblems expands enough to represent all universe positions. Even empty regions are allocated.

### Consequence

Naive vEB impractical for large word sizes such as 64-bit universes.

---

## Theorem 4: Sparse vEB with dictionary uses linear space

### Statement

If only non-empty clusters are allocated and cluster pointers are stored in linear-space dictionaries, vEB uses \(O(n)\) space for \(n\) stored elements.

### Assumptions

- Dictionary overhead is linear in number of stored key-value pairs.
- Only non-empty clusters have allocated recursive structures.
- Summary structures are also sparse.

### Proof intuition

Each allocated cluster exists because it contains at least one element. Charge the constant overhead of its parent dictionary entry to the minimum element in that cluster. Each element is charged only constant number of times across the recursive representation.

### Detailed proof sketch

For every dictionary entry \((c, pointer)\) in a vEB node, cluster \(c\) is non-empty. Let \(m_c\) be the minimum element of that cluster. Charge entry cost to \(m_c\). Since each recursive minimum belongs to one parent cluster, it receives one such charge. Across all recursive structures, the number of minima is proportional to the number of represented elements, so total allocated dictionary entries and cluster nodes is \(O(n)\).

### Consequence

vEB’s fast \(O(\log\log u)\) operations can be combined with near-linear space, typically with randomized hashing.

---

## Theorem 5: Combining vEB/Y-fast and fusion-tree bounds

### Statement

Given word size \(w\ge\log n\), choosing between vEB/Y-fast and fusion-tree style data structures gives predecessor query time:

\[
O\left(\min\left\{\log w,\frac{\log n}{\log w}\right\}\right)
\le O(\sqrt{\log n}).
\]

### Proof intuition

One bound improves when \(w\) is small; the other improves when \(w\) is large. The worst crossover is when they are equal.

### Detailed proof

Let:

\[
a=\log w,
\qquad
b=\frac{\log n}{\log w}=\frac{\log n}{a}.
\]

We need bound \(\min\{a,b\}\). If \(a\le\sqrt{\log n}\), then min is at most \(a\le\sqrt{\log n}\). If \(a>\sqrt{\log n}\), then:

\[
b=\frac{\log n}{a}<\sqrt{\log n}.
\]

Thus always:

\[
\min\{a,b\}\le\sqrt{\log n}.
\]

### Consequence

Word-RAM predecessor can beat balanced BST’s \(O(\log n)\) time under standard assumptions.

---

# 16. Transcript-এর board derivations reconstructed

## 16.1 vEB recursion depth

Sequence:

\[
u, u^{1/2}, u^{1/4}, u^{1/8},\dots
\]

After \(t\) levels:

\[
u^{1/2^t}.
\]

Stop when this is constant, say 2:

\[
u^{1/2^t}=2.
\]

Taking log:

\[
\frac{\log u}{2^t}=1
\Rightarrow
2^t=\log u
\Rightarrow
 t=\log\log u.
\]

## 16.2 Insert naive vs actual recurrence

Naive pessimistic:

\[
T(u)=2T(\sqrt u)+O(1).
\]

Let \(u=2^w\):

\[
R(w)=2R(w/2)+O(1)=O(w)=O(\log u).
\]

Actual due to min trick:

\[
T(u)=T(\sqrt u)+O(1)=O(\log\log u).
\]

## 16.3 Space recurrence intuition

Naive:

\[
S(u)=(\sqrt u+1)S(\sqrt u)+O(\sqrt u)=\Theta(u).
\]

Sparse:

\[
S(n)=O(n)
\]

because empty clusters not allocated.

## 16.4 X-fast path monotonicity

For a leaf \(x\notin S\), the path to root has node bits:

\[
0,0,\dots,0,1,1,\dots,1.
\]

Binary search on this monotone sequence gives first 1 in \(O(\log w)\)।

---

# 17. যেসব জায়গায় ট্রান্সক্রিপ্ট অস্পষ্ট

1. **“You can't n numbers faster than n log n”** — এখানে transcription missing word আছে। সম্ভাব্য অর্থ: “You can't sort \(n\) numbers faster than \(n\log n\).” Lecture context অনুযায়ী এটি comparison sorting lower bound।

2. **vEB cluster universe line:** transcript-এ “square root 2u” ধরনের phrase এসেছে, কিন্তু mathematical context থেকে intended meaning \(\sqrt u\) universe size।

3. **Predecessor pseudocode condition:** board transcription-এ `cluster C Min is less than x` বলা হয়েছে, কিন্তু formal algorithm-এ compare করতে হয় `low(x)` with cluster minimum. অর্থাৎ condition হওয়া উচিত \(i>C.min\)।

4. **If previous cluster not found:** lecture board sketch দ্রুত হয়েছে। Formal predecessor algorithm-এ summary predecessor `NIL` হলে global `V.min` return করতে হয়, provided \(x>V.min\)।

5. **Hashing guarantee:** lecture dictionary problem black box হিসেবে নেয়। Exact reference এবং deterministic/randomized guarantee বিস্তারিত করা হয়নি। Notes-এ expected/high-probability wording ব্যবহার করা হয়েছে।

6. **Y-fast trie details:** lecture শেষে সময়ের কারণে sketch দিয়েছে। Full update rules, representative choice, split/merge invariant rigorousভাবে বলা হয়নি। Notes-এ standard high-level interpretation দেওয়া হয়েছে, কিন্তু full implementation proof omitted।

7. **Historical references:** lecture-এ years/conference names oralভাবে বলা হয়েছে; notes-এ transcript অনুযায়ী রাখা হয়েছে, external verification করা হয়নি।

---

# 18. Key takeaways

1. \(\Omega(n\log n)\) sorting lower bound শুধু comparison model-এর জন্য।
2. Word-RAM model integer bit operations exploit করে faster algorithms সম্ভব করে।
3. Predecessor problem ordered search-এর core abstraction।
4. Dynamic predecessor data structure sorting algorithm তৈরি করতে পারে।
5. vEB tree universe square-root recursion দিয়ে \(O(\log\log u)=O(\log w)\) query/update দেয়।
6. vEB insertion দ্রুত হওয়ার critical trick: global minimum separately store করা।
7. Naive vEB \(\Theta(u)\) space নেয়; sparse clusters + dictionary দিয়ে \(O(n)\) space পাওয়া যায়।
8. X-fast trie bit-tree এবং binary search over levels ব্যবহার করে \(O(\log w)\) query দেয়, কিন্তু \(O(nw)\) space নেয়।
9. Y-fast trie indirection/bucketing দিয়ে X-fast-এর \(w\) space factor দূর করে।
10. Fusion trees complementary bound \(O(\log_w n)\) দেয়; vEB/Y-fast + fusion মিলিয়ে \(O(\sqrt{\log n})\)-type predecessor bound পাওয়া যায়।

---

# 19. Important definitions

## Predecessor

\[
\operatorname{pred}_S(z)=\max\{x\in S:x<z\}.
\]

## Static data structure

Preprocessed set fixed; updates নেই।

## Dynamic data structure

Query ছাড়াও insert/delete support করে।

## Word-RAM

A model where word-sized integers and pointers fit in \(w\) bits and word-level arithmetic/bit operations take constant time.

## Universe size

\[
u=2^w.
\]

## high/low decomposition

\[
high(x)=\left\lfloor\frac{x}{\sqrt u}\right\rfloor,
\quad
low(x)=x\bmod \sqrt u,
\quad
index(c,i)=c\sqrt u+i.
\]

## Summary structure

vEB node-এর recursive data structure that stores IDs of non-empty clusters.

## Dictionary problem

Key-value pairs store করে constant-time lookup/update support করার data-structural problem.

## X-fast trie

Trie/OR-tree representation where only occupied prefixes are stored in hash tables by level; predecessor query binary searches levels.

## Y-fast trie

X-fast trie over representatives plus balanced BST buckets of size \(\Theta(w)\), using indirection to obtain linear space.

## Indirection

A data-structure design technique: global expensive structure stores representatives; small local structures store actual items.

---

# 20. Important algorithms/theorems

## Algorithms

1. Static predecessor by sorted array + binary search.
2. Dynamic predecessor by balanced BST.
3. Sorting via dynamic predecessor.
4. vEB predecessor.
5. vEB insert.
6. Sparse vEB with dictionary-backed clusters.
7. X-fast predecessor via binary search over ancestor path.
8. Y-fast predecessor via representatives + buckets.

## Theorems/claims

1. Comparison sorting requires \(\Omega(n\log n)\) comparisons.
2. vEB predecessor time: \(O(\log\log u)\).
3. vEB insert time: \(O(\log\log u)\) using min-buffer trick.
4. Naive vEB space: \(\Theta(u)\).
5. Sparse vEB space: \(O(n)\) with dictionary/hash table.
6. X-fast query: \(O(\log w)\), space \(O(nw)\).
7. Y-fast query: \(O(\log w)\), space \(O(n)\).
8. Fusion-tree query bound: \(O(\log_w n)\).
9. Combined predecessor bound:

\[
O\left(\min\left\{\log w,\frac{\log n}{\log w}\right\}\right)
\le O(\sqrt{\log n}).
\]

---

# 21. Formula summary

| Concept | Formula |
|---|---:|
| Universe size | \(u=2^w\) |
| Word size lower assumption | \(w\ge \log n\) |
| Predecessor | \(\max\{x\in S:x<z\}\) |
| vEB decomposition | \(x=c\sqrt u+i\) |
| high | \(\lfloor x/\sqrt u\rfloor\) |
| low | \(x\bmod\sqrt u\) |
| index | \(c\sqrt u+i\) |
| vEB recurrence | \(T(u)=T(\sqrt u)+O(1)\) |
| vEB time | \(O(\log\log u)=O(\log w)\) |
| naive insert recurrence | \(T(u)=2T(\sqrt u)+O(1)=O(\log u)\) |
| naive vEB space | \(S(u)=\Theta(u)\) |
| sparse vEB space | \(O(n)\) |
| X-fast space | \(O(nw)\) |
| Y-fast bucket size | \(\Theta(w)\) |
| Fusion tree query | \(O(\log_w n)=O(\log n/\log w)\) |
| Combined bound | \(O(\min\{\log w,\log n/\log w\})\le O(\sqrt{\log n})\) |

---

# 22. Exam-style questions

## Conceptual questions

1. “Sorting requires \(\Omega(n\log n)\)” statementটি কেন incomplete? Model specify করে correct statement লেখো।
2. Word-RAM model comparison model-এর চেয়ে stronger কেন?
3. Static এবং dynamic predecessor problem-এর মধ্যে পার্থক্য কী?
4. Dynamic predecessor data structure দিয়ে sorting কীভাবে করা যায়?
5. vEB tree কেন universe size \(u\)-এর উপর recursive, \(n\)-এর উপর নয়?

## Algorithm design questions

6. vEB tree-এর `Predecessor` pseudocode লেখো এবং প্রতিটি case explain করো।
7. vEB insertion-এ `min` separately store করার প্রয়োজন কেন? Counterexample বা recurrence দিয়ে বোঝাও।
8. Sparse vEB implement করতে dictionary কীভাবে ব্যবহার করবে?
9. X-fast trie query-তে binary search কোন monotone property-এর উপর করা হয়?
10. Y-fast trie কীভাবে X-fast trie-এর \(O(nw)\) space কমিয়ে \(O(n)\) করে?

## Proof questions

11. Prove \(T(u)=T(\sqrt u)+O(1)\Rightarrow T(u)=O(\log\log u)\)।
12. Naive vEB space কেন \(\Theta(u)\)? Recurrence solve বা intuition দাও।
13. Sparse vEB linear space charging argument formalize করো।
14. Show:

\[
\min\left\{\log w,\frac{\log n}{\log w}\right\}\le \sqrt{\log n}.
\]

15. vEB predecessor correctness induction proof লেখো।

## Worked calculation questions

16. \(u=256\) হলে vEB recursion universe sizes কী কী? Depth কত?
17. \(u=16\), \(S=\{1,4,6,9,10,15\}\)। vEB high/low decomposition দেখিয়ে \(pred(11)\), \(pred(9)\), \(pred(0)\) বের করো।
18. Word size \(w=64\) হলে vEB query asymptotically কী? \(\log w\) কত approximately?
19. যদি \(w=\log n\), fusion tree query bound কত? vEB bound কত?
20. যদি \(w=2^{\sqrt{\log n}}\), combined bound evaluate করো।

---

# 23. Research-level intuition and further thinking

## 23.1 Data structure lower bounds depend on model

Predecessor problem-এর জন্য cell-probe/word-RAM lower bounds একটি গভীর research area. Lecture-এর শেষে ইঙ্গিত দেওয়া হয়েছে যে vEB/fusion tree style bounds nearly optimal for linear or near-linear space predecessor structures. এর মানে শুধু clever algorithm নয়—model, word size, space bound, randomization—সব একসঙ্গে lower bound নির্ধারণ করে।

## 23.2 Why predecessor is harder than dictionary

Dictionary শুধু exact key lookup করে। Predecessor order-sensitive: key absent হলেও nearest smaller key খুঁজতে হয়। Hashing exact lookup সহজ করে, কিন্তু order preserve করে না। vEB/X-fast/Y-fast structures hashing এবং order decomposition combine করে।

## 23.3 Universe recursion vs branching factor

vEB square-root recursion একটি unusual divide-and-conquer:

- binary search reduces candidate count by half → \(\log n\);
- vEB reduces bit-length by half → \(\log w=\log\log u\)।

এটি bit-level algorithms-এর power দেখায়।

## 23.4 Indirection as a general paradigm

Y-fast trie-এর bucket idea অনেক data structure-এ দেখা যায়:

- B-trees;
- exponential search trees;
- fusion-tree based structures;
- cache-oblivious structures;
- dynamic graph/data structure sparsification.

The pattern: expensive global operation কমানোর জন্য representative layer + local small structures।

## 23.5 Sorting open question

Lecture-এ বলা হয়েছে word-RAM integer sorting linear time-এ সম্ভব কি না open question. এই ধরনের question দেখায় যে even basic problem sorting model-sensitive হলে research frontier তৈরি হয়।

---

# 24. কী কী revise করা উচিত

1. Comparison sorting lower bound decision tree proof.
2. Binary search tree predecessor operation.
3. Word-RAM model: word size, universe, bitwise operations.
4. \(u=2^w\), \(\log\log u=\log w\) conversion.
5. Square-root decomposition: high/low/index functions.
6. vEB invariants: min, max, summary, clusters.
7. vEB predecessor algorithm and correctness.
8. vEB insertion algorithm and min-buffer trick.
9. Recurrence solving: \(T(u)=T(\sqrt u)+O(1)\).
10. Naive vs sparse vEB space.
11. Dictionary/hashing black box and why it gives linear space.
12. X-fast trie: OR tree, monotone path, binary search over levels.
13. Y-fast trie: indirection, buckets, representatives.
14. Fusion tree bound and combined \(O(\sqrt{\log n})\) argument.

---

# 25. One-page compressed revision sheet

## Problems

- Predecessor: \(pred_S(z)=\max\{x\in S:x<z\}\)
- Dynamic predecessor supports insert/delete.

## Models

- Comparison model → sorting lower bound \(\Omega(n\log n)\).
- Word-RAM → keys are \(w\)-bit integers, \(u=2^w\), bit operations constant time.

## vEB

- Split \(x\) into \((c,i)\), each in \([0,\sqrt u-1]\).
- Store non-min elements in clusters; store non-empty cluster IDs in summary.
- `min` stored separately.
- Query/update recurrence: \(T(u)=T(\sqrt u)+O(1)\).
- Time: \(O(\log\log u)=O(\log w)\).
- Naive space: \(\Theta(u)\).
- Sparse dictionary space: \(O(n)\).

## X-fast/Y-fast

- X-fast: store occupied prefixes/1-nodes in hash tables by level.
- Query: binary search monotone leaf-root path → \(O(\log w)\).
- Space: \(O(nw)\).
- Y-fast: X-fast over \(n/w\) representatives + buckets of size \(\Theta(w)\).
- Space: \(O(n)\), query \(O(\log w)\).

## Fusion comparison

- Fusion tree query: \(O(\log_w n)=O(\log n/\log w)\).
- Best of both:

\[
O\left(\min\left\{\log w,\frac{\log n}{\log w}\right\}\right)
\le O(\sqrt{\log n}).
\]

---

# 26. Suggested self-test

নিচের প্রশ্নগুলোর উত্তর না দেখে লিখে ফেলতে পারলে lecture-এর core বোঝা হয়ে যাবে।

1. vEB predecessor algorithm-এ কেন same cluster আগে check করা হয়?
2. Summary structure exactly কী store করে?
3. Insert-এ `min` swap না করলে কী invariant ভাঙবে?
4. \(2T(\sqrt u)\) recurrence কোথা থেকে আসে এবং কেন actual recurrence নয়?
5. Sparse vEB linear space proof-এ কোন object-কে কোন element-এর উপর charge করা হয়?
6. X-fast trie-তে leaf-root path monotone কেন?
7. Y-fast trie bucket size \(\Theta(w)\) না হয়ে \(\Theta(\log n)\) হলে কী সমস্যা/পরিবর্তন হতো?
8. \(w\ge\log n\) assumption কোথা থেকে আসে?
9. Dynamic predecessor দিয়ে sorting করলে duplicate কীভাবে handle করবে?
10. vEB এবং fusion tree কোন \(w\)-regime-এ ভালো?

---

**End of lecture note.**
