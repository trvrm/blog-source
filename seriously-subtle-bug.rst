A Seriously Subtle Bug
======================

:tags: linux, python, uwsgi, bottle
:category: Software
:author: Trevor
:slug: seriously-subtle-bug
:date: 2015-01-01

I build and maintain a number of web applications built using Python_, Bottle_, and uWSGI_.
In general, I've found this a very powerful and robust software stack.  However, this week
we encountered a very strange issue that took us many hours to fully diagnose.

.. _Python: http://python.org
.. _uWSGI: http://uwsgi-docs.readthedocs.org/en/latest/index.html
.. _Bottle: http://bottlepy.org/docs/dev/index.html


Our first indication that something was wrong was when our automated monitoring tools warned
us that one of our sites was offline.  We manage our applications through the uWSGI Emperor_
service, which makes it easy to restart errant applications.  Simply touching the config file for
the application in question causes it to be reloaded:

.. code-block:: sh

    $ touch /etc/uwsgi-emperor/vassals/myapp.ini

.. _Emperor: http://uwsgi-docs.readthedocs.org/en/latest/Emperor.html



This brought our systems back up, but obviously didn't explain the problem, and over the coming weeks
it recurred several times, usually several days apart.  So, obviously my first step was to look at
the log files.   Our first indication of trouble was a log line from our database connection layer:

.. code-block:: sh

     OperationalError: could not create socket: too many open files


Which actually led us away from the real cause of the bug to start with - at first we thought that
we were simply creating too many database connections.  But further examination reassured us that yes,
our database layer was fine, our connections were getting opened and closed correctly. Postgres has
*excellent* introspective tools, if you know how to use them; in this case the following is very
helpful:

.. code-block:: sql

    SELECT * FROM pg_stat_activity;


which revealed that we had no more database connections open than expected.  So, our next step was the
linux systems administration tool :code:`lsof`.  This tool lists information about currently open files

.. code-block:: sh

    $ sudo lsof > lsof.txt

    COMMAND     PID   TID       USER   FD      TYPE             DEVICE  SIZE/OFF       NODE NAME
    init          1             root  cwd       DIR                8,1      4096          2 /
    init          1             root  rtd       DIR                8,1      4096          2 /
    init          1             root  txt       REG                8,1    265848   14422298 /sbin/init
    ...

... followed by thousands more lines.  Armed with this information, we could figure out how many files
each process was using.

Enter Pandas
------------

While it would be quite possible to search and filter this data using traditional Unix tools such as :code:`awk`
and :code:`grep`, I'm finding that more and more I'm staying inside the python ecosystem to do systems administration
and analysis tasks.  I use the Pandas_ data analysis library heavily, and it was a perfect fit for this particular task.

.. _Pandas: http://pandas.pydata.org/

.. code-block:: sh

    $ ipython

.. code-block:: python

    import pandas
    widths=[9,6,6,11,5,10,19,10,12,200]
    frame=pandas.read_fwf('lsof.txt',widths=widths)
    frame.columns

.. parsed-literal::

    Index([u'COMMAND', u'PID', u'TID', u'USER', u'FD', u'TYPE', u'DEVICE', u'SIZE/OFF', u'NODE', u'NAME'], dtype='object')


So now we have a DataFrame (a construct very similar to an Excel worksheet) with a list of every open file on the system, along
with the process id and name of the program that is holding it open.  Our next step was to ask Pandas to tell us which processes
had the *most* files open:

.. code-block:: python

    frame.PID.value_counts().head()

.. parsed-literal::

    2445     745
    2454     745
    ...

So process **2445** has 745 open files.  OK, what is that process?

.. code-block:: python

    frame[frame.PID==2445][['USER','COMMAND']]

.. parsed-literal::

              USER    COMMAND
    3083  www-data  uwsgi-cor
    3084  www-data  uwsgi-cor
    3085  www-data  uwsgi-cor
    ...



So we've learned, then, that a uWSGI process belonging to www-data is holding open more than 700 files.  Now, under
Ubuntu, this is going to be a problem very soon, because the maximum number of files that www-data may have open
per-process is 1024.

.. code-block:: sh

    $ sudo su www-data
    $ ulimit -n


.. parsed-literal::

    1024


So, clearly one of our web application processes is opening files and not closing them again.  This is the kind of
bug that I *hate* as a programmer, because it wouldn't show up in development, when I'm frequently restarting the
application, or even in testing, but only appears under real-world load.  But at least now we have a path towards
temporary remediation.  So first we simply increased the limits in :code:`ulimit` so that the service would run longer
before this bug re-appeared.  But we still wanted to understand *why* this was happening.

Next Steps
----------

Again, we used Pandas to interrogate the output of :code:`lsof`, but this time to find out whether there was a pattern
to the filenames that were being left open

.. code-block:: python

    frame.NAME.value_counts().head()


Which revealed to us that the the vast majority of the files being left open were ones that we were delivering through
our Bottle Python application. Specifically, they were being served through the static_file_ function.

.. _static_file: http://bottlepy.org/docs/dev/tutorial.html#tutorial-static-files


We verified this by hitting the url that was serving up those static files, and watching the output of lsof.  Immediately we
saw that yes, every time we served that file, the open count for that file went up.  So, we clearly had a resource leak
on our hands.  Now, this surprised me, because usually the memory management and garbage collection
in Python is excellent, and I've left the days of manually tracking resources in C long behind me.

So, next I constructed some test cases. Firstly, I ran our software on a test virtual machine to verify that I could
recreate the bug.  Then, I wrote a very bare-bones Bottle app that simply served a static file:

.. code-block:: python

    import bottle

    application=bottle.Bottle()

    @application.get('/diagnose')
    def test():
        return bottle.static_file('cat.jpg', '.')



