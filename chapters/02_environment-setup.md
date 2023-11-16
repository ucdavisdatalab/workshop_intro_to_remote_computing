Setting Up Software
===================

<!-- TODO: Define "computing environment" in the intro -->

This chapter will show you how to set up a computer so that it has the software
you need and it's a comfortable place to work. Along the way, you'll learn two
fundamental skills:

* How to install software, whether or not you have administrator access
* How to edit configuration files, particularly for the Unix shell

The examples in this chapter focus on remote computers, but these skills are
also useful for configuring laptops and desktops.


:::{admonition} Learning Objectives
+ Use Farm's module system
+ List popular tools for installing software on POSIX computers
+ Explain what virtual environments are and why they're useful
+ Explain the difference between conda and mamba
+ Install mamba via miniforge
+ Create virtual environments with mamba
+ Install software with mamba
+ Explain the difference between `.bashrc`, `.profile`, and `.bash_profile`
+ Create shell aliases
+ Create and modify environment variables
:::


What's an Environment?
----------------------

A **computing environment** is a collection of hardware, software, and
associated settings. Whenever you run software (including code) on your
computer, it runs in a computing environment. Being able to set up, inspect,
maintain, and document computing environments is important because:

* A computer may not have the software you need pre-installed. Often the best
  and sometimes the only solution is to install the software yourself. Doing it
  yourself gives you more control over what's installed and is typically faster
  than asking an administrator to do it for you. For your personal computer,
  you may be the sole administrator. You can use the same tools to set up and
  maintain environments on your computer as you do on remote computers.

* Differences between software versions can cause subtle bugs. Specifying a
  required software environment, with versions, can make it easier to
  collaborate on, distribute, and revisit research projects.

* Different projects may require different computing environments, and you may
  need to switch between them frequently. Switching software environments and
  settings can be quick and painless with the right tools. With modern compute
  clusters and cloud computing services, even switching hardware environments
  can be relatively easy.

* Inspecting the computing environment is the first step to diagnosing most
  computing problems. If you ask someone for help, they'll likely want to know
  which hardware, software, and settings you're using.

This chapter focuses on software environments and settings, because configuring
these is often the first thing you'll need to do, and because it's not always
possible to choose your hardware environment. Chapter 4 addresses ways to
choose the hardware environment on compute clusters and cloud computing
services.


Installing Software
-------------------

### Package and Environment Managers

A **package manager** is a software tool that downloads and installs software
packages. If you've used R or Python, you may already be familiar with the
package managers these languages provide.

:::{note}
Most modern Linux distributions include a package manager and recommend using
it to install software. Nevertheless, it is possible to install software on
Linux without a package manager. One way is to download the source code for the
software and compile it yourself; another is to download a pre-built binary.
[FlatPak][] and [AppImage][] are two popular formats for distributing pre-built
binaries.
:::

[FlatPak]: https://flatpak.org/
[AppImage]: https://appimage.org/


Sometimes you may need to simultaneously work with several conflicting versions
of software. For example, suppose you're working on a new project that requires
Python 3.12 or newer, but you also want to be able to examine results from an
older project that requires Python 3.9 or older. What should you do?

A lightweight solution is to use an **environment manager**, a software tool
that can create **virtual environments**. Each virtual environment behaves like
an independent software environment, so you can install conflicting software in
separate virtual environments. Many environment managers have a package manager
built in.

[Micromamba][mm] is the environment manager we recommend. Micromamba is based
on the Mamba environment manager, which is, in-turn, based on the [Conda][]
environment manager. The three tools are drop-in replacements for one another,
and it's common to refer to the virtual environments created by all three as
**conda environments**. We recommend Micromamba over Mamba and Conda because it
is substantially faster and because it eliminates some of the pitfalls of the
other two. That said, Micromamba is less mature than Mamba and Conda, so you
may occasionally encounter missing features or bugs. {numref}`micromamba` goes
into detail about how to install and use micromamba.

[mm]: https://mamba.readthedocs.io/en/latest/user_guide/micromamba.html
[Conda]: https://docs.conda.io/
[mamba]: https://mamba.readthedocs.io/

Examples of other popular package and environment managers are:

* [Homebrew][] for macOS and Linux
* [Chocolatey][] for Windows
* Advanced Packaging Tool (APT) for Debian-based Linux distributions
* [Nix][] for Linux and macOS
* [Spack][] for Linux and macOS, focused on high-performance computing
* [EasyBuild] for Linux, focused on research computing

