---
layout: post
title: "Reversing C++ - Part 2 Copy Constructors, Destructors and Assignment Operators"
date: 2014-11-08 22:33:55 +0000
comments: true
---

Welcome to part 2 of this series on basic C++ reversing with IDA.

In [part 1]({% post_url 2014-10-10-reversing-c-plus-plus-part-1-basic-construction %}) we looked at basic object construction. In this post we will take a brief look at copy constructors, destructors and assignment operators, before moving onto inheritance and virtual methods in part 3.

Lets start by looking at the previous example where we saw no copy constructor, destructor or assignment operator:

{% img /images/reversing/part-1-5.png %}

It's fairly obvious why we don't see a copy constructor or assignment operator as we haven't defined them ourselves and the compiler hasn't needed to define default versions for us as we've not carried out any copying or assignment. What should happen though, is implicit destruction of our object as it goes out of scope. So why don't we see any object destruction? Why hasn't the compiler generated a default constructor? The reason is that all of the member variables are primative types. As primitage types do not require a destructor, neither does out class. If, however, we were to add a new member variable that did require destruction then a default destructor would be generated. 

Let's prompt the compiler to generate a default copy constructor, destructor and assignment operator with the following code:

``` cpp
#include <string>

class TestClass
{
public:
    TestClass():
        mStringA(),
        mValueA(0),
        mValueB(0)
    {
    }

private:
    std::string mStringA;
    int mValueA;
    int mValueB;
};

int main()
{
    TestClass* testClassA = new TestClass;
    TestClass testClassB = *testClassA;
    *testClassA = testClassB;

    delete testClassA;

    return 0;
}
```

If we open the resulting stripped binary (see [previous post]({% post_url 2014-10-10-reversing-c-plus-plus-part-1-basic-construction %})) the decompiler produces the following:

{% img /images/reversing/part-2-1.png %}

Here we see the main function (`sub_80486FC()`) followed by the constructor (`sub_80487A6()`), copy constructor (`sub_80497CE()`), assignment operator (`sub_8048814()`) and destructor (`sub_8048800()`). Note that we have one object created on the heap with the `new()` operator and that the compiler adds a check on line 14 of `main()` to ensure that the object is valid before calling the destructor and delete operators.

Following the addition of further type information and some renaming as described in [part 1]({% post_url 2014-10-10-reversing-c-plus-plus-part-1-basic-construction %}) we can come up with something more readable (we don't define the string type, but the offsets of the remaining variables tell us that it's the size of the dword, so we set it as such in the structure definition). In addition to the steps described in [part 1]({% post_url 2014-10-10-reversing-c-plus-plus-part-1-basic-construction %}) we can remove the temporary valiables from the decompiler output by right clicking the variable and selecting `Map to another variable =`, resulting the in the following (to rename the destructor using the `~` character you may also need to upaate the `NameChars` variable in the `ida.cfg` configuration file):

{% img /images/reversing/part-2-2.png %}

Adding custom copy constructors, destructors and assignment operators has no additional affect on the compilation or output from IDA other than the changes to the respective implementations. 

In the next installment we will look at inheritance and virtual functions.
