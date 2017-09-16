==================
Guest installation
==================

This section of the tutorial will guide the user through the preparation of a virtual disk image, which will be used as base disk for the virtual Sandbox.
Although most of the preparation steps are common to any kind of Guest, this guide targets the case of single tier configuration for a Windows 7 SP1 32bit client, using Virtualbox as virtualization tool.

In summary, the user needs to:

#. Create the virtual disk and virtual machines
#. Install a fresh Windows 7 SP1 32 bit OS
#. Install the Guest Agent
#. Save and store the VM image

Creating the Virtual Machine
----------------------------
The very first step consists in creating a temporary Virtual Machine, used only to prepare the virtual disk.
This task can be achieved similarly, both in Windows and Linux.

Windows
#######
The first step is to create a virtual disk. So, open a command prompt and type the following commands.

.. code-block:: none

    C:\> cd \
    C:\> cd "%PROGRAMFILES%\Oracle\Virtualbox"
    C:\> VBoxManage createhd --filename C:\InstallAnalyzer\Disks\guest_preparation.vdi --size 25000

Now, we want to create a VM and attach the newly created disk to that.

.. code-block:: none

    C:\> VBoxManage createvm --name "guest_preparation" --ostype "Windows7" --register
    C:\> VBoxManage storagectl "guest_preparation" --name "SATA Controller" --add sata --controller IntelAHCI
    C:\> VBoxManage storageattach "guest_preparation" --storagectl "SATA Controller" --port 0 --device 0 --type hdd --medium "C:\InstallAnalyzer\Disks\guest_preparation.vdi"

In order to install Windows 7 on the machine, we will need a virtual CD/DVD drive, that will mount our installation media. Therefore:

.. code-block:: none

    C:\> VBoxManage storagectl "guest_preparation" --name "IDE Controller" --add ide
    C:\> VBoxManage storageattach "guest_preparation" --storagectl "IDE Controller" --port 0 --device 0 --type dvddrive --medium /path/to/windows_7_SP1_32.iso

Finally, let's configure some other settings, such as ram, vram and hardware acceleration

.. code-block:: none

    C:\> VBoxManage modifyvm "guest_preparation" --ioapic on
    C:\> VBoxManage modifyvm "guest_preparation" --boot1 dvd --boot2 disk --boot3 none --boot4 none
    C:\> VBoxManage modifyvm "guest_preparation" --memory 2048 --vram 128
    C:\> VBoxManage modifyvm "guest_preparation" --nic1 nat --nictype1 virtio

At this point the virtual machine is ready to be started.

Linux
#####
The first step is to create a virtual disk. So, open a command prompt and type the following commands.

.. code-block:: none

    $ cd /home/ubuntu/InstallAnalyzer
    $ VBoxManage createhd --filename /home/ubuntu/InstallAnalyzer/Disks/guest_preparation.vdi --size 25000

Now, we want to create a VM and attach the newly created disk to that.

.. code-block:: none

    $ VBoxManage createvm --name "guest_preparation" --ostype "Windows7" --register
    $ VBoxManage storagectl "guest_preparation" --name "SATA Controller" --add sata --controller IntelAHCI
    $ VBoxManage storageattach "guest_preparation" --storagectl "SATA Controller" --port 0 --device 0 --type hdd --medium "/home/ubuntu/InstallAnalyzer/Disks/guest_preparation.vdi"

In order to install Windows 7 on the machine, we will need a virtual CD/DVD drive, that will mount our installation media. Therefore:

.. code-block:: none

    $ VBoxManage storagectl "guest_preparation" --name "IDE Controller" --add ide
    $ VBoxManage storageattach "guest_preparation" --storagectl "IDE Controller" --port 0 --device 0 --type dvddrive --medium /path/to/windows_7_SP1_32.iso

Finally, let's configure some other settings, such as ram, vram and hardware acceleration

.. code-block:: none

    $ VBoxManage modifyvm "guest_preparation" --ioapic on
    $ VBoxManage modifyvm "guest_preparation" --boot1 dvd --boot2 disk --boot3 none --boot4 none
    $ VBoxManage modifyvm "guest_preparation" --memory 2048 --vram 128
    $ VBoxManage modifyvm "guest_preparation" --nic1 nat --nictype1 virtio

At this point the virtual machine is ready to be started.

Install the Operating system
----------------------------
The next step consists in installing Windows 7 SP1 32 bit on the vm. To do so, simply start the VM.
To start the VM, just execute the following command from the command prompt (for both Windows and Linux):

.. code-block:: none

    ...> VBoxManage startvm "guest_preparation"

The virtual machine will start its boot sequence. The windows installation procedure should begin shortly after the boot up. The user now needs to follow the installation wizard like in a normal machine.

Once the installation is completed, the installation wizard will prompt the user to choose an username and a password. While the user can type any kind of username, it is strictly **mandatory to set no password**, so that the auto log-on is activated. Note that this system will not work in case a password is set.

The virtual machine prepared in advance is equipped with a single network adapter using a paratirtualized network interface.
While most of the Linux distributions do include a driver for the paravirtualized NIC, windows 7 does not.
This means that we need to install a specific driver, manually.
To do so, let's first shut down the VM we created.
Then, from the host, let's download the drivers `from here <https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso>`_.
Once the download is complete, we need to mount that iso into the drive of the virtual machine.
Once again, open a terminal/command prompt, navigate within the Virtualbox path, and type the following commands:

