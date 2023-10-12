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

```
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

```
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

```
#![/path/to/interpreter] [arguments]
```

Often you'll have several interpreters to choose from (different versions of
Python, for example). Technically there's no constraint on what you choose, but
it's usually best to let your `env` command direct your computer to whatever
interpreter is in your current environment. This is especially important when
you've copied a script onto a remote computer, where the path configuration may
be different than the one on your own.

The following examples use `env` to implement a shebang for Bash, Python, and R:

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

### File permissions

To use your script (in this case, the Bash version), call it from the command
line:

```
$ ./beacon.sh DataLab
zsh: permission denied: ./beacon.sh
```

This will very likely throw an error. The default file permissions for creating
new files on your computer typically do not include allowing that file to be
executed. This is to protect against users unwittingly running malicious code
from a third-party (including, potentially, other users on a computer).

To inspect the file permissions for `beacon.sh`, use `ls -l`:

```
$ ls -l beacon.sh
-rw-r--r--  1 datalab  datalabgrp  85 Oct 12 13:30 beacon.sh
```

We're interested in the first part, `-rw-r--r--`. These are called **permission
bits**. They determine whether a file has permission to be:

+ `r`: read
+ `w`: written
+ `x`: executed

The first bit in this sequence designates whether something is a directory,
[symlink][sym], or a file. The next nine parts designate who has what
permissions, using three bits for each to flag this:

[sym]: https://en.wikipedia.org/wiki/Symbolic_link

+ `-`: `beacon.sh` is a file
+ `rw-`: the user (`datalab`) who created it can read and write it
+ `r--`: other users in this user's group (`datalabgrp`) can read it
+ `r--`: users outside that group can also read it

At present, no one is allowed to execute the file. But the command line program
`chmod` can change this. There are two ways to use `chmod`. One involves
knowing [octal][oct] permission representations, which are too detailed for
this session; instead, using the symbolic representation of `r`, `w`, and `x`
will suffice.

[oct]: https://www.ibm.com/docs/en/zos/3.1.0?topic=directories-using-octal-numbers-specify-permissions

To give a file execute (`x`) permissions, type the following:

```
$ chmod u+x beacon.sh
```

Above, `u+x` adds "executable" to `beacon.sh`. Note that it only does so for
the original user (`u`).

```
$ ls -l beacon.sh
-rwxr--r--  1 datalab  datalabgrp  85 Oct 12 13:30 beacon.sh
```

With this done, the script may be executed:

```
$ ./beacon.sh DataLab
Hello, DataLab
Hello, DataLab
Hello, DataLab
Hello, DataLab
...
```

Managing Multiple Sessions
--------------------------

With no control flow to specify when the script should stop, `beacon.sh` will
run indefinitely. With this is a toy example, which is easily stopped by typing
`Ctrl+c`, there are many instances where you might write a script with the
intent of having it run for a long time. Fitting a big model is one example; or
perhaps you have a big file to download instead. Keeping your code running
requires you to keep the **session** in which you started your script running
as well. In Unix-speak, a session is a group of **processes**, which are
associated with your terminal from the time you log in to the time you log out.
Each process is dedicated to a separate program in your session, and your
computer assigns them unique ids when those programs start.

Every time you open your terminal application, you start a session. Various
processes start as well. Closing your terminal application (even
unintentionally, as in the case of a lost internet connection) terminates all
processes associated with a session. That would "kill" `beacon.sh`. In this
instance there's no harm in that, but if you have a script that you want to run
for multiple hours, or even days, you will need a way to keep your session
alive. One way to do this is with a **terminal multiplexer**. Multiplexers are
applications that enable you to run one or more sessions at the same time and
keep those sessions alive for as long as you'd like.

### The basics of `tmux`

The two most popular multiplexers are `screen` and `tmux`. Most computers will
come with one or both of these applications installed, though you can also
install them separately with a package manager like Homebrew and even conda.
This chapter will show you the basics of `tmux`. It's a little more featureful
than `screen` and it fits nicely with other work patterns associated with
programs like `vim`.

````{dropdown} A note for MacOS + conda users
MacOS has a path building utility that `tmux` calls twice when it initializes,
and this can throw off your conda environment. To fix this, you'll need to do a
further bit of configuration.

First, determine which shell you are using:

```
$ echo $SHELL
```

If you see `/bin/zsh`, create a new file in your Home directory:

```
touch ~/.zprofile
```

If you see `/bin/bash`, create a slightly different one:

