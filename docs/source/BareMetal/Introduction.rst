.. _multitier_baremetal:

Introduction
============
Most of the Malware Sandbox Analysis System freely available nowadays only rely on virtualization for performing the analysis.
In fact, any virtualization system facilitates the analysis by implementing snapshots and rollback functions, which are particularly useful to revert the status of the sandbox after every analysis.
Moreover, most hypervisors also export a series of APIs which can be used to automate the control the sandboxes.
In the end, virtualization also optimizes the hardware usage: a powerful machine can host many small virtual sandboxes.

On the other hand, binary analysis performed on bare metal nodes becomes relevant in some circumstances.
For instance, bare metal hosts do not suffer from hypervisor detection which is sometimes used by evasive malware.
In particular, bare metal analysis can be used as a comparision meter when compared with analysis happening on a virtual system.
This represents an interesting use-case that justifies the need of bare-metal sandboxing.

TheThing has been developed to support bare metal guests from the very beginning.
Moreover, bare metal support has been developed in respect with both scalability and performance.
In particular, TheThing takes full advantage of diskless boot together with iSCSI protocol in order to implement *centralized image management* and *fast rollback*.
