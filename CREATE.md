# Create a new package

1. Set the package name

   ```sh
   pkg_name=odr-audioenc
   ```

1. Set the package version

   ```sh
   pkg_version=3.3.1
   ```

1. Set the distribution name

   ```sh
   distrib=unstable
   ```

1. Update the sbuild environment

   ```sh
   rm $HOME/.cache/sbuild/${distrib}-amd64.tar.zst
   mmdebstrap \
     --include=ca-certificates \
     --skip=output/dev \
     --variant=buildd \
     ${distrib} \
     $HOME/.cache/sbuild/${distrib}-amd64.tar.zst \
     https://deb.debian.org/debian
   ```

1. Create the initial debianized git environment

   ```sh
   upstream="https://github.com/Opendigitalradio/${pkg_name}/archive/refs/tags/v${pkg_version}.tar.gz"
   mkdir "${pkg_name}"
   cd "${pkg_name}"
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

1. Tag the debian release:

   ```sh
   gbp buildpackage \
     --git-debian-branch=debian/${distrib} \
     --git-tag-only
   ```

1. [Send the package to debian mentors](MENTORS.md)

1. Once the package lands in the unstable repository,
[push the local repository to debian salsa](SALSA.md)
