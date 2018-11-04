---
title: Valgrind Basics
layout: post
---

## January 31, 2018 - Understanding Memory in JavaScript

I had some nice feedback regarding yesterdays post. It wasn't clear what memory allocation was or
how it worked and I am going to explore that a bit! Also, lots of people use languages that have
garbage collection these days so I want to take a look into that topic, since how garbage collectors
actually work is new to me.

### Memory allocation and deallocation basics

Memory allocation refers to the process when you have a piece of data and you want to use it later .
For example, variable assignment frequently (but not always) results in a memory allocation.  This is a memory allocation in JavaScript:

```javascript
var myFunction = function() { return 0; };
```

This will allocate a number of bytes of memory to save this function (in this case 32 bytes!). The amount of memory allocated for
something might vary, and it depends on a number of factors. If you are interested in dynamic memory in
JS, there is some very good writing about it such as this [V8 blog post about dynamic
memory](https://moduscreate.com/blog/dynamic-memory-and-v8-with-javascript/).

So, what happens after memory is allocated? If you are writing JavaScript, you usually don't need to worry about it. JavaScript has garbage collection, which
is a process that keeps track of your memory usage and frees it regularly in relation to
whether or not that code is still in use, for example if there is still a reference to a block of
memory. Again taking JavaScript as an example:

```javascript
function myFunction() {
  var myString = "Hi"; // memory is allocated
};
// memory allocated for myString is out of scope, has no references and can be freed
```

In this example, every time we call `myFunction()`, `myString` will be allocated and deallocated
(freed)

In a language such as C, there is no garbage collector, and you are expected to free memory on your
own. You do allocation with `malloc` and deallocation with `free`. It looks something like this:

```c
#include <stdlib.h>

int main() {
        int *ptr = malloc(sizeof(int));
        if (!ptr)
              return 1;
        free(ptr);
        return 0;
}
```

For more details about how this works in JavaScript, check out the [MDN post about
Memory Management](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management)

### Understanding more about garbage collection

The garbage collector is a crucial part of how memory in JavaScript works. I will write more about
this at a future date, and link it here but for now I will summarize.

The garbage collector tracks memory allocation as it happens, and draws a tree of what is currently
in use, and what is not. If an object no longer has any links (references, contexts) to the current
context, then it can be treated as garbage. Garbage collection works a bit backwards from how you would expect, in that it tracks
what is currently "alive" and designates everything else as garbage ([ref for java, but similar idea](https://www.dynatrace.com/resources/ebooks/javabook/how-garbage-collection-works/)).

When a garbage collection cycle starts, it goes through all of the elements in the tree that are
currently "alive" and marks them to be kept. If something isn't in this tree, it is ignored, and
the memory that was used for it is marked as free. This is called a Mark and Sweep algorithm. Read
more about [garbage collection in js at
MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management)

If you are curious to see how your program is building its GC root tree, you can use firefox memory
tool to take a look! In the memory tab, take a snapshot, and change the "view" from "tree view" to
"dominators", click on a node and you will see the following!

![2018-01-31-memory-graph](./images/2018-01-31-memory.png)


### Memory Problems in JavaScript (and other garbage collected languages)

So, we are taking care of one type of memory issues through garbage collection, that is the freeing
of memory that is no longer accessible. But that doesn't mean that all potential memory issues are
solved. For example, valgrind lists out four different classes of memory leak: Definitely lost,
indirectly lost, possibly lost, and still reachable. Garbage collection can take care of "Definitely
lost" cases, but not "still reachable" or "possibly lost"

You might be familiar with a couple of situations in JavaScript that are symptoms of memory problems. You can read further about these in [Chrome's memory
documentation](https://developers.google.com/web/tools/chrome-devtools/memory-problems/).

The chrome team has also come up with a nice model for optimizing pages for performance and memory
usage, which you can find [here](https://developers.google.com/web/fundamentals/performance/rail)

