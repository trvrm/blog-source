logrotate
=========

:tags: logrotate, linux
:category: Systems
:slug: logrotate
:author: Trevor
:date: 2015-01-01

I have a large, complex `mailing system <|filename|postfix.rst>`_ that processes
a significant amount of data every hour.  While I'm developing it, I want to know
what it's doing, and whether it's having any problems.  So I use the excellent
python logging_ library to produce comprehensive monitoring data.

.. _logging: https://docs.python.org/2/library/logging.html


The only problem is that these log files can get pretty big.  And because I don't
know ahead of time when I'm going to need to hunt through them, I tend to leave
the logging system in a fairly verbose state.

Enter logrotate_.  This is a standard service on Ubuntu that regularly rotates
your log files, throwing away old data, compressing middle-age data, and leaving
young log files fresh and accessible.  Thus you are protected from
runaway log file growth and nasty calls in the middle of the night from your
monitoring service telling you that your server just died because the hard
drives were full.

.. _logrotate: http://www.thegeekstuff.com/2010/07/logrotate-examples/

A default ubuntu installation comes with logrotate already set up for various services.
If you don't have it, install it with :code:`apt-get install logrotate`, and
then it's mostly just a question of copying a file from :code:`/etc/logrotate.d/`
and modifying it according to your needs.

:code:`vi /etc/logrotate.d/myservice`

.. code-block:: sh

  /var/log/myservice/*.log {
    rotate 7
    daily
    compress
    missingok
    notifempty
  }


And that's it!  The actually invocation of the :code:`logrotate` command will
get triggered regularly by a script in /etc/cron.daily

You can also **force** a rotation, a useful option when testing out a new configuration,
via

.. code-block:: sh

  logrotate -f /etc/logrotate.d/myservice




One quick word of warning: if you're using the python logging_ library, then
you'll want to use the :code:`WatchedFileHandler` class.  If the logfile gets
rotated out while it's in use, WatchedFileHandler will notice this, close the file
stream and open a new one.
