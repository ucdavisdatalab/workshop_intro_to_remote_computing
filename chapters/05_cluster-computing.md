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

What is Parallel Computation?
-----------------------------

The default mode of computation is **sequential** or **single-threaded**
computation. In this mode, a program, which comprises a number of small
individual steps or instructions, is executed by a single CPU, one instruction
after the other. The sequence of instructions that are executed over time, from
the beginning of the program to its conclusion, is called the "thread of
execution." In a sequential program, there is only one thread of execution,
which progresses in a deterministic and orderly way.

In **parallel computation,** on the other hand, a program contains multiple
threads of execution. These threads are executed independently of each other,
and typically at the same time, using multiple CPUs cores, or multiple CPUs, or
even multiple computers. Importantly, while the instructions in each individual
thread of execution are still executed deterministically in sequence, there is
no implicit ordering between instructions in different threads of execution,
meaning that the order of execution of instructions across an entire program is
no longer deterministic. Dealing with this non-determinancy and the problems it
causes is one of the big challenges of parallel programming.

### Multi-threading vs Distributed Computing

In almost all cases, the multiple threads of execution that compose a parallel
computation are working towards a common goal, such as multiplying very large
matrices, or solving large sets of partial differential equations. To achieve
those common goals, there typically needs to be some form of communication
between individual threads of execution. There are two major models of parallel
computing based on two different methods of communication between threads of
execution.

In **multi-threading,** different threads of execution primarily communicate
with each other by reading from or writing to the same memory. For example, one
thread might write a result to some variable, and a different thread might later
retrieve that result by reading from the same variable. Besides sharing memory,
different threads also need mechanisms to synchronize with each other, for
example to ensure that one thread only reads a result from some variable *after*
another thread has writting that result to that variable. As mentioned before,
there is no implicit order of execution between different threads, meaning that
this order has to be established explicitly by the programmer through the use
of synchronization. This is typically done via mechanisms provided by the
operating system, such as POSIX pthreads.

In **distributed computing,** on the other hand, different threads of execution
do not share memory, and instead communicate and synchronize by sending messages
to each other. Unlike multi-threading, which relies on the existence of shared
memory that can be accessed by all threads, and therefore only works on a single
computer, distributed computing can work on multiple computers, by sending
messages between computers over some network, such as Ethernet or the Internet.
For example, one thread of execution might calculate some partial result, and
then pack that result into a message and explicitly send that message to another
thread of execution on a different computer, for example over a TCP connection.
That other thread of execution will explicitly wait for such a message to
arrive, extract the partial result from the message, and then use it for its own
computation. This send/receive paradigm not only exchanges data, but also
synchronizes between the involved threads, because implicitly, a message has to
be sent before it can be received. Therefore, there is no need for a dedicated
synchronization method, as it is necessary in multi-threading.

Of the two models, multi-threading is typically the easier one for programmers.
A multi-threaded program often does not look very different from a sequential
program, because it is based on the same paradigm of reading and writing the
values of variables from and to memory. Many algorithms can be extended from
sequential programming to multi-threading in a straightforward manner. The
primary difference between sequential and multi-threaded programs is the need
for explicit synchronization, which is usually supported by the operating
system.

Turning a sequential program into a distributed program, on the other hand,
typically requires a lot more effort because the familiar paradigm of reading
and writing from and to memory no longer applies to the program as a whole.
Oftentimes, distributed computing requires a major re-structuring of existing
sequential programs, including the use of completely different algorithms. In
addition, there is also the difficulty of actually sending messages between
parts of a program running on different computers. There can be a multitude of
different types of networks, or different ways how the individual computers are
connected to each other. For that reason, distributed computing typically relies
on higher-level support libraries such as MPI ("message passing interface") that
hide the ugly details of inter-computer communication from the programmer.

The primary benefit of using distributed computing over multi-threading is
**scalabililty.** If one needs to run a parallel computation using thousands or
tens of thousands of threads of execution to solve a very large problem, it is
much easier and cheaper to build multiple computers that together contain such
a large number of CPUs and connect them via a high-speed network than to build
a single supercomputer containing the same number of CPUs. For that reason, the
vasy majority of high-performance computing resources available today are such
networks.

### Hybrid Models of Computation

On a high-performance computing resource comprising a network of individual
computers, it is typically the case that each of those computers also contain
multiple CPUs or CPU cores. It is therefore natural to use a hybrid model of
parallel computation where a program is split into a number of **processes**
which each run on individual computers and communicate by sending messages to
each other, and to split each of those processes in turn into a number of
**threads** which run on the same computer and communicate with each other by
sharing memory. These days, almost all computers contain multiple CPU cores,
typcially between 4 and 16, and therefore this hybrid model of computation is
the de-facto standard model, to the point that the job management and scheduling
systems described in the remainder of this chapter assume that all programs
submitted by users follow it.

