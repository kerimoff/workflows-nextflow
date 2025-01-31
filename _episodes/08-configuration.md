---
title: "Nextflow configuration"
teaching: 30
exercises: 15
questions:
- "What is the difference between the workflow implementation and the workflow configuration?"
- "How do I configure a Nextflow workflow?"
- "How do I assign different resources to different processes?"
- "How do I separate and provide configuration for different computational systems?"
- "How do I change configuration settings from the default settings provided by the workflow?"
objectives:
- "Understand the difference between workflow implementation and configuration."
- "Understand the difference between configuring Nextflow and a Nextflow script."
- "Create a Nextflow configuration file."
- "Understand what a configuration scope is."
- "Be able to assign resources to a process."
- "Be able to refine configuration settings using process selectors."
- "Be able to group configurations into profiles for use with different computer infrastructures."
- "Be able to override existing settings."
- "Be able to inspect configuration settings before running a workflow."
keypoints:
- "Nextflow configuration can be managed using a Nextflow configuration file."
- "Nextflow configuration files are plain text files containing a set of properties."
- "You can define process specific settings, such as cpus and memory, within the `process` scope."
- "You can assign different resources to different processes using the process selectors `withName` or `withLabel`."
- "You can define a profile for different configurations using the `profiles` scope. These profiles can be selected when launching a pipeline execution by using the `-profile` command-line option"
- "Nextflow configuration settings are evaluated in the order they are read-in."
- "Workflow configuration settings can be inspected using `nextflow config <script> [options]`."
---

## Nextflow configuration

A key Nextflow feature is the ability to decouple the workflow implementation, which describes
the flow of data and operations to perform on that data, from the configuration settings required by the underlying execution platform. This enables the workflow to be portable, allowing it to run on different computational platforms such as an institutional HPC or cloud infrastructure, without needing to modify the workflow implementation.

We have seen earlier that it is possible to provide a `process` with
directives. These directives are process specific configuration settings.
Similarly, we have also provided parameters to our workflow which
are parameter configuration settings. These configuration settings
can be separated from the workflow implementation, into a
configuration file.

## Configuration files

Settings in a configuration file are sets of name-value pairs
(`name = value`). The `name` is a specific property to set,
while the `value` can be anything you can assign to a variable (see
[nextflow scripting](02-nextflow_scripting)), for example, strings,
booleans, or other variables.
It is also possible to access any variable defined in the
host environment such as `$PATH`, `$HOME`, `$PWD`, etc.

~~~
// nextflow.config
my_home_dir = "$HOME"
~~~
{: .language-groovy}

> ## Accessing variables in your configuration file
>
> Generally, variables and functions defined in a
> configuration file are not accessible from the
> workflow script. Only variables defined using the
> `params` scope and the `env` scope (without `env` prefix) can
> be accessed from the workflow script.
>
> ~~~
> workflow {
>     MY_PROCESS( Channel.fromPath(params.input) )
> }
> ~~~
> {: .language-groovy}
{: .callout}

