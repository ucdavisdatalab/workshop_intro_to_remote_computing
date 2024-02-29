Cluster Computing
=================

:::{admonition} Learning Objectives
+ Explain how partitioning and job scheduling works 
+ Know the basics of submitting and monitoring jobs with Slurm
+ Run compute-intensive jobs interactively
+ Submit and run compute-intensive jobs to a task queue
+ Explain what parallel processing is
+ Know the basics of writing code for multi-thread and multi-partition
  processes
:::

What is Slurm?
--------------

Terms to define: cluster, node (+ head node), workload manager/job scheduler,
job priority, multithreaded, parallel, Slurm

Slurm describes computations at three different levels of granularity. From
smallest to largest, they are:

* A **task** is a process that runs on a single compute node. A compute node
  may have more than one CPU or a CPU with multiple cores, so tasks can be
  multithreaded.

* A **step** is a single command in a computation, which launches one or more
  tasks (processes). In a batch script (for `sbatch`), each line is one step.

* A **job** is an entire computation, made up of one or more steps. Jobs can be
  initiated with the `srun` or `sbatch` commands.

Distinguishing between tasks, steps, and jobs is especially important when you
want to carry out non-interactive computations in parallel across multiple
nodes. When you work interactively on a single node, the distinction is not as
important: `srun` creates a single job and step where you can interactively
enter tasks.

When you submit a job to Slurm, it's added to a job queue, called a
**partition**, until the compute resources you requested are available. Slurm
uses different partitions for different types of hardware (for example, nodes
with GPUs) and different levels of job priority.
{numref}`monitoring-patitions-and-jobs` gives examples of partitions on Farm
and how they relate to priority.


(monitoring-patitions-and-jobs)=
Monitoring Partitions and Jobs
------------------------------

Most clusters are a shared by many users and have many different kinds of
nodes. Given this scope, it's helpful to get information about the organization
and state of the cluster before requesting compute resources for a specific
job.

### Partitions

You can use the `sinfo` command to list available partitions, their nodes, and
the maximum time you can request for a job. This command will also show you
whether partitions and nodes are up and running. Use the command's
`--summarize` argument to get a summary, which is easier to read on compute
clusters that have many nodes. Let's try running the command on Farm:

```sh
sinfo --summarize
```

```
PARTITION  AVAIL  TIMELIMIT   NODES(A/I/O/T) NODELIST
low2          up    4:00:00       16/43/1/60 cpu-3-[50-57,62-69],<...>
med2          up 150-00:00:       16/43/1/60 cpu-3-[50-57,62-69],<...>
high2         up 150-00:00:       16/43/1/60 cpu-3-[50-57,62-69],<...>
low           up    4:00:00      4/71/26/101 cpu-8-[62-77,86-96],<...>
med*          up 150-00:00:      4/71/26/101 cpu-8-[62-77,86-96],<...>
high          up 150-00:00:      4/71/26/101 cpu-8-[62-77,86-96],<...>
bigmeml       up    4:00:00          2/3/4/9 bigmem[1-8,10]
bigmemm       up 150-00:00:          2/3/4/9 bigmem[1-8,10]
bigmemh       up 150-00:00:          1/3/4/8 bigmem[1-8]
bigmemht      up 150-00:00:          1/0/0/1 bigmem10
bit150h       up 150-00:00:          1/0/0/1 bigmem9
ecl243        up 150-00:00:          1/0/0/1 bigmem9
bml           up    4:00:00       11/11/2/24 bm[1-24]
bmm           up 150-00:00:       11/11/2/24 bm[1-24]
bmh           up 150-00:00:       11/11/2/24 bm[1-24]
gpul          up    4:00:00          4/0/0/4 gpu-4-[54,56],gpu-5-[50,58]
bgpu          up 150-00:00:          2/0/0/2 gpu-4-54,gpu-12-92
gpuh          up 7-00:00:00          2/0/0/2 gpu-5-[50,58]
gpum          up 7-00:00:00          2/0/0/2 gpu-5-[50,58]
gpu-a100-h    up 14-00:00:0          1/0/0/1 gpu-4-56
```

In the output, the fourth column shows available nodes, idle nodes, nodes
running a job, and the total number of nodes on a partition. If you ran `sinfo`
without `--summarize`, you'll see a slightly different set of rows and columns.

Your account on a cluster will not necessarily have access to every partition.
To see which partitions you can use, run this command:

```sh
sacctmgr show associations where user=<username>
```

For instance, to see which partitions Nick can use:

```sh
sacctmgr show associations where user=nulle
```