[Homebrew]: https://brew.sh/
[Chocolatey]: https://chocolatey.org/
[Spack]: https://spack.io/
[Nix]: https://nixos.org/
[EasyBuild]: https://easybuild.io/


:::{note}
A more complex solution is to use virtualization tools, such as [Podman][],
[Docker][], and [VirtualBox][]. These provide complete control over the
operating system and software in a computing environment, so they provide
stronger guarantees of reproducibility. The cost is that these tools are often
slower than environment managers and using them requires more technical
knowledge.

For most research projects, using an environment manager provides adequate
flexibility and reproducibility. DataLab staff use virtualization infrequently.
:::

[Podman]: https://podman.io/
[Docker]: https://www.docker.com/
[VirtualBox]: https://www.virtualbox.org/


### Farm Modules

:::{note}
This section is specific to Farm. While some other servers and clusters may
have a similar module system, many do not.
:::

On most of UC Davis' HPC clusters, the HPC Core Facility staff maintain a
collection of ready-to-use virtual environments called **modules**. These are
an alternative to setting up environments yourself.

:::{note}
After learning to use Micromamba or another environment manager, you may find
it more convenient and less of a cognitive burden to just manage environments
yourself rather than use modules.
:::

You can list all of the modules available on a cluster with the `module avail`
command. For example, as of writing, the output from `module avail` on Farm
begins:

```
----------------------------------------------------------------- /share/apps/22.04/modulefiles/spack/core -----------------------------------------------------------------
cuda/11.7.1  gcc/5.5.0  hwloc/2.9.3                      jdk/17.0.1   julia/1.9.0  julia/1.9.3      nvhpc/23.1         openjdk/16.0.2  pigz/2.7    slurm/23.02.6
gcc/4.9.4    gcc/7.5.0  intel-oneapi-compilers/2022.2.1  julia/1.8.5  julia/1.9.2  libevent/2.1.12  openjdk/11.0.17_8  openmpi/4.1.5   pmix/4.2.6  ucx/1.14.1

----------------------------------------------------------------- /share/apps/22.04/modulefiles/spack/libs -----------------------------------------------------------------
amdfftw/3.2   bzip2/1.0.8  eigen/3.4.0  gmp/6.2.1  h5z-zfp/1.1.0  hypre/2.26.0               librsvg/2.51.0  mpfr/4.1.0      netcdf-cxx4/4.3.1     nvhpc/23.1  zlib/1.2.13
boost/1.80.0  cgal/5.4.1   fftw/3.3.10  gsl/2.7.1  hdf5/1.12.2    intel-oneapi-mkl/2022.2.1  libszip/2.1.1   netcdf-c/4.9.0  netcdf-fortran/4.6.0  pigz/2.7
```

Modules are displayed in groups defined by the server administrators. Each
module has a name and a version number, separated by a slash. For instance,
`julia/1.9.3` is the `julia` module, version `1.9.3`.

To get help with a module, use the `module help` command with the module's
name and optionally its version. For example, to get help with the `julia`
module:

```sh
module help julia
```

```
-------------------------------------------------------------------
Module Specific Help for /share/apps/22.04/modulefiles/spack/core/julia/1.9.3:

The Julia Language: A fresh approach to technical computing
-------------------------------------------------------------------
```

To load a module, use the `module load` command. Again, you need to specify the
module's name and optionally its version. As an example, suppose you decide you
want to try out the [Julia][] programming language on Farm. If you try to run
`julia`, you'll see an error message saying it's not installed:

[Julia]: https://julialang.org/

```sh
julia
```

```
julia: command not found
```

You can load the `julia` module to switch to an environment where Julia is
installed:

```sh
module load julia
```

Now running `julia` will open the Julia interpreter (you can exit Julia by
typing `exit()` and pressing enter).

You can check which modules are loaded at any given time with the `module list`
command:

```sh
module list
```

```
Currently Loaded Modulefiles:
 1) slurm/23.02.6   2) openmpi/4.1.5
```

You might see different output if you've loaded or unloaded modules.

Finally, to unload a module, you can use the `module unload` command with the
module's name:

```sh
module unload julia
```

Note that when you log out, Farm will automatically unload any modules you
loaded.


(micromamba)=
### Micromamba


Editing Configurations
----------------------

### Bash

### Shell Tips

### Other Configuration Files
