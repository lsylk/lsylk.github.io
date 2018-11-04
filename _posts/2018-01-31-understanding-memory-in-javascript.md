---
layout: post
title: "Understanding Memory in JavaScript"
tags:
 - javascript
 - memory
date: 2018-1-31
---

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

Now that we have memory allocated, what happens when we don't need in any more. A program can't just
grow continuously in memory usage, as this would slow down the computer and eventually stop working!. If you are writing
JavaScript however, you usually forget about memory allocation. This is because JavaScript has garbage collection, which
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

### Tricky situations

You might be familiar with a couple of situations that are memory problems. These can be broken down
into three general categories: *Memory Leaks*, *Memory Bloat*, *frequent garbage collection*. The
chrome team has done a great job summarizing how to find and debug them in [their memory
documentation](https://developers.google.com/web/tools/chrome-devtools/memory-problems/), but to
summarize quickly:

* Memory leaks slow your page down over time, and are caused by memory being kept alive
  unnecessarily. These are not memory leaks in the conventional sense, instead they are
  what is referred to as "logical memory leaks". In other words, they are used somehow by the
  program, but that use is not important and the memory is being kept for no reason.
  For example, DOM nodes that are not displayed, event listeners for events that no
  longer happen, etc.

* Memory bloat occurs when you allocate a large amount of data that is persisted. This causes the
  page to be slow at all times

* Frequent Garbage collection's main symptom is the page seeming to pause on occasion. This occurs
  when a lot of objects are allocated, and then no longer in use.

The chrome team has also come up with a nice model for optimizing pages for performance and memory
usage, which you can find [here](https://developers.google.com/web/fundamentals/performance/rail)

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
