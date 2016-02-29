<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#orgheadline3">Making the case for adopting GNU Guix for Linux package management</a>
<ul>
<li><a href="#orgheadline1">GNU Guix - in its own words</a></li>
<li><a href="#orgheadline2">GNU Guix advantages broken down</a></li>
</ul>
</li>
<li><a href="#orgheadline4">GNU Guix advantages as compared to bio.brew</a></li>
<li><a href="#orgheadline19">Administration SIMR: Installing, Configuring &amp; Deploying Guix</a>
<ul>
<li><a href="#orgheadline5">Deploying Base Machine (Mango/Catalpa)</a></li>
<li><a href="#orgheadline6">Install build preconditions on GUIX_SRVR</a></li>
<li><a href="#orgheadline7">Configure NFS share</a></li>
<li><a href="#orgheadline8">Configuration of quix</a></li>
<li><a href="#orgheadline9">Deploy: quix</a></li>
<li><a href="#orgheadline12">Advanced Guix</a>
<ul>
<li><a href="#orgheadline10">Creating a new recipe for installing a new application</a></li>
<li><a href="#orgheadline11">Using Emacs interface to Guix</a></li>
</ul>
</li>
<li><a href="#orgheadline14">Alternate installation notes</a>
<ul>
<li><a href="#orgheadline13">bootstrapping</a></li>
</ul>
</li>
<li><a href="#orgheadline17">configure emacs</a>
<ul>
<li><a href="#orgheadline15">Publishing a list of installed packages to our confluence based software catalog</a></li>
<li><a href="#orgheadline16">Notifying interested parties when packages are updated</a></li>
</ul>
</li>
<li><a href="#orgheadline18">Site configuration</a></li>
</ul>
</li>
<li><a href="#orgheadline29">Using Guix</a>
<ul>
<li><a href="#orgheadline28">Using the `Guix package` command</a>
<ul>
<li><a href="#orgheadline20">Learning what applications have already been installed.</a></li>
<li><a href="#orgheadline21">Getting details about an installed application.</a></li>
<li><a href="#orgheadline22">Learning which installed applications are in your current environment.</a></li>
<li><a href="#orgheadline23">Controlling which installed applications are in your current environment.</a></li>
<li><a href="#orgheadline24">Setting up a project specific environment.</a></li>
<li><a href="#orgheadline25">Learning what applications are available for installation</a></li>
<li><a href="#orgheadline26">Using Guix instead of python's pip</a></li>
<li><a href="#orgheadline27">Enabling reproducible research</a></li>
</ul>
</li>
</ul>
</li>
<li><a href="#orgheadline34">References &amp; Links</a>
<ul>
<li><a href="#orgheadline30">Reference Manuals</a></li>
<li><a href="#orgheadline31">(White) Papers</a></li>
<li><a href="#orgheadline32">Package Sources</a></li>
<li><a href="#orgheadline33">Tips / Blogs</a></li>
</ul>
</li>
<li><a href="#orgheadline35">Q/A</a></li>
<li><a href="#orgheadline41">Deployment notes</a>
<ul>
<li><a href="#orgheadline36"><span class="todo nilTODO">TODO</span> :</a></li>
<li><a href="#orgheadline37">packages installed on mango that we would probably want to uninstall</a></li>
<li><a href="#orgheadline38">What is special about SIMR deployment</a></li>
<li><a href="#orgheadline39">notes to self</a></li>
<li><a href="#orgheadline40">Using git</a></li>
</ul>
</li>
</ul>
</div>
</div>


# Making the case for adopting GNU Guix for Linux package management<a id="orgheadline3"></a>

There are many approaches to software package management.  The reasons
for selecting GNU Guix are outlined in this section.

## GNU Guix - in its own words<a id="orgheadline1"></a>

