---
layout: post
title:  "Building and installing lxml with PyPy"
date:   2013-9-19
categories: pypy python
---
## Introduction
The major issue my colleagues and I found when we started running some projects with [PyPy](http://pypy.org/) was the
[lxml](https://pypi.python.org/pypi/lxml/3.2.3) library. It uses [Cython](http://docs.cython.org/src/quickstart/overview.html),
which can run with PyPy if you write your code [portably enough](http://docs.cython.org/src/userguide/pypy.html).
So an effort began to port lxml to use [CFFI](https://cffi.readthedocs.org/en/release-0.7/). This effort can be found
on [this fork](https://github.com/amauryfa/lxml) and this is the code we’re going to install from.

## Resolving dependencies

We are going to install lxml on a ubuntu 13.04, be warned that installation in OSX might give you serious headaches (<10.8). Start by running the following apt-get:

{% highlight bash %}
$ sudo apt-get install libxml2 libxslt1-dev zlib1g-dev
{% endhighlight %}

These packages are needed to build lxml (with Python or PyPy).

## Bootstraping your environment

You’ll need to have PyPy’s binary to build lxml with, you can folow Andrews Medina’s guide to install it (but it’s in portuguese…)

Assuming you have it installed let’s create a virtual environment (with virtualenv and virtualenvwrapper) to install lxml in:

{% highlight bash %}
$ mkvirtualenv lxml-pypy -p /path/to/pypy/bin/pypy
{% endhighlight %}

Now clone the lxml fork and checkout to the CFFI branch:

{% highlight bash %}
$ git clone https://github.com/amauryfa/lxml.git
$ cd lxml
$ git checkout origin/cffi
{% endhighlight %}

Now build and install (double check if you’re in the right virtual environment):

{% highlight bash %}
$ python setup.py build
$ python setup.py install
{% endhighlight %}

Done!
