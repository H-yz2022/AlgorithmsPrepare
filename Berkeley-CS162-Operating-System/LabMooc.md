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
git commit -m "YOUR CODE"
```
```
 gedit $(grep -rl "YOUR CODE" .) &

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
//nano kern/trap/trap.c
gedit kern/trap/trap.c kern/debug/kdebug.c
```
### Step 1 打印函数调用栈部分

``` kern/debug/kdebug.c
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

``` kern/trap/trap.c 
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
``` kern/trap/trap.c 
case IRQ_OFFSET + IRQ_TIMER:{
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
}
```
定义一个静态变量 ticks 记录时钟中断次数，每达到 TICK_NUM（100 次）就打印一次信息


### Challenge
```
grep -rn "LAB1 CHALLENGE 1" 
```
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
``` kern/trap/trap.c
case T_SWITCH_TOU:
        if (tf->tf_cs != USER_CS) {
            switchk2u = *tf;
            switchk2u.tf_cs = USER_CS;
            switchk2u.tf_ds = switchk2u.tf_es = switchk2u.tf_ss = USER_DS;
            switchk2u.tf_esp = (uint32_t)tf + sizeof(struct trapframe) - 8;

            // set eflags, make sure ucore can use io under user mode.
            // if CPL > IOPL, then cpu will generate a general protection.
            switchk2u.tf_eflags |= FL_IOPL_MASK;

            // set temporary stack
            // then iret will jump to the right stack
            *((uint32_t *)tf - 1) = (uint32_t)&switchk2u;
        }
        break;
    case T_SWITCH_TOK:
        if (tf->tf_cs != KERNEL_CS) {
            tf->tf_cs = KERNEL_CS;
            tf->tf_ds = tf->tf_es = KERNEL_DS;
            tf->tf_eflags &= ~FL_IOPL_MASK;
            switchu2k = (struct trapframe *)(tf->tf_esp - (sizeof(struct trapframe) - 8));
            memmove(switchu2k, tf, sizeof(struct trapframe) - 8);
            *((uint32_t *)tf - 1) = (uint32_t)switchu2k;
        }
        break;

```


## 遇到的问题/错误
``` GEMINI VERSION --WRONG
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
Why it crashes

When int T_SWITCH_TOU executes from kernel code (CPL 0), the CPU trap gate pushes only EFLAGS, CS, EIP (3 words) — it does not push ESP/SS, because no ring change happened going in. So those fields don't exist on the stack at all; whatever bytes sit past your trapframe's tf_eflags are just leftover stack garbage.

Now on the way out, your handler sets tf_cs = USER_CS. When trapentry.S eventually executes iret, the CPU sees the target CS has RPL 3 while the current CPL is 0 — that's a ring change, and hardware-mandated iret behavior is to also pop ESP/SS in that case, whether or not you intended it. It pops them from whatever garbage lies past your frame. You get a bogus stack pointer/segment loaded into the CPU → instant triple fault (VirtualBox/QEMU just silently reboots, which is almost certainly what you're seeing).

The reverse direction has a mirror-image bug: going ring3→ring0, hardware did push ESP/SS (5 words) on entry. If you just flip tf_cs to KERNEL_CS and leave the frame as-is, iret now sees CS RPL 0 == current CPL 0 → no ring change → it only pops 3 words and leaves ESP wherever it happens to be, which is now sitting 8 bytes into the middle of your trapframe instead of back at the correct kernel stack location. Stack corruption follows immediately.

So the fix isn't just editing field values — you have to physically reshape the trapframe to match what iret will actually consume, and repoint the frame that trapentry.S restores from.


## 知识点总结


### Problem in lab 1 challenges
1. Boot chain — how we even get to kern_init
- BIOS → bootloader (bootasm.S): BIOS loads the first 512-byte sector (bootblock) into memory and jumps to it. This code starts in real mode (16-bit, 1MB address limit, no protection).
- A20 line, GDT load, switch to protected mode: the bootloader sets up a minimal temporary GDT and flips the PE bit in CR0 to enter protected mode (32-bit addressing, segmentation/protection enforced).
- bootmain.c: reads the kernel ELF off disk and jumps to its entry point.

