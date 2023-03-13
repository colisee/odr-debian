This document describes the steps required to prepare
the environment in charge of handling the debian packages.

## Software installation

1. Install the following debian packages:
   ```
   sudo apt install --yes \
     binfmt-support \
     debmake \
     debspawn \
     git-buildpackage \
     qemu-user-static \
     reprepro
   ```

## Software configuration

Run the following commands:
1. Session variables
   ```
   cat << 'EOF' >> $HOME/.bashrc

   DEBFULLNAME="Robin ALEXANDER"
   DEBEMAIL="robin.alexander@netplus.ch"
   export DEBEMAIL DEBFULLNAME
   EOF
   ```
1. git
   ```
   git config --global user.name="Robin ALEXANDER"
   git config --global user.email="robin.alexander@netplus.ch"
   ```
1. devscripts
   ```
   cat << 'EOF' > $HOME/.devscripts
   DEBUILD_DPKG_BUILDPACKAGE_OPTS="-i -I -us -uc"
   DEBUILD_LINTIAN_OPTS="-i -I --show-overrides"
   EOF
   ```
1. git-buildpackage
   ```
   cat << 'EOF' > $HOME/.gbp.conf
   [DEFAULT]
   pristine-tar = True
   upstream-branch = upstream/latest
   EOF
   ```
1. Sign-off and sign-on again

## Create the debian build environments

1. Create a dedicated tree structure:
   ```
   mkdir -p $HOME/odr-mmbtools/build-area
   ```
1. Create the debian build environments for all the distributions and architectures tracked by the Opendigitalradio debian repository:
   ```
   for dist in bullseye unstable; do
     for arch in amd64 arm64 armhf; do
       debspawn create --arch=${arch} ${dist}
     done
   done
   ```
