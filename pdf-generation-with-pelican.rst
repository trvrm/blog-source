PDF Generation With Pelican
===========================

:tags: python, pelican
:category: Software
:author: Trevor
:date: 2015-01-01

The existing documentation is a little unclear on this, because it says
you need to add `PDF_GENERATOR=True`:code: to your `pelicanconf.py` file.

This advice is out of date: PDF generation has been moved to a plugin.

So you need to first make sure you have :code:`rst2pdf` installed:

.. code-block:: sh

    $ sudo apt-get install rst2pdf

and then add the following to :code:`pelicanconf.py`

.. code-block:: python

    PLUGIN_PATH = '../pelican-plugins'  # or wherever.
    PLUGINS = ['pdf']




However, doing this seems to screw up the pygments highlighting on my regular
html output.  This is because deep in the rst2pdf code, in a file called :code:`pygments2style.py`,
all the pygment elements have their CSS classes prepended with :code:`pygment-`.  I haven't
figured out how to generate HTML and PDF nicely at the same time.