```
touch ~/.bash_profile
```

Open the file (either `.zprofile` or `.bash_profile`) and enter the following:

```sh
if [[ ! -z "$TMUX" ]]; then
    PATH=""
    eval `/usr/libexec/path_helper -s`
fi
```

This will flush your path when `tmux` initializes and rebuild it so that conda
is where it should be.

Save and exit. You should be set!
````

To start `tmux`, simply type it in your command line:

```
$ tmux
```

![New `tmux` session](../img/tmux_new.png)

You are now in a **window**, which sits inside a screen. It looks the same as
your regular terminal view, except for a new bar at the bottom. Note the
number: `[0]`. This means you are working in your first session.

Within your window, start `beacon.sh`:

```
$ ./beacon.sh DataLab
Hello, DataLab
...
```

That will kick off the script, which you'll leave running. Now, type the
sequence `Ctrl+b d`. This is a **keybinding** that sends a command to `tmux`.
In this case, the command is to leave, or **detach**, your session. Doing so
should bring you back to your terminal view. You'll see the following in that
view:

```
$ tmux
[detached (from session 0)]
```

Detaching from the session will keep it alive but change the screen you're
viewing. You can go about your work and leave `beacon.sh` running. If you'd
like to check in on it, or terminate the script, enter:

```
$ tmux attach -t 0
```

That is, `attach` to the target (`-t`), which in this case is session `0`.

Entering this command will bring you back to a screen view that contains the
output of `beacon.sh`. Use `Ctrl+c` to stop your script. Then enter the
following to close `tmux`:

```
$ exit
```

It's a good idea to name your sessions, rather than relying on their index
positions.

```
$ tmux new -s "beacon"
```

The following starts a new session without entering it:

```
$ tmux new -ds "not_entered"
```

In other words, create a `new` session (`-s`) named `not_entered` in a detached
(`-d`) state.

List your current sessions with:

```
$ tmux ls
beacon: 1 windows (created Thu Oct 12 15:43:30 2023)
not_entered: 1 windows (created Thu Oct 12 15:46:07 2023)
```

Attach to a named session like so:

```
$ tmux attach -t "not_entered"
```

And close a session from your normal terminal view:

```
$ tmux kill-session -t "beacon"
```

Terminate all sessions using:

```
tmux kill-server
```

This functionality becomes extremely powerful with remote systems. Using
`tmux`, you can start a session on a remote computer, kick off a script, detach
from the session, and log off the remote computer altogether. All the while,
the script will continue running until you log in again.

### Windows and panes

In `tmux`, a window is not the same thing as a session. The latter is a
container for the former, and you can have multiple windows. Think of them like
tabs in a web browsers.

As with sessions, naming windows is a good idea. The following starts a new
session, `beacon`, with window `win1`.

```
tmux new -s "beacon" -n "win1"
```

From `win1`, you can create another window, `win2`:

```
tmux new-window -n "win2"
```

![A `tmux` session with two windows](../img/tmux_windows.png)

Note the bottom bar, which displays the two windows. To switch between them,
enter `Ctrl+b` and type `:` to enter the command mode (if you've used `vim`
before, this will feel familiar). Then, type:

```
:select-window -t "win1"
```

Terminate a window like so:

```
:kill-window -t "win2"
```

Shorthand keybindings allow for quick movement between windows:

+ `Ctrl+b p`: toggle the previous window
+ `Ctrl+b n`: toggle the next window
+ `Ctrl+b x`: close the current window

Finally, windows can be split into **panes**. This divides a window into
different areas, which can all run different processes. The syntax is similar
to the above. Enter command mode with `Ctrl+b`, then enter:

```
:split-window -v
```

...to split the window into two vertical panes. Use `-h` to split horizontally.
As before, keybindings make this faster.

+ `Ctrl+b "`: do a vertical split
+ `Ctrl+b %`: do a horizontal split

![A `tmux` window with three panes running multiple
programs](../img/tmux_panes.png)

Navigate panes in command mode using your arrow keys.

+ `Ctrl+b left`: move to the pane on the left
+ `Ctrl+b right`: move to the pane on the right
+ `Ctrl+b up`: move to the pane above
+ `Ctrl+b down`: move to the pane below

This only scratches the surface of `tmux`. It has much more functionality than
the above, and nearly all of those functions may be configured according to
your preferences. Use the application's `man` page to find out more, or consult
a [cheat sheet][cs] online.

[cs]: https://devhints.io/tmux

Monitoring Your Code
--------------------