Settings are also partitioned into
scopes, which govern the behaviour of different elements of the
workflow. For example, workflow parameters are governed from the
`params` scope, while process directives are governed from the `process` scope. A full list of the available scopes can be found in the
[documentation](https://www.nextflow.io/docs/latest/config.html#config-scopes). It is also possible to define your own scope.

Configuration settings for a workflow are often stored in the file
`nextflow.config` which is in the same directory as the workflow script.
Configuration can be written in either of two ways. The first is using
dot notation, and the second is using brace notation. Both forms
of notation can be used in the same configuration file.

An example of dot notation:
~~~
params.input = ''             // The workflow parameter "input" is assigned an empty string to use as a default value
params.outdir = './results'   // The workflow parameter "outdir" is assigned the value './results' to use by default.
~~~
{: .language-groovy }

An example of brace notation:
~~~
params {
    input  = ''
    outdir = './results'
}
~~~
{: .language-groovy }

Configuration files can also be separated into multiple files and
included into another using the `includeConfig` statement.

~~~
// nextflow.config
params {
    input  = ''
    outdir = './results'
}

includeConfig 'system_resources.config'
~~~
{: .language-groovy}
~~~
// system_resources.config
process {
    cpus = 1    // default cpu usage
    time = '1h' // default time limit
}
~~~
{: .language-groovy}

## How configuration files are combined

Configuration settings can be spread across several files. This also
allows settings to be overridden by other configuration files. The
priority of a setting is determined by the following order,
ranked from highest to lowest.

1. Parameters specified on the command line (`--param_name value`).
1. Parameters provided using the `-params-file` option.
1. Config file specified using the `-c` my_config option.
1. The config file named `nextflow.config` in the current directory.
1. The config file named `nextflow.config` in the workflow project directory (`$projectDir`: the directory where the script to be run is located).
1. The config file `$HOME/.nextflow/config`.
1. Values defined within the workflow script itself (e.g., `main.nf`).

If configuration is provided by more than one of these methods,
configuration is merged giving higher priority to configuration
provided higher in the list.

Existing configuration can be completely ignored by using `-C <custom.config>` to use only configuration provided in the `custom.config` file.

> ## Configuring Nextflow vs Configuring a Nextflow workflow
>
> Parameters starting with a single dash `-` (e.g., `-c my_config.config`) are configuration
> options for `nextflow`, while parameters starting with a double
> dash `--` (e.g., `--outdir`) are workflow parameters defined in the `params` scope.
>
> The majority of Nextflow configuration settings must be provided
> on the command-line, however a handful of settings can also
> be provided within a configuration file, such as
> `workdir = '/path/to/work/dir'` (`-w /path/to/work/dir`) or
> `resume = true` (`-resume`), and do not
> belong to a configuration scope.
{: .callout}

> ## Determine script output
> Determine the outcome of the following script executions.
> Given the script `print_message.nf`:
> 
> ~~~
> nextflow.enable.dsl = 2
> 
> params.message = 'hello'
> 
> workflow {
>     PRINT_MESSAGE(params.message)
> }
> 
> process PRINT_MESSAGE {
>     echo true
> 
>     input:
>     val my_message
> 
>     script:
>     """
>     echo $my_message
>     """
> }
> ~~~
> {: .language-groovy}
> 
> and configuration (`print_message.config`):
> 
> ~~~
> params.message = 'Are you tired?'
> ~~~
> {: .language-groovy}
> 
> What is the outcome of the following commands?
> 
> 1. `nextflow run print_message.nf`
> 1. `nextflow run print_message.nf --message '¿Que tal?'`
> 1. `nextflow run print_message.nf -c print_message.config`
> 1. `nextflow run print_message.nf -c print_message.config --message '¿Que tal?'`
> 
> > ## Solution
> > 1. 'hello' - Workflow script uses the value in `print_message.nf`
> > 1. '¿Que tal?' - The command-line parameter overrides the script setting.
> > 1. 'Are you tired?' - The configuration overrides the script setting
> > 1. '¿Que tal?' - The command-line parameter overrides both the script and configuration settings.
> {: .solution}
{: .challenge}

## Configuring process behaviour

Earlier we saw that `process` directives allow the specification of
settings for the task execution such as `cpus`, `memory`, `conda`
and other resources in the pipeline script. This is useful when
prototyping a small workflow script, however this ties the configuration
to the workflow, making it less portable. A good practice is to
separate the process configuration settings into another file.

The `process` configuration scope allows the setting of any process directives in the Nextflow configuration file.

For example:

~~~
// nextflow.config
process {
    cpus = 2
    memory = 8.GB
    time = '1 hour'
    publishDir = [ path: params.outdir, mode: 'copy' ]
}
~~~
{: .language-groovy }

> ## Unit values
>
> Memory and time duration units can be specified either using a string
> based notation in which the digit(s) and the unit can be separated by
> a space character, or
> by using the numeric notation in which the digit(s) and the unit are
> separated by a dot character and not enclosed by quote characters.
>
>  | String syntax   | Numeric syntax | Value                 |
>  |-----------------|----------------|-----------------------|
>  | '10 KB'         | 10.KB          | 10240 bytes           |
>  | '500 MB'        | 500.MB         | 524288000 bytes       |
>  | '1 min'         | 1.min          | 60 seconds            |
>  | '1 hour 25 sec' | -              | 1 hour and 25 seconds |
>
{: .callout}

These settings are applied to all processes in the workflow. A
process selector can be used to apply the configuration to a
specific process or group of processes.

### Process selectors

The resources for a specific process can be defined using `withName:`
followed by the process name ( either the simple name e.g., `'FASTQC'`,
or the fully qualified name e.g., `'NFCORE_RNASEQ:RNA_SEQ:SAMTOOLS_SORT'`),
and the directives within curly braces.
For example, we can specify different `cpus` and `memory` resources
for the processes `INDEX` and `FASTQC` as follows:

~~~
// configuration_process-names.config
process {
    withName: INDEX {
        cpus = 4
        memory = 8.GB
    }
    withName: FASTQC {
        cpus = 2
        memory = 4.GB
    }
}
~~~
{: .language-groovy }

When a workflow has many processes, it is inconvenient to specify
directives for all processes individually, especially if directives
are repeated for groups of processes. A helpful strategy is to annotate
the processes using the `label` directive (processes can have multiple
labels). The `withLabel` selector then allows the configuration of all
processes annotated with a specific label, as shown below:

~~~
// configuration_process_labels.nf
nextflow.enable.dsl=2

process P1 {

    label "big_mem"

    script:
    """
    echo P1: Using $task.cpus cpus and $task.memory memory.
    """
}

process P2 {

    label "big_mem"

    script:
    """
    echo P2: Using $task.cpus cpus and $task.memory memory.
    """
}

workflow {

    P1()
    P2()

}
~~~
{: .language-groovy}

~~~
// configuration_process-labels.config
process {
    withLabel: big_mem {
        cpus = 2
        memory = 4.GB
    }
}
~~~
{: .language-groovy}

Another strategy is to use process selector expressions. Both
`withName:` and `withLabel:` allow the use of regular expressions
to apply the same configuration to all processes matching a pattern.
Regular expressions must be quoted, unlike simple process names
or labels.

- The `|` matches either-or, e.g., `withName: 'INDEX|FASTQC'`
applies the configuration to any process matching the name `INDEX`
or `FASTQC`.
- The `!` inverts a selector, e.g., `withLabel: '!small_mem'` applies
the configuration to any process without the `small_mem` label.
- The `.*` matches any number of characters, e.g.,
`withName: 'NFCORE_RNASEQ:RNA_SEQ:BAM_SORT:.*'` matches all processes
of the workflow `NFCORE_RNASEQ:RNA_SEQ:BAM_SORT`.

A regular expression cheat-sheet can be found
[here](https://www.jrebel.com/system/files/regular-expressions-cheat-sheet.pdf) if you would like to
write more expressive expressions.

#### Selector priority

When mixing generic process configuration and selectors, the following
priority rules are applied (from highest to lowest):

1. `withName` selector definition.
1. `withLabel` selector definition.
1. Process specific directive defined in the workflow script.
1. Process generic `process` configuration.

> ## Process selectors
>
> Create a Nextflow config, `process-selector.config`, specifying
> different `cpus` and `memory` resources for the two processes
> `P1` (cpus 1 and memory 2.GB) and
> `P2` (cpus 2 and memory 1.GB),
> where `P1` and `P2` are defined as follows:
>
> ~~~
> // process-selector.nf
> nextflow.enable.dsl=2
>
> process P1 {
>   echo true
>
>   script:
>   """
>   echo P1: Using $task.cpus cpus and $task.memory memory.
>   """
> }
>
> process P2 {
>   echo true
>
>   script:
>   """
>   echo P2: Using $task.cpus cpus and $task.memory memory.
>   """
> }
>
> workflow {
>   P1()
>   P2()
> }
> ~~~
> {: .language-groovy }
>
> > ## Solution
> > ~~~
> > // process-selector.config
> > process {
> >     withName: P1 {
> >         cpus = 1
> >         memory = 2.GB
> >     }
> >     withName: P2 {
> >         cpus = 2
> >         memory = 1.GB
> >     }
> > }
> > ~~~
> > {: .language-groovy}
> > ~~~
> > $ nextflow run process-selector.nf -c process-selector.config 
> > ~~~
> > {: .language-bash}
> > ~~~
> > N E X T F L O W  ~  version 21.04.0
> > 
> > Launching `process-selector.nf` [clever_borg] -
> > revision: e765b9e62d
> > executor >  local (2)
> > [de/86cef0] process > P1 [100%] 1 of 1 ✔
> > [bf/8b332e] process > P2 [100%] 1 of 1 ✔
> > P2: Using 2 cpus and 1 GB memory.
> >
> > P1: Using 1 cpus and 2 GB memory.
> > ~~~
> >  {: .output}
> {: .solution}
{: .challenge}

#### Dynamic expressions

A common scenario is that configuration settings may depend on the
data being processed. Such settings can be dynamically expressed
using a closure. For example, we can specify the `memory` required
as a multiple of the number of `cpus`. Similarly, we can publish
results to a subfolder based on the sample name.

~~~
process FASTQC {

    input:
    tuple val(sample), path(reads)

    script:
    """
    fastqc -t $task.cpus $reads
    """
}
~~~
{: .language-groovy }

~~~
// nextflow.config
process {
    withName: FASTQC {
        cpus = 2
        memory = { 2.GB * task.cpus }
        publishDir = { "fastqc/$sample" }
    }
}
~~~
{: .language-groovy }

## Configuring execution platforms

Nextflow supports a wide range of execution platforms, from
running locally, to running on HPC clusters or cloud infrastructures.
See https://www.nextflow.io/docs/latest/executor.html for the
full list of supported executors.

![nf-executors](https://seqera.io/training/img/nf-executors.png)

The default executor configuration is defined within the `executor`
scope (https://www.nextflow.io/docs/latest/config.html#scope-executor).
For example, in the config below we specify the executor as
Sun Grid Engine, `sge` and the number of tasks the executor will
handle in a parallel manner (`queueSize`) to 10.

~~~
// nextflow.config
executor {
    name = 'sge'
    queueSize = 10
}
~~~
{: .language-groovy }

The `process.executor` directive allows you to override
the executor to be used by a specific process. This can be
useful, for example, when there are short running tasks
that can be run locally, and are unsuitable for submission
to HPC executors (check for guidelines on best practice use
of your execution system). Other process directives such as
`process.clusterOptions`, `process.queue`, and `process.machineType`
can be also be used to further configure processes depending
on the executor used.  

~~~
//nextflow.config
executor {
    name = 'sge'
    queueSize = 10
}
process {
    withLabel: 'short' {
        executor = 'local'
    }
}
~~~
{: .language-groovy }

## Configuring software requirements

An important feature of Nextflow is the ability to manage
software using different technologies. It supports the Conda package
management system, and container engines such as Docker, Singularity,
Podman, Charliecloud, and Shifter. These technologies
allow one to package tools and their dependencies into a software environment
such that the tools will always work as long as the environment can be loaded.
This facilitates portable and reproducible workflows.
Software environment specification is managed from the `process` scope,
allowing the use of process selectors to manage which processes
load which software environment. Each technology also has its own
scope to provide further technology specific configuration settings.

### Software configuration using Conda

Conda is a software package and environment management system that runs on
Linux, Windows, and Mac OS. Software packages are bundled into
Conda environments along with their dependencies for a particular
operating system (Not all software is supported on all operating systems).
Software packages are tied to conda channels, for example,
bioinformatic software packages are found and installed
from the BioConda channel.

A Conda environment can be configured in several ways:
- Provide a path to an existing Conda environment.
- Provide a path to a Conda environment specification file (written in YAML).
- Specify the software package(s) using the
`<channel>::<package_name>=<version>` syntax (separated by spaces),
which then builds the Conda environment when the process is run.

~~~
process {
    conda = "/home/user/miniconda3/envs/my_conda_env"
    withName: FASTQC {
        conda = "environment.yml"
    }
    withName: SALMON {
        conda = "bioconda::salmon=1.5.2"
    }
}
~~~
{: .language-groovy }

There is an optional `conda` scope which allows you to control the
creation of a Conda environment by the Conda package manager.
For example, `conda.cacheDir` specifies the path where the Conda
environments are stored. By default this is in `conda` folder of the `work` directory.

> ## Define a software requirement in the configuration file using conda
>
> Create a config file for the Nextflow script `configure_fastp.nf`.
> Add a conda directive for the process name `FASTP` that includes the bioconda package `fastp`, version 0.12.4-0.
> **Hint** You can specify the conda packages using the syntax `<channel>::<package_name>=<version>` e.g. `bioconda::salmon=1.5.2`
> Run the Nextflow script `configure_fastp.nf` with the configuration file using the `-c` option.
>
> ~~~
> // configure_fastp.nf
> nextflow.enable.dsl = 2
>
> params.input = "data/yeast/reads/ref1_1.fq.gz"
>
> workflow {
>     FASTP( Channel.fromPath( params.input ) )
>     FASTP.out.view()
> }
>
> process FASTP {
>     input:
>     path read
> 
>     output:
>     stdout
> 
>     script:
>     """
>     fastp -A -i ${read} -o out.fq 2>&1
>     """
> }
> ~~~
> {: .language-groovy }
>
> > ## Solution
> > ~~~
> > // fastp.config
> > process {
> >     withName: 'FASTP' {
> >         conda = "bioconda::fastp=0.12.4-0"
> >     }
> > }
> > ~~~
> > {: .language-groovy}
> >
> > ~~~
> > nextflow run configure_fastp.nf -c fastp.config -process.echo
> > ~~~
> > {: .language-bash}
> > ~~~
> > N E X T F L O W  ~  version 21.04.0
> > Launching `configure_fastp.nf` [berserk_jepsen] - revision: 28fadd2486
> > executor >  local (1)
> > [c1/c207d5] process > FASTP (1) [100%] 1 of 1 ✔
> > Creating Conda env: bioconda::fastp=0.12.4-0 [cache /home/training/work/conda/env-a7a3a0d820eb46bc41ebf4f72d955e5f]
> > ref1_1.fq.gz 58708
> > Read1 before filtering:
> > total reads: 14677
> > total bases: 1482377
> >
> > Q20 bases: 1466210(98.9094%)
> > Q30 bases: 1415997(95.5221%)
> > 
> > Read1 after filtering:
> > total reads: 14671
> > total bases: 1481771
> > Q20 bases: 1465900(98.9289%)
> > Q30 bases: 1415769(95.5457%)
> >
> > Filtering result:
> > reads passed filter: 14671
> > reads failed due to low quality: 6
> > reads failed due to too many N: 0
> > reads failed due to too short: 0
> >
> > JSON report: fastp.json
> > HTML report: fastp.html
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

### Software configuration using Docker

Docker is a container technology. Container images are
lightweight, standalone, executable package of software
that includes everything needed to run an application:
code, runtime, system tools, system libraries and settings.
Containerized software is intended to run the same regardless
of the underlying infrastructure, unlike other package management
technologies which are operating system dependant (See
the [published article on Nextflow](https://doi.org/10.1038/nbt.3820)).
For each container image used, Nextflow uses Docker to spawn
an independent and isolated container instance for each process task.

To use Docker, we must provide a container image path using the
`process.container` directive, and also enable docker in the docker
scope, `docker.enabled = true`. A container image path takes the form
`(protocol://)registry/repository/image:version--build`.
By default, Docker containers
run software using a privileged user. This can cause issues,
and so it is also a good idea to supply your user and group
via the `docker.runOptions`.

~~~
process.container = 'quay.io/biocontainers/salmon:1.5.2--h84f40af_0'
docker.enabled = true
docker.runOptions = '-u $(id -u):$(id -g)'
~~~
{: .language-groovy }

### Software configuration using Singularity

Singularity is another container technology, commonly used on
HPC clusters. It is different to Docker in several ways. The
primary differences are that processes are run as the user,
and certain directories are automatically "mounted" (made available)
in the container instance. Singularity also supports building
Singularity images from Docker images, allowing Docker image paths
to be used as values for `process.container`.

Singularity is enabled in a similar manner to Docker.
A container image path must be provided using `process.container` and
singularity enabled using `singularity.enabled = true`.

~~~
process.container = 'https://depot.galaxyproject.org/singularity/salmon:1.5.2--h84f40af_0'
singularity.enabled = true
~~~
{: .language-groovy }

> ## Container protocols
>
> The following protocols are supported:
> - `docker://``: download the container image from the Docker Hub and convert it to the Singularity format (default).
> - `library://``: download the container image from the Singularity Library service.
> - `shub://``: download the container image from the Singularity Hub.
> - `docker-daemon://`: pull the container image from a local Docker installation and convert it to a Singularity image file.
> - `https://`: download the singularity image from the given URL.
> - `file://`: use a singularity image on local computer storage.
{: .callout}

## Configuration profiles

One of the most powerful features of Nextflow configuration is to
predefine multiple configurations or `profiles` for different
execution platforms. This allows a group of predefined settings to
be called with a short invocation, `-profile <profile name>`.

Configuration profiles are defined in the `profiles` scope,
which group the attributes that belong to the same profile
using a common prefix.

~~~
//configuration_profiles.config
profiles {

    standard {
        params.genome = '/local/path/ref.fasta'
        process.executor = 'local'
    }

    cluster {
        params.genome = '/data/stared/ref.fasta'
        process.executor = 'sge'
        process.queue = 'long'
        process.memory = '10GB'
        process.conda = '/some/path/env.yml'
    }

    cloud {
        params.genome = '/data/stared/ref.fasta'
        process.executor = 'awsbatch'
        process.container = 'cbcrg/imagex'
        docker.enabled = true
    }

}
~~~
{: .language-groovy }

This configuration defines three different profiles: `standard`,
`cluster` and `cloud` that set different process configuration
strategies depending on the target execution platform. By
convention the standard profile is implicitly used when no
other profile is specified by the user. To enable a specific
profile use `-profile` option followed by the profile name:

~~~
nextflow run <your script> -profile cluster
~~~
{: .language-bash}

> ## Configuration order
>
> Settings from profiles will override general settings
> in the configuration file. However, it is also important
> to remember that configuration is evaluated in the order it
> is read in. For example, in the following example, the `publishDir`
> directive will always take the value 'results' even when the
> profile `hpc` is used. This is because the setting is evaluated
> before Nextflow knows about the `hpc` profile. 
>
> ~~~
> params.results = 'results'
> process.publishDir = params.results
> profiles {
>     hpc {
>         params.results = '/long/term/storage/results'
>     }
> }
> ~~~
> {: .language-groovy}
> If the `publishDir`
> directive is moved to after the `profiles` scope, then `publishDir`
> will use the correct value of `params.results`.
> 
> ~~~
> params.results = 'results'
> profiles {
>     hpc {
>         params.results = '/long/term/storage/results'
>     }
> }
> process.publishDir = params.results
> ~~~
> {: .language-groovy}
{: .callout}

## Inspecting the Nextflow configuration

You can use the command `nextflow config` to print the resolved
configuration of a workflow. This allows you to see what settings
Nextflow will use to run a workflow.

~~~
$ nextflow config configure_fastp.nf -profile cloud
~~~
{: .language-bash}

~~~
params {
   genome = '/data/stared/ref.fasta'
}

process {
   executor = 'awsbatch'
   container = 'cbcrg/imagex'
}

docker {
   enabled = true
}
~~~
{: .output}

{% include links.md %}
