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




Bare metal sandboxes take advantages of diskless boot over iSCSI protocol.
In particular the HostController exposes on iSCSI target for each Sandbox node.
Each sandbox is configured to mount the iSCSI target and booting the OS directly from there.
Such objective is achieved by booting into a _PreXecution Environment_ (PXE), mounting the associated iSCSI target and then booting.

The preparation of such nodes is time consuming and requires basic knowledge of iSCSI and PXE technologies.
However, in this tutorial we will guide the user in the whole process.

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