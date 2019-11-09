---
layout: page
title:  "Crunch"
date:   2019-11-09
categories: crunch
---

## Setup

OSL has its own [Rocks
Cluster](https://en.wikipedia.org/wiki/Rocks_Cluster_Distribution) based on
Rocks 7.0.

The head node is [crunch.osl.northwestern.edu](crunch.osl.northwestern.edu)
and can be reached via ssh.

Its current load (and various other statistics) can be seen at:
[http://crunch.osl.northwestern.edu/ganglia/](http://crunch.osl.northwestern.edu/ganglia/).
You need to be on campus or using VPN to get to this URL.

## Policies

* The cluster is not intended for interactive use except for setting up jobs
  and some debugging that cannot be done elsewhere.  You should not run any
  resource-intensive tasks without submitting them through the job scheduler.

* Be nice to your fellow users! Always be conscious about the resources that
  your job might take (number of threads for multi-threaded programs, memory
  limit, and time limit).

* Be careful with the amount of output your code produces.  All output will be
  written to an output file.  If your program produces a lot of output, you
  might create a huge output file, potentially filling up the entire disk (and
  killing the cluster).  So, keep your output to a reasonable amount.  On the
  other hand, you want enough output to see if your program is actually making
  progress.

* User files are automatically backed up, except for $HOME/scratch.  Because
  of this you should **run your job only in a subdirectory of
  $HOME/scratch.**  Your experiments might produce a lot of output files, and
  there is no need to back up those files.  Once you have some data that
  should be backed up (e.g., when you have the results for a paper) you can
  move them into another part of your home directory, but please keep that to
  a reasonable amount of storage.

* Related to the previous point: If you have **huge data files** that are
  obtained from the web and can easily be retrieved again, there is no need to
  back them up with our system.  So, please put them under $HOME/scratch as
  well.

* The scheduler does not verify if your program uses more threads than you
  requested.  Please keep this in mind and **always make sure you limit the
  number of threads explicitly in your program**.  Many programs (including
  CPLEX, Gurobi, and matlab) will take as much as they can, unless told
  otherwise.  You are the one who needs to tell them.

* The default memory limit is 2GB per thread.  You can increase the limit in
  the submission script (see below).  However, the schedule is set up to tally
  up all the requested memory, and new jobs will not be started if no free
  memory is left.  So, don't request more memory than necessary, or your job
  won't start or you will prevent others from executing their jobs.

## Software

### Python

Python 2.7 and 3.6 are installed on the cluster.  You need to add the
following line to your `~/.bashrc`:

```
# Python
export PATH="/opt/python/bin:$PATH"
export LD_LIBRARY_PATH="/opt/python/lib:$LD_LIBRARY_PATH"
```

### Matlab

You need to add the following line to your `~/.bashrc`:

```
# Matlab
export PATH="/share/apps/matlab_current/bin:$PATH"
```

On the cluster, you can use Matlab only without a GUI, so you should start it
with:

```
matlab -singleCompThread -nojvm
```

**Note: If you want to use CPLEX in Matlab, you need to use an older version:**

```
# Matlab
export PATH="/share/apps/MATLAB/R2015b/bin:$PATH"
```

**You must specify the number of threads whenever your run MATLAB without the
`-singleCompThread` flag!**

Note: If you are running Matlab R2015b, you need to put the line
```
export FNP_IP_ENV=1
```
into your submit script, somewhere before the line that has the Matlab command.

Also note to add an exit command to the end of your MATLAB script to allow the
process to terminate. Otherwise the job will stay active on the queue even if
it is idle.

### AMPL

You need to add the following line to your `~/.bashrc` file (the second
directory is for the machine-dependent license file)

```
# AMPL
export PATH="/share/apps/AMPL/bin:/share/apps/AMPL/lic-$HOSTNAME:$PATH"
```

The following solvers are installed

* CPLEX

Make sure you limit the number of threads by setting the right solver option
(unless the solver is single-threaded).  The number of threads must also be
set in the submission script in the `-pe smp XXX` argument.

### R

You need to add the following line to your `~/.bashrc` file:

```
# R
export PATH="/share/apps/R/bin:$PATH"
```

### Gurobi

Note: The following is probably no longer necessary after the upgrade of the
cluster OS:

Gurobi requires python 2.7 so linuxbrew python 2.7 needs to be first in your
path, e.g.:

```
# Gurobi
export PATH="/share/apps/linuxbrew/bin:$PATH"
export GRB_LICENSE_FILE="/share/apps/gurobi_current/linux64/gurobi.lic"
export GUROBI_HOME="/share/apps/gurobi_current/linux64"
```

Interactive test:
```
$/share/apps/gurobi_current/linux64/bin/gurobi.sh
```

### CPLEX

The most recent version of CPLEX is installed in `/share/apps/cplex_current`.
Make sure you limit the number of threads when you run CPLEX!

* Path to Matlab files: `/share/apps/cplex_current/cplex/matlab/x86-64_linux`
  **(Matlab version R2016b does not work with CPLEX, use R2015b)**

### Julia

The most recent version of Julia is installed in `/share/apps/julia_current`.
To use it, add the following to your `~/.bashrc`:

```
# Julia
export PATH="/share/apps/julia_current/bin:$PATH"
```

To use a package in Julia (e.g. JuMP), type the following in Julia, and the
package will be installed to the local version of Julia

```julia
Pkg.add("JuMP")
using JuMP
```

To use CPLEX/Gurobi, you need to first add the solver path to `~/.bashrc`, and
then install/load the package, same as above.

### MKL

The Intel Math Kernel Library is installed in `/share/apps/intel`.  You need to
add the following line into your `~/.bashrc` file to make sure the environment
variables are correctly set:

```
# MKL
source /share/apps/intel/bin/compilervars.sh intel64
```

You also need to run this when you use binaries compiled with the MKL on the
cluster.

Documentation is [https://software.intel.com/en-us/intel-mkl/documentation
here].  And there is also a
[https://software.intel.com/en-us/articles/intel-mkl-link-line-advisor Link
Line Advisor] that helps you to find out which compiler options to use if you
compile code with the MKL.

Note: We could also install the Intel MPI libraries.

### Knitro

The Knitro NLP solve is installed in `/share/apps/knitro_current`.  To use it,
you need to set the `ARTELYS_LICENSE` environment variable by

```
export ARTELYS_LICENSE=/share/apps/knitro_current/lic/$HOSTNAME
```

The AMPL solver executable `knitroampl` (option solver 'knitroampl') is
installed in the knitroampl subdirectory, so you need to set

```
export PATH="/share/apps/knitro_current/knitroampl:$PATH"
```

### Linuxbrew

Latest versions of useful programs (such as gcc or python) are installed with
Linuxbrew (see below) in `/share/apps/linuxbrew`.  To use these programs, add
the following lines to your `~/.bashrc` file:

```
# Linuxbrew software
eval $(/share/apps/linuxbrew/bin/brew shellenv)
```

## Using the job scheduler

### Submitting jobs

You submit jobs with the `qsub` command.  As argument, you give it the name of a
script that includes the commands to be executed, and specifies options (such
as resource requirements) for the scheduler.

**IMPORTANT: You must run your experiments under $HOME/scratch.**

Otherwise, all the (most likely temporary output) of your experiments will be
backed up and you might fill up our backup server.  If you have files after an
experiment that you want to keep, you can move them somewhere else in your
home directory (but be reasonable about the required disk space).

Here is an example (say, submit_ampl.sh)

```
#!/bin/bash
#

########################
##### qsub options #####
########################
#
# Any line starting with #$ is interpreted as command line option for qsub

#### Working directory
#  These options specify the directory where the job is to be executed.
#  The default is your home directory (which you don't want).
#  If you use the -wd option, you need to make sure that directory exists.
#
#$ -cwd                           # Current working directory (where you run qsub)
#                                 # MUST BE UNDER $HOME/scratch !!!!!
#
# #$ -wd /home/me/scratch/CoolStuffIsHere # Run job /home/me/scratch/CoolStuffIsHere

# Name of job (and output files)
#$ -N Ampl
# Join standard error and output
#$ -j y

# Set priority (default is 0, smaller values give lower priority to job)
# #$ -p -10

##### Shell that is used
#$ -S /bin/bash

#### Number of threads
#  If you have a multi-threaded application, you need to specify here how many
#  cores your process uses.
#  Note: You explicitly have to tell you program how many threads to use
#$ -pe smp 8

#### Run time limit
#  Specify maximum CPU time after which job is to be killed (format HH:MM:SS).
#$ -l h_rt=00:10:00    # in this example, we set 10 minutes

#### Pass all of your environment variables through to SGE
#$ -V

#### Memory limit
#  specifies the maximum amount of memory this job can take
#  This is per thread, so the total amount is this number times the number
#  of threads. The default value is 2g.
#$ -l h_vmem=4g  # here we choose 4g, so that overall we reserve up to 8*4g=32g total

#########################
##### Your commands #####
#########################

# For example, run AMPL here (with cplex threads limit to 8 to match your specification above)
ampl ampl.run
```

If you edit the script file on a Windows computer and try to run them on the
cluster (using qsub), the job will fail. The problem is that Windows line
termination uses an extra character that qsub fails to recognize. These
characters can be stripped out using

```
dos2unix submit_ampl.sh
```

You can then submit the job with

```
qsub submit_ampl.sh
```

### Showing current queue

```
qstat -u "*"
```

shows you which jobs are submitted and running.  This command can also be used
to see more details about individual jobs (option -j JOB_ID), or other status
information.

### Deleting or killing jobs

```
qdel JOB_ID
```
where JOB_ID is the job id that is found with qstat.

### Monitoring progress

The output of your running job is directed into the output file that you
specified.  You can look at this file while the job is running and see how
things are going.

### Additional qsub options

* -V : By default, only a few environment variables will be passed to the job.
  The -V option passes all environment variables
* -N JobName : Give a name to the job (the default is simply the name of the
  submitted script).
* -o OutputFile : Direct output to this file
* -j y : Join stderr and stdout output
* -t n:m : Start job array. You can start a set of runs with a single
  submission, using the -t option.  For example, -t 1:40 still start 40 jobs,
  where in each job the bash environment variable SGE_TASK_ID has a value
  between 1 and 40.
* -p N : Set priority for job.  Default is 0, smaller values give lower
  priority to job.  You should use that when you have many jobs to run

For a complete list of options, please check
[https://github.com/BIMSBbioinfo/intro2UnixandSGE/blob/master/sun_grid_engine_for_beginners/how_to_submit_a_job_using_qsub.md].

### Submitting Matlab jobs

Matlab jobs must be submitted as scripts.  You need to write a matlab file
(say, "my_script.m") that has all the commands you want Matlab to run.  For
example:

```
n = 1000;
tic
A = randn(n,n);
b = randn(n);
sol = A\b;
norm(sol)
toc
```

To check if your script works, run it with the '-r' option:

```
matlab -nojvm -r my_script.m
```

Once you are happy with this, the executable lines at the end of your qsub
submission script are:

```
MATLAB_SCRIPT="my_script.m"
matlab -nojvm -singleCompThread -r "try; run('$MATLAB_SCRIPT'); catch me; display(me); end; quit;"
```

As you see, the shell variable MATLAB_SCRIPT is set to the name of your
script. (Note, there must be no space between around the "=" character!).  You
should really only edit the value of the MATLAB_SCRIPT variable, and not
modify the matlab command line.

If there is an error, the output file will display what the exception "me"
said.

If you want to let Matlab parallelize some of the computations (e.g., linear
algebra), you can start it without the -singleCompThread option, but then the
first line in your my_script.m file must be:
```
maxNumCompThreads(XXX)
```
where XXX is the number of threads you want to allow Matlab to use.  If you do
this, you also must tell the scheduler that you need that many threads, with
the following line in our submission script:
```
#$ -pe smp XXX
```
Remember that -singleCompThread and "maxNumCompThreads" only control the
number of threads in matlab libraries.  If you just an external library that
runs multi-threaded (like CPLEX), you need to set the thread limit explicitly
for that library as well.

### Submitting MPI jobs

By default, the cluster uses OpenMPI.  As usual, you need to use the MPI
compilers (mpicc, mpif77, etc) to generate your executables.

**IMPORTANT: You must use the system compilers and cannot use the latest GNU
compilers from Linuxbrew, so you need to disable any paths for Linuxbrew (see
above)!**

You can use the same submission script as above, but you need set the proper
parallel environment by replacing the section on "Number of threads" above by
something like:

```
#### Number of mpi processes
#$ -pe orte 16
```

Here, we assume that you use 16 MPI jobs.  If each of these jobs is
multi-threaded, we have to figure out what has to be set here.

The executable can then be called as usual with
```
mpirun -np 16 /opt/mpi-tests/bin/mpi-ring
```
Again, make sure the number of jobs (here 16) is the same in the mpirun
argument and in the -pe argument.  There are probably other considerations to
make things work really efficiently, but this gets at least the basic version
going.  (There is also a parallel environment "mpi" instead of "orte", but I
(AW) did not get that to work properly.)

### Submitting Julia jobs

Julia jobs are submitted as scripts.  You need to write a julia source file
(say, "my_script.jl") that has all the commands you want Julia to run.  For
example:
```
n = 1000;
tic();
A = rand(n,n);
b = rand(n);
sol = A\b;
norm(sol)
toc();
```
Once you are happy with this, the executable lines at the end of your qsub
submission script are
```
julia my_script.jl
```

#### Array of Julia Jobs

To pass an array of jobs use

```
qsub -t 2:6:2 trial.sh
```
then the values 2,4,6 will be passed through $SGE_TASK_ID to the julia code.
Inside the shell file to the numbers 2, 4, 6 to the Julia code use the following
```
julia myscript.jl $SGE_TASK_ID
```
Inside the Julia code these values can be accessed through the variable ARGS.
ARGS is an array of strings corresponding to all the arguments passed. For
example if we run
```
julia myscript 1
```
Then `ARGS[1] = "1"`. Note that this is a string. In order to convert it into
a number, use:
```
a1 = parse(Int64,ARGS[1])        #a1 = 1    (a Int64)
```

### Submitting jobs in batches

When submitting a large number of jobs, it might be useful to submit them in
batches in order to let resources available for other users. Say for example
that you need to run 1,000 jobs (each requiring 1 processor), but only using
10 out the 140 processors available in crunch (as of 09/17/2018). With out
setting the batches, crunch will schedule incoming jobs after the 1,000 jobs
are finished. For example, suppose you want to submit the following
helloworld.sh script that receives one argument (handled with $1)

```
#! /bin/bash
# Same set up as above
#$ -cwd                           # Current working directory (where you run qsub)
#                                 # MUST BE UNDER $HOME/scratch !!!!!
#$ -pe smp 1               #Number of processors
#$ -l h_rt=2:00:00      #Max cpu time
#$ -l h_vmem=1g        #RAM allocation

# Your program!
echo "Job "$1
sleep 10
```

Further, we need to submit this 1,000 times using the following script
(batchtest.sh). To set a batch size, one option is to use holds. By setting a
hold on a job, the job won't be executed until something else happens. In this
example, jobs are in hold until a predecesor job finished. So for a batch of
10, we will set as the predecesor of i-th to be i-10th job. For the first 10
jobs,  the predecesor won't exist and so they will get scheduled right away
(or as soon as they get to the top of the queue). The option in the qsub
commat is **-hold_jid**.

```
# batchtest.sh script
#!/bin/bash
jdesc="batch_test"
batch_size=10
i_job=1
for n in {1..1000}
do
    # Set job  name "batch_test<i_job>"
    # Set a predecesor for the job (the job name)
    # Pass n as a parameter (note that n and i_job could be different)
    qsub -N $jdesc$i_job -hold_jid $jdesc$(($i_job-$batch_size)) ./helloworld.sh $n
    i_job=$(($i_job+1))
done
```

**CAUTION:** In general, when you write shell scripts, it is a good idea to
first check that the command lines that you want to have executed, are indeed
what your script does.  As a simple check, put the shell command "echo" in
front of any commands that does something. In the above, you would write
```
echo qsub -N $jdesc$i_job -hold_jid $jdesc$(($i_job-$batch_size)) ./helloworld.sh $n
```
This will print the command itself instead of executing it, so that you can
verify that it is indeed the command you want. Running ./batchtest.sh with the
echo edit outputs:
```
[user@crunch scratch]$ ./batchtest.sh
 qsub -N batch_test1 -hold_jid batch_test-9  ./helloworld.sh 1
 qsub -N batch_test2 -hold_jid batch_test-8  ./helloworld.sh 2
 qsub -N batch_test3 -hold_jid batch_test-7  ./helloworld.sh 3
 ...
 qsub -N batch_test98 -hold_jid batch_test88 -o batch_test98 ./helloworld.sh 98
 qsub -N batch_test99 -hold_jid batch_test89 -o batch_test99 ./helloworld.sh 99
 qsub -N batch_test100 -hold_jid batch_test90 -o batch_test100 ./helloworld.sh 100
```

When this script is submitted, 10 jobs will start (check with qstat -s r):
```
 job-ID  prior   name       user         state submit/start at     queue                          slots ja-task-ID
 -----------------------------------------------------------------------------------------------------------------
 40233 0.55500 batch_test dduque       r     09/17/2018 15:30:11 all.q@crunch.local                 1
 40234 0.55500 batch_test dduque       r     09/17/2018 15:30:11 all.q@crunch.local                 1
 40235 0.55500 batch_test dduque       r     09/17/2018 15:30:11 all.q@crunch.local                 1
 40236 0.55500 batch_test dduque       r     09/17/2018 15:30:11 all.q@crunch.local                 1
 40237 0.55500 batch_test dduque       r     09/17/2018 15:30:11 all.q@crunch.local                 1
 40238 0.55500 batch_test dduque       r     09/17/2018 15:30:11 all.q@crunch.local                 1
 40239 0.55500 batch_test dduque       r     09/17/2018 15:30:11 all.q@crunch.local                 1
 40240 0.55500 batch_test dduque       r     09/17/2018 15:30:11 all.q@crunch.local                 1
 40241 0.55500 batch_test dduque       r     09/17/2018 15:30:11 all.q@crunch.local                 1
 40242 0.55500 batch_test dduque       r     09/17/2018 15:30:11 all.q@crunch.local                 1
```
and the others will have status `hqw`, meaning that they are in hold as oppose
 to `qw` (job is waiting for resources).
```
 job-ID  prior   name       user         state submit/start at     queue                          slots ja-task-ID
 -----------------------------------------------------------------------------------------------------------------
  40263 0.00000 batch_test dduque       hqw   09/17/2018 15:30:02                                    1
  40264 0.00000 batch_test dduque       hqw   09/17/2018 15:30:02                                    1
  40265 0.00000 batch_test dduque       hqw   09/17/2018 15:30:02                                    1
  40266 0.00000 batch_test dduque       hqw   09/17/2018 15:30:02                                    1
  40267 0.00000 batch_test dduque       hqw   09/17/2018 15:30:02                                    1
  40268 0.00000 batch_test dduque       hqw   09/17/2018 15:30:02                                    1
  40269 0.00000 batch_test dduque       hqw   09/17/2018 15:30:02                                    1
  ....
```

To have a job wait until multiple other jobs are completed, just separate the
job names with commas
[https://stackoverflow.com/questions/11525214/wait-for-set-of-qsub-jobs-to-complete
(reference)]. For example, to have batch_test11 wait for all of jobs 1 through
10:
```
 qsub -N batch_test11 -hold_jid batch_test1,batch_test2,batch_test3,batch_test4,batch_test5,batch_test6,batch_test7,batch_test8,batch_test9,batch_test10  ./helloworld.sh 11
```
This can be used to have jobs run in groups.

If each of jobs 11 through 20 are told to hold for all of jobs 1 through 10,
then jobs 11 through 20 will all be released at the same time. This can be
useful when running thousands of short jobs. If you only hold for the job 100
before each one, you may block others who submitted jobs requesting many cores
from ever getting on the cluster since you will always have single node jobs
that will grab open cores. By holding in groups, others will be able to access
cores whenever a group is finishing. This is a good way to be able to use many
cores for thousands of single node jobs when no one else is running jobs, but
also avoid preventing others from accessing cores within an hour of them
submitting their job.

## Limiting number of threads

Because we need to share the resources, we need to limit the number of threads
that a job is using.  Here are ways to specify the number threads for
different codes

### Matlab

At the very beginning, execute
```
 maxNumCompThreads(num_threads)
```
This will limit the number of threads that Matlab will use for the
computations that it has control over (BLAS, parfor loops).  However, if you
run other libraries (like CPLEX) from Matlab, you need to limit the number of
threads explicitly again for such a library.

Alternatively, you can also start Matlab with the "-singleCompThread" option.
Then Matlab will take only a single thread.

**Apparently, Matlab does not always listen to the command above - so, when
your job is running, please check if it is really only using the specified
number of threads.**

### CPLEX

Set option "threads" to the number of threads.  In AMPL, you can do this with
```
 option cplex_options 'threads=8';
```
For Julia, when creating the optimization model do the following
```
using JuMP
using CPLEX
model = Model(with_optimizer(CPLEX.Optimizer))
MOI.set(model, MOI.NumberOfThreads(), 8)
```
