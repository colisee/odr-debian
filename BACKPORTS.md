This document describes the steps required to create the
debian package for a backports distribution.

As of June 2024, stable-backports points to `bookworm-backports`.

1. If the backports branch does not exist:
   ```
   git checkout debian/latest
   git checkout -b debian/bookworm-backports
   ```
1. If the backports branch exists:
   ```
   git checkout debian/bookworm-backports
   git merge debian/latest
   ```

1. Update the `debian/changelog` file:
   ```
   gbp dch \
     --bpo \
     --debian-branch=debian/bookworm-backports \
     --distribution=bookworm-backports
   ```
1. Apply the checklist
1. Build the debian package:
   ```
   gbp buildpackage \
     --git-debian-branch=debian/bookworm-backports \
     --git-export \
     --git-export-dir=../build \
     --git-build="sbuild --build-dir=../build" \
     --git-ignore-new
   ```
1. Verify the results from lintian, fix the problems if any and repeat the 
previous build until you are satisfied
1. Commit the changes
1. Commit the changes in `debian/changelog`
   ```
   git add debian/changelog
   git commit -m "Rebuild for bookworm-backports"
   ```
1. Tag the debian release:
   ```
   gbp buildpackage \
     --git-debian-branch=debian/bookworm-backports \
     --git-tag-only
   ```
1. Sign the package:
   ```
   debsign ../build/${mmbtool_name}_${mmbtool_version}*source.changes
   ```
1. Send the package to the debian repository:
   ```
   dput \
     -f \
     mentors \
     ../build/${mmbtool_name}_${mmbtool_version}*source.changes
   ```
