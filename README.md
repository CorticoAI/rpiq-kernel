# RPI Qemu Kernel

Builds an ARM kernel intended to allow QEMU emulation of Raspberry Pi
OS images (typically Raspbian). rpiq-kernel is based off the beloved
[qemu-rpi-kernel](https://github.com/dhruvvyas90/qemu-rpi-kernel), but
at this point has wildly diverged.

The rpiq-kernel is intended for use with QEMU virt devices, and so
includes the necessary kernel configs. This allows you to spin up VMs
that more closely resemble Raspberry Pi hardware in its specs as
compared to what qemu-rpi-kernel allows. It tracks the rpi-4.19.y
branch, and uses the vexpress board config. In addition to the virt
device configs, rpiq-kernel also enables some features I selfishly
need, such as Docker capability and XFS. I am open to piling more
dishes into the kitchen sink.

## Building

The Raspberry Pi published Linux kernel is in a git submodule, so
you'll need to make sure that you've pulled it in addition to this
repo. See regular git submodule documentation.

You'll need an ARM cross compiler. On Debian/Ubuntu systems, you
install the `gcc-arm-linux-gnueabihf` package.

You'll also need QEMU, natch, which should be installable easily by
any reasonable Linux distribution, or Homebrew if you've chosen the
cushy life of an OS X user.

Building is easy:

```
$ ./build-rpiq-kernel

... wait for it to compile (~8m on my 2016-ish laptop)

$ ls output/
rpiq-kernel-4.19.97-r1  vexpress-v2p-ca9.dtb
```

The `rpiq-kernel-` file is your kernel. The `.dtb` file is used if
you're emulating a VExpress board, but if you use virt emulation you
don't need it.

If you have a Raspbian image, you can run it like so:

```
$ qemu-system-arm -M virt -m 1024 -drive file=2019-09-26-raspbian-buster-lite.img,if=none,format=raw,id=disk -device virtio-blk-device,drive=disk -netdev user,id=network -device virtio-net-device,netdev=network -device virtio-rng-pci -kernel output/rpiq-kernel-4.19.97-r1 -append 'console=ttyAMA0,115200 root=/dev/vda2' -nographic
```

This starts QEMU up and boots via the text-only serial console. It
does not appear to be possible to get QEMU to set up a display in ARM
mode, so you're stuck with the serial console.

If you want to start up QEMU as a VExpress board rather than virt,
then:

```
$ qemu-system-arm -M vexpress-a9 -cpu cortex-a9 -dtb output/vexpress-v2p-ca9.dtb -m 1024 -drive file=2019-09-26-raspbian-buster-lite.img,if=sd,format=raw,id=disk -net user -kernel output/rpiq-kernel-4.19.97-r1 -append 'console=ttyAMA0,115200 root=/dev/mmcblk0p2' -nographic
```

But honestly I don't know why you'd want to do that cause it's slower.

## Customizing

You can add extra kernel configs to a `custom_config` file. For
example:

```
$ echo "CONFIG_BTRFS_FS=y" > custom_config
```

## Vagrant Build

Just in case you don't have a Linux machine, you can run the build in
a Vagrant VM with the supplied `Vagrantfile`.

First, make sure that you've set up git submodules on your host
machine and have the Linux source. Git performance in Vagrant VMs can
be really bad.

```
$ vagrant up
$ vagrant ssh

... logs into the VM

$ cd /vagrant
$ ./build-rpiq-kernel

... ctrl-d to log out

$ vagrant destroy
```

## References

- Raspberry Pi Foundation Linux kernel:
  https://github.com/raspberrypi/linux

- Info about virt and ARM builds:
  https://wiki.qemu.org/Documentation/Platforms/ARM

- Fantastic reference on how QEMU devices work and the inspiration for
  virt: https://www.qemu.org/2018/02/09/understanding-qemu-devices/

## Notes

- rpiq-kernel has only been tested with Raspbian and private RPi
  compatible Linuxes. Notably, NOOBS is untested.

- Tested with QEMU that ships with Ubuntu Bionic (version 2.11+), and
  hand built QEMU version 4+.
