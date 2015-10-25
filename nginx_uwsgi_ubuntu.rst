Nginx and UWSGI on Ubuntu 14
============================

:tags: Python, nginx, uwsgi
:category: Systems
:author: Trevor
:slug: nginx-uwsgi-ubuntu-14
:date: 2015-01-01

The documentation for Nginx and UWSGI is long and complex, but with ubuntu 14
it's actually pretty straightforward to get them up and running.

I present here a a setup that uses nginx and uwsgi emperor to host
multiple python web applications simultaneously on an ubuntu 14 machine.

First, the packages

.. code-block:: sh

    $ sudo apt-get install nginx
    $ sudo apt-get install uwsgi uwsgi-emperor uwsgi-plugin-python


Our configuration files will now be under :code:`/etc/nginx` and :code:`/etc/uwsgi-emperor`

You can start, stop, and reload nginx as follows:

.. code-block:: sh

    $ sudo service nginx start
    $ sudo service nginx stop
    $ sudo service nginx reload

The last command is useful when changing configuration settings.

Now set up a site by creating a file in :code:`/etc/nginx/sites-available`

.. code-block:: sh

    #/etc/nginx/sites-available/mysite
    server{

        server_name     your_host_name;

        location /app1 {
            uwsgi_pass unix:/tmp/app1.socket;
            include uwsgi_params;
        }
        location /app2 {
            uwsgi_pass unix:/tmp/app2.socket;
            include uwsgi_params;
        }
    }


Then,

.. code-block:: sh

    $ sudo ln -s /etc/nginx/apps-available/mysite /etc/nginx/sites-enabled



.. alert-warning::
    **Warning**

    A previous version of this tutorial had the sockets placed in :code:`/run/uwsgi`.
    This was a mistake, because under Ubuntu :code:`/run` is mounted as a :code:`tmpfs`, and its content will be deleted on reboot
    Your uwsgi sub-directory will vanish and the uwsgi services will not restart.


Next, set up your 'vassals' (http://uwsgi-docs.readthedocs.org/en/latest/Emperor.html)

Create  :code:`/etc/uwsgi-emperor/vassals/app1.ini` as follows:

.. code-block:: ini

    [uwsgi]
    plugin = python
    processes = 2
    socket = /tmp/app1.socket
    chmod-socket = 666

    chdir = /srv/app1
    wsgi-file = /srv/app1/main.py

    uid = www-data
    gid = www-data


And for your second application, create  :code:`/etc/uwsgi-emperor/vassals/app2.ini` as similarly:

.. code-block:: ini

    [uwsgi]
    plugin = python
    processes = 2
    socket = /tmp/app2.socket
    chmod-socket = 666

    chdir = /srv/app1
    wsgi-file = /srv/app2/main.py

    uid = www-data
    gid = www-data



The simple act of *creating* or touching a .ini file in :code:`/etc/uwsgi-emperor/vassals` will cause
the emperor process to try to restart your application.

Of course, your applications don't exist yet, so let's create them.  The simplest wsgi
application can be only a few lines long:

Create :code:`/srv/app1/main.py`

.. code-block:: python

    def application(env, start_response):
        start_response('200 OK', [('Content-Type','text/html')])
        return ["Hello World, I am app1"]


And :code:`/srv/app2/main.py`

.. code-block:: python

    def application(env, start_response):
        start_response('200 OK', [('Content-Type','text/html')])
        return ["I, however, am app2. "]



And that's it!

Visiting http://your_host_name/app1 or http://your_host_name/app2 should return the text
you put in the python files.
