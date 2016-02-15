.. _setup:

Setup
=====

Paperless isn't a very complicated app, but there are a few components, so some
basic documentation is in order.  If you go follow along in this document and
still have trouble, please open an `issue on GitHub`_ so I can fill in the gaps.

.. _issue on GitHub: https://github.com/danielquinn/paperless/issues


.. _setup-download:

Download
--------

The source is currently only available via GitHub, so grab it from there, either
by using ``git``:

.. code:: bash

    $ git clone https://github.com/danielquinn/paperless.git
    $ cd paperless

or just download the tarball and go that route:

.. code:: bash

    $ wget https://github.com/danielquinn/paperless/archive/master.zip
    $ unzip master.zip
    $ cd paperless-master


.. _setup-installation:

Installation & Configuration
----------------------------

You can go multiple routes with setting up and running Paperless. The `Vagrant
route`_ is quick & easy, but means you're running a VM which comes with memory
consumption etc. If you are running Linux, you can also `use Docker`_.
Alternatively the standard, `bare metal`_ approach is a little more complicated.

.. _Vagrant route: setup-installation-vagrant_
.. _use Docker: setup-installation-docker_
.. _bare metal: setup-installation-standard_


.. _setup-installation-standard:

Standard (Bare Metal)
.....................

1. Install the requirements as per the :ref:`requirements <requirements>` page.
2. Change to the ``src`` directory in this repo.
3. Edit ``paperless/settings.py`` and be sure to set the values for:
    * ``CONSUMPTION_DIR``: this is where your documents will be dumped to be
      consumed by Paperless.
    * ``PASSPHRASE``: this is the passphrase Paperless uses to encrypt/decrypt
      the original document.  The default value attempts to source the
      passphrase from the environment, so if you don't set it to a static value
      here, you must set ``PAPERLESS_PASSPHRASE=some-secret-string`` on the
      command line whenever invoking the consumer or webserver.
    * ``OCR_THREADS``: this is the number of threads the OCR process will spawn
      to process document pages in parallel. The default value gets sourced from
      the environment-variable ``PAPERLESS_OCR_THREADS`` and expects it to be an
      integer. If the variable is not set, Python determines the core-count of
      your CPU and uses that value.
4. Initialise the database with ``./manage.py migrate``.
5. Create a user for your Paperless instance with
   ``./manage.py createsuperuser``. Follow the prompts to create your user.
6. Start the webserver with ``./manage.py runserver <IP>:<PORT>``.
   If no specifc IP or port are given, the default is ``127.0.0.1:8000``.
   You should now be able to visit your (empty) `Paperless webserver`_ at
   ``127.0.0.1:8000`` (or whatever you chose).  You can login with the
   user/pass you created in #5.
7. In a separate window, change to the ``src`` directory in this repo again, but
   this time, you should start the consumer script with
   ``./manage.py document_consumer``.
8. Scan something.  Put it in the ``CONSUMPTION_DIR``.
9. Wait a few minutes
10. Visit the document list on your webserver, and it should be there, indexed
    and downloadable.

.. _Paperless webserver: http://127.0.0.1:8000


.. _setup-installation-vagrant:

Vagrant Method
..............

1. Install `Vagrant`_.  How you do that is really between you and your OS.
2. Run ``vagrant up``.  An instance will start up for you.  When it's ready and
   provisioned...
3. Run ``vagrant ssh`` and once inside your new vagrant box, edit
   ``/opt/paperless/src/paperless/settings.py`` and set the values for:

    * ``CONSUMPTION_DIR``: this is where your documents will be dumped to be
      consumed by Paperless.
    * ``PASSPHRASE``: this is the passphrase Paperless uses to encrypt/decrypt
      the original document.  The default value attempts to source the
      passphrase from the environment, so if you don't set it to a static value
      here, you must set ``PAPERLESS_PASSPHRASE=some-secret-string`` on the
      command line whenever invoking the consumer or webserver.

4. Initialise the database with ``/opt/paperless/src/manage.py migrate``.
5. Still inside your vagrant box, create a user for your Paperless instance with
   ``/opt/paperless/src/manage.py createsuperuser``. Follow the prompts to
   create your user.
6. Start the webserver with ``/opt/paperless/src/manage.py runserver 0.0.0.0:8000``.
   You should now be able to visit your (empty) `Paperless server`_ at
   ``172.28.128.4:8000``.  You can login with the user/pass you created in #5.