.. code-block:: none

    ...> VBoxManage storageattach "guest_preparation" --storagectl "IDE Controller" --port 0 --device 0 --type dvddrive --medium /path/to/downloaded_virtio_driver.iso


Then start the VM again and wait for the OS to complete the bootstrap.

.. code-block:: none

    ...> VBoxManage startvm "guest_preparation"


Once the boot is complete, open a command prompt from within the VM, and type the following command:

.. code-block:: none

    C:\> mmc devmgmt.msc


Navigate among the devices and look for the unrecognized Ethernet Controller under "Other devices" category.
Then right click on it and chose "Update driver".
A driver installation wizard will begin.
Select "Look for a driver in a specific location" and browse the D: drive, where the virtio iso has been mounted.
The system will automatically recognize and will prompt confirmation for installing the needed driver. Be sure to accept the driver installation.

Windows update and activation
-----------------------------
At this stage the Guest is capable of surfing the Internet via its own virtual adapter (which is natted behind the network of the host).
Therefore, the user can now activate the operating system. We also suggest to update the system via Windows Update. Once the updates have been correctly installed, it is strongly advised to disable windows automatic updates.
Doing so, the system will not try to update the OS during the analysis.
On the contrary, in case automatic updates are left enabled, there is a chance that they will trigger during the analysis, impacting on performance and potentially biasing sniffed network traffic.

Once the system has been correctly activated and updates have been performed, the user can then proceed with the installation of the GuestAgent bootstrapper.

Install .NET 4.0, VC Redist 2013 and VC Redist 2015
###################################################
Before installing the GuestAgent Bootstrapper, the system must comply with some software package dependency.
In particular, the agents rely on both Visual Studio 2013/2015 Visual C++ redistributable packages and .NET 4.0.

Thus, download and install the 32bit version of the .NET framework from the Microsoft website. 
The correct version of the .NET framework can be `found here <https://www.microsoft.com/en-us/download/details.aspx?id=17718>`_.

Then, do the same for the VC++ 2013 and VC++ 2015 Redistributable Packages. 
The VC++ 2013 32 bit version `is available here <http://download.microsoft.com/download/0/5/6/056dcda9-d667-4e27-8001-8a0c6971d6b1/vcredist_x86.exe>`_.
The VC++ 2015 32 bit version `is available her <https://download.microsoft.com/download/6/A/A/6AA4EDFF-645B-48C5-81CC-ED5963AEAD48/vc_redist.x86.exe>`_.

**Note:**: It may happens that those software packages fail the installation when the system is not updated. In case of installation failure, we do recommend to update the system and perform the installation once again.

Install the GuestAgent Bootstrapper
-----------------------------------
From within the Virtual Machine, open a browser and `navigate to this URL <https://github.com/albertogeniola/TheThing-GuestBootstrapper/blob/master/Windows%207%20SP1%2032bit/dist/agent_setup.exe?raw=true>`_.
Then, execute the installation of the bootstrapper, by simply double clicking on it.
Then, follow the wizard to complete the installation.
The installer will take care of downloading the needed python environment, necessary dependencies and will also install the bootstrap autostart task.

To double check the bootstrapper installation, reboot the virtual machine.
Just after Windows loads up, the bootstrapper program should automatically start, complaining about *no response from any sniffer*.
If that is the case, the bootstrapper is correctly working. Now close the bootstrapper and shut down the virtual machine correctly.

Pack the image
--------------
The image is finally ready. We just need to save it to a known location and make it "immutable".
To do so, we need to shut down the virtual machine, release the disk from the storage controller, make it immutable and then remove the guest_preparation machine.

Windows
#######
Open a prompt and execute the following:

.. code-block:: none

    C:\> cd "%PROGRAMFILES%\Oracle\Virtualbox"
    C:\> vboxmanage storageattach "guest_preparation" --storagectl "SATA Controller" --port 0 --device 0 --medium none
    C:\> vboxmanage modifymedium "C:\InstallAnalyzer\Disks\guest_preparation.vdi" --type immutable --compact


Now we want to remove the virtual machine used to setup the virtual disk.

.. code-block:: none

    C:\> vboxmanage unregistervm "guest_preparation" --delete

Finally, make the virtual disk readonly. This step is not mandatory. However it is recommended toi make the disk readonly so that there is no change for Virtualbox to write on it.

.. code-block:: none

    C:\> attrib +R "C:\InstallAnalyzer\Disks\guest_preparation.vdi"

Linux
#####
Similarly to the windows commands, let's open a terminal and release the disk from the virtual machine.

.. code-block:: none

    $ vboxmanage storageattach "guest_preparation" --storagectl "SATA Controller" --port 0 --device 0 --medium none
    $ vboxmanage modifymedium /home/ubuntu/InstallAnalyzer/Disks/guest_preparation.vdi" --type immutable --compact

Then remove the virtual machine:

.. code-block:: none

    $ vboxmanage unregistervm "guest_preparation" --delete


Finally, make the virtual disk readonly. This step is not mandatory. However it is recommended toi make the disk readonly so that there is no change for Virtualbox to write on it.

.. code-block:: none

    $ chmod 555 "/home/ubuntu/InstallAnalyzer/Disks/guest_preparation.vdi"
