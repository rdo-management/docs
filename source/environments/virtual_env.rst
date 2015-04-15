Virtual Environment
===================

RDO-Manager can be deployed in a virtual environment using virtual
machines instead of actual baremetal. However, one baremetal machine is still
needed to act as the host for the virtual machines.


Minimum System Requirements
---------------------------
To deploy a minimal RDO cloud with RDO-Manager you need the following virtual
machines:

* 1x Undercloud VM (where RDO-Manager lives)
* 1x Overcloud Controller
* 1x Overcloud Compute

Each virtual machine must consist of at least 4GB of memory and 40GB of disk
space [#]_.

The baremetal machine must meet the following minimum system requirements:

* Virtualization hardware extenstions enabled (nested KVM is not supported)
* 1 quad core CPU
* 12GB free memory
* 120GB disk space

For minimal **HA (high availability)** deployment you need at least 3x Overcloud
Controllers which increases the minimum system requirements up to:

* 20GB free memory
* 200GB disk space.


Preparing the Virtual Environment
----------------------------------
For simplified automated way of how to setup virtual environment follow
Instack's documentation: `Preparing Virtual Environment
<https://repos.fedorapeople.org/repos/openstack-m/instack-undercloud/
internal-html/virt-setup.html>`_

TBD


.. rubric:: Footnotes

.. [#] The virtual machine disk files are thinly provisioned and will not take
       up the full space initially.

