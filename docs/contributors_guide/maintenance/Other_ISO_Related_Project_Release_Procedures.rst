Other ISO-Related Project Release Procedures
============================================

This section will describe the release procedures we use for the
miscellaneous, non-Puppet-module components required to build a
SIMP ISO.  Specifically, it describes the per-component procedures
to create a `GitHub`_ release and then deploy that release to
`packagecloud`_.  The relevant components include

* ``rubygem-simp-cli``
* ``simp-adapter``
* ``simp-doc``
* ``simp-environment``
* ``simp-gpgkeys``
* ``simp-rsync``
* ``simp-utils``

Pre-Release Checklist
---------------------

The bulk of the work to release each component is to verify that the
component is ready for release.  Below is the list of verifications
that must be executed before proceeding with the release.  If any
of these steps fail, the problem identified must be fixed before
you can proceed with the tag and release steps.

#. Verify a new release is warranted and the version has been properly
   updated.  For this example, we will use the ``simp-adapter`` project.

   * Clone the component repository and checkout the development
     branch to be tagged

     .. code-block:: bash

        git clone https://github.com/simp/simp-adapter.git
        cd simp-adapter
        git checkout master # this step isn't needed for master branch

   * FIXME  The existing rake task ``compare_latest_tag`` was written
     for Puppet modules, and may not yet work here.  So, these
     instructions assume you have to compare manually the development
     branch with the last release tag.

     .. code-block:: bash

        git fetch -t origin

        # manually figure out which is last tag

        git diff tags/<last release tag> --name-only

        # manually verify mission-impacting changes have been
        # made (i.e., changes that warrant a release) and the
        # version has been updated in the CHANGELOG, version.rb
        # and/or build/<component>.spec file.

#. Verify the changelog information is available and can be
   extracted

   * FIXME  The existing rake task ``changelog_annotation`` was written
     for Puppet modules, and may not yet work here.  So, these
     instructions assume you have to visually inspect the appropriate
     file (CHANGELOG or %changelog section of <component>.spec file)

   * FIXME ``simp-doc`` requires Changelog.rst from simp-core to be
     current.  Perhaps it makes more sense to do this tag and release
     along with the SIMP ISO release?  At a minimum, the Changelog.rst
     on the ``simp-doc`` master branch must be correct.

#. Verify RPMs can be created for this component

   .. code-block:: bash

      bundle update
      bundle exec rake pkg:rpm[epel-6-x86_64]
      bundle exec rake pkg:rpm[epel-7-x86_64]

#. Verify the component's unit test have succeeded

   * Navigate to the project's TravisCI results page and verify the
     tests for the development branch to be tagged and released have
     passed.  For our project, this page is
     https://travis-ci.org/simp/simp-adapter/branches

     .. IMPORTANT::

        If the tests in TravisCI fail, you **must** fix them before
        proceeding.  The automated release procedures will only
        succeed, if the unit tests succeed in TravisCI.

