# Update the git repository on salsa.debian.org

This document describes the steps necessary to create or update the debian
package source repository in <https://salsa.debian.org>

1. If the repository does not exist in salsa, yet

   ```sh
   git push --all --set-upstream git@salsa.debian.org:ralex/${pkg_name}.git
   git push --tags
   ```

1. If the repository already exists in salsa

   ```sh
   git push --all
   git push --tags
   ```
