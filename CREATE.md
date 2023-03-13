This document describes the steps required to create the
initial debian package.

## Create the initial debian package for unstable

1. Create the initial debianized git environment:
   ```
   mmbtool_name=odr-dabmod
   mmbtool_version=2.6.0
   upstream="https://github.com/Opendigitalradio/${mmbtool_name}/archive/refs/tags/v${mmbtool_version}.tar.gz"

   mkdir -p ${HOME}/odr-mmbtool_names/${mmbtool_name}
   cd ${HOME}/odr-mmbtool_names/${mmbtool_name}
   git init
   gbp import-orig \
     --upstream-branch=upstream/latest \
     --debian-branch=debian/latest \
     ${upstream}
   ```
1. Add the debian template files:
   ```
   debmake \
     --package ${mmbtool_name}
     --upstreamversion ${mmbtool_version}
   ```
1. Review and complete the content of the debian directory. Check the [Guide for Debian Maintainer](https://www.debian.org/doc/manuals/debmake-doc/index.en.html)
1. Build the debian package:
   ```
   build=$(pwd)/../build-area;
   dist=unstable;
   branch=debian/latest;
   git checkout ${branch};
   gbp buildpackage \
     --git-ignore-new \
     --git-debian-branch=${branch} \
     --git-export-dir=${build} \
     --git-builder=debspawn build --lintian --results-dir=${build}/${dist} ${dist}
   ```
1. Verify the results from lintian, fix the problems and repeat the previous build
