Topology description
--------------------
As already mentioned, the topology implemented by bare-metal guests includes at least 3 distinct hardware nodes and at least one smart plug:

    - HostController: implementing the central DB, the HostController Agent, the iSCSI service and crawlers;
    - Sniffer: a dedicated machine used as network gateway with sniffing capabilities;
    - Sandboxes: one or more dedicated machines used as sandboxes for the analysis;
    - SmartPlug: one for each Sandbox taking part in the topology. At the moment, only TP-Link HS100 is supported.

.. image:: img\n_tiers_baremetal_details.png
    :alt: N-Tiers - Baremetal topology details

From the picture above we identify two distinct network domains.
The first one is identified by a private LAN connecting together HostController, Sandboxes and the Sniffer.
The second one simply connects the sniffer to the external Internet.

Like in the other topologies, the sniffer behaves as a gateway: it routes traffic incoming on ETH1 (LAN) to ETH1(WAN) and vice versa, implementing NAT.
Therefore, the sniffer is configured to automatically receive an IP address on its ETH0.
On the contrary, it will statically acquire *192.168.0.1* on ETH1.
The sniffer will then serve DHCP and DNS requests on ETH1, assigning IPs from *192.168.0.2* up to *192.168.0.128*.

The HostController resides in the same LAN where Sandboxes are.
Therefore, it binds the address *192.168.0.251*.
Again, since we plan to use a single HostController, the database is hosted locally to the HostController and will bind the loopback address (*127.0.0.1*).

Sandboxes reside in the private LAN and acquire an IP address directly from the sniffer, via DHCP.
Each sandbox is connected to the power line through a smart plug, which joins the private LAN too.
The sniffer will then be configured to assign specific IP addresses to smart plugs so that each of them will be leased an known IP address via DHCP.
Such configuration is necessary in order to create a binding between each smart plug and its connected sandbox.
More details are available in the Sniffer configuration section.

Notes on the iSCSI diskless boot
--------------------------------

.. image:: img\iscsi boot.png
    :alt: Sequence diagram descripbint iSCSI boot procedure for bare-metal sandboxes

Bare metal sandboxes take advantages of diskless boot over iSCSI protocol in order to implement clean state rollback and centralized image management.
The idea is to use diskless sandboxes, which boot over the network via a *PreXecution Environment (PXE)*.
Each sandbox is configured to boot via a permanent USB stick containing a custom version of *iPXE*.
When the sandbox boots up, the iPXE script will perform a special HTTP request against the HostController Service.
When this happens, the HostController will allocate a new differencing disk upon a base virtual disk.
Then it exposes the newly created disk via an iSCSI target. Then, the web request performed by the sandbox is responded with the iSCSI boot parameter that the specific sandbox needs to apply.
As a result, the Sandbox will hook the specified SAN and will boot up.
Every IO against the disk will go trough iSCSI and will only affect the differencing disk attached to that iSCSI LUN.

When the analysis is over, the HostController will hard reboot the Sandbox using the associated power plug.
Therefore the sandbox will boot again in the iPXE environment and the process reiterates.
The only difference with the previous step is that the differencing disk will be deleted and created again (such operation takes almost no time).

Requirements and limitations
----------------------------
Despite other different topologies, the current one necessitates a Windows Server OS to host the HostController service.
Such requirement is due to the need of a scriptable iSCSI service, alongside the support of virtual disks and differencing virtual disks (VHD and VHDX).
While there is no current plan to support Linux hosts for this specific topology, the implementation of a Linux compatible service would not be impossible.

The current implementation also assumes that the guest machines are equal in terms of hardware (same mainboard - CPU).
Such limitation depends on the usage of a single "base image", which includes all the basic configuration and clean state for a single sandbox.
In fact, different sandboxes would probably require different drivers and configurations, thus many different base-image for each hardware configuration.
Such requirement would vanish the benefits of maintaining a single central disk image.

In order to implement machine hard control, we use smart plugs.
Unfortunately, the system only supports TP-Link HS100 smart plugs at the current stage.
However, it is not diffucult to implement specific software layer driver for different smart plugs.
Expert users or pyuthon programmers could adapt the code in order to work with different smart plugs.