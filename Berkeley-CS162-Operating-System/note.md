# CS61C
## What does the OS do?
- One of the first things that runs when your computer starts (right after firmware/bootloader)
- Loads, runs and manages programs:
 + Multiple programs at the same time (time-sharing)
 + Isolates programs from each other (isolation)
 + Multiplex resources between applications (e.g., devices)
- Services: File System, Network stack, etc. 
- Finds and controls all the devices in the machine in general way (using "device drivers")
- Agenda
1) Devices and l/O
2) OS Boot Sequence and Operation
3) Multiprogramming/time-sharing
4) Introduction to Virtual Memory<br>

<img width="296" height="176" alt="image" src="https://github.com/user-attachments/assets/2d30d3ac-e45d-4140-b66b-a48e92e6ee51" /><br>

## Instruction Set Architecture for 1/O
- What must the processor do for 1/O?
    + Input: reads a sequence of bytes
    + Output: writes a sequence of bytes
- Some processors have special input and output instructions
- Alternative model (used by MIPS):
    + Use loads for input, stores for output (in small pieces)
    + Called Memory Mapped Input/Output
    + A portion of the address space dedicated to communication paths to Input or Output devices (no

<img width="290" height="161" alt="image" src="https://github.com/user-attachments/assets/fc16b492-0339-4782-8285-6089b1802925" /><br>


## Processor-1/O Speed Mismatch
- 1GHz microprocessor can execute 1B load or store instructions per second, or 4,000,000 KB/s data rate
- 1/0 data rates range from 0.01 KB/s to 1,250,000 KB/s the processor loads it
- Input: device may not be ready to send data as fast as
- Also, might be waiting for human to act; processor stores it
- Output: device not be ready to accept data as fast as
- What to do?

## Polling

- Processor Checks Status before Acting
    + Path to a device generally has 2 registers: Control Register, says it's OK to read/write (1/O ready) [think of a flagman on a road]
    + Data Register, contains data
- Processor reads from Control Register in loop, waiting for device to set Ready bit in Control reg (0--->1)to say it's OK
- Processor then loads from (input) or writes to (output) data register
    + Load from or Store into Data Register resets Ready bit (0--->1) of Control Register
 - This is called "Polling'<br>
<img width="260" height="172" alt="image" src="https://github.com/user-attachments/assets/aef40aa7-467e-49f9-94c2-c3076598d453" /><br>

### Cost of Polling?
Assume for a processor with a 1GHz clock it takes400 clock cycles for a polling operation (calling polling routine, accessing the device, and returning).
- Determine % of processor time for polling movement
- Mouse: polled 30 times/sec so as not to miss user
- Floppy disk (Remember those?): transferred data in 2-Byte units and had a data rate of 50 KB/second.
- No data transfer can be missed.
- Hard disk: transfers data in 16-Byte chunks and can transfer at 16 MB/second. Again, no transfer can be missed

#### % Processor time to poll
- Mouse Polling [clocks/sec]<br>
= 30 [polls/s]*400 [clocks/poll]=12K [clocks/s]
-  % Processor for polling:<br>
12*103[clocks/s]/1*109 [clocks/s]=0.0012%
- Polling mouse little impact on processor

- Clicker Time<br>
Hard disk: transfers data in 16-Byte chunks and can transfer at 16 MB/second. No transfer can be missed. What percentage of processor time is spent in polling?<br>
. A:2%<br>
· B:4%<br>
· C:20%<br>
· D:40% ---> right<br>
· E:80%<br>
- % Processor time to poll hard disk<br>
Frequency of Polling Disk= 16 [MB/s]/16 [B/poll] = 1M [polls/s]
- Disk Polling, Clocks/sec= 1M[polls/s]*400 [clocks/poll]<br>
=400M[clocks/s]
- % Processor for polling:<br>
400*106[clocks/s]/1*10[clocks/s]=40%<br>
-> Unacceptable <br>
(Polling is only part of the problem - main problem is that accessing in small chunks is inefficient) <br>
<img width="295" height="177" alt="image" src="https://github.com/user-attachments/assets/44014d5f-ae32-4bce-8c96-805137e39820" />

### Supervisor Mode
- If something goes wrong in an application, it can crash the entire machine. What about malware, etc.?
- The OS may need to enforce resource constraints to applications (e.g., access to devices).
- To protect the OS from the application, CPUs have a supervisor mode bit (also need isolation, more later).
    + You can only access a subset of instructions and (physical) memory when not in supervisor mode (user mode).
    + You can change out of supervisor mode using a special instruction. but not into it (unless there is an interrupt)
- How to switch back to OS? OS sets timer interrupt,when interrupts trigger, drop into supervisor mode.
- What if we want to call into an OS routine? (e.g., to registers, and then raise software interrupt read a file, launch a new process, send data, etc.)
- Need to perform a syscall: set up function arguments in
- OS will perform the operation and return to user mode
- This way, the OS can mediate access to all resources, including devices, the CPU itself, etc.

###  Multiprogramming/time-sharing
- The OS runs multiple applications at the same time.
- But not really (unless you have a core per process)
- Switches between processes very quickly. This is called a "context switch".
- When jumping into process, set timer interrupt.
    + When it expires, store PC, registers, etc. (process state).
    + Pick a different process to run and load its state.
    + Set timer, change to user mode, jump to the new PC.
- Deciding what process to run is called scheduling.

#### Protection, Translation, Paging
Supervisor mode does not fully isolate applications from each other or from the OS.
- Application could overwrite another application's memory.
- Remember your Project 1 linker: application assumes that code is in certain location. How to prevent overlaps?
- May want to address more memory than we actually have (e.g., for sparse data structures).
- Solution: Virtual Memory. Gives each process the illusion of a full memory address space that it has completely for itself.


#### Simple Base and Bound Translation
Base and bounds registers are visible/ accessible only when precessor is running in supervisor mode


#### Separate Areas for Program and Data
- What is an advantage of this separation?<br>
As users come and go, the storage is "fragmented". Therefore, at some stage programs have to be moved around to compact the storage.<br>
Page Memory Systems: Processor-generated address can be split into: page number and offset.<br>
A page table contains the physical address of the base of each page. Page tables make it possible to store the pages of a program non-contiguously.<br>
- Where Should Page Tables Reside?
Space required by the page tables (PT) is proportionalto the address space, number of users, ...<br>
-----> Too large to keep in registers
- Idea: Keep PTs in the main memoryanother to access the data word<br>
- Needs one reference to retrieve the page base address and
-----> doubles the number of memory references!

## Precise Traps
- Trap handler's view of machine state is that everyinstruction prior to the trapped one has completed, andno instruction after the trap has executed.
- Implies that handler can return from an interrupt by restoring user registers and jumping to EPC
     + Interrupt handler software doesn't need to understand the pipeline of the machine, or what program was doing!
     + More complex to handle trap caused by an exception
- Providing precise traps is tricky in a pipelined superscalar out-of-order processor!
     + But handling imprecise interrupts in software is even worse

 
### Trap Handling in 5-stage Pipeline

Save Exception Until Commit

<img width="375" height="182" alt="image" src="https://github.com/user-attachments/assets/e76f27c3-c06d-4faf-b67f-6af99d22b1bf" />

### Dynamic Address Translation
- Motivation
     + In early machines, I/O operations were slow and each word transferred involved the CPU
     + Higher throughput if CPU and 1/O of 2 or more programs were overlapped.
     + How?= multiprogramming with DMA I/O devices, interrupts
- Location-independent programs
     + Programming and storage management ease
     + ----> need for a base register
- Protection
     + Independent programs should not affect each other inadvertently
     + ----> need for a bound register
- Multiprogramming drives requirement for resident supervisor (os) software to manage context switches between multiple programs
- Atlas Demand Paging Scheme
- On a page fault:
    + Input transfer into a free page is initiated
    + The Page Address Register (PAR) is updated
    + If no free page is left, a page is selected to be replaced(based on usage)
    + The replaced page is written on the drum
    + The page table is updated to point to the new location of the page on the drum
- Size of Linear Page Table<br>
With 32-bit addresses, 4-KB pages & 4-byte PTEs:<br>
  ----> 2^20 PTEs, i.e, 4 MB page table per user<br>
  ----> 4 GB of swap needed to back up full virtual address space<br>
- Hierarchical Page Table
- Translation Lookaside Buffers (TLB)
    + Address translation is very expensive! In a two-level page table, each reference becomes several memory accesses
    + Solution: Cache translations in TLB
    + TLB hit = Single-Cycle Translation
    + TLB miss = Page-Table Walk to refill

# Discussion 0 C,x86
A typical C program’s memory is divided into five segments.
| Segment | Purpose |
| :--- | :--- |
| **Text** | Machine code of the compiled program |
| **Initialized Data** | Initialized global and static memory |
| **Uninitialized Data** | Uninitialized global and static memory |
| **Heap** | Dynamically allocated memory |
| **Stack** | Local variables and argument passing |

### Example 1.1 
1. Consider a valid double pointer char** dbl.char in a 32-bit system. What is sizeof(*dbl char)?<br>
     4. dbl_char is a double pointer, so dereferencing it once will still give a pointer. A 32-bit system means memory addresses will be 32 bits or equivalently 4 bytes.

2. Consider strings initialized as<br>
char* a = "162 is the best";<br>
char b[] = "162 is the best";<br>
Are a and b different?<br>
Yes. Since it’s defined as a literal, a points to a string literal in a read only section of memory.<br>
On the other hand, b resides on the stack. It is equivalent to declaring<br>
char b[] = {'1', '6', '2', ' ', 'i', 's', ' ', 'b', 'e', 's', 't', 0}.<br>

### Example 1.3 
1. We want to debug the program using GDB. How should we compile the program?<br>
gcc-g singer.c-o singer. We need a-g flag for debugging information. The-o simply controls the name of the executable that is created which defaults to a.out if not specified.<br>

### Example 2.1
1. Between SP and BP, which has a higher memory address?<br>
bp has a higher memory address. The stack grows downwards, meaning the top of the stack moves towards lower addresses.<br>

2. Based on the differences of RISC and CISC, why might x86 have less GPRs compared to RISC-V?<br>
The reduced instruction set of RISC-V means the processor requires less hardware space for transistors, leaving more room for GPRs.<br>

4. True or False: Right before the caller jumps to the desired function,the stack must be 16-byte aligned<br>
False. The stack needs to be 16-byte aligned right after the parameters have been pushed onto the stack. Return address is pushed right before jumping.

### Example 2
``` C
intp=0;

 intbar(intx,inty,intz){
    intw=x+y-z;
    returnw+1;
 }

 void foo(inta,intb){
    p=a+b+bar(3,4,5);
}
```
``` stack frame
1 p:
2 .zero 4
3 bar:
4 pushl %ebp
5 movl %esp,%ebp
6 subl $16,%esp
7 movl 8(%ebp),%edx
8 movl 12(%ebp),%eax
9 addl %edx,%eax
10 subl 16(%ebp),%eax
11 movl %eax,-4(%ebp)
12 movl-4(%ebp),%eax
13 addl $1,%eax
14 leave
15 ret
16 foo:
17 pushl %ebp
18 movl %esp,%ebp
19 pushl %ebx
20 subl $4,%esp
21 movl 8(%ebp),%edx
22 movl 12(%ebp),%eax
23 leal (%edx,%eax),%ebx
24 subl $4,%esp
25 pushl $5
26 pushl $4
27 pushl $3
28 call bar
29 addl $16,%esp
30 addl %ebx,%eax
31 movl %eax,p
32 nop
33 movl-4(%ebp),%ebx
34leave
35ret

```

# Discussion 1

## Example 1.1
1. What is the importance of address translation?<br>
Address translation is necessary for the idea of virtual memory which provides an isolation abstraction. This gives the illusion that each process is the sole user of the address space. It also provides protection between different processes since virtual addresses will not translate to the same physical address, preventing processes from accessing and potentially corrupting each
other’s memory. <br>

2. Similar to what’s done in the prologue at calling convention, what needs to happen before a mode transfer occurs? <br>
The processor’s state (e.g. registers) need to be saved in the thread control block (TCB) because the kernel may overwrite the registers when it executes its own code.<br>

## Example 2.1
1. How many new processes (not including the original process) are created when the following programis run? Assume all fork calls succeed.
``` C
int main(void) {
    for (int i = 0; i < 3; i++)
      pid_t fork_ret = fork();
    return 0;
 }
```
7.<br>
Newly forked processes will continue to execute the loop from wherever the parent process left off.<br>

2. What are the possible outputs when the following program is run?
``` C
int main(void) {
       int stuff = 5;
       pid_t fork_ret = fork();
       printf("The last digit of pi is %d\n", stuff);
       if (fork_ret == 0)
          stuff = 6;
       return 0;
}
```
fork creates a new process, meaning the address spaces are identical but not shared. As a result, changing stuff in the child process will have no effect on the value of stuff in the parent process, regardless of the execution order of the two processes. This means the following will be printed.<br>
The last digit of pi is 5.<br>
The last digit of pi is 5.<br>
However, it is possible the fork doesn’t succeed as no such assumption was made. As a result, the child process would never be created, only resulting in one statement being printed.<br>

##fork()
- 程序可以调用函数fork(来创建一个新的进程
  + 操作系统需要分配一个新的并且唯一的进程ID
  + 因此在内核中，这个系统调用会运行
  + 1) new_pid = next_pid++ ;
  + 翻译成机器指令
     1) LOAD next_pid Regl
     2) STORE Regl new_pid
     3) INC Regl
     4) STORE Regl next_pid
