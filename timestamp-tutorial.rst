Postgres Timestamps
===================

:tags: Python, Postgres
:category: Database
:author: Trevor
:date: 2015-01-01

At my company, I maintain a large distributed. data collection platform
. Pretty much every record we collect needs to be stamped with a
`created`:code: field. But because the incoming data comes from sources on
various devices in multiple countries and timezones, making sure that
the timestamps are precise and meaningful can be a challenge.
`Postgres <http://www.postgresql.org/>`__ can do this very elegantly,
but can also trip you up in subtle ways.

Postgres has two subtly different timestamp data types: `TIMESTAMP`:code:
and `TIMESTAMP WITH TIMEZONE`:code:.

The former stores year/month/day/hour/minutes/second/milliseconds, as
you’d expect, and the later ALSO stores a timezone offset, expressed in
hours.

We can switch between the two using the `AT TIMEZONE`:code: syntax, but, and
here is the tricky bit the function goes BOTH WAYS, and you can easily
get confused if you don’t know what type you’re starting with.

Furthermore, Postgres will sometimes sneakily convert one to the other
without you asking.

.. code-block::  python

    %load_ext sql
    %config SqlMagic.feedback=False

.. code-block::  python

    %%sql
    postgresql://testuser:password@localhost/test



.. parsed-literal::

    u'Connected: testuser@test'



.. code-block::  python

    %sql SELECT NOW();



.. raw:: html

    <table class="table table-bordered table-striped is-striped is-bordered">
        <tr>
            <th>now</th>
        </tr>
        <tr>
            <td>2014-08-18 22:33:58.998549-04:00</td>
        </tr>
    </table>



