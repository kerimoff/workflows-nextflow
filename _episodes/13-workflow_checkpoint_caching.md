---
title: "Workflow caching and checkpointing"
teaching: 30
exercises: 10
questions:
- "How can I restart a Nextflow workflow after an error?"
- "How can I add new data to a workflow?"
- "Where can I find intermediate data and results?"
objectives:
- "Resume a Nextflow workflow using the `-resume` option."
- "Restart a Nextflow workflow using new data."
keypoints:
- "Nextflow automatically keeping track of all the processes executed in your pipeline via  caching  and checkpointing."
- "You can restart a Nextflow workflow using new data using new data skipping steps that have been successfully executed."
- "Nextflow stores intermediate data in a working directory."
---

One of the key features of a modern workflow management system, like Nextflow, is the ability to restart a pipeline after an error from the last successful process execution. Nextflow achieves this by automatically keeping track of all the processes executed in your pipeline via  caching  and checkpointing.

## Resume

To restart from the last successfully executed process we add the command line option `-resume` to the Nextflow command.

For example, the command below would resume the `wc.nf` script from the last successful process.

~~~
$ nextflow run wc.nf --input 'data/yeast/reads/ref1*.fq.gz' -resume
~~~
{: .language-bash}

We can see in the output  that the results from the process `NUM_LINES` has been retrieved from the cache.
~~~
Launching `wc.nf` [condescending_dalembert] - revision: fede04a544
[c9/2597d5] process > NUM_LINES (1) [100%] 2 of 2, cached: 2 ✔
ref1_1.fq.gz 58708

ref1_2.fq.gz 58708
~~~
{: .output}


> ## Resume a pipeline
> Resume the Nextflow script `wc.nf` by re-running the command and adding the parameter `-resume`
> and the parameter `--input 'data/yeast/reads/temp33*'`:
>
> {: .language-bash}
> > ## Solution
> > ~~~
> > $ nextflow run wc.nf --input 'data/yeast/reads/temp33*' -resume
> > ~~~
> > If your previous run was successful the output will look similar to this:
> >
> > ~~~
> > N E X T F L O W  ~  version 20.10.0
> > Launching `wc.nf` [nauseous_leavitt] - revision: fede04a544
> > [21/6116de] process > NUM_LINES (4) [100%] 6 of 6, cached: 6 ✔
> >temp33_3_2.fq.gz 88956
> >
> >temp33_3_1.fq.gz 88956
> >
> >temp33_1_1.fq.gz 82372
> >
> >temp33_2_2.fq.gz 63116
> >
> >temp33_1_2.fq.gz 82372
> >
> >temp33_2_1.fq.gz 63116
> >  ~~~
> > {: .output }
> > You will see that the execution of the process `NUMLINES` is actually skipped (cached text appears), and its results are retrieved from the cache.
> {: .solution}
{: .challenge}


## The Work directory

By default the pipeline results are cached in the directory `work` where the pipeline is launched.

We can use the Bash `tree` command to list the contents of the work directory.
**Note:** By default tree does not print hidden files (those beginning with a dot `.`). Use the `-a`   to view  all files.

~~~
$ tree -a work
~~~
{: .language-bash}

~~~
work/
├── 5e
│   └── dd382c2b8777ad43e24c35f50dc0bf
│       ├── .command.begin
│       ├── .command.err
│       ├── .command.log
│       ├── .command.out
│       ├── .command.run
│       ├── .command.sh
│       ├── .exitcode
│       └── ref1_1.fq.gz -> /Users/kerimov/Work/GitHub/nf-training/data/yeast/reads/ref1_1.fq.gz
├── 71
│   └── d4e5514667f56a8e6f4d237f86d16e
│       ├── .command.begin
│       ├── .command.err
│       ├── .command.log
│       ├── .command.out
│       ├── .command.run
│       ├── .command.sh
│       ├── .exitcode
│       └── temp33_3_2.fq.gz -> /Users/kerimov/Work/GitHub/nf-training/data/yeast/reads/temp33_3_2.fq.gz
[...truncated...]
~~~
{: .output }

