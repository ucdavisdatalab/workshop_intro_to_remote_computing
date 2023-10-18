Setting Up Your Remote Computing Environment
============================================

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


Installing Software
-------------------



### Package Managers

[spack]: https://spack.io/
[conda]: https://docs.conda.io/
[mamba]: https://mamba.readthedocs.io/

<!--
Homebrew
EasyBuild
Nix
FlatPak
AppImage
apt
-->

### Farm Modules

:::{note}
This section is specific to Farm. While some other servers and clusters may
have a similar module system, many do not.
:::

On most of UC Davis' HPC clusters, the HPC Core Facility staff maintain a
collection of ready-to-use software environments called **modules**.

You can list all of the modules available on a cluster with this command:

```sh
module avail
```

```sh
module help
```

```sh
module load
```

```sh
module unload
```

:::{note}
The modules available on Farm are
:::

### Mamba


Editing Configurations
----------------------

### Bash

### Shell Tips

### Other Configuration Files