`now()`:code: returns a `TIMESTAMP WITH TIME ZONE`:code:. It shows the current
**local** time, and the offset between that time and UTC
(http://time.is/UTC)

But if we put the output from `now()`:code: into a field that has type
`TIMESTAMP`:code: we will get a silent conversion:

.. code-block::  python

    %sql SELECT NOW():: timestamp



.. raw:: html

    <table class="table table-bordered table-striped is-striped is-bordered">
        <tr>
            <th>now</th>
        </tr>
        <tr>
            <td>2014-08-18 22:33:58.998549</td>
        </tr>
    </table>



Which is **not** the current UTC time. We have stripped the timezone
offset right of it. However, if we **explicitly** do the conversion, we
get:

.. code-block::  python

    %sql SELECT NOW() AT TIME ZONE 'UTC';



.. raw:: html

    <table class="table table-bordered table-striped is-striped is-bordered">
        <tr>
            <th>timezone</th>
        </tr>
        <tr>
            <td>2014-08-19 02:33:58.998549</td>
        </tr>
    </table>



Which *is* the current UTC time: (http://time.is/UTC)

It's worth reviewing the `Postgresql documentation on this
construct <http://www.postgresql.org/docs/9.1/static/functions-datetime.html#FUNCTIONS-DATETIME-ZONECONVERT-TABLE>`__
at this point.

.. raw:: html

   <table class="table table-bordered">
   <tr><th>

Expression

.. raw:: html

   </th><th>

Return Type

.. raw:: html

   </th><th>

Description

.. raw:: html

   </th></tr>
   <tr><td>

timestamp without time zone AT TIME ZONE zone

.. raw:: html

   </td><td>

timestamp with time zone

.. raw:: html

   </td><td>

Treat given time stamp without time zone as located in the specified
time zone

.. raw:: html

   </td></tr>
   <tr><td>

timestamp with time zone AT TIME ZONE zone

.. raw:: html

   </td><td>

timestamp without time zone

.. raw:: html

   </td><td>

Convert given time stamp with time zone to the new time zone, with no
time zone designation

.. raw:: html

   </td></tr><table class="table table-bordered table-striped is-striped is-bordered">

The danger here is that the `AT TIMEZONE`:code: construct goes **both
ways**. If you don't know what type you're feeding in, you won't know
what type you're getting out. I've been bitten by this in the past;
ending up with a timestamp that is wrong by several hours because I
wasn't clear about my inputs.

Specifically, consider a table that looks like this:

.. code-block::  python

    %%sql
    DROP TABLE IF EXISTS test;
    CREATE TABLE test(name TEXT, created TIMESTAMP DEFAULT NOW());






Which I then populate:

.. code-block::  python

    %%sql
    INSERT INTO test (name) VALUES ('zaphod beeblebrox');
    INSERT INTO test(name,created) VALUES('ford prefect',now() at time zone 'utc');
    SELECT * FROM test;



.. raw:: html

    <table class="table table-bordered table-striped is-striped is-bordered">
        <tr>
            <th>name</th>
            <th>created</th>
        </tr>
        <tr>
            <td>zaphod beeblebrox</td>
            <td>2014-08-18 22:34:03.620583</td>
        </tr>
        <tr>
            <td>ford prefect</td>
            <td>2014-08-19 02:34:03.621957</td>
        </tr>
    </table>



Note that the second record contains the current UTC time, but the first
contains the current time **local to the database server**. This *seems*
a good idea, and tends to work fine in local testing. But when you try
to maintain a system where the database may be in one province, the data
*collected* in another, and then *reviewed* in a third, you start to
understand why this is too simplistic.

The fact that it's 10:12 now in Toronto isn't very helpful for a record
that's getting created for a user in Halifax and is monitored from
Vancouver.

So it's probably best to save timestamps WITH their timezone so as to
avoid any ambiguity. This is the recommendation given
`here <http://justatheory.com/computers/databases/postgresql/use-timestamptz.html>`__.

In our above example, the simplest approach is to change the table
definition:

.. code-block::  python

    %%sql
    DROP TABLE IF EXISTS test;
    CREATE TABLE test(name TEXT, created TIMESTAMP WITH TIME ZONE DEFAULT (NOW() ));






.. code-block::  python

    %%sql
    INSERT INTO test (name) VALUES ('zaphod beeblebrox');
    INSERT INTO test(name,created) VALUES('ford prefect',now() );
    SELECT * FROM test;



.. raw:: html

    <table class="table table-bordered table-striped is-striped is-bordered">
        <tr>
            <th>name</th>
            <th>created</th>
        </tr>
        <tr>
            <td>zaphod beeblebrox</td>
            <td>2014-08-18 22:35:15.988764-04:00</td>
        </tr>
        <tr>
            <td>ford prefect</td>
            <td>2014-08-18 22:35:15.989726-04:00</td>
        </tr>
    </table>



So now the dates are globally meaningful. But I *still* have to be
careful, because if I use the wrong date format to populate this table,
it'll still get messed up.

.. code-block::  python

    %sql INSERT INTO test(name,created) VALUES ('arthur dent',now() at time zone 'utc')
    %sql SELECT * FROM test;



.. raw:: html

    <table class="table table-bordered table-striped is-striped is-bordered">
        <tr>
            <th>name</th>
            <th>created</th>
        </tr>
        <tr>
            <td>zaphod beeblebrox</td>
            <td>2014-08-18 22:35:15.988764-04:00</td>
        </tr>
        <tr>
            <td>ford prefect</td>
            <td>2014-08-18 22:35:15.989726-04:00</td>
        </tr>
        <tr>
            <td>arthur dent</td>
            <td>2014-08-19 02:35:15.990308-04:00</td>
        </tr>
    </table>



Note how **arthur dent** has completely the wrong created time.

Now, if I want to *report* on this data, I'm going to now have to
specify *which* timezone I want the dates formatted too:

.. code-block::  python

    %sql delete from test WHERE name='arthur dent';






.. code-block::  python

    %sql select name, created FROM test;



.. raw:: html

    <table class="table table-bordered table-striped is-striped is-bordered">
        <tr>
            <th>name</th>
            <th>created</th>
        </tr>
        <tr>
            <td>zaphod beeblebrox</td>
            <td>2014-08-18 22:35:15.988764-04:00</td>
        </tr>
        <tr>
            <td>ford prefect</td>
            <td>2014-08-18 22:35:15.989726-04:00</td>
        </tr>
    </table>



gives me timestamps formatted in the timezone of the database server,
which isn't necessarily particularly helpful, which *may* be helpful,
but will be less so if the actual *users* of the data are in a different
time zone.

.. code-block::  python

    %sql  SELECT name, created at time zone 'utc' FROM test;



.. raw:: html

    <table class="table table-bordered table-striped is-striped is-bordered">
        <tr>
            <th>name</th>
            <th>timezone</th>
        </tr>
        <tr>
            <td>zaphod beeblebrox</td>
            <td>2014-08-19 02:35:15.988764</td>
        </tr>
        <tr>
            <td>ford prefect</td>
            <td>2014-08-19 02:35:15.989726</td>
        </tr>
    </table>



gives me the time formatted in the UTC timezone, and

.. code-block::  python

    %sql select CREATED at time zone 'CST' FROM test;



.. raw:: html

    <table class="table table-bordered table-striped is-striped is-bordered">
        <tr>
            <th>timezone</th>
        </tr>
        <tr>
            <td>2014-08-18 20:35:15.988764</td>
        </tr>
        <tr>
            <td>2014-08-18 20:35:15.989726</td>
        </tr>
    </table>



gives me the time formatted for central standard time.

external data
~~~~~~~~~~~~~

Now so far we've been letting the database create the timestamps, but
sometimes we want to save data provided to us from an external source.
In this case it's very important the we know what timezone the incoming
data comes from. So our middleware should *require* that all dates
include a timestamp. Fortunately, if we're writing javascript
applications, we get this automatically:

.. code-block::  python

    %%html
    <div id="js-output"></div>


.. raw:: html

    <div id="js-output"></div>


.. code-block::  python

    %%javascript
    var d = JSON.stringify(new Date())


.. parsed-literal::

    "2014-08-19T02:41:12.872Z"


.. code-block::  python

    import psycopg2,pandas
    def execute(sql,params={}):
        with psycopg2.connect(database='test') as connection:
            with connection.cursor() as cursor:
                cursor.execute(sql,params)

So let's imagine that we got this string submitted to us by a client,
and we're going to store it in the database via some Python code.

.. code-block::  python

    sql="INSERT INTO test (name, created) VALUES ( 'externally created date', %(date)s)"
    params=dict(date="2014-08-19T02:35:24.321Z")
    execute(sql,params)
.. code-block::  python

    %sql SELECT * FROM test



.. raw:: html

    <table class="table table-bordered table-striped is-striped is-bordered">
        <tr>
            <th>name</th>
            <th>created</th>
        </tr>
        <tr>
            <td>zaphod beeblebrox</td>
            <td>2014-08-18 22:35:15.988764-04:00</td>
        </tr>
        <tr>
            <td>ford prefect</td>
            <td>2014-08-18 22:35:15.989726-04:00</td>
        </tr>
        <tr>
            <td>externally created date</td>
            <td>2014-08-18 22:35:24.321000-04:00</td>
        </tr>
    </table>



And now we're getting to the point where all our timestamp data is both
stored and displayed unambiguously.
