# Environment initial setup

This document describes the steps required to prepare
the environment in charge of handling the debian packages.

## Software installation

1. Install the following debian packages

   ```sh
   sudo apt install --yes \
     debhelper \
     debmake \
     dput \
     git-buildpackage \
     sbuild
   ```

## Software configuration

Run the following commands:

1. Set shell variables

   ```sh
   cat >> $HOME/.profile << 'EOF'

   DEBFULLNAME="First_name Last_name"
   DEBEMAIL="Your_email"
   export DEBFULLNAME DEBEMAIL
   EOF
   ```

1. Set git defaults

   ```sh
   git config --global user.name "First_name Last_name"
   git config --global user.email "Your_email"
   ```

1. Set sbuild defaults

   ```sh
   tee $HOME/.sbuildrc << 'EOF'
   ##############################################################################
   # PACKAGE BUILD RELATED (source-only-upload as default)
   ##############################################################################
   # -s
   $build_source = 1;
   # --source-only-changes
   $source_only_changes = 1;

   ##############################################################################
   # POST-BUILD RELATED (turn off functionality by setting variables to 0)
   ##############################################################################
   $run_lintian = 1;
   $lintian_opts = ['-v', '-i', '-I', '-E', '--pedantic', '--profile', 'debian'];
   $run_piuparts = 0;
   $piuparts_opts = ['--schroot', '%r-%a-sbuild'];
   $run_autopkgtest = 0;
   $autopkgtest_root_args = '';
   $autopkgtest_opts = [ '--', 'schroot', '%r-%a-sbuild' ];

   ##############################################################################
   # PERL MAGIC
   ##############################################################################
   1;
   EOF
   ```

1. Set git-buildpackage defaults

   ```sh
   tee $HOME/.gbp.conf << 'EOF'
   [DEFAULT]
   builder = sbuild
   pristine-tar = True
   upstream-branch = upstream/latest
   EOF
   ```

1. Set dput defaults

   ```sh
   tee $HOME/.dput.cf << 'EOF'
   [mentors]
   fqdn = mentors.debian.net
   incoming = /upload
   method = https
   allow_unsigned_uploads = 0
   progress_indicator = 2
   allowed_distributions = .*
   EOF
   ```

1. Set devscripts defaults

   ```sh
   tee $HOME/.devscripts << 'EOF'
   USCAN_SYMLINK=rename
   EOF
   ```

## Create the debian build environments

1. Create a dedicated tree structure:

   ```sh
   mkdir -p $HOME/dev/debian
   ```

1. join the `sbuild` group

   ```sh
   sudo sbuild-adduser $LOGNAME
   sudo newgrp sbuild
   ```

1. Create the debian build environment for unstable:

   ```sh
   codename=unstable
   sudo sbuild-createchroot \
     --include=eatmydata,ccache \
     --alias=sid \
     --alias=UNRELEASED \
     ${codename} \
     /srv/chroot/${codename}-amd64-sbuild \
    http://deb.debian.org/debian
   ```

1. Create the debian build environment for the stable backports:

   ```sh
   codename=$(lsb_release --codename --short)
   sudo sbuild-createchroot \
     --extra-repository="deb http://deb.debian.org/debian ${codename}-backports main non-free" \
     --chroot-prefix=${codename}-backports \
     --include=eatmydata,ccache \
     ${codename} \
     /srv/chroot/${codename}-backports-amd64-sbuild \
    http://deb.debian.org/debian
   ```