+ Link forward: protected mode is what makes privilege levels (rings), segment descriptors, and later paging possible — none of Lab1's ring0/ring3 story exists in real mode.

2. Segmentation & the GDT — defining what "ring 0" and "ring 3" even mean
- Global Descriptor Table (GDT): an array of segdesc entries, each describing a segment's base, limit, type, and DPL (Descriptor Privilege Level — 0–3).
- gdt_init() builds real entries: SEG_KTEXT/SEG_KDATA at DPL 0, SEG_UTEXT/SEG_UDATA at DPL 3 — all flat (base 0, limit 4GB), so segmentation isn't used for isolation here, only for privilege tagging.
- Segment selectors (cs, ds, es, ss, fs, gs) are indices into the GDT plus an RPL (Requested Privilege Level) in the low 2 bits — that's why USER_CS = 0x1b (0x18 | 3) and KERNEL_CS = 0x8 (0x8 | 0).
- CPL (Current Privilege Level): the CPU's actual current ring, taken from the low 2 bits of cs. This is what all protection checks compare against.

Link forward: the whole Challenge 1 exercise is about manufacturing a trapframe whose cs/ss values, once loaded by iret, change the CPU's CPL. Understanding CPL vs DPL vs RPL is the crux of why the switch code touches exactly those fields.

3. TSS (Task State Segment) — where the CPU gets a kernel stack from mid-flight
- struct taskstate ts, loaded into the GDT at SEG_TSS, then activated with ltr(GD_TSS).
- ts.ts_esp0 / ts.ts_ss0: whenever the CPU takes a trap that raises privilege (ring 3 → ring 0), it doesn't trust the current (untrusted, user-controlled) stack — it looks up esp0/ss0 in the TSS and switches to that stack automatically, in hardware, before your trap handler runs a single instruction.
- In this lab it's set to a temporary stack0[1024] — deliberately noted in the source as "not safe here... will be set to KSTACKTOP in lab2," because a real kernel needs a per-process kernel stack, not a single shared 1KB scratch buffer.

Link forward: this is exactly the mechanism that made your bug so nasty — once you're at ring 3, every timer interrupt silently uses this TSS-provided stack behind the scenes, independent of anything in trap.c. It's also why timing-dependent stack corruption (from the movl %ebp,%esp bug) could manifest differently across boots.

