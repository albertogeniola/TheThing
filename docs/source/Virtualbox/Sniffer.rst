====================
Sniffer Installation
====================
A sniffer is a machine (which might be physical or virtual) acting as a gateway between the sandboxes and the Internet.
In this tutorial we will refer to it with both *gateway* and *sniffer* terms.
It implements some basic network services for guest machines, alongside a backend web interface, used by the HostController to manage sniffing sessions.
In accordance with the single tier topology, we will virtualize the sniffer alongside the sandboxes.

The user might opt for one of the following options in order to install the prepare the sniffer:
    - Download and use the pre-configured VDI image for *Virtualbox Emulated Sniffer* `from here <http://>`_ (**recommended for first-time users**)
    - Manually configure from scratch a Sniffer VM (**needed for custom network configuration**)

The first option is advised for users who approach to the system for the first time or are not used to any linux environment.
In fact, the pre-configured image has been customized in accordance with the network configuration presented in :ref:`Introduction <twotiers_openstack_topology>`.
If the user has stuck with the topology presented in the previous steps, this image will work out-of-the-box.
On the contrary, if some network configuration has been customized, it is necessary to adjust or configure the image from scratch.

Option A: Download preconfigured image
--------------------------------------
The pre-configured image of Sniffer is [available here](). Such image is a compressed dynamically allocated VDI, of about 1.5 GB.
Once downloaded, extract the image into a known location, e.g. **C:\\InstallAnalyzer**.

The given image configures the first NIC, *eth0*, of the gateway as WAN access.
In our specific configuration, this interface is given the static address *172.16.0.2*.
Its traffic flows through the Virtualbox NAT netowrk, which is internally controlled by the HostController Agent.
The second nic, *eth1*, is then attached to the internal ant connecting guests to the sniffer.
A DHCP service is bound to that NIC and serves addresses from *192.168.2.2/24* up to *192.168.2.250/24*.
The provided image also contains the last version of the MiddleRouter software, that is used for sniffing traffic from guest instances.

Credentials used by that image are:
- Username: ubuntu
- Password: reverse

Such image is ready to be used in test environments.
Before using such image in a production environment we **strongly advise to change username/password credentials**.
This can be done in a number of ways. The easiest is to mount the image on a VM, run it, and change the password of ubuntu user.

That's it for the sniffer. Just proceed to the next step of the tutorial.

Option B: Create the Sniffer virtual machine
--------------------------------------------
In case the user wants to configure the sniffer by herself, she needs to perform the following steps.
Note that this guide only covers the case of installation on Linux Host (in either virtual or physical hosts).

VM Setup
########
To start, let's create a VM with Virtualbox.
Use the Virtualbox wizard to create an Ubuntu virtual machine.
Make sure the VM is given at least 2 Gb of ram memory and a (virtual) disk of 25 Gb of free space.
Once the VM has been created, edit it by enabling both the second and the third network adapters.
Make sure all the three NICs use paravirtualized network (virtio-net) emulation mode.
Then attach *Adapter 1* to NAT and *Adapter 3* to Host-Only adapter created during HostConstroller configuration.
Leave *Adapter 2* unattached.

Then, download and install your favourite Linux distribution on the virtual machine.
Ubuntu 16.04LTS 64bit is suggessted, since it is the one we tested in this tutorial.

MiddleRouter installation
#########################
The next step is to install the MiddleRouter software on the fresh linux installed OS.
This operation is pretty straight forward thanks to the setup script available `here <https://raw.githubusercontent.com/albertogeniola/TheThing-Sniffer/master/install_script.sh>`_ .
Thus, reboot the vm, log into a terminal and then issue the following commands.

.. code-block:: none

    $ cd /tmp
    $ git clone https://github.com/albertogeniola/TheThing-Sniffer.git
    $ cd middlerouter
    $ sudo chmod +x install_script.sh
    $ sudo -H ./install_script.sh
    << follow the wizard >>

When prompted to install the **sniffer agent** type "Y".
Optionally, you may want to install the pcap analyzer on the same host.
If that is the case, answer "y" when prompted.
In our specific case, being a single tier node, we also install the network analyzer on the sniffer.

Configure network dapters
#########################
We now must configure the network connectivity of the sniffer to reflect our network topology as presented in :ref:`the topology description <vbox_topology>`.
In particular:

    - **ETH0** is used as external network connectivity and statically acquires *172.16.0.2*, routing traffic to *172.16.0.1* (VboxNat gateway).
    - **ETH1** is used as interface for internal network (TestNet), which guests connect to. In our case we bind it to *192.168.0.1/24*. Both DHCP and DNS services will insist on this interface.
    - **ETH2** is connected directly to the Host via the Virtualbox HostOnly adapter. The sniffer binds *192.168.56.2* while the Host binds *192.168.56.1*.


