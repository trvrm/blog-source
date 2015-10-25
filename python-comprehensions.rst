Python Comprehensions
=====================

:tags: Python
:category: Software
:slug: python-comprehensions
:author: Trevor
:date: 2015-01-01



Python **list comprehensions** are one of the most powerful and useful
features of the language. However, I've noticed even quite experienced
Python programmers using less powerful idioms when a list comprehension
would be the perfect solution to their problem, and even though I've
been a Python developer for more than a decade, I've recently learned
some very nice aspects of this feature.

What's a List Comprehension?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Python is such a strong language in part because of its willingness to
steal ideas from other languages. Python list comprehensions are an idea
that comes from
`Haskell <http://www.haskell.org/haskellwiki/List_comprehension>`__.
Fundamentally, they are a kind of 'syntactic sugar' for construct lists
from other data sources in a tight, elegant fashion.

One of the things I like most about them is they eliminate the need to
manually create loop structures and extra variables. So consider the
following:

.. code-block:: python

    squares=list()
    for i in range(10):
        squares.append(i**i)
    squares



.. parsed-literal::

    [1, 1, 4, 27, 256, 3125, 46656, 823543, 16777216, 387420489]



With List Comprehensions we can eliminate both the :code:`for` loop and the
calls to :code:`append()`

.. code-block:: python

    [i**i for i in range(10)]



.. parsed-literal::

    [1, 1, 4, 27, 256, 3125, 46656, 823543, 16777216, 387420489]



Comprehensions work with any kind of iterable as an import source:

.. code-block:: python

    [ord(letter) for letter in "hello world"]



.. parsed-literal::

    [104, 101, 108, 108, 111, 32, 119, 111, 114, 108, 100]



Multiple generators
~~~~~~~~~~~~~~~~~~~

To make things a little more complex, we can specifiy more than one
input data source:

.. code-block:: python

    [(i,j) for i in xrange(2) for j in xrange(3)]



.. parsed-literal::

    [(0, 0), (0, 1), (0, 2), (1, 0), (1, 1), (1, 2)]



Instead of just boring numbers, we could use this to construct some
sentences.

.. code-block:: python

    [(number, animal) for number in range(3) for animal in ['cats','dogs','elephants']]



.. parsed-literal::

    [(0, 'cats'),
     (0, 'dogs'),
     (0, 'elephants'),
     (1, 'cats'),
     (1, 'dogs'),
     (1, 'elephants'),
     (2, 'cats'),
     (2, 'dogs'),
     (2, 'elephants')]



Furthermore, we have a lot of control over *how* we construct the final
output objects - we can put any valid python expression in the
left-hand-side.

.. code-block:: python

    [
        "{0} {1}".format(adjective,animal)
        for adjective in ['red','cute','hungry']
        for animal in ['cat','puppy','hippo']
    ]



.. parsed-literal::

    ['red cat',
     'red puppy',
     'red hippo',
     'cute cat',
     'cute puppy',
     'cute hippo',
     'hungry cat',
     'hungry puppy',
     'hungry hippo']



or even

.. code-block:: python

    [
        "There are {0} {1} {2}".format(number, adjective,animal)
        for number in range(2,4)
        for adjective in ['cute','hungry']
        for animal in ['puppys','bats']
    ]



.. parsed-literal::

    ['There are 2 cute puppys',
     'There are 2 cute bats',
     'There are 2 hungry puppys',
     'There are 2 hungry bats',
     'There are 3 cute puppys',
     'There are 3 cute bats',
     'There are 3 hungry puppys',
     'There are 3 hungry bats']



Dictionary Comprehensions
~~~~~~~~~~~~~~~~~~~~~~~~~

An equally powerful construct is the *dictionary comprehension*. Just
like list comprehensions, this enables you to construct python
dictionaries using a very similar syntax.

.. code-block:: python

    {
        key:value
        for key,value in [
            ('k','v'),
            ('foo','bar'),
            ('this','that')
        ]
    }



.. parsed-literal::

    {'foo': 'bar', 'k': 'v', 'this': 'that'}



Armed with these tools, we can write very concise code to transform data
from one structure to another. Recently I've found them *very* helpful
for unpacking nested data structures.

Consider a simple org-structure:

.. code-block:: python

    departments=[
        {'name':'Manufacturing', 'staff': ["Jacob","Jonah", "Chloe","Liam"]},
        {'name':'Marketing','staff':["Emily","Shawn","Alex"]},
        {'name':'HR','staff':["David","Jessica"]},
        {'name':'Accounts','staff':["Nicole"]}
    ]

Now let's extract some data from it.

.. code-block:: python

    #Department names
    [department['name'] for department in departments]



.. parsed-literal::

    ['Manufacturing', 'Marketing', 'HR', 'Accounts']



.. code-block:: python

    #Staff count
    sum([len(department['staff']) for department in departments])



.. parsed-literal::

    10



.. code-block:: python

    #All staff names
    [
        name
        for department in departments
        for name in department['staff']
    ]



.. parsed-literal::

    ['Jacob',
     'Jonah',
     'Chloe',
     'Liam',
     'Emily',
     'Shawn',
     'Alex',
     'David',
     'Jessica',
     'Nicole']



Note how in the last example the *second* data-generating clause,
:code:`department['staff']`, used a reference from the *first* one.

We can take this even further. Let's make our org-chart a little more
complicated...

.. code-block:: python

    departments=[
        {
            'name':'Manufacturing',
            'staff': [
                {'name':"Jacob",'salary':50000},
                {'name':"Chloe",'salary':60000},
                {'name':"Liam",'salary':70000},
                {'name':"Jonah",'salary':55000},
            ]
        },
        {
            'name':'Marketing',
            'staff':[
                {'name':"Emily",'salary':50000},
                {'name':"Shawn",'salary':45000},
                {'name':"Alex",'salary':40000},
            ]
        },
        {
            'name':'HR',
            'staff':[

                {'name':"David",'salary':50000},
                {'name':"Jessica",'salary':60000},
           ]
        },
        {
            'name':'Accounts',
            'staff':[
                {'name':"Nicole",'salary':40000}
            ]
        }
    ]

Calculate the total salary:

.. code-block:: python

    sum(
        person['salary']
        for department in departments
        for person in department['staff']
    )



.. parsed-literal::

    520000



Now let's calculate the wages bill by department, and put the results in
a dictionary

.. code-block:: python

    {
        department['name'] : sum(person['salary'] for person in department['staff'])
        for department in departments
    }



.. parsed-literal::

    {'Accounts': 40000, 'HR': 110000, 'Manufacturing': 235000, 'Marketing': 135000}



Conclusion
----------

I've been finding this type of approach *very* helpful when working with
document-oriented data stores. We store a lot of data in JSON documents,
either on the file system or in Postgresql. For that data to be useful,
we have to be able to quickly mine, explore, select and transform it.
Tools like `JSONSelect <http://jsonselect.org/#overview>`__ do exist,
but JSONSelect is only available in Javascript, and doesn't allow you to
do the kind of rich expression-based transforms as you roll up the data
that Python does.

I also find that it avoids many common programming pitfalls:
mis-assigned variables, off-by-one errors and so on. You'll note that in
all the examples above I *never* need to create a temporary variable or
explicitly construct a :code:`for`-loop.
