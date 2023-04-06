This document describes the steps required to create the
initial debian package from scratch.

## Create the initial debian package for unstable

1. Create the initial debianized git environment:
   ```
   mmbtool_name=odr-audioenc
   mmbtool_version=3.3.1
   mmbtool_dir="${HOME}/odr-mmbtools/${mmbtool_name}"
   upstream="https://github.com/Opendigitalradio/${mmbtool_name}/archive/refs/tags/v${mmbtool_version}.tar.gz"

   mkdir -p "${mmbtool_dir}"
   cd "${mmbtool_dir}"
   wget \
     --output-document="../${mmbtool_name}_${mmbtool_version}.tar.gz" \
     ${upstream}
   git init
   distrib=unstable
   gbp import-orig \
     --no-interactive \
     --upstream-branch=upstream/latest \
     --debian-branch=${distrib}/latest \
     "../${mmbtool_name}_${mmbtool_version}.tar.gz"
   ```
1. Add the debian template files:
   ```
   debmake \
     --package ${mmbtool_name} \
     --upstreamversion ${mmbtool_version}
   ```
1. Review and complete the content of the debian directory. Check the [Guide for Debian Maintainer](https://www.debian.org/doc/manuals/debmake-doc/index.en.html)
1. Build the debian package:
   ```
   gbp buildpackage \
     --git-builder="debspawn build --results-dir=$HOME/odr-mmbtools/build-area/${distrib} --lintian ${distrib}" \
     --git-export=WC \
     --git-export-dir="$HOME/odr-mmbtools/build-area"
   ```
1. Verify the results from lintian, fix the problems if any and repeat the 
previous build until you are satisfied
1. Commit and tag the changes
   ```
   git add debian/
   git commit -m "Initial debian package for ${distrib}"
   ```
1. Build and sign the package
   ```
   gbp buildpackage \
     --git-builder="debspawn build --results-dir=$HOME/odr-mmbtools/build-area/${distrib} --sign ${distrib}" \
     --git-debian-tag="${distrib}/%(version)s" \
     --git-tag \
     --git-debian-branch=${distrib}/latest \
     --git-export-dir="$HOME/odr-mmbtools/build-area"
   ```
1. Send the package to the debian repository
   ```
   dput \
     mentors \
     $HOME/odr-mmbtools/build-area/${distrib}/${mmbtool_name}*.changes
   ```

## Create the initial debian package for stable (ex: bullseye)

1. Create a new branch
   ```
   distrib=bullseye
   git checkout unstable/latest
   git checkout -b ${distrib}/latest
   ```
1. Change the debian/changelog file
   ```
   sed -e "s/unstable/${distrib}/g" -i "debian/changelog"
   ```
1. Build the debian package:
   ```
   gbp buildpackage \
     --git-builder="debspawn build --results-dir=$HOME/odr-mmbtools/build-area/${distrib} --lintian ${distrib}" \
     --git-export=WC \
     --git-export-dir="$HOME/odr-mmbtools/build-area"
   ```
1. Verify the results from lintian, fix the problems if any and repeat the 
previous build until you are satisfied
1. Commit and tag the changes
   ```
   git commit -a -m "Initial debian package for ${distrib}"
   ```
1. Build and sign the package
   ```
   gbp buildpackage \
     --git-builder="debspawn build --results-dir=$HOME/odr-mmbtools/build-area/${distrib} --sign ${distrib}" \
     --git-debian-tag="${distrib}/%(version)s" \
     --git-tag \
     --git-debian-branch=${distrib}/latest \
     --git-export-dir="$HOME/odr-mmbtools/build-area"
   ```

## Push the repository to salsa.debian.org

1. Get back to the branch unstable/latest
   ```
   git checkout unstable/latest
   ```
1. Create the remote repository with all branches
   ```
   git push \
     --all \
     --set-upstream \
     git@salsa.debian.org:ralex/${mmbtool_name}.git
   ```
1. Push all the tags to the remore repository
   ```
   git push \
     --tags \
     git@salsa.debian.org:ralex/${mmbtool_name}.git
   ```