```
   Cluster    Account       User  Partition     Share   Priority GrpJobs       GrpTRES GrpSubmit     GrpWall   GrpTRESMins MaxJobs       MaxTRES MaxTRESPerNode MaxSubmit     MaxWall   MaxTRESMins                  QOS   Def QOS GrpTRESRunMin
---------- ---------- ---------- ---------- --------- ---------- ------- ------------- --------- ----------- ------------- ------- ------------- -------------- --------- ----------- ------------- -------------------- --------- -------------
      farm datalabgrp      nulle      high2         1                                                                                                               50000                           datalabgrp-high2-qos
      farm datalabgrp      nulle       med2         1                                                                                                               50000                            datalabgrp-med2-qos
      farm datalabgrp      nulle       low2         1                                                                                                               50000                            datalabgrp-low2-qos
      farm datalabgrp      nulle gpu-a100-h         1                                                                                                               50000                           datalabgrp-gpu-a100+
      farm ctbrowngrp      nulle gpu-a100-h         1                                                                                                               50000                           ctbrowngrp-gpu-a100+
      farm ctbrowngrp      nulle       gpum         1                                                                                                               50000                            gpum-users-gpum-qos
      farm ctbrowngrp      nulle       gpuh         1                                                                                                               50000                            ctbrowngrp-gpuh-qos
      farm ctbrowngrp      nulle        med         1                                                                                                               50000                                adamgrp-med-qos
      farm ctbrowngrp      nulle        low         1                                                                                                               50000                                adamgrp-med-qos
      farm ctbrowngrp      nulle        bmh         1                                                                                                               50000                             ctbrowngrp-bmh-qos
      farm ctbrowngrp      nulle        bmm         1                                                                                                               50000                             ctbrowngrp-bmm-qos
      farm ctbrowngrp      nulle        bml         1                                                                                                               50000                             ctbrowngrp-bml-qos
      farm ctbrowngrp      nulle      high2         1                                                                                                               50000                           ctbrowngrp-high2-qos
      farm ctbrowngrp      nulle       med2         1                                                                                                               50000                            ctbrowngrp-med2-qos
      farm ctbrowngrp      nulle       low2         1                                                                                                               50000                            ctbrowngrp-low2-qos
```


### Jobs

Sometimes you might want detailed information about jobs running on a partition
or even a specific node. You can use the `squeue` command to get this
information. Think of `squeue` as the Slurm equivalent to `top` or `htop`.

On big clusters, the output of `squeue` can be quite long. Below, we use `head`
to take only the first 10 rows:

```sh
squeue | head -n 10
```

```
  JOBID PARTITION     NAME  USER ACCOUNT ST       TIME   TIME_LEFT NODES CPU MIN_ME NODELIST(REASON)
9375115      bgpu     bash user1   acct1  R    7:42:28     2:17:32     1 6   32G    gpu-4-54
9375105      bgpu  UserJob user2   acct1  R    8:21:50  7-03:38:10     1 6   200G   gpu-12-92
9297800   bigmemh   sbatch user3   acct2  R 9-17:58:59     6:01:01     1 16  120G   bigmem8
6173446   bigmemm McL-hmm- user4   acct3 PD       0:00  2-00:00:00     1 4   8G     (launch failed requeued held)
9375794   bigmemm     bash user5   acct4  R    5:13:36  3-18:46:24     1 16  40G    bigmem10
9377337   bigmemm raNTallg user6   acct5  R       0:42 20-19:59:18     1 2   4G     bigmem10
9377130   bigmemm raNTallg user6   acct5  R      37:10 20-19:22:50     1 2   4G     bigmem10
9376376   bigmemm raallga1 user6   acct5  R    1:38:22 20-18:21:38     1 2   4G     bigmem10
9318810       bmh   censor user7   acct6  R 4-03:19:08  5-20:40:52     1 4   20G    bm20
```

Above, `squeue` lists out jobs along with a fair bit of metadata about each
one. The following table breaks out this information into its individual
components:

| Column    | Explanation                                                         |
|-----------|---------------------------------------------------------------------|
| JOBID     | Unique identifier for the job                                       |
| PARTITION | Partition for the job                                               |
| NAME      | Name of the job                                                     |
| USER      | Name of the user running the job                                    |
| ACCOUNT   | The HPC account to which this user belongs                          |
| ST        | Job status; `R` is "running", `PD` is "pending", `S` is "suspended" |
| TIME      | Total run time of the job thus far                                  |
| TIME_LEFT | Remaining allocated time                                            |
| NODES     | Number of nodes allocated to the job                                |
| CPU       | Number of CPU cores allocated to the job                            |
| MIN_ME    | Minimum memory allocated to the job                                 |
| NODELIST  | Node(s) on which the job is running                                 |

When you're planning a job, you might want to run it on a specific partition.
Use the `--partition` flag to check which jobs are currently running. If the
partition's nodes are all busy, your job won't start until some become idle.

