

...



# Prepare the server

## Install ISCSI target
Our setup is based on an iSCSI service on the top of Windows Server 2012 R2.
The very first step of the tutorial consists in installing the iSCSI File Target service on the server.
This can be done via the ServerManager, by selecting "Install server role" and specifying the following role:

File & ISCSI Services / ISCSI Target Server

<IP>
<IQN>
base_disk