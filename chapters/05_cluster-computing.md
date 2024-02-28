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
