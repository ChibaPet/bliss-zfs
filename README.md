# bliss-zfs
ZFS metapackaging for Debian

This offers an example of how one might structure ZFS packaging for an
internal repository for an organization.

This organizes OpenZFS major versions into components that live in the
local repository. There's an overall metapackage that depends on the most
recent two per-kernel metapackages, each of which depends on the ZFS
packages needed to run ZFS on a particular kernel. The top-level
metapackage also conflicts with linux-image-amd64 newer than the versions
for which we've built OpenZFS kmods. Everything but the top-level
metapackage lives in a repository component directory tied to the major
version of OpenZFS it represents.

Using the naming in this repository, your sources.list might include:

```
deb http://repo.in.my.domain/debian bullseye main openzfs2.1
```

It'd pull bliss-zfs from main, and that would reference the appropriate
files in (in this case) the openzfs2.1 component.

This repository is churning a bit on its first day given gotchas when
trying to install older versions of OpenZFS, distinguished only by package
version. The current plan splits OpenZFS major version-related packages
into their own component directories in the local repository. (None of this
was painful when only shipping a single major version of OpenZFS but this
isn't sufficiently flexible.)

Build each metapackage with "dpkg-deb -b packagename" and populate your
local repository as desired.

When Debian releases a new kernel, create a new per-kernel kmod for each
major version of OpenZFS you want to support, update the top-level
metapackage to bump the two kernels it wants, and to conflict against
linux-image-amd64 newer than the new one that was just released, and your
client systems ought to gracefully update kernel and ZFS together. (Note
that if you update ZFS outside of a kernel bump, client systems will want
to run depmod manually and then rebuild initramfs files.)

Please report any errors found. I'd like this to be a reliable mechanism
for sites that don't want to rely on DKMS across their infrastructures.
(The carbon footprint alone is a compelling reason to avoid DKMS at scale.)