- 假设两个进程并发执行
  + 如果next_pid等于100，那么其中一个进程得到的ID应该是100，另一个进程的ID应该是101，next_pid应该增加到102

<img width="317" height="204" alt="37b2398a546abb03604e071b744b3c02" src="https://github.com/user-attachments/assets/7515eaee-d3a0-4751-b5be-1c22d408e667" />
<br>
- Atomic Operation<br>
- 原子操作是指一次不存在任何中断或者失败的执行<br>
  + 该执行成功结束
  + 或者根本没有执行
  + 并且不应该发现任何部分执行的状态
- 实际上操作往往不是原子的
  + 有些看上去是原子操作，实际上不是
  + 连x++这样的简单语句，实际上是由3条指令构成的
  + 有时候甚至连单条机器指令都不是院子的
        +    Pipeline, super-scalar, out-of-order, page fault<br>
<img width="305" height="212" alt="fcee1b192a7b55a27cd65e3280bb7fed" src="https://github.com/user-attachments/assets/b6ae34c9-cd60-4468-89bc-2831ffdfc09c" />
<br>


### Example 2.1 Pintos Lists

# Discussion 2 Threads, I/O
##  Threads
1. int pthread_create
2. void pthread_exit
3. int pthread_yield
4. int pthread_join

### Example Threads
<img width="332" height="254" alt="3ed8e30d6d0b79a428df1b345a1dc478" src="https://github.com/user-attachments/assets/15dd3f4e-bef6-4e62-8ab5-4ffca5b7740e" />
<img width="290" height="264" alt="6a605c271584efc02299e069baa1ed50" src="https://github.com/user-attachments/assets/210a1c80-7679-4118-b5f6-cce673ddddbb" /><br>





