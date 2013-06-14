Making RPMS using rpmbuild and mock
===================================

Setting up
----------

 1.  __Never build RPMs as `root`__: it's recommended to create a dummy user
    specifically for making RPMs, however I normally build them as myself.

 2. __Install the necessary development tools__:
    * `yum install rpmdevtools`
    * `yum groupinstall "Development Tools"`

 3. __Set up the RPM directory structure__: run the `rpmdev-setuptree` utility, which
    will create an `rpmbuild` directory containing `BUILD`, `RPMS`, `SOURCES`, `SPECS`
    and `SRPMS`.

Building an RPM
---------------

Building an RPM typically requires:

 *  a __spec file__, which contains the instructions on how to unpackage, build and
    install a package, and also specifies which files to include in the final RPM
 *  the __source code__ for the package, normally a `tar.gz` or `zip` archive.
 *  some RPMs also have associated __patch files__ which are referenced in the spec file.

The spec file should be put into the `SPECS` directory and the source archives and
patches should be in `SOURCES`.

To build the RPM: `cd` to the `rpmbuild` directory, then do:

    rpmbuild -v -bb --clean SPECS/<specfile>

(The `-bb` option tells `rpmbuild` to only build the installable binary RPM.)

Useful common `rpmbuild` options:

    -bb: build binary RPM file
    -bs: build source SRPM file
    -ba: build all (i.e. both RPM and SRPM)
    -v : verbose output
    -vv: very verbose output
    --clean: remove the build tree after the packages have been built

(More advanced options include:

    -bi: only perform the steps in the `install` stage from the spec file
    -bl: only perform the steps in the `file` stage

Note that these must be combined with the `--short-circuit` option, to prevent
`rpmbuild` redoing the other stages from the spec file.)

Installing and removing RPMs
----------------------------

`yum` is the recommended way of installing packages:

    $ sudo yum install /path/to/package.rpm

(For older versions of `yum` the `localinstall` option is preferred to `install`.)

You might need to include the `--nogpgcheck` option if `yum` refuses to install
because of missing GPG keys (likely for locally generated RPM files).

To remove:

    $ sudo yum remove package

`yum` is preferred to using the `rpm` program directly, however the equivalent
`rpm` options are:

    $ sudo rpm -i /path/to/package.rpm
    $ sudo rpm -e package

Querying RPMs
-------------

To list the contents of an RPM:

    $ rpm -qpl package.rpm

Using mock to create RPMs for multiple platforms
------------------------------------------------

`mock` is a wrapper for `rpmbuild` which can create RPMs for multiple target
platforms on a single Red Hat-based distribution.

To set up (do once per system):

 1. Install the `mock` package e.g. `yum install mock`
 2. Add the user that will build the RPMs to the `mock` group e.g.
    `sudo /usr/sbin/usermod -a G mock $USER`

The general procedure for building an RPM for a specific system using `mock`
is then:

 1. __Set up a chroot for the target system__: (a chroot is a mock filesystem
    that `mock` uses to simulate the environment of the target platform)

    `mock -r <platform> --init`

    Platforms are defined by config files in `/etc/mock` as `<platform>.cfg`,
    for example `fedora-15-x86_64.cfg` or `epel-5-x86_64.cfg`.

 2. __Create an SRPM__:

    `mock -r <platform> --buildsrpm --spec /path/to/package.spec --sources /path/to/dir/with/sources`
 3. __Create an RPM__:

    `mock -r <platform> --rebuild /path/to/package.src.rpm`

By default the outputs (logs and RPMs) are put under `/var/mock/<platform>/results/`.

If `mock`'s RPM build fails then you will need to inspect the logs in order to work
out what went wrong (most commonly missing dependencies which weren't identified when
using `rpmbuild` natively).

I have a script `mock-it.sh` in my `mock-utils` github repo which wraps `mock` and
performs these steps in an automated fashion:

 * [mock-utils](https://github.com/pjbriggs/mock-utils)

### Dealing with "chain of dependencies" in mock ###

`mock` has problems dealing with "chains of dependencies", when one custom package
depends on one or more other custom packages (which may themselves also depend on
custom packages).

The standard approach is to create a local RPM repository which contains the
dependencies and then add this repository to the `cfg` file for the target platform,
before using `mock` to build the package.

My `mock-it.sh` script can do this automatically; alternatively th `smock.pl` can
handle dependency resolution in a more sophisticated manner (caveat: I have never tried
it):

 * [smock](http://www.redhat.com/archives/rhl-devel-list/2008-November/msg01229.html)

Creating a new RPM from scratch
-------------------------------

If the package you want to make into an RPM doesn't already have a spec file then
you will need to create a new one for it.

The recommended procedure for creating a new spec file is:

 * __Build the package by hand first__: this is important as it's the means for
   gathering the information required to populate the spec file; specifically it
   uncovers any non-standard steps or build problems that will require patches.
 * __Create and populate the spec file__: `rpmdev-newspec <name>` can be used to
   make a new blank spec file `<name>.spec` (personally I tend to use an existing
   spec file as a template).
 * __Build the package using rpmbuild__: typically this might take a few iterations
   to debug the spec file.
 * __Use mock to build RPMs for target systems__: this might also uncover previously
   hidden dependency issues.

### Spec file hints & tips ###

 * __Problem: source code archive unpacks into a non-standard directory__

   In the `%prep` section, use `%setup -n <dir>` if the source code archive unpacks
   into a directory other than `<package_name>-<version>`

 * __Problem: how to turn off the creation of a debug package__

   Add `%define debug_package %{nil}` at the head of the spec file.

 * __Problem: rpmbuild's automatic dependency detection causes problems by specifying
   a dependency on a package which is provided by the RPM__:

   This can happen for things like Perl modules that are included in the package;
   try using `AutoReqProv: no` to turn this off.

   (See also <http://www.rpm.org/max-rpm/s1-rpm-depend-auto-depend.html>)

 * __Problem: package doesn't include a specific installation step__:

   This is often the case when the package build doesn't include a `make install`
   step; in this case use the Linux `install` program and explicitly install files
   to an appropriate location within the `%install` section, e.g.

   `install -m 0755 <program> %{buildroot}%{_bindir}`

   `install -m 0644 <file> %{buildroot}/%{_datadir}`

   (Preface the installation destination with the `%{buildroot}` macro to avoid
   `rpmbuild` trying to install into the actual filesystem.)

### Patches ###

Use

    diff -u path/to/original_file path/to/new_file > file.patch

to generate patch files.

Patch files are specified in the spec file using the `Patch#` directive, e.g.

    Patch0:   file1.patch
    Patch1:   file2.patch
    ...

and are applied within the `%prep` section using `%patch#`, e.g.

    %patch0 -p1

### Useful macros ###

See <http://fedoraproject.org/wiki/Packaging:RPMMacros>

General:

    bindir  = /usr/bin
    datadir = /usr/share

Perl:

    perl_vendorarch: architecture-specific location for Perl modules
    perl_vendorlib : general library location for Perl modules