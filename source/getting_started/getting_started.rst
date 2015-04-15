.. _getting_started:

Getting Started with RDO-Manager
================================

With these few steps you will be able to simply deploy RDO to your environment
using our POC defaults in few steps.


Prepare for using RDO-Manager
-----------------------------

#. Make sure you have RDO-Manager running in your virtual environment:

   * `Preparing Virtual Environment <https://repos.fedorapeople.org/repos/
     openstack-m/instack-undercloud/html/
     virt-setup.html#preparing-the-host-machine>`_
   * :ref:`installation`

#. Log into your RDO-Manager (undercloud) machine as non-root user::

    ssh root@<rdo-manager-machine>

    su - stack

#. In order to use RDO-Manager CLI commands you need to source needed
   passwords::

    source stackrc


Get Images
----------

..
    You can simply download and use provided overcloud images to get started::

        sudo yum install -y wget
        wget --no-verbose --no-parent --recursive --level=1 --no-directories --reject 'index.html*' -P ./images/ http://file.brq.redhat.com/~jcoufal/rdo-manager/images/centos/

    .. note:: If you happen to need to build overcloud images, please follow these
       steps: :ref:`building_images`


At the moment we have to build images manually, please follow:

* :ref:`building_images`


.. _getting_started_create_images:

Now create overcloud images in Glance::

    # Save parameters so that we don't have to repeat them in following commands
    image_name="overcloud-full"
    image_location="./"

    # Create kernel/ramdisk images (use above defined variables)
    openstack image create ${image_name}-kernel --public --disk-format=aki --container-format=aki --file="${image_location}${image_name}.vmlinuz"
    openstack image create ${image_name}-ramdisk --public --disk-format=ari --container-format=ari --file="${image_location}${image_name}.initrd"

    # Save kernel and ramdisk IDs
    kernel_id=$(openstack image show ${image_name}-kernel | grep ' id ' | awk '{print $4}')
    ramdisk_id=$(openstack image show ${image_name}-ramdisk | grep ' id ' | awk '{print $4}')

    # Create overcloud-full image with associations to kernel and ramdisk images
    openstack image create overcloud-full \
      --public \
      --disk-format=qcow2 \
      --container-format=bare \
      --property kernel_id=$kernel_id \
      --property ramdisk_id=$ramdisk_id \
      --file="${image_location}${image_name}.qcow2"

    # You can check image list by `openstack image list`


Create provisioning PXE boot images (for provisioning PXE booting)::

    openstack image create bm-deploy-kernel --public --disk-format=aki --container-format=aki --file="${image_location}deploy-ramdisk-ironic.kernel"
    openstack image create bm-deploy-ramdisk --public --disk-format=ari --container-format=ari  --file="${image_location}deploy-ramdisk-ironic.initramfs"


Copy node discovery images to proper place::

    sudo cp "${image_location}discovery-ramdisk.kernel" /tftpboot/discovery.kernel
    sudo cp "${image_location}discovery-ramdisk.initramfs" /tftpboot/discovery.ramdisk


Register and Discover Nodes
---------------------------

Import nodes.json file in order to register multiple nodes at once::

    # instackenv.json file format is not supported at the moment, let's generate valid structure
    jq '.nodes' instackenv.json > nodes.json

    # Register multiple nodes
    openstack baremetal import --json nodes.json


Add extra parameters needed for node boot::

    # Requirements: "bm-deploy-kernel" and "bm-deploy-ramdisk" images are required to be in Glance
    # You can check with `openstack image list`

    openstack baremetal configure boot


Start and check discovery process of all nodes::

    openstack baremetal introspection all start
    openstack baremetal introspection all status

.. note:: **Discovery has to finish without errors.**
   The process can take up to 10 minutes. If the process takes longer, see
   `Discovery Troubleshooting (TBD) <#>`_.



Create Flavors (Node Profiles)
------------------------------

Create and setup flavors for Control, Compute and Ceph Storage roles::

    # Create flavors and set properties:

    openstack flavor create baremetal_control --id=auto --ram=4096 --disk=40 --vcpus=1
    openstack flavor set baremetal_control --property cpu_arch=x86_64 --property profile=control

    openstack flavor create baremetal_compute --id=auto --ram=4096 --disk=40 --vcpus=1
    openstack flavor set baremetal_compute --property cpu_arch=x86_64 --property profile=compute

    # Even if we don't deploy ceph, Instack's deploy script checks for the flavor
    openstack flavor create baremetal_ceph-storage --id=auto --ram=4096 --disk=40 --vcpus=1
    openstack flavor set baremetal_ceph-storage --property cpu_arch=x86_64 --property profile=ceph-storage


Deploy RDO
----------

::

    instack-deploy-overcloud


Post-deployment
---------------

Testing RDO
^^^^^^^^^^^
Download testing image::

    curl -O http://file.brq.redhat.com/~jcoufal/rdo-manager/images/test/fedora-user.qcow2

Trigger Instack testing script::

    instack-test-overcloud


Working with RDO
^^^^^^^^^^^^^^^^

Go to: :ref:`working_with_overcloud`


Redeploying RDO
^^^^^^^^^^^^^^^

The overcloud can be redeployed when desired.

#. First, delete any existing Overcloud::

    heat stack-delete overcloud

#. Confirm the Overcloud has deleted. It may take a few minutes to delete::

    # This command should show no stack once the Delete has completed
    heat stack-list

#. Although not required, discovery can be rerun. Reset the state file and then rediscover nodes::

    sudo cp /usr/libexec/os-apply-config/templates/etc/edeploy/state /etc/edeploy/state

    openstack baremetal introspection all start
    openstack baremetal introspection all status

#. Deploy the Overcloud again::

    instack-deploy-overcloud
