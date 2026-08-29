# Lab1 in MOOC
[Link in Gemini](https://share.gemini.google/Q7zx5ucLFWol)
[Link in Github](https://github.com/yusong-shen/mooc_os_lab)

平台使用是Oracler软件设置 虚拟机内 Linux Ubuntu

## 前期准备需要的部分terminal 中的指令：

### Exercise




## 遇到的问题

## 知识点总结


### Problem in lab 1 challenges
``` bash
+++ switch to user mode +++
trapframe at 0x7b2c
  ...
  ds   0x----0010
  es   0x----0010
  cs   0x----0008
  trap 0x00000078 (unknown trap)
```


# Lab2 in MOOC
## 前期准备需要的部分terminal 中的指令：
``` bash
cd ../lab2

cgit checkout lab2
```

``` bash
grep -rn "YOUR CODE" .
```

``` YOUR CODE 
./kern/mm/default_pmm.c:12:// LAB2 EXERCISE 1: YOUR CODE

./kern/mm/pmm.c:350: /* LAB2 EXERCISE 2: YOUR CODE

./kern/mm/pmm.c:403: /* LAB2 EXERCISE 3: YOUR CODE

./kern/debug/kdebug.c:296: /* LAB1 YOUR CODE : STEP 1 */

./kern/trap/trap.c:37: /* LAB1 YOUR CODE : STEP 2 */

./kern/trap/trap.c:144: /* LAB1 YOUR CODE : STEP 3 */

./kern/trap/trap.c:159: //LAB1 CHALLENGE 1 : YOUR CODE you should modify below codes. 


```


``` bash
cd ../lab2
```
``` bash
make clean
make grade
make qemu
```


## 

- 同步/合并 Lab 1 的代码
- 更改your code部分
- 
1. 练习 1：实现 Best-Fit（或 First-Fit）连续物理内存分配算法
    + 主要文件：kern/mm/default_pmm.c
    + 任务目标：根据提示实现物理页面的分配与释放逻辑（包括 default_init, default_init_memmap, default_alloc_pages, default_free_pages 等接口）。
    + 切入点：阅读并理解 default_pmm.c 中的函数注释，掌握空闲链表（free_list）的管理方式。
```
nano kern/mm/default_pmm.c
```
- LAB2 EXERCISE 1: YOUR CODE  you should rewrite functions: default_init, default_init_memmap, default_alloc_pages, default_free_pages.
- 算法规则（First-Fit）
- 合并效率$O(N)$ 遍历且容易出现遗漏$O(1)$ 常数时间检查前后的 prev/next



2. 练习 2：实现寻找虚拟地址对应的页表项（Page Table Entry）
    + 主要文件：kern/mm/pmm.c
    + 找到位置：第 350 行附近的 get_pte() 函数。
    + 任务目标：根据给定的页表基址和虚拟地址，建立或查找二级页表结构，返回对应的 PTE 指针。需要注意内存分配失败以及页表项存在位的处理。

3. 练习 3：释放某虚地址所在的页并取消对应页表项映射
    + 主要文件：kern/mm/pmm.c
    + 找到位置：第 403 行附近的 page_remove_pte() 函数。
    + 任务目标：解除物理页与虚拟地址的映射关系，并在引用计数归零时释放该物理页，同时刷洗 TLB（页表缓存）。

## 遇到的问题

## 知识点总结


