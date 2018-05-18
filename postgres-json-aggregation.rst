Postgres JSON Aggregation
=========================

:tags: Python, Postgres
:category: Database
:author: Trevor
:date: 2015-01-01


I've been using the new JSON functionality in Postgres a lot recently:
I'm fond of saying that Postgresql is the best NoSQL database available
today. I'm quite serious about this: having used key-value and JSON
stores such as CouchDB in the past, it's amazing to me how the Postgres
developers have managed to marry the best of traditional relational
technology with the flexibility of schema-free JSON documents.

As of version 9.3, postgres allows you to create JSON column types, and
provides a number of functions to access and iterate through the data
stored in them.

This week I discovered another hidden gem - :code:`json_agg()`. This
function lets you take the results from an aggregation operation and
convert them into a JSON block - very helpful if you're then going to
work with the returned data in a language like Python

To demonstrate this, we'll first set up some simple tables.

.. code-block:: python

    %load_ext sql
    %config SqlMagic.feedback=False

.. code-block:: python

    %%sql
    postgresql://testuser:password@localhost/test



.. parsed-literal::

    u'Connected: testuser@test'



.. code-block:: python

    %%sql
    CREATE TABLE IF NOT EXISTS person (
        name text primary key
    );

    INSERT INTO person (name) VALUES
    ('emily'),('arthur'),('nicki'),('oliver')
    ;




.. parsed-literal::

    []



We can query this in the usual way:

.. code-block:: python

    %sql SELECT * FROM person;



.. raw:: html

    <table class="table table-bordered  is-striped is-bordered">
        <tr>
            <th>name</th>
        </tr>
        <tr>
            <td>emily</td>
        </tr>
        <tr>
            <td>arthur</td>
        </tr>
        <tr>
            <td>nicki</td>
        </tr>
        <tr>
            <td>oliver</td>
        </tr>
    </table>



But we can also use :code:`json_agg()`

.. code-block:: python

    %sql SELECT json_agg(name) FROM person



.. raw:: html

    <table class="table table-bordered  is-striped is-bordered">
        <tr>
            <th>json_agg</th>
        </tr>
        <tr>
            <td>[u'emily', u'arthur', u'nicki', u'oliver']</td>
        </tr>
    </table>



Which gives us a single object to work with. So far, this isn't
particularly helpful, but it becomes very useful when we start doing
:code:`JOINS`

.. code-block:: python

    %%sql
    CREATE TABLE IF NOT EXISTS action(
        id serial primary key,
        created timestamp with time zone default now(),
        person_name text references person,
        type text not null
    );

    INSERT INTO action(person_name, type) VALUES ('emily','login');
    INSERT INTO action(person_name, type) VALUES ('emily','pageview');
    INSERT INTO action(person_name, type) VALUES ('arthur','login');
    INSERT INTO action(person_name, type) VALUES ('emily','logout');
    INSERT INTO action(person_name, type) VALUES ('nicki','password_change');
    INSERT INTO action(person_name, type) VALUES ('nicki','createpost');



.. parsed-literal::

    []



If we want to ask Postgres to give us every user and every action
they've performed, we *could* do it this way:

.. code-block:: python

    %sql SELECT person.name,  action.type , action.created FROM action JOIN person ON action.person_name=person.name



.. raw:: html

    <table class="table table-bordered  is-striped is-bordered">
        <tr>
            <th>name</th>
            <th>type</th>
            <th>created</th>
        </tr>
        <tr>
            <td>emily</td>
            <td>login</td>
            <td>2014-11-08 17:45:05.963569-05:00</td>
        </tr>
        <tr>
            <td>emily</td>
            <td>pageview</td>
            <td>2014-11-08 17:45:05.964663-05:00</td>
        </tr>
        <tr>
            <td>arthur</td>
            <td>login</td>
            <td>2014-11-08 17:45:05.965214-05:00</td>
        </tr>
        <tr>
            <td>emily</td>
            <td>logout</td>
            <td>2014-11-08 17:45:05.965741-05:00</td>
        </tr>
        <tr>
            <td>nicki</td>
            <td>password_change</td>
            <td>2014-11-08 17:45:05.966274-05:00</td>
        </tr>
        <tr>
            <td>nicki</td>
            <td>createpost</td>
            <td>2014-11-08 17:45:05.966824-05:00</td>
        </tr>
    </table>



But then iterating through this recordset is a pain - I can't easily
construct a nested for loop to iterate through each person and then
through each action.

Enter :code:`json_agg()`

.. code-block:: python

    %sql SELECT person.name,  json_agg(action) FROM action JOIN person ON action.person_name=person.name GROUP BY person.name



.. raw:: html

    <table class="table table-bordered is-striped is-bordered">
        <tr>
            <th>name</th>
            <th>json_agg</th>
        </tr>
        <tr>
            <td>arthur</td>
            <td>[{u'person_name': u'arthur', u'type': u'login', u'id': 3, u'created': u'2014-11-08 17:45:05.965214-05'}]</td>
        </tr>
        <tr>
            <td>emily</td>
            <td>[{u'person_name': u'emily', u'type': u'login', u'id': 1, u'created': u'2014-11-08 17:45:05.963569-05'}, {u'person_name': u'emily', u'type': u'pageview', u'id': 2, u'created': u'2014-11-08 17:45:05.964663-05'}, {u'person_name': u'emily', u'type': u'logout', u'id': 4, u'created': u'2014-11-08 17:45:05.965741-05'}]</td>
        </tr>
        <tr>
            <td>nicki</td>
            <td>[{u'person_name': u'nicki', u'type': u'password_change', u'id': 5, u'created': u'2014-11-08 17:45:05.966274-05'}, {u'person_name': u'nicki', u'type': u'createpost', u'id': 6, u'created': u'2014-11-08 17:45:05.966824-05'}]</td>
        </tr>
    </table>



Which becomes much more usable in Python:

.. code-block:: python

    people = %sql SELECT person.name,  json_agg(action) FROM action JOIN person ON action.person_name=person.name GROUP BY person.name
.. code-block:: python

    for name, actions in people:
        print name

.. parsed-literal::

    arthur
    emily
    nicki


.. code-block:: python

    for name, actions in people:
        print name
        for action in actions:
            print '\t',action['type']

.. parsed-literal::

    arthur
    	login
    emily
    	login
    	pageview
    	logout
    nicki
    	password_change
    	createpost


So now we've managed to easily convert relational Postgres data into a
hierarchical Python data structure. From here we can easily continue to
XML, JSON, HTML or whatever document type suits your need.
