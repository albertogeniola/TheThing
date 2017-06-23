# Two tiers configuration, using Openstack.
A two tiers configuration uses two distinct nodes to implement the analysis infrastructure.
In particular, a local node will host the central database and the host controller, while a remote cloud infrastructure (i.e. Openstack) is in charge of running the sandboxes and the sniffers.

In contrast with the **Single tier** configuration, this one has the advantage of offloading the heavy virtualization task to a remote infrastructure (e.g. Openstackcloud).

In general, we do recommend this configuration if one or more of the following conditions are met:

- The user can leverage a powerful Openstack infrastructure (which might be remote or in his intranet)
- The numberof installers to be analyzed is great and throughput should be high
- The user trusts the remote cloud infrastructure

This tutorial will guide the user in the installation of Central Database, Host Controller and in the preparation of the sniffer instance to be used on the openstack cloud. Moreover, the user will learn how to deploy the infrastructure on an Openstack Cloud.

## Topology description
A two tiers topology consists in two nodes: a local node handling the central database together with the Host Agent, and a remote cloud infrastructure.
_Figure 1_ depicts the configuration goal of the two tiers topology that the user will deploy following this tutorial.

![Two tiers - openstack topology details](img/TwoTiersOSConf1.png "Figure 1: Topology overview")

As already mentioned, there are two major entities in game: a local server (tier 1) hosts the database, the crawlers and the HostController Agent while the remote openstack cloud hosts sniffers and sandboxes.
Communication between HostController and database happens locally to the tier 1. This means that the DB instance will only bind the loopback address. On the contrary, communication among sniffer, HostController Agent and Sandboxes happen via Internet/Intranet, depending on the user's specific topology. 

### Tier 1
For sake of simplicity, we assume the user is using public Internet. This means that a gateway exists bnetween the Tier 1 and the ISP, which usually implements both L3/L4 services such as routing, nat and firewall.
For this reason, the GuestController agent needs to bind the private address provided by the gateway (either via DHCP or statically assigned). Moreover the user must configure by herself the necessary firewall/natting rules to allow communication between remote cloud and GuestAgent.
In particular the user must redirect incoming traffic to port TCP 9000 to the HostController Agent. 
Referring to the picture, the rule should be defined like the following:

External IP | External Port Start | External Port End | Internal IP | Internal Port
--- | --- | --- | --- | --- |
0.0.0.0 | 9000 | 9000 | X.X.X.X | 9000 |

### Tier 2
The Openstack network configuration is based on the coexistence of two networks: an Internal Network and an External Network. 
Sandboxes live within the Internal Network.
They receive the network address via Openstack DHCP service. Moreover, the DHCP service also configures the default gateway of sandboxes, setting it to the address of the Sniffer.
Therefore, the sniffer acts as a gateway between the Internal Network (_192.168.0.0/24_) and the External Network (192.168.1.0/24).

The sniffer is configured to acquire the address of the public network via DHCP, while statically binding _192.168.0.1_ on the Internal Network.
Moreover, the sniffer must also acquire a floating public IP, so that it can be easily reachable by the HostController via the public Internet.

## What needs to be done manually
Most of the network configuration of the remote openstack cloud is performed automatically by the HostController Manager, based on its configuration file. 
However, there are a few things that cannot be automated. The user has to perform such tasks based on her specific network topology.
In particular:
- Open port TCP 9000 on tier 1 via port mapping, allowing incoming connections from the Internet (or from the specific openstack neutron node)
- Set the HostController Agent to listen on the private IP address where TCP 9000 has been mapped to (X.X.X.X)
- Allocate the External Network on Openstack cloud (we assume its CIDR is 192.168.1.0/24)
- Create a Router on the External Network that routes traffic to the Internet

The rest of networking configuration is handled automatically by the HostController Agent, which uses the Openstack API directly to setup the Internal Network, connect the required interfaces, provide routing capabilities to the Sniffer Instance.

## What's next?
The entire tutorial is divided into 5 steps, to be followed in order:

1. Introduction
1. [Database and HostController installation](2_DB_and_HostController.md) (_next step_)
1. [Sniffer installation](3_Sniffer.md)
1. [Guest installation](4_Guest.md)
1. [HostController configuration](5_Configuration.md)

