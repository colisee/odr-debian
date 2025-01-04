This document describes the steps required to update
a debian package with a new upstream version.

## unstable

1. Set the working variables
   ```
   mmbtool_name=odr-audioenc
   mmbtool_version=3.3.1
   mmbtool_dir="${HOME}/dev/debian/${mmbtool_name}"
   ```
1. Set the distribution name
   ```
   distrib=unstable
   ```
1. Clone or update the remote repository
   ```
   cd $(dirname "${mmbtool_dir}")
   if [ -d "${mmbtool_name}" ]; then
     cd ${mmbtool_dir}
     gbp pull
   else
     gbp clone \
       --all \
       git@salsa.debian.org:ralex/${mmbtool_name}.git
     cd ${mmbtool_dir}
   fi
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
1. Update the sbuild environment:
   ```
   sudo sbuild-update --update --dist-upgrade --clean --autoclean --autoremove ${distrib}
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
1. Verify the results from lintian, fix the problems if any and repeat the 
previous build until you are satisfied. Ignore issues with debian/changelog. For each fixed issue,
perform a git commit

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

## Update the debian package for backports

Once your debian package is available in `testing`, you can [update the 
package for backports](BACKPORTS.md)