---
layout: post
title:  "Bleeding Realtime Kernel Packages Available for Jessie x86"
date:   2015-05-30 18:45:00
categories: news
---

Bleeding Xenomai and RTAI packages are now available for Jessie, x86
architectures.

**WARNING**: These packages are known not to work with Machinekit.
They are for developer testing only.  Most folks should **NOT** use
these packages.

**WARNING**: These packages are unsupported.

<!---more--->

## Methodology

These packages are currently based on the following sources:

- [Xenomai 3.0-rc4][xeno-3rc4] snapshot
  - [Xenomai packaging][xeno-pkg] git tree `3.0-rc4` branch
  - Minor updates to upstream packaging needed
- [ShabbyX/RTAI][rtai-shabbyx] git tree `master` branch
  - [RTAI packaging][rtai-pkg] git tree `4.1` branch
- [Linux 3.16.7][linux] tarball
  - [Linux packaging][linux-pkg] git tree `3.16.7` branch
  - Based on upstream Debian packaging; see `README.md` for methodology

Built with [dxsbuild][dxsbuild]; see [commit
`d13c1815`][dxsbuild-d13c1815].

[xeno-3rc4]: http://git.xenomai.org/xenomai-3.git/snapshot/xenomai-3-3.0-rc4.tar.bz2
[xeno-pkg]: https://github.com/zultron/xenomai-deb/tree/3.0-rc4
[rtai-shabbyx]: https://github.com/ShabbyX/RTAI.git
[rtai-pkg]: https://github.com/zultron/rtai-deb/tree/4.1
[linux]: http://www.kernel.org/pub/linux/kernel/v3.0/linux-3.16.7.tar.xz
[linux-pkg]: https://github.com/zultron/linux-ipipe-deb/tree/3.16.7
[dxsbuild]: https://github.com/zultron/dxsbuild/
[dxsbuild-d13c1815]: https://github.com/zultron/dxsbuild/commit/d13c1815b699100863d41410ab113768222478f3

## Problems

In case of problems, please file issues on the respective
[Xenomai][xeno-pkg], [RTAI][rtai-pkg] or [Linux][linux-pkg] GitHub
repository trackers.

## Installation

If you already have the
[Dovetail Automata Jessie package repository][da-jessie] enabled,
follow the below instructions.  If not, manually install the
[`dovetail-automata-keyring` package][da-keyring] first.

[da-jessie]: http://www.dovetail-automata.com/articles/2015/05/28/machinekit-jessie-packages/
[da-keyring]: http://deb.dovetail-automata.com/pool/main/d/dovetail-automata-keyring/

### Enable the `jessie-bleeding` Apt repo

    $ echo deb http://deb.dovetail-automata.com \
        jessie-bleeding main | \
        sudo sh -c \
            'cat > /etc/apt/sources.list.d/da-jessie-bleeding.list'
    $ sudo apt-get update

### Install/upgrade Xenomai packages

    $ sudo apt-get install \
        libxenomai-dev libxenomai1 xenomai-runtime \
        linux-image-3.16.0-4-xenomai.x86-*

### Install/upgrade RTAI packages

    $ sudo apt-get install \
        librtai-dev librtai1 rtai python-rtai \
        linux-headers-3.16.0-4-common-rtai.x86 \
        linux-headers-3.16.0-4-rtai.x86-* \
        linux-image-3.16.0-4-rtai.x86-*

### Reboot

Reboot into the new kernel.

Check the running kernel version:

    $ uname -r
    3.16.0-4-rtai.x86-amd64

For Xenomai, run unit tests:

    $ xeno-test

For RTAI, there are no packaged tests.  Please file an issue on the
[RTAI packaging GitHub tracker][rtai-pkg-tracker] if this is something
you need.

[rtai-pkg-tracker]: https://github.com/zultron/rtai-deb/issues

## Uninstall

Uninstalling is the reverse of installing.

Remove Xenomai packages:

    $ sudo apt-get remove \
        libxenomai-dev libxenomai1 xenomai-runtime \
        linux-image-3.16.0-4-xenomai.x86-*

Remove RTAI packages:

    $ sudo apt-get remove \
        librtai-dev librtai1 rtai python-rtai \
        linux-headers-3.16.0-4-common-rtai.x86 \
        linux-headers-3.16.0-4-rtai.x86-* \
        linux-image-3.16.0-4-rtai.x86-*


Remove `jessie-bleeding` Apt repository:

    $ sudo rm /etc/apt/sources.list.d/da-jessie-bleeding.list
