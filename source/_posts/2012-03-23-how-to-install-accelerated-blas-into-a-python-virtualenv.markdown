---
comments: true
date: 2012-03-23 20:43:33
layout: post
slug: how-to-install-accelerated-blas-into-a-python-virtualenv
title: How to install accelerated BLAS into a Python virtualenv
wordpress_id: 426
categories:
- Web Development
tags:
- python virtualenv blas ubuntu
---

## Background #

Some mathematically intense operations that use Numpy/Scipy can run faster with accelerated basic linear algebra subroutine (BLAS) libraries installed on your system (e.g., [gensim's](http://radimrehurek.com/gensim/) corpus processing).

To see what BLAS libraries you are using, do:

{% codeblock lang:python %}python -c 'import numpy; numpy.show_config()'{% endcodeblock %}

If none of them are installed, you probably want to install one or
more. [ATLAS](http://math-atlas.sourceforge.net/) is always a good bet, since it's portable and
self-optimizing. There are others out there targeted at particular CPU architectures.

Unfortunately, the [Scipy docs](http://docs.scipy.org/doc/numpy/user/install.html) are out of date regarding installing accelerated BLAS libraries on Ubuntu. The instructions I have written below work for Ubuntu 10.04, the current LTS (long-term support) version, and though I haven't tried to run them on a more recent version, it's possible they work with those as well.

## Prereqs #

On Ubuntu 10.04, and possibly other versions, you need liblapack-dev and gfortran (yes, fortran):

{% codeblock lang:bash %}
$ sudo apt-get install liblapack-dev
$ sudo apt-get install gfortran
{% endcodeblock %}

## Instructions #

Install the accelerated linear algebra libraries (ATLAS/LAPACK) in your virtualenv on Ubutu:

{% codeblock lang:bash %}
#!/bin/bash
workon [envname]
pip uninstall numpy ## only if numpy is already installed
pip uninstall scipy ## only if scipy is already installed
export LAPACK=/usr/lib/liblapack.so
export ATLAS=/usr/lib/libatlas.so
export BLAS=/usr/lib/libblas.so
{% endcodeblock %}

Now you can install numpy and scipy into the same virtualenv and be confident they will perform operations using the accelerated BLAS routines:

{% codeblock lang:bash %}
$ pip install numpy
$ pip install scipy
{% endcodeblock %}
