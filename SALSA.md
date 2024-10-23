This document describes the steps necessary to create or update the debian package source
repository in https://salsa.debian.org

You should follow these instructions only once the package is included in unstable

## Create the repository
1. Create the remote repository with all branches
   ```
   git push --all --set-upstream git@salsa.debian.org:ralex/${mmbtool_name}.git
   ```
1. Push all the tags to the remore repository
   ```
   git push --tags
   ```

## Update the repository
1. To update the existing repository in salsa
   ```
   gbp push --upstream-branch=upstream/latest --debian-branch=debian/latest --pristine-tar
   ```
