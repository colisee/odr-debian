This document describes the steps required to create the
initial debian package from scratch.

## Create the initial debian package for unstable

1. Set working variables
   ```
   mmbtool_name=odr-audioenc
   mmbtool_version=3.3.1
   mmbtool_dir="${HOME}/odr-mmbtools/${mmbtool_name}"
   upstream="https://github.com/Opendigitalradio/${mmbtool_name}/archive/refs/tags/v${mmbtool_version}.tar.gz"
   ```
1. Set distribution name
   ```
   distrib=unstable
   ```
1. Create the initial debianized git environment:
   ```
   mkdir -p "${mmbtool_dir}"
   cd "${mmbtool_dir}"
   wget \
     --output-document="../${mmbtool_name}_${mmbtool_version}.tar.gz" \
     ${upstream}
   git init
   gbp import-orig \
     --no-interactive \
     --debian-branch=debian/latest \
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
     --git-export=WC \
     --git-export-dir="$HOME/odr-mmbtools/build-area"
   ```
1. Verify the results from lintian, fix the problems if any and repeat the 
previous build until you are satisfied
1. Commit the changes:
   ```
   git add debian/
   git commit -m "Initial debian package for ${distrib}"
   ```
1. Add the debian tag:
   ```
   gbp tag --debian-branch=debian/latest
   ```
1. Sign the package:
   ```
   debsign ../build-area/${distrib}/${mmbtool_name}*.changes
   ```
1. Send the package to the debian repository:
   ```
   dput \
     -f \
     mentors \
     $HOME/odr-mmbtools/build-area/${distrib}/${mmbtool_name}-${mmbtool_version}*.changes
   ```

## Create the initial debian package for stable (ex: bullseye)

1. Set distribution name
   ```
   distrib=bullseye
   ```
1. Create a new branch
   ```
   git checkout debian/latest
   git checkout -b debian/${distrib}
   ```
1. Change the debian/changelog file (distribution and version)
   ```
   sed \
     -e "s/unstable/${distrib}/g" \
     -e "s/(\(.*\))/(\1~deb11u1)/g" \
     -i "debian/changelog"
   ```
1. Build the debian package:
   ```
   gbp buildpackage \
     --git-export=WC \
     --git-export-dir="$HOME/odr-mmbtools/build-area"
   ```
1. Verify the results from lintian, fix the problems if any and repeat the 
previous build until you are satisfied
1. Commit and tag the changes
   ```
   git commit -am "Initial debian package for ${distrib}"
   ```
1. Add the debian tag:
   ```
   gbp tag --debian-branch=debian/${distrib}
   ```
1. Sign the package:
   ```
   debsign --debs-dir ../build-area/${distrib}
   ```

## Push the repository to salsa.debian.org

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
