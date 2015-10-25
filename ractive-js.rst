Ractive.js
==========

:tags: javascript, ractive
:category: Software
:slug: ractive-js
:author: Trevor
:date: 2014-01-01

Ractive.js_ caught my eye this week.  I've been using Backbone_ for the last couple
of years to develop single page client-side applications, and I've liked how
it doesn't get in your way, but simply allows you to get on with your work.

When I noticed that three of the tools I was using: Coffeescript, Underscore, and
Backbone_ were all written by the same guy, I realised that Jeremy Ashkenas
is a seriously genius level developer.  I've loved working with his tools; and I
love the fact that the code that he produces is so readable.  I'm always more comfortable
working with a library when I can read through the entire source code if I run
into problems.  Check out the nicely annotated source code for Backbone, for example,
here: http://backbonejs.org/docs/backbone.html


(On that note, I love working with Bottle_ for exactly the same reason - the entire
framework is contained in a single, very readable, Python file.)


But one problem that comes up time and time again, no matter what library or framework
you're using, is the problem of binding data to controls.  Currently in my Backbone-based
code I have event handling code that reacts to user input and updates the model, and
I have more code that reacts to changes in the data model and updates the UI.  Wouldn't
it be nice if this two-way data binding could happen automatically?  Wouldn't it be
nice, for example, if you could do something like this:

Imagine having a user interface like this:

.. code-block:: html

    <label>
        <input type='checkbox' checked='{{visible}}'> visible?
    </label>

And underlying data like this:

.. code-block:: javascript

    data: {
      "visible": false
    }


And imagine if all the data binding was handled for you, so that clicking on the checkbox
will automatically change the value of the underlying javascript object, and changing
the value of the object via :code:`ractive.set('visible',true)` updated the interface.

That is exactly what Ractive does.  I haven't tried using it in production yet, but
at the moment it feels like the next logical iteration in javascript frameworks.

I was busy using Backbone heavily when Angular and Knockout came out, so I don't have
much experience with them, but asking around the shop the consensus seems to be that
Ractive looks significantly nicer to use than Knockout.

And the best feature so far is their *awesome* tutorial_.  This is, apparently, entirely
written in Ractive, and guides you step by step through all the basic concepts of
the library in an elegant, interactive fashion.  No more jumping through package installations
and dependency hell before you can try out a new framework.  *This* is the way to
introduce people to your work.  I'm very impressed.

.. _tutorial: http://learn.ractivejs.org/hello-world/1/
.. _Backbone: http://backbonejs.org/
.. _Ractive.js: http://www.ractivejs.org/
.. _Bottle: http://bottlepy.org/docs/dev/_modules/bottle.html
