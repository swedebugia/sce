<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#orgheadline2">1. GNU Guix - in its own words</a></li>
<li><a href="#orgheadline1">2. GNU Guix advantages broken down</a></li>
</ul>
</div>
</div>

There are many approaches to software package management.  The reasons
for selecting GNU Guix are outlined in this section.

# GNU Guix - in its own words<a id="orgheadline2"></a>

The [introduction to the GNU Guix documentation](http://www.gnu.org/software/guix/manual/guix.html#Introduction) is jam-packed with
details that present Guix's advantages.  But, it a bit of a hard read
to the un-initiated; it is both technical and dense.  If reading past
its first paragraph proves challenging, skip down to read [2](#orgheadline1) and then return to this excerpt from the intro:

"GNU Guix is a functional package management tool for the GNU (linux)
system. Package management consists of all activities that relate to
building packages from sources, honoring their build-time and run-time
dependencies, installing packages in user environments, upgrading
installed packages to new versions or rolling back to a previous set,
removing unused software packages, etc.

The term functional refers to a specific package management discipline
pioneered by Nix (see Acknowledgments). In Guix, the package build and
installation process is seen as a function, in the mathematical
sense. That function takes inputs, such as build scripts, a compiler,
and libraries, and returns an installed package. As a pure function,
its result depends solely on its inputs—for instance, it cannot refer
to software or scripts that were not explicitly passed as inputs. A
build function always produces the same result when passed a given set
of inputs. It cannot alter the system’s environment in any way; for
instance, it cannot create, modify, or delete files outside of its
build and installation directories. This is achieved by running build
processes in isolated environments (or containers), where only their
explicit inputs are visible.

The result of package build functions is cached in the file system, in
a special directory called the store. Each package is installed in a
directory of its own, in the store—by default under /gnu/store. The
directory name contains a hash of all the inputs used to build that
package; thus, changing an input yields a different directory name.

This approach is the foundation of Guix’s salient features: support
for transactional package upgrade and rollback, per-user installation,
and garbage collection of packages (see Features)."

# GNU Guix advantages broken down<a id="orgheadline1"></a>

There is much contained in and implied by GNU Guix's introductory
section, and elsewhere, within its manual.  This section intends to
clearly list the practical consequences of our adopting Guix.

-   Guix package specifications, being "functional", <span class="underline">cannot</span> be
    incomplete; for instance, it is <span class="underline">impossible</span> for a deployed Guix
    package to inadvertently depend upon an aspect of the environment
    (such as a shared library) in which it was built and deployed.

-   Guix strictly control the environment in which packages are built:
    builds always take place in a 'clean' environment using the linux
    chroot mechanism.  This makes it easy, when re-packaging an
    application for Guix, to identify and fix incompletely specified
    packages (i.e. packages having tacit prerequisites).  Doing so as
    early as possible in a package's adoption will mitigate exposure to
    downstream errors.  This is of special advantage to novice and/or
    less-than-expert package developers and deployers.

-   Guix detects and prohibits program name collisions; loading
    conflicting packages into a users environment is <span class="underline">impossible</span>; this
    prevents the ambiguity and associated error that can arise when,
    for example, two packages define different programs by the same
    name and both are placed in the user's execution PATH.

-   Guix packages naturally extend to include all package resources,
    including man pages, libraries, as well as binaries.  This is a
    result of their using the underlying build engine's (i.e. gnu make)
    installation targets.

-   Guix packages, being independent of host on which they are built,
    can be downloaded already built by upstream servers known as
    [substitues](https://www.gnu.org/software/guix/manual/html_node/Substitutes.html#Substitutes), with the assurance of their being bit-level identical
    to the results of the generally longer process of configuration and
    compilation on local servers.

-   Guix provides for transactional package upgrades and roll-backs,
    allowing testing of new package deployments with a guaranteed means
    of rapid recovery should testing warrant.

-   Guix allows for the creation of multiple environment profiles,
    which may be per-user (typically), or per-group (i.e. lab), or per
    project (i.e. directory).  These profiles allow for isolation of
    the effects of package changes to only those subscribing to the
    profile.

-   Guix allows for unprivileged package management; users do not need
    special elevated privileges (root or sudo) to create custom
    environment prifiles.  Especially noteworthy is that this allows
    for rationally sharing package management as a distributed
    responsibility.  This includes installing new site-available
    applications (for those available in the guix repositories).  Thus,
    if one user observes that a new release of a site-installed
    application has become available, that user can safely install the
    upgrade centrally and immediately start using it, without effecting
    any other user's environment; without any further coordination,
    when another user <span class="underline">are</span> ready to adopt the upgrade, they will find
    that installation is unnecessary, as it has already occurred.

-   Guix explicitly represents package dependencies, allowing for a
    [garbage collection](https://www.gnu.org/software/guix/manual/html_node/Invoking-guix-gc.html#Invoking-guix-gc) process to identify and delete packages that are
    installed but no longer required; this allows for reclaiming
    storage, and reducing the size of an installed software catalog.

-   Guix provides a set of [build systems](https://www.gnu.org/software/guix/manual/guix.html#Build-Systems) providing support for language
    specific package management systems, including R, perl, python,
    ruby, haskell, emacs.  This should allow a single approach for
    managing computing environments for each of these languages/tools,
    as opposed to needing to master the ideosyncracies of multiple
    approaches, i.e. perlbrew for perl, pyenv or pythonbrew for python,
    rbenv for ruby, etc.

-   Guix provides command to [display package information](https://www.gnu.org/software/guix/manual/guix.html#Emacs-Commands), allowing to
    automate the production of catalogs of software package.  Such
    catalogs may then be shared, printed, emailed, or embedded into
    community-visible web pages as various means of advertising package
    availability to the research community.

-   Guix package specifications, being written in [Guile/scheme](https://www.gnu.org/software/guix/manual/guix.html#Defining-Packages), do no depend
    upon the users SHELL (i.e. guix works equally well with zsh, bash,
    tcsh, etc) (TODO: check on this)

-   Guix [simplifies package installation](https://www.gnu.org/software/guix/manual/guix.html#Invoking-guix-package): viz. \`guix package -i abyss\`
    serves to download, compile, test, and install the most recent
    version of abyss known to guix repository.

-   Guix has a future - it is the package manager for the new GNU
    backed linux distribution, [GuixSD](https://www.gnu.org/software/guix/).

-   Guix is [well documented](https://www.gnu.org/software/guix/manual/)

-   Guix features are under continued development, including tools for
    -   [production of graphs](https://www.gnu.org/software/guix/manual/guix.html#Invoking-guix-graph) for visualizing dependencies between packages
    
    -   [shell 'completion'](https://www.gnu.org/software/guix/manual/guix.html#Emacs-Completions) facilities
    
    -   [emacs integration](https://www.gnu.org/software/guix/manual/guix.html#Emacs-Interface) (optional) as interface for package development
        and management

-   Guix has [rich community support](https://www.gnu.org/software/guix/contribute/) - bugs will be fixed - features will be improved.

-   Guix community has already prepared [many recipes](https://www.gnu.org/software/guix/packages/) -of which
    currently [54 are bioinformatics](http://guix.mdc-berlin.de/packages?/?search=bioinfo) packages.

-   Guix packaging is relatively easy to learn - it is reasonably
    documented and there are \`lint\` style tools making checked recipes
    for being well-structured; they identify common errors in package
    specification.

-   Guix development is open source - it is open to input from all
    community members

-   Guix exposes the actual system calls to the package developer
