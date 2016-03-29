Installing
##########

System requirements
===================

- postgresql >= 9.4
- postgresql-server-dev-9.4
- postgresql-contrib-9.4
- postgis >= 2.2 (with shp2pgsql command line)
- ogr2ogr >= 1.9.0 (gdal-bin package)
- osm2pgsql >= 0.87
- osmconvert (osmctools package)
- python >= 2.7 <3
- python-dev >= 2.7 <3
- pip package installer >= 1.5
- virtualenv
- git


Database configuration
======================

This document assumes that you have already configured a PostgreSQL database running
with the PostGIS extension.

Configuration recommended for the **shared_buffer** parameter (postgresql.conf) is 1/4 of the RAM.
The **checkpoint_segment** can be increased to 128 to speed up data loading.


Quick summary to create a new database

.. code-block:: bash

    $ su postgres
    $ psql
    postgres=# create user prkng with password 'prkng' superuser;
    postgres=# create database prkng owner prkng encoding 'utf-8';
    postgres=# \c prkng prkng
    Password for user prkng: ****
    You are now connected to database "prkng" as user "prkng"
    prkng=> create extension postgis;

Remember to change the database name if you are running a test instance (i.e. `prkng_test`).


Development mode
==================

Checkout the code
-----------------

.. code-block:: bash

    $ git clone https://github.com/ArnaudA/prkng-process prkng
    $ cd prkng
    # create an isolated Python environment
    $ virtualenv venv
    $ source venv/bin/activate

Install the project and its dependencies in editable mode

.. code-block:: bash

    $ pip install -r requirements-dev.txt


Minimal Configuration
---------------------

A configuration file is needed to launch the application.

You can simply create the file in the root directory of the project with this name ``prkng.cfg``.
It will be used automatically.

Or you can create a file pointed with an environment variable named ``PRKNG_SETTINGS``

On Linux, you can export it via

.. code-block:: bash

    $ export PRKNG_SETTINGS=/path/to/prkng.cfg

Example of content ::

    DEBUG = True
    LOG_LEVEL = 'debug'
    PG_DATABASE = 'prkng'
    PG_USERNAME = 'user'
    PG_PASSWORD = '***'
    DOWNLOAD_DIRECTORY = '/tmp'  # used to download temporary data

    PG_TEST_HOST = 'localhost'
    PG_TEST_DATABASE = 'prkng_test'
    PG_TEST_PORT = '5432'
    PG_TEST_USERNAME = 'user'
    PG_TEST_PASSWORD = '***'



Build this documentation
------------------------

.. code-block:: bash

    $ cd doc/
    $ make html

Go to ``<file:///home/user/path/to/prkng/doc/_build/html>`_


Launch the tests
----------------

In order to launch the tests, you will have to create a test database in PostgreSQL
and fill the connection parameters in the ``prkng.cfg`` file

Then launching the test from the root directory

.. code-block:: bash

    $ py.test -v prkng


Command line ``prkng-process``
----------------------

.. code-block:: bash

    $ prkng-process update

This command will:

    - download the most recent parking information for all cities we support
    - download associated OpenStreetMap areas
    - load the previous data in the PostgreSQL database (overwrite older data)
    - load service areas (shapefiles provided in the repo for each city)

.. code-block:: bash

    $ prkng-process process

This command will process all data and generate parking slots (will erase any older data)

.. code-block:: bash

    $ prkng-process update-areas

Re-imports the service area shapefiles from the ``data`` directory, stores metadata and uploads statics to S3.

.. code-block:: bash

    $ prkng-process export

Creates a compressed and timestamped export of necessary parking data, destined for import on production and test servers.
