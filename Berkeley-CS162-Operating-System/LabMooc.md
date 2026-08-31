# Lab1 in MOOC
[Link in Gemini](https://share.gemini.google/Q7zx5ucLFWol)
[Link in Github](https://github.com/yusong-shen/mooc_os_lab)

平台使用是Oracler软件设置 虚拟机内 Linux Ubuntu

## 前期准备需要的部分terminal 中的指令：
1. VirtualBox 双向剪贴板
2. 使用镜像
3. 保存并提交代码

```
git clone https://github.com/yusong-shen/mooc_os_lab.git
```
```
cd ~/mooc_os_lab/labcodes/lab1
git add .
git commit -m "complete lab1 code"
```
## Exercise Code
```
lab1 中包含一个bootloader 和一个OS。这个bootloader可以切换到X86保护模式，能够读磁盘

并加载ELF执行文件格式，并显示字符。而这lab1中的OS只是一个可以处理时钟中断和显示字符

的幼儿园级别OS。
```

```
cd mooc_os_lab/labcodes/lab1
ls
grep -rn "YOUR CODE" .
make grade
nano kern/trap/trap.c
gedit kern/trap/trap.c &
```
### Step 1 打印函数调用栈部分

```
void
print_stackframe(void) {
     /* LAB1 YOUR CODE : STEP 1 */
     // 1. 读取当前的 ebp 和 eip 寄存器值
     uint32_t ebp = read_ebp();
     uint32_t eip = read_eip();

     int i, j;
     // 3. 最多遍历 STACKFRAME_DEPTH (20) 层栈帧，且当 ebp 为 0 时终止（到达最外层）
     for (i = 0; i < STACKFRAME_DEPTH && ebp != 0; i++) {
         // (3.1) 打印当前栈帧的 ebp 和 eip
         cprintf("ebp:0x%08x eip:0x%08x ", ebp, eip);

         // (3.2) 打印函数的 4 个参数
         // 参数存放在 [ebp + 8] 开始的内存区域，即 (uint32_t *)ebp + 2 偏移处
         uint32_t *args = (uint32_t *)ebp + 2;
         cprintf("args:");
         for (j = 0; j < 4; j++) {
             cprintf("0x%08x ", args[j]);
         }
         cprintf("\n");

         // (3.4) 打印源码中的函数名、文件名及行号信息
         // 注意：传入 eip - 1 是为了防止 eip 正好指向指令边界而定位到下一行代码
         print_debuginfo(eip - 1);

         // (3.5) 弹出当前栈帧，更新 eip 和 ebp 以追踪上一级函数
         // [ebp + 4] 存放的是返回地址 (eip)
         // [ebp] 存放的是上一级函数的帧指针 (ebp)
         eip = ((uint32_t *)ebp)[1];
         ebp = ((uint32_t *)ebp)[0];
     }
}
```



### Step 2 修改 idt_init 函数中断处理逻辑部分

```
/* idt_init - initialize IDT to each of the entry points in kern/trap/vectors.S */
void
idt_init(void) {
     /* LAB1 YOUR CODE : STEP 2 */
     // 1. 声明外部变量 __vectors，它保存了 vectors.S 中所有 ISR 的入口地址
     extern uintptr_t __vectors[];

     // 2. 遍历并填充 IDT 表项 (共 256 项)
     // 参数说明：SETGATE(gate, istrap, sel, off, dpl)
     // istrap: 0 表示中断门，1 表示陷阱门
     // GD_KTEXT: 内核代码段选择子
     // __vectors[i]: 第 i 个中断服务例程的入口地址
     // DPL_KERNEL: 内核权限 (0)
     int i;
     for (i = 0; i < sizeof(idt) / sizeof(struct gatedesc); i++) {
         SETGATE(idt[i], 0, GD_KTEXT, __vectors[i], DPL_KERNEL);
     }
     
     // 特别设置：把系统调用/切换权限的中断门 DPL 设置为用户态权限 DPL_USER (3)，允许用户态发起
     SETGATE(idt[T_SYSCALL], 1, GD_KTEXT, __vectors[T_SYSCALL], DPL_USER);

     // 3. 告诉 CPU IDT 的位置和大小
     lidt(&idt_pd);
}
```
声明 __vectors[] 数组，将中断入口填充到 IDT 表中，并用 lidt 指令载入 IDT 描述符

### Step 3 修改 trap_dispatch 函数中的时钟中断部分
```
case IRQ_OFFSET + IRQ_TIMER:
        /* LAB1 YOUR CODE : STEP 3 */
        /* handle the timer interrupt */
        // 1. 静态累加器记录时钟中断触发次数
        static size_t ticks = 0;
        ticks++;
        
        // 2. 每达到 TICK_NUM (100) 次，调用 print_ticks() 打印提示
        if (ticks % TICK_NUM == 0) {
            print_ticks();
        }
        break;
```
定义一个静态变量 ticks 记录时钟中断次数，每达到 TICK_NUM（100 次）就打印一次信息


### Challenge

``` kern/init/init.c
// 取消这几行的注释（去掉前面的 //）：
    lab1_switch_to_user();
    lab1_switch_to_kernel();
static void
lab1_switch_to_user(void) {
    // LAB1 CHALLENGE 1 : TODO
    // 触发 T_SWITCH_TOU (120) 中断，由内核中断处理逻辑切换到用户态
    asm volatile (
        "sub $0x8, %%esp \n"          // 预留栈空间模拟中断压栈
        "int %0 \n"                    // 发起 T_SWITCH_TOU 软中断
        "movl %%ebp, %%esp"            // 恢复栈指针
        : 
        : "i"(T_SWITCH_TOU)
    );
}

static void
lab1_switch_to_kernel(void) {
    // LAB1 CHALLENGE 1 : TODO
    // 触发 T_SWITCH_TOK (121) 中断，由中断处理逻辑切换回内核态
    asm volatile (
        "int %0 \n"                    // 发起 T_SWITCH_TOK 软中断
        "movl %%ebp, %%esp \n"         // 恢复栈指针
        : 
        : "i"(T_SWITCH_TOK)
    );
}
```
```
case T_SWITCH_TOU:
    // 1. 修改代码段和数据段选择子为用户态 (RPL=3)
    tf->tf_cs = USER_CS; // 0x1B
    tf->tf_ds = USER_DS; // 0x23
    tf->tf_es = USER_DS;
    tf->tf_ss = USER_DS;

    // 2. 修改 EFLAGS 中的 IOPL（使得用户态可以打印/访问 IO 端口，仅限 lab1 的测试需求）
    tf->tf_eflags |= FL_IOPL_MASK;

    // 3. 注意栈的处理：如果之前是在内核态（同一个特权级），硬件引发中断时不会自动压入 esp 和 ss。
    // 你需要确保 iret 弹出正确的用户态栈地址，或者构造对应的空间。
    cprintf("+++ switch to user mode +++\n");
    print_trapframe(tf);
    break;
```

在进入 T_SWITCH_TOU 之前，CPU 并没有在栈上为你留出 esp 和 ss 的空间，直接向 tf->tf_esp 写入数据会破坏其他内存数据（踩内存）。

```
case T_SWITCH_TOU:
    if (tf->tf_cs != USER_CS) {
        // 1. 设置用户态段选择子
        tf->tf_cs = USER_CS;
        tf->tf_ds = USER_DS;
        tf->tf_es = USER_DS;
        tf->tf_ss = USER_DS;

        // 2. 设置用户态栈指针 esp
        // 确保 iret 弹栈后 esp 指向一个安全的地址
        tf->tf_esp = (uintptr_t)tf + sizeof(struct trapframe) - 8;

        // 3. 允许用户态进行 IO 操作（如 cprintf 打印），避免触发 GPF 异常
        tf->tf_eflags |= FL_IOPL_MASK;

        // 4. (可选) 保证中断响应开启
        tf->tf_eflags |= FL_IF;
        cprintf("+++ switch to user mode +++\n");
        print_trapframe(tf);
    }
    break;
```
```
case T_SWITCH_TOK:
    if (tf->tf_cs != KERNEL_CS) {
        // 1. 恢复内核态段选择子
        tf->tf_cs = KERNEL_CS; // 0x08
        tf->tf_ds = KERNEL_DS; // 0x10
        tf->tf_es = KERNEL_DS;

        // 2. 恢复内核态 EFLAGS（清空 IOPL）
        tf->tf_eflags &= ~FL_IOPL_MASK;

        // 3. 打印调试信息（注意：必须在修改 tf->tf_esp 或平移栈之前打印）
        cprintf("+++ switch to kernel mode +++\n");
        print_trapframe(tf);

        // 4. 【核心点】由于从 Ring 3 切回 Ring 0，原本硬件压入了 esp 和 ss。
        // 但 iret 发现返回目标是 Ring 0 时，不会弹出 esp 和 ss。
        // 我们需要把整个 trapframe 结构体在栈上向高地址平移 8 字节（覆盖掉 tf_esp 和 tf_ss 的位置），
        // 或者直接调整 esp，确保 iret 正确对齐。
        
        /* 简易平移方式（或在 init.c 中处理栈）： */
        struct trapframe *tf_new = (struct trapframe *)((uintptr_t)tf + 8);
        memmove(tf_new, tf, sizeof(struct trapframe) - 8);
        
        // 修改当前中断处理函数的栈帧指针，让 iret 从平移后的 tf_new 弹栈
        *((struct trapframe **)&tf - 1) = tf_new;
    }
    break;

```

## 遇到的问题/错误
```
case T_SWITCH_TOU:
        if (tf->tf_cs != USER_CS) {
            // 设置用户态数据段和代码段选择子
            tf->tf_cs = USER_CS;
            tf->tf_ds = USER_DS;
            tf->tf_es = USER_DS;
            tf->tf_ss = USER_DS;
            // 设置 EFLAGS 允许中断 (IF位)
            tf->tf_eflags |= FL_IF;
            cprintf("+++ switch to  user  mode +++\n");
            print_trapframe(tf);
        }
        break;

    case T_SWITCH_TOK:
        if (tf->tf_cs != KERNEL_CS) {
            // 设置内核态数据段和代码段选择子
            tf->tf_cs = KERNEL_CS;
            tf->tf_ds = KERNEL_DS;
            tf->tf_es = KERNEL_DS;
            cprintf("+++ switch to kernel mode +++\n");
            print_trapframe(tf);
        }
        break;
```



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

手动同步 Lab 1 代码到 Lab 2
``` bash 
cp ../lab1/kern/debug/kdebug.c kern/debug/kdebug.c
cp ../lab1/kern/trap/trap.c kern/trap/trap.c
```
``` bash
nano kern/mm/default_pmm.c
nano kern/mm/pmm.c
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
    + 空闲链表维护：按物理地址从低到高的顺序将空闲页块连接在双向链表 free_list 中。
    + 需要更改的部分
        - default_init_memmap：原代码使用的是 list_add(&free_list, &(base->page_link))，这会将新节点插入到头部。为了保证链表始终按物理地址递增排序，应该插入到合适位置（或在初始化阶段使用 list_add_before 插入末尾）。
        - default_free_pages：原代码在查找插入位置和合并逻辑上写得比较粗糙（直接遍历链表尝试进行相等判断，且最后简单的 list_add 并没有保证按地址顺序插入）。需要修正：先顺着 free_list 遍历找到第一个地址高于 base 的节点 le，然后将 base 插入到 le 之前；插入后再尝试与前一个节点（prev）和后一个节点（next）进行物理地址相邻合并。

``` C
#include <pmm.h>
#include <list.h>
#include <string.h>
#include <default_pmm.h>

free_area_t free_area;

#define free_list (free_area.free_list)
#define nr_free (free_area.nr_free)

static void
default_init(void) {
    list_init(&free_list);
    nr_free = 0;
}

static void
default_init_memmap(struct Page *base, size_t n) {
    assert(n > 0);
    struct Page *p = base;
    for (; p != base + n; p ++) {
        assert(PageReserved(p));
        p->flags = p->property = 0;
        set_page_ref(p, 0);
    }
    base->property = n;
    SetPageProperty(base);
    nr_free += n;

    // 保证链表按物理地址升序插入：将初始化块插入到 free_list 末尾（或合适位置）
    if (list_empty(&free_list)) {
        list_add(&free_list, &(base->page_link));
    } else {
        list_entry_t* le = &free_list;
        while ((le = list_next(le)) != &free_list) {
            struct Page* page = le2page(le, page_link);
            if (base < page) {
                list_add_before(le, &(base->page_link));
                break;
            }
            if (list_next(le) == &free_list) {
                list_add(le, &(base->page_link));
                break;
            }
        }
    }
}

static struct Page *
default_alloc_pages(size_t n) {
    assert(n > 0);
    if (n > nr_free) {
        return NULL;
    }
    struct Page *page = NULL;
    list_entry_t *le = &free_list;
    
    // First-Fit: 遍历按地址排序的链表，寻找第一个满足大小要求的空闲块
    while ((le = list_next(le)) != &free_list) {
        struct Page *p = le2page(le, page_link);
        if (p->property >= n) {
            page = p;
            break;
        }
    }
    
    if (page != NULL) {
        list_entry_t* prev = list_prev(&(page->page_link)); // 新代码在删除 page 之前，先记录了它在前驱节点 prev
        list_del(&(page->page_link));
        
        // 如果分配后还有剩余，切分出剩余部分并重新插回原位置（保持按地址排序）
        if (page->property > n) {
            struct Page *p = page + n;
            p->property = page->property - n;
            SetPageProperty(p);
            list_add(prev, &(p->page_link)); //由于切分出来的剩余块 p 紧跟在已被分配的 page 原位置之后，所以将 p 直接接在 prev 后面，完美保留了链表原本的物理地址顺序。
        }
        
        nr_free -= n;
        ClearPageProperty(page);
    }
    return page;
}

static void
default_free_pages(struct Page *base, size_t n) {
    assert(n > 0);
    struct Page *p = base;
    for (; p != base + n; p ++) {
        assert(!PageReserved(p) && !PageProperty(p));
        p->flags = 0;
        set_page_ref(p, 0);
    }
    base->property = n;
    SetPageProperty(base);
    
    // 1. 寻找合适的插入位置（按物理地址从低到高）
    list_entry_t *le = &free_list;
    while ((le = list_next(le)) != &free_list) {
        p = le2page(le, page_link);
        if (p > base) {
            break;
        }
    }
    list_add_before(le, &(base->page_link));
    nr_free += n;

    // 2. 尝试向高地址方向合并（检查 base 与下一个节点是否相邻）
    list_entry_t *next_le = list_next(&(base->page_link));
    if (next_le != &free_list) {
        p = le2page(next_le, page_link);
        if (base + base->property == p) {
            base->property += p->property;
            ClearPageProperty(p);
            list_del(&(p->page_link));
        }
    }

    // 3. 尝试向低地址方向合并（检查上一个节点与 base 是否相邻）
    list_entry_t *prev_le = list_prev(&(base->page_link));
    if (prev_le != &free_list) {
        p = le2page(prev_le, page_link);
        if (p + p->property == base) {
            p->property += base->property;
            ClearPageProperty(base);
            list_del(&(base->page_link));
        }
    }
}

```

- LAB2 EXERCISE 1: YOUR CODE  you should rewrite functions: default_init, default_init_memmap, default_alloc_pages, default_free_pages.
- 算法规则（First-Fit）
- 合并效率$O(N)$ 遍历且容易出现遗漏$O(1)$ 常数时间检查前后的 prev/next
```

```


2. 练习 2：实现寻找虚拟地址对应的页表项（Page Table Entry）
    + 主要文件：kern/mm/pmm.c
    + 找到位置：第 350 行附近的 get_pte() 函数。
    + 任务目标：根据给定的页表基址和虚拟地址，建立或查找二级页表结构，返回对应的 PTE 指针。需要注意内存分配失败以及页表项存在位的处理。

```
pte_t *
get_pte(pde_t *pgdir, uintptr_t la, bool create) {
    pde_t *pdep = &pgdir[PDX(la)];        // 1. 查找页目录项 (PDE)
    
    if (!(*pdep & PTE_P)) {               // 2. 判断页表是否存在 (PTE_P 为 0 表示不存在)
        if (!create) {                    // 3. 如果不需要创建，直接返回 NULL
            return NULL;
        }
        struct Page *page = alloc_page(); // 4. 分配一个物理页用来作为页表 (PT)
        if (page == NULL) {
            return NULL;
        }
        set_page_ref(page, 1);            // 5. 设置该页被引用次数为 1
        uintptr_t pa = page2pa(page);     // 6. 获取该物理页的物理地址
        memset(KADDR(pa), 0, PGSIZE);     // 7. 将页表内容清零 (注意必须用内核虚拟地址)
        *pdep = pa | PTE_P | PTE_W | PTE_U;// 8. 设置 PDE 的标志位 (存在、可写、用户可访问)
    }
    
    // 9. 从页表中取出对应的页表项 (PTE) 虚拟地址并返回
    pte_t *ptp = (pte_t *)KADDR(PDE_ADDR(*pdep));
    return &ptp[PTX(la)];
}
```

1. x86 二级页表（Two-Level Page Table）映射机制在 32 位 x86 保护模式下，虚拟地址（线性地址 la）长度为 32 位，分为三部分：$$\text{32 位虚拟地址} = \text{PDX (10 bit)} + \text{PTX (10 bit)} + \text{Offset (12 bit)}$$PDE (Page Directory Entry，页目录项)：通过 PDX(la) 索引。PDE 记录的是二级页表的物理基地址及权限。PTE (Page Table Entry，页表项)：通过 PTX(la) 索引。PTE 记录的是最终物理页的物理基地址及权限。
2. 物理地址与内核虚拟地址的转换（PADDR vs KADDR）CPU 开启分页（MMU）后，只能通过虚拟地址读写内存！alloc_page() 返回的是物理页的管理结构体 struct Page *；通过 page2pa(page) 拿到的是物理地址 pa。在内核代码中直接访问 pa 会引发内存错误（因为 CPU 会把 pa 当作虚拟地址解析）。必须调用 KADDR(pa) 将物理地址转为内核虚拟地址后才能做 memset 或数组指针运算。PDE_ADDR(*pdep) 能够屏蔽掉 PDE 里的低 12 位属性比特（如 PTE_P），提取出纯净的页表物理地址。

3. 练习 3：释放某虚地址所在的页并取消对应页表项映射
    + 主要文件：kern/mm/pmm.c
    + 找到位置：第 403 行附近的 page_remove_pte() 函数。
    + 任务目标：解除物理页与虚拟地址的映射关系，并在引用计数归零时释放该物理页，同时刷洗 TLB（页表缓存）。
```
static inline void
page_remove_pte(pde_t *pgdir, uintptr_t la, pte_t *ptep) {
    if (*ptep & PTE_P) {                  // 1. 检查该页表项 (PTE) 是否映射了有效物理页
        struct Page *page = pte2page(*ptep); // 2. 获取该 PTE 对应的物理页 Page 结构指针
        page_ref_dec(page);               // 3. 引用计数减 1
        if (page_ref(page) == 0) {        // 4. 如果引用计数降为 0，释放该物理页
            free_page(page);
        }
        *ptep = 0;                        // 5. 清空该页表项 (解除映射)
        tlb_invalidate(pgdir, la);        // 6. 刷新 TLB 快表
    }
}
}
```


1. 为什么页目录项权限要设置为 PTE_P | PTE_W | PTE_U？在硬件层面，硬件在检查权限时采用“严格逻辑与”原则：最终的访问权限取决于 PDE 和 PTE 权限的并集约束。在 get_pte 阶段，我们将 PDE 的权限设置得宽松一点（允许用户 PTE_U 和写入 PTE_W），具体这页内存到底能不能被用户访问或写入，交给具体调用者在 PTE 层级去细化控制（例如 page_insert）。
2. 引用计数（Reference Count）与物理页回收一个物理页可以被多个虚拟页映射（例如共享内存、fork 时的 Copy-On-Write 机制）。只有当引用计数 page_ref 降为 0 时，说明没有任何虚拟地址在使用这块物理页，才可以安全调用 free_page() 将其还给物理内存管理器（PMM）。5. TLB（快表）失效（TLB Invalidation）CPU 内置了硬件缓存 TLB (Translation Lookaside Buffer) 来加速虚拟地址到物理地址的转换。当我们在内存中修改了 PTE（取消了映射或改变了映射）时，CPU 的 TLB 里的旧缓存不会自动同步。必须显式调用 tlb_invalidate(pgdir, la) 执行 invlpg 指令刷新 TLB，否则 CPU 会继续沿用旧的缓存映射导致不可预料的错误。


## 遇到的问题

```
list_add(&free_list, &(p->page_link));
```
在切分完空闲块后，旧代码把剩余的块 p 强行插入到了 &free_list 的头部（list_add 默认插入到头部）。这直接打破了链表的物理地址升序排列，导致高地址的空闲块被插到了低地址最前面。
```
list_entry_t* prev = list_prev(&(page->page_link));
...
list_add(prev, &(p->page_link));
```

## 知识点总结
- First-Fit 算法（首次适应分配算法） 是操作系统内存管理（以及动态内存分配，如 malloc）中一种非常基础且高效的连续内存分配策略。
- 它的核心思想可以用一句话概括：“寻找第一个能装下的空闲内存块。”
# Lab3 in MOOC
## 前期准备需要的部分terminal 中的指令：

## Exercise Code

### Step 1

### Step 2
### Step 3
### Challenge

## 遇到的问题/错误

## 知识点总结