## I/O
1. Low-Level API: 
2. High-Level API
3. Interprocess Communication IPC <br>
Pipes are one-way communication channels between processes on the same physical machine.<br>
Sockets are two-way communication channels between processes. <br>

### Example I/O
1. What’s the difference between fopen and open?<br>
fopen is a high-level API, while open is a low-level API. fopen will return a FILE* type, while open will return an integer<br>
<br>
2. What will the test.txt file look like after this program is run? You may assume read and write fully succeed (i.e. read/write the specified number of bytes).<br>
For reference, SEEK_SET will set the offset to offset bytes, while SEEK
CUR will set the offset to its current location plus offset bytes. Seeking past the end of a file will set the bytes to 0.<br>

``` C
int main() {
    char buffer[200];
    memset(buffer, 'a', 200);
    int fd = open("test.txt", O_CREAT|O_RDWR);
    write(fd, buffer, 200);
    lseek(fd, 0, SEEK_SET);
    read(fd, buffer, 100);
    lseek(fd, 500, SEEK_CUR);
    write(fd, buffer, 100);
}
```
Step-by-Step Execution Trace<br>
1. open("test.txt", O_CREAT|O_RDWR): Creates/opens test.txt. File size: 0 bytes. Initial offset: 0.
2. write(fd, buffer, 200): Writes 200 'a' characters.File size: 200 bytes. Current offset: 200.
3. lseek(fd, 0, SEEK_SET): Moves offset to 0 (beginning of the file).
4. read(fd, buffer, 100): Reads 100 bytes of 'a'.Current offset advances from 0 to 100.
5. lseek(fd, 500, SEEK_CUR): Moves the offset forward by 500 bytes from current position 100. New offset: $100 + 500 = \mathbf{600}$. Since offset 600 is past the end of the file (which was 200 bytes), bytes 200 to 599 are unallocated gap space filled with literal null bytes (\0).
6. write(fd, buffer, 100): Writes 100 bytes of 'a' starting at offset 600.Final file size: 700 bytes.



### Example 2.2 File Descriptor Fun
2. What does the following program print to standard output?
``` C
int main(int argc, char **argv) {
    int fd;
    if ((fd = open("out.txt", O_CREAT|O_TRUNC|O_WRONLY, 0644)) < 0)
    exit(1);
    printf("Thelastdigitofpiis ...");
    fflush(stdout);
    dup2(fd,1);
    printf("five");
    exit(0);
}
```
Standard out will only print<br>
The last digit of pi is...<br>
since we replaced the file descriptor 1 was remapped to correspond to the file descriptor corresponding to the open out.txt. "five" will end up in out.txt.<br>

## Echo Server 
### Example 2.3 Echo Server
``` C

```


# Discussion 3 


## Mutual Exclusion

<br>
Mutual exclusion (互斥)<br>
当一个进程处于临界区并访问共享资源时，没有其他进程会处于临界区并且访问任何相同的共享资源<br>
<br>

<br>


###Locks

## Condition Variables
1. Wait
2. Signal
3. Broadcast

