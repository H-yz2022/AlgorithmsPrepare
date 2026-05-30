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
 - w>= lg(space)>=lgn

### Two data structures
1. Van Emde Boos tree (FOCS 1975), Reference(...):
    - update /query: θ($\log w$) time, θ(u) space(can be made θ(n) w/randomization
    - yfast tries: same bounds
2. Fuston trees (Fredman, Willard Jcss 1993)
    - query in time: θ($\log_w n$)time
   ➜ achieve min{log w$ , $\log_w n$} <= $O(\sqrt{\log n})$
   ➜ (w/ dynamic fusion trees) $O(\sqrt{n})$ sorting
   - Faster sorting 

 
 

