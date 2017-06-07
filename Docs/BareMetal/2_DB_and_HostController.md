# Configuration of HostController
The installation and configuration of the HostController for the BareMetal analysis is not much different from the other cases in which a virtual environment is used.
At the time of writing the only operating system supported for baremetal setup is Windows Server 2012. The reason behind this requirements is connected to the requirements of iSCSI service and VHD services, available on Windows server 2012.

## Installing the database
The current version of the infrastructure uses SQLAlchemy as ORM. Although SQLAlchemy suports a variety of DBMS, we only tested the infrastructure with Postgresql, therefore we only provide instructions to install that database.

Download and install the appropriate Postgres DB version [from here](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads#windows) (we recommend Postgres 9.4).

During the installation, stick with the defaults (e.g. use 5432 as listening port). Remember to note down the superuser password for using the databse.

Now we need to create a database named "analyzer" and an user called "host_controller_agent". Open a command prompt and navigate to postgres bin directory (usually _C:\program files\postgreSQL\9.4\bin\\_). Then execute the following commands:

```
    C:\> createdb -U postgres analyzer
    <<Type postgres password>>

    C:\> createuser -U postgres host_controller_agent
    <<Type postgres password>>

    C:\> psql -U postgres
    <<Type postgres password>>

    postgres#: alter user host_controller_agent with encrypted password '<<new_database_password>>';
    postgres#: grant all privileges on database analyzer to host_controller_agent;

    <CTRL+C> to exit
```

We have now created a database named _analyzer_ and an user named _host\_controller\_agent_ with password _<<new_database_password>>_. Obviously, the user has to specify a meaningful password different from the placeholder used in this tutorial. Also, note the trailing semicolon used in both the SQL commands.

To test if everything has worked, just try to log-in as the newly created user:
```
    C:\> psql -U host_controller_agent analyzer
    <<type the db password>>
```

If login was successful, the db has been correctly configured. You can move to the next section.

## Installing Python
This software uses python 2.7 to work. Python 3 is not supported, yet. Both x32 and x64 versions work correctly.

1. Download ([from here](https://www.python.org/ftp/python/2.7.13/python-2.7.13.amd64.msi)) and install python 2.7 into a specific location, e.g. c:\python27_64

1. Open a command prompt and let's create a virtualenv in **C:\InstallAnalyzer** for this project.
```
   C:\> c:\python27_64\scripts\pip.exe install virtualenv
   C:\> cd c:\
   C:\> c:\python27_64\scripts\virtualenv InstallAnalyzer
```

1. It is now time to install windows' building libraries. Python 2.7 uses VisualStudio 2008 building engine.
   Download and install it [from here](https://www.microsoft.com/en-us/download/details.aspx?id=44266) . After installation, make sure VS90COMNTOOLS environment variable has been set correctly.
   If this does not happen (or the user wants to use a more recent build environment), manually setting the path to a VC compiler should do the trick. Have a [look at here](http://stackoverflow.com/questions/2817869/error-unable-to-find-vcvarsall-bat) .

1. Install prebuilt binaries.
   Windows requires some special care in order to build everything up. In particular, this software depends on lxml which needs development packages for libxml2 and libnxslt which are not trivial to build on windows.
   So, we advise to use prebuilt binaries on windows. Download and install the appropriate lxml and pyxml version from [here](http://www.lfd.uci.edu/~gohlke/pythonlibs/#lxml) and [here](http://www.lfd.uci.edu/~gohlke/pythonlibs/#pyxml).
   More precisely, we need to download the following two binary packages, precompiled for python 2.7 x64:
    - http://www.lfd.uci.edu/~gohlke/pythonlibs/tuth5y6k/lxml-3.7.3-cp27-cp27m-win_amd64.whl
    - http://www.lfd.uci.edu/~gohlke/pythonlibs/tuth5y6k/PyXML-0.8.4-cp27-none-win_amd64.whl

   Once downloaded, use a prompt to perform the installation:
```
   C:\> C:\InstallAnalyzer\scripts\pip install lxml-3.7.3-cp27-cp27m-win_amd64.whl
   C:\> C:\InstallAnalyzer\scripts\pip install PyXML-0.8.4-cp27-none-win_amd64.whl
```

## Installing HostController binaries
At this stage, all the "hard" dependencies should be ok. It's time to download and install the HostController Agent.

First, let's clone the git repository of HostController Agent

```
   C:\> git clone https://albertogeniola@bitbucket.org/aaltopuppaper/hostcontroller.git
```

Now we need to build the distributable version and install it via PIP command.
```
   C:\> cd hostcontroller
   C:\> C:\InstallAnalyzer\scripts\python setup.py sdist
   C:\> cd dist
   C:\> C:\InstallAnalyzer\scripts\pip install HostController-0.1.zip
```

## Installing the iSCSI server role
As previously announced, bare metal support is obtained via iSCSI service. The default installation of a Windows Server 2012 system does not enable the iSCSI file service. 

In order to enable such role, open the _Server Manger_ from the Windows taskbar. Then, from the _Manage_ menu, click _Add Roles and Features_.
Next, follow the wizard until the _Server Roles_ tab is selected: a list of items will appear. From that list, select _File And Storage Services_ -> _File and iSCSI Services_ -> _iSCSI Target Server_.
Then click _Next>_ and finish the installation. In order to make changes permanent, the system requires a reboot. So, do it.

## Allocate the Base Image for iSCSI guests
Once the iSCSI service is installed, it is time to create a base virtual disk that will be used as "clean state".
To do so we suggest to use the cmdlets provided by powershell. So open a powershell prompt as administrator and start creating the base disk VHD.

```
New-IscsiVirtualDisk C:\InstallerAnalyzer\Disks\base_disk.vhd –Size 15GB
```

Then, create an iSCSI target.

```
New-IscsiServerTarget -TargetName base_disk -InitiatorIds "MACAddress:XX-XX-XX-XX-XX-XX"
```

Replace the _XX-XX-XX-XX-XX-XX_ with the mac address of the baremetal guest that will be used for operating system instllation and configuration.
In other words, this mac address must belong to the hardware node that will be used for preparing the Guest image in the next step. 

Finally, assign the VHD to the iSCSI target.

```
Add-IscsiVirtualDiskTargetMapping base_disk C:\InstallerAnalyzer\Disks\base_disk.vhd
```

Congratulations, you can now go ahead with the preparation of the Sandbox image.
Before continuing with the next step, note down both the IP address of this node on the LAN and the iqn address of the created iSCSI target. 
Such values will be needed in the next steps.