### Infinite Synchronized Buffer
### Semantics


### Example 2.1
 Will this program compile/run?
``` C
 pthread_mutex_t lock;
 int hello = 0;
void print_hello() {
hello += 1;

printf("First line (hello=%d)\n", hello);
pthread_cond_signal(&cv);
pthread_exit(0);

}

void main() {
pthread_t thread;
pthread_create(&thread, NULL, (void *) &print_hello, NULL);
while (hello < 1)
pthread_cond_wait(&cv, &lock);
printf("Second line (hello=%d)\n", hello);
cv hello.c
}


```
This program will not run because the thread needs to be holding a lock before performing a
condition variable operation like wait or signal.<br>
Moreover, the lock and condition variable was never initialized, which would lead to undefined
behavior.<br>


### Example 2.2 Office Hours Queue


# Discussion 4
## Scheduling
### Example 1.1 Round Robin T/F
### Example 1.2 Life Ain’t Fair
### Example 1.3 Bitcoin Mining



# Discussion 5

# Starvation
Starvation (饥饿)<br>
一个可执行的进程，被调度器持续忽略，以至于虽然处于可执行状态却不被执行<br>
- 可能导致没有线程去买面包：错误时间的上下文切换可能会导致每个线程都认为另外一个线程回去买面包<br>
- 最难处理的:极其不可能发生的事情也会发生在糟糕的时间；就像UNIX中的一些事情<br>
互斥:同一时间临界区中最多存在一个线程<br>
Progress:如果一个线程想要进入临界区，那么它最终会成功<br>
有限等待:如果一个线程i处于入口区，那么在i的请求被接受之前其他线程进入临界区的时间是有限制的<br>
无忙等待(可选):如果一个进程在等待进入临界区，那么在它可以进入之前会被挂起<br>
<br>
1. 解决方法 1 禁用硬件中断：它有效，但是真的不够好。因为太复杂了 -即使对这个简单的例子而言，难以说服你自己它真的有效。A和B的代码不同，每个线程的代码也会略有不同，如果线程过多怎么办?当A在等待的时候，其实是在消耗CPU的时间。这种情况叫做“忙等待(busy-waiting)<br>
2. 解决方法 2 基于软件的解决方法：没有中断，没有上下文切换，因此没有并发————硬件将中断处理延迟到中断被启用之后。>大多数现代计算机体系结构都提供指令来完成
- 进入临界区>禁用中断
- 离开临界区》开启中断
3. 解决方法 3 更高级的抽象：Critical section(临界区)<br>
临界区是指进程中的一段需要访问共享资源并且当另一个进程处于相应代码区域时便不会被执行的代码区域<br>

- 两个线程，TO和T1<br>
``` C Ti的通常结构
do{
enter section 进入区域
critical section临界区exit section 离开区域reminder section 提醒区域
} while (1); 

```
- 线程可能共享一些共有的变量来同步他们的行为<br>


``` C 共享变量 - 初始化
int turn =0;
turn==i//表示该谁进入临界区
Thread Ti
do{
    while (turn != i) ;
    critical section
    turn = j;
    reminder section
) while (1);

```
- 满足互斥，但是有时不满足progress
- (Ti做其他的事情，Tj想要继续运行，但是必须等待Ti处理临界区)

``` C 进程Pi的算法
do {
     flag[i] = TRUE;
     turn = j;
     while ( flag[j] && turn == j);
     CRITICAL SECTION
     flag[i]= FALSE;
     REMAINDER SECTION
}while (TRUE);

```

- 满足进程Pi和Pi之间互斥的经典的基于软件的解决方法(1981年)
- Use two shared data items
``` C 使用两个共享数据项
intturn;//指示该谁进入临界区
boolean flag[];//指示进程是否准备好进入临界区
```
``` C Code for ENTER CRITICAL SECTION
flag[i] = TRUE;
turn = j;
while (flag[j] && turn == j)
```
``` C Code for EXIT CRITICAL SECTION
flag[i] = FALSE;
```
``` C 进程Pi 的算法
flag[0]:= false flag[1]:= false turn := 0// or 1
do{
    flaglil = TRUE;
    while flagli) == true {
        if turn≠i{
            flagl[i] := false
            while turn ≠i{}
            flag[i] := TRUE
        }
    }
    CRITICAL SECTION
    turn :=j
    flag[i] = FALSE;
    EMAINDER SECTION
} while (TRUE);
```

- Bakery 算法
N个进程的临界区
    + 0进入临界区之前，进程接收一个数字
    + 得到的数字最小的进入临界区
    + 如果进程Pi和Pj收到相同的数字，那么如果ij，Pi先进入临界区，否则Pj先进入临界区
    + 编号方案总是按照枚举的增加顺序生成数字
- Dekker算法(1965):第一个针对双线程例子的正确解决方案
- Bakery算法(Lamport1979):针对n线程的临界区问题解决方案
- 复杂:需要两个进程间的共享数据项
- 需要忙等待:浪费CPU时间
- 没有硬件保证的情况下无真正的软件解决方案: Peterson算法需要原子的LOAD和STORE指令
Critical section (临界区) 临界区是指进程中的一段需要访问共享资源并且当另一个进程处于相应代码区域时便不会被执行的代码区域

## Strict Policy
## Deadlock
Deadlock (死锁)<br>
 - 两个或以上的进程，在相互等待完成特定任务，而最终没法将自身任务进行下去<br>

限制申请方式<br>
 - 互斥-共享资源不是必须的，必须占用非共享资源。<br>
 - 占用并等待-必须保证当一个进程请求的资源，它不持有任何其他资源。<br>
    + 需要进程请求并分配其所有资源，在它开始执行之前，或允许进程请求资源，仅当进程没有资源。<br>
    + 资源利用率低;可能发生饥饿。<br>
    + 操作系统与并发编程：基于 3-State Futex 的高效互斥锁实现

无抢占- <br>
如果进程占有某些资源，并请求其它不能被立即分配的资源，则释放当前正占有的资源<br>
被抢占资源添加到资源列表中<br>
只有当它能够获得旧的资源以及它请求新的资源，进程可以得到执行<br>
 - 循环等待
 - 对所有资源类型进行排序，并要求每个进程按照资源的顺序进行申请。<br>

