A sniffer is a machine (which might be physical or virtual) acting as a gateway between analyzer guests and the internet. As such it operates as a gateway and needs to
implement some basic network services for guest machines, alongside a backend allowing the sniffer to be ran by the HostController. At the time of writing, a full working
implementation of a sniffer is available by simply downloading a pre-configured VDI image at XXXXX. The given image configures eth0 of the gateway as WAN/Internet access
and obtains its configuration via DHCP. The second nic, eth1, is then used as NAT for guests. Therefore a DHCP service is bound to that NIC and serves addresses from
192.168.2.2/24 up to 192.168.2.250/24. The provided image also contains the last version of the MiddleRouter software, that is used for sniffing traffic from guest
instances.
Credentials used by that image are:
Username: ubuntu
Pass: reverse
Such image is ready to be used in test environments. Before using such image in a production environment we strongly advise to change username/password credentials.

In case the user wants to configure the sniffer by herself, she needs to perform the following steps. Note that this guide only covers the case of installation on Linux
Host (in either virtual or physical hosts).
0. Make sure the guest/vm has two idependent NICs (eth0 and eth1). The first needs to be connected to outbound internet and acquire the address via DHCP.
1. Download and install your favourite Linux distribution. We assume Ubuntu 14.04 LTS is installed.

2. Install dependant packages
sudo apt-get update
sudo apt-get install build-essential libssl-dev dnsmasq
sudo apt-get install python2.7 python-pip python-dev libffi-dev libfuzzy-dev python-libxml2 libxsl1-dev libxml2-dev
sudo apt-get install python2-dev python2-pip libffi-dev libssl-dev pillow
sudo apt-get install mitmproxy
sudo apt-get install git

3. Install the middle router software from git
git clone https://github.com/albertogeniola/MiddleRouter.git
cd MiddleRouter
sudo pip install . --upgrade

4. Network configuration
The user might now configure the network connectivity of the sniffer so that it reflects his needs. There are a many possible network topologies that can be used, however
the current implementation of the analysis system works as follows.
-> ETH0 is used as external network connectivity. It can be seen as "WAN" network. Therefore it is a good idea to have it configured via DHCP if available on that
network.
-> ETH1 is used as interface for internal network, which guests connect to. Therefore it must be configured as static to some private IP address. A number of services
will bind on that NIC/IP-address, e.g. DHCP. Note that the configuration of this interface will also affect the configuration of the other services offered by the
sniffer. We advise to use 192.168.2.1/24 network. Thus, /etc/network/interface will look like the following.
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
	address 192.168.2.1
	netmask 255.255.255.0
# --------------------------------------------------

5. Configuring DHCP/DNS relay
Edit /etc/dnsmasq.conf and adjust the following options.
# --------------------------------------------------
interface=eth1 # Only provide DHCP DNS on ETH1.
dhcp-range=192.168.2.2,192.168.2.250,255.255.255.0,12h # Eanble DHCP server, so that first lased ip is 192.168.2.2 and last is 192.168.2.250 with a lease time of 12h
# --------------------------------------------------
This configuration tells to dnsmasq to provide DNS relay and DHCP service on eth1, assigning IPs in the range between 192.168.2.2 and 192.168.2.250

6. Enabling forwarding
Edit /etc/sysctl.conf and adjust the forwarding option to match 1.
# --------------------------------------------------
net.ipv4.ip_forward = 1
# --------------------------------------------------
Then make changes permanent by running
sudo sysctl -p /etc/sysctl.conf

Then restart networking
sudo /etc/init.d/networking restart

7. Enabling NAT
In order to provide external connectivity to guests, the gateway needs to enable the NAT service. This is done by leaveraging iptables.
Our objective is to forward packets from internal network, on /dev/eth1, to the external network on /dev/eth0. Therefore we issue the following commands.

sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT

Since IPTABLES rules are not automatically persisted, we need to save them.

sudo apt-get install iptables-persistent
sudo iptables-save > /etc/iptables/rules.v4

8. Install dependent analysis binaries
sudo apt-get install tshark bro
sudo apt-get install cmake make gcc g++ flex bison libpcap-dev libssl-dev python-dev swig zlib1g-dev
cd /home/ubuntu
git clone --recursive git://git.bro.org/bro
cd bro
./configure
sudo make
sudo make install

Once the installation is complete, we can remove the building directory
rm -vR bro

Thus, let's link the bro binary
sudo ln -s /usr/local/bro/bin/bro /usr/bin/bro

Finally let's make tell the system to automatically run this service at startup:
sudo update-rc.d sniffer defaults
sudo update-rc.d sniffer enable

9. Rebootthe system.
Upon successful reboot, make sure the sniffer service is correctly running, by typing:
sudo service sniffer status

In case it is not running, check the output log for the cause:
cat /var/log/sniffer.err

10. From the Host controller, make sure the web-service interface is working. Open a browser and type:
http://sniffer-ip-address:8080

11. If planning to use the image on Openstack, the user has to convert it in raw format. This can be done in a number of ways. For instance, VBoxManage utility is capable
to do that. 