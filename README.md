# regolith-st

This is the terminal package for Regolith, which is an open source project called [Simple Terminal from the suckless.org project](https://st.suckless.org/).  This repo does not contain the actual code for the terminal application, only the Debian package metadata necessary to bundle it as a package and any customizations Regolith has added, expressed as patches against the upstream source.

# Known issues

## `neovim` font colors producing invisible characters in `st`.

See [this issue](https://github.com/regolith-linux/regolith-st/issues/1) for description and resolution.

# How to build this package

This guide assumes you wish to make changes to a Regolith package and host those changes in a new version.  It also assumes you'll be building from an Ubuntu-based system like Regolith and you already have setup the Debian package tools.  See [this concise guide](https://wiki.debian.org/BuildingTutorial) to install the tools and get a familiar with the workflow.

## Check out the package

This step will pull the Regolith package metadata down from GitHub.  The debian metadata is on a branch called `debian`, due to conflicts that this README presents to the build tool.

```bash
$ mkdir workspace

$ git clone -b debian https://github.com/regolith-linux/regolith-st
```

## Download and prepare the upstream source

This step will download the original package source code. We alter the name of the tarball to make Debian's build scripts happy.

```bash
$ wget -O regolith-st_0.8.2.orig.tar.gz https://dl.suckless.org/st/st-0.8.2.tar.gz
```

## Extract source and apply patches

This is an optional step if you want to have a look at the actual files that are being built into the package.  If you just want to build, you can proceed to the next step.

First we need to extract the upstream source into the root of the package:

```bash
$ tar xfzv ../regolith-st_0.8.2.orig.tar.gz
$ mv st-0.8.2/* .
$ rmdir st-0.8.2/ 
```

Next we apply patches to the source we extracted using the `quilt` tool:
```bash
$ quilt push -a
```

At this point the files you see in the package directory are what gets built into the binary pacage.  If you'd like to make changes and have them be added as new patches, use quilt:

```bash
$ quilt new my-change.patch  # create a new empty patch
$ quilt add config.def.h     # tell quilt that config.def.h is part of your new patch
$ vim config.def.h           # Make whatever changes you like
$ quilt refresh	             # tell quilt to refresh the patch based on your file changes
$ quilt pop -a               # roll all the changes back to the original source
```

At this point the source in the package should be just as the upstream tarball we downloaded and extracted.  The only difference is that we have new patch data in debian/patches.  The package build tools know to apply these before building.


## Build the package

Now we will build the package locally and generate the Debian package metadata required for hosting in a Private Package Archive (PPA).

```bash
$ cd regolith-st

$ debuild -S -sa
```

There should now be a number of files generated in the parent directory, such as `regolith-st_0.8.2-1ubuntu18_source.changes`.

## Bump the package version

It's necessary to update the version of the package so that the local package manager will be able to determine that an update is available.  The Debian packaging tools provide the program `dch` for this task.  `dch` modifies the `debian/changelog` file.  This file contains the package versioning metadata and is organized by stanzas of three elements: version, change info, and author of change.  It's similiar in a way to a git commit log + release tags, but managed in a file rather than the git index.  Run `dch` and update the version string, change UNRELEASED to your intended Ubuntu version target (probably `bionic`) and then below add a description of your change.  The rest should be done automatically by the `dch` program.  Verify that your change follows the pattern of entries below it.

## Upload the package to your PPA

<sub>You may wish to change the package version in the `debian\changelog` file to avoid conflicts, but that is optional and up to you.</sub>

```bash
$ dput ppa://<username>/<ppa name> ../regolith-st_0.8.2-1ubuntu18_source.changes
```

## Verification

After uploading the package to your PPA, you should get an email from Launchpad.net regarding if the package was accepted or rejected.  Then, some time later the package should have been build and available for installation.  Assuming you have already added your PPA via `add-apt-repository`, `sudo apt update` and `sudo apt upgrade` should cause the new package to be installed on your local system.