需要系统具有一些额外的先验信息提供 <br>
 - 最简单和最有效的模式是要求每个进程声明它可能需要的每个类型资源的最大数目。
 - 资源的分配状态是通过限定提供与分配的资源数量，和进程的最大需求。
 - 死锁避免算法动态检查的资源分配状态，以确保永远不会有一个环形等待状态。

 - 当一个进程请求可用资源，系统必须判断立即分配是否能使系统处于安全状态。
 - 系统处于安全状态指:针对所有进程，存在安全序列。
 - 序列<PI，P2，...，PN>是安全的:针对每个Pi，Pi要求的资源能够由当前可用的资源+所有的P,持有的资源来满足，其中Ki。
    + 操作系统与并发编程：基于 3-State Futex 的高效互斥锁实现如果Pi,资源的需求不是立即可用，那么Pi,可以等到所有P,完成。
    + 操作系统与并发编程：基于 3-State Futex 的高效互斥锁实现当Pi+1,完成后，P.可以得到所需要的资源，执行，返回所分配的资源，并终
    + 操作系统与并发编程：基于 3-State Futex 的高效互斥锁实现用同样的方法，Pi+2，Pi+3和Pn。能获得其所需的资源。

### Example 1.1 Round Robin T/F

> **Problem Description:**  
> 在 Try #2 方案中，使用独立的 `bool maybe_waiters` 存在状态竞态（Data Race）和死锁风险。为了彻底解决这一问题，需要将“锁占用”与“等待状态”合并到一个原子变量中，采用 **三种状态（UNLOCKED, LOCKED, CONTESTED）** 实现高性能且绝对安全的用户态互斥锁（Mutex）。
> 
> *参考资料: Ulrich Drepper 经典论文 《Futexes Are Tricky》*

---

### 1. 状态定义与类型定义

```c
typedef enum { 
    UNLOCKED = 0,  // 0: 锁空闲，无线程占用
    LOCKED   = 1,  // 1: 锁被占用，但没有其他线程在休眠/等待
    CONTESTED = 2  // 2: 锁被占用，且“可能有”其他线程已经在内核中休眠等待
} Lock;

Lock mylock = UNLOCKED;


#include <linux/futex.h>
#include <sys/syscall.h>
#include <unistd.h>

/**
 * @brief 申请锁操作 (Acquire)
 * @param thelock 指向锁变量的指针
 */
void acquire(Lock *thelock) {
    // 1. 【Fast Path 快速路径】：尝试从 UNLOCKED 直接抢占为 LOCKED
    //    如果成功，说明完全无竞争，立刻返回！(无系统调用，性能极高)
    if (compare_and_swap(thelock, UNLOCKED, LOCKED) == UNLOCKED) {
        return;
    }

    // 2. 【Slow Path 慢速路径】：有竞争，进入循环抢锁
    //    原子的将锁状态强行设置为 CONTESTED (2)
    while (swap(thelock, CONTESTED) != UNLOCKED) {
        // 如果抢锁前它不是 UNLOCKED，说明锁仍被占用，必须陷入内核休眠
        // 只有当 *thelock 的实际值仍然为 CONTESTED 时，内核才会让当前线程休眠
        futex(thelock, FUTEX_WAIT, CONTESTED);
    }
}

/**
 * @brief 释放锁操作 (Release)
 * @param thelock 指向锁变量的指针
 */
void release(Lock *thelock) {
    // 原子地将锁重置为 UNLOCKED，并拿到释放前的值
    // 如果释放前的值是 CONTESTED (2)，说明有人可能在休眠，必须唤醒
    if (swap(thelock, UNLOCKED) == CONTESTED) {
        futex(thelock, FUTEX_WAKE, 1); // 唤醒 1 个睡眠中的线程
    }
}
```

一、 为什么“三状态”设计是最佳解决方案？<br>
在 Try #2 中，使用独立的 int mylock 和 bool maybe_waiters 两个变量，会导致锁状态与等待状态脱节，从而引发数据竞态（Data Race）和唤醒丢失（Lost Wakeup）造成的死锁。<br>
<br>
“三状态模型”的核心创新在于：<br>
将 “锁是否被占用” 与 “是否有线程在休眠（竞争）” 两个关键信息，压缩并合并到了同一个原子变量（thelock）中。<br>
通过这一设计，所有对锁和等待状态的修改均可通过 单条原子指令（CAS 或 Swap） 一步完成，彻底杜绝了状态不一致的问题。<br>
<br>
二、 核心代码逐行拆解<br>
1. acquire() —— 申请锁<br>
① Fast Path（无竞争快速路径）<br>
```C
if (compare_and_swap(thelock, UNLOCKED, LOCKED) == UNLOCKED)
    return;
```
原理解析：检查 *thelock 是否等于 UNLOCKED(0)，若是，原子的将其设为 LOCKED(1) 并返回旧值 UNLOCKED(0)。<br>
性能优势：在无竞争的理想情况下，仅执行一条硬件级 CPU 原子指令即完成加锁，完全不发起 futex 系统调用（Syscall-Free），耗时仅几纳秒。<br>
<br>
② Slow Path（有竞争慢速路径）<br>
```C
while (swap(thelock, CONTESTED) != UNLOCKED) {
    futex(thelock, FUTEX_WAIT, CONTESTED);
}
```
swap(thelock, CONTESTED)：无条件将 *thelock 设为 CONTESTED(2) 并返回旧值。
<br>
旧值为 UNLOCKED(0)：说明在上一步和本步骤之间恰好有线程释放了锁，当前线程成功抢占锁，跳出循环。
<br>
旧值为 LOCKED(1) 或 CONTESTED(2)：说明锁仍被占用。强制将锁状态标记为 CONTESTED，确保持锁线程解锁时知道有等待者存在。
<br>
futex(thelock, FUTEX_WAIT, CONTESTED)：
<br>
内核级防死锁（Atomic Sleep Check）：内核会在让线程进入休眠前，原子地校验 *thelock 当前是否仍等于 CONTESTED。若在调用瞬间有人释放了锁并修改了状态，futex 将立即返回而非陷入休眠，避免了“丢失唤醒”。<br>
<br>
2. release() —— 释放锁<br>
```C
if (swap(thelock, UNLOCKED) == CONTESTED) {
    futex(thelock, FUTEX_WAKE, 1);
}
swap(thelock, UNLOCKED)：将锁恢复为 UNLOCKED(0)，并返回释放前的状态。
```
分支逻辑校验：
<br>
释放前为 LOCKED(1)：说明在持锁期间无任何其他线程试图抢锁（否则状态会被抢锁线程修改为 2）。直接在用户态完成解锁，无需系统调用。<br>

