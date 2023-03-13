This document describes the steps required to build
the debian packages from existing debian sources.

## Test the build execution for the unstable distribution

1. Go to the source directory:
   ```
   cd ${HOME}/odr-mmbtools/<source>
   ```
1. Build the debian package:
   ```
   build=${HOME}/odr-mmbtools/build-area
   dist=unstable
   branch=debian/latest
   git checkout ${branch}
   gbp buildpackage \
     --git-debian-branch=${branch} \
     --git-export-dir=${build} \
     --git-builder=debspawn build --lintian --results-dir=${build}/${dist} ${dist}
   ```
1. Verify the results from lintian and fix the problems

## Build the package for a specific distribution
1. Build the debian packages:
   ```
   build=${HOME}/odr-mmbtools/build-area
   dist=bullseye
   branch=debian/${dist}
   git checkout ${branch}
   for arch in amd64 arm64 armhf; do
     gbp buildpackage \
       --git-debian-branch=${branch} \
       --git-export-dir=${build} \
       --git-builder=debspawn build --lintian --results-dir=${build}/${dist} ${dist}
   done
   ```
