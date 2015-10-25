IPython with Python 3
=====================

:tags: Python, IPython
:category: Software
:slug: ipython-with-python-3
:author: Trevor
:date: 2015-01-01


This took me longer than I was expecting.

In general when working with IPython I use :code:`pip` rather than :code:`apt-get`, as
pip tends to have more up-to-date packages.

In the end I found the simplest thing to do was to set up IPython in an isolated
virtualenv environment.  The main trick is to let virtualenv know what version of
Python you want it to use by default.

.. code-block:: sh

    $ virtualenv --python=python3.4 python_3_demo
    $ cd python_3_demo/
    $ source ./bin/activate
    $ pip install ipython
    $ ipython


.. code-block:: python

    Python 3.4.0 (default, Apr 11 2014, 13:05:11)
    ...

    In [1]: import sys

    In [2]: print(sys.version)
    3.4.0 (default, Apr 11 2014, 13:05:11)
    [GCC 4.8.2]


And voila, I have Python 3 in the best Python interpreter ever built, I'm ready to
start wrapping my head around byte arrays and UTF-8 encodings.
