Forked Puppet Module Release Procedures
=======================================

This section will describe the release procedures for projects
for which SIMP is not the owner.

.. Important::

   If SIMP has made modifications to a project that have
   not been accepted by the owner **and** are needed by SIMP,
   the SIMP Team must create a SIMP-owned fork of the project.
   This is the only way for SIMP to release the modified version
   to `PuppetForge`_.

.. Note::

   You can identify whether a Puppet module is owned by SIMP, by
   examining the outer-most ``name`` entry in the module's
   ``metadata.json`` file.  The value for the ``name`` key will be
   of the form *<owner>*-*<module name>*.

Release to PuppetForge
----------------------

* If the owner has not released the version we desire to `PuppetForge`_,
  we must request a release from the owner.  

* If the owner will not release the version we need, our only recourse
  is to create a SIMP-owned fork of the project, create/update the,
  `.travis.yml` file for automated release and deploy from tag,
  and, as time permits, make any adjustments necessary to ensure the
  original owner's tests run.

RPM Deploy to packagecloud
--------------------------

FILL-ME-IN

#. Obtain/build the RPM

   * If the owner has already released an RPM for the version of the component
     SIMP requires, we will use that RPM.

   * Otherwise, we will

     - Obtain the official key
     - Build a signed RPM from the owner-provided GitHub release tag

#. Publish the RPM to `packagecloud`_

.. _GitHub: https://github.com
.. _packagecloud: https://packagecloud.io/simp-project
.. _PuppetForge: https://forge.puppet.com
