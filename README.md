Administering guix
==================

Installing, Configuring & Deploying guix

Deploying Base Machine (Mango/Catalpa)
--------------------------------------

-   Building on minimally configued CentOS7 (TBW: jenny? define
    minimal - ie.. missing help2man inter alia)

packages installed by Jenny/Dustin on mango at my request
---------------------------------------------------------

these packages are on top of the original base install image

``` {.bash}
sudo yum install libtk (needed by R library MASS and expected to be found on local host).
```

Install build preconditions on GUIX~SRVR~
-----------------------------------------

In particular, package abrt-console-notification needs to be current
(which fixes recent recognized upstream bug:
<https://bugzilla.redhat.com/show_bug.cgi?id=1139001> with a
corresponding fix: <https://rhn.redhat.com/errata/RHBA-2015-0556.html>)

This will be included by a current

``` {.bash}
yum -y update
```

<div>

<span class="label">Install required vendor packages</span>
``` {.bash}
sudo yum -y groupinstall "X Window System" "Desktop" "Fonts" "General Purpose Desktop"
sudo yum -y install autoconf 
sudo yum -y install automake
sudo yum -y install bzip2
sudo yum -y install gettext 
sudo yum -y install gettext-devel # nb gettext on centOS which is at latest already!g
sudo yum -y install gnupg
sudo yum -y install gnutls
sudo yum -y install graphviz
sudo yum -y install guile
sudo yum -y install guile-devel ## includes guile.m4, a dependency of guix ./bootstrap
sudo yum -y install help2man
sudo yum -y install libgcrypt-devel
sudo yum -y install make
sudo yum -y install pkgconfig 
sudo yum -y install sqlite
sudo yum -y install sqlite-devel
sudo yum -y install texinfo
sudo yum -y install texinfo-tex
```

</div>

TODO: could not get gnutls guile to work

in order for guile to pick up JSON, just
GUILE~LOADPATH~="/usr/local/share/guile/site:\$GUILE~LOADPATH~" guile
TODO?: re-install guile-json elsehow?

<div>

<span class="label">install gnutls</span>
``` {.bash}
wget ftp://ftp.gnutls.org/gcrypt/gnutls/v3.3/gnutls-3.3.16.tar.xz
tar xf gnutls-3.3.16.tar.xz
cd gnutls-3.3.16
  --with-libdir=lib64 \
./configure \
  --with-guile-site-dir=no \  ## This will instruct GnuTLS to
                      ## attempt to install the Guile
                      ## bindings where Guile will
                      ## look for them. It will use
                      ## guile-config info pkgdatadir
                      ## to learn the path to use make
# sudo ldconfig suggested by https://lists.gnupg.org/pipermail/gnutls-help/2013-May/003145.html without which
# make errors with:
#../lib/.libs/libgnutls.so: undefined reference to `nettle_secp_224r1'
#../lib/.libs/libgnutls.so: undefined reference to `nettle_secp_192r1'

make
make test
make install

```

</div>

<div>

<span class="label">install guile</span>
``` {.bash}
git clone git://git.savannah.nongnu.org/guile-json.git
cd guile-json
autoreconf -i           #  kudos http://askubuntu.com/questions/27677/cannot-find-install-sh-install-sh-or-shtool-in-ac-aux
./configure
make
sudo make install
```

</div>

Configure NFS share
-------------------

<div>

<span class="label">create NSF exported \${GNU} - TODO TBW by
jenny?</span>
``` {.bash}


```

</div>

``` {.bash}
cat <<'EOF' > guix.init.sh
GUIX_SRVR=catalpa.sgc.loc
GNU=/gnu
export PATH=${GNU}/bin:"${PATH}"
export MANPATH=${GNU}/bin:"${PATH}"
GUIX_PROFILE="$HOME/.guix-profile" \
source "$HOME/.guix-profile/etc/profile"
#export PATH="${HOME}/.guix-profile/bin:${PATH}"
#export MANPATH="${HOME}/.guix-profile/man"
EOF
```

NOTEs:

-   /gnu must be mounted without root squashing
-   \`make check\` fails in NFS mount home since default file perms
    include NFS~FACL~ of 'x' (which is arguable wrong). So, build and
    test/check is happening (for now) in /tmp (which has advantage
    anyway of being faster)
-   deploy to /gnu mounted as network share
-   appropriate configure and/or make options

<div>

<span class="label">install guix, following
<http://www.gnu.org/software/guix/download/></span>
``` {.bash}

#ssh mango
ssh catalpa
bld=/tmp/mec/sce
mkdir -p ${bld}
cd ${bld}

################################################################################
# binary (re)installation - following http://www.gnu.org/software/guix/manual/html_node/Binary-Installation.html with changes for shared /gnu
systemctl stop guix-daemon.service # allows to close any files which might be open in the store so they can be deleted
VAR=/var
NIX_STATE_DIR=${VAR}/guix
cd /tmp
rm -rf ./var ./gnu
wget ftp://alpha.gnu.org/gnu/guix/guix-binary-0.8.3.x86_64-linux.tar.xz
tar xf guix-binary-0.8.3.x86_64-linux.tar.xz --warning=no-timestamp
rm -rf /var/guix /gnu/var/guix 
rm -rf /gnu/* 
mv gnu/* /gnu # which currently moves only /gnu/store
mv var/guix ${VAR} # instead of localhost
##adjust symlinks
ln -sf -T ${VAR}/guix/profiles/per-user/root/guix-profile-1-link ${VAR}/guix/profiles/per-user/root/guix-profile 
ln -sf -T ${VAR}/guix/profiles ${VAR}/guix/gcroots/profiles
ln -sf ${VAR}/guix/profiles/per-user/root/guix-profile ~root/.guix-profile
# change the name of the group
perl -pi -e 's/guixbuild/guix-builder/g'  ~root/.guix-profile/lib/systemd/system/guix-daemon.service 
# install it as a service
/bin/cp -f ~root/.guix-profile/lib/systemd/system/guix-daemon.service /etc/systemd/system
# start the daemon as a service
systemctl start guix-daemon.service
# alternatively, run the daemon from the command line
#~root/.guix-profile/bin/guix-daemon --build-users-group=guix-builder
# make command available to all users:
mkdir /gnu/bin
export PATH=/gnu/bin:"${PATH}"
ln -s -f -t /gnu/bin  ${VAR}/guix/profiles/per-user/root/guix-profile/bin/guix
guix archive --authorize < ~root/.guix-profile/share/guix/hydra.gnu.org.pub
################################################################################
## first packages
guix package -i glibc-utf8-locales ## glibc-locales
export LOCPATH=$HOME/.guix-profile/lib/locale
guix package -i fontconfig
guix package -i gs-fonts
guix package -i font-dejavu
guix package -i font-gnu-freefont
guix package -i emacs
guix package -i gdk-pixbuf # needed by emacs 
export PATH="/root/.guix-profile/bin:/root/.guix-profile/sbin:${PATH}"
gdk-pixbuf-query-loaders  --update-cache
guix package -i libcanberra # needed by emacs - along with:
export GTK_PATH=/gnu/store/*-libcanberra-0.30/lib/gtk-3.0/modules

if [ -z "$GTK_MODULES" ] ; then     
        GTK_MODULES="libcanberra-gtk-module"
else
        GTK_MODULES="$GTK_MODULES:libcanberra-gtk-module"
fi

guix package -i gnome-icon-theme

################################################################################
guix build samtools

################################################################################

wget ftp://alpha.gnu.org/gnu/guix/guix-0.8.3.tar.gz.sig
tar xf guix-0.8.3.tar.gz
cd guix-0.8.3
################################################################################
# first time git clone git://git.savannah.gnu.org/guix.git 
cd guix
git pull

################################################################################

make clean
./bootstrap
#./configure --localstatedir=/gnu/var --exec-prefix=/gnu  
./configure --prefix=/gnu   ## --localstatedir=/gnu/var --exec-prefix=/gnu  
j=40
make -j $J
#make doc/guix.info # allowing: info -f doc/guix.info
make doc/guix.pdf # requires texi2dvi
make doc/guix.html
make -j $j check    

# mv guix-0.8.3 ~/project/sce/ # from tmp into project
# rm -rf  guix-0.8.3* & # clean up tmp
# chmod -R u=rwx  guix-0.8.3 # some didn't want to delete
# rm -rf  guix-0.8.3* & # clean up tmp

## some temp dirs 
##su - guix # become user guix
##cd ~mec/project/sce/guix-0.8.3

ssh catalpa
sudo su 
bld=/tmp/mec/sce
mkdir -p ${bld}
cd ${bld}
rm -rf guix
scp -q -r mec@mango:/tmp/mec/sce/guix .
cd guix
make install  # any value to separately install-data install-exec 

export PATH=/gnu/bin:"${PATH}"

guix archive --authorize < ~root/.guix-profile/share/guix/hydra.gnu.org.pub

systemctl start guix-daemon.service


# start the daemon

#killall guix-daemon # in case already running (i.e. we're developing this install/configure recipe)
#nohup /gnu/bin/guix-daemon --build-users-group=guix-builder &
tail -f /root/nohup.out &
##&2> /var/log/guix-daen.log & ## TODO: get into system.d - have it log - with rotation
systemctl restart guix-daemon.service
systemctl status guix-daemon.service

#rm -rf /gnu/var/guix/profiles/per-user ## the doc says this should happen by the daemon but not!  FIXME! BUG?
mkdir /gnu/var/guix/profiles/per-user ## the doc says this should happen by the daemon but not!  FIXME! BUG?
chmod a+w /gnu/var/guix/profiles/per-user

exit # return to be mec on catalpa
export PATH=/gnu/bin:"${PATH}"
guix build hello
guix package -i hello
# 

guix package -i socat

sudo su
guix package -i socat

## on GUIX_SRVR


/root/.guix-profile/bin/socat TCP4-LISTEN:9999  UNIX:/var/guix/daemon-socket/socket

## On a client node where /gnu is mounted read-write I ran this:

export GUIX_DAEMON_SOCKET=/gnu/var/guix/daemon-socket/`hostname`-socat  & socat UNIX-LISTEN:${GUIX_DAEMON_SOCKET}  TCP4:guix-builder:9999

##socat UNIX-LISTEN:/home/rwurmus/foo TCP4:guix-builder:9999 &    export GUIX_DAEMON_SOCKET=$HOME/foo

At this point I could use

    guix build hello
    guix environment hello


```

</div>

TODO: ensure to use the branch corresponding to release

Considerations: ./configure options:
-   localstatedir " is the value passed to configure as --localstatedir"
    per convention defined in [GNU Coding Standards: Directory
    Variables](https://www.gnu.org/prep/standards/html_node/Directory-Variables.html)
-   [discussion archive](https://gnunet.org/bot/log/guix/2015-02-10) -
    -   it should be shared - is /gnu/var

This recipe has me
-   build/test on mango (which is fast)
-   as mec
-   in /tmp (which is local and therefore does not have wonkey execute
    permission which causes \`make check\` to fail
-   copy to /tmp on catalpa (where I have root)
-   install from catalpa as root

\#\#localstatedir NIX~STATEDIR~=/gnu/var

\> Alternately, you can also do: \> \> guix build
guix --with-source=/path/to/guix

\> One can run: \> \> GUIX~PROFILE~=\$HOME/.guix-profile .
\~/.guix-profile/etc/profile

TODO: there is also a \`guix pull\` command - what is that about

Configuration of quix
---------------------

Difference from <http://www.gnu.org/software/guix/manual/guix.html> :
-   /gnu/store is nfs mounted read/write everywhere
-   /gnu is owned by new user, guix (instead of root)
-   guix-daemon runs as guix (not root)

GUIX~DAEMONSOCKET~: "Actually, clients honor the (undocumented)
‘GUIX~DAEMONSOCKET~’ environment variables, so that’s one thing you
could use."

<div>

<span class="label">example of running a single test which proving the
install</span>
``` {.bash}
make check TESTS=tests/syscalls.scm 
```

</div>

<div>

<span class="label">Make the "builder users" and their group...</span>
``` {.bash}
sudo groupadd guix-builder # already exists

for i in `seq 1 10`; do
    sudo useradd -g guix-builder  -G guix-builder           \
                 -d /var/empty -s `which nologin`          \
                 -c "Guix build user $i" \
                 guix-builder$i;
  done

## NOT: in the above --system          \

# Make the /gnu/store directory, where packages are kept/built
#sudo -u guix mkdir -p /gnu/store

sudo -u guix chgrp -R guix-builder /gnu/store
sudo -u guix chmod -R 1775 /gnu/store

## TODO: do we really want/need to permissions to be 3775 - u=rwx,g=rwx,a=rx sticky and set gid
## chgrp -R guix-builder 

ls guix/tests/*.log | xargs -i echo "{}\n" && cat {} >> guix/test.logs

```

</div>

Notes:
-   purpose of builder user accounts is to allow the daemon process to
    offload package building while keeping things nicely contained

<div>

<span class="label">produce confirmatory report on the "builder
user"</span>
``` {.bash}
getent passwd guix
getent group guix-builder
getent passwd guix-builder1
```

</div>

<div>

<span class="label">Create startup script to deploy within
/etc/profile.d</span>
``` {.bash}
sudo cat <<EOF > sce_guix.sh
???
EOF
```

</div>

Deploy: quix
------------

While still 'experimenting', you might not yet want to take this next
'install' step:

Declare alternate location for storing (custom) packages:

``` {.bash}
gp=$(readline -m ./guix_package)
mkdir gp 
export GUIX_PACKAGE_PATH=${gp}:${GUIX_PACKAGE_PATH}
```

Advanced guix
-------------

### Creating a new recipe for installing a new application

TBW

### Using Emacs interface to guix

install guix.el following <https://github.com/alezost/guix.el>

Alternate installation notes
----------------------------

### bootstrapping

If guix is already installed, you could

``` {.bash}
guix package --install autoconf automake bzip2 gcc-toolchain gettext \
                             guile libgcrypt pkg-config sqlite

```

configure emacs
---------------

``` {.commonlisp}
(package-install 'geiser)
(require 'geiser)
(setq-default geiser-guile-load-path '("~/src/guix"))
```

### Publishing a list of installed packages to our confluence based software catalog

PLAN:

> How to coordinate guix/plack content with wiki user content:
>
> Have one wiki/confluence page for each scientific applications,
> systematiclly named after the app, created according to a template.
>
> Have special section of the page reserved for being written to by guix
>
> have tool to update/create wiki page for any guix managed app
>
> First time we run it it pre-creates all pages for guix managed apps.

### Notifying interested parties when packages are updated

Users 'self-select' by watching associated confluence page

Site configuration
------------------

Everyone needs following additional shell
Using guix
==========

The section presents how to use guix to learn about available
applications, and to get them loaded into your unix environment.

Using the \`guix package\` command
----------------------------------

You will most often use the \`quix package\` command to

### Learning what applications have already been installed.

TBW

### Getting details about an installed application.

TBW

### Learning which installed applications are in your current environment.

TBW

### Controlling which installed applications are in your current environment.

TBW

### Setting up a project specific environment.

TBW

### Learning what applications are available for installation

TBW

### Using guix instead of python's pip

TBW

### Enabling reproducible research

Why guix
========

TWB

guix is better than bio.brew
----------------------------

SIMR has gone through a variety of approaches to managing deployment and
documentation of a wide scientific applications over the years. Most
recently, we are using in-house developed bio-brew.

The following issues with current bio.brew approach should be addressed
in this effort.
-   obscure/opaque:
    -   it wraps/hides the actual commands that are used to
        configure/make/test/install. It does this in an attempts to be a
        useful abstraction, but in fact it is not. The recipes should
        either be in SHELL such as bash, or well documented interface to
        unix system.
    -   it only exposes selected parameters to install (i.e. can not
        cause make to run verbosely without changing the wrappers)
    -   it adds one more layer between source code and install with
        intervening **undocumented** variables for communicating state
        between recipes and the engine.
-   redundant
    -   recipes must explicitly list the files to be installed:
        /n/local/bin/bio.brew/recipes/blast+ which requires recipes to
        be re-written when upstream application changes to change
        binaries. Most install mantras have their own list. It should be
        sufficient to simply provide a PREFIX.
-   INCOMPLETE: does not
    -   run tests
    -   detect collisions (due to either multiple versions of a package,
        or multiple packages with identically named files)
    -   install manpages, libraries, (anything other than binaries)
        without special code.
    -   provide for environment configuration with shells other than
        bash (i.e. csh is bash only (i.e. for~env~ assumes bash)
    -   allow swapping between alternate builds (i.e. deactivate not
        supported)
    -   does not handle versioned man pages - or manpages at all for
        that matter
    -   environment variables are NOT version specific - not possible to
        have different environments for each versioned install of an
        app.
    -   allow end-user control of apps in end-user environment - it is
        an all or none affair.
-   ISSUES:
    -   advises to 'bb remove full.pkg.name'
    -   the 'logs' directory contains non-logs
    -   logs are not version specific - installing/building a new
        version overwrites the log of any previous install
    -   uses wierd terminology - 'install' does not install typically it
        only builds.
    -   installs things using symlinks. This is unexpected by some apps,
        esp naively coded ones, such as university coded ones like rsem,
        and leads to long testing/debugging session.
-   TODO: contribute patch of rsem-calculate-expression to use
    FindBin::RealBin
-   GUIX deployment STATUS / notes to self
-   install from source or from 0.8.3 tarball
-   how to bootstrap network ready install?
-   if build, then ./configure --prefix=/gnu

or ./configure --localstatedir=/gnu/var --exec-prefix=/gnu
-   init.d system.d
-   still needed? mkdir /gnu/var/guix/profiles/per-user \#\# the doc
    says this should happen by the daemon but not! FIXME! BUG? chmod
    a+rwx /gnu/var/guix/profiles/per-user
-   TBW: transition plan?

<http://debbugs.gnu.org/cgi/bugreport.cgi?bug=20381>
-   "Interacting with a remote daemon" - socat test and report "If you
    could test this and provide feedback about the other options
    discussed there, that would be great (please email
    20381@debbugs.gnu.org.)"

We use 'guix' package manager to manage our CentOS-based bioinformatics
and other scientific computing applications. This document describes how
to this is done. The section "Everybody's guix" shows you the basic
package management method you need to see what applications are
installed and how to choose between them. "Installing, Configuring, and
Deploying guix" details the specifics of our site deployment. "Q/A"
provides answers to questions that were not readily apparent from a
first read of the documentation. "TODO:" lists local configurations and
integrations that, once performed, will make guix even more useful to
the administrator and end-user. "References & Links" provides pointers
found valuable during the initial study and deployment of guix.
Q/A
===

-   How is package recipe used for alternate versions of the package???
-   How does this approach track with changes in the recipe??
-   How are build options handled (i.e. \`make --with-gtk \`)???
-   TODO:
-   address all TODO/FIXME/TBW items.
-   provide answers to all the Q/A
-   enable/use emacs interface to guix package management
-   socat
-   reports of locally installed packages are posted on wiki
-   feed to install activity
-   method for categorizing packages using tags
-   Document: "Linux package management with guix"
    -   is available
        -   in github
        -   published to wiki
    -   is used as base material for series of workshops
-   References & Links
-   [GNU Guix Reference
    Manual](http://www.gnu.org/software/guix/manual/guix.html)
    ([pdf](https://www.gnu.org/software/guix/manual/guix.pdf))
-   [Reproducible and User-Controlled Software Environments in HPC with
    Guix](http://arxiv.org/abs/1506.02822)
-   [GNU Guix - Git Repositories on
    Savannah](http://savannah.gnu.org/git/?group=guix)
-   [GNU Guix Package
    List](https://www.gnu.org/software/guix/package-list.html) - lists
    packages that are part of base guix (you get 'out of the box') -
    includes a few bioinfo packages - samtools, bwa, etc
-   [GNU's advanced distro and transactional package manager —
    GuixSD](https://www.gnu.org/software/guix/) - a GNU linux distro
    which uses guix as its package manager.

-   [A searchable package list](http://guix.mdc-berlin.de) - 'You can
    search for “bioinfo” and all packages in the bioinformatics module
    should be displayed. There are also some machine learning packages
    and statistics packages that are used in bioinformatics, but are not
    located in this module.'
-   dedicated build system for R packages and an importer for CRAN
    packages. Creating a Guix package expression from a CRAN package
    (e.g. DBI) now takes little more than this:

    guix import cran DBI

    The output is an expression that takes very little editing can be
    bound to a variable in statistics.scm. To add the package to Guix
    upstream then only requires submitting a simple patch to
    guix-devel@gnu.org.
-   [Emacs interface for Guix package
    manager](http://hackersome.com/p/alezost/guix.el) - out of date -
    screen shots
-   [rekado - GNU Guix in an HPC
    environment](http://elephly.net/posts/2015-04-17-gnu-guix.html) -
    blog post -
    -   refers to set of recipes [guix.git - guix bioinformatics source
        archive](http://git.savannah.gnu.org/cgit/guix.git/tree/gnu/packages/bioinformatics.scm)
    -   [guix-nonfree/bioinformatics-nonfree.scm](https://github.com/BIMSBbioinfo/guix-nonfree/blob/master/bimsb/packages/bioinformatics-nonfree.scm) -
        Ricardo Wurmus' repos of 'non-free' bioinfo apps including:
        DiNup, macs-1, tophat(!), viennarna
-   [GNU Guix initd
    scriptPrimary](https://gnunet.org/gnu-guix-initd-script) - HowTo:
    start guix daemon on system startup
-   [Guix package manager without "make
    install"](http://dustycloud.org/blog/guix-package-manager-without-make-install/) -
    HowTo: run guix out of the src tree without installing it
-   [Reproducible Development Environments with GNU
    Guix](http://dthompson.us/reproducible-development-environments-with-gnu-guix.html) -
    blog post -
    -   considers new(ish) \`guix environment\` command as replacement
        for Python's pip/virtualenv, PHP's composer, node.js' npm
    -   demos a package which does not have any source code of its own -
        it only has run-time preconditions of other packages which must
        be "on path". (this is what spack called a "meta-package")
    -   demos usage of \`--pure\` option
-   <https://github.com/pjotrp/guix-notes> includes sections on:
    -   on installation
    -   [Hacking Guix](https://github.com/pjotrp/guix-notes):
    -   current notes / bioinformatics / hacking tips /
    -   lots on ruby gems
-   [[<https://github.com/BIMSBbioinfo/guix/blob/master/gnu/packages/bioinformatics.scm>][(define-module
    (gnu packages bioinformatics)] - bioinfo repos - now taken down -
    including: bedops, bedtools, python2-pybedtools, bowtie,, bwa,
    python2-bx-python, clipper, clustal-omega,
    crossmap,cutadapt,flexbar, hisat, htseq, htsjdk, macs, miso,....
    samtools, seqan, star, shogun, vcftools,

-   Notes/Suggestions/Best Practices:

> snippets - things poelple do
>
> ./pre-inst-env guix build ipcalc --keep-failed ;;; note: source file
>
> \$
> ./configure --with-libgcrypt-prefix=\$HOME/.guix-profile/ --localstatedir=/var
>
> --prefix=/tmp

nderstand: binary deployment (when not altering: –with-store-dir OR
–localstatedir)

-   CURRENTLY: Guix is not designed to be run in a centralised manner. A
    Guix daemon is supposed to run on each system as root and it listens
    to RPCs from local users only. In an environment with multiple
    clusters and multiple workstations this approach requires
    considerable effort to qmake it work correctly and securely.

[guix
publish](http://www.gnu.org/software/guix/manual/guix.html#Invoking-guix-publish) -
consider using

guix export & import - can translate nix pkgs and be used to move
packages across machines/installations.

``` {.bash}
git config --global user.name "Cook, Malcolm"
git config --global user.email "MEC@stowers.org"

#cd existing-project
git init
git add SCE.org # --all
git commit -m "Initial Commit"
git remote add origin http://MEC@stash/scm/cbio/sce.git
git push origin master
```