In summary, a parallel program following this default model has the following
components:

* A single **program,** which executes a desired computation.

* One or more **processes,** which each run on a different computer, and
  communicate with each other by passing messages, typically using a support
  library such as MPI.

* For each process, one or more **threads,** which all run on the same computer,
  and communicate with each other by sharing memory and using operating system-
  supplied synchronization mechanisms such as POSIX pthreads.

What is a Cluster?
------------------

A **cluster** is a high-performance computing resource comprising a collection
of individual computers that are connected via an internal high-speed
communication network, intended to run large-scale parallel computations based
on the aforementioned "standard" hybrid model of parallel computation. A cluster
is distinguished from other types of supercomputers by the absence of shared
memory that is accessible to *all* CPUs in the cluster, which means that, in
order to use the full computing power of a cluster, parallel programs must
follow the distributed computing or hybrid model. On the other hand, clusters
typically offer very large amounts of secondary (hard drive) storage that can be
accessed by all CPUs, usually in the form of network-attached storage (NAS).

A ***node*** is one single computer in a cluster. Fundamentally, there are two
classes of nodes: there is a number of ***compute nodes*** that together execute
large computations, and -- typically -- a single ***head node*** that manages
the entire cluster. The head node is also the "face" of the cluster, where users
log in, run commands, and copy data between the user's local computer, the
cluster, and the Internet. Usually, each node in a cluster, but especially the
compute nodes, are themselves high-performance computers containing a medium
number (typically dozens) of CPU cores across multiple CPUs, large amounts of
memory, and sometimes one or more GPUs ("graphical processing units") that can
compute certain types of problems with very high efficiency.

A **workload manager** or **job scheduler** is a software system that manages
the allocation of cluster resources to the **jobs,** i.e., parallel programs,
that the cluster's users want to run. Instead of running their parallel programs
directly, for example from the command line when logging into a cluster, users
submit their jobs to the manager. The manager will then ensure that all
resources a job needs are available to it before it is started, assign the job
to a subset of the cluster's compute nodes depending on the number of processes
the job wants to use, and finally run the job and keep the user informed of the
job's progress.

What is Slurm?
--------------

Slurm ("Simple Linux Utility for Resource Management") is the workload manager
used on UC Davis's Farm cluster.

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

Use `sinfo` to list available partitions, their nodes, and the maximum time you
can request for a job. This command will also show you whether partitions are
up and running.

```
$ sinfo --summarize
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

The fourth column in `sinfo` shows available nodes, idle nodes, nodes running a
job, and the total number of nodes on a partition. That's a decent overview,
but detailed information about specific nodes is also available with `squeue`.
Think of the latter command as the Slurm equivalent to `top` or `htop`. It will
show you information about all the jobs, the users running them, and the status
of those jobs.

On big clusters, the output of `squeue` can be quite long. Below, we use `head`
to take only the first 10 rows.

```
$ squeue | head -n 10
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
| JOBID     | Unique id for the job                                               |
| PARTITION | Which partion a job is running on                                   |
| NAME      | Name of the job                                                     |
| USER      | Name of the user running the job                                    |
| ACCOUNT   | The HPC account to which this user belongs                          |
| ST        | Job status; `R` is "running", `PD` is "pending", `S` is "suspended" |
| TIME      | Total time of the job thus far                                      |
| TIME_LEFT | Time left on the request                                            |
| NODES     | Number of nodes this job uses                                       |
| CPU       | Number of CPU cores this job uses                                   |
| MIN_ME    | The job memory                                                      |
| NODELIST  | Which node the job is running on                                    |

When you're planning a job, you might intend to run it on a specific partition.
Use the `--partition` flag to check which jobs are currently running. It may be
that most of the partition's nodes are full, so you'll learn that your job
won't start until they've finished.

The command below asks `squeue` for information about the `bgpu` partition,
which has GPU capabilities. Currently, there are two jobs running on it: one
has about two hours remaining, while the other has been allocated up to a week
of run time.

```
$ squeue --partition=bgpu
  JOBID PARTITION    NAME  USER ACCOUNT ST    TIME  TIME_LEFT NODES CPU MIN_ME NODELIST(REASON)
9375115      bgpu    bash user1   acct1  R 7:42:28    2:17:32     1 6   32G    gpu-4-54
9375105      bgpu UserJob user2   acct1  R 8:21:50 7-03:38:10     1 6   200G   gpu-12-92
```

