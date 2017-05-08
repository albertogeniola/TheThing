# Guest installation
This section of the tutorial will guide the user through the preparation of a virtual disk image, which will be used as base disk for the virtual Sandbox.
Although most of the preparation steps are common to any kind of Guest, this guide targets the case of single tier configuration for a Windows 7 SP1 32bit client, using Virtualbox as virtualization tool.

In summary, the user needs to:

1. Creating the virtual disk and virtual machines
1. Installing a fresh Windows 7 SP1 32 bit OS
1. Installing the Guest Agent
1. Saving and storing the VM image

## Creating the Virtual Machine
The very first step consists in creating a temporary Virtual Machine, used only to prepare the virtual disk.
This task can be achieved similarly, both in Windows and Linux.

### Windows
The first step is to create a virtual disk. So, open a command prompt and type the following commands.

```
C:\> cd \
C:\> cd "%PROGRAMFILES%\Oracle\Virtualbox"
C:\> VBoxManage createhd --filename C:\InstallAnalyzer\Disks\guest_preparation.vdi --size 25000
```

Now, we want to create a VM and attach the newly created disk to that.

```
C:\> VBoxManage createvm --name "guest_preparation" --ostype "Windows7" --register
C:\> VBoxManage storagectl "guest_preparation" --name "SATA Controller" --add sata --controller IntelAHCI
C:\> VBoxManage storageattach "guest_preparation" --storagectl "SATA Controller" --port 0 --device 0 --type hdd --medium "C:\InstallAnalyzer\Disks\guest_preparation.vdi"
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
C:\> VBoxManage modifyvm "guest_preparation" --nic1 nat --nictype1 virtio
```

At this point the virtual machine is ready to be started.

### Linux
The first step is to create a virtual disk. So, open a command prompt and type the following commands.

```
$ cd /home/ubuntu/InstallAnalyzer
$ VBoxManage createhd --filename /home/ubuntu/InstallAnalyzer/Disks/guest_preparation.vdi --size 25000
```

Now, we want to create a VM and attach the newly created disk to that.

```
$ VBoxManage createvm --name "guest_preparation" --ostype "Windows7" --register
$ VBoxManage storagectl "guest_preparation" --name "SATA Controller" --add sata --controller IntelAHCI
$ VBoxManage storageattach "guest_preparation" --storagectl "SATA Controller" --port 0 --device 0 --type hdd --medium "/home/ubuntu/InstallAnalyzer/Disks/guest_preparation.vdi"
```

In order to install Windows 7 on the machine, we will need a virtual CD/DVD drive, that will mount our installation media. Therefore:

```
$ VBoxManage storagectl "guest_preparation" --name "IDE Controller" --add ide
$ VBoxManage storageattach "guest_preparation" --storagectl "IDE Controller" --port 0 --device 0 --type dvddrive --medium /path/to/windows_7_SP1_32.iso
```

Finally, let's configure some other settings, such as ram, vram and hardware acceleration

```
$ VBoxManage modifyvm "guest_preparation" --ioapic on
$ VBoxManage modifyvm "guest_preparation" --boot1 dvd --boot2 disk --boot3 none --boot4 none
$ VBoxManage modifyvm "guest_preparation" --memory 2048 --vram 128
$ VBoxManage modifyvm "guest_preparation" --nic1 nat --nictype1 virtio
```

At this point the virtual machine is ready to be started.

## Install the Operating system
The next step consists in installing Windows 7 SP1 32 bit on the vm. To do so, simply start the VM.
To start the VM, just execute the following command from the command prompt (for both Windows and Linux):

```
...> VBoxManage startvm "guest_preparation"
```

The virtual machine will start its boot sequence. The windows installation procedure should begin shortly after the boot up. The user now needs to follow the installation wizard like in a normal machine.

Once the installation is completed, the installation wizard will ask the user to choose an username and a password. While the user can type any kind of username, it is strictly **mandatory to set no password**, so that the auto log-on is activated. Note that this system will not work in case a password is set.

The virtual machine prepared in advance is equipped with a single network adapter using a paratirtualized network interface. While most of the Linux distributions bundle a driver for the paravirtualized NIC, windows 7 does not. This means that we need to install a specific driver manually.
To do so, let's first shut down the VM we created. Then, from the host, let's download the drivers from [here](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso). Once the download is complete, we need to mount that iso into the drive of the virtual machine.
Once again, open a terminal/command prompt, navigate within the Virtualbox path, and type the following commands:

```
...> VBoxManage storageattach "guest_preparation" --storagectl "IDE Controller" --port 0 --device 0 --type dvddrive --medium /path/to/downloaded_virtio_driver.iso
```

Then start the VM again and wait for the OS to complete the bootstrap.

```
...> VBoxManage startvm "guest_preparation"
```

Once the boot is complete, open a command prompt from within the VM, and type the following command:

```
C:\> mmc devmgmt.msc
```

Navigate among the devices and look for the unrecognized Ethernet Controller under "Other devices" category. Then right click on it and chose "Update driver".
A driver installation wizard will begin. Select "Look for a driver in a specific location" and browse the D: drive, where the virtio iso has been mounted.
The system will automatically recognize and will prompt confirmation for installing the needed driver. Be sure to accept the driver installation.

At this point, we the virtual machine should also have internet connectivity. We can now proceed with the following step.

## Install the GuestAgent Bootstrapper

