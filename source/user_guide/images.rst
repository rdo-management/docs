Images
======

<TODO - This site needs more information>


.. _building_images:


Building images
---------------
The built images will automatically have the same base OS as the
running undercloud. To choose a different OS:

    * CentOS: ``export NODE_DIST=centos7`` and switch to `CentOS documentation <../../centos/user_guide/images.html#building-images>`_.
    * RHEL 7.1: ``export NODE_DIST=rhel7`` and switch to `RHEL documentation <../../rhel/user_guide/images.html#building-images>`_.


.. only:: CentOS

   Currently you have selected **CentOS** distribution.


.. only:: RHEL

   Currently you have selected **RHEL** distribution.

   * Download the RHEL 7.1 cloud image or copy it over from a different location, for example:
     https://access.redhat.com/downloads/content/69/ver=/rhel---7/7.1/x86_64/product-downloads,
     and define the needed environment variables for RHEL 7.1::

         export DIB_LOCAL_IMAGE=rhel-guest-image-7.1-20150224.0.x86_64.qcow2
         export REG_METHOD=portal
         export REG_USER="[your username]"
         export REG_PASSWORD="[your password]"
         export REG_POOL_ID="[pool id]"   # Find this with `sudo subscription-manager list --available`
         export REG_REPOS="rhel-7-server-rpms rhel-7-server-extras-rpms rhel-ha-for-rhel-7-server-rpms \
         rhel-7-server-optional-rpms rhel-7-server-openstack-6.0-rpms"


Build images::

  instack-build-images


.. note::

   If you followed Getting Started flow, after images are built :ref:`go back to the flow<getting_started_create_images>`.
