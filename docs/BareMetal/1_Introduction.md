# Introduction
Bare metal sandboxes take advantages of diskless boot over iSCSI protocol.
In particular the HostController exposes on iSCSI target for each Sandbox node.
Each sandbox is configured to mount the iSCSI target and booting the OS directly from there. 
Such objective is achieved by booting into a _PreXecution Environment_ (PXE), mounting the associated iSCSI target and then booting.

The preparation of such nodes is time consuming and requires basic knowledge of iSCSI and PXE technologies. 
However, in this tutorial we will guide the user in the whole process.

## Architecture overview



1. Introduction
1. [Database and HostController installation](2_DB_and_HostController.md) (_next step_)
1. [Guest installation](4_Guest_Preparation.md)
1. [Sniffer installation](4_Sniffer.md)
1. [HostController configuration](5_Configuration.md)