The [introduction to the GNU Guix documentation](http://www.gnu.org/software/guix/manual/guix.html#Introduction) reads:

> "GNU Guix is a functional package management tool for the GNU (linux)
> system. Package management consists of all activities that relate to
> building packages from sources, honoring their build-time and run-time
> dependencies, installing packages in user environments, upgrading
> installed packages to new versions or rolling back to a previous set,
> removing unused software packages, etc.
> 
> The term functional refers to a specific package management discipline
> pioneered by Nix (see Acknowledgments). In Guix, the package build and
> installation process is seen as a function, in the mathematical
> sense. That function takes inputs, such as build scripts, a compiler,
> and libraries, and returns an installed package. As a pure function,
> its result depends solely on its inputs—for instance, it cannot refer
> to software or scripts that were not explicitly passed as inputs. A
> build function always produces the same result when passed a given set
> of inputs. It cannot alter the system’s environment in any way; for
> instance, it cannot create, modify, or delete files outside of its
> build and installation directories. This is achieved by running build
> processes in isolated environments (or containers), where only their
> explicit inputs are visible.
> 
> The result of package build functions is cached in the file system, in
> a special directory called the store. Each package is installed in a
> directory of its own, in the store—by default under /gnu/store. The
> directory name contains a hash of all the inputs used to build that
> package; thus, changing an input yields a different directory name.
> 
> This approach is the foundation of Guix’s salient features: support
> for transactional package upgrade and rollback, per-user installation,
> and garbage collection of packages (see Features)."

## GNU Guix advantages broken down<a id="orgheadline2"></a>

-   Guix package specifications, being "functional", <span class="underline">cannot</span> be
    incomplete; for instance, it is <span class="underline">impossible</span> for a deployed Guix
    package to inadvertently depend upon an aspect of the environment
    (such as a shared library) in which it was built and deployed.

-   Guix strictly controls the environment in which packages are built:
    builds always take place in a 'clean' environment (using the linux
    chroot mechanism, inter alia).  This makes it (near) impossible for
    any aspect of the package builders environment to inadvertently
    "leak" into the package binary.  This also makes it necessary, when
    (re-)packaging an application for Guix, to identify any
    incompletely specified packages (i.e. packages having tacit
    prerequisites).  Doing so as early as possible in a package's
    adoption will mitigate exposure to downstream errors.  This is of
    special advantage to novice and/or less-than-expert package
    developers and deployers.

-   Guix prohibits creating environments which load packages containing
    name collisions. This prevents the ambiguity and associated error
    that can arise when two packages define different programs by the
    same name and both are placed in the user's execution PATH.

-   Guix provides for transactional package upgrades and roll-backs.
    Being transactional, these operations either succeed in their
    entirety or they fail - partial upgrades or partial roll-backs are
    not possible, even in the face of process interruption or power
    failure.

-   Guix rollbacks allows easy testing of new package installation with
    a guaranteed means of rapid recovery should post-installation
    testing warrant.

-   Guix allows for the creation of multiple environment profiles,
    which may be per-user (typically), per group (i.e. lab), per
    application, or per project.  These profiles allow for isolation of
    the effects of package changes to only those subscribing to the
    profile.

-   Guix allows for unprivileged package management; users do not need
    special elevated privileges (root or sudo) to create custom
    environment profiles.  Especially noteworthy is that this allows
    for rationally sharing package management as a distributed
    responsibility.  This includes installing new site-available
    applications (for those having available recipes in the Guix
    repositories).  Thus, if one user observes that a new release of a
    site-installed application has become available, that user can
    safely install the upgrade centrally and immediately start using
    it, without effecting any other user's environment; without any
    further coordination, when another user <span class="underline">is</span> ready to adopt the
    upgrade, they will find that installation is unnecessary, as it has
    already occurred.

-   Guix explicitly represents package dependencies, allowing for a
    [garbage collection](https://www.gnu.org/software/guix/manual/html_node/Invoking-guix-gc.html#Invoking-guix-gc) process to identify and delete packages that are
    installed but no longer required; this allows for reclaiming
    storage, and reducing the size of an installed software catalog.

-   A Guix package, once installed in a profile, will not be altered,
    for example, by having one of the packages upon which it depends
    upgraded.

-   Guix abstracts the idea of [build systems](https://www.gnu.org/software/guix/manual/guix.html#Build-Systems) to not only encompass the
    standard Gnu recipe "(configure; make; make test; make install)"
    but to also extend to common language/tool specific
    package/module/add-on managers, including those for R, perl,
    python, ruby, haskell, and emacs.  This should allow a single
    command line protocol for managing the building and deployment of
    such packages.  Additionally, when deployed using guix, they can
    also be enabled with guix using the \`[environment](https://www.gnu.org/software/guix/manual/guix.html#Invoking-guix-environment)\` sub-command,
    providing a unified approach where previously multiple tools might
    have been employed (perlbrew for perl, pyenv or pythonbrew for
    python, rbenv for ruby, etc.)

-   Guix approach to re-packaging facilitates including exactly all the
    package's resources: man pages, libraries, as well as binaries.
    This is a result of their using the underlying/native build
    engine's (i.e. gnu make) installation method.

-   Guix packages, once compiled are independent on the host on which
    they are built, or its specific GNU/Linux distribution, or its
    version.  Thus, for example, packages built under CentOS 6.5 on
    x86\_64 will run under operating system on x86\_64, for example,
    CentOS 7.x and or Ubuntu 15.10 or 14.04.  (They may occasionally be
    dependent upon the machine architecture, i.e. for certain compiler
    optimizations).

-   Guix packages, being independent of host on which they are built,
    can be downloaded already built by upstream servers known as
    [substitutes](https://www.gnu.org/software/guix/manual/html_node/Substitutes.html#Substitutes), saving on the generally longer process of
    configuration and compilation on local servers.

-   Guix packages can be developed and deployed locally without
    necessarily contributing to the upstream open-source repository.
    This allows in-house/bespoke software to be deployed using Guix
    without forcing to release it publicly.

-   Guix facilitates building analysis pipelines that are reproducible
    locally (through environments or shared profiles) and can be shared
    with external collaborators (i.e. using guix's archive subcommand).

-   Guix intends to guarantee the reproducibility of a software
    environment across HPC systems; a computational pipeline should be
    portable into another Guix environment with the expectation that it
    will run identically.

-   Guix provides commands to [display package information](https://www.gnu.org/software/guix/manual/guix.html#Emacs-Commands) which can be
    used to automate the production and publishing of software package
    catalogs.  Such catalogs may then be shared, printed, emailed, or
    embedded into community-visible web pages as various means of
    advertising package availability to the research community.

-   Guix package specifications, being written in [Guile/scheme](https://www.gnu.org/software/guix/manual/guix.html#Defining-Packages), do not
    depend upon the users SHELL (i.e. guix works equally well with zsh,
    bash, tcsh, etc) (TODO: confirm).

-   Guix [simplifies package installation](https://www.gnu.org/software/guix/manual/guix.html#Invoking-guix-package): viz. \`guix package -i abyss\`
    serves to download, compile, test, and install the most recent
    version of abyss known to guix repository.

-   Guix has a future - it is the package manager for [GuixSD](https://www.gnu.org/software/guix/), the new
    GNU/Linux distribution backed by [the GNU project](https://www.gnu.org/).

-   Guix is [well documented](https://www.gnu.org/software/guix/manual/).

-   Guix features are under continued development, including tools for
    -   [production of graphs](https://www.gnu.org/software/guix/manual/guix.html#Invoking-guix-graph) for visualizing dependencies between packages
    -   [shell 'completion'](https://www.gnu.org/software/guix/manual/guix.html#Emacs-Completions) facilities
    -   [emacs integration](https://www.gnu.org/software/guix/manual/guix.html#Emacs-Interface) (optional) as interface for package development
        and management.

-   Guix has [rich community support](https://www.gnu.org/software/guix/contribute/); bugs will be fixed - features will
    be improved.

-   Guix community has already prepared [over 3,000 recipes](https://www.gnu.org/software/guix/packages/), of which
    currently [114 are bioinformatics](http://guix.mdc-berlin.de/packages?/?search=bioinfo) packages.

-   Guix packaging is relatively easy to learn. It is reasonably
    documented and there are \`lint\` style tools that check recipes for
    being well-structured; they identify common errors in package
    specification.

-   Guix development is open source.  It is open to input from all
    community members.  It is free software!

Guix provides a well-engineered, supportable, portable, and extensible
approach to software package and environment management. It allows to
adopt and create unambiguous, clearly-specified, stand-alone, and
reproducible software environments.  It provides the administrative
flexibility to deploy with transactional control a common standard
suite of applications, as well as the agility to quickly respond to
individual user and application requirements for more customized
environments.  Guix reduces the administrative burden for software
package and environment management by allowing it to be distributed
among researchers, with a natural means of coordination of effort.  By
offering a unified approach to environment management, it replaces the
need for multiple other such tools, reducing support overhead.  Guix
can serve the needs for computational reproducibility through means of
sharing these environments across a range of hosts and computer
operating systems with both internal collaborators as well as the
broader research community.

In short, Guix provides a rigorous response to requirements for
package and environment integrity in a rapidly evolving research
computing environment while easing and distributing the burden of
administration.

# GNU Guix advantages as compared to bio.brew<a id="orgheadline4"></a>

SIMR has gone through a variety of approaches to the package
management of a wide scientific applications over the years.
Currently, we are using [bio.brew](https://github.com/metalhelix/bio.brew) which was developed in-house by
former SIMR member Jim Vallandingham.

Many issues have been identified while working with bio.brew.  Those
marked with "!!!" might be considered egregious.  ALL should be
addressed in our move to Guix.  

Bio.brew is missing many features present in other package managers,
in that it does not:

-   provide any assurance that an application packaged with bio.brew
    is complete.  A negative consequence of this is that there are
    bio.brew managed applications which run on some hosts but not
    others (based on the availability of an installed library).
-   provide a means of representing dependencies between packages,
    thus none of the downstream advantages (garbage collection and
    visual graphing of dependencies) are possible.
-   !!! detect collisions (due to either multiple versions of a
    package, or multiple packages with identically named files)
-   !!! extend naturally to install manpages, libraries, (anything
    other than binaries) without special code.  !!!
-   !!! allow swapping between alternate builds (i.e. for starters
    \`bb deactivate <pkg>\` was never fully/correctly implemented)
-   !!! allow for version specific man pages or environment variables;
    so, it is not possible to have different environments for each
    versioned install of an app.
-   run package tests and protect against installing failing packages
-   provide for environment configuration with shells other than bash
    (i.e. csh is bash only (i.e. for\_env assumes bash
-   allow end-user control of apps in end-user environment - it is an
    all or none affair.
-   implement package "deactivation" - packages can be completely
    deleted only (remove).
-   provide for more than one environment - neither users, groups of
    users, applications can be provided custom environments.  It is
    an all-or-nothing proposition.
-   provide roll-back capabilities - returning to a prior version of
    a package requires deleting the current version and re-installing
    the old version (assuming the old version is still to be found)

-   bio.brew is obscure/opaque:
    -   it wraps/hides the actual shell commands that are used to
        configure/make/test/install.  It does this in an attempts to be a
        useful abstraction, but in fact it is not.  The recipes should
        either be in SHELL such as bash, or well documented interface to
        unix system.
    -   it is selective in which build parameters are exposed (i.e. can
        not cause make to run verbosely without changing the wrappers)
    -   it adds one more layer between source code and install with
        intervening !!!
    -   !!! it is poorly documented
    -   !!! it uses undocumneted variables for communicating state
        between recipes and the engine.
-   redundant
    -   !!! recipes must explicitly re-list the files to be installed which
        this is already typically done with the packages build script.
        /n/local/bin/bio.brew/recipes/blast+ which requires recipes to be
        re-written when upstream application changes to change binaries.
        Most install mantras have their own list.  It should be
        sufficient to simply provide a PREFIX.
-   ISSUES:
    -   the 'logs' directory contains non-logs
    -   logs are not version specific - installing/building a new version
        overwrites the log of any previous install
    -   uses wierd terminology - 'install' does not install typically it
        only builds.
    -   installs things using symlinks which is unexpected by some apps,
        esp naively coded ones, such as university coded ones like rsem,
        and leads to long testing/debugging session.
    -   bio.brew has no community
    -   bio.brew is end of life

# Administration SIMR: Installing, Configuring & Deploying Guix<a id="orgheadline19"></a>

## Deploying Base Machine (Mango/Catalpa)<a id="orgheadline5"></a>

-   Building on minimally configued CentOS7 (TBW: jenny? define
    minimal - ie.. missing help2man inter alia)

## Install build preconditions on GUIX\_SRVR<a id="orgheadline6"></a>

In particular, package abrt-console-notification needs to be current
(which fixes recent recognized upstream bug:
<https://bugzilla.redhat.com/show_bug.cgi?id=1139001> with a
corresponding fix: <https://rhn.redhat.com/errata/RHBA-2015-0556.html>)

This will be included by a current

    yum -y update

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
    sudo yum -y install guile-devel ## includes guile.m4, a dependency of Guix ./bootstrap
    sudo yum -y install help2man
    sudo yum -y install libgcrypt-devel
    sudo yum -y install make
    sudo yum -y install pkgconfig 
    sudo yum -y install sqlite
    sudo yum -y install sqlite-devel
    sudo yum -y install texinfo
    sudo yum -y install texinfo-tex

TODO: could not get gnutls guile to work

in order for guile to pick up JSON, just
GUILE\_LOAD\_PATH="/usr/local/share/guile/site:$GUILE\_LOAD\_PATH" guile
TODO?: re-install guile-json elsehow?

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

    git clone git://git.savannah.nongnu.org/guile-json.git
    cd guile-json
    autoreconf -i                   #  kudos http://askubuntu.com/questions/27677/cannot-find-install-sh-install-sh-or-shtool-in-ac-aux
    ./configure
    make
    sudo make install

## Configure NFS share<a id="orgheadline7"></a>


    cat <<'EOF' > Guix.init.sh
    GUIX_SRVR=catalpa.sgc.loc
    GNU=/gnu
    export PATH=${GNU}/bin:"${PATH}"
    export MANPATH=${GNU}/bin:"${PATH}"
    GUIX_PROFILE="$HOME/.Guix-profile" \
    source "$HOME/.Guix-profile/etc/profile"
    #export PATH="${HOME}/.Guix-profile/bin:${PATH}"
    #export MANPATH="${HOME}/.Guix-profile/man"
    EOF

NOTEs: 

-   /gnu must be mounted without root squashing
-   \`make check\` fails in NFS mount home since default file perms
    include NFS\_FACL of 'x' (which is arguable wrong).  So, build and
    test/check is happening (for now) in /tmp (which has advantage
    anyway of being faster)
-   deploy to /gnu mounted as network share
-   appropriate configure and/or make options

    #ssh mango
    ssh catalpa
    bld=/tmp/mec/sce
    mkdir -p ${bld}
    cd ${bld}
    
    ################################################################################
    # binary (re)installation - following http://www.gnu.org/software/Guix/manual/html_node/Binary-Installation.html with changes for shared /gnu
    systemctl stop Guix-daemon.service # allows to close any files which might be open in the store so they can be deleted
    VAR=/var
    NIX_STATE_DIR=${VAR}/Guix
    cd /tmp
    rm -rf ./var ./gnu
    wget ftp://alpha.gnu.org/gnu/Guix/Guix-binary-0.8.3.x86_64-linux.tar.xz
    tar xf Guix-binary-0.8.3.x86_64-linux.tar.xz --warning=no-timestamp
    rm -rf /var/Guix /gnu/var/Guix 
    rm -rf /gnu/* 
    mv gnu/* /gnu # which currently moves only /gnu/store
    mv var/Guix ${VAR} # instead of localhost
    ##adjust symlinks
    ln -sf -T ${VAR}/Guix/profiles/per-user/root/Guix-profile-1-link ${VAR}/Guix/profiles/per-user/root/Guix-profile 
    ln -sf -T ${VAR}/Guix/profiles ${VAR}/Guix/gcroots/profiles
    ln -sf ${VAR}/Guix/profiles/per-user/root/Guix-profile ~root/.Guix-profile
    # change the name of the group
    perl -pi -e 's/Guixbuild/Guix-builder/g'  ~root/.Guix-profile/lib/systemd/system/Guix-daemon.service 
    # install it as a service
    /bin/cp -f ~root/.Guix-profile/lib/systemd/system/Guix-daemon.service /etc/systemd/system
    # start the daemon as a service
    systemctl start Guix-daemon.service
    # alternatively, run the daemon from the command line
    #~root/.Guix-profile/bin/Guix-daemon --build-users-group=Guix-builder
    # make command available to all users:
    mkdir /gnu/bin
    export PATH=/gnu/bin:"${PATH}"
    ln -s -f -t /gnu/bin  ${VAR}/Guix/profiles/per-user/root/Guix-profile/bin/Guix
    Guix archive --authorize < ~root/.Guix-profile/share/Guix/hydra.gnu.org.pub
    ################################################################################
    ## first packages
    Guix package -i glibc-utf8-locales ## glibc-locales
    export LOCPATH=$HOME/.Guix-profile/lib/locale
    Guix package -i fontconfig
    Guix package -i gs-fonts
    Guix package -i font-dejavu
    Guix package -i font-gnu-freefont
    Guix package -i emacs
    Guix package -i gdk-pixbuf # needed by emacs 
    export PATH="/root/.Guix-profile/bin:/root/.Guix-profile/sbin:${PATH}"
    gdk-pixbuf-query-loaders  --update-cache
    Guix package -i libcanberra # needed by emacs - along with:
    export GTK_PATH=/gnu/store/*-libcanberra-0.30/lib/gtk-3.0/modules
    
    if [ -z "$GTK_MODULES" ] ; then     
            GTK_MODULES="libcanberra-gtk-module"
    else
            GTK_MODULES="$GTK_MODULES:libcanberra-gtk-module"
    fi
    
    Guix package -i gnome-icon-theme
    
    ################################################################################
    Guix build samtools
    
    ################################################################################
    
    wget ftp://alpha.gnu.org/gnu/Guix/Guix-0.8.3.tar.gz.sig
    tar xf Guix-0.8.3.tar.gz
    cd Guix-0.8.3
    ################################################################################
    # first time git clone git://git.savannah.gnu.org/Guix.git 
    cd Guix
    git pull
    
    ################################################################################
    
    make clean
    ./bootstrap
    #./configure --localstatedir=/gnu/var --exec-prefix=/gnu  
    ./configure --prefix=/gnu   ## --localstatedir=/gnu/var --exec-prefix=/gnu  
    j=40
    make -j $J
    #make doc/Guix.info # allowing: info -f doc/Guix.info
    make doc/Guix.pdf # requires texi2dvi
    make doc/Guix.html
    make -j $j check    
    
    # mv Guix-0.8.3 ~/project/sce/ # from tmp into project
    # rm -rf  Guix-0.8.3* & # clean up tmp
    # chmod -R u=rwx  Guix-0.8.3 # some didn't want to delete
    # rm -rf  Guix-0.8.3* & # clean up tmp
    
    ## some temp dirs 
    ##su - Guix # become user Guix
    ##cd ~mec/project/sce/Guix-0.8.3
    
    ssh catalpa
    sudo su 
    bld=/tmp/mec/sce
    mkdir -p ${bld}
    cd ${bld}
    rm -rf Guix
    scp -q -r mec@mango:/tmp/mec/sce/Guix .
    cd Guix
    make install  # any value to separately install-data install-exec 
    
    export PATH=/gnu/bin:"${PATH}"
    
    Guix archive --authorize < ~root/.Guix-profile/share/Guix/hydra.gnu.org.pub
    
    systemctl start Guix-daemon.service
    
    
    # start the daemon
    
    #killall Guix-daemon # in case already running (i.e. we're developing this install/configure recipe)
    #nohup /gnu/bin/Guix-daemon --build-users-group=Guix-builder &
    tail -f /root/nohup.out &
    ##&2> /var/log/Guix-daen.log & ## TODO: get into system.d - have it log - with rotation
    systemctl restart Guix-daemon.service
    systemctl status Guix-daemon.service
    
    #rm -rf /gnu/var/Guix/profiles/per-user ## the doc says this should happen by the daemon but not!  FIXME! BUG?
    mkdir /gnu/var/Guix/profiles/per-user ## the doc says this should happen by the daemon but not!  FIXME! BUG?
    chmod a+w /gnu/var/Guix/profiles/per-user
    
    exit # return to be mec on catalpa
    export PATH=/gnu/bin:"${PATH}"
    Guix build hello
    Guix package -i hello
    # 
    
    Guix package -i socat
    
    sudo su
    Guix package -i socat
    
    ## on GUIX_SRVR
    
    
    /root/.Guix-profile/bin/socat TCP4-LISTEN:9999  UNIX:/var/Guix/daemon-socket/socket
    
    ## On a client node where /gnu is mounted read-write I ran this:
    
    export GUIX_DAEMON_SOCKET=/gnu/var/Guix/daemon-socket/`hostname`-socat  & socat UNIX-LISTEN:${GUIX_DAEMON_SOCKET}  TCP4:Guix-builder:9999
    
    ##socat UNIX-LISTEN:/home/rwurmus/foo TCP4:Guix-builder:9999 &    export GUIX_DAEMON_SOCKET=$HOME/foo
    
    At this point I could use
    
        Guix build hello
        Guix environment hello

TODO: ensure to use the branch corresponding to release

Considerations:
 ./configure options:

-   localstatedir " is the value passed to configure as
    --localstatedir" per convention defined in [GNU Coding Standards:
    Directory Variables](https://www.gnu.org/prep/standards/html_node/Directory-Variables.html)
-   [discussion archive](https://gnunet.org/bot/log/Guix/2015-02-10) - 
    -   it should be shared -  is /gnu/var

This recipe has me 

-   build/test on mango (which is fast)
-   as mec
-   in /tmp (which is local and therefore does not have wonkey execute
    permission which causes \`make check\` to fail
-   copy to /tmp on catalpa (where I have root)
-   install from catalpa as root

\##localstatedir
NIX\_STATE\_DIR=/gnu/var

> Alternately, you can also do:
>
>   Guix build Guix --with-source=/path/to/Guix

> One can run:
>
>   GUIX\_PROFILE=$HOME/.Guix-profile . ~/.Guix-profile/etc/profile

TODO: there is also a \`Guix pull\` command - what is that about

## Configuration of quix<a id="orgheadline8"></a>

Difference from <http://www.gnu.org/software/Guix/manual/Guix.html> :

-   /gnu/store is nfs mounted read/write everywhere
-   /gnu is owned by new user, Guix (instead of root)
-   Guix-daemon runs as Guix (not root)

GUIX\_DAEMON\_SOCKET: "Actually, clients honor the (undocumented) ‘GUIX\_DAEMON\_SOCKET’
environment variables, so that’s one thing you could use."

    make check TESTS=tests/syscalls.scm

    sudo groupadd Guix-builder # already exists
    
    for i in `seq 1 10`; do
        sudo useradd -g Guix-builder  -G Guix-builder           \
                     -d /var/empty -s `which nologin`          \
                     -c "Guix build user $i" \
                     Guix-builder$i;
      done
    
    ## NOT: in the above --system          \
    
    # Make the /gnu/store directory, where packages are kept/built
    #sudo -u Guix mkdir -p /gnu/store
    
    sudo -u Guix chgrp -R Guix-builder /gnu/store
    sudo -u Guix chmod -R 1775 /gnu/store
    
    ## TODO: do we really want/need to permissions to be 3775 - u=rwx,g=rwx,a=rx sticky and set gid
    ## chgrp -R Guix-builder 
    
    ls Guix/tests/*.log | xargs -i echo "{}\n" && cat {} >> Guix/test.logs

Notes:

-   purpose of builder user accounts is to allow the daemon process to offload
    package building while keeping things nicely contained

    getent passwd Guix
    getent group Guix-builder
    getent passwd Guix-builder1

    sudo cat <<EOF > sce_Guix.sh
    ???
    EOF

## Deploy: quix<a id="orgheadline9"></a>

While still 'experimenting', you might not yet want to take this next
'install' step:

Declare alternate location for storing (custom) packages: 

    gp=$(readline -m ./Guix_package)
    mkdir gp 
    export GUIX_PACKAGE_PATH=${gp}:${GUIX_PACKAGE_PATH}

## Advanced Guix<a id="orgheadline12"></a>

### Creating a new recipe for installing a new application<a id="orgheadline10"></a>

TBW

### Using Emacs interface to Guix<a id="orgheadline11"></a>

install Guix.el following <https://github.com/alezost/Guix.el>

## Alternate installation notes<a id="orgheadline14"></a>

### bootstrapping<a id="orgheadline13"></a>

If Guix is already installed, you could

    Guix package --install autoconf automake bzip2 gcc-toolchain gettext \
                                 guile libgcrypt pkg-config sqlite

## configure emacs<a id="orgheadline17"></a>

    (package-install 'geiser)
    (require 'geiser)
    (setq-default geiser-guile-load-path '("~/src/Guix"))

### Publishing a list of installed packages to our confluence based software catalog<a id="orgheadline15"></a>

PLAN:

> How to coordinate Guix/plack content with wiki user content:
> 
> Have one wiki/confluence page for each scientific applications,
> systematiclly named after the app, created according to a template.
> 
> Have special section of the page reserved for being written to by Guix
> 
> have tool to update/create wiki page for any Guix managed app
> 
> First time we run it it pre-creates all pages for Guix managed apps.

### Notifying interested parties when packages are updated<a id="orgheadline16"></a>

Users 'self-select' by watching associated confluence page

## Site configuration<a id="orgheadline18"></a>

Everyone needs following additional shell

# Using Guix<a id="orgheadline29"></a>

The section presents how to use Guix to learn about available
applications, and to get them loaded into your unix environment.

## Using the \`Guix package\` command<a id="orgheadline28"></a>

You will most often use the \`quix package\` command to 

### Learning what applications have already been installed.<a id="orgheadline20"></a>

TBW

### Getting details about an installed application.<a id="orgheadline21"></a>

TBW

### Learning which installed applications are in your current environment.<a id="orgheadline22"></a>

TBW

### Controlling which installed applications are in your current environment.<a id="orgheadline23"></a>

TBW

### Setting up a project specific environment.<a id="orgheadline24"></a>

TBW

### Learning what applications are available for installation<a id="orgheadline25"></a>

TBW

### Using Guix instead of python's pip<a id="orgheadline26"></a>

TBW

### Enabling reproducible research<a id="orgheadline27"></a>

# References & Links<a id="orgheadline34"></a>

## Reference Manuals<a id="orgheadline30"></a>

-   [GNU Guix Reference Manual](http://www.gnu.org/software/guix/manual/guix.html) ([pdf](https://www.gnu.org/software/guix/manual/guix.pdf))

## (White) Papers<a id="orgheadline31"></a>

-   [Reproducible and User-Controlled Software Environments in HPC with Guix](http://arxiv.org/abs/1506.02822)
-   [Reproducible Development Environments with GNU Guix](http://dthompson.us/reproducible-development-environments-with-gnu-guix.html) - blog post -
    -   considers new(ish) \`Guix environment\` command as replacement for
        Python's pip/virtualenv, PHP's composer, node.js' npm
    -   demos a package which does not have any source code of its own -
        it only has run-time preconditions of other packages which must be
        "on path". (this is what spack called a "meta-package")
    -   demos usage of \`--pure\` option
-   [GNU's advanced distro and transactional package manager — GuixSD](https://www.gnu.org/software/guix/) -
-   a GNU linux distro which uses Guix as its package manager.
-   [GNU Guix - Git Repositories on Savannah](http://savannah.gnu.org/git/?group=guix)
-   [GNU Guix for managing bioinformatics software (or any other software)](https://www.linkedin.com/pulse/gnu-guix-managing-bioinformatics-software-any-other-altuna-akalin)

## Package Sources<a id="orgheadline32"></a>

-   [GNU Guix Package List](https://www.gnu.org/software/guix/package-list.html) - lists packages that are part of base Guix
    (you get 'out of the box') - includes many bioinfo packages -
    samtools, bwa, etc
-   [A searchable package list](http://guix.mdc-berlin.de) - 'You can search for “bioinfo” and all
    packages in the bioinformatics module should be displayed.  There
    are also some machine learning packages and statistics packages that
    are used in bioinformatics, but are not located in this module.'
-   [rekado - GNU Guix in an HPC environment](http://elephly.net/posts/2015-04-17-gnu-guix.html) -  blog post -
    -   refers to set of recipes [Guix.git - Guix bioinformatics source
        archive](http://git.savannah.gnu.org/cgit/guix.git/tree/gnu/packages/bioinformatics.scm)
-   dedicated build system for R packages and an importer for CRAN
    packages.  Creating a Guix package expression from a CRAN package
    (e.g. DBI) now takes little more than this:
    
    Guix import cran DBI
    
    The output is an expression that takes very little editing can be
    bound to a variable in statistics.scm.  To add the package to Guix
    upstream then only requires submitting a simple patch to
    Guix-devel@gnu.org.
-   [Guix-nonfree/bioinformatics-nonfree.scm](https://github.com/BIMSBbioinfo/guix-nonfree/blob/master/bimsb/packages/bioinformatics-nonfree.scm) - Ricardo Wurmus' repos
    of 'non-free' bioinfo apps including: DiNup, macs-1, tophat(!), viennarna

## Tips / Blogs<a id="orgheadline33"></a>

-   [GNU Guix initd scriptPrimary](https://gnunet.org/gnu-guix-initd-script) - HowTo: start Guix daemon on system startup
-   [Guix package manager without "make install"](http://dustycloud.org/blog/guix-package-manager-without-make-install/) - HowTo: run Guix out of
    the src tree without installing it
-   <https://github.com/pjotrp/Guix-notes> includes sections on:
    -   on installation
    -   [Hacking Guix](https://github.com/pjotrp/guix-notes):
    -   current notes / bioinformatics / hacking tips /
    -   lots on ruby gems
-   [(define-module (gnu packages bioinformatics)](https://github.com/BIMSBbioinfo/Guix/blob/master/gnu/packages/bioinformatics.scm)  - bioinfo repos - now taken down - 
    including: bedops, bedtools, python2-pybedtools, bowtie,, bwa,
    python2-bx-python, clipper, clustal-omega,
    crossmap,cutadapt,flexbar, hisat, htseq, htsjdk, macs,
    miso,.... samtools, seqan, star, shogun, vcftools,

# Q/A<a id="orgheadline35"></a>

-   How is package recipe used for alternate versions of the package???
-   How does this approach track with changes in the recipe??
-   How are build options handled (i.e. \`make --with-gtk \`)???

# Deployment notes<a id="orgheadline41"></a>

## TODO :<a id="orgheadline36"></a>

-   address all TODO/FIXME/TBW items.
-   provide answers to all the Q/A
-   enable/use emacs interface to Guix package management
-   socat
-   reports of locally installed packages are posted on wiki
-   feed to install activity
-   method for categorizing packages using tags
-   Document: "Linux package management with Guix"
    -   is available
        -   in github
        -   published to wiki
    -   is used as base material for series of workshops

## packages installed on mango that we would probably want to uninstall<a id="orgheadline37"></a>

    ssh mec@mango
    find -L /n/local/bin -maxdepth 2 -type f -executable -exec bash -c 'ldd {} | grep \"not found'" > /dev/null' \; -print

    ssh mango
    find -L /n/local/bin -maxdepth 2 -type f -executable -exec bash -c 'ldd {} | grep "not found" ' -print \; | tee | grep -v 'not found' | sort | uniq -c

these packages are on top of the original base install image

    sudo yum install  gmp gmp-devel # needed by pandoc
    
    sudo yum install libtk (needed by R library MASS and expected to be found on local host).
    
    
    yum install freeglut-devel
    
    Installed:
      freeglut-devel.x86_64 0:2.8.1-3.el7                                           
    
    Dependency Installed:
      freeglut.x86_64 0:2.8.1-3.el7                                                 
      gl-manpages.noarch 0:1.1-7.20130122.el7                                       
      libX11-devel.x86_64 0:1.6.3-2.el7                                             
      libXau-devel.x86_64 0:1.0.8-2.1.el7                                           
      libXdamage-devel.x86_64 0:1.1.4-4.1.el7                                       
      libXext-devel.x86_64 0:1.3.3-3.el7                                            
      libXfixes-devel.x86_64 0:5.0.1-2.1.el7                                        
      libXxf86vm-devel.x86_64 0:1.1.3-2.1.el7                                       
      libdrm-devel.x86_64 0:2.4.60-3.el7                                            
      libxcb-devel.x86_64 0:1.11-4.el7                                              
      libxshmfence.x86_64 0:1.2-1.el7                                               
      libxshmfence-devel.x86_64 0:1.2-1.el7                                         
      mesa-libGL-devel.x86_64 0:10.6.5-3.20150824.el7                               
      mesa-libGLU.x86_64 0:9.0.0-4.el7                                              
      mesa-libGLU-devel.x86_64 0:9.0.0-4.el7                                        
      xorg-x11-proto-devel.noarch 0:7.7-12.el7                                      
    
    Dependency Updated:
      libX11.x86_64 0:1.6.3-2.el7                                                   
      libX11-common.noarch 0:1.6.3-2.el7                                            
      libXext.x86_64 0:1.3.3-3.el7                                                  
      libdrm.x86_64 0:2.4.60-3.el7                                                  
      libxcb.x86_64 0:1.11-4.el7                                                    
      mesa-libEGL.x86_64 0:10.6.5-3.20150824.el7                                    
      mesa-libGL.x86_64 0:10.6.5-3.20150824.el7                                     
      mesa-libgbm.x86_64 0:10.6.5-3.20150824.el7                                    
      mesa-libglapi.x86_64 0:10.6.5-3.20150824.el7

## What is special about SIMR deployment<a id="orgheadline38"></a>

-   CURRENTLY: Guix was not initially designed to be run in a
    centralised manner. A Guix daemon is supposed to run on each system
    as root and it listens to RPCs from local users only. In an
    environment with multiple clusters and multiple workstations this
    approach requires considerable effort to make it work correctly and
    securely.

## notes to self<a id="orgheadline39"></a>

-   install from source or from 0.8.3 tarball
-   how to bootstrap network ready install?
-   if build, then
    ./configure --prefix=/gnu

or
        ./configure --localstatedir=/gnu/var --exec-prefix=/gnu  

-   init.d system.d
-   still needed?
    mkdir /gnu/var/Guix/profiles/per-user ## the doc says this should happen by the daemon but not!  FIXME! BUG?
    chmod a+rwx /gnu/var/Guix/profiles/per-user
-   TBW: transition plan?

<http://debbugs.gnu.org/cgi/bugreport.cgi?bug=20381>

-   "Interacting with a remote daemon" - socat test and report "If you
    could test this and provide feedback about the other options
    discussed there, that would be great (please email
    20381@debbugs.gnu.org.)"

We use 'Guix' package manager to manage our CentOS-based
bioinformatics and other scientific computing applications.  This
document describes how to this is done.  The section "Everybody's
Guix" shows you the basic package management method you need to see
what applications are installed and how to choose between them.
"Installing, Configuring, and Deploying Guix" details the specifics of
our site deployment.  "Q/A" provides answers to questions that were
not readily apparent from a first read of the documentation.  "TODO:"
lists local configurations and integrations that, once performed, will
make Guix even more useful to the administrator and
end-user. "References & Links" provides pointers found valuable during
the initial study and deployment of Guix.

"
> I have resolved a main issue by having /gnu mounted without root
squashing and running gnu-daemon as root, as strongly suggested after
building with \`./configure --localstatedir=/gnu/var
--exec-prefix=/gnu\`

Ludo, did I tell you, as you asked, I did build guix from master also without any FAILS and only SKIPPED one test, containers.scm.
"

## Using git<a id="orgheadline40"></a>

(replace-regexp-in-string "\n\\\\'" "" nil)

    git config --global user.name "Cook, Malcolm"
    git config --global user.email "MEC@stowers.org"
    
    #cd existing-project
    git init
    git add SCE.org # --all
    git add SCE.md # --all
    git commit -m "Initial Commit"
    git remote add origin http://MEC@stash/scm/cbio/sce.git
    git remote add stash http://MEC@stash/scm/cbio/sce.git
    git remote add origin git@github.com:malcook/sce.git
    
    git push origin master
    git push origin master
    
    
    git remote set-url --add --push origin  http://MEC@stash/scm/cbio/sce.git
    
    git remote set-url --add --push origin  http://MEC@stash/scm/cbio/sce.git
    git remote set-url --add --push origin git@github.com:malcook/sce.git
    git remote add origin git@github.com:malcook/sce.git
    
    git add SCE.org
    git commit -m 'Why adopt?'