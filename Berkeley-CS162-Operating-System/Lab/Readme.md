Table of Content 
[](https://cs162.org/static/hw/hw-intro/docs/executable/)

# Set up
[Deatils of set up](https://claude.ai/code/artifact/fe196d6b-ee4b-4695-b998-1eb7c8fc7310)

## VS Code

## Docker
Write in Docker
``` 
Help you set up Docker to work with this repository instead AND Clone the workspace repo
git clone https://github.com/Berkeley-CS162/cs162-workspace.git cd cs162-workspace⧉
Build and start it once, in the foreground
Wait for the line Docker workspace is ready!, then stop it with Ctrl+C.

docker-compose up⧉
Start it again, this time in the background
docker-compose up -d⧉
Sanity-check it over SSH
Password is workspace.

ssh workspace@127.0.0.1 -p 16222⧉
Optional: name it so you don't retype the port
Add to ~/.ssh/config:

Host docker162 HostName 127.0.0.1 Port 16222 User workspace⧉
Then it's just ssh docker162 from anywhere, including VS Code.
```
To close Docker
```
docker-compose down
```
To restart it later, just run

```
docker-compose up -d
```
### use VS Code Remote SSH:

Open VS Code
Press Ctrl+Shift+P
Search "Remote-SSH: Connect to Host"
Select docker162
Enter password: workspace

# HW 0
[HW0 Description](https://cs162.org/static/hw/hw-intro/docs/executable/)
## Steps

###

## Questions
From source code to executable
Now that you’ve seen how map works, let’s take a dive into how we went from high-level C code to an executable.

Before we start, we’ll be using a few compiler flags which are likely new to you. Here’s a summary of the flags we’ll be using.

- Wall – Enables all compiler warnings

- m32 – Compiles the code for the i386 architecture.

- E - Invokes the PREPROCESSOR only.

- S – Invokes the COMPILER only.

- c – Invokes the COMPILER and ASSEMBLER only.

Important: Please use i386-gcc instead of gcc for this homework.

Let’s now invoke the compiler. The compiler takes high-level C code and produces a variant of x86 known as 8086 or i386 assembly.

To compile map.c, run:
```
i386-gcc -m32 -S -o map.S map.c
```
This will only invoke the compiler for map.c and output the assembly code in map.S.

1. Generate recurse.S and find which instructions correspond to the recursive call of recur(i - 1).
Now we will assemble our compiled code into an executable. To assemble our code we can run:
```
i386-gcc -m32 -c map.S -o map.o
```
This turns our raw x86 code (map.S) into machine code or an object file (map.o).

We can also combine these steps by just running i386-gcc -m32 -c on our C file directly. We can run:
```
i386-gcc -m32 -c recurse.c -o recurse.o
```
The assembler converts the raw assembly code into an object file which contains code as well as other data and metadata necessary for execution. Different operating systems use different types of object files. In this class, we will be using ELF (Executable and Linkable Format), the object format used by Linux. Let’s start by taking a look at map.o and recurse.o. These are object files, so we will use the objdump program to read them.
```
i386-objdump -D map.o
i386-objdump -D recurse.o
```

2. What do the .text and .data sections contain? Provide a qualitative description.
The assembler generates a symbol table which is part of the object file. The symbol table contains all the symbols that can be globally referenced (referenced outside the object file) from another object file (i.e. global/static variables and functions).

3. What command do we use to view the symbols in an ELF file? (Hint: We can use objdump again, look at man objdump to find the right flag).
Here’s an excerpt from the map.o symbol table:
```
00000000 g O .data 00000004 stuff
00000000 g F .text 00000060 main
...
00000000 *UND* 00000000 malloc
00000000 *UND* 00000000 recur
```
4. What do the g, O, F, and *UND* flags mean?
Finally, let’s link our 2 object files to create an executable.
```
i386-gcc -m32 map.o recurse.o -o map
```
Note that we could’ve just called i386-gcc -m32 map.c recurse.c -o map on the C files to do this entire process in a single command. Often times build systems will separate these commands in order to speed up compile times (since only the changed files need to be recompiled).

5. Examine the symbol table of the entire map program now. What has changed? Give a general description, including what happened to recur.
objdump can be used to look at more than just the symbol table—it can show us the structure of the executable. Run i386-objdump -x -d map. You will see that your program has several segments, names of functions and variables in your program correspond to labels with addresses or values. The guts of everything is chunks of stuff within segments.

In the objdump output these segments are under the section heading. There’s actually a slight nuance between these two terms which you can read more about online.

Using the output of objdump, answer the following questions:

6. What segment(s)/section(s) contains recur (the function)? (The address of recur in objdump will not be exactly the same as what you saw in gdb. An optional stretch exercise is to think about why. Hint: See the Wikipedia article on relocation.)

7. What segment(s)/section(s) contains global variables? Hint: look for the variables foo and stuff.

8. Do you see the stack or heap segment anywhere? Explain.

9. Based on the output of map, in which direction does the stack grow? (Reminder: Please use i386-exec ./map to run map.)

When you ran map, you might have noticed that it prints "CS362 is the best!". However, we wanted to print "CS162 is the best!".

Let’s see what happened by invoking the preprocessing stage. The compiler takes your C code and will output new C code. What does this really do? Time to find out!

To preprocess map.c, run:
```
i386-gcc -m32 -E -o map.i map.c
```
10. You can see that gcc produces a map.i that is far larger than the original map.c file. Notice that define directives perform string replacement.

11. Modify Makefile to make sure that "CS162 is the best!" is printed instead. You may not modify or add any other files. Hint: Refer to this page from the GCC documentation.

# HW 1
[](https://cs162.org/static/hw/hw-list/)