Checking with `sinfo` will tell you whether there is another node for your job
on your desired partition.

```
$ sinfo --partition=bgpu
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
bgpu         up 150-00:00:      2    mix gpu-4-54,gpu-12-92
```

Unfortunately, there isn't. If you want to use this partition, you'll need to
wait your turn.

Running Jobs
------------

Once you know which partitions and nodes are available, you can submit a
request to **allocate** some space to do your work. There are two ways to do
this: in real time, or by adding your job to a queue.

### Real Time Interactions

Running a job in real time puts you into an environment that resembles any
other command line interface. It's an ideal way to develop and test code with
the added space and compute power of a big partition.

Use the `srun` command to request an interactive job on a partition. This
command has a large number of parameters, so we encourage you to consult its
`man` page; but the purposes of introducing it, the most important parameters
include one that specifies which partition you want to work on and one for how
much time you'd like to do your work.

The command below uses `srun` to request 60 minutes for a node on the `med`
partition.

```
$ srun --partition=med --time=60 --pty /bin/bash -il
srun: job 9377346 queued and waiting for resources
srun: job 9377346 has been allocated resources
Unloading openmpi/4.1.5
Unloading slurm/23.02.7
Loading slurm/23.02.7
Loading openmpi/4.1.5
```

In addition to `--partition` and `--time`, the four other pieces of this
command specify how you want to interact on this node. They specify that you
want to...
+ Use a terminal for interactions (`--pty`)
+ Load Bash as a shell for handling those interactions (`/bin/bash`)
+ Type commands (`-i`)
+ Run your job immediately (`-l`)

:::{admonition} Don't want to type that sequence for every job request?
Consider aliasing it to a new command in your `~/.bashrc` file. That might look
like the following:

```sh
alias start_cpu_session='srun --partition=med --time=60 --pty /bin/bash -il'
```

If you'd like to change parameters for different requests, you could create a
Bash function instead.
:::

The result of the above parameters is a command line interface, just like the
one you use to interact with on your own computer or other virtual machines.
Except, now you're on a node, deep in the computing cluster and equipped with
extra memory and compute power.

If you'd like to see which node you're on, use `hostname`:

```
$ hostname
cpu-8-87.farm.hpc.ucdavis.edu
```

Or, use `squeue` to see more information about your session. Send the command
the `--me` flag to return your jobs.

```
$ squeue --me
  JOBID PARTITION NAME    USER  ACCOUNT ST TIME TIME_LEFT NODES CPU MIN_ME NODELIST(REASON)
9377346       med bash datalab datalabg  R 0:07     59:53     1 2   2000M  cpu-8-87
```

The above should align with your `srun` request, though note the extra columns:
with `srun`, it's possible to request multiple nodes, change the memory
allocation, and more. We will discuss more of these parameters below.

From here, you're free to do your work: write code, run scripts, move data
around---anything. When you're done, simply use `logout` to return to the
head node:

```
$ logout
$ hostname
farm
```

### Submitting to a Queue

While `srun` gives you the interactivity of any other command line interface,
not everything needs to be run in these real time interactions. This is
especially the case with big jobs, like training a large model: there would be
no reason to keep a terminal open while the model trained. Instead, you would
want to kick off the process and simply wait for the results, which could take
hours, if not days.

A terminal multiplexer like `tmux` would be one way to keep a session running
in a detached state while you go about your day, but Slurm also allows you to
submit a non-interactive job via `sbatch`. This command adds that job to a
queue and runs it when the partition becomes available. Technically it's
possible to write out the details of that job on the command line, but it's
easier to create a **batch script** that specifies the parameters of your
allocation request as well as the operations required to run the job. You
submit this script to `sbatch` and the program will follow those operations
when it's your job's turn.

A batch script is written in Bash and (typically) saved to a file with a `.sh`
extension. It has two parts:
1. A header, which defines the parameters of your job; you should prepend every
   parameter with `#SBATCH`
2. The operations you'd like `sbatch` to follow; think of these like command
   line instructions

The following example uses our `beacon.{py,R,sh}` script from the previous
chapter to print a message continuously.

```sh
#!/bin/bash -login
#SBATCH --partition=med
#SBATCH --job-name=beacon
#SBATCH --time=0-12:00:00
#SBATCH --nodes=1
#SBATCH --mem=10MB
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --error /path/to/log/%x_%j.err
#SBATCH --output /path/to/log/%x_%j.out
#SBATCH --mail-type=ALL
#SBATCH --mail-user=<your@email.com>

set -e
set -x

cd ~/beacon
python3 beacon.py DataLab

env | grep SLURM
scontrol show job ${SLURM_JOB_ID}
sstat --format 'JobId,MaxRSS,AveCPU' -P ${SLUR_JOB_ID}.batch
```

