# bliss-zfs
ZFS metapackaging for Debian

This offers an example of how one might structure ZFS packaging for an
internal repository for an organization. It describes metapackages that
help make use of kmods built using the upstream OpenZFS instructions here:

```
https://openzfs.github.io/openzfs-docs/Developer%20Resources/Custom%20Packages.html
```

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

Note that I build the upstream "custom" packages in LXC containers, and
hence avoid blindly building against the running kernel. You can achieve
this as follows:

```
MYKERNEL=5.10.0-23-amd64
./configure \
    --with-linux=/lib/modules/$MYKERNEL/source \
    --with-linux-obj=/lib/modules/$MYKERNEL/build
make -j6 pkg-utils deb-kmod
```

It'd pull bliss-zfs from main, and that would reference the appropriate
files in (in this case) the openzfs2.1 component.

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
