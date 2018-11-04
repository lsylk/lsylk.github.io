
#### First, about memory allocation

If you have worked primarily with languages that include garbage collection, you might not be
familiar with problems that occur with managing a program's memory usage yourself.
Memory allocation refers to the process when, as your code is being executed, it comes across an
expression that needs to be saved in memory for use later. This frequently happens when you save
something to a variable. JavaScript for example allows you to allocate memory without thinking
about it! This is a memory allocation:

```javascript
var myFunction = function() { return 0; };
```

This will allocate 2464 bytes of memory to save this function. The amount of memory allocated for
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
// memory is out of scope and can be freed
```

In this example, every time we call `myFunction()`, `myString` will be allocated and deallocated
(freed)

In a language such as C, there is no garbage collector, and you are expected to free memory on your
own.


