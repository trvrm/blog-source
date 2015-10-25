SQL Magic
=========

:tags: Python, IPython
:category: Software
:slug: sql-magic
:author: Trevor
:date: 2015-01-01


I'm finding the :code:`%sql` magic function extremely useful. It turns
IPython into a very nice front-end to Postgresql.

First, make sure you have the :code:`ipython-sql` extension installed:

``pip install ipython-sql``

https://pypi.python.org/pypi/ipython-sql

Then we load the extension

.. code-block:: python

    %load_ext sql

Then we set up our database connection.

.. code-block:: python

    %%sql
    postgresql://testuser:password@localhost/test



.. parsed-literal::

    u'Connected: testuser@test'



And now we can start interacting directly with the database as if we
were at the :code:`psql` command line.

.. code-block:: python

    %%sql
    CREATE TABLE people (first text, last text, drink text);
    INSERT INTO people (first,last,drink)
    VALUES
        ('zaphod','beeblebrox','pan galactic gargle blaster'),
        ('arthur','dent','tea'),
        ('ford','prefect','old janx spirit')
        ;


.. parsed-literal::

    Done.
    3 rows affected.




.. parsed-literal::

    []



.. code-block:: python

    %sql select * from people

.. parsed-literal::

    3 rows affected.




.. raw:: html

    <table class="table table-bordered table-striped">
        <tr>
            <th>first</th>
            <th>last</th>
            <th>drink</th>
        </tr>
        <tr>
            <td>zaphod</td>
            <td>beeblebrox</td>
            <td>pan galactic gargle blaster</td>
        </tr>
        <tr>
            <td>arthur</td>
            <td>dent</td>
            <td>tea</td>
        </tr>
        <tr>
            <td>ford</td>
            <td>prefect</td>
            <td>old janx spirit</td>
        </tr>
    </table>



We can access the results as a python object:

.. code-block:: python

    result = %sql select * from people
    len(result)




.. parsed-literal::

    3



And we can even get our recordset as a **pandas** dataframe

.. code-block:: python

    %config SqlMagic.autopandas=True
.. code-block:: python

    frame = %sql select * from people
    frame



.. raw:: html

    <div style="max-height:1000px;max-width:1500px;overflow:auto;">
    <table class="table table-bordered table-striped">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>first</th>
          <th>last</th>
          <th>drink</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td> zaphod</td>
          <td> beeblebrox</td>
          <td> pan galactic gargle blaster</td>
        </tr>
        <tr>
          <th>1</th>
          <td> arthur</td>
          <td>       dent</td>
          <td>                         tea</td>
        </tr>
        <tr>
          <th>2</th>
          <td>   ford</td>
          <td>    prefect</td>
          <td>             old janx spirit</td>
        </tr>
      </tbody>
    </table>
    <p>3 rows × 3 columns</p>
    </div>



.. code-block:: python

    frame['first'].str.upper()



.. parsed-literal::

    0    ZAPHOD
    1    ARTHUR
    2      FORD
    Name: first, dtype: object
