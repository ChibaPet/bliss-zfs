# bliss-zfs
ZFS metapackaging for Debian

This offers an example of how one might structure ZFS packaging for an
internal repository for an organization.

The overall structure includes a "bliss-zfs" metapackage that points to a
metapackage which specifies the currently preferred version of OpenZFS.
This metapackage conflicts with kernel packaging newer than a specific
version of linux-image-amd64, so that we never install a kernel for which
we've not yet built a ZFS kmod. It also depends on the latest two ZFS kmod
packages we've built, each of which depends on its relevant kernel, the end
result being that the package system wants to keep around the latest kernel
we support and one kernel previous.

Build each metapackage with "dpkg-deb -b packagename" and populate your
local repository as desired.

When Debian releases a new kernel, create a new per-kernel kmod for each
major version of OpenZFS you want to support, update the per-major-revision
metapackage to bump the two kernels it wants and to conflict against
linux-image-amd64 newer than the new one that was just released, and your
client systems ought to gracefully update kernel and ZFS together. (Note
that if you update ZFS outside of a kernel bump, client systems will want
to run depmod manually and then rebuild initramfs files.)
