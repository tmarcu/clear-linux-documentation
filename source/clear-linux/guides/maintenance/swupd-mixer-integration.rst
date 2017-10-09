.. _swupd_mixer_integration:

Adding custom packages and bundles
##################################



Prerequisites
=============

To use the new swupd mixer feature, you'll need a recent image of Clear
Linux OS for Intel Architecture with the following bundle installed. If you
don't have it already, you can add it with the :command:`swupd bundle-add`
command like so::

  # swupd bundle-add mixer

What is it
================
The software updater now has native integration with the mixer tool. One of the
highest requests for Clear Linux OS for Intel Architecture was the ability for
users to add their own custom content to their OS. Mixer initially enabled this,
but it required users to spin their own full OS, and be the OSV for it making
all the releases and update content.

With this new feature, users can now add custom content ``without`` having to
compose the entire OS themselves. For example, a user may want a package that
does not exist in our OS yet. Creating an entire OS just to have a single package
is doable, but not the correct way to solve the problem. Instead, users can now
add custom packages and bundles using the :file:`add-pkg.sh` script, and simply
calling :command:`swupd update && swupd bundle add <new_bundle>` to add just
their new content. The regular upstream content will still be pulled from the
normal URL, while local content will be added from a local update store. *Note*
that while this feature is intended to enable additive, unique content, it is
still possible to add non-unique packages, i.e replacing something already provided
by upstream with a local version. However that is not guaranteed to always work, thus
the updater will warn the user if conflicts are found before creating the local content.

How it works
============
The software updater now checks to see if there is any content in :file:`/usr/share/mix`.
If content does exist, then it knows user content must be incorporated into the updater
commands. Internally it does all the same operations to retrieve the upstream manifests
and content, and will also do the same for the local content. Upstream Manifest.MoMs
are still signature verified, so users are guaranteed all content not provided by their
local system is secure and correct.

When a user calls :file:`add-pkg.sh`, the script performs all the necessary mixer
commands and bookeeping to create a minimal mix. This mix contains only the new user
bundle, and os-core (to keep track of the versioning and timestamp changes). To keep
swupd from becoming too complex and multiplexing seperate Manifest.MoMs, the script also
merges the upstream Manifest.MoM (once it is openssl verified), with the local Manifest.MoM.
The result is a merged Manifest.MoM that contains any updates from upstream, with the
local updates concantenated to it. The new merged manifest is then signed with the user's
certificate, which swupd will use to verify the manifest is the one it should be.

Consuming only one Manifest.MoM relieves some burden from the updater, which now can
simply read it, and understand if a bundle originates from upstream, or from local. If
a maniest is tagged as local in the Manifest.MoM, the updater will load that manifest
from disc instead of going over the network, and all files from it will be marked as
local as well. Thus when the file list is traversed for downloading, it "downloads"
the local content directly from disc, while pulling the upstream changins from their
regular source. All of this is done without any extra flags or configs needed, so users
can expect swupd to operate the same as before, but now support multiple sources for
content.

While the version of the OS changes, it only does so by multiplying by 1000. This is
so swupd can still easily determine the base OS version, for example 18220 becomes
18220000, showing the base version ``18220`` and allowing for ``100`` new "mixes" or
package adds within that version (increment by 10 x 100). By doing so, swupd can still
check if there is a version > 18220, and if so, it will automatically generate the
new Manifest.MoM for the user on the next update operation. This means autoupdate will
still work, and users will continue recieving security updates per normal update
schedule. On top of this, a mixed file will exist at :file:``/usr/share/defaults/swupd/mixed``
signifying that the system is on an augmented version.

How to use it
=============
Using the new feature is simple. A new folder will exist, :file:``/usr/share/mix``, which
is where the local update content will go. In that folder, simply :command:``cp`` your
RPMs into the rpms folder.

#. **Adding RPMs to local folder**
Copy files into correct folder::
      # sudo cp <RPMs you want to add> /usr/share/mix/rpms/

   This is where the add-pkg.sh script looks to for its workspace.

#. **Add Rpms and create content**
run::
      # sudo add-pkg.sh <pkg_name> <bundle_name>

   This automatically creates the <bundle_name> bundle if it does not exist,
   adds the RPM name to it, and generates all the mixer content and Manifests
   needed.

#. **Migrate to local setup**. Now that the content exists, the easiest way to have swupd
   become aware of it and use it, is to use the new :command:``--migrate`` flag for an update.
run::
   # sudo swupd update --migrate

   This will update your system to the mix ready state using the new Manifest.MoM.

#. **Add your bundle to your system** To add your bundle, simply run::

   # sudo swupd bundle-add <my new bundle>



Getting back to upstream
-------------------------

You may wish to go back to official upstream with no user content. This can easily be done by
calling :command:``swupd verify --fix --force -m <upstream version> -C /usr/share/clear/update-ca/Swupd_Root.pem``, which will verify fix the system back to the upstream version
you're augmenting with local content. Fro here on regular swupd commands will only reach out
to the upstream server, since /usr/share/mix will no longer exist.
