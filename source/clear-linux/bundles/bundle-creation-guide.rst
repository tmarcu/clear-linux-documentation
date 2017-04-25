.. _bundle-creation-guide:

Bundle creation guide
#####################

Bundles provide a set of functionality to the system independently of the
number of upstream open source projects needed for this functionality. Visit
our :ref:`bundles overview<bundles_overview>` for more information regarding
bundles.

Bundle goals
============

* Add simplicity for users.
* Separate dependency sets to provide optimal and minimal updates.
* Separate the bundles' functionality cleanly and without overlaps.

Assumptions
===========

Users, generally, are not bothered by unused applications on their system.

Users prefer to install fewer bundles, therefore, having sets of bundles with
more generic functionality for use cases such as system administration or
network modification makes sense.


Bundles rules
=============

Creating a bundle
-----------------

#. Bundles must encapsulate functionality, not content. For example, the
   ``containers-basic`` bundle provides containers functionality for running
   container images from DockerHub\*. The bundle is not specific to Docker\*,
   rkt\*, or any other container manager.

#. Bundles must have a size appropriate to their features. The bundle size is
   tracked and bugs opened when the size exceeds a certain value. Bundles
   should avoid splitting for the sake of size reduction. Only real feature
   splits should lead to bundle splitting. There is a case for splitting to
   do grouping though:

.. note::
   To improve the swupd behavior, there is a subset of infrastructure bundles
   which are not required to be functional, see the `libX11client` bundle as
   an example.

#. Bundles must be created with one of these three types:
   * ${name}-basic - Bundles which enable a basic use case.
   * ${name}-extras - Bundles which extend a use case.
   * ${name}-dev - Bundles which contain tooling and libraries needed to
     build any package a basic bundle contains. These bundles are auto-
     generated.

#. Bundles must only contain packages. Any runtime dependencies are
   automatically added when the build reads the spec file as RPM
   dependencies.

#. Bundles can be used to provide functionality for a particular programming
   language. Examples: `c-basic`, `go-basic`, `ruby-basic`, `perl-basic`,
   `python- basic`, `R-basic`, `java-basic`.

#. Bundles must include sensible package levels. Generally, sub-packages
   should not be part of a bundle. The bundle maintainer has final say on
   whether a sub-package can be included in a bundle.

#. Packages can exist in more than one bundle.

#. Bundles may not have circular dependencies. For example, if bundleA includes
   bundleB, bundleB cannot include bundleA.

#. Bundles must not reference undefined bundles or packages.

#. Bundles must not be empty (they must either include or directly reference
   packages).

#. Bundles have four possible statuses:
   * *WIP*: Bundles available for installation but with partial functionality.
   * *Active*: Bundles available for installation with their entire
     functionality enabled.
   * *Deprecated*: Bundles no longer available for installation.
   * *Deleting*: Bundles will be removed in the next format bump.
   * *Infrastructure*: Bundles without functionality on their own.
   Only *Active* bundles are displayed in swupd search.

#. A new bundle's documentation must provide the functionality it includes as
   well as the use case it serves. The documentation requires two approvals:

   #. The documentation team must ensure the documentation is clear,
      consistent, and can be properly published.

   #. The QA team must create BBT tests based on the information provided in
      the documentation.

#. New bundles are approved by the bundle maintainer (William Douglas)

Changing a bundle
-----------------

#. All changes to a bundle must involve the owner of the specific bundle and
   the bundle maintainer.

#. Removing a bundle or a functionality from a bundle requires a format bump
   and has the following prerequisites:
   * Update the BBT tests with the changed bundles
   * The split off functionality must be available in alternate bundles
     (unless the function is no longer supported in |CL|).

#. Bundles can be removed completely but it must be a management decision.

#. To remove a functionality from a bundle, the replacement functionality
   must added to a bundle included in the bundle which lost the
   functionality.

#. Functionality can be removed entirely but it must be a management decision.

#. Any bundle changes must be accompanied by the appropriate documentation
   updates. The documentation changes require two approvals:

   #. The documentation team must ensure the documentation is clear,
      consistent, and can be properly published.

   #. The QA team must create BBT tests based on the information provided in
      the documentation.

#. When modifying a bundle, consider whether the functionality is new or an
   extension. If it is an extension, consider whether it is optional or
   mandatory.

   * For new functionality: Add a new bundle with the necessary packages.

   * For an optional extension or an additional implementation:

      * Consider adding a parallel `-extras` bundle
      * Include the functionality in another bundle which includes other
        sibling implementations
      * Include an auto-fallback to a prior implementation if the
        prerequisites aren't met, for example `containers-virt`.

   * For mandatory extensions or replacement implementations: Replace the
     packages of the existing bundle with the mandatory packages or
     replacement packages.

   .. note::
      Modifying bundles is a particularly tricky area with complex trade offs
      related to: bundle size, user expectations, and lack of telemetry
      visibility into what users actually run.

Testing a bundle
----------------

#. The BBT project is where bundle tests should reside.

#. While there is not enough tracking from telemetry to know much about how
   Clear Linux bundles are used, the distribution is at a point where it
   should not break end user functions.

#. Tagging criteria for bundle changes: The change must not introduce BBT
   regressions (currently not documented how a change can be tested to
   validate this), it must pass review by the bundle maintainer, and it must
   not show any errors being run against the bundle-tools verify-bundles.sh
   script.

Limitations
===========

Bundle removal and renames are handled with different processes than the ones
swupd uses to handle file level changes. Before a removal or rename can
happen at the bundle level, a change to the bundle-chroot-builder and a swupd
format bump are required.

By definition, users are not able to install a single package on a |CL|
system.

One possible solution to this limitation is our :ref:`mixer_tool`.

TODO
====

Examples: It would be helpful to include at least one example of how to
create a bundle: identifying the packages required for the main and
corresponding "-dev" package. testing the bundle (install / upgrade
scenarios). getting the bundle published.
