This document describes the steps required to push the
debian package to debian mentors.

1. Remove the snapshot artifacts
   ```
   rm ../${pkg_name}_${pkg_version}*gbp*
   ```
1. Sign the package:
   ```
   debsign -S
   ```
1. Send the package to the debian repository:
   ```
   dput \
     ../${pkg_name}_${pkg_version}*source.changes
   ```
