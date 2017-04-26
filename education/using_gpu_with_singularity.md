---
layout: page
title: GPU-enabled Singularity
permalink: /education/gpu_singularity/
---

# GPU-enabled Singularity #

Similar to Docker, a Singularity container (image) is a self-contained filesystem, operating system and software stack. As Singularity does not require a root-level daemon to run its images it is compatible for use with ShARC's scheduler inside your job scripts. The running images also uses the credentials of the person calling it.

This in-practice means that an image created on your local machine with all your research software installed for local development will also run on the ShARC cluster.

Docker

Needs exact version of nvidia driver

This tutorial will show you how to create Singularity images that are portable across different machines
