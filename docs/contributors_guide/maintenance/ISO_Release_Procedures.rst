ISO Release Procedures
======================

This section will describe the partially-automated, release procedures
we use for SIMP ISOs.  Specifically, it describes the procedures to

* Tag a ``simp-core`` project release in `GitHub`_.
* Build SIMP ISOs for CentOS 6 and CentOS 7 from that `GitHub`_ release
* Deploy the SIMP ISOs to `simp-project`_.

Pre-Release Checklist
---------------------

The bulk of the work to release both CentOS 6 and CentOS 7 versions of
a SIMP ISO is to verify that each ISO is ready for release. Below is
the list of verifications that must be executed *for each ISO* before
proceeding with the release of that ISO. If any of these steps fail,
the problem identified must be fixed befor you can proceed with the tag
and release steps.

#. Verify that all artifacts used to create the ISO have signed RPMs
   that have been pushed to packagecloud.   This will include:

   * SIMP-owned Puppet modules
   * Forked Puppet modules
   * Utility RPMs (rubygem-simp-cli, simp-adapter, simp-utils, etc.)
   * ``simp-doc``
   * FIXME - should this include OS RPMs?
   * FIXME - Has this been automated already using the tags in the
     (presumed) updated ``Puppetfile.stable``?

#. Verify that ``Puppetfile.stable`` file for the ``simp-core`` project
   is complete and accurate

   *  It includes all the SIMP-owed Puppet modules, other Puppet modules
      that are depenencies of SIMP-owned Puppet modules, and utilities
      to configure the SIMP system when installed from ISO.

   * The URL for each artifact corresponds to the tag for its signed,
     published RPM.

#. Verify the ``simp-core`` Changelog.rst has been updated.:

   * FIXME - Has the generation of the Changelog.rst from component
     CHANGELOGs been automated in any fashion?

#. Verify the ``simp-core`` RPM can be generated for this module

   .. code-block:: bash

      git clone https://github.com/simp/simp-core.git``
      cd simp-core/src
      bundle update
      bundle exec rake pkg:rpm[epel-6-x86_64]
      bundle exec rake pkg:rpm[epel-7-x86_64]

#. Verify the ``simp-core`` unit tests have succeeded

   * Navigate to the project's TravisCI results page and verify the
     tests for the development branch to be tagged and released have
     passed.  For our project, this page is
     https://travis-ci.org/simp/simp-core/branches

     .. IMPORTANT::

        If the tests in TravisCI fail, you **must** fix them before
        proceeding.  The automated release procedures will only
        succeed, if the unit tests succeed in TravisCI.

#. Verify the SIMP ISO can be built from the local simp-core RPM and
   RPMs pushed to packagecloud

   * FIXME - The magic happens here....Trevor's build script?

#. Verify a server kicked from the SIMP ISO can be bootstrapped

   * Use `simp-packer`_ to verify the SIMP ISO can be bootstrapped

     .. NOTE::

        The `simp-packer`_ project is the easiest way to create a SIMP
        VM that has been bootstrapped.

#. Verify with ``pupmod-simp-simp`` acceptance tests that this
   aggregation of module versions interoperate.

   * Determine the version of ``pupmod-simp-simp`` to be used in this
     SIMP ISO release.  This version can be pulled from the
     ``Puppetfile.stable``.

   * Checkout that version of the ``pupmod-simp-simp`` project.
     For this discussion, we will assume it is ``4.0.0``.

     .. code-block:: bash

        git clone https://github.com/simp/pupmod-simp-simp.git``
        cd pupmod-simp-simp
        git fetch -t origin
        git checkout tags/4.0.0

   * Create a ``.fixtures.yml`` file that sets the versions of
     each dependency to the version contained in the
     ``Puppetfile.stable`` file for this ISO release.

     .. NOTE::

        Currently, there are prototype utilities to generate the
        ``.fixtures.yml`` file for you.  When these utilities are
        released,  this documentation will be (thankfully) updated.

   * Run the acceptance tests with and without FIPS mode enabled

     .. code-block:: bash

        bundle update
        BEAKER_fips-yes bundle exec rake beaker:suites
        bundle exec rake beaker:suites

#. Verify all other major capabilities that are not otherwise tested
   in acceptance tests function as advertised

   * A client can be PXE booted using the SIMP ISO
   * FIXME What else?

#. Verify the set of RPMs in the SIMP ISO can upgrade the last full
   SIMP release.

   * Bring up a CentOS server that was kicked from the appropriate SIMP
     ISO and for which ``simp config`` and ``simp bootstrap`` has been
     run.  (Reminder: The `simp-packer`_ project is the easiest way to
     create a SIMP VM that has been bootstrapped.)

   * Copy the SIMP and system RPMs packaged in the SIMP ISO to the
     server and install with yum.

     - FIXME Should put RPMs into appropriate updates repos, run
       something like the following

       .. code-block:: bash

          cd <updates dir>
          createrepo .
          chown -R root.apache ./*
          find . -type f -exec chmod 640 {} \;
          find . -type d -exec chmod 750 {} \;
          yum clean all;
          yum make cache
          yum update

Release ``simp-core`` to GitHub
-------------------------------

``simp-core`` is configured to to automatically create a `GitHub`_ 
release, when an annotated tag is created for the `GitHub`_
project **and** the TravisCI tests for the annotated tag push succeed.
(FIXME:  ``simp-core`` .travis.yml has broken logic to push to
PuppetForge, as well)
To create the annotated tag:

#. Clone the component repository and checkout the development
   branch to be tagged

   .. code-block:: bash

      git clone git@github.com:simp/simp-core.git
      cd simp-core
      git checkout master # this step isn't needed for master branch

#. Create the annotated tag for the release.  In this example, we
   are assuming the version is ``6.1.0`` and we are using the
   full Changelog.rst content.

   .. code-block:: bash

      git tag -a 6.0.2 -F Changelog.rst --cleanup--whitespace
      git push origin 6.0.2

#. Verify TravisCi completes successfully

   .. IMPORTANT::
      If any of the required TravisCI builds for the project fail, for
      example due to intermittent connectivity problems with `GitHub`_,
      you can complete the release process by manually restarting the
      failed build on the Travis page for that build.

#. Verify release exists on `GitHub`_.  This release will have been
   created by ``simp-auto``.

Build Signed simp-core RPM and Deploy to packagecloud
-----------------------------------------------------

FILL-ME-IN

* Obtain the official key
* Build a signed ``simp-core`` RPM  from its release tag using the official key
* Publish signed RPM to `packagecloud`_


Build Final ISO and Deploy to simp-project 
------------------------------------------

FILL-ME-IN

* FIXME Re-build ISO using signed ``simp-core`` RPM
* Push to `simp-project`_

Notify Mailing List
-------------------

FILL-ME-IN

.. _GitHub: https://github.com
.. _packagecloud: https://packagecloud.io/simp-project
.. _simp-project: http://simp-project.com/ISO/SIMP
.. _simp-packer: https://github.com/simp/simp-packer
