# Upload to mentors.debian.net

This document describes the steps required to push the
debian package to debian mentors.

1. Set the package version

   ```sh
   pkg_version=$(git tag --list "upstream/*" | tail -n 1 | sed -e "s:upstream/::")
   ```

1. Remove the snapshot artifacts

   ```sh
   rm ../${pkg_name}*_${pkg_version}*gbp*
   ```

1. Sign the package:

   ```sh
   debsign -S
   ```

1. Send the package to the debian repository:

   ```sh
   dput \
     mentors \
     ../${pkg_name}_${pkg_version}*source.changes
   ```
