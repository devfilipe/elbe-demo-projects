# elbe-demo-projects

ELBE XML image definitions for the demo project. Each file in this repository
describes a complete root filesystem — the target architecture, Debian suite,
package list, partition layout, and post-install fine-tuning — that ELBE will
build inside its initvm.

## Image definitions

### `x86_64-qemu-minimal.xml`

A minimal bootable disk image for x86-64 QEMU. Goals: smallest possible
footprint, serial-console-only (no VGA), fast to build.

| Property | Value |
|---|---|
| Architecture | `amd64` |
| Debian suite | `bookworm` |
| Debootstrap variant | `minbase` |
| Image | `sda.img` — 1 GiB GPT, BIOS boot + ext4 root |
| Console | `ttyS0` at 115200 bps |
| Default login | `root` / `root` |
| Custom packages | `elbe-demo-pkg-hello` |

Key packages: `linux-image-amd64`, `grub-pc`, `systemd`, `systemd-sysv`,
`udev`, `elbe-demo-pkg-hello`.

### `x86_64-qemu-hdimg.xml`

A bootable disk image for x86-64 QEMU with support for both BIOS and UEFI
firmware. Slightly larger than the minimal image and includes a richer base
system.

| Property | Value |
|---|---|
| Architecture | `amd64` |
| Debian suite | `bookworm` |
| Debootstrap variant | `minbase` |
| Image | `sda.img` — 2 GiB GPT: BIOS boot + EFI (100 MiB, FAT32) + ext4 root |
| Console | `ttyS0` at 115200 bps (also accessible via VGA) |
| Default login | `root` / `root` |
| Custom packages | `elbe-demo-pkg-hello` |

Key packages: `linux-image-amd64`, `grub-pc`, `grub-efi-amd64-bin`,
`systemd`, `udev`, `dbus`, `bash`, `bash-completion`, `elbe-demo-pkg-hello`.

### `armhf-ti-beaglebone-black.xml`

Root filesystem for the Texas Instruments BeagleBone Black (AM335x, ARMv7).
Produces an SD card image with U-Boot and a Debian bookworm rootfs. This image
does not include custom packages and is provided as a reference for ARM/
cross-compilation targets.

| Property | Value |
|---|---|
| Architecture | `armhf` |
| Debian suite | `bookworm` |
| Image | `sdcard.img` — 1.5 GiB MBR: U-Boot env partition + ext4 root |
| Bootloader | U-Boot (`u-boot-omap`) |
| Console | `ttyO0` at 115200 bps |
| Default login | `root` / `root` |

Key packages: `linux-image-armmp`, `u-boot-omap`, `u-boot-menu`,
`systemd-resolved`, `openssh-server`, `busybox`.

## Local APT repository

The x86-64 images reference a local flat APT repository served by the ELBE
dev container at `http://10.0.2.2:8080/repo/0` (the host as seen from inside
QEMU user networking). This repository is managed by
[`elbe-demo-apt-repository`](https://github.com/devfilipe/elbe-demo-apt-repository)
and must be populated before submitting a build.

```xml
<url>
  <binary>[trusted=yes] http://10.0.2.2:8080/repo/0 ./</binary>
  <source>[trusted=yes] http://10.0.2.2:8080/repo/0 ./</source>
</url>
```

## Building an image

Images are built inside the ELBE initvm. The recommended way is through the
`elbe-ui` web interface, but the equivalent CLI command is:

```bash
elbe initvm submit --qemu --directory /workspace/.elbe/initvm \
  --skip-build-sources --output /workspace/builds \
  /workspace/projects/elbe-demo-projects/<xml-name>.xml
```

## License

See the [LICENSE](LICENSE) file.
