# TheThing: automated installer analyzer
TheThing is an automated sandbox analyzer developed precisely for analyzing PUP installers. As such, it has three main objectives:
- To automate the installers containing PUP
- To monitor and collect information regarding system changes during such installations
- To store collected data into a database for later study

To achieve such goals, TheThing is implemented via a distributed architecture, composed by:
- Crawler(s) **(optional)**: collect jobs from the Internet and save them into the database
- A central database: stores jobs and results. Also stores some state for ongoing analysis, used internally by host controllers.
- Host controller(s): handles the life-cycle of guests via specific hypervisor API and perform some post-processing on collected data.
- Guest agent(s): represent machines used as Sandbox in which analysis happens. They can be either virtual or physical.
- Network sniffer(s): implement gateways for providing controller Internet access to guests. Controlled by host controllers, they are in charge of sniffing traffic flowing among guests.

Ideally, each part of TheThing runs on a distinct node, that can be either physical or virtual. However, the user may also opt for a single-node architecture, installing all the components on the same machine. To let the system scale, user should install central database and each host controllers on distinct nodes hardware nodes. Obviously the type and number of different components running on each hardware node strongly depends on hardware characteristics of the underlying machine.

In order to support both virtual and bare metal guests, communication among nodes of the architecture is performed via standard TCP/IP socket. Therefore, any configuration of the system must comply with a specific network topology. Currently, TheThing supports the following network scheme.

![Supported Network Topology](Docs/img/topology.png)

Guests (sandboxes) are equipped with a single network interface (eth0), connected to a private network. On the same network there must be a sniffer instance, which acts as a gateway, providing basic DHCP/DNS/NAT capabilities to the guests. The sniffer is equipped with two network interfaces: eth0 is connected to external Internet, while eth1 is connected to the private network, binding a static IP address. Therefore, traffic from/to eth1 is routed to eth0, enabling guests to access the Internet via natting. 

# Important notes
Before going through installation and configuration of the infrastructure, it is really important to understand how TheThing works from an high perspective. More specifically, it is worth understanding which and how the nodes of the infrastructure communicate each others. 

## Central DB
The central database consists in the synchronization point of the entire infrastructure. It holds results, jobs and status of current analysis. 
In general, both crawlers and HostControllers need to communicate with the central database. Therefore, it is usually a good idea to condensate the DB instance, the HostController and the crawlers on a single physical server. By doing so, network communication between HostController and Database happens locally via the loopback interface. Nevertheless, such configuration enables a more secure setup of the infrastructure, because DB access can be limited exclusively to the loopback domain (127.0.0.1).

In general, the **DB must be reachable by crawlers and HostControllers**. When using multiple distributed HostController, the DB must allow incoming connection from IPs where HostControllers and crawlers are running. **We strongly recommend to not expose the DB on the Internet**, but connect HostControllers and DB via a private LAN (using a switch).

## HostController
An HostController is a service running on a node, in charge of handling the life-cycle of sandboxes. To do so, it uses specific APIs, depending on the sandbox type it has to handle. For example, a HostController might use Openstack API for orchestrating the analysis of binaries via an Openstack cloud. Similarly, it can use Virtualbox API to do the same. Nevertheless, it can handle the life-cycle of bare-metal machines.

From network perspective, each **HostController must be able to communicate with the central DB and should be reachable by the sandboxes it administrates**. Ideally, it uses a private NIC (or a local socket) to communicate with the DB while listening on a public IP for requests incoming from GuestAgents.

## Guests
A guest is a sandbox, which can either be physical or virtual, in which binaries are executed. Current version of this infrastructure only supports Windows 7 SP1 32 bit operating system. Guests' life-cycle is handled by a HostController. That means that the HostController decides when to spawn a Guest and what jobs the guest should be assigned.

Guests are configured in such a way that a specific software module, the GuestAgent, runs at startup. That software module is in charge of retrieving the binary to analyze, automate its installation and collect resource accesses performed by the binary during its analysis. All this information is stored locally and then transmitted back to the HostController. 

