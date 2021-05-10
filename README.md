# Debian 'mmdebstrap' Artifacts

Just a repository to make available some artifacts produced by the `mmdebstrap`
tool, quick and easy.

> The `mmdebstrap` does not work consistently well on OSes other than Debian. Therefore, it's easier to work with a Docker container... or simply let **Github Actions** does the work.

**Download Artifacts:** https://github.com/intjelic/debian-mmdebstrap-artifacts/actions (pick the last excecuted workflow and then pick the variation of the artifact you're interested in).

The 'mmdebstrap' tool is an alternative to 'debootstrap' to create chrootable
Debian filesystems. The advantage of mmdebstrap is that it's also able to
create them for different architectures other than the host architecture.
Architectures such as `armhf` or `arm64`, which is useful given most of us are
running on the `amd64` architecture.

**What are those artifacts useful for ?**

Creating a Debian filesystem is one of the step needed in order to make images
for VM like QEMU. Those artifacts have the particularity of providing the "boot
files" (initrd and vmlinuz) that QEMU is able to use in order to boot the
Debian OS without setting up more complex boot loader. The downside is that any
update of the Debian system will produce updated versions of those files (when
a new kernel is available in Debian package repositories).

Therefore, inside the archive, you will find a structure similar to this.

```
initrd.img-4.19.0-16-armmp-lpae
vmlinuz-4.19.0-16-armmp-lpae
debian-buster-standard-armhf/
  bin/
  dev/
  etc/
  home/
[...]
```

Where the `initrd` and `vmlinuz` files are to be passed to the QEMU's `-initrd`
and `-kernel` flags. Note that you can also pass additional options to the
kernel using the `-append` flag. Consult the QEMU documentation for more
information.

**Cross-architecture chrooting**

Chrooting into the Debian filesystem before you make an image out of it is
useful if you need to make any additional configuration such as changing the
root password, creating additional system users, install more packages, set up
the network interfaces, etc.

When the host has the same architecture, it's straight-forward and it can be
accomplished with the following command.

```
sudo chroot debian-buster-standard-amd64/
```

When the host has a different architecture, `qemu-user-static` can be used
easily. Assuming you're on `amd64` and want to chroot into a `armhf` Debian
filesystem, you would the following.

```
sudo apt-get install qemu-user-static

cp /usr/bin/qemu-arm-static debian-buster-standard-armhf/usr/bin/
sudo chroot debian-buster-standard-armhf/ qemu-arm-static /bin/bash
```

After you're done, you probably want to delete `qemu-arm-static` with
`sudo rm debian-buster-standard-armhf/usr/bin/qemu-arm-static`.

Note that you should not rely on information provided by the `uname` command.

**Fork and adjust to your needs**

All the commands that are executed in order to create those artifacts are in
the `github/workflows/mmdebstrap.yml` file and it can be changed as needed.

Adjust the matrix to produce only the artifacts that you need. It might get
expensive if the matrix is producing too many jobs. Notably, you'll want to
change the architecture and the Debian version to be installed (e.g `armhf`,
`arm64`, `amd64`, `stretch`, `buster` and `bullseye`).

Next, change the `variant` variable, to select the system packages to install.
The `standard` value is the default for debootstrap, and values `essential`,
`required` and `important` are also supported. Consult mmdebstrap documentation
for more information.

For a real desktop installation, you might want to use different network
settings, which can also be adjusted in the matrix, use either `ifupdown`,
`systemd-networkd` or `network-manager`. But then after, you will need to do
the proper configuration too (advanced use).

While you can still chroot into the produced Debian filesystem in order to
install additional packages, you can also do it right in the 'mmdebstrap'
step. Add the comma-separated packages to the `--include` flag of the
mmdebstrap tool.
