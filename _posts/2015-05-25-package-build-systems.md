---
layout: post
title:  "Build Systems for Software Package Distributions"
date:   2015-05-25 10:53:51
categories: Build
---

Looking for a package build system that does everything?  So are we.

[Dovetail Automata][DA] ships a
[Debian package distribution for the Machinekit project][MK-deb].  The
distro is small, and getting smaller as some dependency packages are
being integrated into upstream distributions, but small doesn't mean
simple.  We need builds for Debian Wheezy, Debian Jessie and Ubuntu
Trusty (at least), and for `x86_64`, `i386` and `armv7l` CPU
architectures.  Packages contain interdependencies, and results of one
build are inputs for another.  And the kernel packages take a long
time with two 'featuresets' per build.  These all add extra
requirements on a build system that, it turns out, most are unable to
meet.

In our (ongoing) quest to find the perfect build system, we've
investigated quite a number of options, ranging from free, cloud-based
systems, to private instances of build service software, to simple
chroot builders.  Here is a simple comparison of those we've tried, a
few we haven't, and where we are today.

[DA]: http://dovetail-automata.com
[MK-deb]: http://www.machinekit.io/docs/packages-debian/

<!---more--->

# Cloud services

The big commercial GNU/Linux distro vendors each have their own free,
public, cloud-based package build service.  They differ most obviously
in which distro they compile packages for.  Other differences include
CPU architecture lists and how interdependencies are handled.

We tried most of these, and all fall short.  The biggest shortcoming:
there was no combination of services, much less a single service, that
could build all required combinations of distro, release and CPU
architecture; the most difficult turns out to be Debian `armhf`
architecture.

## [Launchpad][Launchpad]

We didn't try out [Ubuntu's package build service][Launchpad], but it
looks well-suited for Ubuntu-only builds.

Pros:

- ARM-architecture builds

Cons:

- Ubuntu only

[Launchpad]: https://launchpad.net/

## [COPR][COPR]

The Red Hat-sponsored Fedora project has a [relatively new][COPR] (~2
years) [build service][COPR].  Distinct from [`koji`][koji], the
official Fedora build service, COPR is a catch-up attempt following
Launchpad and openSUSE Build Service.

Pros:

- Allows configuring external package repositories

Cons:

- Fedora and Red Hat distributions only
- No ARM architecture
- Unreliable service:  often down

[COPR]: https://copr.fedoraproject.org/
[koji]: https://fedoraproject.org/wiki/Koji

## [OBS][OBS-svc] (openSUSE Build Service)

The [openSUSE Build Service][OBS-svc] uniquely advertises cross-distro package
builds, presumably an attempt to compete with the bigger Red Hat.

Pros:

- Some cross-distro package builds
- ARM builds for some distros

Cons:

- Poor support for `.deb` packages; see below
- No ARM builds for Debian
- Long or large package builds can fail

The OBS cross-distro promise is so enticing that I have spent a couple
of weeks in total trying to cajole it into building Debian packages.
It turns out that Debian support is quite poor for a few reasons.  Its
macros meant for sharing source tarballs (which have special names in
Debian source packages) only work for `3.0 (quilt)` packages.  The NIH
version control system doesn't support subdirectories, so it can't
manage the many Debian packages that contain a `source` directory.
Plus, the Debian documentation is extremely poor; I had to read the
OBS source code to reverse-engineer the problems I ran into.

OBS can be tricked into building Debian packages, but the source
packages must be pre-built and uploaded to OBS; there is really no
shared source with RPM packages, as claimed.  Building the kernel
packages was also frustrating, because when they landed on a build
host that was slower or had limited disk space, the build would fail.
It could sometimes take ten attempts over two or more days to
successfully build the package.

[OBS-svc]: https://build.opensuse.org/

# Build service private instance

Unlike cloud build services that have poor support for Debian and ARM
architecture, a private build service instance can have any
architecture builders required.  The difficulty with private instances
is they are difficult to set up and maintain, and probably need to be
run on private infrastructure, since there is currently only one,
brand-new cloud that has ARM architecture hosts.

## Debian [`buildd`][buildd]

The Debian [`buildd`][buildd] is the clear choice for Debian builds.  We never
experimented with it, since we were initially building RPM packages,
and ended up building our own service.

[buildd]: https://buildd.debian.org/

## [Koji][koji]

[Koji][koji], which the Fedora project uses in its official build
service, builds RPM packages only.  Like all private instances it is
difficult to set up and maintain, and the poor documentation makes
this even more difficult.

## [OBS][OBS-sw] (Open Build Service) private instance

[Open Build Service][OBS-sw] is the software behind the openSUSE Build
Service, and shares pros and cons.  Presumably, the incomplete ARM
support and failing builds could be fixed.  Although Debian support is
poor, a private OBS instance would have the most complete support for
any single build service.

[OBS-sw]: http://openbuildservice.org/

# Single package chroot builders

Because no build service was able to deliver all the packages needed,
the Dovetail Automata builder uses a much simpler chroot-based
package builder.  These are great for building a single package for
many distros.  When building packages with interdependencies, or
cross-building, the configuration can become complicated and more
infrastructure than just a simple shell script is needed.

## [`pbuilder`][pbuilder]

[`pbuilder`][pbuilder] builds Debian packages.  It is not cross-build capable, but
can run builds, slowly, under emulation.  It is losing popularity to
`sbuild`.

[pbuilder]: http://pbuilder.alioth.debian.org/

## [`sbuild`][sbuild]

[`sbuild`][sbuild] builds Debian packages, and has the advantage of being
cross-build capable.  In practice, few packages cross-build as of
Jessie, but this will improve.  In the meantime, emulated builds work,
and can be sped up a small amount by using `distcc` and a
cross-compiler.

[sbuild]: https://wiki.debian.org/sbuild

## [Mock][mock]

[Mock][mock], Red Hat's chroot build tool, builds RPMs.
Cross-building is discouraged in Fedora.

[mock]: https://fedoraproject.org/wiki/Mock

# Conclusion:  What we picked

None of the cloud-based build services could build all the packages
for our distro, even after having abandoned RPM packages.  A private
build service instance could have, and we run a rack of servers here
in the shop, but we believe it's better for the Machinekit project to
make the build infrastructure uncomplicated so that others can
reproduce it for themselves and build and test for other environments.

So, we chose `sbuild`, and wrote a stack of shell scripts around it
called [`dxsbuild`][dxsbuild] that can build a list of packages for several
distributions and several architectures.  Because it uses `sbuild`, it
can cross-build for distros where toolchains exist; where they don't,
it can run `distcc` with a cross-compiler to speed up emulated builds,
and it speeds up any type of build with `ccache`.

The `dxsbuild` scripts add extra infrastructure around the single
package build.  They add build results to a package repository that is
used by following builds of interdependent packages.  They also run
the special source package configuration step in a chroot that some
packages like the kernel and Machinekit require.

Still, `dxsbuild` is not a build service.  To make `dxsbuild`
portable, it was designed to run in a Docker container, so it's ready
to be run on many cloud platforms.  The hope here is that it can be
used as a building block for a build service.  A future article will
describe how a Buildbot slave might run in the `dxsbuild` Docker
container, and the container started on demand in a cloud-based
Buildbot configuration.

[dxsbuild]: https://github.com/zultron/dxsbuild/
