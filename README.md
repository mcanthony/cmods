cmods
=====

A header-only library for C that rewrites some popular
libraries' headers in a more modular form.

the story
---------

After seeing [@Snaipe](https://github.com/Snaipe)'s post about
['Modules in C99'](http://snaipe.me/c/modules-in-c99/) come up
on Hacker News, I was extremely excited. You see, this was
something I was trying to find a way to do a few months ago,
and I gave up when everything I tried kept breaking.

But Snaipe figured it out! Good on him. Now, it occurred to me
that it might be useful to be able to use popular libraries
like this.

It did take a while to do, especially for `libc` (due to
inconsistencies between implementations - I just read
original headers and manpages), but I feel like it was worth it.

installation
------------

```bash
$ sudo rm -rvf /usr/local/include/cmods		# uninstall old
$ sudo cp -rvf src /usr/local/include/cmods	# install fresh
```

usage
-----

In your project folder, create two files:

#### cmods.c

Here, include whatever cmods packages that you want:

```c
// cmods.c

#include <cmods/c.h> // yep that's all you need to do
```

When you compile your project, just compile and link
this file in like any other.

#### cmods.h

Meanwhile, include the same packages as modules here:

```c
#pragma once
// cmods.h

#include <cmods/modules/c.h>
```

Include this header in any `.c` file in your project,
or alternatively, if you have a header that you
include in all of your implementation files, include
this one at the top of that.

---

For any package modules that you use here, you don't
need to include their normal headers anywhere else.

That means no more of this:

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <string.h>
```

Instead, after you have created the two top level
files, you can just do this:

```c
#include "cmods.h"
```

compatibility
-------------

`cmods` is only needed at compile time. During the
compile process, it builds a large table of all of
the included functions and embeds it within the
resulting binary. In other words, as long as the
resulting binary is properly linked, even if
`cmods` is removed or altered, it will have no
effect on the running of the application.

OS level compatibility targets Linux and OS X. I
try my best to test on both (Ubuntu Latest Stable
and OS X Mountain Lion). It may work on other
platforms sometimes and may not at other times.

Compiler compatibility targets the latest version
of GCC and Clang available on Ubuntu Latest Stable.
Beyond that, you may have to take chances.

performance
-----------

At least on x86 and x64, function pointers offer
virtually identical performance to normally
called functions. See [this excellent post
on StackOverflow](http://stackoverflow.com/questions/2438539/does-function-pointer-make-the-program-slow)
for more details.

dangerous runtime magic
-----------------------

Storing all of the functions in a giant table
lets us do some really interesting ***DANGEROUS RUNTIME
MAGIC***. You have been warned. If you accidentally
set the planet on fire that's on you. Please do
understand that using `cmods` in this way is
generally a really bad idea.

Now then. Since `cmods` just stores a giant global
table of function pointers, if you have a function
with an identical signature, you can in theory replace
an existing function with your own. You just need
to do this before running any other functions that
may use said functions. Ideally, you should do this
first thing when you launch your program.

Let's say you wanted to replace the `malloc()` family
of functions. Here's how you could do it.

```c
... // other stuff


int main (int argc, char **argv) {
	struct cmods_libc *libc = &c;	// tables generated by cmods are
					// usually marked   const
					// to prevent this from accidentally
					// happening
	// patch the table
	libc->lib.malloc =	myMalloc;
	libc->lib.calloc =	myCalloc;
	libc->lib.realloc =	myRealloc;
	libc->lib.free =	myFree;

	// cleanup the pointer to prevent accidents
	libc = NULL;

	...	// do whatever else

	return 0;
}
```

license
-------

As with most of my open source work, `cmods` is
licensed under the [MIT License](./LICENSE).

Do whatever the hell you want to with it, but
I'm not responsible.

And yes, I can include the headers of projects
that are GPL or licensed with any other viral
license. Their license will apply to you if
you use their libraries or whatnot, but as been
ruled before, APIs themselves are not copyrightable.

contributing
------------

`cmods` packages consist of two files:

 - a declaration file (ie: the module)
 - a table file (ie: the main package)

Both files should be named `<package_name>.h`,
however, the declaration file should go in
`src/modules/`, while the table file should go
in `src/`.

The table file should include the declaration file.
Both files should start out with:

```c
#pragma once
// libcmods - the c modules library
//
// lib<package_name>

#include "common.h"
```

Try as best you can to adhere to the style in
`c.h`. Consider it to be a general guide.

When you finish, send a pull request.

questions
---------

Have a question? Open an issue and ask away.
