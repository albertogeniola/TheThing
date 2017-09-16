Host Controller Installation
============================
In this section the user will learn how to configure the HostController for the single tier topology presented in the introduction.
In particular, he will learn how to install the central database and the HostController binaries, alongside all the necessary software dependency.

Installing the database
-----------------------
The current version of the infrastructure uses SQLAlchemy as ORM. Although SQLAlchemy suports a variety of DBMS, we only tested the infrastructure with Postgresql, therefore we only provide instructions to install that database.

Windows
#######
Download and install the appropriate Postgres DB version `from here <https://www.enterprisedb.com/downloads/postgres-postgresql-downloads#windows>`_ (we recommend Postgres 9.4).

During the installation, stick with the defaults (e.g. use 5432 as listening port). Remember to note down the superuser password for using the databse.

Now we need to create a database named "analyzer" and an user called "host_controller_agent". Open a command prompt and navigate to postgres bin directory (usually *C:\\program files\\postgreSQL\\9.4\bin\\*). Then execute the following commands:

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


We have now created a database named *analyzer* and an user named *host\_controller\_agent* with password *<<new_database_password>>*. Obviously, the user has to specify a meaningful password different from the placeholder used in this tutorial. Also, note the trailing semicolon used in both the SQL commands.

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


It's now time to create the database we will use for our infrastructure.

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

Installing Virtualbox
---------------------
Current version of the infrastructure has been tested with Virtualbox 5.1. Although newer versions of Virtualbox might be available, we do recommend to stick with the version 5.1 that has been proved working at the time of writing.
If supported by the host, we also suggest to stick with the 64 bit version of Virtualbox, which might take full advantage of resources exposed by the underlying system.

Windows
#######
Download and install the x64 bit Windows binaries of Virtualbox 5.1 `from here <http://download.virtualbox.org/virtualbox/5.1.10/VirtualBox-5.1.10-112026-Win.exe>`_.
Just run the installer and follow the installation wizard.

Linux
#####
The first step consists in `downloading the appropriate <https://www.virtualbox.org/wiki/Linux_Downloads>`_ version of Virtualbox for Linux system.
Detailed instructions for installing Virtualbox on your favourite Linux distribution are `available here <https://www.virtualbox.org/wiki/Linux_Downloads>`_.
In this tutorial we will only provide instructions for installing Virtualbox 5.1.20 x64 on Linux Ubuntu 16.04 LTS.
The following code snippet describes how to add the Virtualbox official repository to our apt sources and how to download and install Virtualbox 5.1.

.. code-block:: none

    $ wget -q -O - https://www.virtualbox.org/download/oracle_vbox.asc | sudo apt-key add -
    $ echo "deb http://download.virtualbox.org/virtualbox/debian vivid contrib" | sudo tee /etc/apt/sources.list.d/oracle-vbox.list
    $ sudo apt-get update
    $ sudo apt-get install dkms virtualbox-5.1


Installing Python
-----------------
This software uses python 2.7 to work. Python 3 is not supported, yet. Both x32 and x64 versions work correctly. However, when using VirtualBox as hypervisor, the python architecture must match the virtualbox architecture. In other terms, if using a 32 bit version of Virtualbox, the user has to stick with python 2.7 32bit; on the contrary, when using Virtualbox 64bit, HostController should use python 2.7 64 bits. This is a limtation introduced by COM binding libraries currently available for Virtualbox SDK.

We assume the user opted to use a 64bit version of Virtualbox on a 64bit OS (either Windows or Linux).
Windows
#######
#. Download (`from here <https://www.python.org/ftp/python/2.7.13/python-2.7.13.amd64.msi>`_) and install python 2.7 into a specific location, e.g. c:\\python27_64

#. Open a command prompt and let's create a virtualenv in **C:\\InstallAnalyzer** for this project.

    .. code-block:: none

       C:\> c:\python27_64\scripts\pip.exe install virtualenv
       C:\> cd c:\
       C:\> c:\python27_64\scripts\virtualenv InstallAnalyzer

#. It is now time to install windows' building libraries. Python 2.7 uses VisualStudio 2008 building engine.
   Download and install it `from here <https://www.microsoft.com/en-us/download/details.aspx?id=44266>`_ .
   After installation, make sure VS90COMNTOOLS environment variable has been set correctly.
   If this does not happen (or the user wants to use a more recent build environment), manually setting the path to a VC compiler should do the trick.
   Have a `look at here <http://stackoverflow.com/questions/2817869/error-unable-to-find-vcvarsall-bat>`_.

#. Install prebuilt binaries.
   Windows requires some special care in order to build everything up. In particular, this software depends on lxml which needs development packages for libxml2 and libnxslt which are not trivial to build on windows.
   So, we advise to use prebuilt binaries on windows. Download and install the appropriate lxml and pyxml version `from here <http://www.lfd.uci.edu/~gohlke/pythonlibs/#lxml>`_ and `here <http://www.lfd.uci.edu/~gohlke/pythonlibs/#pyxml>`_.
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

Install Virtualbox SDK API
--------------------------
The HostController Agent relies on Virtualbox API bindings in order to programmatically control virtual machines.
Hence, it requires the Virtualbox SDK to be installed on the host, matching the Virtualbox version installed.
Windows
#######
At first, download the appropriate version of Virtualbox SDK in accordance with the Virtualbox version installed previously.
In our specific case, we need to download the SDK version for Virtualbox 5.1.10, `from here <http://download.virtualbox.org/virtualbox/5.1.20/VirtualBoxSDK-5.1.20-114628.zip>`_.

