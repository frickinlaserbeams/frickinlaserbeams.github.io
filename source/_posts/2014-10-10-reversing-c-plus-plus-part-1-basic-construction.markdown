---
layout: post
title: "Reversing C++ - Part 1 Basic Construction"
date: 2014-10-10 23:27:26 +0100
comments: false
categories: [c++, ida, decompilation, reverse engineering]
published: true 
---

I've been developing applications in C++ for around 10 years but now find myself on the opposite end; reversing them. With some previous experience reversing C code, I can attest that C++ is an entirely different beast and brings an entirely new set of challenges.

To help build some experience of reversing C++ and to re-enforce my understanding of C++ compilation, I've been doing some exercises to see how different C++ constructs appear following disassembly and decompilation in [IDA](http://www.hex-rays.com/products/ida). This series of posts covers these exercises and hopefully provides some benefit to others who may be interested in this topic. 

Note that I'm using IDA with the [Hex-Rays Decompiler](http://www.hex-rays.com/products/decompiler/index.shtml) for these examples. If you don't already have a copy of IDA, you can get an evaluation copy of IDA from the [Hex-Rays website](http://www.hex-rays.com) (or use the free version), but you'll have to purchase the decompiler as no evaluation is offered.

So to start with, let's look at basic object construction with the following example code:

``` cpp Basic Construction Example
class TestClass
{
public:
    TestClass():
        mValueA(0)
        mValueB(0)
    {
    }

private:
    int mValueA;
    int mValueB;
};

int main()
{
    TestClass testClassA;
    return 0;
}
```

Working on a 32-bit Linux box I compile and strip the binary as shown below. Note that we strip the binary with the assumption that 'real' applications will also have their symbols stripped.

``` bash
$ g++ basicConstructionExample.cpp -o basicConstructionExample 
$ strip -s basicContructionExample
```

If we now open the resulting binary in IDA and hit F5 we see that our code look something like this:

{% img /images/reversing/part-1-4.png %}

The output from the decompiler is pretty good, but not quite what we see in the original source code. 

1. The function and variable names are incorrect.
2. The `TestClass` object has been identified as an `int` type in both the `main` function and the `TestClass` constructor. 
3. The function signature of the `TestClass` constructor has been identified as returning an `int` value rather than having a `void` return type. 
4. Declaration of the `TestClass` object is separate from construction of the object.  

Some of these differences we would expect, such as the missing function names (we stipped all these) and the fact that object declaration and initialisation are separate. However, this is not something which we see explicitly when developing the code in the first place. Luckily, IDA provides us with tools that we can use to fix up some of these differences and bring the code back to resembling what we would see when developing it.

Firstly, we can update the function and variable names to something more readable, by simply highlighting the existing name in the listing and hitting `n`.

Next, we can address the `TestClass` type. Binaries do not contain explicit data on how structures and classes are composed, so it's no surprise that IDA doesn't provide us with this information either. As such, we need to tell IDA about the composition of the class ourselves. This we can do using the 'Structures' subview which can be accessed from the 'View' menu.

{% img /images/reversing/part-1-1.png %}

From here we can create a new structure by pressing `Insert` (`i` on OSX) from within the 'Structures' view. As we already know the name or our structure we can go ahead an name it TestClass, but when reversing unknown structures it may be wise to leave renaming of the structure until it's purpose is known. 

Once the structure is created, we can add new members to the with the `d` (data), `a` (ascii) or `*` (array) keys. As shown in the constructor, code our class contains two dword members, one at `*a1` and another at `*(a1 + 4)`, so we can go ahead and add two `dd` (dword) members and rename them with `n`. Here's the finished product:

{% img /images/reversing/part-1-2.png %}

Now that we have our class structure defined, we can update the references in the decompiler output with the correct types by highlighting the variable, right clicking, selecting 'Convert to struct ' and then 'TestClass' from the list in the subsequent pop-up box:

{% img /images/reversing/part-1-3.png %}

Note that the value `v0` in `main` is not actually a pointer, so we can change this to a `TestClass` instance manually by right clicking on the variable, selecting 'Set lval type' and providing the correct type.

The final step is to change the return type of the construtor, which can be acheived by highlighting the function name and hitting `Y`.

With all these changes made, the final output view looks much more familiar: 

{% img /images/reversing/part-1-5.png %}

One thing we haven't covered here is the creation of objects on the heap. Actually, the output from the decompiler is exactly the same, bar the use of `operator new` for the allocation of the memory on the heap (plus the fact that `v0` is a pointer as identified by the use of `operator new`) and the use of `operator delete` for de-allocation.

{% img /images/reversing/part-1-6.png %}

That's pretty much it for basic object construction. In the next part we'll take a look at destructors and member functions.
