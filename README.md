# Workshop: Introduction to Remote Computing

[![License: CC BY-SA 4.0](https://img.shields.io/badge/License-CC_BY--SA_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by-sa/4.0/)

_[UC Davis DataLab](https://datalab.ucdavis.edu/)_  
_Fall 2024_  
_Instructors: Oliver Kreylos, Nick Ulle, Wesley Brooks_  
_Authors: Nick Ulle, Oliver Kreylos, Tyler Shoemaker_  
_Maintainer: Nick Ulle <<naulle@ucdavis.edu>>_  

* [Reader](https://ucdavisdatalab.github.io/workshop_intro_to_remote_computing/)

<!--
* [Event Page](https://datalab.ucdavis.edu/eventscalendar/YOUR_EVENT/)
-->

This workshop series provides an introduction to accessing and computing on
remote servers such as UC Davis's "Farm" cluster. The series covers everything
you need to know to get started: how to set up and use SSH to log in and
transfer files, how to install software with conda, how to reserve computing
time and run programs with SLURM, and shell commands that are especially useful
for working with servers.

### Learning Objectives

After this workshop series, learners should be able to:

+ Use SSH to log in to a server
+ Transfer files to and from a server
+ Set up and use conda/mamba to install software on a server
+ Use SLURM to run interactive and non-interactive software on a server
+ Explain etiquette for using a server cluster such as Farm

### Prerequisites

Participants must have taken DataLab’s "Overview of Remote and High Performance
Computing (HPC)” workshop and "Introduction to the Command Line" workshop
series, or have equivalent prior experience. Participants must be comfortable
with basic Linux shell syntax.


## Contributing

The course reader is a live webpage, hosted through GitHub, where you can enter
curriculum content and post it to a public-facing site for learners.

To make alterations to the reader:
	  
1.  Check in with the reader's current maintainer and notify them about your 
    intended changes. Maintainers might ask you to open an issue, use pull 
    requests, tag your commits with versions, etc.

2.  Run `git pull`, or if it's your first time contributing, see
    [Setup](#setup).

3.  Edit an existing chapter file or create a new one. Chapter files may be 
    either Markdown files (`.md`) or Jupyter Notebook files (`.ipynb`). Either 
    is fine, but you must remain consistent across the reader (i.e. don't mix 
    and match filetypes). Put all chapter filess in the `chapters/` directory.
    Enter your text, code, and other information directly into the file. Make 
    sure your file:

    - Follows the naming scheme `##_topic-of-chapter.md/ipynb` (the only 
      exception is `index.md/ipynb`, which contains the reader's front page).
    - Begins with a first-level header (like `# This`). This will be the title
      of your chapter. Subsequent section headers should be second-level
      headers (like `## This`) or below.

    Put any supporting resources in `data/` or `img/`.

4.  Run the command `jupyter-book build .` in a shell at the top level of the
    repo to regenerate the HTML files in the `_build/`.

5.  When you're finished, `git add`:
    - Any files you edited directly
    - Any supporting media you added to `img/`

    Then `git commit` and `git push`. This updates the `main` branch of the
    repo, which contains source materials for the web page (but not the web
    page itself).

6.  Run the following command in a shell at the top level of the repo to update
    the `gh-pages` branch:
    ```
    ghp-import -n -p -f _build/html
    ```
    This uses the [`ghp-import` Python package][ghp-import], which you will
    need to install first (`pip install ghp-import`). The live web page will
    update automatically after 1-10 minutes.

[ghp-import]: https://github.com/c-w/ghp-import


## Setup

### Python Packages

We recommend using [mamba][] to manage Python dependencies. The `env.yml` file
in this repo contains a list of packages necessary to build the reader. You can
create a new conda environment with all of the packages listed in that file
with this shell command:

```sh
mamba env create --file env.yaml
```

[mamba]: https://mamba.readthedocs.io/
