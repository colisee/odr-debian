This document describes the steps required to create the
initial debian package from scratch.

## Create the initial debian package for unstable

1. Set the tool name
   ```
   mmbtool_name=odr-audioenc
   ```
1. Set the tool version
   ```
   mmbtool_version=3.3.1
   ```
1. Set the distribution name
   ```
   distrib=unstable
   ```
1. Create the initial debianized git environment:
   ```
   mmbtool_dir="${HOME}/dev/debian/${mmbtool_name}"
   upstream="https://github.com/Opendigitalradio/${mmbtool_name}/archive/refs/tags/v${mmbtool_version}.tar.gz"
   mkdir -p "${mmbtool_dir}"
   cd "${mmbtool_dir}"
   wget \
     --output-document="../${mmbtool_name}_${mmbtool_version}.tar.gz" \
     ${upstream}
   git init
   gbp import-orig \
     --no-interactive \
     --upstream-branch=upstream/latest \
     --debian-branch=debian/latest \
     "../${mmbtool_name}_${mmbtool_version}.tar.gz"
   rm ../${mmbtool_name}_${mmbtool_version}*.tar.gz
   ```
1. Add the debian template files:
   ```
   debmake \
     --package ${mmbtool_name} \
     --upstreamversion ${mmbtool_version}
   ```
1. Review and complete the content of the debian directory. Check the [Guide for Debian Maintainer](https://www.debian.org/doc/manuals/debmake-doc/index.en.html)
1. Build the debian package:
   ```
   gbp buildpackage \
     --git-debian-branch=debian/latest \
     --git-export \
     --git-export-dir=../build \
     --git-ignore-new
   ```
1. Verify the results from lintian, fix the problems if any and repeat the 
previous build until you are satisfied
1. Commit the changes:
   ```
   git add debian/
   git commit -m "Initial release"
   ```
1. Create the final build and tag the debian release:
   ```
   gbp buildpackage \
     --git-debian-branch=debian/latest \
     --git-export \
     --git-export-dir=../build \
     --git-tag
   ```
1. [Send the package to debian mentors](MENTORS.md)
1. Once the package lands in the testing/backports repository,
[push the local repository to debian salsa](SALSA.md)

## Create the initial debian package for backports

Once your debian package is available in `testing`, you can [create the 
package for backports](BACKPORTS.md)