4. IDT (Interrupt Descriptor Table) — routing traps to handlers
- nstruct gatedesc idt[256]: one entry per trap/interrupt vector, each pointing at an entry stub and carrying its own DPL.
- SETGATE / idt_init(): for most vectors, entries are set at DPL_KERNEL — meaning only ring-0 code (or a hardware event) can invoke them via int n. T_SYSCALL and (in your fix) T_SWITCH_TOK are deliberately raised to DPL_USER because they must be callable from ring 3 — the x86 rule is CPL ≤ gate DPL for software int, so leaving a gate at DPL_KERNEL would fault immediately when called from ring 3.
- lidt(&idt_pd): loads the IDT register so the CPU knows where this table lives.
- __vectors[] / vectors.S (generated by tools/vector.c): 256 tiny stubs, one per vector, each pushing a trap number (and a dummy error code if the CPU doesn't push one itself) before jumping to the shared __alltraps.

Link forward: this table is why int $T_SWITCH_TOU and int $T_SWITCH_TOK work at all — they're just software-triggered entries into the same mechanism hardware exceptions and IRQs use.

5. Traps, exceptions, and interrupts — the three things landing in the same table
- Exceptions (divide error, #GP, page fault, ...): synchronous, caused by the currently executing instruction.
- Software interrupts: synchronous, explicitly triggered by int n — this is how T_SWITCH_TOU/T_SWITCH_TOK/T_SYSCALL work; they're not "real" hardware events, just a way to force CPU into privileged, table-dispatched code.
- Hardware (asynchronous) interrupts: IRQs from devices (timer, keyboard, serial), remapped via the PIC (pic_init()) to vectors starting at IRQ_OFFSET (32) so they don't collide with the CPU-reserved 0–31 exception vectors.
- trapframe: hardware pushes some fields automatically (eip, cs, eflags, plus esp, ss only if privilege changes); __alltraps pushes the rest (segment regs, general regs, trap number) to build the full struct trapframe uniformly for every entry type.
- trap_dispatch(): one big switch on tf_trapno that routes to the right handling — timer tick counting, console input, or (your addition) the ring switches.

Link forward: this uniform trapframe is exactly the object your T_SWITCH_TOU/T_SWITCH_TOK code hand-edits to fake a privilege change — you're not doing anything hardware doesn't already do on a real exception, you're just doing it deliberately and by hand.

6. iret semantics — the actual mechanism your bug lived in
- Same-privilege return (CS RPL == current CPL): iret pops exactly 3 words — eip, cs, eflags. esp/ss are untouched.
- Privilege-raising or -lowering return (CS RPL != current CPL): iret pops 5 words — eip, cs, eflags, esp, ss — and actually switches stacks using the popped esp/ss.
- This asymmetry is why T_SWITCH_TOU's handler must grow the effective frame (compute a synthetic tf_esp) while T_SWITCH_TOK's must shrink it (memmove down by 8 bytes) — they're preparing for a 5-word pop and a 3-word pop respectively.
- *((uint32_t*)tf - 1) = ...: this overwrites the value __alltraps will popl %esp into right before falling through to iret — i.e., it redirects the return path to your hand-rewritten frame instead of the original one.

Link forward: this is the literal center of Challenge 1 — everything else in the lab (GDT, TSS, IDT) exists to make this one iret call transition between rings safely.

7. EFLAGS / IOPL — why user-mode cprintf didn't just work
- EFLAGS carries, among other things, the IOPL (I/O Privilege Level) field — the CPU only allows in/out instructions when CPL ≤ IOPL.
- Console output (cprintf → port I/O) needs in/out. At ring 3 with default IOPL=0, any I/O instruction is an instant #GP.
- switchk2u.tf_eflags |= FL_IOPL_MASK — set to 3 specifically so ring-3 code in this lab can still legally do I/O (not realistic for a real OS, but needed for the test to print from "user mode" at all).
- T_SWITCH_TOK clears it back on the way to ring 0.

Link forward: this is a second, independent privilege axis (separate from CPL/DPL) that has to be handled correctly, or you get a #GP even with a perfectly correct trapframe otherwise.

8. Clock/timer interrupt — the thing running concurrently with all of this
- clock_init(): programs the PIT (Programmable Interval Timer) to fire IRQ_TIMER periodically.
- intr_enable() (sets IF in eflags): globally allows maskable hardware interrupts to actually reach the CPU.
- Each tick goes through the exact same IDT/__alltraps/trap_dispatch path as everything else — incrementing ticks, printing every TICK_NUM.

Link forward: this is what made your earlier debugging so confusing — the timer keeps firing (and correctly using the TSS esp0 stack) while your ring-3 test code runs, so a bug in the ring-3 code and a completely unrelated, correctly-functioning timer interrupt were interleaving in the same output stream.

The one-sentence chain

Protected mode + GDT define what privilege levels exist → IDT defines how the CPU reaches handler code for any trap → hardware trap entry + __alltraps builds a uniform trapframe → TSS esp0 supplies a safe stack whenever a trap raises privilege → trap_dispatch for T_SWITCH_TOU/T_SWITCH_TOK hand-edits that trapframe's cs/ss/esp/eflags → iret's privilege-sensitive pop behavior is what actually flips the CPU's CPL when that edited frame is restored → IOPL is a second gate that must also be opened for ring-3 code to still do I/O → and the timer interrupt keeps firing through this whole sequence, using the same machinery, which is why a bug anywhere in the chain tends to show up as timing-dependent chaos rather than a clean, deterministic crash.


# Lab2 in MOOC




## 前期准备需要的部分terminal 中的指令：
``` bash
cd ../lab2

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
gedit kern/mm/default_pmm.c kern/mm/pmm.c
```
OR
```
gedit $(grep -rl "YOUR CODE" .) &
```
```
find . -name "*~" -type f
find . -name "*~" -type f -delete
```
```
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


#define free_list (free_area.free_list)
#define nr_free (free_area.nr_free)



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

``` C
pte_t *
get_pte(pde_t *pgdir, uintptr_t la, bool create) {
    pde_t *pdep = &pgdir[PDX(la)];        // 1. 查找页目录项 (PDE)
    
    if (!(*pdep & PTE_P)) {               // 2. 判断页表是否存在 (PTE_P 为 0 表示不存在)
        if (!create) {                    // 3. 如果不需要创建，直接返回 NULL
            return NULL;
        }
        struct Page *page = alloc_page(); // 4. 分配一个物理页用来作为页表 (PT)
        //if (page == NULL) {
        //    return NULL;
        //}
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

- 
# Lab3 in MOOC
## 前期准备需要的部分terminal 中的指令：
``` bash
cd ../lab3
cd ~/mooc_os_lab/labcodes/lab1
```

``` bash
grep -rn "YOUR CODE" .
```

``` YOUR CODE 
./kern/mm/swap_fifo.c:52:    /*LAB3 EXERCISE 2: YOUR CODE*/ 
./kern/mm/swap_fifo.c:67:     /*LAB3 EXERCISE 2: YOUR CODE*/ 
./kern/mm/vmm.c:350:    /*LAB3 EXERCISE 1: YOUR CODE
./kern/mm/vmm.c:368:    /*LAB3 EXERCISE 1: YOUR CODE*/
./kern/mm/vmm.c:375:    /*LAB3 EXERCISE 2: YOUR CODE
./kern/mm/default_pmm.c:12:// LAB2 EXERCISE 1: YOUR CODE
./kern/mm/pmm.c:363:    /* LAB2 EXERCISE 2: YOUR CODE
./kern/mm/pmm.c:416:    /* LAB2 EXERCISE 3: YOUR CODE
./kern/debug/kdebug.c:296:     /* LAB1 YOUR CODE : STEP 1 */
./kern/trap/trap.c:39:     /* LAB1 YOUR CODE : STEP 2 */
./kern/trap/trap.c:183:        /* LAB1 YOUR CODE : STEP 3 */
./kern/trap/trap.c:198:    //LAB1 CHALLENGE 1 : YOUR CODE you should modify below codes.
```

同步 Lab 1& 2 代码到 Lab 3
``` bash 
sudo apt install -y meld   # one-time, if not already installed
meld lab1/kern/debug/kdebug.c   lab3/kern/debug/kdebug.c
meld lab1/kern/trap/trap.c      lab3/kern/trap/trap.c
meld lab2/kern/mm/default_pmm.c lab3/kern/mm/default_pmm.c
meld lab2/kern/mm/pmm.c         lab3/kern/mm/pmm.c
```

``` bash 
gedit $(grep -rl "LAB3" .) &
```
``` bash 
find . -name "*~" -type f
find . -name "*~" -type f -delete
```
``` bash 
make clean
make grade
make qemu
make clean && make
```


## Exercise Code
```
#if 0
    /*LAB3 EXERCISE 1: YOUR CODE*/
    ptep = ???              //(1) try to find a pte, if pte's PT(Page Table) isn't existed, then create a PT.
    if (*ptep == 0) {
                            //(2) if the phy addr isn't exist, then alloc a page & map the phy addr with logical addr

    }
    else {
    /*LAB3 EXERCISE 2: YOUR CODE
    * Now we think this pte is a  swap entry, we should load data from disk to a page with phy addr,
    * and map the phy addr with logical addr, trigger swap manager to record the access situation of this page.
    *
    *  Some Useful MACROs and DEFINEs, you can use them in below implementation.
    *  MACROs or Functions:
    *    swap_in(mm, addr, &page) : alloc a memory page, then according to the swap entry in PTE for addr,
    *                               find the addr of disk page, read the content of disk page into this memroy page
    *    page_insert ： build the map of phy addr of an Page with the linear addr la
    *    swap_map_swappable ： set the page swappable
    */
        if(swap_init_ok) {
            struct Page *page=NULL;
                                    //(1）According to the mm AND addr, try to load the content of right disk page
                                    //    into the memory which page managed.
                                    //(2) According to the mm, addr AND page, setup the map of phy addr <---> logical addr
                                    //(3) make the page swappable.
        }
        else {
            cprintf("no swap_init_ok but ptep is %x, failed\n",*ptep);
            goto failed;
        }
   }
#endif
```
### Step 1
``` vmm.c
 if ((ptep = get_pte(mm->pgdir, addr, 1)) == NULL) {
        cprintf("get_pte in do_pgfault failed\n");
        goto failed;
    }
    
    if (*ptep == 0) { // if the phy addr isn't exist, then alloc a page & map the phy addr with logical addr
        if (pgdir_alloc_page(mm->pgdir, addr, perm) == NULL) {
            cprintf("pgdir_alloc_page in do_pgfault failed\n");
            goto failed;
        }
    }
    else { // if this pte is a swap entry, then load data from disk to a page with phy addr
           // and call page_insert to map the phy addr with logical addr
        if(swap_init_ok) {
            struct Page *page=NULL;
            if ((ret = swap_in(mm, addr, &page)) != 0) {
                cprintf("swap_in in do_pgfault failed\n");
                goto failed;
            }    
            page_insert(mm->pgdir, page, addr, perm);
            swap_map_swappable(mm, addr, page, 1);
        }
        else {
            cprintf("no swap_init_ok but ptep is %x, failed\n",*ptep);
            goto failed;
        }
   }
   ret = 0;
failed:

```
### Step 2
``` vmm.c

        if(swap_init_ok) {
            struct Page *page=NULL;
            if ((ret = swap_in(mm, addr, &page)) != 0) {
                cprintf("swap_in in do_pgfault failed\n");
                goto failed;
            }    
            page_insert(mm->pgdir, page, addr, perm);
            swap_map_swappable(mm, addr, page, 1);
        }
        else {
            cprintf("no swap_init_ok but ptep is %x, failed\n",*ptep);
            goto failed;
        }
```


``` swap_fifo.c
static int
_fifo_map_swappable(struct mm_struct *mm, uintptr_t addr, struct Page *page, int swap_in)
{
    list_entry_t *head=(list_entry_t*) mm->sm_priv;
    list_entry_t *entry=&(page->pra_page_link);
 
    assert(entry != NULL && head != NULL);
    //record the page access situlation
    /*LAB3 EXERCISE 2: YOUR CODE*/ 
    //(1)link the most recent arrival page at the back of the pra_list_head qeueue.
    list_add(head, entry);
    return 0;
}
/*
 *  (4)_fifo_swap_out_victim: According FIFO PRA, we should unlink the  earliest arrival page in front of pra_list_head qeueue,
 *                            then set the addr of addr of this page to ptr_page.
 */
static int
_fifo_swap_out_victim(struct mm_struct *mm, struct Page ** ptr_page, int in_tick)
{
     list_entry_t *head=(list_entry_t*) mm->sm_priv;
         assert(head != NULL);
     assert(in_tick==0);
     /* Select the victim */
     /*LAB3 EXERCISE 2: YOUR CODE*/ 
     //(1)  unlink the  earliest arrival page in front of pra_list_head qeueue
     //(2)  set the addr of addr of this page to ptr_page
     /* Select the tail */
     list_entry_t *le = head->prev;
     assert(head!=le);
     struct Page *p = le2page(le, pra_page_link);
     list_del(le);
     assert(p !=NULL);
     *ptr_page = p;
     return 0;
}
```

``` trap.c 
static void
trap_dispatch(struct trapframe *tf) {
    char c;

    int ret;
    static struct trapframe switchk2u, *switchu2k;
```
```
static volatile int in_swap_tick_event = 0;
extern struct mm_struct *check_mm_struct;
```

### Challenge

## 遇到的问题/错误

- 不应该直接复制lab1 lab2内容：
    + 需要用meld
- 有些code跟标准答案有出入：
    + vmm.c: ptep = get_pte(mm->pgdir, addr, 1); and goes straight to if (*ptep == 0). get_pte(..., 1) allocates a new page-table page when the PDE is missing, and that allocation can fail under memory pressure — in that case it returns NULL. Your version would then dereference a null pointer (*ptep) instead of failing gracefully into -E_NO_MEM via the existing failed: label. It passed your test run because the test environment always has free memory to satisfy that allocation, so the failure path never actually triggers — but it's a latent crash bug, not just style. Worth adding that check now since it's a one-line fix and it's exactly the kind of thing that would bite you on a stress test or a grading harness that simulates OOM.
``` 
if ((ptep = get_pte(mm->pgdir, addr, 1)) == NULL) {
    cprintf("get_pte in do_pgfault failed\n");
    goto failed;
}
```
vmm.c step 2 (swap-in path) — identical between yours and the professor's. Nothing to change here.
    + swap_fifo.c — functionally equivalent, just mirrored conventions, and both are internally consistent so neither is "more correct":
        - Professor: insert with list_add(head, entry) (= insert right after head, i.e. at head->next), evict from head->prev.
        - You (per our earlier exchange): insert with list_add_before(head, entry) (= insert right before head, i.e. at head->prev), evict from head->next (list_next(head)).
- 排除错误的方法：
···
grep -n "in_swap_tick_event\|check_mm_struct" kern/trap/trap.c
168:    extern struct mm_struct *check_mm_struct;
170:    if (check_mm_struct != NULL) {
171:        return do_pgfault(check_mm_struct, tf->tf_err, rcr2());
176:static volatile int in_swap_tick_event = 0;
177:extern struct mm_struct *check_mm_struct;
moocos@moocos-VirtualBox:~/mooc_os_lab/labcodes/lab3$
···
  

## 知识点总结
1. Virtual address space — the range of addresses a process (or the kernel, via check_mm_struct in this lab's tests) can reference, distinct from actual RAM addresses. Lab3 is the layer that decides what's legal in that space before Lab2's allocator is ever touched.
2. mm_struct — one per address space. Holds the doubly-linked list of VMAs (mmap_list), a one-entry lookup cache (mmap_cache), the page directory pointer (pgdir), and sm_priv, an opaque pointer the swap manager uses for its own bookkeeping (in this lab, it points at the FIFO queue head).
3. vma_struct (Virtual Memory Area) — one contiguous sub-range of an address space with uniform permissions: vm_start, vm_end, vm_flags (VM_READ / VM_WRITE / VM_EXEC). A process's full address space is a set of these — e.g. one VMA for code, one for heap, one for stack — each with different permissions.
4. Page Directory / Page Table (PDT/PT), PDE/PTE — the two-level x86 translation structure. A PDE points to a page table; a PTE points to (or records the swapped-out location of) one 4 KB physical page. get_pte walks/creates this structure; a PTE value of 0 means "nothing mapped yet," a non-zero-but-not-present value in this lab's convention means "swapped out — the bits encode where on disk."
5. Page fault exception (#PF) — raised by the CPU when a memory access can't be satisfied by the current page tables. The hardware hands the handler two things: CR2 (the faulting linear address) and an error code whose low 3 bits are P (present), W/R (write vs. read), U/S (user vs. supervisor). do_pgfault's switch (error_code & 3) is reading exactly those bits to decide whether the fault is even legitimate for this VMA before doing any allocation work.
6. Demand paging — not eagerly allocating physical memory for a VMA at creation time; only backing an address with a real page the first time it faults. This is why do_pgfault's *ptep == 0 branch exists — it's the "first touch" allocation path.\
7. Swap subsystem — the mechanism that lets total virtual memory exceed physical RAM by evicting a physical page's contents to disk and reusing the frame, then restoring it on next access. swap_in/swap_out do the disk I/O; the policy of which page to evict is delegated to a page replacement algorithm (PRA).
8. struct swap_manager (the FIFO in swap_fifo.c) — a function-pointer table (init, init_mm, map_swappable, swap_out_victim, tick_event, check_swap, …) — i.e. a hand-rolled vtable. This is the OS-textbook pattern for swapping PRA implementations (FIFO here; LRU/Clock in harder variants of this lab) without touching the fault-handling code in vmm.c at all.
9. FIFO PRA — evict whichever resident page arrived first, tracked with a circular doubly-linked list (pra_list_head, list_add/list_del from list.h). Cheap, no per-access bookkeeping — and, notoriously, subject to Belady's anomaly (adding more physical frames can increase the fault count). It's the simplest correct PRA, which is why it's the one assigned first.
10. Reference/access recency — the thing FIFO deliberately ignores. A page can be evicted the instant after it was heavily used, just because it happened to arrive early. This is the conceptual gap that LRU/Clock algorithms close (see §4).
11. Tick event / in_swap_tick_event — a hook off the Lab1 timer interrupt that can drive swapping periodically rather than only synchronously inside a page fault. It's why trap.c needs extern struct mm_struct *check_mm_struct — the timer handler, running outside the fault path, still needs to know which address space it's managing.
<br>
### The logical flow (cause → effect through the files you edited)
1. CPU faults → Lab1's IDT entry for vector 14 fires → trap.c's trap_dispatch recognizes T_PGFLT → calls pgfault_handler → do_pgfault(mm, error_code, addr) in vmm.c.
2. Legality check — find_vma(mm, addr) walks mm->mmap_list (a plain linked list here — see §4) to find a VMA containing addr. If none exists, or the error-code bits are inconsistent with that VMA's vm_flags (e.g. a write fault against a read-only VMA), do_pgfault fails immediately — before ever consulting Lab2's allocator. This is the "policy before mechanism" boundary between Lab3 and Lab2.
3. Resolve the PTE — get_pte(mm->pgdir, addr, 1) finds or creates the PTE slot (allocating a page-table page
via Lab2's alloc_page if needed — this can fail, hence the NULL check discussed earlier).
4. Two branches, both eventually calling into Lab2:
    - Never mapped (*ptep == 0): pgdir_alloc_page = Lab2's alloc_page + page_insert. Pure demand paging, no disk involved.
    - Previously swapped out (PTE non-zero, not present): swap_in reads the page back from disk into a freshly allocated frame, page_insert remaps it, and swap_map_swappable hands the page to the FIFO manager — which is where _fifo_map_swappable appends it to pra_list_head.
5. Memory pressure — when Lab2's allocator later needs a free frame and none exists, the swap layer calls swap_out_victim, which is _fifo_swap_out_victim here: pop the oldest entry off the FIFO list, write it to disk, and hand the now-free physical frame back to Lab2. This is the step that actually closes the loop and makes memory overcommit possible.
6. Self-test — _fifo_check_swap deliberately touches more distinct virtual pages than there are physical frames, in a specific order, and asserts pgfault_num after each access. This single test exercises all of the above at once: find_vma correctness, the demand-paging branch, the swap-in branch, and — critically — that your FIFO insert/evict ordering matches the expected trace. If your insert-end and evict-end conventions in swap_fifo.c are internally consistent (as discussed previously), this test passes regardless of which physical end of the list you called "front."

### One-paragraph summary
Lab3 slots a policy layer (mm_struct/VMA legality + PRA choice) between Lab1's fault delivery and Lab2's raw page allocator: a fault first asks "is this address even allowed, and how?" (VMA + error-code check), then "do I need a fresh page or a swapped-out one back?" (the *ptep == 0 branch vs. swap_in), and only then touches Lab2's allocator. The FIFO swap manager is a deliberately minimal, swappable (pun intended) PRA implementation wired in through a vtable so it can be replaced by Clock/LRU without touching do_pgfault — and the specific rough edges above (global list state, no recency awareness, linear VMA lookup, an unchecked allocation failure) are the standard list of things a "Lab3+" or systems-course follow-up would tighten.


# Lab4 in MOOC
## 前期准备需要的部分terminal 中的指令：
``` bash
cd ../lab4
cd ~/mooc_os_lab/labcodes/lab4
```

``` bash
grep -rn "YOUR CODE" .
```

``` YOUR CODE 

```

同步 Lab 1& 2 代码到 Lab 3
``` bash 
#sudo apt install -y meld   # one-time, if not already installed
meld lab1/kern/debug/kdebug.c   lab3/kern/debug/kdebug.c
meld lab1/kern/trap/trap.c      lab3/kern/trap/trap.c
meld lab2/kern/mm/default_pmm.c lab3/kern/mm/default_pmm.c
meld lab2/kern/mm/pmm.c         lab3/kern/mm/pmm.c
```

``` bash 
gedit $(grep -rl "LAB4" .) &
```
``` bash 
find . -name "*~" -type f
find . -name "*~" -type f -delete
```
``` bash 
make clean
make grade
make qemu
make clean && make
```
## Exercise Code

### Step 1

### Step 2
### Step 3
### Challenge

## 遇到的问题/错误

## 知识点总结



# Lab5 in MOOC
## 前期准备需要的部分terminal 中的指令：
## Exercise Code

### Step 1

### Step 2
### Step 3
### Challenge

## 遇到的问题/错误

## 知识点总结