#. Verify the component's acceptance tests have succeeded

   * Run the appropriate acceptance test rake task, if it exists.
     For this project, ``rake beaker:suites`` is the appropriate task

     .. code-block:: bash

       bundle exec rake beaker:suites

     .. NOTE::

        * If the GitLab instance for the project is configured and
          current (it is sync'd every 3 hours), you can look at
          the latest acceptance test results run by GitLab.  For
          our project, the results would be at
          https://gitlab.com/simp/rubygem-simp-rake-helpers/pipelines.

#. Verify the RPM for this component can be used to upgrade the last
   full SIMP release and interoperates with it.  For both CentOS 6
   and CentOS 7, do the following:

   * Bring up a CentOS server that was kicked from the appropriate SIMP
     ISO and for which ``simp config`` and ``simp bootstrap`` has been
     run.

     .. NOTE::

        The `simp-packer`_ project is the easiest way to create a SIMP
        VM that has been bootstrapped.

   * Copy the component RPM generated from the above RPM verification
     step to the server and install with yum.  For example,

     .. code-block:: bash

       sudo yum install simp-adapter-0.0.3-0.el7.noarch.rpm

     .. NOTE::

        If the component requires updated dependencies, those RPMs will
        have to be built and installed at the same time.

   * Verify the ``puppet agent`` runs succeed on the Puppet master
     and client.  On each server

     .. code-block:: bash

        #login as root
        puppet agent -t

#. Verify the RPM for this component can be used to create SIMP ISOs
   for CentoOS 6 and CentOS 7, each of which can be configured via
   ``simp config`` and bootstrapped via ``simp bootstrap``.  For
   CentOS 6 and CentOS 7:

   * Checkout the ``simp-core`` project for the last SIMP release.
     For this discussion, we will assume it is ``6.0.0-0``.

     .. code-block:: bash

        git clone https://github.com/simp/simp-core.git``
        cd simp-core
        git fetch -t origin
        git checkout tags/6.0.0-0

   * Create a ``Puppetfile.tracking`` file that contains the contents
     of ``Puppetfile.stable`` in which the URLs for the component and
     any of its updated dependencies have been updated to reference
     the versions under test.

   * Build each ISO for CentOS 6 and CentOS 7.  For example

    .. code-block:: bash

       PUPPET_VERSION="~> 4.8.2" \
       SIMP_BUILD_verbose=yes \
       SIMP_PKG_verbose=yes \
       SIMP_BUILD_distro=CentOS/7/x86 _64 \
       bundle exec rake build:auto[/net/ISO/Distribution_ISOs]

    .. IMPORTANT::
       The most reliable way to build each ISO is from a clean
       checkout of ``simp-core``.

    * Use `simp-packer` to verify the SIMP ISO can be bootstrapped

Release to GitHub
-----------------

FIXME.

Only rubygem-simp-cli is setup of to release to GitHub, when an
annotated tag is pushed to its GitHub project *and* TraviCI succeeds.
Need to fix the remaining assets and then update this description.

Each SIMP ISO-related project is configured to automatically create a
`GitHub`_ release, when an annotated tag is created for the `GitHub`_ 
project **and** the TravisCI tests for the annotated tag push succeed.
To create the annotated tag:

#. Clone the component repository and checkout the development
   branch to be tagged

   .. code-block:: bash

      git clone git@github.com:simp/simp-adapter.git
      cd simp-adapter
      git checkout master # this step isn't needed for master branch

#. Generate the changelog content

   * FIXME Extract the changelog content from the ``CHANGELOG.md``,
     ``CHANGELOG``, or ``build/<component>.spec`` file

   .. code-block:: bash

      vim foo

#. Create the annotated tag.  In this example the content of 'foo' is::

      Release of 0.0.4

      * Removed packaged auth.conf in favor of managing it with Puppet

   .. code-block:: bash

      git tag -a 0.0.4 -F foo
      git push origin 0.0.4

   .. NOTE::

      For markdown-style changelogs, you will need to specify
      ``--cleanup=whitespace`` so comment headers are not stripped.

#. Verify TravisCi completes successfully

   .. IMPORTANT::
      If any of the required TravisCI builds for the project fail, for
      example due to intermittent connectivity problems with `GitHub`_,
      you can complete the release process by manually restarting the
      failed build on the Travis page for that build.

#. Verify release exists on `GitHub`_.  This release will have been created by
   ``simp-auto``.

Build Signed RPM and Deploy to packagecloud
--------------------------------------------

FILL-ME-IN

* Obtain the official key
* Build each signed RPM from its release tag using the official key
* Publish each signed RPM to packagecloud

.. _GitHub: https://github.com
.. _packagecloud: https://packagecloud.io/simp-project
.. _simp-packer: https://github.com/simp/simp-packer
.. _`RPM spec file template`: https://raw.githubusercontent.com/simp/rubygem-simp-rake-helpers/master/lib/simp/rake/helpers/assets/rpm_spec/simpdefault.spec
