Asynchronous Python
===================

:tags: Python, gevent
:category: Software
:slug: asynchronous-python
:author: Trevor
:date: 2015-01-01

It's possible to get python to do node-like non-blocking requests, this could
take away one of the key reasons for using node.

The following is a full bottle-based python web application.

A client can sucessfully call /test while another client is waiting for
/slowproxy to return a result from a slow web service.

.. code-block:: python

    from gevent import monkey; monkey.patch_all()

    from bottle import route, run
    import time

    @route('/sleep/<seconds:int>')
    def sleep(seconds):
        time.sleep(seconds)
        return "Slept For {0}".format(seconds)

    @route('/test')
    def test():
         return 'test'


    @route('/slowproxy/<seconds:int>')
    def slowproxy(seconds):
        import requests
        url="https://s.nooro.com/sleeptest.php?seconds=%i" %seconds
        response=requests.get(url)
        response.raise_for_status()
        return response.text

    run(host='0.0.0.0', port=8080,server='gevent')



My first attempt used grequests, but apparently that's not even necessary.

I guess that the call to monkey.patch_all() even patches the socket code
that requests uses.  I'm very impressed.
