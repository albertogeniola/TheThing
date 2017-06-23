Welcome to TheThing's documentation!
====================================
In short, TheThing is an automated analysis sandbox, mainly oriented at analyzing freeware installers on Windows platform.
As such, its main goal is to automate the installation procedure of software, while collecting information about the underlying changes happening to the underlying system.
During each analysis the system monitors the disk, registry and network, and keeps track of every action performed by the software being analyzed.
Therefore, a report is produced, summarizing all the information collected during the analysis, together with installation screenshots and a description of interactions performed against the installer user interface (if any).

If you are familiar with malware sandbox analysis system (MSAS), you might think this software applies similar principles to a complete different problem, which has not been addressed by any other MSAS at the moment.
In particular, the strength of this software stands in the combination of UI automation and dynamic analysis of software.
Such approach is particularly relevant when dealing with **Potentially Unwated Applications** (PUP) installers.
This class of installers do indeed require human interaction in order to trigger the PUP installation.
In fact, classic MSAS tend to fail the analysis of such PUP droppers because they are not able to click through the installation procedure.

TheThing was developed precisely to offer a valid tool for collecting information that can be later used for discriminating PUP droppers from clean software.

.. toctree::
   :maxdepth: 2
   :caption: Contents:

   Introduction
   Virtualbox/Introduction
   Openstack/Introduction


Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