From the networking perspective, the GuestAgent needs to connect to a specific HostController, via a TCP connection on port 9000 (by default). This means that the **HostController must be reachable by guests**. The easiest way to ensure such a requirement is to publish the HostController over the Internet, binding a public IP. _Other approaches are possible_, but we don't discuss them here.

## Sniffer
A sniffer is a virtual or physical entity that provides basic network services to a number of guests, while implementing sniffing capabilities. Sniffers are controlled by HostController via a web-service interface. This essentially means that HostControllers should be able to communicate with sniffers via a web interface. Again, the most compatible (and also most insecure) way of providing such capability is to assign the sniffer a public IP.

## Prerequisites
Each type of node has different prerequisites.
* HostController: developed in Python, can be installed on Linux or Windows hosts (Linux 14.04 LTS and Windows Server 2012 have been tested). It requires Python 2.7 to be installed on the system. 
* Central Database: can be one of the technologies supported by SqlAlchemy. We recommend to use PostgreSql, since it is the only tested dbms. 
* Crawlers: developed in Python, can be installed on Linux or Windows hosts. We do recommend to install it alongside one of the HostController instances. 
* Guest: only Windows 7 SP1 operating systems are supported.
* Sniffer: currently is implemented as a software router on the top of a standard Linux distribution (Ubuntu Server 14.04 LTS). Requires at least two network interfaces. Minimum hardware resource needed for the infrastructure 

All the nodes of the infrastructure can either be physical or virtual, depending on the user's specific needs. 

# Installation
Being a distributed system, the deployment of TheThing is quite time consuming. Moreover, TheThing might be configured in a number of ways, depending on the specific sandbox system used (hypervisor/baremetal). In this document we provide two distinct tutorials for deploying the thing: a single tier configuration using VirtualBox as hypervisor and a two-tiers configuration using a remote Openstack cloud infrastructure. Both of them are briefly described below.

## Single-tier configuration, using VirtualBox
![Single tier tutorial](Docs/img/1_tier_virtualbox.png)

This is the easiest configuration possible, recommended for first time users. It only requires a single node to work and implements all networking communication locally. In fact, all the components of the infrastructure are deployed on a single node. The virtualization system used for this purpose is VirtualBox. 

Although the 1-tier configuration is the easiest to start from, it cannot scale. We suggest to start using this configuration and once confident, move to a more scalable configuration.

[**Installation and Deployment instructions for such topology are available HERE**](Docs/Virtualbox/1_Introduction.md).
## Two-tiers configuration, using Openstack
![Two tiers tutorial](Docs/img/2_tiers_openstack.png)

The two tiers configuration uses a node for hosting the database, the crawlers and a HostController. In this specific case, Openstack cloud is used as virtualization option. Such configuration enables better scalability, in accordance with resources available on the specific Openstack cloud. 

[**Installation and Deployment instructions for such topology are available HERE**](Docs/Openstack/1_Introduction.md).

## Baremetal, [todo]

**Installation and Deployment instructions for such topology are available HERE**.

## Mixed multitier, [todo]

**Installation and Deployment instructions for such topology are available HERE**.


## Host Controller
### Python.
This software uses python 2.7 to work. Python 3 is not supported, yet. Both x32 and x64 versions work correctly, however the user has to carefully decide when to use one or another. If you plan to use VirtualBox as hypervisor and COM as binding method, then the python version you use needs to be consistent with the virtual box architecture.
Let's assume the user opted for x64 python 2.7.
1. Install python 2.7 into a specific location, e.g. c:\python27_64

2. Create a virtualenv for this project.
   c:\python27_64\scripts\pip.exe install virtualenv
   cd c:\
   c:\python27_64\scripts\virtualenv InstallAnalyzer

