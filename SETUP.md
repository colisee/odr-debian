This document describes the steps required to prepare
the environment in charge of handling the debian packages.

## Software installation

1. Install the following debian packages:
   ```
   sudo apt install --yes \
     debhelper \
     devscripts \
     sbuild \
     git-buildpackage \
     reprepro
   ```

## Software configuration

Run the following commands:
1. shell variables
   ```
   cat << 'EOF' >> $HOME/.bashrc

   DEBFULLNAME="Robin Alexander"
   DEBEMAIL="robin.alexander@netplus.ch"
   export DEBEMAIL DEBFULLNAME
   EOF
   ```
1. git
   ```
   git config --global user.name "Robin Alexander"
   git config --global user.email "robin.alexander@netplus.ch"
   ```
1. sbuild
   ```
   cat >~/.sbuildrc << 'EOF'
   ##############################################################################
   # PACKAGE BUILD RELATED (source-only-upload as default)
   ##############################################################################
   # -d
   $distribution = 'unstable';
   # -A
   $build_arch_all = 1;
   # -s
   $build_source = 1;
   # --source-only-changes
   $source_only_changes = 1;
   # -v
   $verbose = 1;

   ##############################################################################
   # POST-BUILD RELATED (turn off functionality by setting variables to 0)
   ##############################################################################
   $run_lintian = 1;
   $lintian_opts = ['--info', '--pedantic', '--show-overrides'];
   $run_piuparts = 0;
   $piuparts_opts = ['--schroot', 'unstable-amd64-sbuild'];
   $run_autopkgtest = 0;
   $autopkgtest_root_args = '';
   $autopkgtest_opts = [ '--', 'schroot', '%r-%a-sbuild' ];

   ##############################################################################
   # PERL MAGIC
   ##############################################################################
   1;
   EOF
   ```
1. git-buildpackage
   ```
   cat << 'EOF' > $HOME/.gbp.conf
   [DEFAULT]
   builder = sbuild
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

## Create the debian build environments

1. Create a dedicated tree structure:
   ```
   mkdir -p $HOME/odr-mmbtools/build-area/{unstable,bookworm,bullseye}
   ```
1. join the `sbuild` group
   ```
   sudo sbuild-adduser $LOGNAME
   sudo newgrp sbuild
   ```
1. Create the debian build environments for all the distributions and architectures tracked by the Opendigitalradio debian repository:
   ```
   for dist in bullseye bookworm unstable; do
     sudo sbuild-createchroot \
       --include=eatmydata,ccache \
       ${dist} \
       /srv/chroot/${dist}-amd64-sbuild \
       http://ftp.us.debian.org/debian
   done
   ```
