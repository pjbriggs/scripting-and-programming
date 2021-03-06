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

To find out what package a file comes from:

    $ rpm -qf /usr/bin/<program>

(NB `yum whatprovides */<file>` does something similar.)

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

### Useful links for using mock ###

 * <http://fedoraproject.org/wiki/Projects/Mock>
 * <http://gr8can8dian.wordpress.com/2011/01/22/building-a-fedora-rpm-with-mock/>
 * <http://www.openfusion.net/linux/mocking_rpms>

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

 * __Problem: rpmbuild detects "standard RPATHs" and stops with an error__:

    To allow standard RPATHS, add this line to the end of the `%install` section:
    `export QA_RPATHS=0x0001`

 * __Problem: package doesn’t have a version number__:

   In these cases the suggested solution is to use an “epoch” number:
   <http://www.rpm.org/max-rpm-snapshot/s1-rpm-depend-manual-dependencies.html>

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

Miscellaneous stuff
-------------------

 * __To find out whether a package is available from EPEL__

   Use the `bodhi-client` package (install via `yum`) and then do e.g.

   `bodhi bfast`

   which will list the latest updates for that package across all branches i.e.
   different Fedora and EPEL versions (if applicable).

 * __EPEL5 checksum error__

   When trying to use SRPMs built by Fedora 15 as input to `rpmbuild` for EPEL5,
   you are likely to get an md5 sum mismatch fatal error. This is because from
   Fedora 11 onwards the RPMs use SHA256 but EL5 uses MD5). You can get around this
   by using

   `rpm-build-md5`

   to build the SRPMS specifically for EPEL5. It’s not a problem for EPEL6.
   (`rpm-build-md5` is in the `fedora-packager` package in yum.)

 * __Making "binary" RPMs__

   Essentially anything that’s just being untarred and installed (e.g. R packages,
   data files, or pre-built executables etc) can also be RPM-ed. A useful convention
   is to add `-bin` to the package name in these cases.

 * __Relocatable RPMs__

   Relocatable packages can be installed under a user-defined directory by specifying
   `rpm`’s `--prefix` option at installation time.

   This is done by including the `Prefix` directive in the spec file, e.g.

   `Prefix:  /usr`

   which indicates that all file paths starting with `/usr` should have this replaced
   by the path specified by `--prefix`. The drawback with this approach is that `yum`
   doesn’t support `--prefix` so package management would have to be done manually.

 * __Python packages and setuptools__

   `Setuptools` and its variants support options to build RPMs and spec files
   automatically:

   `python setup.py bdist_rpm # creates the full RPM`

   or

   `python setup.py bdist_rpm --spec-only`

 * __Making your own RPM repository__

    * Goes into Apache webroot
    * Run `repobuilder` (check name)
    * Put the rpms in there

Resources & Links
-----------------

 * <http://www.rpm.org/>
 * <http://fedoraproject.org/wiki/How_to_create_an_RPM_package> (includes template SPEC file)
 * <http://fedoraproject.org/wiki/Packaging/NamingGuidelines> (includes naming conventions
   for things like Python packages)
 * Packaging Python: <http://fedoraproject.org/wiki/Packaging:Python>
 * Info on signing RPMs: <http://pmc.ucsc.edu/~dmk/notes/RPMs/Creating_RPMs.html>