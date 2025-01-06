This document describes the steps required to update
a debian package with a new upstream version.

## Common
1. Set the package name
   ```
   pkg_name=odr-audioenc
   ```
1. Set the package version
   ```
   pkg_version=3.3.1
   ```
1. Clone or update the remote repository
   ```
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
   ```
   distrib=unstable
   ```
1. Update the sbuild environment:
   ```
   sudo sbuild-update --update --dist-upgrade --clean --autoclean --autoremove ${distrib}
   ```
1. Switch to the debian/latest branch
   ```
   git checkout debian/latest
   ```
1. Update with the new upstream version
   ```
   gbp import-orig \
     --debian-branch=debian/latest \
     --uscan
   ```
1. Change the debian/changelog using temporary snapshots
   ```
   gbp dch \
     --debian-branch=debian/latest \
     --snapshot
   ```
1. Build the debian package:
   ```
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
   ```
   gbp dch \
     --debian-branch=debian/latest \
     --distribution=${distrib} \
     --urgency=low \
     --commit \
     --release
   ```
1. Create the final build and tag the release:
   ```
   gbp buildpackage \
     --git-debian-branch=debian/latest \
     --git-tag
   ```
1. [Send the package to debian mentors](MENTORS.md)
1. Once the package lands in the testing/backports repository,
[push the local repository to debian salsa](SALSA.md)

## backports
1. Set the distribution name
   ```
   distrib=bookworm-backports
   ```
1. Update the sbuild environment:
   ```
   sudo sbuild-update --update --dist-upgrade --clean --autoclean --autoremove ${distrib}
   ```
1. Switch to the backports branch:
   ```
   git checkout debian/bookworm-backports
   ```
1. Merge with the latest branch:
   ```
   git merge \
     -Xtheirs \
     $(git tag --list "debian/${pkg_version}*" | tail -n 1)
   ```
1. Update the `debian/changelog` file:
   ```
   gbp dch \
     --bpo \
     --debian-branch=debian/${distrib} \
     --distribution=${distrib} \
     --urgency=low
   ```
1. Build the debian package:
   ```
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
   ```
   git add debian/changelog
   git commit -m "Rebuild for ${distrib}"
   ```
1. Tag the debian release:
   ```
   gbp buildpackage \
     --git-debian-branch=debian/${distrib} \
     --git-tag-only
   ```
1. [Send the package to debian mentors](MENTORS.md)
1. Once the package lands in the backports repository,
[push the local repository to debian salsa](SALSA.md)
