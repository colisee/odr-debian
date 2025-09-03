# Update an existing package

This document describes the steps required to update
a debian package with a new upstream version.

## Common

1. Set the package name

   ```sh
   pkg_name=odr-audioenc
   ```

1. Set the package version

   ```sh
   pkg_version=3.3.1
   ```

1. Clone or update the remote repository

   ```sh
   pkg_dir="${HOME}/dev/debian/${pkg_name}"
   cd $(dirname "${pkg_dir}")
   if [ -d "${pkg_name}" ]; then
     cd "${pkg_dir}"
     gbp pull
   else
     gbp clone \
       --all \
       git@salsa.debian.org:ralex/${pkg_name}
     cd "${pkg_dir}"
   fi
   ```

## unstable

1. Set the distribution name

   ```sh
   distrib=unstable
   ```

1. Update the sbuild environment:

   ```sh
   sudo sbuild-update --update --dist-upgrade --clean --autoclean --autoremove ${distrib}
   ```

1. Switch to the debian/latest branch

   ```sh
   git checkout debian/latest
   ```

1. Update with the new upstream version

   ```sh
   gbp import-orig \
     --debian-branch=debian/latest \
     --uscan
   ```

1. Change the debian/changelog using temporary snapshots

   ```sh
   gbp dch \
     --debian-branch=debian/latest \
     --snapshot
   ```

1. Build the debian package:

   ```sh
   gbp buildpackage \
     --git-debian-branch=debian/latest \
     --git-ignore-new
   ```

1. Verify the results from lintian, fix the problems
if any and repeat the previous build until you are
satisfied. Ignore issues with debian/changelog. For each
fixed issue, perform a git commit

1. Verify the [Check-list](CHECKLIST.md) and perform a git commit, if applicable

1. Finalize the debian/changelog

   ```sh
   gbp dch \
     --debian-branch=debian/latest \
     --distribution=${distrib} \
     --urgency=low \
     --commit \
     --release
   ```

1. Create the final build and tag the release:

   ```sh
   gbp buildpackage \
     --git-debian-branch=debian/latest \
     --git-tag
   ```

1. [Send the package to debian mentors](MENTORS.md)

1. Once the package lands in the testing/backports repository,
[push the local repository to debian salsa](SALSA.md)

## backports

1. Set the distribution name

   ```sh
   distrib=bookworm-backports
   ```

1. Update the sbuild environment:

   ```sh
   sudo sbuild-update --update --dist-upgrade --clean --autoclean --autoremove ${distrib}
   ```

1. Switch to the backports branch:

   ```sh
   git checkout debian/bookworm-backports
   ```

1. Merge with the latest branch:

   ```sh
   git merge \
     -Xtheirs \
     $(git tag --list "debian/${pkg_version}*" | tail -n 1)
   ```

1. Update the `debian/changelog` file:

   ```sh
   gbp dch \
     --bpo \
     --debian-branch=debian/${distrib} \
     --distribution=${distrib} \
     --urgency=low
   ```

1. Build the debian package:

   ```sh
   gbp buildpackage \
     --git-debian-branch=debian/${distrib} \
     --git-ignore-new
   ```

1. Verify the results from lintian, fix the problems
if any and repeat the previous build until you are
satisfied. Ignore issues with debian/changelog. For each
fixed issue, perform a git commit
1. Verify the [Check-list](CHECKLIST.md) and perform a git commit, if applicable
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