For example, this command asks `squeue` for information about the `bgpu`
partition, which has GPU capabilities:

```sh
squeue --partition bgpu
```

```
  JOBID PARTITION    NAME  USER ACCOUNT ST    TIME  TIME_LEFT NODES CPU MIN_ME NODELIST(REASON)
9375115      bgpu    bash user1   acct1  R 7:42:28    2:17:32     1 6   32G    gpu-4-54
9375105      bgpu UserJob user2   acct1  R 8:21:50 7-03:38:10     1 6   200G   gpu-12-92
```

Currently, there are two jobs running on the `bgpu` partition: one has about
two hours remaining, while the other has been allocated up to a week of run
time.

:::{note}
For most Slurm commands (and most shell commands) you can set arguments for
parameters by putting a space between the parameter and argument (`--partition
bgpu`) or by putting an equals sign between the parameter and argument
(`--partition=bgpu`). You can use whichever form is most comfortable.
:::

Checking with `sinfo` will tell you whether there is another node for your job
on your desired partition.

```sh
sinfo --partition bgpu
```
```
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
bgpu         up 150-00:00:      2    mix gpu-4-54,gpu-12-92
```

Unfortunately, there isn't. If you want to use this partition, you'll need to
wait your turn.


Interactive Jobs
----------------

Once you know which partitions and nodes are available, you can submit a
request to Slurm to **allocate** compute resources for your computations. There
are two ways to do this: in real time, or by adding your job to a queue.

Running a job in real time puts you into an environment that resembles any
other command line interface. It's an ideal way to develop and test code with
the added space and compute power of a big partition.

You can use the `srun` command to request an interactive job on a partition.
This command has a large number of parameters, so we encourage you to consult
its `man` page; but for the purposes of introducing it, the most important
parameters are:

| Parameter          | Explanation                              |
|--------------------|------------------------------------------|
| `--time`           | time limit                               |
| `--pty`            | a process to run in pseudo terminal mode |
| `--partition`      | which partition to use                   |
| `--mem`            | minimum memory per node                  |
| `--job-name`       | the name of the job                      |

The `--time` parameter is required and its argument can take a variety of
forms; two common forms are `minutes` or `days-hours`. For instance, `--time
15` requests 15 minutes, while `--time 0-1` requests 1 hour.

The `--pty` parameter is required *for interactive jobs*. Its argument should
be a path to a shell. Any subsequent arguments will be passed to the shell, so
generally you should set `--pty` last. Most of the time, you'll set `--pty
/bin/bash -il` to run Bash as an interactive login (`-il`) shell.

Try running this command, which requests 30 minutes on the `med` partition:

```sh
srun --partition=med --time=30 --pty /bin/bash -il
```

```
srun: job 9377346 queued and waiting for resources
srun: job 9377346 has been allocated resources
Unloading openmpi/4.1.5
Unloading slurm/23.02.7
Loading slurm/23.02.7
Loading openmpi/4.1.5
```

The result is a command line interface, just like the one you use to interact
with on your own computer or other machines. Except, now you're on a node, deep
in the computing cluster and equipped with extra memory and compute power.

:::{tip}
Don't want to type that sequence for every job request? Consider aliasing it to
a new command in your `~/.bashrc` file (recall {numref}`aliases`). That might
look like the following:

```sh
alias scpu='srun --partition=med --time=60 --pty /bin/bash -il'
```

If you'd like to change parameters for different requests, you could create a
Bash function instead.
:::

You can use the `hostname` command to check which node you're using:

```sh
hostname
```

```
cpu-8-87
```

Alternatively, you can use `squeue` to get information about your jobs. Pass
the `--me` flag to list only your own jobs:

```sh
squeue --me
```
```
  JOBID PARTITION NAME    USER  ACCOUNT ST TIME TIME_LEFT NODES CPU MIN_ME NODELIST(REASON)
