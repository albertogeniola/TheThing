.. _singletier_vbox:
Single tier configuration, using Virtualbox.
============================================

A single tier configuration consists in installing all the mandatory parts of the distributed system on a single hardware node. Obviously, such a configuration frustrates the advantages of a distributed system (scalability, robustness, etc.). On the other hand, it enables a simpler configuration scheme and simplifies some security issues affecting more complex topologies.

In general, we do recommend this configuration if one or more of the following conditions are met:

- This is the first time the user is experimenting the infrastructure.
- Only a single, robust and high-performance server can be used for deploying the infrastructure.
- Security is a must.
- The number of installers to be analyzed is small or analysis time can be long.

In this tutorial, the user will learn how to deploy the central DB, an HostController Agent and the Virtualbox virtualization system both on a Windows Server 2012 64bit operating system and Linux Ubuntu 16.04 LTS. Each one of the following sections is indeed splitted in two parts: the first one describes install instructions for Windows, while the second one regards Linux.

.. toctree::
   :maxdepth: 3
   :caption: Contents:

   Topology
   HostController
   Guest
   Sniffer
   Configuration

