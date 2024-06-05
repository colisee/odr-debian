This document describes the steps required to create the
initial debian package from scratch.

## Create the initial debian package for unstable

1. Set the tool name
   ```
   mmbtool_name=odr-audioenc
   ```
1. Set the tool version
   ```
   mmbtool_version=3.3.1
   ```
1. Set the distribution name
   ```
   distrib=unstable
   ```
1. Create the initial debianized git environment:
   ```
   mmbtool_dir="${HOME}/odr-mmbtools/${mmbtool_name}"
   upstream="https://github.com/Opendigitalradio/${mmbtool_name}/archive/refs/tags/v${mmbtool_version}.tar.gz"
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
   rm ../${mmbtool_name}_${mmbtool_version}*.tar.gz
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
     --git-debian-branch=debian/latest \
     --git-export \
     --git-export-dir=../build/${mmbtool_name} \
     --git-ignore-new
   ```
1. Verify the results from lintian, fix the problems if any and repeat the 
previous build until you are satisfied
1. Commit the changes:
   ```
   git add debian/
   git commit -m "Initial debian package"
   ```
1. Create the final build and tag the debian release:
   ```
   gbp buildpackage \
     --git-debian-branch=debian/latest \
     --git-export \
     --git-export-dir=../build/${mmbtool_name} \
     --git-tag
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

## Push the repository to salsa.debian.org

1. Create the remote repository with all branches
   ```
   git push --all --set-upstream git@salsa.debian.org:ralex/${mmbtool_name}.git
   ```
1. Push all the tags to the remore repository
   ```
   git push --tags
   ```
