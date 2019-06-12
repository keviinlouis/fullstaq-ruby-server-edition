# Fullstaq Ruby: a server-optimized Ruby distribution

> _This is hypothetical documentation that describes a product that does not yet exist._

Fullstaq Ruby is a Ruby distribution that's optimized for use in servers. It is the easiest way to:

 * Install Ruby on servers — we supply precompiled binaries.
 * Keep Ruby security-patched or up-to-date — integrates with your OS package manager.
 * Significantly reduce memory usage of your Ruby apps — a memory reduction of 50% is quite realistic.
 * Increase performance — thanks to the usage of better memory allocators.

You can think of Fullstaq Ruby as a competitor of `apt/yum install ruby`, `rbenv install` and `rvm install`. We supply [native OS packages](#how-it-works) for various Ruby versions, which are optionally compiled with [Jemalloc](#what-is-jemalloc-and-how-does-it-benefit-me) or [malloc_trim](#what-is-malloc-trim-and-how-does-it-benefit-me), allowing for lower memory usage and potentially increased performance. Our [packaging method](#minor_version_packages) allows much easier security patching.

_Fullstaq Ruby is a sister project of [Rubywhale](https://gist.github.com/FooBarWidget/b9f01c777a6fcdbe0c0e8d69a1c26ad3), which provides similar benefits but is optimized for containers and cloud-native environments._

**Table of contents:**

 * [Key features](#key-features)
 * [Background](#background)
   - [Why was Fullstaq Ruby created?](#why-was-fullstaq-ruby-created)
   - [Who is behind Fullstaq Ruby?](#who-is-behind-fullstaq-ruby)
 * [How it works](#how-it-works)
   - [Package organization](#package-organization)
   - [Rbenv integration](#rbenv-integration)
   - [Minor version packages: a great way to keep Ruby security-patched](#minor-version-packages-a-great-way-to-keep-ruby-security-patched)
   - [About variants](#about-variants)
   - [Comparisons to other systems](#comparisons-to-other-systems)
     + [Vs RVM and Rbenv](#vs-rvm-and-rbenv)
     + [Vs Ruby packages included in operating systems' official repositories](#vs-ruby-packages-included-in-operating-systems-official-repositories)
     + [Vs the Brightbox PPA](#vs-the-brightbox-ppa)
     + [Vs JRuby, TruffleRuby and Rubinius](#vs-jruby-truffleruby-and-rubinius)
     + [Vs Rubywhale](#vs-rubywhale)
 * [Installation](#installation)
   - [RHEL/CentOS](#rhelcentos)
   - [Debian/Ubuntu](#debianubuntu)
   - [Deactivate Git-based Rbenv](#deactivate-git-based-rbenv)
   - [Activate Rbenv shell integration (optional)](#activate-rbenv-shell-integration-optional)
     + [System-wide shell integration](#system-wide-shell-integration)
 * [Usage after installation](#usage-after-installation)
   - [Using a specific Ruby version](#using-a-specific-ruby-version)
   - [Usage with Rbenv](#usage-with-rbenv)
   - [Passenger for Nginx/Apache integration](#passenger-for-nginxapache-integration)
   - [Puma, Unicorn or Passenger Standalone integration](#puma-unicorn-or-passenger-standalone-integration)
   - [Capistrano integration](#capistrano-integration)
 * [FAQ](#faq)
   - [What is Jemalloc and how does it benefit me?](#what-is-jemalloc-and-how-does-it-benefit-me)
   - [What is malloc_trim and how does it benefit me?](#what-is-malloc-trim-and-how-does-it-benefit-me)
   - [Is Fullstaq Ruby faster than regular Ruby (MRI)?](#is-fullstaq-ruby-faster-than-regular-ruby-mri)
   - [Why does Fullstaq Ruby integrate with Rbenv?](#why-does-fullstaq-ruby-integrate-with-rbenv)
   - [I do not need multiple Rubies (and have no need for Rbenv), is Fullstaq Ruby suitable for me?](#i-do-not-need-multiple-rubies-and-have-no-need-for-rbenv-is-fullstaq-ruby-suitable-for-me)
   - [Which variant should I pick?](#which-variant-should-i-pick)
 

---

## Key features

 * **Precompiled binaries: save time and energy, increase security**

   Stop wasting time and energy with compiling Ruby or applying source patches. We supply precompiled Ruby binaries so that you don't have to. Improve security by eliminating the need to install a compiler on your server.

 * **Native OS packages: install and update with tools you already use**

   We supply our binaries as [_native OS packages_](#how-it-works) (e.g. via APT, YUM). Easily integrate with your configuration management tools, no need to integrate yet another installer. Easily keep Ruby security-patched or up-to-date with tools you already use.

 * **Jemalloc and `malloc_trim` integration: reduce memory usage, increase performance**

   > Main articles:
   > - [What is Jemalloc and how does it benefit me?](#what-is-jemalloc-and-how-does-it-benefit-me)
   > - [What is malloc_trim and how does it benefit me?](#what-is-malloc-trim-and-how-does-it-benefit-me)
   > - [Is Fullstaq Ruby faster than regular Ruby (MRI)?](#is-fullstaq-ruby-faster-than-regular-ruby-mri)

   In Hongli Lai's research project [What Causes Ruby Memory Bloat?](https://www.joyfulbikeshedding.com/blog/2019-03-14-what-causes-ruby-memory-bloat.html), Hongli has identified the OS memory allocator as a major cause of memory bloating in Ruby.

   There are two good solutions for this problem: either by using [the Jemalloc memory allocator](#what-is-jemalloc-and-how-does-it-benefit-me), or by using [the `malloc_trim` API](#what-is-malloc-trim-and-how-does-it-benefit-me).

   Both solutions require patching the Ruby interpreter, but we've done that for you so that you don't have to. Use our binaries, and benefit from reduced memory usage and potentially increased performance out-of-the-box and in a hassle-free manner.

 * **Multiple Ruby versions: ensure compatibility**

   Not all apps are deployed against the latest Ruby — at least, not all the time. We supply binaries for _multiple Ruby versions_. Enjoy the benefits of Fullstaq Ruby no matter which Ruby version you use.

 * **Rbenv integration: manage Ruby versions with already-familiar tools**

   Our multi-Ruby support works by integrating with the popular Ruby version manager [Rbenv](https://github.com/rbenv/rbenv). Continue to benefit from Rbenv ecosystem tooling like [capistrano-rbenv](#capistrano-integration). No need to learn another tool for managing Ruby versions.

## Background

### Why was Fullstaq Ruby created?

Fullstaq Ruby came about for two reasons:

 * **Optimizing for server use cases.**

   Ruby uses a lot of memory. In fact, [way too much and much more than it is "supposed to"](https://www.joyfulbikeshedding.com/blog/2019-03-14-what-causes-ruby-memory-bloat.html). Fortunately, there are "simple" solutions that provide a lot of benefit to users without requiring difficult code changes in Ruby: by compiling Ruby against the [Jemalloc](http://jemalloc.net/) memory allocator, or by modifying Ruby to make use the `malloc_trim` API. In case you use Jemalloc, you even get a nice performance improvement.

   > Main articles:
   > - [What is Jemalloc and how does it benefit me?](#what-is-jemalloc-and-how-does-it-benefit-me)
   > - [What is malloc_trim and how does it benefit me?](#what-is-malloc-trim-and-how-does-it-benefit-me)
   > - [Is Fullstaq Ruby faster than regular Ruby (MRI)?](#is-fullstaq-ruby-faster-than-regular-ruby-mri)

   Unfortunately for people who use Ruby in server use cases (e.g. most Ruby web apps), Ruby core developers are a bit careful/conservative and are hesitant to adopt Jemalloc, citing concerns that Jemalloc may cause regressions in non-server use cases, as well as other compatibility-related concerns.

   That's where Fullstaq Ruby comes in: we are opinionated. Fullstaq Ruby does not care about non-server use cases, and we prefer a more progressive approach, so we make choices to optimize for that use case.

 * **Installing the Ruby version you want, and keeping it security-patched, is a miserable experience.**

   The most viable ways to install Ruby are:

    1. From OS repositories, e.g. `apt install ruby`/`yum install ruby`.
    2. Compiling Ruby using a version manager like RVM or Rbenv.
    3. Installing precompiled binaries supplied by RVM or Rbenv.

   All of these approaches have problems:

    + _OS repos drawbacks:_

        - They don't always have the Ruby minor version that you want (or they require you to upgrade your distro to get the version you want).
        - There is no way to install a specific tiny version.
        - Very bad support for managing multiple Ruby versions (e.g. can't easily activate a different Ruby on a per-user or per-project basis).

    + _Compilation via RVM/Rbenv drawbacks:_

        - Requires a compiler, which is a security risk.

    + _RVM/Rbenv (whether compilation or precompiled binaries) drawbacks:_

        - Keeping Ruby security-patched is a hassle.
        - Upgrading to a new Ruby tiny version (e.g. 2.5.0 -> 2.5.1) requires explicit intervention.
        - After upgrading the Ruby tiny version, you need to update all your application servers and deployment code to explicitly use that new version, and you need to reinstall all your gems.

   Fullstaq Ruby addresses all of these problems by combining native OS packages and Rbenv. See [How it works](#how-it-works)

### Who is behind Fullstaq Ruby?

 * Fullstaq Ruby is created by [Hongli Lai](https://www.joyfulbikeshedding.com), CTO at Phusion, author of [the Passenger application server](https://www.phusionpassenger.com/), and creator of the now-obsolete Ruby Enterprise Edition.
 * Fullstaq Ruby is created in partnership with [Fullstaq](https://fullstaq.com/), a cloud native, Kubernetes and DevOps technology partner company in the Netherlands.

If you like this work, please star this repo, follow [@honglilai on Twitter](https://twitter.com/honglilai), and/or [contact Fullstaq](https://fullstaq.com/contact/). Fullstaq can take your technology stack to the next level by providing consultancy, training, and much more.

## How it works

> _See also: [Installation](#installation)_

The Fullstaq Ruby native OS packages allow you to install Rubies by adding our repository and installing them through the OS package manager. The highlights are:

 * **Native OS packages.**

   Rubies are installed by adding our repository and installing through the OS package manager.

 * **We supply packages for each minor version.**

   For example, there are packages for Ruby 2.5 and 2.6. These packages always contain the most recent tiny version. Learn more below in subsection [Minor version packages](#minor-version-packages).

 * **We _also_ supply packages for each tiny version.**

   If you require a very specific tiny version: we support that too! For example we have packages for 2.5.5 and 2.6.0.

 * **3 variants for each Ruby version: normal, jemalloc, malloctrim.**

   Learn more below in subsection [About variants](#about-variants).

 * **Each Ruby installation is just a normal directory.**

   When you install a Ruby package, files are placed in `/usr/lib/fullstaq-ruby/versions/<VERSION>`.

 * **We supply Rbenv as a native OS package.**

   We use Rbenv to switch between different Ruby versions. Rbenv has been slightly modified to support the notion of system-wide Ruby installations.

 * **All Ruby packages register a Ruby version inside the Rbenv system.**

   This registration is done on a system-wide basis (in `/usr/lib/rbenv/versions`, as opposed to `~/.rbenv/versions`).

 * **Parallel packages.**

   All Ruby versions — and all their variants — are installable in parallel.

### Package organization

> _See also: [Installation](#installation)_

Let's say you're on Ubuntu (RHEL/CentOS packages use a different naming scheme). Let's pretend Fullstaq Ruby only packages Ruby 2.6.2 and Ruby 2.6.3 (and let's pretend the latter is also the latest release). You will be able to install the following packages:

 * Version 2.6:
    - Normal variant: `apt install fullstaq-ruby-2.6`
    - Jemalloc variant: `apt install fullstaq-ruby-2.6-jemalloc`
    - Malloctrim variant: `apt install fullstaq-ruby-2.6-malloctrim`
 * Version 2.6.2:
    - Normal variant: `apt install fullstaq-ruby-2.6.2`
    - Jemalloc variant: `apt install fullstaq-ruby-2.6.2-jemalloc`
    - Malloctrim variant: `apt install fullstaq-ruby-2.6.2-malloctrim`
 * Version 2.6.3:
    - Normal variant: `apt install fullstaq-ruby-2.6.3`
    - Jemalloc variant: `apt install fullstaq-ruby-2.6.3-jemalloc`
    - Malloctrim variant: `apt install fullstaq-ruby-2.6.3-malloctrim`

All these packages can be installed in parallel. None of them conflict with each other, not even the variants.

### Rbenv integration

Suppose you install all packages listed in the [Package organization](#package-organization) example. That will register Rubies in the system-wide Rbenv versions directory:

 * /usr/lib/rbenv/versions/2.6
 * /usr/lib/rbenv/versions/2.6-jemalloc
 * /usr/lib/rbenv/versions/2.6-malloctrim
 * /usr/lib/rbenv/versions/2.6.2
 * /usr/lib/rbenv/versions/2.6.2-jemalloc
 * /usr/lib/rbenv/versions/2.6.2-malloctrim
 * /usr/lib/rbenv/versions/2.6.3
 * /usr/lib/rbenv/versions/2.6.3-jemalloc
 * /usr/lib/rbenv/versions/2.6.3-malloctrim

These registrations are symlinks. The actual Ruby installation is in `/usr/lib/fullstaq-ruby`. So for example, one symlink looks like this:

    /usr/lib/rbenv/versions/2.6 -> /usr/lib/fullstaq-ruby/versions/2.6

If you run `rbenv versions`, you'll see:

    $ rbenv versions
    * system (set by /home/hongli/.rbenv/version)
      2.6
      2.6-jemalloc
      2.6-malloctrim
      2.6.2
      2.6.2-jemalloc
      2.6.2-malloctrim
      2.6.3
      2.6.3-jemalloc
      2.6.3-malloctrim

Installed Fullstaq Rubies are available to all users on the system. They complement any Rubies that may be installed in `~/.rbenv/versions`.

You activate a specific version by using regular Rbenv commands:

     $ rbenv local 2.6.3
     $ rbenv exec ruby -v
     ruby 2.6.3

     $ rbenv local 2.6.3-jemalloc
     $ rbenv exec ruby -v
     ruby 2.6.3 (jemalloc variant)


<a name="minor_version_packages"></a>

### Minor version packages: a great way to keep Ruby security-patched

`fullstaq-ruby-2.6.2` and `fullstaq-ruby-2.6.3` are **tiny version packages**. They package a specific tiny version.

`fullstaq-ruby-2.6` is a **minor version package**. It always contains the latest tiny version! If today the latest Ruby 2.6 version is 2.6.3, then `fullstaq-ruby-2.6` contains 2.6.3. If tomorrow 2.6.4 is released, then `fullstaq-ruby-2.6` will be updated to contain 2.6.4 instead.

**We recommend installing the minor version package/image over installing tiny version packages/images**:

 * No need to regularly check whether the Ruby developers have released a new tiny versions. The latest tiny version will be automatically installed as part of the regular OS package manager update process (e.g. `apt upgrade`/`yum update`). This is safe because Ruby follows semantic versioning.
 * No need to reinstall all your gems or update any configuration files after a tiny version update has been installed. A minor version package utilizes the same paths, regardless of the tiny version that it contains.

   For example, the `fullstaq-ruby-2.6` package always contains the following, no matter which tiny version it actually is:

    - /usr/lib/rbenv/versions/2.6/bin/ruby
    - /usr/lib/rbenv/versions/2.6/lib/ruby/gems/2.6.0

### About variants

So there are 3 variants: normal, jemalloc, malloctrim. What are the differences?

 - **Normal**: The original Ruby. No third-party patches applied.

   Normal variant packages have no suffix, e.g.: `apt install fullstaq-ruby-2.6.2`

 - **Jemalloc**: Ruby is linked to the Jemalloc memory allocator.

    * Pro: Uses less memory than the original Ruby.
    * Pro: Is usually faster than the original Ruby. (How much faster? [AppFolio benchmark](http://engineering.appfolio.com/appfolio-engineering/2018/2/1/benchmarking-rubys-heap-malloc-tcmalloc-jemalloc), [Ruby Inside benchmark](https://medium.com/rubyinside/how-we-halved-our-memory-consumption-in-rails-with-jemalloc-86afa4e54aa3))
    * Con: May not be compatible with all gems (though such problems should be rare).

   Learn more: [What is Jemalloc and how does it benefit me?](#what-is-jemalloc-and-how-does-it-benefit-me)

   Jemalloc variant packages/images have the `-jemalloc` suffix, e.g.: `apt install fullstaq-ruby-2.6.2-jemalloc`

 - **Malloctrim**: Ruby is patched to make use of the `malloc_trim` API to reduce memory usage.

    * Pro: Uses less memory than the original Ruby.
    * Pro: Unlike _jemalloc_, there are no compatibility problems.
    * Con: _May_ be slightly slower. ([How much slower?](https://www.joyfulbikeshedding.com/blog/2019-03-29-the-status-of-ruby-memory-trimming-and-how-you-can-help-with-testing.html))

   Learn more: [What is malloc_trim and how does it benefit me?](#what-is-malloc-trim-and-how-does-it-benefit-me)

   Malloctrim variant packages/images have the `-malloctrim` suffix, e.g.: `apt install fullstaq-ruby-2.6.2-malloctrim`

**Recommendation:** use the _jemalloc_ variant, unless you actually observe compatibility problems.

### Comparisons to other systems

#### Vs RVM and Rbenv

RVM and Rbenv are Ruby version managers. They allow you to install and switch between multiple Ruby versions. They allow installing Ruby by compiling from source, or sometimes by downloading precompiled binaries (the availability of binaries for different platforms is limited).

Fullstaq Ruby is not a Ruby version manager. Fullstaq Ruby is a Ruby distribution: we supply binaries for Ruby. You can think of Fullstaq Ruby as a replacement for `rvm install` and `rbenv install`.

The differences between `rvm/rbenv install` and Fullstaq Ruby are:

 * Fullstaq Ruby is installed via the OS package manager. Rubies installed with `rvm/rbenv install` are not managed via the OS package managers.
 * Fullstaq Ruby updates are supplied through the OS package manager, which can be easily automated. Updates using `rvm/rbenv install` require more effort.
 * When you upgrade Ruby using `rvm/rbenv install`, the path to Ruby and the gem path changes, and so you will need to reinstall all gems and explicit reconfigure application servers to use the upgraded Ruby version -- even if you upgrade to a newer tiny version.

   Fullstaq Ruby supports the concept of [minor version packages](#minor_version_packages), which solves this hassle, making security patching much easier.

 * Fullstaq Ruby provides Ruby [variants with Jemalloc and malloc_trim integration](#about-variants). RVM and Rbenv do not.

   The Jemalloc and malloc_trim variants allow significant reduction in memory usage, and possibly also performance improvements. Learn more: [What is Jemalloc and how does it benefit me?](#what-is-jemalloc-and-how-does-it-benefit-me), [What is malloc_trim and how does it benefit me?](#what-is-malloc-trim-and-how-does-it-benefit-me)

   RVM and Rbenv still allow you to apply the Jemalloc and malloc_trim patches, but they don't provide binaries for that (while Fullstaq Ruby does) and so you will be required to compile from source.

#### Vs Ruby packages included in operating systems' official repositories

 * Many operating systems' official repositories only provide a limited number of Ruby versions. If the Ruby version you want isn't available, then you're out of luck: you'll need to upgrade or downgrade the entire OS to get that version, or you need to install it using some other method.

   Fullstaq Ruby provides native OS packages for many more Ruby versions, for multiple OS versions.

 * The Ruby packages provided by many OS repositories also don't allow you to easily switch between versions on a per-user or per-app basis.

   Fullstaq Ruby allows easy switching between versions by integrating with Rbenv.

 * OS repositories package specific minor versions of Ruby. They regularly update their packages to the latest tiny version for that minor package. This is good for security-patching reasons, but if you need to install a specific tiny Ruby version (e.g. for compatibility reasons) then you're out of luck.

   Fullstaq Ruby provides [both minor version packages and specific tiny version packages](#minor_version_packages).

 * Fullstaq Ruby provides [Jemalloc and malloc_trim integration](#about-variants). OS official repositories do not.

   The Jemalloc and malloc_trim variants allow significant reduction in memory usage, and possibly also performance improvements. Learn more: [What is Jemalloc and how does it benefit me?](#what-is-jemalloc-and-how-does-it-benefit-me), [What is malloc_trim and how does it benefit me?](#what-is-malloc-trim-and-how-does-it-benefit-me)

#### Vs the Brightbox PPA

The Brightbox PPA is an Ubuntu APT repository provided by [Brightbox](https://www.brightbox.com/).

Like Fullstaq Ruby, the Brightbox PPA contains packages for multiple Ruby versions, for multiple Ubuntu versions.

The differences are:

 * The Brightbox PPA packages specific minor versions of Ruby. They regularly update their packages to the latest tiny version for that minor package. This is good for security-patching reasons, but if you need to install a specific tiny Ruby version (e.g. for compatibility reasons) then you're out of luck.

   Fullstaq Ruby provides [both minor version packages and specific tiny version packages](#minor_version_packages).

 * Fullstaq Ruby provides [Jemalloc and malloc_trim integration](#about-variants). The Brightbox PPA does not.

   The Jemalloc and malloc_trim variants allow significant reduction in memory usage, and possibly also performance improvements. Learn more: [What is Jemalloc and how does it benefit me?](#what-is-jemalloc-and-how-does-it-benefit-me), [What is malloc_trim and how does it benefit me?](#what-is-malloc-trim-and-how-does-it-benefit-me)

 * Fullstaq Ruby also provides Debian, RHEL and CentOS packages.

 * Fullstaq Ruby has much better support for managing multiple Ruby versions, thanks to the Rbenv integration.

#### Vs JRuby, TruffleRuby and Rubinius

JRuby, TruffleRuby and Rubinius are alternative Ruby implementations, that are different from the official Ruby (MRI).

Fullstaq Ruby is not an alternative Ruby implementation. It is a distribution of MRI.

#### Vs Rubywhale

[Rubywhale](https://gist.github.com/FooBarWidget/b9f01c777a6fcdbe0c0e8d69a1c26ad3#mounting-a-directory-and-running-the-container-with-user) is a Ruby Docker base image that's highly optimized for container- and cloud-native environments. Rubywhale also integrates with Jemalloc and malloc_trim, and so provides similar benefits as Fullstaq Ruby. However, Fullstaq Ruby is not especially designed for containerization, and thus provides no Docker base images.

## Installation

### RHEL/CentOS

 * Supported RHEL/CentOS versions: 7, 8
 * Supported architectures: x86-64

Add the Fullstaq Ruby repository and install `fullstaq-ruby-common`:

    sudo curl -sSLo /etc/yum/repos.d/fullstaq-ruby.repo https://rubypackages.fullstaq.com/centos/el.repo
    sudo yum install fullstaq-ruby-common

Ruby packages are now available as `fullstaq-ruby-<VERSION>`:

    $ sudo yum search fullstaq-ruby
    ...
    fullstaq-ruby-2.6-rev3 : Latest Ruby 2.6 (currently 2.6.3), Fullstaq distribution
    fullstaq-ruby-2.6.0-rev0 : Ruby 2.6.0, Fullstaq distribution
    fullstaq-ruby-2.6.1-rev0 : Ruby 2.6.1, Fullstaq distribution
    fullstaq-ruby-2.6.2-rev0 : Ruby 2.6.2, Fullstaq distribution
    ...

You can either install a specific tiny version....

    sudo yum install fullstaq-ruby-2.6.3

...or ([recommended!](#minor_version_packages)) you can install the latest tiny version of a minor release (e.g. the latest Ruby 2.6):

~~~bash
# This will auto-update to the latest tiny version when it's released
sudo yum install fullstaq-ruby-2.6
~~~

You can even install multiple versions in parallel if you really want to:

~~~bash
# Installs the latest 2.6. At the time of writing that's 2.6.3
sudo yum install fullstaq-ruby-2.6

# In parallel, *also* install Ruby 2.6.1
sudo yum install fullstaq-ruby-2.6.1

# In parallel, *also* install Ruby 2.6.2
sudo yum install fullstaq-ruby-2.6.2
~~~

**Next steps:**

 - [Deactivate Git-based Rbenv](#deactivate-git-based-rbenv)
 - [Activate Rbenv shell integration](#activate-rbenv-shell-integration)
 - [Usage after installation](#usage-after-installation)

### Debian/Ubuntu

 * Supported Debian versions: 9
 * Supported Ubuntu versions: 18.04, 19.04
 * Supported architectures: x86-64

Add the Fullstaq Ruby repository and install `fullstaq-ruby-common`:

    [TODO]
    sudo apt install fullstaq-ruby-common

Ruby packages are now available as `fullstaq-ruby-<VERSION>`:

    $ sudo apt search fullstaq-ruby
    ...
    fullstaq-ruby-2.6/bionic,bionic rev3 amd64
      Latest Ruby 2.6 (currently 2.6.3), Fullstaq distribution

    fullstaq-ruby-2.6.0/bionic,bionic rev0 amd64
      Ruby 2.6.0, Fullstaq distribution

    fullstaq-ruby-2.6.1/bionic,bionic rev0 amd64
      Ruby 2.6.1, Fullstaq distribution

    fullstaq-ruby-2.6.2/bionic,bionic rev0 amd64
      Ruby 2.6.2, Fullstaq distribution
    ...

You can either install a specific tiny version....

    sudo apt install fullstaq-ruby-2.6.3

...or ([recommended!](#minor_version_packages)) you can install the latest tiny version of a minor release (e.g. the latest Ruby 2.6):

~~~bash
# This will auto-update to the latest tiny version when it's released
sudo apt install fullstaq-ruby-2.6
~~~

You can even install multiple versions in parallel if you really want to:

~~~bash
# Installs the latest 2.6. At the time of writing that's 2.6.3
sudo apt install fullstaq-ruby-2.6

# In parallel, *also* install Ruby 2.6.1
sudo apt install fullstaq-ruby-2.6.1

# In parallel, *also* install Ruby 2.6.2
sudo apt install fullstaq-ruby-2.6.2
~~~

**Next steps:**

 - [Deactivate Git-based Rbenv](#deactivate-git-based-rbenv)
 - [Activate Rbenv shell integration](#activate-rbenv-shell-integration)
 - [Usage after installation](#usage-after-installation)

### Deactivate Git-based Rbenv

_Note: you only need to perform this step if you already had a Git-based Rbenv installed. Otherwise you can skip to the next step: [Activate Rbenv shell integration](#activate-rbenv-shell-integration)._

Fullstaq-Ruby relies on a sightly modified version of Rbenv, for which we supply an OS package. This package installs the `rbenv` binary to /usr/bin.

You should modify your shell files to remove your Git-based Rbenv installation from your PATH, so that /usr/bin/rbenv is used instead. For example in your .bash_profile and/or .bashrc, **remove** lines that look like this:

~~~bash
# REMOVE lines like this!
export PATH="$HOME/.rbenv/bin:$PATH"
~~~

There is no need to remove `eval "$(rbenv init -)"`. You're still going to use Rbenv — just one that Fullstaq Ruby provides. More about that in the next step, [Activate Rbenv shell integration](#activate-rbenv-shell-integration).

There is also no need to remove the `~/.rbenv` directory. The Ruby versions installed in there are still supported — *in addition* to system-wide ones installed by the Fullstaq Ruby packages.

### Activate Rbenv shell integration (optional)

_Note: you only need to perform this step if you didn't already have Rbenv shell integration installed. You can skip this step if you already had Rbenv shell integration installed from a previous Git-based Rbenv installation._

For an optimal Rbenv experience, you should activate its [shell integration](https://github.com/rbenv/rbenv#how-rbenv-hooks-into-your-shell).

Run this command, which will tell you to add some code to one of your shell files (like .bashrc or .bash_profile):

    /usr/bin/rbenv init

Be sure to restart your shell after installing shell integration.

#### System-wide shell integration

Adding to .bashrc/.bash_profile only activates the shell integration for that specific user. If you want to activate shell integration for all users, you should add to a system-wide shell file. For example if you're using bash, then:

 * Ubuntu: /etc/bash.bashrc
 * RHEL/CentOS: /etc/bashrc

## Usage after installation

### Using a specific Ruby version

Ruby versions are installed to `/usr/lib/fullstaq-ruby/versions/<VERSION>`. Each such directory has a `bin` subdirectory which contains `ruby`, `irb`, `gem`, etc.

Suppose you installed Ruby 2.6 (normal variant). You can execute it directly:

    $ /usr/lib/fullstaq-ruby/versions/2.6/bin/ruby --version
    ruby 2.6.3

    $ /usr/lib/fullstaq-ruby/versions/2.6/bin/gem install --no-document nokogiri
    ...

But for convenience, it's better to add `/usr/lib/fullstaq-ruby/versions/<VERSION>/bin` to your PATH so that you don't have to specify the full path every time. See [Activate Rbenv shell integration (optional)](#activate-rbenv-shell-integration-optional).

### Usage with Rbenv

The recommended way to use Fullstaq Ruby is through the Rbenv integration. You should [learn Rbenv](https://github.com/rbenv/rbenv#readme) if you are not familiar with it. Here is a handy [cheat sheet](https://devhints.io/rbenv).

Suppose you installed Ruby 2.6 (normal variant). Just activate it using Rbenv:

    $ rbenv local 2.6
    $ ruby --version
    ruby 2.6.3

### Passenger for Nginx/Apache integration

First, find out the full path to the Ruby version's binary that you want to use: [Using a specific Ruby version](#using-a-specific-ruby-version).

Next, specify it in the Nginx or Apache config file:

 * Nginx: `passenger_ruby <full path to ruby executable>;`
 * Apache: `PassengerRuby <full path to ruby executable>`

Restart Nginx or Apache after you've made the change.

### Puma, Unicorn or Passenger Standalone integration

First, find out the full path to the Ruby version's binary that you want to use: [Using a specific Ruby version](#using-a-specific-ruby-version).

Next, modify your Puma/Unicorn/Passenger Standalone startup script (e.g. Systemd unit) and make sure that it executes your application server using the full path to your Ruby executable. This is usually done by modifying the `ExecStart` option.

For example suppose you have a Systemd unit file `/etc/systemd/system/puma.service` that looks like this:

~~~ini
# Just an example!

[Unit]
Description=Puma HTTP Server
After=network.target

[Service]
Type=simple
User=app
WorkingDirectory=<YOUR_APP_PATH>
ExecStart=/home/app/.rbenv/versions/2.5.5/bin/ruby -S bundle exec puma -C puma.rb
Restart=always

[Install]
WantedBy=multi-user.target
~~~

Make sure that your `ExecStart` command is prefixed by a call to `/full-path-to-ruby -S`, like this:

~~~ini
ExecStart=/usr/lib/fullstaq-ruby/versions/2.6.2-jemalloc/bin/ruby -S bundle exec puma -C puma.rb
~~~

> Don't forget the `-S`!

Restart your application server after you've made a change, for example `sudo systemctl restart puma`.

### Capistrano integration

If you use Capistrano to deploy your app, then you should use the [capistrano-rbenv](https://github.com/capistrano/rbenv) plugin. In your deploy/config.rb make sure you set `rbenv_ruby` to the Ruby version to you want, possibly with a [variant suffix](#about-variants). Examples:

~~~ruby
# Use Ruby 2.6 (latest tiny version), normal variant
set :rbenv_ruby, '2.6'

# Use Ruby 2.6.2, normal variant
set :rbenv_ruby, '2.6.2'

# Use Ruby 2.6.2, jemalloc variant
set :rbenv_ruby, '2.6.2-jemalloc'
~~~

## FAQ

### What is Jemalloc and how does it benefit me?

[Jemalloc](http://jemalloc.net/) is a custom memory allocator. It uses a smarter algorithm than the default Linux glibc memory allocator, and thus is faster and results in less memory fragmentation (which reduces memory usage significantly). Jemalloc has been used successfully by e.g. Firefox and Redis to reduce their memory footprint.

### What is malloc_trim and how does it benefit me?

[`malloc_trim()`](http://man7.org/linux/man-pages/man3/malloc_trim.3.html) is an API that is part of the glibc memory allocator. In Hongli Lai's research project [What Causes Ruby Memory Bloat?](https://www.joyfulbikeshedding.com/blog/2019-03-14-what-causes-ruby-memory-bloat.html), Hongli has identified the OS memory allocator as a major cause of memory bloating in Ruby. Luckily, simple fixes exist, and one fix is to invoke `malloc_trim()` which tells the glibc memory allocator to release free memory back to the kernel.

Is is found that, if Ruby calls `malloc_trim()` during a garbage collection, then memory usage can be reduced significantly.

However, `malloc_trim` may have some [performance impact](https://www.joyfulbikeshedding.com/blog/2019-03-29-the-status-of-ruby-memory-trimming-and-how-you-can-help-with-testing.html).

### Is Fullstaq Ruby faster than regular Ruby (MRI)?

If you pick the Jemalloc variant, then yes it is often faster. See [About variants](#about-variants). See also these benchmarks:

 * [AppFolio: Benchmarking Ruby's Heap: malloc, tcmalloc, jemalloc
](http://engineering.appfolio.com/appfolio-engineering/2018/2/1/benchmarking-rubys-heap-malloc-tcmalloc-jemalloc)
 * [Ruby Inside: How we halved our memory consumption in Rails with jemalloc](https://medium.com/rubyinside/how-we-halved-our-memory-consumption-in-rails-with-jemalloc-86afa4e54aa3)

### Why does Fullstaq Ruby integrate with Rbenv?

Many users have a need to install and switch between multiple Rubies on a single machine, so we needed a Ruby version manager. Rather than inventing our own, we chose to use Rbenv because it's popular and because it's easy to integrate with.

### I do not need multiple Rubies (and have no need for Rbenv), is Fullstaq Ruby suitable for me?

Yes. The multi-Ruby-support via Rbenv is quite lightweight and is unintrusive, weighting a couple hundred KB at most. Even if you do not need Rbenv, the fact that Fullstaq Ruby uses Rbenv doesn't get in your way and does not meaningfully increase resource utilization.

### Which variant should I pick?

See: [About variants](#about-variants).