7. In a separate window, run ``vagrant ssh`` again, but this time once inside
   your vagrant instance, you should start the consumer script with
   ``/opt/paperless/src/manage.py document_consumer``.
8. Scan something.  Put it in the ``CONSUMPTION_DIR``.
9. Wait a few minutes
10. Visit the document list on your webserver, and it should be there, indexed
    and downloadable.

.. _Vagrant: https://vagrantup.com/
.. _Paperless server: http://172.28.128.4:8000


.. _setup-installation-docker:

Docker Method
.............

1. Install `Docker`_
2. Install `docker-compose`_ [1]_
3. Modify ``docker-compose.yml`` and adapt the following environment variables:

   ``PAPERLESS_PASSPHRASE`` *(relevant for both webserver and consumer)*
     This is the passphrase Paperless uses to encrypt/decrypt the original
     document. Set it for both the webserver-container and the
     consumer-container, and make sure they are identical, otherwise the
     webserver will fail to decrypt the documents encrypted by the consumer.

   ``PAPERLESS_OCR_THREADS`` *(relevant only for consumer)*
     This is the number of threads the OCR process will spawn to process
     document pages in parallel. If the variable is not set, Python determines
     the core-count of your CPU and uses that value.

   ``PAPERLESS_OCR_LANGUAGES`` *(relevant only for consumer)*
     If you want the OCR to recognize other languages in addition to the default
     English, set this parameter to a space separated list of three-letter
     language-codes after `ISO 639-2/T`_. For a list of available languages --
     including their three letter code -- see the `Debian packagelist`_.

   ``USERMAP_UID`` and ``USERMAP_GID`` *(relevant for both webserver and consumer)*
     If you want to mount the consumption volume (directory ``/consume`` within
     the containers) to a host-directory -- which you probably want to do --
     access rights might be an issue. The default user and group ``paperless``
     in the containers have an id of 1000. The containers will enforce that the
     owning group of the consumption directory will be ``paperless`` to be able
     to delete consumed documents. If your host-system has a group with an id of
     1000 and you don't want this group to have access rights to the consumption
     directory, you can use ``USERMAP_GID`` to change the id in the container
     and thus the one of the consumption directory. Furthermore, you can change
     the id of the default user as well using ``USERMAP_UID``.

4. Run ``docker-compose up -d``. This will create and start the necessary
   containers.
5. To be able to login, you will need a super user. To create it, execute the
   following command::

     docker-compose run --rm webserver createsuperuser

   This will prompt you to set a username (default ``paperless``), an optional
   e-mail address and finally a password.
6. The default ``docker-compose.yml`` exports the webserver on your local port
   8000. If you haven't adapted this, you should now be able to visit your
   `Paperless webserver`_ at ``http://127.0.0.1:8000``. You can login with the
   user and password you just created.
7. Add files to consumption directory the way you prefer to.

   **Important!** While the consumption container will ensure at startup that
   it can delete a consumed file from a host-mounted directory, it might not be
   able to read the document in the first place if the access rights to the file
   are incorrect. Make sure that the documents you put into the consumption
   directory will either be readable by everyone (``chmod u+r yourfile.pdf``) or
   readable by the default user or group id 1000 (or the one you have set with
   ``USERMAP_UID`` or ``USERMAP_GID`` respectively).

.. _Docker: https://www.docker.com/
.. _docker-compose: https://docs.docker.com/compose/install/
.. _ISO 639-2/T: https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes
.. _Debian packagelist: https://packages.debian.org/search?suite=jessie&searchon=names&keywords=tesseract-ocr-

.. [1] You of course don't have to use docker-compose, but it simplifies
   deployment immensely. If you know your way around Docker, feel free to
   tinker around without using compose!


.. _making-things-a-little-more-permanent:

Making Things a Little more Permanent
-------------------------------------

Once you've tested things and are happy with the work flow, you can automate the
process of starting the webserver and consumer automatically.  If you're running
on a bare metal system that's using Systemd, you can use the service unit files
in the ``scripts`` directory to set this up.  If you're on another startup
system or are using a Vagrant box, then you're currently on your own. If you are
using Docker, you can set a restart-policy_ in the ``docker-compose.yml`` to
have the containers automatically start with the Docker daemon.

.. _restart-policy: https://docs.docker.com/engine/reference/commandline/run/#restart-policies-restart
