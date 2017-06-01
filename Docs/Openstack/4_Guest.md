# Guest installation
This section of the tutorial will guide the user through the preparation of a virtual disk image, which will be used as base disk for the virtual Sandbox.
Although most of the preparation steps are common to any kind of Guest, this guide targets the case of single tier configuration for a Windows 7 SP1 32bit client, to be used with an Openstack KVM service.

In summary, the user needs to:

1. Creating the base image via Virtualbox or QEMU
1. Installing the Guest Agent
1. Saving and storing the VM image

The creation of the base image consists of creating a virtual machine and installing Windows 7 32bit on it. 
Such task can be performed both on a Windows or a Linux host. In this tutorial we'll only explain how to do that from within a Windows Host.
If needing to prepare the Guest Image from a Linux Host, refer to the good tutorial maintained by [Openstack Documentation](https://docs.openstack.org/image-guide/windows-image.html). 
In scu case, the user has to prepare the image via QEMU and then install the Guest Agent binaries before uploading the image to the Glance image service.

## Creating the base image on Windows
This tutorial shows how to create the base Guest Image by using Virtualbox as hypervisor and then injecting the needed drivers via DISM utility.

The first step is to create a virtual disk. Since we will then need to inject drivers into the disk, we use a disk format that is compatible with DISM utility, i.e. a VHD disk. 
So, open a command prompt as an administrator and type the following commands:

```
C:\> cd "%PROGRAMFILES%\Oracle\Virtualbox"
C:\> VBoxManage createhd --filename "C:\InstallAnalyzer\Disks\guest_preparation.vhd" --size 25000 --fromat VHD
```

Now, we want to create a VM and attach the newly created disk to that.

```
C:\> VBoxManage createvm --name "guest_preparation" --ostype "Windows7" --register
C:\> VBoxManage storagectl "guest_preparation" --name "SATA Controller" --add sata --controller IntelAHCI
C:\> VBoxManage storageattach "guest_preparation" --storagectl "SATA Controller" --port 0 --device 0 --type hdd --medium "C:\InstallAnalyzer\Disks\guest_preparation.vhd"
```

In order to install Windows 7 on the machine, we will need a virtual CD/DVD drive, that will mount our installation media. Therefore:

```
C:\> VBoxManage storagectl "guest_preparation" --name "IDE Controller" --add ide
C:\> VBoxManage storageattach "guest_preparation" --storagectl "IDE Controller" --port 0 --device 0 --type dvddrive --medium /path/to/windows_7_SP1_32.iso
```

Finally, let's configure some other settings, such as ram, vram and hardware acceleration

```
C:\> VBoxManage modifyvm "guest_preparation" --ioapic on
C:\> VBoxManage modifyvm "guest_preparation" --boot1 dvd --boot2 disk --boot3 none --boot4 none
C:\> VBoxManage modifyvm "guest_preparation" --memory 2048 --vram 128
C:\> VBoxManage modifyvm "guest_preparation" --nic1 nat --nictype1 82540EM
```

At this point the virtual machine is ready for the installation of the operating system, so let's start it right away:

```
C:\> VBoxManage startvm "guest_preparation"
```

The virtual machine will start its boot sequence. The windows installation procedure should begin shortly after the boot up. The user now needs to follow the installation wizard like in a normal machine.

Once the installation is completed, the installation wizard will ask the user to choose an username and a password. While the user can type any kind of username, it is strictly **mandatory to set no password**, so that the auto log-on is activated. Note that this system will not work in case a password is set.

##### System updates (also applies for Linux Host preparation)
At this stage the Guest is capable of surfing the Internet via its own virtual adapter (which is natted behind the network of the host).
Therefore, the user can now activate the operating system. If needed, the user can also update the system via Windows Update. Once the updates have been correctly installed, ww strongly advise to disable the automatic update feature.
Doing so, the system will not try to update the OS during the analysis. On the contrary, in case automatic updates are left enabled, there is a chance that they will trigger during the analysis, impacting on performance and potentially biasing sniffed network traffic.

Once the system has been correctly activated and updates have been performed, the user can then proceed with the installation of the GuestAgent bootstrapper.

##### Install .NET 4.0 and VC Redist 2013 (also applies for Linux Host preparation)
Before installing the GuestAgent Bootstrapper, the system must comply with some software package dependency.
In particular, the agents rely on both Visual Studio 2013 Visual C++ registributable and .NET 4.0.

Thus, download and install the 32bit version of the .NET framework from the Microsoft website. 
The correct version of the .NET framework can be [found here](https://www.microsoft.com/en-us/download/details.aspx?id=17718).

Then, do the same for the VC 2013 ++ Redistributable Package. 
The 32 bit version [is available here](http://download.microsoft.com/download/0/5/6/056dcda9-d667-4e27-8001-8a0c6971d6b1/vcredist_x86.exe).

##### Install the GuestAgent Bootstrapper (also applies for Linux Host preparation)
From within the Virtual Machine, open a browser and dowload the precompiled installation package for the guest agent at [this URL](https://albertogeniola@bitbucket.org/aaltopuppaper/guestagents/raw/0594043ec791e95944487a3646c9994ebf045fd6/ClientBootstrapper/dist/agent_setup.exe). 

Then, execute the installation of the bootstrapper, by simply double clicking on it. Then, follow the wizard to complete the installation. The installer will take care of downloading the needed python environment, necessary dependencies and will also install the bootstrap autostart task.

To double check the bootstrapper installation, reboot the virtual machine. 
Just after Windows loads up, the bootstrapper program should automatically start, complaining about _no response from any sniffer_. 
If that is the case, the bootstrapper is correctly working. 
Now close the bootstrapper and shut down the virtual machine correctly.

##### User's defined customization (also applies for Linux Host preparation)
At this stage, the user might apply some specific customization to the image. For instance, he might want to install a new browser or some flash player plugin. 
He can also install common software usually available on desktop computers, such as Java runtime or Microsoft Office. 
If planning to analyze evasive binaries, the user should also surf the web and create fake social network accounts, so that cookies are left on the system.
In our tutorial we do not perform any of these operations.

##### Disable startup repair on unclean reboot
It might happen that a VM reboots unexpectedly. When this happens, Microsoft Windows OS tend to start the Startup Recovery process, which require human actions to be completed.
In our case, such recovery process cannot be applied, therefore we need to disable it.

To do so, open a command prompt as administrator and type the following command:

```
C:\> bcdedit /set {default} recoveryenabled No
```

##### Installing virt-io specific drivers
Openstack virtualization driver might differ from the one used by Virtualbox. Most of the times Openstack is configured to uses libvirt and KVM for its guests.
In such case, we need to make our guests compatible with KVM module. Thus, we need to manually inject some _virtio_ drivers into the Windows installation.

To do so, let's gracefully shutdown the virtual machine used so far (Start->shutdown from within the Guest).
Then, let's download the VIRTIO signed drivers provided by RedHat [at this address](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso).
Once done, mount or extract the ISO into a specific location. We will refer to that as _VIRTIO_DIR_.

It is not time to open an elevated command prompt from within the host. Then, we can mount the VHD image via the DISM utility.

```
C:\> Dism /Mount-Image /ImageFile:C:\InstallAnalyzer\Disks\guest_preparation.vhd /Index:1 /MountDir:C:\test
```

Hence, let's add the storage, network, USB and PCI drivers with the following commands:

```
C:\> Dism /Image:C:\test /Add-Driver /Driver:VIRTIO_DIR/viostor/w7/x86
C:\> Dism /Image:C:\test /Add-Driver /Driver:VIRTIO_DIR/NetKVM/w7/x86
C:\> Dism /Image:C:\test /Add-Driver /Driver:VIRTIO_DIR/vioserial/w7/x86
C:\> Dism /Image:C:\test /Add-Driver /Driver:VIRTIO_DIR/Balloon/w7/x86
```

Finally, commit the changes and unmount the image:

```
C:\> Dism /Unmount-Image /MountDir:C:\InstallAnalyzer\Disks\guest_preparation.vhd /Commit
```

## Uploading the image to the Glance Service
Once the image has been prepared, we can now safely upload it to the Openstack cloud. This can be done via Horizon web service (if available) or via CLI. 
We choose the CLI, so we issue the following command:

```
glance image-create --name SandboxImage --disk-format vhd --container-format bare --file C:\InstallAnalyzer\Disks\guest_preparation.vhd --progress
```

This concludes the preparation of the Sandbox image.

## What next?
The entire tutorial is divided into 5 steps, to be followed in order:

1. [Introduction](1_Introduction.md)
1. [Database and HostController installation](2_DB_and_HostController.md)
1. [Sniffer installation](4_Guest.md)
1. Guest installation
1. [HostController configuration](5_Configuration.md) (_next step_)
