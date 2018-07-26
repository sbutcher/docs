Title: Python performance in containers
Date: 2017-7-03 10:20
Category: Benchmarking
Author: Simon Butcher
Tags: containers, benchmarking, python

[Singularity](http://singularity.lbl.gov/) is a container solution designed for HPC.
Due to the secure and simple design, it can be easily used to provide
applications for use with HPC clusters where other containers, such as Docker
would not be suitable.

## Testing python inside containers

Linux containers offer a great way to compare software performance on a production system without installing extra packages on your shared filesystem.

After early tests on 3 different node type using python 2.7.8 on CentOS6 containers showed SCL python running the same code between 16-25% more slowly than a self-compiled python 2.7.8 running natively, further tests were performed, instead with CentOS 7.

A single-core python job to find [primes of a large number](https://github.com/sbutcher/python_test/blob/master/python_prime2.py) was run natively on a compute node via the Univa Grid Engine job scheduler. The job was then repeated on the same node inside a container running the same version of python compiled with gcc, and also a packaged python provided by the CentOS [Software Collections Library](https://wiki.centos.org/AdditionalResources/Repositories/SCL). The SCL is marketed as a simple way to get multiple versions of python on your enterprise OS without having to compile new versions. Jobs were run using CentOS 7.3 and python 2.7.13 supplied in the following ways:

* (A) Native OS
* (B) Singularity container, python compiled from source
* (C) Singularity container, python compiled from source, configured with --enable-optimizations flag
* (D) Singularity container, python supplied via CentOS SCL

Where python was compiled, the same gcc version 4.8.5 was used. Tasks A and B can be compared directly to establish if there is any performance impact on using containers vs native OS.

## Method

Each task was run multiple times on an Lenovo Nextscale nx360m5 compute node with 2xXeon E5-2683v4 processors. The first experiment involved running one job at a time on an exclusive node (in total 10 jobs for each of the 4 different task types). The second experiment ran batches of 10 jobs at a time on an exclusive node, grouped by task type.

Wallclock values were taken from the Grid Engine `qacct` command. The total wallclock time is reported by the job scheduler, and includes any overheads a container may introduce, such as time to load a container file. The measured time does not include the time taken to perform pre-requisite tasks such as creating the re-usable container or compile python. Jobs were run on a node with no other jobs running, but part of a production system that is susceptible to external influences such as GPFS filesystem load and system temperatures.

## Results

The plot shows the time taken to complete each job. Results from both experiments are shown.

<img src="/rplot-both.png">

#### Mean values over 10 runs on Xeon E5-2683v4 (jobs running separately)

| Task | Walltime/s | I/O ops/s | Memory/MB |
| --- | --: | --: | --: |
| Native  | 4682 | 3759 | 560 |
| Container  | 4608 | 3971 | 554 |
| Container,<br/> enable-optimisations | 9544 | 3948 | 1142 |
| Container, SCL  | 5564 | 4009 | 676 |

#### Mean values over 10 runs on Xeon E5-2683v4 (like-tasks run simultaneously)

| Task | Walltime/s | I/O op/s | Memory/MB |
| --- | --: | --: | --: |
| Native  | 5060 | 3779 | 605 |
| Container  | 5066 | 3966 | 605 |
| Container,<br/> enable-optimisations | 10440 | 3967 | 1244 |
| Container, SCL  | 6022 | 4033 | 726|

## Summary

It can be observed from the results that for the single core, CPU-bound job, the python compiled inside a container produces similar results to the compiled python on the native OS. Therefore, the container does not affect performance, in terms of CPU resource or RAM. In fact, the python container was often faster for the separate runs.

The simultaneous run tasks showed less jitter in the results, as expected, since all jobs of each type were run at the same time. The average runtime was increased slightly for all tasks, including the native OS. Some thermal throttling may be occurring on the CPU as 10 processors would be maxed out instead of 1 for the job duration.

The SCL python runs consistently slower than the compiled python on all nodes types, suggesting that, although a convenient method for system administrators to provide alternate versions, it may not be suitable for HPC environments. A container running python would perform better.

The python compiled with `--enable-optimizations` performed very poorly, and should be a lesson to system administrators not to blindly follow suggestions without testing. Quite why it performed so badly requires further investigation.

Containers provide an excellent way to provision tricky applications, particularly in the Bioinformatics and Deep-learning disciplines, but additionally provide a safe and easy way to gain performance improvements and new features offered by the latest and greatest versions of common applications, while offering an easy way to compare different configuration and compilation options that, as we have seen, could have significant impact on performance, which is critical in an HPC environment.

## Data files

The Singularity definition files used for these tests can be found on [Github](https://github.com/sbutcher/python_test).

## References

* [Python project](https://python.org)
* [CentOS](https://centos.org)
* [Singularity](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0177459)
* Work was performed on QMUL's [Apocrita Cluster](http://doi.org/10.5281/zenodo.438045)
