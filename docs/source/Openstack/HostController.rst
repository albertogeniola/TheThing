Host Controller Installation
============================
In this section the user will learn how to configure the HostController for the two tiers topology presented in the :ref:`introduction <twotiers_openstack>` (:ref:`network topology here <twotiers_openstack_topology>`).
In particular, he will learn how to install the central database and the HostController binaries, alongside all the necessary software dependency.

Installing the database
-----------------------
The current version of the infrastructure uses SQLAlchemy as ORM.
Although SQLAlchemy suports a variety of DBMS, we only tested the infrastructure with Postgresql, therefore we only provide instructions to install that database.

Windows
#######
Download and install the appropriate Postgres DB version `from here <https://www.enterprisedb.com/downloads/postgres-postgresql-downloads#windows>`_ (we recommend Postgres 9.4).

During the installation, stick with the defaults (e.g. use 5432 as listening port).
Remember to take note of the superuser password used during the installation process of the database (this will be needed later on).

Now we need to create a database named "analyzer" and an user called "host_controller_agent".
Open a command prompt and navigate to postgres bin directory (usually *C:\\program files\\postgreSQL\9.4\bin\\*).
Then execute the following commands:

.. code-block:: none

   C:\> createdb -U postgres analyzer
   <<Type postgres password>>

   C:\> createuser -U postgres host_controller_agent
   <<Type postgres password>>

   C:\> psql -U postgres
   <<Type postgres password>>

   postgres#: alter user host_controller_agent with encrypted password '<<new_database_password>>';
   postgres#: grant all privileges on database analyzer to host_controller_agent;

   <CTRL+C> to exit


We have now created a database named *analyzer* and an user named *host\_controller\_agent* with password *<<new_database_password>>*. 
Obviously, the user has to specify a meaningful password different from the placeholder used in this tutorial. 
Also, note the trailing semicolon used in both the SQL commands.

To test if everything has worked, just try to log-in as the newly created user:

.. code-block:: none

    C:\> psql -U host_controller_agent analyzer
    <<type the db password>>


If login was successful, the db has been correctly configured. You can move to the next section.

Linux
#####
Let's download the necessary binaries

.. code-block:: none

    $ sudo apt-get update
    $ sudo apt-get install postgresql


Time to create the database we will use for our infrastructure.

.. code-block:: none

    $ sudo -u postgres createdb analyzer


Now let's create an user named host_controller_agent

.. code-block:: none

    $ sudo -u postgres createuser host_controller_agent


It is then time to create a password for the new user and assign to it all privileges for operatring on analyzer DB.

.. code-block:: none

    $ sudo -u postgres psql postgres
    postgres=#: alter user host_controller_agent with encrypted password '<<new_database_password>>';
    postgres=#: grant all privileges on database analyzer to host_controller_agent;
    <press CTRL+D to exit>

At this point, db configuration should be over. Let's try to login and see if everything is working as expected.

.. code-block:: none

    $ psql -h 127.0.0.1 -U host_controller_agent -d analyzer
    <<type the db password>>

    <<press CTRL+D to exit>>

If login was successful, the db has been correctly configured. You can move to the next section.

Installing Python
-----------------
This software uses python 2.7 to work. Python 3 is not supported, yet. Both x32 and x64 versions work correctly.

We assume the user opted to use a 64bit OS (either Windows or Linux).

Windows
#######
#. Download (`from here <https://www.python.org/ftp/python/2.7.13/python-2.7.13.amd64.msi>`_) and install python 2.7 into a specific location, e.g. *C:\\python27_64\\*

#. Open a command prompt and let's create a virtualenv in **C:\\InstallAnalyzer** for this project.

    .. code-block:: none

       C:\> c:\python27_64\scripts\pip.exe install virtualenv
       C:\> cd c:\
       C:\> c:\python27_64\scripts\virtualenv InstallAnalyzer

#. It is now time to install windows' building libraries. Python 2.7 uses VisualStudio 2008 building engine.
   Download and install it `from here <https://www.microsoft.com/en-us/download/details.aspx?id=44266>`_ .
   After installation, make sure *VS90COMNTOOLS* environment variable has been set correctly.
   If this does not happen (or the user wants to use a more recent build environment), manually setting the path to a VC compiler should do the trick.
   `Have a look at here <http://stackoverflow.com/questions/2817869/error-unable-to-find-vcvarsall-bat>`_ .

#. Install prebuilt binaries.
   Windows requires some special care in order to build everything up.
   In particular, this software depends on lxml which needs development packages for libxml2 and libnxslt which are not trivial to build on windows.
   So, we advise to use prebuilt binaries on windows.
   Download and install the appropriate lxml and pyxml version `from here <http://www.lfd.uci.edu/~gohlke/pythonlibs/#lxml>`_ and `here <http://www.lfd.uci.edu/~gohlke/pythonlibs/#pyxml>`_ .
   More precisely, we need to download the following two binary packages, precompiled for python 2.7 x64:
    - http://www.lfd.uci.edu/~gohlke/pythonlibs/tuth5y6k/lxml-3.7.3-cp27-cp27m-win_amd64.whl
    - http://www.lfd.uci.edu/~gohlke/pythonlibs/tuth5y6k/PyXML-0.8.4-cp27-none-win_amd64.whl

   Once downloaded, use a prompt to perform the installation:

    .. code-block:: none

       C:\> C:\InstallAnalyzer\scripts\pip install lxml-3.7.3-cp27-cp27m-win_amd64.whl
       C:\> C:\InstallAnalyzer\scripts\pip install PyXML-0.8.4-cp27-none-win_amd64.whl


Linux
#####
Python installation on Linux is pretty straight forward. In fact, most Linux versions already bundle a Python interpreter.
Anyways, open a terminal and issue the following commands in order to install python2.7 alongside some software dependencies needed by the HostController Agent.

.. code-block:: none

    $ sudo apt-get install python-pip python2.7-dev python2.7 unzip python-lxml python-zsi python-cffi python-ssdeep


We now want to create a virtualenv for our HostController Agent in **/home/ubuntu/InstallAnalyzer**, so we run the following commands:

.. code-block:: none

    $ sudo pip2 install virtualenv
    $ cd /home/ubuntu
    $ virtualenv InstallAnalyzer


We also need to install lxml libraries in our virtualenv.

.. code-block:: none

    $ cd InstallAnalyzer/bin
    $ pip2 install lxml ssdeep

Installing HostController binaries
----------------------------------
At this stage, all the "hard" dependencies should be ok. It's time to download and install the HostController Agent.

Windows
#######
First, let's clone the git repository of HostController Agent

.. code-block:: none

   C:\> git clone https://albertogeniola@bitbucket.org/aaltopuppaper/hostcontroller.git


Now we need to build the distributable version and install it via PIP command.

.. code-block:: none

   C:\> cd hostcontroller
   C:\> C:\InstallAnalyzer\scripts\python setup.py sdist
   C:\> cd dist
   C:\> C:\InstallAnalyzer\scripts\pip install HostController-0.1.zip

Linux
#####
Let's download the HostController Agent binaries from the official git repository.

.. code-block:: none

    $ cd /home/ubuntu
    $ git clone https://github.com/albertogeniola/HostController1.1_python.git

Now let's build and install those binaries into our virtualenv:

.. code-block:: none

    $ cd /home/ubuntu
    $ cd HostController1.1_python
    $ /home/ubuntu/InstallAnalyzer/bin/python2.7 setup.py sdist
    $ sudo /home/ubuntu/InstallAnalyzer/bin/pip2.7 install dist/HostController-0.1.tar.gz --upgrade