9377346       med bash datalab datalabg  R 0:07     59:53     1 2   2000M  cpu-8-87
```

The above should align with your `srun` request, though note the extra columns:
with `srun`, it's possible to request multiple nodes, change the memory
allocation, and more. We will discuss more of these parameters below.

Slurm also sets environment variables on the node with information about the
job. You can use the command `env | grep SLURM` to print all of them. For
instance, `SLURM_JOB_ID` contains the job's unique identifier:

```sh
echo $SLURM_JOB_ID
```
```
9377346
```

:::{tip}
Slurm's `scontrol` and `sstat` commands provide even more ways to get
information about a job. See their `man` pages for details.
:::

From here, you're free to do your work: write code, run scripts, move data
around---anything. When you're done, simply use `exit` or `logout` to end the
job and return to the head node.

You can confirm that you're back on the head node with `hostname`:

```sh
hostname
```
```
farm
```


Non-interactive (Batch) Jobs
----------------------------

Interactive jobs are convenient for prototyping code and experimenting with
data, but once you've worked out the details, interactivity isn't necessary and
may even be undesirable. This is especially true for long-running jobs, like
running a simulation or training a model: there's no reason to keep a terminal
open while the computation runs. Instead, it's more convenient to start the
job, log off, and go about your business until you get a notification---hours
or possibly days later---that the job is complete.

You can use the `sbatch` command to submit a non-interactive job to Slurm. The
command adds the job to a queue and runs it when the requested resources become
available.

:::{note}
Another way to keep a job running is to run it in a terminal multiplexer like
`tmux` and detach.
:::


(hello-batch-scripts)=
### Hello, Batch Scripts!

The `sbatch` and `srun` commands have many parameters in common, but with
`sbatch` the preferred way to set the parameters is to create a **batch
script**. Here's an example of a simple batch script that prints the hostname
of the allocated node:

```sh
#!/bin/bash

#SBATCH --partition med
#SBATCH --time 10

srun hostname
```

Every batch script, including this one, has three parts:

1. A shebang (`#!`) line, as introduced in {numref}`shebangs`.
2. An `sbatch` header, made up of lines that begin with `#SBATCH`. Each line
   sets a parameter for the job.
3. Shell commands to run as part of the job. Lines that begin with `srun` are
   treated as independent steps and run in a new shell. Generally, each command
   that performs a computation should be prefixed with `srun` so that Slurm can
   report job status information correctly. Commands that change the overall
   job state, such as `cd`, should NOT be prefixed with `srun`.

:::{important}
Commands in a batch script inherit the working directory from `sbatch`. You can
use `cd` to change the working directory from within a batch script.
:::

:::{tip}
You can set parameters for `srun` commands within a batch script to specify the
subset of resources needed for a particular step. For example, you might use
`sbatch` to allocate multiple nodes, but have a "setup" step in your batch
script that only requires one node to run.
:::

Use a text editor (such as `vim`) to create a new file, paste in the batch
script above, and save it as `hello.sh`. Then submit the batch script to Slurm
by running:

```sh
sbatch hello.sh
```
```
Submitted batch job 10070274
```

Slurm confirms that the job has been submitted and prints the job's identifier.

:::{note}
Technically, you can write a Slurm batch script in any language that uses the
pound sign (`#`) for comments (such as Python or R). That said, most people
write batch scripts as shell scripts, and we recommend following this
convention to keep things simple.

You can always run code in other languages from a batch script by calling the
appropriate command (for instance, `python` or `R`).
:::

You can use `squeue` with `--me` or `--job <job_id>` to check the status of the
job. If the job isn't listed in the output from `squeue`, it probably finished
running already!

When the job is done, you'll see a new file named `slurm-<job_id>.out` in your
working directory:
```sh
ls
```
```
hello.sh  slurm-10070274.out
```

This file contains the printed output from the job and a summary of the job.
You can display the contents with `cat` or open the file in a text editor:

```sh
cat slurm-10070274.out 
```
```
==========================================
SLURM_JOB_ID = 10070274
SLURM_NODELIST = cpu-10-88
==========================================
cpu-10-88

############### Job 10070274 summary ###############
Name                : hello.sh
User                : nulle
Account             : ctbrowngrp
Partition           : med
Nodes               : cpu-10-88
Cores               : 2
GPUs                : 0
State               : COMPLETED
ExitCode            : 0:0
Submit              : 2024-02-28T16:01:48
Start               : 2024-02-28T16:01:50
End                 : 2024-02-28T16:01:50
Reserved walltime   : 00:30:00
Used walltime       : 00:00:00
Used CPU time       : 00:00:00
% User (Computation): 41.27%
% System (I/O)      : 57.14%
Mem reserved        : 2000M
Max Mem used        : 52.00K (cpu-10-88)
Max Disk Write      : 0.00  (cpu-10-88)
Max Disk Read       : 0.00  (cpu-10-88)
```

### Configuring & Cancelling Jobs

Slurm provides many `sbatch` (and `srun`) parameters which you can set to
control what kinds of resources are allocated to a job. Here are some useful
parameters that we haven't described yet:

| Parameter          | Explanation                                                         |
|--------------------|---------------------------------------------------------------------|
| `--mail-user`      | email address to send job status notifications                      |
| `--mail-type`      | types of events to send notifications about                         |
| `--output`         | path to save output; can include `%j` as the job identifier         |
| `--error`          | path to save error messages; can include `%j` as the job identifier |
| `--nodes`          | number of nodes to allocate                                         |
| `--ntasks`         | maximum number of simultaneous tasks (processes)                    |
| `--cpus-per-task`  | number of CPUs per task                                             |

