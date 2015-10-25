More Ractive
============

:tags: javascript, ractive
:category: Software
:slug: more-ractive
:author: Trevor
:date: 2015-01-01



I've used Ractive_ for several projects recently.
It's been like a breath of fresh air.  I've been writing user interfaces of one
kind or another for more than a decade, and keeping *what's displayed to the user*
in sync with *what's stored in the data structures* has often been a source of
frustration.


.. _Ractive: http://www.ractivejs.org/

In the javascript world, I've mostly used backbone_  and
jQuery_ for creating interactive web pages.  While these
tools are very good at what they do, I still find myself writing a fair amount
of code to update data in the model whenever a user interacts with a control, and
to update the control displays whenever data in the model changes.

.. _backbone: http://backbonejs.org/
.. _jQuery: http://jquery.com/

Enter **Ractive**.  It's not the only library to handle 2-way data binding - Angular,
Knockout and React all play in this space, but it's my current favourite.


Anyway, here's a little demo....

Given a ractive template like this:


.. code-block:: html

   <p>Type in the boxes below.<\p>
   <input class="form-control" value="{{var1}}">
   <input class="form-control" value="{{var2}}">
   <p>Current values are var1:<code>{{var1}}</code>, var2:<code>{{var2}}</p>
   <button class="btn btn-primary" on-click="changeme">Set var1</button>


and javascript like this:

.. code-block:: javascript

     var ractive = new Ractive({
        el:"#demo",
        template:template,
        data:{var1:'beeblebrox',var2:'lintilla'}
     });


We get:


.. raw:: html

    <script src='http://cdn.ractivejs.org/latest/ractive.js'></script>

    <div id="demo" class="well"></div>
    <script type="text/javascript">

         var template='<p>Type in the boxes below.<\p> \
                      <input class="form-control" value="{{var1}}">  \
                      <input class="form-control" value="{{var2}}">  \
                      <p>Current values are var1:<code>{{var1}}</code>, var2:<code>{{var2}}</p>\
                      <br><button class="btn btn-primary" on-click="changeme">Set var1</button>';

         var ractive = new Ractive({
            el:"#demo",
            template:template,
            data:{var1:'beeblebrox',var2:'lintilla'}
         });

         ractive.on('changeme',function(){
            ractive.set('var1','zarniwoop');
         });
    </script>


Neat, huh?  I haven't had to write any code to manually react to keyup, or change events
in the input controls - Ractive simply takes care of the fact that I refer to :code:`var1`
in both the output paragraph and the control value, and binds the two elements together,
refreshing them whenever needed.

The code for responding to the button click is simply:

.. code-block:: javascript

    ractive.on('changeme',function(){
        ractive.set('var1','zarniwoop');
    });


By setting data in the underlying model, the user interface automatically updates,
again without any manual intervention.


UI development might be fun again....
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

I have the same feeling on discovering Ractive that I had when I was first shown
jQuery.  All of a sudden, a bunch of boring, fiddly manual tasks are taken care
of in an intuitive way.  And unlike other frameworks, *all* ractive does is data-binding.
It doesn't try to be a control library, an AJAX toolkit or a Model-View-Controller
framework.  For those who like all-in-one solutions, this will be a weakness, but as
someone who believes in the unix philosophy of building systems from tools that
each do one thing well, I'm very impressed.
