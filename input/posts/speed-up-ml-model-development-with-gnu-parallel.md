Title: Speed up ML model development with GNU Parallel
Lead: for fun and profit.
Published: 4/6/2017
Tags: 
    - parallel-computing
    - gnu-parallel
    - bash
    - machine-learning
    - python
---

## Introduction

Recently, I feel like I've found the Holy Grail of parallel/distributed computation - the [GNU Parallel](https://www.gnu.org/software/parallel/) program. This program allows you to run any command, script, pipeline or program in parallel with different arguments, possibly even distributing the jobs between several nodes. In contrast to frameworks such as Hadoop, it practically doesn't require any setup and has a very small overhead (perhaps it's not as robust though, and may scale worse for big data). I believe that any machine learning dev who works on "medium data", has access to a few Linux machines (e.g. in academia or a small startup/company) and would like to find these tricky hyperparameters more efficiently can benefit from this program. Especially if you're working with a kind of model that's not easy to parallalelize, or for any reason works only on a single core.

My motivation for this post is that I've used GNU Parallel to set up a framework for distributed language model development with several stages - configuration, data preprocessing, training, testing and results presentation, and I was very satisfied with the results. It took some searching on the Internet to set it up, so I hope to simplify the process for others by writing this. If you find it useful and want more, GNU Parallel has a [very thorough tutorial](https://www.gnu.org/software/parallel/parallel_tutorial.html).

## Installation

I assume you're working on a Linux machine, so that one's easy - either use your package manager, or install the latest version using the magic command (taken from Parallel's tutorial):

    (wget -O - pi.dk/3 || curl pi.dk/3/ || fetch -o - http://pi.dk/3) | bash

## Some examples

I will show you how to use the GNU Parallel either on a local machine or with a set of nodes. The code for these examples is also available on [my GitHub profile](https://github.com/pzelasko/parallel-example).

Let's start with what we're going to compute - as a toy example, I've chosen to adapt the [scikit-learn library tutorial example](http://scikit-learn.org/stable/tutorial/basic/tutorial.html) and run it using Python 3.6. The script `example.py` uses an [existing dataset named Iris](http://scikit-learn.org/stable/auto_examples/datasets/plot_iris_dataset.html#sphx-glr-auto-examples-datasets-plot-iris-dataset-py) and trains SVM classifier to distinguish between three different types of irises. It accepts arguments `--set-gamma` and `--set-c`, which are hyperparameters for the SVM model - these are the guys we want to find the best values for. The `example.py` script saves the model to file and prints accuracy for the hyperparameters it received.

Parallelization on the local machine can be done with a simple command:

    $ parallel \
        ./example.py --set-gamma {1} --set-c {2} \
        ::: 0.1 1 10 \
        ::: 0.2 2 20

Notice that the first line (1) is just the invocation of the program, without any additional arguments - it will just use the default settings, e.g. utilizing all the available cores (1 per job). The second line (2) is the invocation of the script running our job, with placeholders `{1}` and `{2}` instead of arguments. These placeholders will be replaced before the command is run with values froms lines (3) and (4) - notice the `:::` syntax, after which there appears a list of values that should be tested. In this example, we're running with values for gamma=(0.1, 1, 10) and C=(0.2, 2, 20). GNU Parallel will by default run the command with every combination of values from both lists, so in this example it's gonna be 9 jobs.

Now, let's go to the distributed part - I'm gonna assume that you've set up the remote nodes to ssh without a password, and that they also have Python installed along with the scikit-learn library (or modify the example to use Docker ;)). Distribution of the jobs is really simple now - we're just gonna extend the previous command a little:

    $ parallel --sshloginfile nodefile --workdir ~/parallel-example --basefile example.py --return "iris_svm_g{1}_c{2}.mdl" \
        ./example.py --set-gamma {1} --set-c {2} \
        ::: 0.1 1 10 \
        ::: 0.2 2 20

First of all, notice that only the line (1) has changed. Let's see each of the options:
- `--sshloginfile nodefile` indicates that the nodes are listed in a file named nodefile. The nodefile contains one node address per line, with optional prefix specifying number of cores to use (e.g. `32/192.156.0.200`, to use up to 32 cores on the machine found at address 192.156.0.200). Also, `:` is the special address which includes the local machine in the list of nodes;
- `--workdir ~/parallel-example` specifies where working directory, which will be created on the remote node. Our script will be saving the trained model there;
- `--basefile example.py` tells GNU Parallel that the file `example.py` has to be copied (rsynced) to the remote node before any job is launched. If every job had a separate input file, we could use the `--transferfile` option (see the tutorial);
- `--return "iris_svm_g{1}_c{2}.mdl"` will first fill the placeholders with values specific for the launched job, and after it's finished, it will try to copy (rsync) the specified file (or directory) back the machine which launched the task. By default, the file will also remain on the remote node (that can be changed with `--cleanup`).

## Conclusions

Hopefully, this brief intro to GNU Parallel will make the first steps gentle for you. Of course there's a lot more options and possibilities than what I've shown - for that, you should really read its official tutorial. Once again, it's definitely a lot easier and faster to set up GNU Parallel than any "serious" distributed computation framework.

