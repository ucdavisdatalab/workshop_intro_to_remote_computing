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

Terms to define: partition, node (+ head node), workload manager, job priority


Partition Info and Monitoring
-----------------------------

`sinfo`: partition and node information

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

`squeue`: partition and node availability

```
$ squeue | head -n 10
  JOBID PARTITION     NAME  USER ACCOUNT ST        TIME   TIME_LEFT NODES CPU MIN_ME NODELIST(REASON)
9375115      bgpu     bash user1   acct1  R     7:42:28     2:17:32     1 6   32G    gpu-4-54
9375105      bgpu  UserJob user2   acct1  R     8:21:50  7-03:38:10     1 6   200G   gpu-12-92
9297800   bigmemh   sbatch user3   acct2  R  9-17:58:59     6:01:01     1 16  120G   bigmem8
6173446   bigmemm McL-hmm- user4   acct3 PD        0:00  2-00:00:00     1 4   8G     (launch failed requeued held)
9375794   bigmemm     bash user5   acct4  R     5:13:36  3-18:46:24     1 16  40G    bigmem10
9377337   bigmemm raNTallg user6   acct5  R        0:42 20-19:59:18     1 2   4G     bigmem10
9377130   bigmemm raNTallg user6   acct5  R       37:10 20-19:22:50     1 2   4G     bigmem10
9376376   bigmemm raallga1 user6   acct5  R     1:38:22 20-18:21:38     1 2   4G     bigmem10
9318810       bmh   censor user7   acct6  R  4-03:19:08  5-20:40:52     1 4   20G    bm20
```

Running Jobs
------------

Once you know which partitions and notes are available, you can submit a
request to **allocate** some space to do your work. There are two ways to do
this: in real time, or by adding your job to a queue.

### Real Time Interactions

Running a job in real time puts you into an environment that resembles any
other command line interface. It's an ideal way to develop and test code with
the added space and compute power of a big partition.

Use the `srun` command to request an interactive job on a partition. This
command has a number of parameters, so we encourage you to consult its `man`
page; but for our purposes, the most important ones include a parameter that
specifies which partition you want to work on and how much time you'd like for
doing your work.

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

If you'd like to change parameters for different requests, you might create a
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

Or, use `squeue` to see more information about your session:

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
no reason to keep a terminal open while the model trained; instead, you would
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

A batch script is written in Bash. It has two parts:
1. A header, which defines the parameters of your job (all prepended with
   `#SBATCH`)
2. The operations you'd like `sbatch` to follow (all written in Bash)

The following example uses our `beacon.{py,R,sh}` script from the previous
chapter to print a message continuously to a file via `sbatch`.

```sh
#!/bin/bash -login
#SBATCH --partition=med
#SBATCH --job-name=beacon
#SBATCH --time=0-12:00:00
#SBATCH --nodes=1
#SBATCH --mem=10MB
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH -e /path/to/log/%x_%j.err
#SBATCH -o /path/to/log/%x_%j.out
#SBATCH --mail-type=ALL
#SBATCH --mail-user=<you@email.com>

set -e
set -x

cd ~/beacon
python3 beacon.py DataLab > file.txt

env | grep SLURM
scontrol show job ${SLURM_JOB_ID}
sstat --format 'JobId,MaxRSS,AveCPU' -P ${SLUR_JOB_ID}.batch
```

Let's walk through the components.
1. First, this script contains a shebang, which tells `sbatch` to use Bash and
   your user environment
2. Next come the `sbatch` **directives**. These are the parameters of your
   request, which are exactly the same parameters you would use to use `srun`.
   A few are worth noting in more detail:
   + Each batch job has a **job name**, which you specify with `--job-name`
   + Using `--mem` specifies how much memory you need for your job
   + The `--ntasks` command tells Slurm whether you would like to run multiple
     parts of your script simultaneously
   + Setting a value for `--cpus-per-task` specifies how many CPU cores each
     task should use
   + The `-e` command redirects any error messages to a file for logging
     purposes. Above, `%x_%j` formats the error file as `job-name_job-id.err`
   + The `-o` command redirects any output from your job to a file. As with the
     error file, the format of the file name is `job-name_job-id.out`
   + Setting `--mail-type` requests job updates from Slurm
   + Those updates will go to your email address, which you set with
     `--mail-user`
3. With the directives declared, we set two more environment variables:
   + Setting `-e` ensures that the script will exit if a command fails
   + Setting `-x` prints each command and its arguments to the `.err` file for
     your job (this is helpful for debugging purposes)
4. Next comes the actual job. In this portion of the script, Bash goes to a
   directory `~/beacon` and then calls Python on `beacon.py`. The output is
   sent to a file
5. The last portion is all logging:
   + We retrieve the environment information from Slurm with `env | grep SLURM`
   + We use that information to display detailed information about our job with
     `scontrol ...`
   + And finally, we plug job information into a Slurm command, `sstat`, which
     tells us information about how much memory and compute power the job used


`scancel`

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
