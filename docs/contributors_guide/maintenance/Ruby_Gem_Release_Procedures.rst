Ruby Gem Release Procedures
===========================

This section will decribe the release procedures for the SIMP Ruby gems
used to build and test SIMP components. Specifically, it describes the
per-component procedures to create a `GitHub`_ release and then deploy
that release to `RubyGems.org`_.  The relevant components include

* ``rubygem-simp-beaker-helpers``
* ``rubygem-simp-rake-helpers``
* ``rubygem-simp-rspec-puppet-facts``

Pre-Release Checklist
---------------------

The bulk of the work to release a component is to verify that the
component is ready for release.  Below is the list of verifications
that must be executed before proceeding with the release.  If any
of these steps fail, the problem identified must be fixed before
you can proceed with the tag and release steps.

#. Verify a new release is warranted and the version has been properly
   updated.  For this example, we will use the ``rubygem-simp-rake-helpers`` project.

   * Clone the component repository and checkout the development
     branch to be tagged

     .. code-block:: bash

        git clone https://github.com/simp/rubygem-simp-rake-helpers.git
        cd rubygem-simp-rake-helpers
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
        # version has been updated in the CHANGELOG.md, version.rb
        # and/or TBD file.

#. Verify the changelog information is available and can be
   extracted

   * FIXME  The existing rake task ``changelog_annotation`` was written
     for Puppet modules, and may not yet work here.  So, these
     instructions assume you have to visually inspect the appropriate
     file (e.g., CHANGELOG.md).

#. Verify the component's dependencies are correct in the ``Gemfile``
   (Visual inspection)

#. Verify a Ruby gem can be created for this component

   .. code-block:: bash

      bundle update
      bundle exec rake pkg:gem

#. Verify the component's unit test have succeeded

   * Navigate to the project's TravisCI results page and verify the
     tests for the development branch to be tagged and released have
     passed.  For our project, this page is
     https://travis-ci.org/simp/rubygem-simp-rake-helpers/branches

     .. IMPORTANT::

        If the tests in TravisCI fail, you **must** fix them before
        proceeding.  The automated release procedures will only
        succeed, if the unit tests succeed in TravisCI.

#. Verify the component's acceptance tests have succeeded

   * Run the appropriate acceptance test rake task, if it exists.
     For this project, ``rake -T`` shows that ``rake acceptance``
     is the appropriate task

     .. code-block:: bash

       bundle exec rake acceptance

     .. NOTE::

        * If the GitLab instance for the project is configured and
          current (it is sync'd every 3 hours), you can look at
          the latest acceptance test results run by GitLab.  For
          our project, the results would be at
          https://gitlab.com/simp/rubygem-simp-rake-helpers/pipelines.

#. Verify SIMP components can use this gem to build and test
   tasks. 

   * Install the gem you just built, locally.  

     .. code-block:: bash

        rvm all do gem install dist/simp-rake-helpers-4.0.1.gem

   * Download the latest versions of most of the SIMP components using
     the ``simp-core`` project.

     .. code-block:: bash

        git clone https://github.com/simp/simp-core.git``
        cd simp-core
        bundle update
        bundle exec rake deps:checkout

   * If the major version number for the gem has increased, for the
     following projects, update the Gemfile to permit the newer version

     - All projects in ``src/assets/``
     - All projects in ``src/rsync``
     - All projects in ``src/rubygems/``
     - All SIMP-owned projects in ``src/puppet/modules/``

   * In each project listed above, execute

     .. code-block:: bash

        bundle update
        bundle exec rake spec
        bundle exec rake beaker:suites || bundle exec rake acceptance


Release To GitHub and Deploy to RubyGems.org
--------------------------------------------

Now, for the slightly easier part.  Each SIMP Ruby gem is configured
to automatically create a `GitHub`_ release and push the release to
`RubyGems.org`_, when an annotated tag is created for the `GitHub`_ 
project
**and** the TravisCI tests for the annotated tag push succeed.
To create the annotated tag:

#. Clone the component repository and checkout the development
   branch to be tagged

   .. code-block:: bash

      git clone git@github.com:simp/rubygem-simp-rake-helpers.git
      cd rubygem-simp-rake-helpers
      git checkout master # this step isn't needed for master branch

#. Generate the changelog content

   * FIXME Extract the changelog content from the CHANGELOG.md

   .. code-block:: bash

      vim foo

#. Create the annotated tag.  In this example the content of 'foo' is::

      Release of 4.0.1

      * Reverted the bundler pinning since it was causing too many issues on CI
        systems

   .. code-block:: bash

      git tag -a 4.0.1 -F foo
      git push origin 4.0.1

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

#. Verify release exists on `RubyGems.org`_. 

.. _GitHub: https://github.com
.. _RubyGems.org: https://rubygems.org/