Let's walk through the components. Lines from the header portion of the script
are below:

| Line               | Explanation                                           |
|--------------------|-------------------------------------------------------|
| `/bin/bash -login` | use Bash and your user environment                    |
| `--partition`      | which partition to use                                |
| `--job-name`       | the name of the job                                   |
| `--nodes`          | how many nodes the job runs on                        |
| `--mem`            | maximum memory to use                                 |
| `--ntasks`         | number of simultaneous operations                     |
| `--cpus-per-task`  | cpu cores/task; more cores means more compute         | 
| `--error`          | send errors to a file formatted `job-name_job-id.err` |
| `--output`         | send output to a file formmated `job-name_job-id.out` |
| `--mail-type`      | get updates from Slurm sent to your email             |
| `--mail-user`      | send updates to this address                          |

Now the job operations:

| Line             | Explanation                                               |
|------------------|-----------------------------------------------------------|
| `set -e`         | Exit the script if a command fails                        |
| `set -x`         | Print commands to the `.err` file (helpful for debugging) |
| `cd <...>`       | Go to a directory                                         |
| `python3 <...>`  | Run a script                                              |

Note that these are just command line instructions. If you have a module or
virtual environment to activate, a directory to create, multiple scripts to run
in sequence, etc., you can script this out.

Finally, we include some optional operations for getting usage statistics about
your job, which you might use to determine whether you've requested adequate
memory, time, etc.

| Line             | Explanation                                   |
|------------------|-----------------------------------------------|
| `env <...>`      | Retrieve Slurm-specific environment variables |
| `scontrol <...>` | Use those variables to show job information   |
| `sstat <...>`    | Get usage statistics about the job            |

All of the above is saved to a `beacon.sh` file. Before submitting it, create a
`log` directory to store any logging information your code creates as it runs.

```
$ cd ~/beacon
$ mkdir log
```

With that done, you're ready to submit the script. You do so from the head
node.

```
$ sbatch beacon.sh
```

If you check `squeue`, you'll see it either waiting to be run or running:

```
$ squeue --me
  JOBID PARTITION   NAME    USER    ACCOUNT ST TIME TIME_LEFT NODES CPU MIN_ME NODELIST(REASON)
9384372       med beacon datalab datalabgrp  R 0:15  11:59:45     1 2   10M    cpu-8-87
```

And if you've told Slurm to update you via email, you should have an email in
your inbox telling you that your job is in the queue. Slurm will notify you
when the job starts and ends.

Recall however that `beacon.{sh,py.R}` uses a `while` loop to print a name
indefinitely. That means it will only end when your request time runs out or
when you end the program early. To do the latter, take note of the `JOBID` in
the `squeue` output and enter that as an argument to `scancel`:

```
$ scancel 9384372
```

Slurm will cancel the job, send logging information to the `.out` file, and
notify you via email that the job is no longer running.

Opening the `.out` file will show your job details:

```
$ cat log/beacon_9384572.out
==========================================
SLURM_JOB_ID = 9384372
SLURM_NODELIST = cpu-8-87
==========================================

############### Job 9384372 summary ###############
Name                : beacon
User                : datalab
Account             : datalabgrp
Partition           : med
Nodes               : cpu-8-87
Cores               : 2
GPUs                : 0
State               : CANCELLED by 1804235,CANCELLED
ExitCode            : 0:0
Submit              : 2024-01-22T09:35:08
Start               : 2024-01-22T09:35:09
End                 : 2024-01-22T09:35:30
Reserved walltime   : 12:00:00
Used walltime       : 00:00:21
Used CPU time       : 00:00:00
% User (Computation): --
% System (I/O)      : --
Mem reserved        : 10M
Max Mem used        : --
Max Disk Write      : --
Max Disk Read       : --
```

Etiquette
---------

Using storage responsibly (callback to `du`/`df`)

Using shared storage (if lab/dept has it)

Using `/scratch/`

Compression when uploading/downloading files



Parallel Processing
-------------------

### Multi-thread jobs


### Multi-partition jobs



Project Organization
--------------------

Callout to reproducible research reader

Keeping files in sync with `rsync`

Keeping files in sync with `git`

General advice about working on servers vs locally

* SSH tunneling for RStudio/Jupyter/VSCode?
