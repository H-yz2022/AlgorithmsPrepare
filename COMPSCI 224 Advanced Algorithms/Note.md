# Intro
 SCRIBING 10%.
 P sets 60%.
 Final Project 30%.

- Goals: 
  - Increased ability to analyze and algorithms.
  - Looking at different models within which to analyze algorithms.

 ## Word RAM Model
never faster than nlog(n)

 ### Static predecessor
 - data structure represents set S of items {x<sub>1</sub>,...x<sub>n</sub>}
 - Query perd(x)=max{x∈S:x<z};predecessor of z is the maximum of x and S
 - want low space, fast query
 - Stative vs Dynamic: no insertion in static; dynamic allows insertions
 - Binary search tree.
   + Example solution: Store numbers sorted, do binary search(static)
   +  O(lgn) dynamic query using balanced BST
   + Comparison-based sorting
  
 ### Word RAM Model
never faster than nlog(n)
 - items are integers in {0,1,...,2^w-1}
 - w="word size", u=2^w-1
 - also assume that pointers fit in a word
 - space>=n
 - w>= lg(space)>=lgn.

### Two data structures
1. Van Emde Boos tree (FOCS 1975), Reference(...):
    - update /query: θ($\log w$) time, θ(u) space(can be made θ(n) w/randomization
    - yfast tries: same bounds
2. Fuston trees (Fredman, Willard Jcss 1993)
    - query in time: θ($\log_w n$)time.
   ➜ achieve min{log w$ , $\log_w n$} <= $O(\sqrt{\log n})$.
   ➜ (w/ dynamic fusion trees) $O(\sqrt{n})$ sorting.
   - Faster sorting
      + $O(lg lg n)$ delete. (Itan, STOC 2002)
      + $O(\sqrt{lg lg n})$ random time (Han,Thomp FOCS 2002)
    
 ### Word RAM Model (continue)
 Assume that given x,y fitting in a word each, we can do:.
     + / * - ~ ^ | $ >>  <<   in constant time.
 
 ### Van Emde Boos tree (vEB tree)
 vEB tree defined recursively
  - Triangle vEB_u tree bottom has $\sqrt{n}$ vEB_($\sqrt{n}$), the top have onevEB_(\sqrt{n}) and one minimum element in the tip.
 <!--图片 ![cat](https://example.com/cat.png)-->
Fields of vEB_u (V).
 - (\sqrt{n}) site array V.cluster[0], ..., V.cluster[($\sqrt{n}$)-1] for  V.cluster[] means for vEB(\sqrt{u}) data structure
 - V.summary is a vEB(\sqrt{u}) instance
 - V.min/V.max are integers in {0,...,u-1}
 - x∈{0,...,u-1},
 - Write x in binary x=10010011, where 1001 is c and 001 is i, then x=<c,i>, c,i∈{0,...,u-1},
 - How to do a query for x? 按位与和位移运算在常数时间提取c, i
 - How to search for the predecessor of x in this recursive data structure? 1)insert i into the cluster c； if the c cluster happened to be empty, insert c into the summary. The summary keeps track of which is non-empty.
2)look for the minimum element in c cluster; if mine is bigger, then my predecessor lives in the same cluster. Therefore, recursively do a predecessor in i cluster; if it is empty or bigger than or equal to min, then my predecessor does not live in the same cluster, it lives in the largest cluster before me that is non-empty. Therefore, do a predecessor in c cluster in the summary and return a max in that cluster.

``` C
pred (V, x=<c,i>)
if x > V.max: return V.amx
else if V.cluster[c].min< x:
    return pred(V.cluster[c],i)
else:
    c'=pred(V.summary,c)
   return V.cluster[c'].max
```
for this, only have one recursive call:
pred time T(u)=T(($\sqrt{n}$)+O(1).
         ➜T(u)=O(lg lg u)
``` python
insert(V,x=<c,i>)
if v=∅：
    V.min∈x', return
if x< V.min:
    swap(x,V.min)
if V.cluster[c].min=∅
    insert(V.summary,c)
insert(V.cluster[c].i)
```
insertion time T(u)<= 2*T(($\sqrt{u}$)+O(1).
               T(w)<= 2*T(w/2)+O(1).
 However, 2 is too pessimistic:
               T(u)<= 1*T(($\sqrt{u}$)+O(1).
               T(w)<= 1*T(w/2)+O(1).
                ➜T(u)=O(lg lg u).

Space of vEB
S(u)=($\sqrt{u}$+1)+ $O(1)$
➜S(u)=θ(u)



