---
layout: page
title: "Setup"
permalink: /setup/
root: ..
---

# Running the lessons on your local machine

There are two items that you need to download:

1. The training material.
2. The training dataset.

## Training material

Download the training material copy & pasting the following command in the terminal:

~~~
$ git clone https://github.com/kerimoff/nf-training.git
$ cd nf-training
~~~
{: .language-bash}


## Training software

The simplest way to install the software for this course is using conda.

To install conda see [here](https://carpentries-incubator.github.io/introduction-to-conda-for-data-scientists/setup/).

To create the training environment run:

~~~
conda env create -f environment.yml
~~~
{: .language-bash}

Then activate the environment by running

~~~
conda activate nf-train
~~~
{: .language-bash}

### Data

Data is already in the ready format for usage.


## Visual Studio Code editor setup

Any text editor can be used to write Nextflow scripts. A recommended  code editor is [Visual Studio Code](https://code.visualstudio.com/).

Go to [Visual Studio Code](https://code.visualstudio.com/) and you should see a download button. The button or buttons should be specific to your platform and the download package should be  installable.


### Nextflow language support in Visual Studio Code

You can add Nextflow language support in Visual Studio Code by clicking the [install](https://marketplace.visualstudio.com/items?itemName=nextflow.nextflow) button on the Nextflow language extension.




## Nextflow install without conda

Nextflow can be used on any [POSIX](https://en.wikipedia.org/wiki/POSIX) compatible system (Linux, OS X, etc). It requires Bash and Java 8 (or later, up to 12) to be installed.

Windows systems may be supported using a POSIX compatibility layer like Cygwin (unverified) or, alternatively, installing it into a Linux VM using virtualization software like VirtualBox or VMware.

## Nextflow installation

Install the latest version of Nextflow copy & pasting the following snippet in a terminal window:

~~~
# Make sure that Java v8+ is installed:
java -version

# Install Nextflow
export NXF_VER=20.10.0
curl get.nextflow.io | bash
~~~


## Add Nextflow binary to your user's PATH:
~~~
mv nextflow ~/bin/
# OR system-wide installation:
# sudo mv nextflow /usr/local/bin
~~~
{: .language-bash}

Check the correct installation running the following command:

~~~
nextflow info
~~~
{: .language-bash}

Check the correct installation running the following command:

~~~
nextflow info
~~~
{: .language-bash}

## nf-core/tools installation without conda

### Pip

~~~
pip install nf-core
~~~
{: .language-bash}

{% include links.md %}
