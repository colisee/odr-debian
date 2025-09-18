# Backports

Once your debian package is available in `testing`, you can create the
package for backports.

1. Set the distribution name

   ```sh
   distrib="trixie-backports"
   ```

1. Update the sbuild environment:

   ```sh
   sudo sbuild-update \
     --update \
     --dist-upgrade \
     --clean \
     --autoclean \
     --autoremove ${distrib}
   ```

1. Switch to the backports branch and merge:

   ```sh
   git checkout debian/${distrib}
   if ! [ $? ]; then
     git checkout debian/latest
     git checkout -b debian/${distrib}
   else
     git merge \
       -Xtheirs \
       $(git tag --list "debian/*-?" | tail -n 1)
   fi
   ```

1. Update the `debian/changelog` file:

   ```sh
   gbp dch \
     --bpo \
     --debian-branch=debian/${distrib} \
     --distribution=${distrib}
   ```

1. Check the backports release number in `debian/changelog` and fix it, if
applicable

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