Once downloaded, extract the sdk content somewhere, cd into sdk/installer and run

.. code-block:: none

    C:\> C:\InstallAnalyzer\scripts\python vboxapisetup.py install

Then, copy the vboxwebservice bindinds:

.. code-block:: none

    C:\> cd ../bindings/webservice/python/lib
    C:\> copy *.py C:\InstallAnalyzer\lib\site-packages\vboxapi\

Linux
#####
First of all we need to download and unzip the correct version of Virtualbox SDK, that is 5.1.20 at the time of writing.

.. code-block:: none

    $ cd /home/ubuntu
    $ wget http://download.virtualbox.org/virtualbox/5.1.20/VirtualBoxSDK-5.1.20-114628.zip
    $ unzip VirtualBoxSDK-5.1.20-114628.zip

Then we install the SDK into our virtualenv by issuing the following commands.

.. code-block:: none

    $ cd sdk/installers
    $ export VBOX_INSTALL_PATH=/usr/lib/virtualbox
    $ sudo -E /home/ubuntu/InstallAnalyzer/bin/python2.7 vboxapisetup.py install

Installing HostController binaries
----------------------------------
At this stage, all the "hard" dependencies should be ok. It's time to download and install the HostController Agent.

Windows
#######
First, let's clone the git repository of HostController Agent

.. code-block:: none

   C:\> git clone https://github.com/albertogeniola/TheThing-HostController.git

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

Now let's build and install those binaries into our virtualenv

.. code-block:: none

    $ cd /home/ubuntu
    $ cd HostController1.1_python
    $ /home/ubuntu/InstallAnalyzer/bin/python2.7 setup.py sdist
    $ sudo /home/ubuntu/InstallAnalyzer/bin/pip2.7 install dist/HostController-0.1.tar.gz --upgrade

Creating HostOnly adapter via VirtualBox
----------------------------------------
According to the network topology presented in [Introduction](1_Introduction.md], the Host controller must provide a HostOnly Adapter interface. Such interface must be allocated manually via VBoxManage interface. In general, Virtualbox automatically creates a HostOnly adapter at installation time. However this might change in the future, so the user has not to rely on such assumption.

Windows
#######
Open a command prompt with administrator rights, navigate into the virtualbox installation directory and check if any HostOnly adapter is available for use.

.. code-block:: none

   C:\> cd "%PROGRAMFILES%\Oracle\Virtualbox"
   C:\> VBoxManage list -l hostonlyifs

If no interface is listed, let's create one, otherwise skip the next command.

.. code-block:: none

   C:\> VBoxManage hostonlyif create


Now let's verify its configuration:

.. code-block:: none

   C...>VBoxManage list -l hostonlyifs
    C:\Program Files\Oracle\VirtualBox>VBoxManage list -l hostonlyifs
    Name:            VirtualBox Host-Only Ethernet Adapter #2
    GUID:            cf51993c-1be7-43d2-8eea-707c98ca3722
    DHCP:            Disabled
    IPAddress:       192.168.208.1
    NetworkMask:     255.255.255.0
    IPV6Address:     fe80:0000:0000:0000:3814:453d:ba44:c6a2
    IPV6NetworkMaskPrefixLength: 64
    HardwareAddress: 0a:00:27:00:00:26
    MediumType:      Ethernet
    Status:          Up
    VBoxNetworkName: HostInterfaceNetworking-VirtualBox Host-Only Ethernet Adapter #2

The HostOnly nic has been given "VirtualBox Host-Only Ethernet Adapter #2" as name, but its network configuration needs to be adjusted to reflect our topology. So we run the following command:

.. code-block:: none

   C...>VBoxManage hostonlyif ipconfig "VirtualBox Host-Only Ethernet Adapter #2" --ip 192.168.56.1 --netmask 255.255.255.0

Let's double check the new configuration:

.. code-block:: none

   C...>VBoxManage list -l hostonlyifs

    Name:            VirtualBox Host-Only Ethernet Adapter #2
    GUID:            cf51993c-1be7-43d2-8eea-707c98ca3722
    DHCP:            Disabled
    IPAddress:       192.168.56.1
    NetworkMask:     255.255.255.0
    IPV6Address:     fe80:0000:0000:0000:3814:453d:ba44:c6a2
    IPV6NetworkMaskPrefixLength: 64
    HardwareAddress: 0a:00:27:00:00:26
    MediumType:      Ethernet
    Status:          Up
    VBoxNetworkName: HostInterfaceNetworking-VirtualBox Host-Only Ethernet Adapter #2

Linux
#####
Open a terminal and verify if there is any HostOnly interface available.

.. code-block:: none

   $ VBoxManage list -l hostonlyifs

If no interface is listed, let's create one, otherwise skip the next command.

.. code-block:: none

   C:\> VBoxManage hostonlyif create

Now let's verify its configuration:

.. code-block:: none

    $ VBoxManage list -l hostonlyifs
    ...

The HostOnly nic has been given "VirtualBox Host-Only Ethernet Adapter #2" as name, but its network configuration needs to be adjusted to reflect our topology. So we run the following command:

.. code-block:: none

   $ VBoxManage hostonlyif ipconfig "..." --ip 192.168.56.1 --netmask 255.255.255.0

Let's double check the new configuration:

.. code-block:: none

   $ VBoxManage list -l hostonlyifs
    ...

Quick test
----------
TODO: make sure HostController can start