释放前为 CONTESTED(2)：说明存在线程因抢锁失败已陷入内核休眠，必须发起 futex(..., FUTEX_WAKE, 1) 系统调用唤醒等待队列中的一个线程。<br>
<br>
三状态 Futex 互斥锁 巧妙利用 UNLOCKED(0)、LOCKED(1)、CONTESTED(2) 三种状态，将锁状态与等待队列标记融合在一起：

无竞争场景：完全在用户态（Userspace）完成，零系统调用开销。<br>

高竞争场景：依赖内核级 futex 系统调用精确控制线程休眠与唤醒，兼顾极致性能与并发安全性。<br>





# Discussion 6 Paging, Caches


# Discussion 7 I/O


# Discussion 8 Queueing Theory, File Systems
## Queuing Theory
### IPC 概述
1. 概述 进程通信的机制及同步;<br>
      不使用共享变量的进程通信;<br>
      + IPC facility 提供2个操作:send(message)-消息大小固定或者可变 receive(message);<br>
      + 如果P和Q想通信，需要: 在它们之间建立通信链路通过 send/receive交换消息<br>
      + 通信链路的实现物理(例如，共享内存，硬件总线),逻辑(例如，逻辑属性)<br>
2. 通信模型
3. 直接及间接通信       
      + 进程必须正确的命名对方:sendmessage)<br>
            1) send(P,message)-发送信息到进程P
            2) receive(Q,message)- 从进程Q接受消息
      + 通信链路的属性<br>
      1)自动建立链路<br>
      2)一条链路恰好对应一对通信进程<br>
      3)每对进程之间只有一个链接存在<br>
      4)链接可以是单向的，但通常为双向的<br>
<br>        
4. 阻塞与非阻塞<br>
- 消息传递可以是阻塞或非阻塞<br> 
- 阻断被认为是同步的<br>
          1)Blocking send has the sender block until the message is received<br>
          2)Blocking receive has the receiver block until a message is available<br>
- 非阻塞被认为是异步的      
          1)Non-blocking send has the sender send the message and continue<br>
          2)Non-blocking receive has the receiver receive a valid message or null<br>
       
5. 通信链路缓冲
- 队列的消息被附加到链路;可以是以下3种方式之一:
      1. 0容量-0messages<br>发送方必须等待接收方(rendezvous)
      2. 有限容量-nmessages的有限长度<br>发送方必须等待，如果队列满
      3. 无限容量-无限长度<br>发送方不需要等待
     
      
      
### IPC 
6. 信号 Signal(信号) 
      + 软件中断通知事件处理<br>
      + Examples: SIGFPE, SIGKILL, SIGUSR1, SIGSTOP, SIGCONT
      + 接收到信号时会发生什么
        1) Catch:指定信号处理函数被调用<br>
        2) Ignore:依靠操作系统的默认操作Example: Abort, memory dump, suspend or resume process<br>
        3) Mask:闭塞信号因此不会传送,可能是暂时的(当处理同样类型的信号)<br>
      + 不足:不能传输要交换的任何数据<br>
      <br>
      + <img width="580" height="316" alt="image" src="https://github.com/user-attachments/assets/b22aa9cc-b057-40c1-aa0e-96d1a991730a" /><br>

7. 管道
      + 子进程从父进程继承文件描述符: file descriptor 0 stdin, 1 stdout, 2 stderr
      + 进程不知道(或不关心!)从键盘，文件，程序读取，或写入到终端，文件，程序。
      + % ls | more
<img width="400" height="141" alt="image" src="https://github.com/user-attachments/assets/366c272e-d7b7-465e-a675-a1ddefd3e939" />

8. 消息队列
消息队列按FIFO的来管理消息
      + Mlessage:作为一个字节序列存储
      + Message Queues:消息数组
      + FIFO & FILO configuration
      + <img width="837" height="327" alt="image" src="https://github.com/user-attachments/assets/9b1672aa-2309-4e65-bfb7-356fba64e67e" />

9. 共享内存
      + 进程: 每个进程都有私有地址空间;在每个地址空间内，明确地设置了共享内存段
      + 优点:快速、方便地共享数据
      + 不足:必须同步数据访问
      + <img width="837" height="391" alt="image" src="https://github.com/user-attachments/assets/7dc66f17-608d-4d6d-90a5-f0b4cc14a34b" />
      <br>
      + 最快的方法
      + 一个进程写另外一个进程立即可见<br>
      + 没有系统调用干预<br>
      + 没有数据复制<br>
      + 不提供同步<br>
      + 由程序员提供同步<br>

##  File Systems
1. 基本概念<br>
1）文件系统和文件<br>
      + 文件系统:一种用于持久性存储的系统抽象 <br>
      + 在存储器上:组织、控制、导航、访问和检索数据，大多数计算机系统包含文件系统；个人电脑、服务器、笔记本电脑；iPod、Tivo /机顶盒、手机/掌上电脑；Google可能是由一个文件系统构成的<br>
      + 文件:文件系统中一个单元的相关数据在操作系统中的抽象<br>
2）文件系统功能<br>
      + 分配文件磁盘空间：<br>1）管理文件块(哪一块属于哪一个文件)<br>2）管理空闲空间(哪一块是空闲的)<br>3）分配算法(策略)<br>
      + 管理文件集合<br>1）定位文件及其内容<br>2）命名:通过名字找到文件的接口<br>3）最常见:分层文件系统<br>4）文件系统类型(组织文件的不同方式)<br>
