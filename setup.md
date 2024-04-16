---
layout: page
title: "Setup"
permalink: /setup/
root: ..
---

# Running the lessons on your local machine

## Training material

Download the training material copy & pasting the following command in the terminal:

~~~
$ git clone https://github.com/kerimoff/nf-training.git
$ cd nf-training
~~~
{: .language-bash}


## Training software

The simplest (and recommended) way to install the software for this course is using conda.

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




## In case you have problems in installing software with conda

Try your best to install all the software listed in environment.yml manually. If you still have no success then our training helpers can help you in the classroom.
