This document describes the steps required to update
a debian package with a new upstream version.

## Update unstable/latest

1. Set working variables
   ```
   mmbtool_name=odr-audioenc
   mmbtool_version=3.3.1
   mmbtool_dir="${HOME}/odr-mmbtools/${mmbtool_name}"
   ```
1. Set distribution name
   ```
   distrib=unstable
   ```
1. Clone or update the remote repository
   ```
   cd ${HOME}/odr-mmbtools
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
1. Change the debian/changelog file if a new upstream version was imported
   ```
   gbp dch \
     --debian-branch=debian/latest \
     --distribution=${distrib} \
     --urgency=low \
     --release
   ```
1. Build the debian package:
   ```
   gbp buildpackage \
     --git-builder="sbuild --dist=unstable --build-dir=$HOME/odr-mmbtools/build-area/unstable" \
     --git-debian-branch=debian/latest \
     --git-ignore-new
   ```
1. Verify the results from lintian, fix the problems if any and repeat the 
previous build until you are satisfied
1. Commit the changes if you modified any debian-related files
   ```
   git commit -am "Debian files changes induced by upstream ${mmbtool_version}"
   ```
1. Create the final build and tag the release:
   ```
   gbp buildpackage \
     --git-builder="sbuild --dist=unstable --build-dir=$HOME/odr-mmbtools/build-area/unstable" \
     --git-debian-branch=debian/latest \
     --git-tag
   ```
1. Sign the package:
   ```
   debsign ../build-area/${distrib}/${mmbtool_name}*.changes
   ```
1. Send the package to the debian repository
   ```
   dput \
     -f
     mentors \
     $HOME/odr-mmbtools/build-area/${distrib}/${mmbtool_name}*.changes
   ```
