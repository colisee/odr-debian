This document describes the steps required to push the
debian package to debian mentors.

1. Sign the package:
   ```
   debsign ../build/${mmbtool_name}_${mmbtool_version}*.changes
   ```
1. Send the package to the debian repository:
   ```
   dput \
     -f \
     mentors \
     ../build/${mmbtool_name}_${mmbtool_version}*.changes
   ```
