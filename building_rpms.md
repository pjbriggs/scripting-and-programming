Making RPMS: notes
==================

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
