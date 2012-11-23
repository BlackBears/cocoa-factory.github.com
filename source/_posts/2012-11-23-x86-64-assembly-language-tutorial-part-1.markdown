---
layout: post
title: "x86_64 Assembly Language Tutorial: Part 1"
date: 2012-11-23 04:21
comments: false
categories: x86_64, assembly language, tutorial, macos
---
The majority of the time, Cocoa developers work at a such a high level of abstraction that we almost forget that all of those abstractions ultimately interact with silicon at the level of machine language.  Few of us will ever need to write such performance-critical code that we need to hand-write assembly language code; but a rudimentary understanding of it will help developers understand how the compiler behaves and how our objects that live in the upper levels of abstraction actually work.  If for no other reason, a passing familiarity with x86\_64 assembly language will comfort the developer a little when the debugger stops on some line of assembly code.  With that, let's dive in.

Registers are the "variables" at a hardware level.  In the x86\_64 architecture, the registers are 64 bits wide, of course; but they have 32, 16, and 8 bit sub-registers that are used for particular instructions.  The following table shows some of those relationships:

64-bit register | Lower 32 bits | Lower 16 bits | Lower 8 bits
--------------- | ------------- | ------------- | ------------
rax	       | eax           | ax            | al
rbx             | ebx           | bx            | bl
rcx             | ecx           | cx            | cl
rdx             | edx           | dx            | dl
rsi             | esi           | si            | sil
rdi             | edi           | di            | dil
rbp             | ebp           | bp            | bpi
rsp             | esp           | sp            | spi
r8              | r8d           | r8w           | r8b

(`r8`-`r15` follow the same convention.)  The `rip` register is the instruction pointer register which points to the instruction being executed.  As we go through this series, we'll introduce more information about the x86\_64 registers.  Before we move on, it's important to note that although there are dozens of registers that the compiler can use, their use is restricted by several factors:

1. **Width of register** - an 8 bit register can't hold a 64-bit value.  (Duh.)
2. **Instructions** - instructions operate on certain types of registers.
3. **Application binary interface** - this is the low-level equivalent of an API, specifying data types, widths, calling conventions, etc.

Our example is a simple C program that prints the first 16 integers:

### C code ###

{% gist 4135115 %}

We've compiled this using no optimizations to see how the compiler behaves when it can't optimize.  In the next installment, we'll take a look at the effect of optimizations on the compiled code.

### Disassembly ###
{% gist 4134960 %}
    
### Step-by-step ###

#### Step 1 ####

``` c-objdump
0x100000ec0:  pushq  %rbp
0x100000ec1:  movq   %rsp, %rbp
```
    
This is a standard preamble for a C function.  `pushq %rbp` saves the base pointer to the stack so that it can be restored later (see `popq %rbp`…)  Then `movq %rsp, %rbp` copies `rsp` to `rbp` setting the base pointer (temporarily for our function) to the stack pointer.  That way we can push variables to the stack if our function requires it.

#### Step 2 ####

``` c-objdump
0x100000ec4:  subq   $48, %rsp
```
    
Grow our stack 48 bytes upward.  The compiler evidently concluded that our function may need 48 bytes of stack space for its use.

#### Step 3 ####

``` c-objdump
0x100000ec8:  movl   $0, -4(%rbp)
```
    
Store a 32-bit zero at the bottom of the stack.  As the stack grows, we use lower-numbered addresses in memory.  Here, the bottom four bytes of the stack are set to zero.  Presumably this is just to make sure we've cleared the stack that we are using.

#### Step 4 ####

``` c-objdump
0x100000ecf:  movl   %edi, -8(%rbp)
```
    
Since `edi` is the first integer argument register - and the lower 32 bits of `rdi` we are pushing this to the stack 8 bytes up.  Since we set the bottom 4 bytes to zero, we are ensuring that the 64 bit value now on the stack is just the value of `argc` from the C code.  Makes sense?

#### Step 5 ####

``` c-objdump
0x100000ed2:  movq   %rsi, -16(%rbp)
```

Now we're saving `rsi` another 8 bytes up the stack.

#### Step 6 ####

``` c-objdump
0x100000ed6:  callq  0x100000f2e
```

Like the comment says, we're calling the start of the autorelease pool.  Remember, we're looking at the disassembly.  If we were looking at the assembly instead, we'd see something like this:  `callq	_objc_autoreleasePoolPush`

#### Step 7 ####

``` c-objdump
0x100000edb:  movb   $0, -17(%rbp)
```

What's going on here?  We're pushing a an 8-bit zero to a position 17 bytes up the stack.  I'll bet this refers to initializing our `uint8_t` loop variable `i` to zero.  We'll see in a minute.

#### Step 8 ####

``` c-objdump
0x100000edf:  movq   %rax, -32(%rbp)
```
    
Now we're moving the accumulator register `rax` to the stack.

#### Step 9 ####

``` c-objdump
0x100000ee3:  movzbl -17(%rbp), %eax
```
   
