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
     --upstream-branch=upstream/latest \
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
     --git-export-dir=../build \
     --git-build="sbuild --build-dir=../build" \
     --git-ignore-new
   ```
1. Verify the results from lintian, fix the problems if any and repeat the 
previous build until you are satisfied
1. Commit the changes:
   ```
   git add debian/
   git commit -m "Initial release"
   ```
1. Create the final build and tag the debian release:
   ```
   gbp buildpackage \
     --git-debian-branch=debian/latest \
     --git-export \
     --git-export-dir=../build \
     --git-build="sbuild --build-dir=../build" \
     --git-tag
   ```
1. Sign the package:
   ```
   debsign ../build/${mmbtool_name}_${mmbtool_version}*.changes
   ```
1. Send the package to the debian repository:
   ```
   dput \
     -f \
     mentors \
     ../build/${mmbtool_name}_${mmbtool_version}*.changes
   ```

## Push the repository to salsa.debian.org
Once the package is included in unstable, you can update the git repository in salsa.debian.org

1. Create the remote repository with all branches
   ```
   git push --all --set-upstream git@salsa.debian.org:ralex/${mmbtool_name}.git
   ```
1. Push all the tags to the remore repository
   ```
   git push --tags
   ```

## Create the initial debian package for stable-backports
As of June 2024, stable-backports points to `bookworm-backports`.

1. Create and checkout the bookworm-backports branch:
   ```
   git checkout debian/latest
   git checkout -b debian/bookworm-backports
   ```
1. Update the `debian/changelog` file:
   ```
   gbp dch \
     --bpo \
     --debian-branch=debian/bookworm-backports \
     --distribution=bookworm-backports
   ```
1. Apply the checklist
1. Build the debian package:
   ```
   gbp buildpackage \
     --git-debian-branch=debian/bookworm-backports \
     --git-export \
     --git-export-dir=../build \
     --git-build="sbuild --build-dir=../build" \
     --git-ignore-new
   ```
1. Verify the results from lintian, fix the problems if any and repeat the 
previous build until you are satisfied
1. Commit the changes
1. Commit the changes in `debian/changelog`
   ```
   git add debian/changelog
   git commit -m "Rebuild for bookworm-backports"
   ```
1. Tag the debian release:
   ```
   gbp buildpackage \
     --git-debian-branch=debian/bookworm-backports \
     --git-tag-only
   ```
1. Sign the package:
   ```
   debsign ../build/${mmbtool_name}_${mmbtool_version}*source.changes
   ```
1. Send the package to the debian repository:
   ```
   dput \
     -f \
     mentors \
     ../build/${mmbtool_name}_${mmbtool_version}*source.changes
   ```
