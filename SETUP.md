This document describes the steps required to prepare
the environment in charge of handling the debian packages.

## Software installation

1. Install the following debian packages:
   ```
   sudo apt install --yes \
     debmake \
     debspawn \
     git-buildpackage \
     reprepro
   ```

## Software configuration

Run the following commands:
1. Session variables
   ```
   cat << 'EOF' >> $HOME/.bashrc

   DEBFULLNAME="Robin Alexander"
   DEBEMAIL="robin.alexander@netplus.ch"
   export DEBEMAIL DEBFULLNAME
   EOF
   ```
1. git
   ```
   git config --global user.name="Robin Alexander"
   git config --global user.email="robin.alexander@netplus.ch"
   ```
1. git-buildpackage
   ```
   cat << 'EOF' > $HOME/.gbp.conf
   [DEFAULT]
   pristine-tar = True
   upstream-branch = upstream/latest
   EOF
   ```
1. dput
   ```
   cat << 'EOF' > $HOME/.dput.cf
   [mentors]
   fqdn = mentors.debian.net
   incoming = /upload
   method = https
   allow_unsigned_uploads = 0
   progress_indicator = 2
   allowed_distributions = .*
   EOF
   ```
1. Sign-off and sign-on again

## Create the debian build environments

1. Create a dedicated tree structure:
   ```
   mkdir -p $HOME/odr-mmbtools/build-area/{unstable,bookworm,bullseye}
   ```
1. Create the debian build environments for all the distributions and architectures tracked by the Opendigitalradio debian repository:
   ```
   for dist in bullseye bookworm unstable; do
     debspawn create ${dist}
   done
   ```