> 提供的便利及特征
      + 1）保护:分层来保护数据安全<br>
      + 2）可靠性/持久性:保持文件的持久即使发生崩溃、媒体错误、攻击等<br>
 <br>
 3）文件属性：名称、类型、位置、大小、保护、创建者、创建时间、最近修改时间、...<br>
 <br>
 4） 文件头<br>
      + 在存储元数据中保存了每个文件的信息
      + 保存文件的属性
      + 跟踪哪一块存储块属于逻辑上文件结构的哪个偏移
 <br>
 5）文件描述符<br>
      + 文件使用模式:使用程序必须在使用前先“打开”文件
      + 内核跟踪每个进程打开的文件:操作系统为每个进程维护一个打开文件表
<br>      
      + 使用程序必须在使用前先“打开”文件
      + 内核跟踪每个进程打开的文件:操作系统为每个进程维护一个打开文件表
  <br>
  需要元数据数据来管理打开文件:  <br>
1. 文件指针:指向最近的一次读写位置，每个打开了这个文件的进程都这个指针
2. 文件打开计数:记录文件打开的次数 -当最后一个进程关闭了文件时，允许将其从打开文件表中移除
3. 文件磁盘位置:缓存数据访问信息
4. 访问权限:每个程序访问模式信息
用户视图:持久的数据结构 <br>
系统访问接口:字节的集合(UNIX)系统不会关心你想存储在磁盘上的任何的数据结构! <br>
操作系统内部视角: <br>
      + 块的集合(块是逻辑转换单元，而扇区是物理转换单元) 
      + 块大小<>扇区大小;在UNIX中，块的大小是4KB
当用户说:给我2-12字节空间时会发生什么
获取字节所在的块
返回块内对应部分
如果说要写2-12字节呢?
获取块
修改块内对应部分
写回块
在文件系统中的所有操作都是在整个块空间上进行的
举个例子，getcO，putc(:即使每次只访问1字节的数据，也会
缓存目标数据4096字节


无结构
单词、比特的队列
简单记录结构
列
固定长度
可变长度
复杂结构
格式化的文档(如，MSWord，PDF)
可执行文件

多用户系统中的文件共享是很必要的
访问控制
谁能够获得哪些文件的哪些访问权限
访问模式:读、写、执行、删除、列举等
文件访问控制列表(ACL)
>〈文件实体，权限
Unix模式
>《用户|组|所有人，读|写|可执行》
用户ID识别用户，表明每个用户所允许的权限及保护模式
组ID允许用户组成组，并指定了组访问权限

指定多用户/客户如何同时访问共享文件
和过程同步算法相似
因磁盘I/0和网络延迟而设计简单
Unix文件系统(UFS)语义
对打开文件的写入内容立即对其他打开同一文件的其他用户可见
共享文件指针允许多用户同时读取和写入文件
会话语义
写入内容只有当文件关闭时可见
锁
一些操作系统和文件系统提供该功能

6) 目录<br>
文件以目录的方式组织起来
目录是一类特殊的文件
每个目录都包含了一张表<name，pointer to file header>
目录和文件的树型结构
早期的文件系统是扁平的(只有一层目录)
层次名称空间
<br>
>典型操作
搜索文件
创建文件
删除文件
枚举目录
重命名文件
>在文件系统中遍历一个路径
操作系统应该只允许内核模式修改目录
确保映射的完整性
应用程序能够读目录(如1s)
>
8) 文件别名<br>


9) 文件系统种类<br>


2. 虚拟文件系统
3. 数据块缓存
4. 打开文件的数据结构文件分配
5. 空闲空间列表
6. 多磁盘管理-
7. RAID
8. 磁盘调度
# Discussion 9 File Systems, Reliability



#  Discussion 10 Distributed Systems

##  4.3 Primes

> **Problem Description:**  
> Edward realizes his previous solution was insufficient, so he decides to implement a slightly more complicated protocol.
> 
> **The client will perform the following:**  
> (a) Send an identifier for the function it wants as an integer (`0` for `ith_prime`, `1` for `is_coprime`).  
> (b) Send all bytes for all the arguments.  
> 
> **The server will then perform the following:**  
> (a) Read identifier.  
> (b) Use identifier to allocate memory and set read size.  
> (c) Read arguments.  


```c
/* Converts NETLONG from network byte order to host byte order. */
uint32_t ntohl(uint32_t netlong);

/* Converts NETLONG from host byte order to network byte order. */
uint32_t htonl(uint32_t hostlong);

/* Converts NETLONG from network byte order to host byte order. */
uint32_t ntohl(uint32_t netlong);
/* Converts NETLONG from host byte order to network byte order. */
uint32_t htonl(uint32_t hostlong);
void receive_rpc (int sock_fd) {
/* Read in procedure identifier. */
uint32_t id;
int bytes_read = 0, cur_read = 0;
while (_______________) {
bytes_read += cur_read;
}
id = _______________;
/* Get sizes and allocate space for arguments and return values. */
char *args, *rets;
size_t arg_bytes, ret_bytes;
get_sizes(id, &args, &arg_bytes, &rets, &ret_bytes);
/* Read in arguments. */
bytes_read = 0;
while (_______________) {
bytes_read += cur_read;
}
/* Call appropriate server stub stub function based on id. */
call_server_stub(id, args, arg_bytes, rets, ret_bytes);
/* Write return values. */
int bytes_written = 0, cur_written = 0;
while (_______________) {
bytes_written += cur_written;
}
/* Clean up socket. */
_______________;
}
void receive_rpc (int sock_fd) {
/* Read in procedure identifier. */
uint32_t id;
int bytes_read = 0, cur_read = 0;
while ((cur_read = read(sock_fd, ((char *) &id) + bytes_read,
sizeof(uint32_t)- bytes_read)) > 0) {
bytes_read += cur_read;
}
id = ntohl(id);
/* Get sizes and allocate space for arguments and return values. */
char *args, *rets;
size_t arg_bytes, ret_bytes;
get_sizes(id, &args, &arg_bytes, &rets, &ret_bytes);
/* Read in arguments. */
bytes_read = 0;
while ((cur_read = read(sock_fd, &args[bytes_read],
arg_bytes- bytes_read)) > 0) {
bytes_read += cur_read;
}
/* Call appropriate server stub stub function based on id. */
call_server_stub(id, args, arg_bytes, rets, ret_bytes);
/* Write return values. */
int bytes_written = 0, cur_written = 0;
while ((cur_written = write(sock_fd, &rets[bytes_written],
ret_bytes- bytes_written)) > 0) {
bytes_written += cur_written;
}
/* Clean up socket. */
close(socket_fd);
}

```

