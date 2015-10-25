CoffeeScript
============

:tags: coffeescript, javascript
:category: Software
:author: Trevor
:date: 2015-01-01

Through a bizarre twist of history, the entire client-side web runs on a language_
that was thrown together in 10 days.

.. _language: http://en.wikipedia.org/wiki/Brendan_Eich#Netscape_and_JavaScript

Despite huge investments in their own proprietary technology by the likes of Sun
Microsystems, Adobe and Microsoft, this weird little spinoff of **self** and **scheme**
is everywhere, while client-side Java, ActiveX and Flash fade into obscurity.

Unsurprisingly for a language developed so quickly, Javascript is pretty ugly.
I'm fond of saying that it's a horrible language, with a really nice language
inside trying to get out.  It gets some things, like scoping rules, very, very
wrong.  But it got other things, like anonymous functions, exactly right, long before
they were adopted in Java, C#, or C++.  Even Python, my favourite language ever,
doesn't get them quite right.

Several people have attempted to build a nicer syntax on top of the javascript
virtual machine.  In fact, the list_ of languages that compile to JS is startlingly
big.

.. _list: https://github.com/jashkenas/coffeescript/wiki/List-of-languages-that-compile-to-JS

For the last couple of years I've been using CoffeeScript_ as my standard javascript syntax.

.. _CoffeeScript: http://coffeescript.org/

From the project page:

    "CoffeeScript is a little language that compiles into JavaScript.
    Underneath that awkward Java-esque patina, JavaScript has always
    had a gorgeous heart. CoffeeScript is an attempt to expose the
    good parts of JavaScript in a simple way."


and I think it achieves this admirably.  It doesn't solve *all* of javascript's problems -
you can still get into trouble with the Infamous Loop Problem, but it does make the language
considerably more succinct, mostly by stealing ideas from Python and Haskell.

Examples
--------

Function definitions go from

.. code-block:: javascript

    function d(x){
        return 2*x
    }

to

.. code-block:: coffeescript

    d = (x) -> 2*x


This makes for very quick object construction:


.. code-block:: coffeescript

    math =
        root:   Math.sqrt
        square: square
        cube:   (x) -> x * square x


It also borrows Python's list comprehension syntax:

.. code-block:: coffeescript

    values=(option.value for option in question.options)


The near complete absense of curly brackets saves a lot of wasted lines in
my source code, and enables me to see what's going on a lot clearer than in raw
javascript.  On the downside, I do find myself fairly regularly testing out code
snippets in the CoffeeScript_ online compiler to make sure that I've properly understood
how they will be interpreted.

Because CoffeeScript is a compiled language, to work with it effectively requires
integrating the compiler into your toolchain.  For my larger projects I've hand-written
a tool using Python's Watchdog_ package to monitor my source code directories and
output compiled javascript everytime a file changes.

.. _Watchdog: https://pypi.python.org/pypi/watchdog

As a nice little extra, my tool jams in a warning message wrapped in an :code:`alert` call
if the compliation fails, so if I introduce a syntax error in my coffeescript, as soon
as I refresh the page that is using it I'll be presented with the source of the problem.
