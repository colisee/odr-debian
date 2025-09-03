# Create a new package

This document describes the steps required to create the
initial debian package from scratch.

## Common

1. Set the package name

   ```sh
   pkg_name=odr-audioenc
   ```

1. Set the package version

   ```sh
   pkg_version=3.3.1
   ```

## unstable

1. Set the distribution name

   ```sh
   distrib=unstable
   ```

1. Update the sbuild environment

   ```sh
   sudo sbuild-update --update --dist-upgrade --clean --autoclean --autoremove ${distrib}
   ```

1. Create the initial debianized git environment

   ```sh
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

   ```sh
   debmake \
     --package ${pkg_name} \
     --upstreamversion ${pkg_version}
   ```

1. Review and complete the content of the debian directory.
Check the [Guide for Debian Maintainer](https://www.debian.org/doc/manuals/debmake-doc/index.en.html)

1. Build the debian package:

   ```sh
   gbp buildpackage \
     --git-debian-branch=debian/latest \
     --git-ignore-new
   ```

1. Verify the results from lintian, fix the problems if any and repeat the
previous build until you are satisfied

1. Commit the changes:

   ```sh
   git add debian/
   git commit -m "Initial release"
   ```

1. Create the final build and tag the debian release:

   ```sh
   gbp buildpackage \
     --git-debian-branch=debian/latest \
     --git-tag
   ```

1. [Send the package to debian mentors](MENTORS.md)

1. Once the package lands in the testing repository,
[push the local repository to debian salsa](SALSA.md)

## backports

Once your debian package is available in `testing`, you can create the
package for backports

1. Set the distribution name

   ```sh
   distrib=bookworm-backports
   ```

1. Update the sbuild environment:

   ```sh
   sudo sbuild-update --update --dist-upgrade --clean --autoclean --autoremove ${distrib}
   ```

1. Create and switch to the backports branch:

   ```sh
   git checkout debian/latest
   git checkout -b debian/bookworm-backports
   ```

1. Merge with the latest branch:

   ```sh
   git merge debian/latest
   ```

1. Update the `debian/changelog` file:

   ```sh
   gbp dch \
     --bpo \
     --debian-branch=debian/${distrib} \
     --distribution=${distrib}
   ```

1. Build the debian package:

   ```sh
   gbp buildpackage \
     --git-debian-branch=debian/${distrib} \
     --git-ignore-new
   ```

1. Verify the results from lintian, fix the problems if any and repeat the
previous build until you are satisfied

1. Apply the checklist

1. Commit the changes

1. Commit the changes in `debian/changelog`

   ```sh
   git add debian/changelog
   git commit -m "Rebuild for ${distrib}"
   ```

1. Tag the debian release:

   ```sh
   gbp buildpackage \
     --git-debian-branch=debian/${distrib} \
     --git-tag-only
   ```

1. [Send the package to debian mentors](MENTORS.md)

1. Once the package lands in the backports repository,
[push the local repository to debian salsa](SALSA.md)
