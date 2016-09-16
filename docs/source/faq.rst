Frequently Asked Questions
==========================

How do I use external modules?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Use ``client.upload_file``. For more detail, see the `API docs`_ and a 
StackOverflow question
`"Can I use functions imported from .py files in Dask/Distributed?"`__
This function supports both standalone file and setuptools's ``.egg`` files
for larger modules.

__ http://stackoverflow.com/questions/39295200/can-i-use-functions-imported-from-py-files-in-dask-distributed
.. _API docs: http://distributed.readthedocs.io/en/latest/api.html#distributed.executor.Executor.upload_file

Too many open file descriptors?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Your operating system imposes a limit to how many open files or open network
connections any user can have at once.  Depending on the scale of your
cluster the ``dask-scheduler`` may run into this limit.

By default most Linux distributions set this limit at 1024 open
files/connections and OS-X at 128 or 256.  Each worker adds a few open
connections to a running scheduler (somewhere between one and ten, depending on
how contentious things get.)

If you are on a managed cluster you can usually ask whoever manages your
cluster to increase this limit.  If you have root access and know what you are
doing you can change the limits on Linux by editing
``/etc/security/limits.conf``.  Instructions are here under the heading "User
Level FD Limits":
http://www.cyberciti.biz/faq/linux-increase-the-maximum-number-of-open-files/


Can I launch nested jobs?
~~~~~~~~~~~~~~~~~~~~~~~~~

Let's say you call some function over a range of different values. In this
function, you have a for-loop that you'd like to parallelize. Is this possible?

Yes, it's possible. One example is given below using global variables. We could
also pass in the Client or re-initialize the Client.

Here's an example using global variables:

.. code-block:: python

    from distributed import Client
    import numpy as np
    
    def g(x):
      return 2*x
    def f(x):
        # return [g(x) for _ in range(5)  # old implementation
        return [client.submit(g) for _ in range(5)]
        
    client = Client()
    y = [client.submit(f, x) for x in [1, 2, 3, 4]]
    
This is common when finding averages given different inputs.
