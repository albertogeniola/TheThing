.. _twotiers_openstack:

Two tiers configuration, using Openstack.
=========================================
A two tiers configuration uses two distinct nodes to implement the analysis infrastructure.
In particular, a local node will host the central database and the host controller, while a remote cloud infrastructure (i.e. Openstack) is in charge of running the sandboxes and the sniffers.

In contrast with the **Single tier** configuration, this one has the advantage of offloading the heavy virtualization task to a remote infrastructure (e.g. Openstackcloud).

In general, we do recommend this configuration if one or more of the following conditions are met:

- The user can leverage a powerful Openstack infrastructure (which might be remote or in his intranet)
- The numberof installers to be analyzed is great and throughput should be high
- The user trusts the remote cloud infrastructure

This tutorial will guide the user in the installation of Central Database, Host Controller and in the preparation of the sniffer instance to be used on the openstack cloud. Moreover, the user will learn how to deploy the infrastructure on an Openstack Cloud.

.. toctree::
   :caption: Contents:

   Topology
   HostController
   Sniffer
   Guest
   Configuration

