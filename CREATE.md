This document describes the steps required to create the
initial debian package from scratch.

## unstable

1. Set the package name
   ```
   pkg_name=odr-audioenc
   ```
1. Set the package version
   ```
   pkg_version=3.3.1
   ```
1. Set the distribution name
   ```
   distrib=unstable
   ```
1. Create the initial debianized git environment:
   ```
   pkg_dir="${HOME}/dev/debian/${pkg_name}"
   upstream="https://github.com/Opendigitalradio/${pkg_name}/archive/refs/tags/v${pkg_version}.tar.gz"
   mkdir -p "${pkg_dir}"
   cd "${pkg_dir}"
   wget \
     --output-document="../${pkg_name}_${pkg_version}.tar.gz" \
     ${upstream}
   git init
   gbp import-orig \
     --no-interactive \
     --upstream-branch=upstream/latest \
     --debian-branch=debian/latest \
     "../${pkg_name}_${pkg_version}.tar.gz"
   rm ../${pkg_name}_${pkg_version}*.tar.gz
   ```
1. Add the debian template files:
   ```
   debmake \
     --package ${pkg_name} \
     --upstreamversion ${pkg_version}
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

## backports

Once your debian package is available in `testing`, you can [create the 
package for backports](BACKPORTS.md)