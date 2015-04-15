.. _working_with_overcloud:

Connecting to RDO (Overcloud)
=============================

Instack type of deployment generates an overcloudrc file appropriate for
**interacting with the deployed overcloud** in the current user's home
directory. To use it, simply source the file::

    source ~/overcloudrc


To **use RDO**, please follow `RDO documentation
<https://www.rdoproject.org/Docs>`_.


.. note::

   To return to **working with the undercloud**, source the stackrc file again::

       source ~/stackrc