See `man sbatch` for more details and a complete parameter list.

Let's try a few of these with the `beacon.py` script from
{numref}`example-script`. Here's a batch script to run the Python script:

```sh
#!/bin/bash

#SBATCH --job-name beacon
#SBATCH --partition med
#SBATCH --time 15
#SBATCH --mem 10MB
#SBATCH --mail-user=<your@email.com>
#SBATCH --mail-type=ALL
#SBATCH --output logs/beacon_%j.out
#SBATCH --error logs/beacon_%j.err
#SBATCH --nodes 1

cd beacon
# The -u parameter makes Python print output immediately.
srun python3 -u beacon.py DataLab
```

This batch script uses a `cd` command to set the working directory. Batch
scripts should include whatever commands are necessary to set up the
environment for your computation. For instance, you might need to run
`micromamba activate <environment>` if you want to run a script that requires
packages installed with Micromamba.

Use a text editor to make a new file, copy in the code above, edit the
`--mail-user` line, and save the file as `~/run_beacon.sh`. Then submit the
job:

```sh
sbatch run_beacon.sh
```
```
Submitted batch job 10070390
```

If you check `squeue`, you'll see it either waiting to be run or running:

```sh
squeue --me
```
```
   JOBID PARTITION     NAME     USER  ACCOUNT ST        TIME   TIME_LEFT NODES CPU MIN_ME NODELIST(REASON)
10070390       med   beacon    nulle ctbrowng  R        0:05       14:55     1 2   10M    cpu-10-72
```

And if you've told Slurm to update you via email, you should have an email in
your inbox telling you that your job is in the queue. Slurm will notify you
when the job starts and ends.

Recall, however, that `beacon.py` uses a `while` loop to print a name
indefinitely. That means it will only end when your request time runs out or
when you end the program early. To do the latter, take note of the `JOBID` in
the `squeue` output and enter that as an argument to `scancel`:

```
scancel 10070390
```

Slurm will cancel the job, log output and errors to the `.out` and `.err`
files, respectively, and notify you via email that the job is no longer
running.

You can use `cat` or a text editor on the `.out` file to see the output:

```sh
cat logs/beacon_10070390.out
```
```
==========================================
SLURM_JOB_ID = 10070390
SLURM_NODELIST = cpu-10-72
==========================================
Hello, DataLab
Hello, DataLab
Hello, DataLab
Hello, DataLab
Hello, DataLab
Hello, DataLab

############### Job 10070390 summary ###############
Name                : beacon
User                : nulle
Account             : ctbrowngrp
Partition           : med
Nodes               : cpu-10-72
Cores               : 2
GPUs                : 0
State               : CANCELLED by 823623,CANCELLED
ExitCode            : 0:0
Submit              : 2024-02-28T18:06:52
Start               : 2024-02-28T18:06:52
End                 : 2024-02-28T18:06:59
Reserved walltime   : 00:15:00
Used walltime       : 00:00:07
Used CPU time       : 00:00:00
% User (Computation): --
% System (I/O)      : --
Mem reserved        : 10M
Max Mem used        : 0.00  (cpu-10-72)
Max Disk Write      : 0.00  (cpu-10-72)
Max Disk Read       : 0.00  (cpu-10-72)
```

:::{tip}
You can use the `set -x` command to in shell scripts to make Bash print out
shell commands as they run. This information can be helpful for debugging,
especially if some of the commands use variables as arguments.
:::


Parallel (Array) Jobs
---------------------

One of the advantages of computing clusters is that they can run computations
in parallel, using multiple CPUs or even multiple nodes. Parallel computations
are usually non-interactive: it's difficult to interact with multiple
computations at once!

With Slurm and `sbatch`, two approaches to run part or all of a job in parallel
are:

1. **Parallel steps**: in this approach, you allocate multiple nodes or CPUs,
   and then set `srun` parameters on the job steps so that each job only uses
   a fraction of the total allocation. This approach is well-suited to jobs
   where you need to run many qualitatively different computations (for
   instance, many different scripts) and the order in which they run doesn't
   matter.

2. **Array jobs**: in this approach, you use the special `--array` job
   parameter to make `sbatch` run the batch script multiple times (an array of
   subjobs). Slurm assigns a unique index to each subjob. This approach is
   well-suited to jobs where you need to run many qualitatively similar
   computations (for instance, one script with many different sets of initial
   parameters) and the order in which they run doesn't matter.

These are not the only approaches, but others tend to be more complicated.

Let's try out both approaches to parallelism using modified versions of the
`hello.sh` script from {numref}`hello-batch-scripts`. We'll make the script run
`hostname` twice, possibly on two different nodes.

### Parallel Steps