Let's edit **/etc/network/interface** so that it matches the following:

.. code-block:: none

    # --------------------------------------------------
    # Loopback device
    auto lo
    iface lo inet loopback

    # WAN/Internet network, where traffic will be routed
    auto eth0
    iface eth0 inet static
        address 172.16.0.2
        netmask 255.255.255.0
        gateway 172.16.0.1
        dns-nameservers 172.16.0.1

    # Internal NIC, where guests are attached
    auto eth1
    iface eth1 inet static
        address 192.168.0.1
        netmask 255.255.255.0

    # HostOnly adapter, used to talk with HostController Agent
    auto eth2
    iface eth2 inet static
        address 192.168.56.2
        netmask 255.255.255.0

    # --------------------------------------------------

**NOTE**:
Interface names might be different on new Linux distributions, because of the new predictable interface naming implementation.
To avoid any issue we suggest to disable such feature by adding a custom switch in the default grub configuration file.
To do so, edit your */etc/default/grub* changing the line from:

.. code-block:: none

    GRUB_CMDLINE_LINUX=""

to:

.. code-block:: none

    GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"


and, finally, update grub.

.. code-block:: none

    $ sudo update-grub

At next reboot, interfaces will be given classical *ethX* names.

Configure network services
##########################
The first step is to adjust the configuration regarding the DHCP service and the DNS relay.
Those are provided by dnsmasq. So, let's edit */etc/dnsmasq.conf*, making it look like the following:

.. code-block:: none

    # --------------------------------------------------
    interface=eth1
    interface=lo
    dhcp-range=192.168.0.2,192.168.0.250,255.255.255.0,12h
    # --------------------------------------------------

This configuration tells to dnsmasq to provide DNS relay and DHCP service on both the loopback interface (lo) and on eth1 (InternalNat).
Is also enables the DHCP server, that assignings IPs in the range between *192.168.0.2* and *192.168.0.250*.

We now need to enable forwarding among interfaces. Edit */etc/sysctl.conf* and adjust the forwarding option to match 1.

.. code-block:: none

    # --------------------------------------------------
    net.ipv4.ip_forward = 1
    # --------------------------------------------------

Then make changes permanent by running

.. code-block:: none

    $ sudo sysctl -p /etc/sysctl.conf

In order to provide external connectivity to guests, the gateway needs to enable the NAT service. This is done by leaveraging _iptables_.
Our objective is to forward packets from internal network, on */dev/eth1*, to the external network on */dev/eth0*. Therefore we issue the following commands.

.. code-block:: none

    $ sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
    $ sudo iptables -A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT
    $ sudo iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT

    $ sudo iptables -t nat -A POSTROUTING -o eth2 -j MASQUERADE
    $ sudo iptables -A FORWARD -i eth2 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT
    $ sudo iptables -A FORWARD -i eth1 -o eth2 -j ACCEPT

Since IPTABLES rules are not automatically persisted, we need to save them.

.. code-block:: none

    $ sudo apt-get install iptables-persistent
    $ sudo iptables-save > /etc/iptables/rules.v4

Security Enforcements
#####################
TODO: limiting netowrk capabilities of guests (i.e. avoid them to access LAN where HOST is connected).

TODO: make the sniffer gateway to bind only *192.168.56.2* and block traffic from *192.168.0.x/24* to *192.168.56.2*.

Check everything is ok
######################
Upon successful reboot, make sure the sniffer service is correctly running, by typing:

.. code-block:: none

    $ sudo reboot
    <<system reboots>>
    $ sudo service sniffer status
    <<verify it is running>>
    $ sudo service analyzer status
    <<verify it is running>>

In case it is not running, check the output log for the cause:

.. code-block:: none

    $ cat /var/log/sniffer.err
    <<inspect>>
    $ cat /var/log/analyzer.err
    <<inspect>>

Finally, the very last test that confirms the correct installation of the sniffer and its good network configurationFrom the Host controller, make sure the web-service interface is working.
Open a browser and type:

.. code-block:: none

    http://192.168.56.1:8080


Image storage
-------------
At this point the image of the sniffer is ready. However, it a good idea to clone it into **C:\\InstallAnalyzer\\**.
To do so, ensure the sniffer is powered off, then open a terminal on the host and type the following command:

.. code-block:: none

    C:\> cd "%PROGRAMFILES%\Oracle\VirtualBox"
    ..\> VBoxManage clonehd <path_to_old_image> "C:\InstallAnalyzer\sniffer.vdi" --format VDI

