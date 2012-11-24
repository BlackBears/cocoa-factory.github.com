---
layout: post
title: "x86_64 Assembly Language Tutorial: Part 3"
date: 2012-11-23 08:48
comments: false
categories: x86_64 assembler tutorial macos
---
In our [first tutorial](http://cocoafactory.com/blog/2012/11/23/x86-64-assembly-language-tutorial-part-1/) in this series, we presented a simple program in C and analyzed its x86_64 disassembly.  We extended the discussion in [Part II](http://cocoafactory.com/blog/2012/11/23/x86-64-assembly-language-tutorial-part-2/) to show register usage according to the x86_64 ABI.

Now, we're going to start to tiptoe gently into the world of Objective-C objects and use that as a platform for peeking into what ARC does to our code behind our backs.  Let's get started with a very simple Objective-C program:

{% gist 4135901 %}

Build and run this little application in Xcode and use the Assistant editor to review its disassembly just like we showed in [Part II](http://cocoafactory.com/blog/2012/11/23/x86-64-assembly-language-tutorial-part-2/):

{% gist 4135938 %}

### Step-by-step ###

Just looking over the disassembly, we see there are many more calls to functions in the Objective-C runtime; this will keep us busy in the step-by-step analysis.

#### Step 1 - Preamble ####

``` c-objdump
0x100000de0:  pushq  %rbp
0x100000de1:  movq   %rsp, %rbp
0x100000de4:  pushq  %r15
0x100000de6:  pushq  %r14
0x100000de8:  pushq  %rbx
0x100000de9:  pushq  %rax
```

As with our previous forays into the world of assembly language, the function call to `main` starts with the typical preamble where we push the base pointer, move the current stack pointer to the base pointer then push several registers that we're going to use later. 

#### Step 2 - Begin autorelease pool  ####

``` c-objdump
0x100000dea:  callq  0x100000e52               ; symbol stub for: objc_autoreleasePoolPush
0x100000def:  movq   %rax, %r14
```

Here we call the `objc_autoreleasePoolPush` function in the Object-C runtime and save its return value - presumably a reference to the autorelease pool to the register `%r14`.

#### Step 3 - Allocating a `Foo` ####

``` c-objdump
0x100000df2:  movq   903(%rip), %rdi           ; (void *)0x0000000100001190: Foo
0x100000df9:  leaq   880(%rip), %rsi           ; { /usr/lib/libobjc.A.dylib`objc_msgSend_vtable1, "alloc" }
0x100000e00:  callq  *874(%rip)                ; { /usr/lib/libobjc.A.dylib`objc_msgSend_vtable1, "alloc" }
```

From the comments, it looks like we're going to allocate a new `Foo`.  The instruction pointer offsets are specified, but they don't really tell us that much.  If we want to learn more, we can switch Xcode's Assistant view to Assembly.  Let's looks at this very carefully.

The first instruction corresponds to the following in the assembly code:

``` c-objdump
movq	L_OBJC_CLASSLIST_REFERENCES_$_(%rip), %rcx
movq	%rcx, %rdi
```

The value from the executable that we are loading into `%rdi` is `rip` + `L_OBJC_CLASSLIST_REFERENCES_$_`.  Let's look at what's at that location:

``` c-objdump
L_OBJC_CLASSLIST_REFERENCES_$_:
	.quad	_OBJC_CLASS_$_Foo

	.section	__TEXT,__objc_methname,cstring_literals
```

`.quad` is an assembler directive that emits an 8-byte integer, in this case the symbol `_OBJC_CLASS_$_Foo`.  Rather than go further down the rabbit hole at this stage, let's just say that this is a reference to the class `Foo` which we are loading into `%rdi`.

Then we have the instruction `0x100000df9:  leaq   880(%rip), %rsi` in the disassembly.  Turning again to the **assembly** code again:

``` c-objdump
leaq	l_objc_msgSend_fixup_alloc(%rip), %rsi
```

We're going to digress for a second examine this symbol `objc_msgSend_fixup_alloc` because it tells us something about the Objective-C runtime.  Most Objective-C methods get dispatched using a hash table in 	`objc_msgSend`.  But some of the most commonly used method are dispatched using a virtual table as a runtime optimization.  In fact, if we look at the comment `{ /usr/lib/libobjc.A.dylib 'objc_msgSend_vtable1', "alloc" }` we can see evidence of the obtimization.  The function `objc_msgSend_vtable1` is the vtable-referenced version of `objc_msgSend` for `alloc`.  For completeness, others include:

Optimized objc_msgSend  | Referenced method
----------------------  | -----------------
`objc_msgSend_vTable0`  | `allocWithZone:`
`objc_msgSend_vTable1`  | `alloc`
`objc_msgSend_vTable2`  | `class`
`objc_msgSend_vTable3`  | `self`
`objc_msgSend_vTable4`  | `isKindOfClass:`
`objc_msgSend_vTable5`  | `respondsToSelector:`
`objc_msgSend_vTable6`  | `isFlipped`
`objc_msgSend_vTable7`  | `length`
`objc_msgSend_vTable8`  | `objectForKey:`
`objc_msgSend_vTable9`  | `count`
`objc_msgSend_vTable10` | `objectAtIndex:`
`objc_msgSend_vTable11` | `isEqualToString:`
`objc_msgSend_vTable12` | `isEqual:`


Back to our function call setup; we've loaded a reference to the class `Foo` into `%rdi` which according to the [x86_64 ABI](http://www.x86-64.org/documentation/abi.pdf) is the first argument to a function.  And we've loaded `objc_msgSend_vTable1` into `%rsi` which is the second argument to a function.  All that's left is to call a function.  Turning to the **assembly** code again, we see that the calling instruction is `callq	*l_objc_msgSend_fixup_alloc(%rip)` meaning that we are calling the address of the symbol `l_objc_msgSend_fixup_alloc` + the address of the instruction pointer.  Following the symbol `l_objc_msgSend_fixup_alloc` further in the assembly, there is a `.quad` value of `_objc_msgSend_fixup` there.  So, we're calling `objc_msgSend_fixup` with the parameters of `Foo` and `alloc`, thereby allocating a `Foo`.  Whew.

#### Step 4 - Initializing a `Foo` ####

Having allocated a `Foo` we'll probably have to initialize it.  Here's the assembly code that does it:

``` c-objdump
0x100000e06:  movq   843(%rip), %rsi           ; "initWithBar:"
0x100000e0d:  movq   508(%rip), %r15           ; (void *)0x00007fff871c3240: objc_msgSend
0x100000e14:  movq   %rax, %rdi
0x100000e17:  movl   $15, %edx
0x100000e1c:  callq  *%r15
```

We move a reference to `initWithBar` into `%rsi` which is always our 2nd argument register.  Then we move `objc_msgSend` to `%r15` which we later call.  The instruction `movq   %rax, %rdi` moves the object returned by `alloc` to `%rdi` which is our 1st function argument register.  So we have arguments 1 and 2 taken care of.  What about the value of `bar`?  The instruction `movl   $15, %edx` loads the decimal value 15 into `%edx`.  Remember that `%edx` is the lower 32 bits of `%rdx` which is the 3rd function argument register in the ABI.  No we have all three arguments to `objc_msgSend` taken care of; and we call it.

#### Step 5 - Calling `printBar` ####

Starting to get the hang of this?  Let's look at how we call a method on our `Foo` instance.

``` c-objdump
0x100000e1f:  movq   %rax, %rbx
0x100000e22:  movq   823(%rip), %rsi           ; "printBar"
0x100000e29:  movq   %rbx, %rdi
0x100000e2c:  callq  *%r15
```

The instruction pair `0x100000e1f:  movq   %rax, %rbx` and `0x100000e29:  movq   %rbx, %rdi` move the `Foo` instance returned by the `initWithBar:` method to `%rdi`.  So the instance, then, is our first function argument.  Then we load a reference to `printBar` into `%rsi` as our second function argument.  Finally, we call `objc_msgSend` again.  (It's stub location was already loaded in `%r15`.)

#### Step 6 - Cleaning up our `Foo` ####

``` c-objdump
0x100000e2f:  movq   %rbx, %rdi
0x100000e32:  callq  0x100000e5e               ; symbol stub for: objc_release
```

Since we created an instance of `Foo`, we have to release it.  Recall that ARC inserts retains and releases for us as needed.  Here's an example of that.  From Step 5, recall that `%rbx` has a reference to our instance.  The instruction `movq   %rbx, %rdi` sets it up as a first function argument.  Next we call `objc_release`

#### Step 7 - Cleaning up from our function ####

``` c-objdump
0x100000e37:  movq   %r14, %rdi
0x100000e3a:  callq  0x100000e4c               ; symbol stub for: objc_autoreleasePoolPop
0x100000e3f:  xorl   %eax, %eax
0x100000e41:  addq   $8, %rsp
0x100000e45:  popq   %rbx
0x100000e46:  popq   %r14
0x100000e48:  popq   %r15
0x100000e4a:  popq   %rbp
0x100000e4b:  ret
```
The rest of our function clean-up is that same as in the prior installment of our tutorial series, popping the autorelease pool, restoring the stack and certain preserved registers.

### Conclusion ###

In this tutorial, we dived into Objective-C objects and learned about method dispatch optimizations in the Objective-C runtime while getting still more practice in interpreting x86_64 assembly language on the Mac.

Question? Comments?  Tweet Alan `@NSBum`.






