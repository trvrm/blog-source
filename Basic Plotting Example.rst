Basic IPython Plotting
======================


:date: 2014-05-20 16:00
:tags: Python, IPython
:category: Software
:author: Trevor

**Things have changed a bit since IPython 1**

Now apparently we want to manually specify the use of inline matplotlib
rather than enable globally in the server.

http://nbviewer.ipython.org/github/ipython/ipython/blob/1.x/examples/notebooks/Part%203%20-%20Plotting%20with%20Matplotlib.ipynb

.. code-block:: python

    %matplotlib inline
    
.. code-block:: python

    import matplotlib.pyplot as plt
    import numpy as np
    
.. code-block:: python

    x = np.linspace(0, 3*np.pi, 500)
    plt.plot(x, np.sin(x**2))
    plt.title('A simple chirp');

    
.. image:: Basic%20Plotting%20Example_files/Basic%20Plotting%20Example_3_0.png
  :alt: basic plotting example