And I immediately saw that this *didn't* trigger any kind of file leak.  The main difference between the two was that our
production application uses Bottle's *mounting* capability to namespace URLS.  So I changed my test application as follows:


.. code-block:: python

    import bottle

    app=bottle.Bottle()

    @app.get('/')
    def test():
        return bottle.static_file('cat.jpg', '.')

    rootapp=bottle.Bottle()
    rootapp.mount("/diagnose", app)
    application=rootapp


And   :code:`lsof` indicated that we *were* leaking files.  Every time I hit `/diagnose`, the open file count for `cats.jpg`
increased by one.

So, we could simply re-write our application to not use :code:`Bottle.mount`, but that wasn't good enough for me.  I wanted
to understand *why* such a simple change would trigger a resource leak.  At this point, it turns out it's good that
I have Aspergers, and with it a tendency to hyper-focus on interesting problems, because it took a long time.  In
fact, I ended up taking the Bottle library, and manually stripping it of every line of code that wasn't related to
simply handling that single URL, in an attempt to understand exactly what the different code paths were between the
leaking program and the safe one.


In doing so, I was greatly aided by the *amazing* introspective powers of Python.  We felt sure that we were
dealing with some kind of resource leak - in Python, every file is handled by a :code:`file` object, and when that object
gets cleaned up by garbage collection, the underlying file handle is closed.  So firstly, I replaced the relevant call to
the :code:`file` constructor with my own object that derived from :code:`file`


.. code-block:: python

    class MonitoredFile(file):
        def __init__(self,name,mode):
            logging.info("Opening {0}".format(name))
            file.__init__(self,name,mode)

        def __del__(self):
            logging.info('file.__del__({0})'.format(self.name))


So this object behaves exactly like a regular file, but logs events when it is created and when it is destroyed.  And sure enough,
I saw that in the file-leaking version of my code, :code:`MonitoredFile:__del__()` was never getting called.  Now in
Python an object should get deleted when its reference count drops to zero, and indeed the Python sys library provides
the :code:`getrefcount` function (https://docs.python.org/2/library/sys.html#sys.getrefcount). By adding some logging statements
with calls to :code:`sys.getrefcount()`, I saw that in the leaking-version of my code, the refcount for our file object was
one higher than in the non-leaking code when it was returned from the main application handler function.

Why should this be?  Eventually, by stripping out all extraneous code from the Bottle library, I discovered that in the version
that was using :code:`Bottle.mount()`, the response object was passed twice through the :code:`_cast()` function.  Bottle can
handle all sorts of things as response objects - strings, dictionaries, JSON objects, lists, but if it notices that it is handling
a *file* then it treats it specially.  The smoking gun code is here:
https://github.com/bottlepy/bottle/blob/854fbd7f88aa2f809f54dd724aea7ecf918a3b6e/bottle.py#L913

.. code-block:: python

    if hasattr(out, 'read'):
        if 'wsgi.file_wrapper' in request.environ:
            return request.environ['wsgi.file_wrapper'](out)
        elif hasattr(out, 'close') or not hasattr(out, '__iter__'):
            return WSGIFileWrapper(out)

Which *looks* innocent enough, and indeed is in the first version of our code.  But in the *second* version, our file handler
gets passed through this code block twice, because it's getting handled recursively.  And, indeed, if :code:`wsgi.file_wrapper`
isn't specified, then :code:`WSGIFileWrapper` is used, and everything is fine.  But in our case, we're serving this application
via uWSGI, which *does* define :code:`wsgi.file_wrapper`.  Now, I'm still not 100% clear what this wrapping function is
*supposed* to do, but on inspecting the uWSGI source_ I see that it is set to call this C function:

.. _source: https://github.com/unbit/uwsgi/blob/ed2ca5d33325dc925f6fc5558d0b817447327049/plugins/python/wsgi_handlers.c#L463

.. code-block:: c

    PyObject *py_uwsgi_sendfile(PyObject * self, PyObject * args) {

        struct wsgi_request *wsgi_req = py_current_wsgi_req();

        if (!PyArg_ParseTuple(args, "O|i:uwsgi_sendfile", &wsgi_req->async_sendfile, &wsgi_req->sendfile_fd_chunk)) {
            return NULL;
        }


        if (PyFile_Check((PyObject *)wsgi_req->async_sendfile)) {
            Py_INCREF((PyObject *)wsgi_req->async_sendfile);
            wsgi_req->sendfile_fd = PyObject_AsFileDescriptor(wsgi_req->async_sendfile);
        }

        // PEP 333 hack
        wsgi_req->sendfile_obj = wsgi_req->async_sendfile;
        //wsgi_req->sendfile_obj = (void *) PyTuple_New(0);

        Py_INCREF((PyObject *) wsgi_req->sendfile_obj);
        return (PyObject *) wsgi_req->sendfile_obj;
    }



And we can clearly see that :code:`Py_INCREF` is getting called on the file object.  So if this function is called twice,
presumably the internal reference count is incremented twice, but only decremented once elsewhere.

And indeed, as soon as I added:

.. code-block:: python

    if 'wsgi.file_wrapper' in environ:
        del environ['wsgi.file_wrapper']


to my application code, the file leaking stopped.


Concluding Thoughts
-------------------

At the moment, I'm not exactly sure whether this is a bug or a misunderstanding.  I'm not sure what :code:`wsgi.file_wrapper` is
supposed to do - I clearly have more research to do, time permitting.  And because this bug only occurred when Bottle and uWSGI
*interacted* -  I couldn't trigger it in one or other environment on its own - it's hard to say that either project has
a bug.  But hopefully this analysis will help prevent others from going through the same headaches I just did.
