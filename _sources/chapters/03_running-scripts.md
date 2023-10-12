---
jupytext:
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
kernelspec:
  display_name: Python 3
  language: Python
  name: python3
---

Running Scripts
===============

:::{admonition} Learning Objectives
+ Write and run executable scripts from the command line
+ Explain what processes, jobs, and sessions are
+ Use a terminal multiplexer (`tmux`) to create multiple persistent sessions
+ Use various utilities (`ps`, `top`, `htop`) to monitor and manage processes
  and jobs as they run
+ Identify resource usage and related system information
+ Understand the trade offs between using local and remote systems during
  active software development
:::

Executable Scripts
------------------

**Executable** files contain a set of instructions that your computer
interprets and follows to perform a task. Here, "instruction" covers a wide
scope of formats: certain files contain machine or binary code, while others
contain human-readable code written in C++, Python, or R. Many executable files
are pre-compiled binaries that can be run directly; some (like C++) must be
compiled before they can be run; and a final group are interpreted files
written in high-level scripting languages (like Python and R), which require an
external program to run.

This chapter focuses on the third group of files: executable scripts.

### Anatomy of an executable script

Using an executable script works very similarly to using command line programs
like `ls`, `grep`, etc.: scripts have names and some can take optional
arguments. The biggest difference is that there is no preset **interpreter**
for your script. Unlike most command line programs, which are precompiled
binaries, you need to execute your script by specifying a program to interpret
it.

At a high level, that looks like the following:

```sh
$ [interpreter] [script] [arg1, arg2, arg3, ...]
```

**Components**
+ Interpreter: a program that interprets and executes instructions written in a
  script; common examples include Bash, Python, and R
+ Script: a plaintext file that contains your code
+ Arguments: extra commands that change the behavior of your script, point it
  to specific resources, etc.

### Shebangs

It is, however, possible to execute a script without specifying the
interpreter from the command line. Simply prepend the script with `./`:

```sh
$ ./[script] [arg1, arg2, arg3, ...]
```

But this requires inserting a **shebang** into the script. A shebang is a
special type of comment that tells your computer how it should interpret the
contents of a script. If you do not include one, your computer will default to
using your shell interpreter (typically Bash or Zsh) to parse the script. As
you might imagine, if you wrote your script in Python or R, this will throw
errors.

Shebangs always go on the first line of a script. They begin with `#!`; this
sequence is followed by an absolute path to your interpreter and any optional
arguments.

```sh
#![/path/to/interpreter] [arguments]
```

Often you'll have several interpreters to choose from (different versions of
Python, for example). Technically there's no constraint on what you choose, but
it's usually best to let your `env` command direct your computer to whatever
interpreter is in your current environment. This is especially important when
you've copied a script onto a remote computer, where the path configuration may
be different than the one on your own.

The following examples use `env` to implement a shebang in Bash, Python, and R:

`````{tab-set}
````{tab-item} Bash
```{code-block} bash
#!/usr/bin/env bash
```
````

````{tab-item} Python
```{code-block} python
#!/usr/bin/env python3
```
````

````{tab-item} R
```{code-block} R
#!/usr/bin/env Rscript
```
````
`````

This will direct your computer to use the environment variables set in `env`.

### Example script

Now that you know the anatomy of an executable script, you can write one
yourself. There are three versions of such a script below, one for Bash,
Python, and R, respectively. The script, `beacon.{sh,py,R}`, accepts one
argument from the command line, `name`. It then prints `Hello, {name}` to
screen in an indefinite `while` loop, waiting for one second after each
iteration.

`````{tab-set}
````{tab-item} Bash
```{code-block} bash
#!/usr/bin/env bash

name=$1
while true; do
    echo "Hello, $name"
    sleep 1
done
```
````

````{tab-item} Python
```{code-block} python
#!/usr/bin/env python3

import sys
import time

def main():
    name = sys.argv[1]
    while True:
        print(f"Hello, {name}")
        time.sleep(1)

if __name__ == "__main__":
    main()
```
````

````{tab-item} R
```{code-block} R
#!/usr/bin/env Rscript

args = commandArgs(trailingOnly=TRUE)
name = args[1]
while (TRUE) {
  cat(paste("Hello", name))
  Sys.sleep(1)
}
```
````
`````

### File Permissions

To use your script (in this case, the Bash version), call it from the command
line:

```sh
$ ./beacon.sh DataLab
zsh: permission denied: ./beacon.sh
```

This will very likely throw an error. The default file permissions for creating
new files on your computer typically do not include allowing that file to be
executed. This is to protect against users unwittingly running malicious code
from a third-party (including, potentially, other users on a computer).

To inspect the file permissions for `beacon.sh`, use `ls -l`:

```sh
$ ls -l beacon.sh
-rw-r--r--  1 you  staff  85 Oct 12 13:30 beacon.sh
```

We're interested in the first part, `-rw-r--r--`. These are called **permission
bits**. They determine whether a file has permission to be:

+ `r`: read
+ `w`: written
+ `x`: executed

The first bit in this sequence designates whether something is a directory,
[symlink][sym], or (in this case) a file. The next nine parts designate who has
what permissions, using three bits for each to flag this:

[sym]: https://en.wikipedia.org/wiki/Symbolic_link

+ `-`: `beacon.sh` is a file
+ `rw-`: the user (`you`) who created it can read and write it
+ `r--`: other users in this user's group (`staff`) can read it
+ `r--`: users outside that group can also read it

At present, no one is allowed to execute the file. But the command line program
`chmod` can change this. There are two ways to use `chmod`. One involves
knowing [octal][oct] permission representations, which are too detailed for
this session; instead, using the symbolic representation of `r`, `w`, and `x`
will suffice.

[oct]: https://www.ibm.com/docs/en/zos/3.1.0?topic=directories-using-octal-numbers-specify-permissions

To give a file execute (`x`) permissions, type the following:

```sh
$ chmod u+x beacon.sh
```

Above, `u+x` adds "executable" to `beacon.sh`. Note that it only does so for
the original user (`u`).

```sh
$ ls -l beacon.sh
-rwxr--r--  1 tyler  staff  85 Oct 12 13:30 beacon.sh
```

With this done, the script may be executed:

```sh
$ ./beacon.sh DataLab
Hello, DataLab
Hello, DataLab
Hello, DataLab
Hello, DataLab
...
```
Managing Sessions
-----------------

With no control flow to specify when the script should stop, `beacon.sh` will
run indefinitely. While this is a toy example, there are many instances where
you might write a script with the intent of having it run for a long time
(fitting a big model, for example). Doing so requires you to keep the
**session** in which you started your script running. In Unix-speak, a session
is a collection of **processes**, which are associated with your current
terminal from the time you log in to the time you log out. Each process is
dedicated to a separate program in your session, and your computer assigns them
all a unique id.

But if you close your terminal (i.e., you exit Terminal, Git Bash, etc., or you
loose your internet connection to a remote computer), all processes associated
with that terminal will shut down. That would "kill" `beacon.sh`. In this
instance there's no harm in that, but if you have a script that you want to run
across multiple hours, or even days, you will need a way to keep your session
alive. One way to do this is with a **terminal multiplexer**. Multiplexers are
applications that enable you to run one or more sessions at the same time and
keep those sessions alive for as long as you'd like.

Monitoring Your Code
--------------------