一、 解题思路<br>
在网络编程中，基于 TCP 套接字 (Socket) 实现远程过程调用（RPC）时，必须克服 TCP 协议自身的两个核心特性：流式传输特性（半包/粘包） 与 网络字节序差异（Endianness）。<br>

为此，自定义 RPC 协议采用了 “固定长度 Header（标示符） + 动态长度 Payload（变长参数）” 的设计范式。服务端处理请求的整体逻辑遵循严格的递进关系：<br>
<br>
[接收固定 Header (4B)] ➔ [网络字节序转换] ➔ [查找/分配 Payload 内存] ➔ [接收完整 Payload] ➔ [执行本地存根] ➔ [写回结果] ➔ [关闭连接]
整体步骤拆解<br>
读取标示符 (id)：客户端首先发送 4 字节的整数 id，指定要调用的函数。服务端需通过循环读取确保 4 字节完全接收。<br>

转换网络字节序：网络传输采用统一的大端序，接收后需立即转换为本地主机字节序，保证 id 值正确。<br>

元数据解析与内存分配 (get_sizes)：依据 id 检索参数量与返回值所需空间，并在堆区动态申请缓冲区内存。<br>

读取变长参数 (args)：根据解出的参数长度，再次通过循环读取，确保所有参数字节完整入缓冲区。<br>

执行本地存根 (call_server_stub)：参数就绪后，调用目标函数处理请求，并将输出结果写入返回缓冲区。<br>

循环写回结果 (rets)：将返回数据写回客户端，同样需采用循环写入防范 partial write（部分写入）。<br>

资源回收 (close)：关闭套接字文件描述符，释放系统网络资源。<br>

二、 代码详解
1. 标示符读取与字节序转换
```C
uint32_t id;
int bytes_read = 0, cur_read = 0;

// 1. 循环读取 4 字节 Header
while ((cur_read = read(sock_fd, ((char *) &id) + bytes_read, sizeof(uint32_t) - bytes_read)) > 0) {
    bytes_read += cur_read;
}

// 2. 将网络大端序转换为主机字节序
id = ntohl(id);
((char *) &id) + bytes_read：&id 获取 id 的首地址；强转为 char * 可确保指针算术运算以单字节为单位偏移；加上 bytes_read 确保后续数据精准写入尚未填充的内存空位。

sizeof(uint32_t) - bytes_read：动态计算剩余待接收字节数，防止缓冲区溢出。

ntohl(id)：Network to Host Long，将网络标准的 Big-Endian 序转换为本地 CPU 的 Endian 序。
```
2. 动态内存分配与参数接收
```C
char *args, *rets;
size_t arg_bytes, ret_bytes;

// 1. 根据 id 获取参数/返回值尺寸并分配内存
get_sizes(id, &args, &arg_bytes, &rets, &ret_bytes);

// 2. 循环读取 Payload 参数
bytes_read = 0;
while ((cur_read = read(sock_fd, &args[bytes_read], arg_bytes - bytes_read)) > 0) {
    bytes_read += cur_read;
}
get_sizes(...)：依据解析正确的 id 查找输入/输出大小，并初始化 args 与 rets 指针。

&args[bytes_read]：由于 args 本身为 char * 类型，采用数组下标取地址法 &args[bytes_read] 等价于指针偏移算法，使得代码更为清晰直观。
```
3. 本地函数执行与结果写回
```C
// 1. 调用本地 Stub 执行目标逻辑
call_server_stub(id, args, arg_bytes, rets, ret_bytes);

// 2. 循环写回计算结果
int bytes_written = 0, cur_written = 0;
while ((cur_written = write(sock_fd, &rets[bytes_written], ret_bytes - bytes_written)) > 0) {
    bytes_written += cur_written;
}

// 3. 清理连接套接字
close(sock_fd);
```
write(...) 循环：TCP 发送缓冲区满或网络拥堵时，write 同样可能发生部分写入（Partial Write），必须依靠 while 循环推动 &rets[bytes_written] 指针，直到发送完所有的 ret_bytes。<br>

close(sock_fd)：正确关闭通信套接字。（注：原题伪代码中 close(socket_fd) 存在变量名拼写笔误，应为 sock_fd）。<br>

三、 代码间强关联与链条关系<br>
各个步骤构成一个紧密耦合的逻辑链条，前一步骤的正确性直接决定后续步骤的执行：<br>
<img width="578" height="132" alt="image" src="https://github.com/user-attachments/assets/c0f6588a-ed23-456b-a9eb-878d5986e707" /><br>


read id 决定 ntohl 的有效性：只有在读满 4 字节后，ntohl 转换出来的数字才具有业务意义，否则会将垃圾数据进行字节序转换。<br>

ntohl 决定 get_sizes 的正确性：若遗漏字节序转换，大端序下的数值 1 在小端序机器上会被解析为 16,777,216，导致 get_sizes 尝试申请内存巨幅超载，引发崩溃。<br>

get_sizes 决定后续读写循环的终止条件：其算出的 arg_bytes 与 ret_bytes 构成了 read args 与 write rets 两个 while 循环退出的基准边界。
<br>
这道经典考题的核心在于评估开发者在系统级网络编程中对底层协议细节与内存控制的理解：<br>

网络字节序 (Endianness Consistency)：掌握 ntohl() / htonl() 在跨平台、跨 CPU 架构通信时的数据对齐与转换作用。<br>

TCP 流式传输与粘包/半包处理 (Stream-oriented I/O)：理解 TCP 无数据边界（Boundaryless）特性，掌握使用 while 循环配合指针偏移量计算和剩余字节数校验来实现可靠的拆包/组包操作。<br>

自定义应用层协议设计 (Application-Layer Protocol Design)：掌握通过“固定 Header + 动态 Payload”的方式显式划分数据帧边界的基本范式。<br>