In the parallel steps approach, the batch script header must allocate the total
number of nodes (with `--nodes`) and number of CPUs (with `--ntasks`) needed
when the parallel steps run. So if you want to run 10 steps in parallel and
each requires an entire node, set `--nodes 10`. For the `hello.sh` script,
we'll set `--nodes 2`.

Then, for each step (line with `srun`) that should be parallel:

* Set the number of nodes (again with `--nodes`) and number of CPUs
  (`--ntasks`) required. So if a step requires 4 CPUs, set `--ntasks 4`.
* Put an ampersand `&` at the end of the line; this makes the shell run the
  command without waiting for it to complete. 

Finally, put a `wait` command at any point where all prior steps must finish
before proceeding. The batch script must include at least one `wait` to make
sure that the job doesn't end before the parallel steps finish.

After making these changes, the `hello.sh` script becomes:

```sh
#!/bin/bash

#SBATCH --partition med
#SBATCH --time 10
#SBATCH --nodes 2

srun --nodes 1 hostname &
srun --nodes 1 hostname &
wait
```

Save this version of the script as `parallel_hello.sh`, and submit it to Slurm
with `sbatch`:

```sh
sbatch parallel_hello.sh
```

The job won't take long to run. When it finishes, use `cat` or a text editor to
take a look at the `.out` file:

```sh
cat slurm-10071069.out
```
```
==========================================
SLURM_JOB_ID = 10071069
SLURM_NODELIST = cpu-10-[18,66]
==========================================
cpu-10-18.farm.hpc.ucdavis.edu
cpu-10-66

############### Job 10071069 summary ###############
Name                : parallel_hello.sh
User                : nulle
Account             : ctbrowngrp
Partition           : med
Nodes               : cpu-10-[18,66]
Cores               : 4
GPUs                : 0
State               : COMPLETED
ExitCode            : 0:0
Submit              : 2024-02-28T22:38:25
Start               : 2024-02-28T22:38:26
End                 : 2024-02-28T22:38:26
Reserved walltime   : 00:10:00
Used walltime       : 00:00:00
Used CPU time       : 00:00:00
% User (Computation): --
% System (I/O)      : --
Mem reserved        : 4000M
Max Mem used        : 0.00  (cpu-10-66,cpu-10-18)
Max Disk Write      : 0.00  (cpu-10-66,cpu-10-18)
Max Disk Read       : 0.00  (cpu-10-66,cpu-10-18)
```

The host names are different, so the `hostname` command ran on two different
nodes.

The parallel steps approach is extremely flexible, because each step can use a
different set of resources (nodes, CPUs, memory, ...) and run an entirely
different command. You can also have a set of parallel steps followed by a set
of serial steps or vice-versa. Just make sure to use `wait` as needed and to
leave off the ampersand `&` on the serial steps.


### Job Arrays

In the job array approach, the batch script header must set the `--array`
parameter with the indices for the array of jobs. The batch script will run
once for each index. The indices must be a comma-separated list or
dash-separated range of unique, positive integers (see `man sbatch` for more
forms). For instance, to use indices from 1 to 20, set `--array 1-20`. To use
indices 1, 3, and 5 to 10, use `--array 1,3,5-10`. 

:::{caution}
On some clusters, there's an upper limit on the indices---typically `1001` or
`5001`.
:::

Adding the setting `--array 1-2` to run 2 subjobs, the `hello.sh` script
becomes:

```sh
#!/bin/bash

#SBATCH --partition med
#SBATCH --time 10
#SBATCH --array 1-2

srun hostname
```

Save this version of the script as `array_hello.sh`, and submit it to Slurm
with `sbatch`:

```sh
sbatch array_hello.sh
```

Once again, the job will finish quickly once it start running. Array jobs
generate a separate `.out` file for each subjob, with the job identifier and
array index in the filename. Use `cat` or a text editor to examine the first
`.out` file:

```sh
cat slurm-10071088_1.out`
```
```
==========================================
SLURM_JOB_ID = 10071089
SLURM_NODELIST = cpu-10-88
==========================================
cpu-10-88

############### Job 10071089 summary ###############
Name                : allocation
User                : nulle
Account             : ctbrowngrp
Partition           :
Nodes               : cpu-10-88
Cores               : 2
GPUs                : 0
State               : COMPLETED
ExitCode            : 0:0
Submit              : 2024-02-28T23:00:33
Start               : 2024-02-28T23:00:33
End                 : 2024-02-28T23:00:35
Reserved walltime   : 00:00:00
Used walltime       : 00:00:02
Used CPU time       : 00:00:00
% User (Computation):  0.00%
% System (I/O)      : 73.68%
Mem reserved        : 0
Max Mem used        : 76.00K (cpu-10-88)
Max Disk Write      : 0.00  (cpu-10-88)
Max Disk Read       : 0.00  (cpu-10-88)
```

Then examine the second `.out` file:
```sh
cat slurm-10071088_2.out`
```
```
==========================================
SLURM_JOB_ID = 10071088
SLURM_NODELIST = cpu-10-88
==========================================
cpu-10-88
```

