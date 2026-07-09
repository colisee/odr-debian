# Backports

Once your debian package is available in `testing`, you can create the
package for backports.

1. Set the package name

   ```sh
   pkg_name=odr-audioenc
   ```

1. Clone or update the remote repository

   ```sh
   if [ -d "${pkg_name}" ]; then
     cd "${pkg_name}"
     gbp pull
   else
     gbp clone \
       --all \
       git@salsa.debian.org:ralex/${pkg_name}
     cd "${pkg_name}"
   fi
   ```

1. Set the codename and distribution

   ```sh
   export codename="$(lsb_release --short --codename)"
   export distrib="${codename}-backports"
   ```

1. Update the sbuild environment:

   ```sh
   rm $HOME/.cache/sbuild/${distrib}-amd64.tar.zst
   mmdebstrap \
     --include=ca-certificates \
     --skip=output/dev \
     --variant=buildd \
     --customize-hook='echo "deb http://deb.debian.org/debian ${distrib} main contrib non-free non-free-firmware" > "$1/etc/apt/sources.list.d/backports.list"' \
     ${codename} \
     $HOME/.cache/sbuild/${distrib}-amd64.tar.zst \
     https://deb.debian.org/debian
   ```

1. Switch to the backports branch and merge:

   ```sh
   if [ $(git branch --list "debian/${distrib}" | wc --lines) -eq 0 ]; then
     git checkout debian/latest
     git checkout -b debian/${distrib}
   else
     git checkout "debian/${distrib}"
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
