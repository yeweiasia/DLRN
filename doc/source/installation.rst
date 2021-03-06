============
Installation
============

Installing prerequisites:

.. code-block:: bash

    $ sudo yum install git createrepo python-virtualenv mock gcc redhat-rpm-config rpmdevtools httpd

Add the user you intend to run as to the mock group:

.. code-block:: bash

    $ sudo usermod -a -G mock $USER
    $ newgrp mock
    $ newgrp $USER

If you want to serve the built packages and the status reports:

.. code-block:: bash

    $ sudo systemctl start httpd

Install DLRN:

.. code-block:: bash

    $ pip install dlrn

Or, if you have virtualenv installed:

.. code-block:: bash

    $ virtualenv dlrn-venv
    $ source drln-venv/bin/activate
    $ pip install dlrn

The httpd module is not strictly required, DLRN does not use it. However, it will output
it's results in a way that is suitable for a web-server to serve. This means you can easily set up
a web-server to serve the finished ``.rpm`` and ``.log`` files.


Configuration
-------------

Configuration is done in an INI-file. An example file called ``projects.ini`` is included.
The configuration file looks like this:

.. code-block:: ini

    [DEFAULT]
    datadir=./data
    baseurl=https://trunk.rdoproject.org/
    distro=rpm-master
    smtpserver=
    maxretries=3
    target=centos
    gerrit=yes
    rsyncdest=
    rsyncport=22

* ``datadir`` is the directory where the packages and repositories will be created.

* ``baseurl`` is the URL to the data-directory, as hosted by your web-server. Unless you are
  installing DLRN for local use only, this must be a publicly accessible URL.

* ``distro`` is the branch to use for building the packages.

* ``target`` is the distribution to use for building the packages (``centos`` or ``fedora``).

* ``smtpserver`` is the address of the mail server for sending out notification emails.
  If this is empty no emails will be sent out. If you are running DLRN locally,
  then do not set an smtpserver.

* ``maxretries`` is the maximum number of retries on known errors before marking the build
  as failed. If a build fails, DLRN will check the log files for known, transient errors
  such as network issues. If the build fails for that reason more than maxretries times, it
  will be marked as failed.

* ``gerrit`` if set to anything, instructs dlrn to create a gerrit
  review when a build fails. See next section for details on how to
  configure gerrit to work.

* ``rsyncdest`` if set, specifies a destination path where the hashed repository
  directories created by DLRN will be synchronized using ``rsync``, after each commit build.
  An example would be ``root@backupserver.example.com:/backupdir``.
  Make sure the user running DLRN has access to the destination server using passswordless SSH.

* ``rsyncport`` is the SSH port to be used when synchronizing the hashed repository. If
  ``rsyncdest`` is not defined, this option will be ignored.

Configuring for gerrit
++++++++++++++++++++++

You first need ``git-review`` installed. You can use a package or install
it using pip.

Then the username for the user creating the gerrit reviews when a
build will fail needs to be configured like this::

  $ git config --global --add gitreview.username "myaccount"

and authorized to connect to gerrit without password.

Configuring your httpd
----------------------

The output generated by DLRN is a file structure suitable for serving with a web-server.
You can either add a section in the server configuration where you map a URL to the
data directories, or just make a symbolic link:

.. code-block:: bash

    $ cd /var/www/html
    $ sudo ln -s <datadir>/repos .