3. Install building libraries (this is for windows, may vary for linux). Python 2.7 uses VisualStudio 2008 building engine.
   Download and install https://www.microsoft.com/en-us/download/details.aspx?id=44266 . After installation, make sure VS90COMNTOOLS environment variable has been set correctly.
   If this does not happen (or the user wants to use a more recent build environment), manually setting the path to a VC compiler should do the trick. Have a look at here:  http://stackoverflow.com/questions/2817869/error-unable-to-find-vcvarsall-bat

4. Install prebuilt binaries.
   Windows requires some special care in order to build everything up. In particular, this software depends on lxml which needs development packages for libxml2 and libnxslt which are not trivial to build on windows.
   So, we advise to use prebuilt binaries on windows. Download and install the appropriave lxml and pyxml version from here: http://www.lfd.uci.edu/~gohlke/pythonlibs/#lxml, http://www.lfd.uci.edu/~gohlke/pythonlibs/#pyxml

   In our case, python 2.7 x64 -> http://www.lfd.uci.edu/~gohlke/pythonlibs/tuth5y6k/lxml-3.7.3-cp27-cp27m-win_amd64.whl, http://www.lfd.uci.edu/~gohlke/pythonlibs/tuth5y6k/PyXML-0.8.4-cp27-none-win_amd64.whl

   C:\InstallAnalyzer\scripts\pip install lxml-3.7.3-cp27-cp27m-win_amd64.whl
   C:\InstallAnalyzer\scripts\pip install PyXML-0.8.4-cp27-none-win_amd64.whl

5. Install vboxapi bindings.
   Once again, the installation of vboxapi requires some manual steps. First, make sure you have virtualbox installed on the system.
   Due to a bug with COM/vboxapi bindings, the user needs to use same architectural version of virtual box and python (if planning to use COM). Therefore, in our case, we will download
   virtualbox x64 bit and its SDK.
   Once downloaded, extract the sdk content somewhere, cd into sdk/installer and run

   C:\InstallAnalyzer\scripts\python vboxapisetup.py install

   Then, copy the vboxwebservice bindinds:
   cd sdk/bindings/webservice/python/lib
   copy *.py C:\InstallAnalyzer\lib\site-packages\vboxapi\

6. Install the rest of the binaries
   At this stage, all the "hard" dependencies should be ok. It's time to install the rest of the package, by running:
   C:\InstallAnalyzer\scripts\pip install HostController-0.1.zip

### Supported Hypervisors

### VirtualBox configuration
The host controller can talk to virtualbox in two ways: COM bindings or SOAP webservice. The first is faster, lighter and less compatible. It will mostly work on Windows and allows to manage the vbox hypervisor installed locally. The second option
will rely on a webservice interface and SOAP bindings, so that the HostController talks to the vbox manager via a web interface. Obviously, the second option is heavier, slower but a way more compatible. It allows to use a x32 version of pytho and a x64 version of virtual box (which is not possible with COM bindings on Windows).
In case the user wants to use COM, no furher step is required.
In case the SOAP bindings are chosen, the user needs to configure the virtualbox webservice. To do so, let's perform the following stesp:
1. create an user for the VBox WebService on the local machine and note down its credentials (they will be needed later when configuring the host controller).
2. Setup the vboxwebsrv as autostart windows service:
	Download NSSM from http://nssm.cc/download
	Create a vbox_web_service.bat file, with the following lines
		set HOMEDRIVE=C
		set HOMEPATH=\Users\username
		"%VBOX_MSI_INSTALL_PATH%\VBoxWebSrv.exe"
    Run nssm install "Virtualbox Web Service"
    A GUI appears. Browse to the newly created bat file and press "install service"
    At this stage, the service should appear among the installed services on the OS. We can start and stop it via net start / net stop

    net start “Virtualbox Web Service”

### Configuring Host Controller
The HostController needs to be configured via its configuration file, controller.conf, which is installed alongside the python scripts. Assuming that the HC was installed into C:\InstallerAnalyzer, its configuration file would be located in C:\InstallAnalyzer\Lib\site-packages\HostController\controller.conf.
Let's open it with our favorite text editor. Go through all the settings file and read comments carefully, adjusting values as needed.


## Network sniffer

## Guest Controller
