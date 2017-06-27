Topology description
--------------------
As already mentioned, the topology implemented by bare-metal guests includes at least 3 distinct hardware nodes:

    - HostController: implementing the central DB, the HostController Agent, the iSCSI service and crawlers;
    - Sniffer: a dedicated machine used as network gateway with sniffing capabilities;
    - Sandboxes: one or more dedicated machines used as sandboxes for the analysis;



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
