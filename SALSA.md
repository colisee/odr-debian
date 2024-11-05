This document describes the steps necessary to create or update the debian package source
repository in https://salsa.debian.org

You should follow these instructions only once the package is included in unstable

## Create the repository
   ```
   git push --all --set-upstream git@salsa.debian.org:ralex/${mmbtool_name}.git
   git push --tags
   ```

## Update the repository
   ```
   git push --all
   git push --tags
   ```