Each file under the unqiue hash directory starting with `.command` and `.exitcode` has its own responsibility. It is very informative to know what are their purpose:

**.exitcode** contains the exit code message after the task is executed. If the task is executed successfully this file contains `0` (e.g. exitcode is zero, there were no errors) else the code is different than zero.

**.command.sh** contains the script has been executed for the task.
~~~
#!/bin/bash -ue
printf 'temp33_3_2.fq.gz '
gunzip -c temp33_3_2.fq.gz | wc -l
~~~
{: .output }

**.command.err** contains error meesages in case they exist (e.g. the task failed to execute). If the task was executed successfully this file will remain emplty.

**.command.log** contains all the log messages generated during the execution of script _.command.sh_

**.command.out** contains all the standart output generated during the execution of script _.command.sh_

**.command.begin** it is the file created when the task starts to be executed. Usually remains empty

**.command.run** a wrapper file to run the script in the specified environment. Nextflow specifies where to submit/run (e.g. local, HPC, cloud) and how to run (with container, with conda env etc.) the job by changing this file.

You will see the input Fastq files are symbolically linked to their original location. If the process input contains a file then this file will be staged (generated a symlink in work directory from the original file) under unique hash directory and used by script.

## How does resume work?

The mechanism works by assigning a unique ID to each task. This unique ID is used to create a separate execution directory, called the working directory, where the tasks are executed and the results stored. A task’s unique ID is generated as a 128-bit hash number obtained from a composition of the task’s:

* Inputs values
* Input files
* Command line string
* Container ID
* Conda environment
* Environment modules
* Any executed scripts in the bin directory

When we resume a workflow Nextflow uses this unique ID to check if:

1. The working directory exists
1. It contains a valid command exit status
1. It contains the expected output files.

If these conditions are satisfied, the task execution is skipped and the previously computed outputs are applied. When a task requires recomputation, ie. the conditions above are not fulfilled, the downstream tasks are automatically invalidated.


Therefore, if you modify some parts of your script, or alter the input data using `-resume`, will only execute the processes that are actually changed.

The execution of the processes that are not changed will be skipped and the cached result used instead.

This helps a lot when testing or modifying part of your pipeline without having to re-execute it from scratch.

> ## Modify Nextflow script and re-run.
>  Alter the timestamp on the file temp33_3_2.fq.gz using the UNIX `touch` command.
> ~~~~
> $ touch data/yeast/reads/temp33_3_2.fq.gz
> ~~~~
> {: .language-bash}
> Run command below.
> ~~~
> $ nextflow run wc.nf --input 'data/yeast/reads/temp33*' -resume
> ~~~
> How many processes will be cached and how many will run ?
> {: .language-bash}
> > ## Solution
> > The output will look similar to this:
> >
> > ~~~
> > N E X T F L O W  ~  version 20.10.0
> > Launching `wc.nf` [gigantic_minsky] - revision: fede04a544
> > executor >  local (1)
> > [71/d4e551] process > NUM_LINES (5) [100%] 6 of 6, cached: 5 ✔
> > temp33_1_2.fq.gz 82372
> >
> > temp33_3_1.fq.gz 88956
> >
> > temp33_2_1.fq.gz 63116
> >
> > temp33_1_1.fq.gz 82372
> >
> > temp33_2_2.fq.gz 63116
> >
> > temp33_3_2.fq.gz 88956
> >  ~~~
> > {: .output }
> > As you changed the timestamp on one file it will only re-run that process.
> > The results from the other 5 processes are cached.
> {: .solution}
{: .challenge}



### Specifying another work directory

Depending on your script, this work folder can take a lot of disk space.
You can specify another work directory using the command line option `-w`

~~~
$ nextflow run <script> -w /some/scratch/dir
~~~
{: .language-bash}

### Clean the work directory

If you are sure you won’t resume your pipeline execution, clean this folder periodically using the command `nextflow clean`.

~~~
$ nextflow clean [run_name|session_id] [options]
~~~
{: .language-bash}

Typically, results before the last successful result are cleaned:

~~~
$ nextflow clean -f -before [run_name|session_id]
~~~
{: .language-bash}
