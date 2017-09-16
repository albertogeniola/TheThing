# Introduction
A sniffer is a machine (which might be physical or virtual) acting as a gateway between analyzer guests and the Internet. In this tutorial we will refer to it with both _gateway_ and _sniffer_ terms.
It implements some basic network services for guest machines, alongside a backend web interface, used by the HostController to manage sniffing sessions.

This tutorial will guide the user in installing and configuring the sniffer in accordance with the network topology presented in the [Introduction](1_Introduction.md).
We strongly advice to follow the tutorial step by step and to pay close attention to each part of it.

## Sniffer Machine setup
We will use a dedicated physical machine as sniffer. Such machine is required to have two distinct network adapters, Linux compatible.
More advanced users might rely on VLANs and virtual interfaces when there is only a single hardware NIC available on the sniffer node.
However such case is not addressed by this specific tutorial.

Let's now install Ubuntu 16.04 LTS on the sniffer node.
To do so, download and burn on a preferred media Ubuntu 16.04LTS 64bit.
Then, plug the media into the sniffer node and start the Ubuntu installation wizard. Follow the wizard until the operating system is installed.

## MiddleRouter installation
The next step is to install the MiddleRouter software on the fresh linux installed OS.
This operation is pretty straight forward thanks to the setup script which is bundled with the middle router source code.
Log into the sniffer terminal and then issue the following commands.

```
$ cd /tmp
$ git clone https://github.com/albertogeniola/TheThing-Sniffer.git
$ cd middlerouter
$ sudo chmod +x install_script.sh
$ sudo -H ./install_script.sh
<< follow the wizard >>
```

When prompted to install the **sniffer agent** type "Y".
Optionally, you may want to install the pcap analyzer on the same host.
If that is the case, answer "y" when prompted.
In our specific case, being a single tier node, we also install the network analyzer on the sniffer.

## Configure network dapters
We now must configure the network connectivity of the sniffer to reflect our network topology as presented in [Introduction](1_Introduction.md).
In particular:

- **ETH0** is used as external network connectivity and dynamically acquires the network address via the external DHCP/Gateway.
- **ETH1** is used as interface for internal network, which guests connect to. In our case we bind it to 192.168.0.1/24. A separate DHCP and DNS services will insist on this interface.

Let's edit __/etc/network/interface__ so that it matches the following:
```
# --------------------------------------------------
# Loopback device
auto lo
iface lo inet loopback

# WAN/Internet network, where traffic will be routed
auto eth0
iface eth0 inet dhcp

# Internal NIC, where guests are attached
auto eth1
iface eth1 inet static
	address 192.168.0.1
	netmask 255.255.255.0

# --------------------------------------------------
```

__NOTE__:
Interface names might be different on new Linux distributions, because of the new predictable interface naming implementation.
To avoid any issue we suggest to disable such feature by adding a custom switch in the default grub configuration file.
To do so, edit your _/etc/default/grub_ changing the line from:

```
GRUB_CMDLINE_LINUX=""
```

to:
```
GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"
```

and, finally, update grub.
```
$ sudo update-grub
```

At next reboot, interfaces will be given classical _ethX_ names.

## Configure network services
The first step is to adjust the configuration regarding the DHCP service and the DNS relay. Those are provided by dnsmasq. So, let's edit _/etc/dnsmasq.conf_, making it look like the following:

```
# --------------------------------------------------
interface=eth1
interface=lo
dhcp-range=192.168.0.2,192.168.0.250,255.255.255.0,12h
# --------------------------------------------------
```
This configuration tells to dnsmasq to provide DNS relay and DHCP service on both the loopback interface (lo) and on eth1 (InternalNat).
Is also enables the DHCP server, that assignings IPs in the range between 192.168.0.2 and 192.168.0.125.

We now need to enable forwarding among interfaces. Edit /etc/sysctl.conf and adjust the forwarding option to match 1.
```
# --------------------------------------------------
net.ipv4.ip_forward = 1
# --------------------------------------------------
```

Then make changes permanent by running
```
$ sudo sysctl -p /etc/sysctl.conf
```

In order to provide external connectivity to guests, the gateway needs to enable the NAT service. This is done by leveraging _iptables_.
Our objective is to forward packets from internal network, on /dev/eth1, to the external network on /dev/eth0. Therefore we issue the following commands.

```
$ sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
$ sudo iptables -A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT
$ sudo iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
```

Since IPTABLES rules are not automatically persisted, we need to save them.

```
$ sudo apt-get install iptables-persistent
$ sudo iptables-save > /etc/iptables/rules.v4
```

## Edit sniffer configuration file
At this point, the user has to setup the sniffer configuration file in accordance with the network topology she has set up.
In particular the sniffer needs to know on which interface the traffic has to be sniffed.
In case the user has strictly followed our tutorial, setting up the environment with the exact network topology and network interface, then there is nothing to align: the sniffer will sinply assume ETH1 is the interface where to listen.

On the contrary, in case the internal traffic is routed via an interface which is not ETH1, the user has to change its name to the correct value.
To do so, open a terminal on the sniffer and edit the file located in: _/usr/share/MiddleRouter/router/conf.py_ and change the value of "CAPTURE_IF" to the correct one.
```
# Change the following line in case eth1 is not the interface where internal traffic is routed
CAPTURE_IF = "eth1"
```

Be advised: that is a python configuration file and might respect python's syntax. The user should only change the right values values she is interested into without adding TABS or spaces.

## Check everything is ok
Upon successful reboot, make sure the sniffer service is correctly running, by typing:

```
$ sudo reboot
<<system reboots>>
$ sudo service sniffer status
<<verify it is running>>
$ sudo service analyzer status
<<verify it is running>>
```

In case it is not running, check the output log for the cause:

```
$ cat /var/log/sniffer.err
<<inspect>>
$ cat /var/log/analyzer.err
<<inspect>>
```

Finally, to verify the correct installation of the sniffer we can verify if the sniffer webservice is visible in the local network.
To do so, from the Host Controller node open a browser and type:

```
http://192.168.0.1:8080
```

Upon successful connection, a simple web page will show up (containing nothing more than a page title).

## What next?
The entire tutorial is divided into 5 steps, to be followed in order:

1. [Introduction](1_Introduction.md)
1. [Database and HostController installation](2_DB_and_HostController.md)
1. [Guest installation](4_Guest_Preparation.md)
1. Sniffer installation
1. [HostController configuration](5_Configuration.md) (_next step_)
