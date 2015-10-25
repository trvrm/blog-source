Remote Systems Administration with Fabric
=========================================

:tags: linux, python
:category: Systems
:author: Trevor
:date: 2015-01-01

A tool I'm finding myself using more and more these days is Fabric_.

.. _Fabric: http://fabric.readthedocs.org/en/1.8/

Fabric is a python utility and library for streamlining systems administration
tasks on multiple machines.

Although I'm primarily a developer and systems architect, I find that as our
company grows I keep getting drawn into sysadmin and devops tasks.   We now
have quite a number of servers which I need to monitor and manage.

Now, as Larry Wall so profoundly said, one of the great virtues of a programmer
is *laziness*, so if there's a way I can perform the same task on multiple machines
without having to manually type out the commands dozens of times, then I'm all for it.

Fabric does precisely that.  It's written in Python, and its configuration files
are python scripts, so there's no need to learn yet another domain-specific language.
I have a file called :code:`fabfile.py` on my local machine that contains a growing
collection of little recipes, and with this I can interact with all the servers
in our infrastructure.

So for example, if I want to see at a glance which version of linux I'm running on
all my servers, I have a task set up in my fabfile like this:

.. code-block:: python

    @parallel
    def lsb_release():
        run('lsb_release -a')


When I invoke this via:

.. code-block:: sh

    $ fab lsb_release


I get a nice little print-out of the current version of all my servers.  Fabric
runs the task in parallel against every host in the :code:`env.hosts` variable,
which can be set at the command line or in the **fabfile**.

The following example allows me to see a list of every database running on every
database server.

.. code-block:: python

    DATABASE_HOST_MACHINES=['dbserver1','dbserver2',...]

    @hosts(DATABASE_HOST_MACHINES)
    def databases():
        with warn_only():
            run('psql -c "select datname from pg_database;"')


And *voila*, a call to

.. code-block:: sh

    $ fab databases

gives me a company-wide view of all our databases.

One further demonstration - this blog itself is generated using Fabric! For details, see
the fabfile my Github_ repository.

.. _Github: https://github.com/trvrm/trvrm.github.io/blob/master/fabfile.py
