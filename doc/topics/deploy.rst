.. _deploy:

Deploying
=========
This section is geared towards helping you deploy this server properly for
production use.

Configuration
-------------
Remember when you generated a config file in :ref:`getting started
<getting_started>`? Well we can do the same thing but a different flag to
generate a default production config file.

.. code-block:: bash

    $ pypicloud-make-config -p prod.ini

Here are the options that you should pay extra attention to:

``zero_security_mode`` should probably be ``false`` (or blank, as false is the
default). Using explicit access control is much better.

``pypi.db.url`` should be a real database. You can use sqlite for small
deploys, but make sure you're using a file instead of ``:memory:``. Redis is
recommended because it's awesome.

``session.secure`` should be ``true`` if the server is visible to the outside
world. This goes hand-in-hand with ensuring that your server always uses HTTPS.
If you set up access control and don't use HTTPS, someone will just sniff your
credentials. And then you'll feel silly.

WSGI Server
-----------
You probably don't want to use waitress for your production server, though it
will work fine for small deploys. I recommend using `uWSGI
<http://uwsgi-docs.readthedocs.org/en/latest/>`_ with `NGINX
<http://nginx.com/>`_. It's fast and mature.

After creating your production config file, it will have a section for uwsgi.
You can run uwsgi with

.. code-block:: bash

    $ uwsgi --ini-paste-logged prod.ini

Now uwsgi is running, but it's not speaking HTTP. It speaks a special uwsgi
protocol that is used to communicate with nginx. Below is a sample nginx
configuration that will listen on port 80 and send traffic to uwsgi.

::

    upstream pypicloud {
        server 127.0.0.1:3031;
    }

    server {
        listen 80;
        server_name pypi.myserver.com;
        location / {
            try_files @pypicloud_proxy;
        }

        location @pypicloud_proxy {
            include uwsgi_params;
            uwsgi_pass pypicloud;
        }
    }

.. note::

    When using access control in production, you may need pip to pass up a
    username/password. Do that by just putting it into the url in the canonical
    way: ``pip install mypkg -i https://username:password@pypi.myserver.com/pypi/``