Here's an instruction that we haven't seen yet: `movzbl`.  This instruction means "move byte to long" so we are moving a byte at `-17(%rbp)` (remember that was hypothesized to be our loop variable…) to the register `%eax` which, you will recall, is the lower 32 bits of `%rax`.  Since we're moving it back off the stack, I wonder if we're getting ready to compare it to our maximum loop index?

#### Step 10 ####

``` c-objdump
0x100000ee7:  cmpl   $16, %eax
```
    
Yep!  `cmpl` means "compare long", so we are comparing decimal 16 to `%eax` the lower 32 bits of the accumulator - the place where we just pulled our byte-to-long off the stack.  Now we'd expect to see some conditional jump next, I think.

#### Step 11 ####

``` c-objdump
0x100000eec:  jge    0x100000f14               ; main + 84 at main.m:19
```

And right again!  

The `jge` instruction is "jump when greater than or equal to"; so if `%eax` is greater than or equal to 16, we'll jump to `0x100000f14` which, if you look ahead pops our autorelease pool and restores the stack, etc. thereby finishing the function.

#### Step 12 ####

``` c-objdump
0x100000ef2:  leaq   113(%rip), %rdi           ; "i = %d\n"
```
    
If we reach this point, our comparison failed and the integer is under 16.  So all that was left to do in the C code is to print it.  In this case, we're loading an address 113 decimal bytes ahead of our current instruction pointer into `%rdi`.  Without seeing what's beyond the end of the function in memory and without the comment, we'd be lost.  But the disassembler gives us a comment that tells us that this points to the format string that we'll use to print the integer value.  But why did the compiler chosse `%rdi` for the this argument?  The answer is buried in the [x86_64 ABI](http://www.x86-64.org/documentation/abi.pdf) in the section 3.2.3. which states that register `%rdi` is used to pass the first argument to functions.

#### Step 13 ####

``` c-objdump
0x100000ef9:  movzbl -17(%rbp), %esi
```
    
Again, our "move byte to long" instruction.  This time moving the integer value to `%esi`.  I bet that's what `printf` is expecting as the integer argument.  Referring again to the [x86_64 ABI](http://www.x86-64.org/documentation/abi.pdf), `%rsi` is used to pass the second argument to functions.  Since `%esi` is the lower 32 bits of `%rsi`, this makes sense.

#### Step 14 ####

``` c-objdump
0x100000efd:  movb   $0, %al
```
    
This is the tricky bit to understand without references to the ABI.  `%rax` is used to pass information about the number of vector registers that are used.  It is also the 1st return register.  Register `%al` is the lower 8 bits of `%rax` so in this instruction we are setting the number of vector registers to zero.  Since we are simply printing an integer, no need for vector registers.

#### Step 15 ####

``` c-objdump
0x100000eff:  callq  0x100000f34               ; symbol stub for: printf
```
    
Now we find the call to `printf` after the setup.  In assembly, this would have been `callq _printf`.

#### Step 16 ####

``` c-objdump
0x100000f04:  movl   %eax, -36(%rbp)
```
    
This action (which is not really necessary) is a "Spill"; if we look at the assembly it labels it as such:

``` c-objdump
movl	%eax, -36(%rbp)         ## 4-byte Spill
```
    
When optimizations are off, the compiler looks ahead and sees that we need `%eax` or one of its sub-registers and pushes it to the stack.  That is a Spill.

#### Step 17 ####

``` c-objdump
0x100000f07:  movb   -17(%rbp), %al
```
    
Now we get the integer loop value at `-17(%rbp)` off the stack into `%al`.  The upcoming use of `%al` is the reason that the compiler spilled the 4 bytes of `%eax` in the last instruction.

#### Step 18 ####

``` c-objdump
0x100000f0a:  addb   $1, %al
0x100000f0c:  movb   %al, -17(%rbp)
```
	
Now, as expected, the increment and move back to the stack.

#### Step 19 ####

``` c-objdump
0x100000f0f:  jmpq   0x100000ee3               ; main + 35 at main.m:16
```
    
Finally, the end of our loop.  Jump back to address `0x100000ee3` which is step 9 above and the setup for our loop index comparison.

#### Step 20 ####

``` c-objdump
0x100000f14:  movq   -32(%rbp), %rdi
```
    
Now we get `%rdi` off the stack; this is the opposite of a Spill - a Reload.

#### Step 21 ####

``` c-objdump
0x100000f18:  callq  0x100000f28               ; symbol stub for: objc_autoreleasePoolPop
```

Pop our Objective-C autorelease pool.

#### Step 22 ####

``` c-objdump
0x100000f1d:  movl   $0, %eax
```

Remember that `%rax` is our first return register; so this instruction is simply setting our return to 0.

#### Step 23 ####

``` c-objdump
0x100000f22:  addq   $48, %rsp
0x100000f26:  popq   %rbp
0x100000f27:  ret
```
  
Restore the stack pointer and the base pointer. And return.

With that, our first installment on x86\_64 comes to a close.  We hope this was a helpful first introduction to x86\_64 assembly language and that you will find it useful in understanding and debugging your applications.

Question? Comments?  Tweet Alan @NSBum.
