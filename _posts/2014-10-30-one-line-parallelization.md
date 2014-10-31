---
layout: post
title: One-line parallelization
excerpt: "Using GNU parallel to parallelize shell commands"
tags: [intro, parallelization]
comments: true
---

Say you just sorted a whole bunch of bam files and now want to create an index for each.

The obvious (but wrong) solution would be `samtools index *.sorted.bam`. Wrong, because samtools index only takes a single input bam file path.

The next obvious (correct) solution might be:
{% highlight bash %}
for i in *.sorted.bam
do
	samtools index $i
done 
{% endhighlight %}
But who wants to type all that? And anyways, this is a post about running things in parallel, which this is not.

Enter the solution using [GNU parallel](http://www.gnu.org/software/parallel/): `ls *.sorted.bam | parallel -j4 samtools index {}`. Simple, runs 4 processes in parallel until they all finish.

Installing parallel is as easy as `brew install parallel` (on OS X with [homebrew](http://brew.sh) installed), or `sudo apt-get install parallel` (on Ubuntu et al; may need an [extra command](http://askubuntu.com/questions/12764/where-do-i-get-a-package-for-gnu-parallel) depending on your version).

A quick intro to parallel:

- inputs can be passed to parallel by piping them in, as above
- alternately, you can specify them using [this format](http://www.gnu.org/software/parallel/parallel_tutorial.html): `parallel echo {} ::: A B C`, which echos A, B, and C (each separately, in a random order)
- The `{}` will be replaced by the inputs as given. You can also lop off the file name extension using `{.}`, for example: `ls *.bam | parallel echo samtools sort {} {.}.sorted`, which issues commands of the variety `samtools sort mysample.bam mysample.sorted`.
- the `--dryrun` option will echo the commands back to you without running them. I find it easier to just type the word "echo" before my command
- by default the output from each command will only be printed to stdout after the command has completed; to intersperse output from all commands in real time, use the `-u` ("ungroup") argument to parallel
- by default, the number of processes is the number of cores available on the computer. Use `-j n` to specify `n` jobs in parallel
- to run more complex commands using parallel, you may need to use quotation marks around the command

A note of caution: monitor your CPU usage when running jobs in parallel. If your processes aren't getting up to 100% CPU usage each, this might be indicative of an file read/write bottleneck, and reducing `-j` might actually help your overall runtime. For example, running `samtools sort` in parallel on lots of files exacerbates the thrashing problem [described previously]({% post_url 2014-10-26-samtools-sort %}) because there even more files being read simultaneously.