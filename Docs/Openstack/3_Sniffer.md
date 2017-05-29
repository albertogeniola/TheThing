# Introduction
A sniffer is a machine (which might be physical or virtual) acting as a gateway between analyzer guests and the Internet. In this tutorial we will refer to it with both _gateway_ and _sniffer_ terms. It implements some basic network services for guest machines, alongside a backend web interface, used by the HostController to manage sniffing sessions.
In accordance with the single tier topology, we will virtualize the sniffer alongside the guest vms.

The user might opt for one of the following options in order to install the sniffer virtual image:
- Download and use the pre-configured VDI image for _Virtualbox Emulated Sniffer_ [from here]() (__recommended for first-time users__)
- Manually configure from scratch a Sniffer VM (__needed for custom network configuration__)

The first option is advised for users who approach to the system for the first time. In fact, the pre-configured image has been customized in accordance with the network configuration presented in [Introduction](1_Introduction.md). If the user has stuck with the topology presented in the previous steps, this image will work out-of-the-box. On the contrary, if some network configuration has been customized, it is necessary to adjust or configure the image from scratch.

## Option A: Download preconfigured image
This approach consists in using a pre-configured linux image which reflects all the configuration needed to work with a two tier architecture via remote Openstack cloud.
Still, the user must change the default access credentials to the machine, preventing others to obtain unlegitimate access to such instance.

So, the first step is to download the preconfigured image, which is [available here]().
Once downloaded, extract the image into a known location, e.g. __C:\InstallAnalyzer__.
Then, use your favourite virtualization system to create a virtual machine and mount such image as primary disk.
Hence, boot the VM and log in with the following credentials:

- Username: ubuntu
- Password: reverse

Once logged in, change the default username/password credentials with the ones you prefer, issuing the following commands:

```
sudo passwd ubuntu
<<type new password>>
<<confirm password>>
```

Now gracefully shutdown the machine and unmount the disk image.

The last step is to upload the image to the Openstack cloud. This can be done via the Horizon web interface (if available) or via the image service CLI (a.k.a. Glance).
If using the CLI, open a terminal and type the following (wither in windows or linux):

```
glance image-create --name SnifferImage --disk-format VDI --file <path_to_prepared_image> --progress
```
That command will upload the image located at _<path_to_prepared_image>_ to the Glance image service, assigning to it the name _SnifferImage_ and showing the progress of the operation. 

That's it for the sniffer. Just go to the next step, [Guest installation](4_Guest.md)\.

## Option B: Create the Sniffer virtual machine
In case the user wants to configure the sniffer by herself, she needs to perform the following steps. Note that this guide only covers the case of installation on Linux Host (in either virtual or physical hosts).

### VM Setup
To start, let's create a VM with Virtualbox. Use the Virtualbox wizard to create an Ubuntu virtual machine. Make sure the VM is given at least 2 Gb of ram memory and a (virtual) disk of 25 Gb of free space. Once the VM has been created, edit it by enabling both the second and the third network adapters. Make sure all the three NICs use paravirtualized network (virtio-net) emulation mode. Then attach _Adapter 1_ to NAT and _Adapter 3_ to Host-Only adapter created during HostConstroller configuration. Leave _Adapter 2_ unattached.

Then, download and install your favourite Linux distribution on the virtual machine. Ubuntu 16.04LTS 64bit is suggessted, since it is the one we tested in this tutorial.

### MiddleRouter installation
The next step is to install the MiddleRouter software on the fresh linux installed OS. This operation is pretty straight forward thanks to the setup script available [here](https://github.com/albertogeniola/MiddleRouter/raw/master/install_script.sh). Thus, reboot the vm, log into a terminal and then issue the following commands.

```
$ cd /tmp
$ git clone https://albertogeniola@bitbucket.org/aaltopuppaper/middlerouter.git
$ cd middlerouter
$ sudo chmod +x install_script.sh
$ sudo -H ./install_script.sh
<< follow the wizard >>
```

When prompted to install the **sniffer agent** type "Y". Optionally, you may want to install the pcap analyzer on the same host. If that is the case, answer "y" when prompted. In our specific case, being a single tier node, we also install the network analyzer on the sniffer.

### Configure network dapters
We now must configure the network connectivity of the sniffer to reflect our network topology as presented in [Introduction](1_Introduction.md).
In particular:

- **ETH0** is used as external network connectivity and dynamically acquires the ip address.
- **ETH1** is used as interface for internal network (TestNet), which guests connect to. In our case we bind it to 192.168.0.1/24. Both DHCP and DNS services will insist on this interface.

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
Interface names might be different on new Linux distributions, because of the new predictable interfgace naming implementation. To avoid any issue we suggest to disable such feature by adding a custom switch in the default grub configuration file.
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

### Configure network services
The first step is to adjust the configuration regarding the DHCP service and the DNS relay. Those are provided by dnsmasq. So, let's edit _/etc/dnsmasq.conf_, making it look like the following:

```
# --------------------------------------------------
interface=eth1
interface=lo
dhcp-range=192.168.0.2,192.168.0.250,255.255.255.0,12h
# --------------------------------------------------
```
This configuration tells to dnsmasq to provide DNS relay and DHCP service on both the loopback interface (lo) and on eth1 (InternalNat).
Is also enables the DHCP server, that assignings IPs in the range between 192.168.0.2 and 192.168.0.250.

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

In order to provide external connectivity to guests, the gateway needs to enable the NAT service. This is done by leaveraging _iptables_.
Our objective is to forward packets from internal network, on /dev/eth1, to the external network on /dev/eth0. Therefore we issue the following commands.

```
$ sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
$ sudo iptables -A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT
$ sudo iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT

$ sudo iptables -t nat -A POSTROUTING -o eth2 -j MASQUERADE
$ sudo iptables -A FORWARD -i eth2 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT
$ sudo iptables -A FORWARD -i eth1 -o eth2 -j ACCEPT
```

Since IPTABLES rules are not automatically persisted, we need to save them.

```
$ sudo apt-get install iptables-persistent
$ sudo iptables-save > /etc/iptables/rules.v4
```

### Security Enforcements
TODO: limiting netowrk capabilities of guests (i.e. avoid them to access LAN where HOST is connected).

TODO: make the sniffer gateway to bind only 192.168.56.2 and block traffic from _192.168.0.x/24_ to _192.168.56.2_.

### Check everything is ok
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

Finally, the very last test that confirms the correct installation of the sniffer and its good network configurationFrom the Host controller, make sure the web-service interface is working. Open a browser and type:
```
http://192.168.56.1:8080
```

## Image upload
At this point the image of the sniffer is ready. We can proceed with the upload into the Openstack cloud.
First, ensure the local virtual machine is powered off, then unmount the disk image.

Then, if Horizon service is available, the user can simply proceed with the image upload via the web interface. 
If command line interface is preferred, the user can do the same by issuing the following command (both in Linux or Windows):

```
glance image-create --name SnifferImage --disk-format VDI --file <path_to_prepared_image> --progress
```

That command will upload the image located at _<path_to_prepared_image>_ to the Glance image service, assigning to it the name _SnifferImage_ and showing the progress of the operation. 

## What next?
The entire tutorial is divided into 5 steps, to be followed in order:

1. [Introduction](1_Introduction.md)
1. [Database and HostController installation](2_DB_and_HostController.md)
1. Sniffer installation
1. [Guest installation](4_Guest.md) (_next step_)
1. [HostController configuration](5_Configuration.md)