In this case, Slurm ran both subjobs on the same node, but used 2 separate CPUs
(most nodes on Farm have at least 4 CPUs).

There's one more important feature of array jobs that the preceding example
didn't demonstrate. When you run an array job, for each subjob, Slurm sets the
`SLURM_ARRAY_TASK_ID` environment variable to its index. This way you can get
the index from within the subjob and use it to select sets of initial
parameters, which code to run, and so on. For example, try running this batch
script:

```sh
#!/bin/bash

#SBATCH --partition med
#SBATCH --time 10
#SBATCH --array 1-2

srun echo "Hello from subjob $SLURM_ARRAY_TASK_ID!"
```

{numref}`case-study-bml-traffic-model` provides a more detailed example of how
to use this feature.

Compared to the parallel steps approach, the array job approach is less
flexible but easier to set up. In the `hello.sh` example, we only had to add
one line to the batch script to make it run in parallel.


(case-study-bml-traffic-model)=
### Case Study: BML Traffic Model

The [Biham-Middleton-Levine traffic model][bml] (BML model) is a simple model
for traffic flow that has been studied by many mathematicians and computer
scientists, including UC Davis' own [Raissa D'Souza][raissa]. The model
consists of a rectangular grid of cells, where each cell can hold at most one
car. Cars are either red and move 1 cell to the right each time they move, or
blue and move 1 cell up each time they move. At each time step, only one color
of cars moves; the other color moves on the next time step. Cars do not travel
if another car is in their destination cell. Depending on the density of the
cars, different patterns of jams and free-flowing traffic emerge.

[bml]: https://en.wikipedia.org/wiki/Biham%E2%80%93Middleton%E2%80%93Levine_traffic_model
[raissa]: https://doi.org/10.1103/PhysRevE.71.066112

The Python script below simulates 1,000 steps of a BML model and then plots the
resulting pattern of cars to a PNG file. The initial parameters for the model
(grid size and proportions of blue and red cars) are selected from 10 different
parameter sets stored in the `PARAMETER_SETS` variable. The script reads a
single index (1-10) from the command line to select the parameter set. In other
words, the script is designed to run as a Slurm array job.

Use a text editor to create a new file, copy and paste the script into the
file, and save it as `bml.py`.

::::{admonition} Python BML Traffic Model Code
:class: dropdown

