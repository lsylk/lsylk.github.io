## January 30, 2018
Hi!

I was looking at the internet today, and found some pretty amazing things, got excited and wasn't too sure how to contain them all, and was honestly
swimming in a bit too much information.

SO, naturally, I kept reading around the internet, and started going through Julia Evans' posts at
[jvns.ca](jvns.ca). I had met her briefly at polyconf in 2016, the very first conference that I attended that
was a proper conference (the previous one being chaos communication camp which is pretty neat).
Anyway, Julia left a strong impression on me. Her talk was exciting, and incredibly informative. And
here I was, two years later, looking at her blog and thinking "I wish I could keep so much stuff in
my head".

I finally came to her blog post about writing, something that she started every day at the recurse
center in new york. I also found the blog of [Sumana](https://www.harihareswara.net/ces.shtml), who
also attended recurse and was writing a lot. What I realized is that all of this amazing writing was
a way of thinking too, so I am giving it a shot.

As a way to start I think I will try just writing what I learned about recently that I am really
excited about.

### How about Valgrind!
This seems like a good place to start:

Valgrind is super exciting. It's a way of understanding what is happening with your memory in your
application. For example, if you are writing some c code that doesn't properly free memory, it will
tell you!

Here is an example c program that isn't doing quite what it should:

leaks.c
```c
#include <stdlib.h>

int main() {
        int *ptr = malloc(sizeof(int));
        if (!ptr)
              return 1;
        return 0;
}
```

If you aren't familiar with c, we can compile our file using a compiler like gcc via `gcc -g
leak.c`. The `-g` flag will tell the compiler to add symbolic debugger info, which helps debuggers
like valgrind do what they do! This way, a debugger can tell you which source file and line number
have a problem, rather than just that there is a problem! Also. I have no idea how to write C and
this is one of my first programs :sweat_smile:

With `gcc -g leak.c` we will get a result called `a.out`.
Neat, so we have our buggy program. It compiles ok, and it also runs. But something is wrong. Let's
see what valgrind has to say.

valgrind is sort of a virtual machine that runs your program and gives you back information that you don't
get from running the program on a regular system.


You can run valgrind like so:

`valgrind ./a.out`

And we will get this output:

```
==30352== Memcheck, a memory error detector
==30352== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==30352== Using Valgrind-3.13.0 and LibVEX; rerun with -h for copyright info
==30352== Command: ./a.out
==30352==
==30352==
==30352== HEAP SUMMARY:
==30352==     in use at exit: 4 bytes in 1 blocks
==30352==   total heap usage: 1 allocs, 0 frees, 4 bytes allocated
==30352==
==30352== LEAK SUMMARY:
==30352==    definitely lost: 4 bytes in 1 blocks
==30352==    indirectly lost: 0 bytes in 0 blocks
==30352==      possibly lost: 0 bytes in 0 blocks
==30352==    still reachable: 0 bytes in 0 blocks
==30352==         suppressed: 0 bytes in 0 blocks
==30352== Rerun with --leak-check=full to see details of leaked memory
==30352==
==30352== For counts of detected and suppressed errors, rerun with: -v
==30352== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)
```

This output doesn't seem entirely useful. It tells us which command we ran, and also that there was
1 allocation of 4 bytes that were definitely lost. We don't know where though, and what are all of
these numbers? (I asked, it is the process id, this info can be found
[here](http://valgrind.org/docs/manual/manual-core.html) at 2.3. This is a useful page)

But this message also gives a hint, at the bottom of the `LEAK SUMMARY`. We can rerun our command with `--leak-check=full` to see details

With that we get the following output:

```
==30434== Memcheck, a memory error detector
==30434== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==30434== Using Valgrind-3.13.0 and LibVEX; rerun with -h for copyright info
==30434== Command: ./a.out
==30434==
==30434==
==30434== HEAP SUMMARY:
==30434==     in use at exit: 4 bytes in 1 blocks
==30434==   total heap usage: 1 allocs, 0 frees, 4 bytes allocated
==30434==
==30434== 4 bytes in 1 blocks are definitely lost in loss record 1 of 1
==30434==    at 0x4C2FB0F: malloc (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==30434==    by 0x10865B: main (leaks.c:4)
==30434==
==30434== LEAK SUMMARY:
==30434==    definitely lost: 4 bytes in 1 blocks
==30434==    indirectly lost: 0 bytes in 0 blocks
==30434==      possibly lost: 0 bytes in 0 blocks
==30434==    still reachable: 0 bytes in 0 blocks
==30434==         suppressed: 0 bytes in 0 blocks
==30434==
==30434== For counts of detected and suppressed errors, rerun with: -v
==30434== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)
```

Now we have some more information. The heap summary tells us that:

```
==30434== 4 bytes in 1 blocks are definitely lost in loss record 1 of 1
==30434==    at 0x4C2FB0F: malloc (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==30434==    by 0x10865B: main (leaks.c:4)
```

Awesome. We not only know the file where the memory was allocated, we know the line. We even know the function (malloc allocated the memory, and it was done in main). AMAZING!

We now know how to fix our program. We need to free the memory after we used it, and before we
terminate our function. Probably close to where we use it. Here is an example.

leaks-fixed.c
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

Then we can recompile it with `gcc -g leaks-fixed.c` and run it with `valgrind --leak-check=full
a.out` to see the output:

```
==30525== Memcheck, a memory error detector
==30525== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==30525== Using Valgrind-3.13.0 and LibVEX; rerun with -h for copyright info
==30525== Command: ./a.out
==30525==
==30525==
==30525== HEAP SUMMARY:
==30525==     in use at exit: 0 bytes in 0 blocks
==30525==   total heap usage: 1 allocs, 1 frees, 4 bytes allocated
==30525==
==30525== All heap blocks were freed -- no leaks are possible
==30525==
==30525== For counts of detected and suppressed errors, rerun with: -v
==30525== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)
```

We can see in the HEAP summary that we have one allocation, and one freed instance of memory, with 4
bytes allocated in total. Perfect! Everything was freed, and there were no leaks. We fixed the
program!

Since C is a new language for me, you might have noticed that this program is a bit simplistic. For
example, does it matter that the memory wasn't freed? The program terminated immediately. And that
is true! This is a formality, but a good one to follow from what I understand. And lets see about a more complex example
debugging memory issues in the future as I get more familiar with both C and valgrind :).

I didn't realize at the time, but you can use valgrind when developing firefox, and
that's how we get information about memory leaks! This is so neat, because now I can run my tests
with valgrind directly! [Information on that can be found here](https://developer.mozilla.org/en-US/docs/Mozilla/Testing/Valgrind)


