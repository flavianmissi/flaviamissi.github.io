---
layout: post
title:  "The proc filesystem"
date:   2013-7-15
categories: linux processes
---

## Intro

If you use tools such as *ps* and *top* then you are already using the proc filesystem even though you never actually ran an *ls* or opened a file belonging to it. The reason for that is that these tools make use of this filesystem to collect information about processes, and this what this filesystem is for – *to store informations about processes*.

But what exactly is the proc filesystem?

The proc filesystem is actually a pseudo filesystem used as an interface to access kernel data structures. It’s mostly informative and read-only, but you can actually configure some stuff there.

## What kind of informations does /proc stores?

Lets take a look at the filesystem structure to understand what exactly it stores. The following is the result of a *ls* inside /proc

{% highlight bash %}
>$ ls -l
dr-xr-xr-x  9 root    root    1
dr-xr-xr-x  9 root    root    10
# ... omitted output
dr-xr-xr-x  9 root    root    9910
dr-xr-xr-x  2 root    root    acpi
dr-xr-xr-x  4 root    root    asound
-r--r--r--  1 root    root    buddyinfo
dr-xr-xr-x  4 root    root    bus
-r--r--r--  1 root    root    cgroups
-r--r--r--  1 root    root    cmdline
-r--r--r--  1 root    root    consoles
-r--r--r--  1 root    root    cpuinfo
-r--r--r--  1 root    root    crypto
-r--r--r--  1 root    root    devices
-r--r--r--  1 root    root    diskstats
-r--r--r--  1 root    root    dma
dr-xr-xr-x  2 root    root    driver
-r--r--r--  1 root    root    execdomains
-r--r--r--  1 root    root    fb
-r--r--r--  1 root    root    filesystems
dr-xr-xr-x  8 root    root    fs
-r--r--r--  1 root    root    interrupts
-r--r--r--  1 root    root    iomem
{% endhighlight %}

Let’s make sense of it: the numbers are directories named by its processes IDs, these directories contains informations of the process it refers, such as the command the process is executing, the command line of the process, the process environment variables, memory mapping information such as libraries that are being used and much more.

It’s worth to note that some of these files’ contents may be null-separated, you can use *cat* with *tr* to replace them, e.g.

{% highlight bash %}
$ cat 1/environ | tr "\000" "\n"
{% endhighlight %}

Now lets run a *ls* on the */proc/1* directory, this *pid* always refers to the *init* process:

{% highlight bash %}
>$ ls -ltr
-r--r--r--  1 root    root   cmdline
-r--r--r--  1 root    root   status
-r--r--r--  1 root    root   stat
lrwxrwxrwx  1 root    root   exe -> /sbin/init
-r--r--r--  1 root    root   limits
lrwxrwxrwx  1 root    root   root -> /
-r--r--r--  1 root    root   wchan
dr-xr-xr-x  3 root    root   task
-r--r--r--  1 root    root   syscall
-r--r--r--  1 root    root   statm
-r--r--r--  1 root    root   stack
-r--r--r--  1 root    root   smaps
-r--r--r--  1 root    root   sessionid
-r--r--r--  1 root    root   schedstat
-rw-r--r--  1 root    root   sched
-r--r--r--  1 root    root   personality
-r--r--r--  1 root    root   pagemap
-rw-r--r--  1 root    root   oom_score_adj
-r--r--r--  1 root    root   oom_score
-rw-r--r--  1 root    root   oom_adj
-r--r--r--  1 root    root   numa_maps
dr-x--x--x  2 root    root   ns
dr-xr-xr-x  5 root    root   net
-r--------  1 root    root   mountstats
-r--r--r--  1 root    root   mounts
-r--r--r--  1 root    root   mountinfo
-rw-------  1 root    root   mem
-r--r--r--  1 root    root   maps
dr-x------  2 root    root   map_files
-rw-r--r--  1 root    root   loginuid
-r--r--r--  1 root    root   latency
-r--------  1 root    root   io
dr-x------  2 root    root   fdinfo
dr-x------  2 root    root   fd
-r--------  1 root    root   environ
lrwxrwxrwx  1 root    root   cwd -> /
-r--r--r--  1 root    root   cpuset
-rw-r--r--  1 root    root   coredump_filter
-rw-r--r--  1 root    root   comm
--w-------  1 root    root   clear_refs
-r--r--r--  1 root    root   cgroup
-r--------  1 root    root   auxv
-rw-r--r--  1 root    root   autogroup
dr-xr-xr-x  2 root    root   attr
{% endhighlight %}

I’ll cover only the most important files, some of their content you’ll find in process management commands output as I said before, such as *ps*, others you’ll only find if you come into this directory.

## /proc/[pid]/task/

This directory contains all threads in the process, one subdirectory per thread. They are named with the id of the thread (*tid*). Within this subdirectory there is basically the same structure as the one in */proc/[pid]*, for shared attributes the file contents are the same, for distinct attributes the corresponding files may have different values (e.g. */proc/[id]/[tid]/status*)

## /proc/[pid]/status

Provides same information as the */proc/[pid]/stat* and */proc/[pid]/statm* formated for humans.

This file gives informations about the process (*/proc/[pid]/stat*) and it’s used by the ps command and also provides information about memory usage (*/proc/[pid]/statm*)

For information about the columns and fields see the proc manual page.

## /proc/[pid]/root

This file is a symbolic link that points to the process’s root directory. Its existence makes container virtualization techniques possible, tools such as *chroot* make use of it. See the *chroot(2)* manual for more information.

## /proc/[pid]/ns/

Subdirectory containing one entry for each namespace that supports being manipulated by setns, if you’re curious and enjoy some black magic, take a look at the manuals of clone and setns.

## /proc/[pid]/coredump_filter

Through this file you can control which memory segments are written to the core dump file when one is performed for the corresponding process. For more information see *core(5)* manual page.

## /proc/[pid]/cmdline

This file holds the complete command line for the process, unless its a zombie, in the case of walkers, this file will be empty.

## /proc/[pid]/cwd

Symbolic link to the current working directory of the process. For instance, if you want to find the current working process for a process, run:

{% highlight bash %}
>$ cd /proc/20/cwd; /bin/pwd
{% endhighlight %}

## /proc/[pid]/environ

This file contains the environment variables for the process, null-separated.

## /proc/[pid]/exe

This file is a symbolic link containing the pathname of the executed command.

These are some of the files that I find important or just curious under */proc/[pid]/* and I might have forgotten some of them, if you think I did, don’t hesitate to tell me so!

Also, as you might have noticed I simply didn’t addressed the files right under */proc*. That’s because I see the information they carry as more important than the former – this is because of my programming background and day-to-day issues. That’s why I am leaving the job to cover those with the manuals (which, BTW, covers the topic very well). use *$ man proc* to get a complete explanation on what information each file can give you and *$ man /proc/filename* for more information about a specific file.