```python
import numpy as np
import matplotlib as mpl
import matplotlib.pyplot as plt

import sys


def main():
    # Get the index as a command line argument.
    try:
        index = int(sys.argv[1])
    except:
        print("Requires an integer index argument.")
        sys.exit(-1)

    # Get the parameter set.
    PARAMETER_SETS = [
        {"shape": (100, 100), "p_up": 0.05, "p_right": 0.05},

        {"shape": (100, 100), "p_up": 0.10, "p_right": 0.05},
        {"shape": (100, 100), "p_up": 0.15, "p_right": 0.05},
        {"shape": (100, 100), "p_up": 0.20, "p_right": 0.05},
        {"shape": (100, 100), "p_up": 0.25, "p_right": 0.05},

        {"shape": (100, 100), "p_up": 0.05, "p_right": 0.10},
        {"shape": (100, 100), "p_up": 0.05, "p_right": 0.15},
        {"shape": (100, 100), "p_up": 0.05, "p_right": 0.20},
        {"shape": (100, 100), "p_up": 0.05, "p_right": 0.25},

        {"shape": (100, 100), "p_up": 0.25, "p_right": 0.25}
    ]
    params = PARAMETER_SETS[index - 1]

    # Run the model.
    model = BMLModel(**params)
    model.run(1000)

    # Plot the final state and save the plot to a file.
    fig = model.plot()
    out_path = f"bml_{index:02}.png"
    fig.savefig(out_path)
    print(f"Saved '{out_path}'")


class BMLModel:
    def __init__(self, shape, p_up = 0.1, p_right = 0.1, rng = None):
        self.time = 0
        self.shape = shape
        n_cells = np.prod(shape)

        # Get number of cars.
        p = p_up + p_right
        if p > 1:
            raise ValueError("Sum of `p_up` and `p_right` must not exceed 1.")
        n_cars = np.round(p * n_cells).astype(int)
        n_up = np.round(p_up * n_cells).astype(int)

        # Set up car positions (column-major indexes).
        if rng is None:
            rng = np.random.default_rng()
        pos = rng.choice(range(n_cells), n_cars, replace = False)
        self.positions_up = pos[:n_up]
        self.positions_right = pos[n_up:]

    def __repr__(self):
        msg = (
            f"BMLModel ({self.shape[0]} x {self.shape[1]})\n"
            f"Time: {self.time}\n"
            f"Blue (up) cars: {len(self.positions_up)}\n"
            f"Red (right) cars: {len(self.positions_right)}"
        )
        return msg

    def run(self, n):
        for i in range(n):
            self.move_right()
            self.move_up()
        self.time += n

    def move_up(self):
        h, w = self.shape
        old_pos = self.positions_up
        pos = old_pos - (old_pos % h) + ((old_pos + 1) % h)

        # Back off if there's a collision.
        is_collision = np.isin(pos, self.positions_right)
        ix = np.flatnonzero(is_collision)
        while len(ix) > 0:
            pos[ix] = old_pos[ix]
            # Did this create any within-color collisions?
            is_collision = ~is_collision & np.isin(pos, pos[ix])
            ix = np.flatnonzero(is_collision)

        self.positions_up = pos

    def move_right(self):
        h, w = self.shape
        old_pos = self.positions_right
        pos = (old_pos + h) % (h * w)

        # Back off if there's a collision.
        is_collision = np.isin(pos, self.positions_up)
        ix = np.flatnonzero(is_collision)
        while len(ix) > 0:
            pos[ix] = old_pos[ix]
            # Did this create any within-color collisions?
            is_collision = ~is_collision & np.isin(pos, pos[ix])
            ix = np.flatnonzero(is_collision)

        self.positions_right = pos

    def xy(self):
        h = self.shape[0]
        x_up = self.positions_up // h
        y_up = self.positions_up - x_up * h
        x_right = self.positions_right // h
        y_right = self.positions_right - x_right * h
        return (x_up, y_up), (x_right, y_right)

    def plot(self):
        # Set up the plot.
        h, w = self.shape
        fig, ax = plt.subplots()
        ax.set_xlim(0, w)
        ax.set_ylim(0, h)
        ax.set_aspect(1)
        ax.set_xticks(np.arange(0, w, 2))
        ax.set_yticks(np.arange(0, h, 2))
        ax.grid(True, linestyle = "dotted")
        ax.set_axisbelow(True)

        xy_up, xy_right = self.xy()
        # Plot blue (up) cars.
        for x, y in zip(*xy_up):
            rect = mpl.patches.Rectangle(
                (x, y), width = 1, height = 1, facecolor = "#0000FF88",
                edgecolor = "#000000")
            ax.add_patch(rect)

        # Plot red (right) cars.
        for x, y in zip(*xy_right):
            rect = mpl.patches.Rectangle(
                (x, y), width = 1, height = 1, facecolor = "#FF000088",
                edgecolor = "#000000")
            ax.add_patch(rect)

        return fig


if __name__ == "__main__":
    main()
```
::::

In order to run the Python script, we need an appropriate environment and a
batch script.

We can use Micromamba to create an environment where the script can
run (see {numref}`micromamba`). First make the environment:

```sh
micromamba env create --name bml
```

Then activate the environment:

```sh
micromamba activate bml
```

Finally, install the packages `python`, `numpy`, and `matplotlib`:

```sh
micromamba install python numpy matplotlib
```

Next, we need to set up the batch script, which should start an array job with
indices 1-10 and run the Python script with the current subjob's index as an
argument. This batch script will do:

```sh
#!/bin/bash

#SBATCH --partition med
#SBATCH --time 10
#SBATCH --array 1-10

srun python bml.py $SLURM_ARRAY_TASK_ID
```

:::{note}
When you run an array job on Farm, Slurm automatically passes on your shell
environment to the subjobs. This is why we didn't have to activate the `bml`
environment from within the batch script.

On other clusters, Slurm may be configured differently. In order to use
`micromamba` from within a batch script, you may have to initialize it first
(within the batch script) with this command:

```sh
eval "$(micromamba shell hook --shell )"
```

Then you can use Micromamba normally.
:::

Save the batch script into a file called `run_bml.py`. Then submit the batch
script to Slurm:

```sh
sbatch run_bml.py
```

It may take a few minutes for the job to finish, but if everything went well,
you'll see 10 PNG files in your working directory. Use `scp` or another file
transfer tool to download the files (see {numref}`transfer-files`), and check
out the traffic patterns!



<!--
Jobs in Practice
----------------

### Etiquette

Using storage responsibly (callback to `du`/`df`)

Using shared storage (if lab/dept has it)

Using `/scratch/`

Compression when uploading/downloading files


### Project Organization

Callout to reproducible research reader

Keeping files in sync with `rsync`

Keeping files in sync with `git`

General advice about working on servers vs locally

* SSH tunneling for RStudio/Jupyter/VSCode?
-->
