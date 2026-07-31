# pacbrew-repo

The OpenAGC graphics package chain, pinned Mesa-Zink runtime, SDL integration,
and Mesa compatibility notes are
documented in [docs/openagc-vulkan-ps5.md](docs/openagc-vulkan-ps5.md).

## Prerequisites
On Debian-flavored operating systems, you can invoke the following commands to
install dependencies used by pacbrew-repo.
```console
john@localhost:ps5-payload-dev/pacbrew-repo$ sudo apt-get update && sudo apt-get upgrade
john@localhost:ps5-payload-dev/pacbrew-repo$ sudo apt-get install cmake pkg-config meson \
    clang lld build-essential autoconf libtool yasm nasm bison flex  gperf pkgconf \
    libarchive-tools autopoint po4a git curl doxygen makepkg pacman-package-manager \
    python3-mako python3-yaml python3-glad
```

## Building and installing to /opt/ps5-payload-sdk
```console
john@localhost:ps5-payload-dev/pacbrew-repo$ export MAKEFLAGS=-j8 # optionally build in parallel on 8 cores
john@localhost:ps5-payload-dev/pacbrew-repo$ ./ci-libs.sh
```
