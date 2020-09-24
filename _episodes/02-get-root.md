---
title: "Get ROOT"
teaching: 10
exercises: 5
questions:
- "How to install ROOT on my system or get access to systems with ROOT pre-installed?"
objectives:
- "Find the most efficient way for you to get access to ROOT"
- "Install ROOT or connect to a machine with ROOT already installed"
keypoints:
- "ROOT is accessible in several different ways"
- "Detailed up-to-date instructions can be found at [https://root.cern/install](https://root.cern/install)"
---

> This section shows you multiple ways to get ROOT. Find below solutions to run ROOT locally, on your VM or Docker container!
{: .testimonial}

## Root on the CMS open data environments

[CVMFS](https://cernvm.cern.ch/portal/filesystem) is a software distribution service that is already set up on many HEP systems. CMSSW including ROOT is distributed via CVMFS, but also other software stacks are available that contain ROOT.

If you worked out the Virtual Machine or the Docker pre-exercises you should have already access to CVMFS.  For convenience, we will assume you will be working in one of these environments.

You can get ROOT via CVMFS through the LCG releases. All information about the releases and contained packages can be found at [http://lcginfo.cern.ch](http://lcginfo.cern.ch).

The following setup line works in the SLC6 shell and the "Outer shell" (SLC7) of the open data VM.  It also works in the SLC6 Docker container, if you mounted `cvfms` according to [these instructions](https://cms-opendata-workshop.github.io/workshop-lesson-docker-preexercises/04-cvmfs-and-brilcalc/index.html).  If the command is executed, you will get a pre-built ROOT setup ready to use:

```bash
source /cvmfs/sft.cern.ch/lcg/views/LCG_95/x86_64-slc6-gcc8-opt/setup.sh
```

One drawback of this way of accessing ROOT is that it will be a little slow.  So, if you prefer and feel comfortable playing around, it would be faster if you installed ROOT locally (following the options below).


## Conda

[Conda](https://docs.conda.io/en/latest/) is a package management system and environment management system, which can install ROOT in a few minutes on Linux and MacOS.

The fastest way to get ROOT is [installing Miniconda](https://docs.conda.io/en/latest/miniconda.html) (a minimal Conda installation) and then run the following two commands:

```bash
conda create -c conda-forge --name <my-environment> root
conda activate <my-environment>
```

You can find detailed information about ROOT on Conda in [this blog post](https://iscinumpy.gitlab.io/post/root-conda/).

## Docker

If you want to use ROOT in a CI system (e.g. GitLab pipelines or GitHub actions), most likely the software will be made available via Docker. The official ROOT docker containers can be found at [https://hub.docker.com/r/rootproject/root](https://hub.docker.com/r/rootproject/root). The different base images and ROOT versions are encoded in the tags, for example `6.22.00-ubuntu20.04`, and `latest` will get you the latest ROOT release (v6.22) based on Ubuntu 20.04. If you want to try it, [get Docker](https://docs.docker.com/get-docker/) and run the following command to start the container with a bash shell:

```bash
run --rm -it rootproject/root:6.22.02-conda /bin/bash
```

## Binary releases and packages

The classic way to distribute software, besides the source code, are plain binary releases. You can download these from the release pages on [https://root.cern/install/all_releases](https://root.cern/install/all_releases) for all major MacOS and Linux versions. If you choose this installation method, make sure [ROOT dependencies](https://root.cern/install/dependencies) are installed on your system. Complete installation instructions for binary releases are available [here](https://root.cern/install/#download-a-pre-compiled-binary-distribution).

In addition, for some Linux distributions, the ROOT community maintains packages in the respective package managers. You can find a list of maintained packages at [https://root.cern/install/#linux-package-managers](https://root.cern/install/#linux-package-managers).

### CMSSW

If you have already a CMSSW area set up, you will be able to find out which version of ROOT you have access to by typing

~~~
scram tool list | grep root
~~~
{: .language-bash}

The version of ROOT that comes with our CMSSW release for open data (the one set up on the VM or Docker), however, is a bit old.  We will need ROOT version > 6.16.

## Verify the ROOT version

Since ROOT has a long history and numerous releases, on old systems such as Scientific Linux 6 (the one used to analyze open data) you may find correspondingly old ROOT versions. However, with the following commands you can easily verify your ROOT version and also find expert details about the ROOT configuration!

```bash
# ROOT version and build tag
root --version

# Again the ROOT version (this also works with older ROOT versions)
root-config --version

# Check that ROOT was built with C++14 support
# The output must contain one of -std=c++{14,17,1z} so that all code examples of this lesson run!
root-config --cflags

# List all the ROOT configuration options that can be checked
root-config --help
```

> ## Find your way to access ROOT!
> For the exercises later you need at least ROOT 6.16 and C++14 support. Feel free to set up for yourself your preferred environment satisfying this requirement!
{: .challenge}


> ## Fallback solution
> As a fallback solution you can always get access to ROOT from the VM or Docker container through CVMFS.  Simply run this setup:
>
> ~~~
> source /cvmfs/sft.cern.ch/lcg/views/LCG_95/x86_64-slc6-gcc8-opt/setup.sh
> ~~~
> {: .language-bash}
{: .solution}

{% include links.md %}
