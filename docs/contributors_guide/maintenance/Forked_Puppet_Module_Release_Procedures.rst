Forked Puppet Module Release Procedures
=======================================

FILL-ME-IN

* Procedures only apply if SIMP is not the owner and is using
  owner-provided GitHub releases.

  .. Important::

    If SIMP has made modifications to a project that both have
    not been accepted by the owner **and** are needed by SIMP,
    the SIMP Team must create a SIMP-owned fork of the project
    and then follow SIMP-owned project release procedures. SIMP
    cannot release its own version of this project PuppetForge
    otherwise.

* Shouldn't need to tag, as we should be using owner-provided tags

* Shouldn't publish to PuppetForge, since we aren't the project 
  owner. Instead, if the version we desire is not already in
  PuppetForge, we will have to ask owner to publish it.

* If signed RPM is not available from the source

  - Obtain official key
  - Build signed RPM from the owner-provided GitHub release tag
  - Publish to packagecloud
