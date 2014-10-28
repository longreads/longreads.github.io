---
layout: post
title: Sorting bams with samtools
excerpt: "In which samtools sort speed is discussed"
tags: [intro, samtools]
comments: true
---

It turns out that `samtools sort` command line options really do matter. When sorting a large bam file, samtools creates lots of intermediate files. The merge it runs to create the final sorted bam tries to read from all of them nearly simultaneously, thrashing even a pretty decent RAID drive (the read head on the drives are spending more time moving between files than actually reading any single file).

A partial solution lies in increasing the amount of memory used by specifying the `-m` option, for example `-m 5G` to give each samtools thread 5GB of memory. The larger the amount of memory, the fewer intermediate files created, the less thrashing involved in the merge step.

In my case, time to sort a 30x whole-genome BAM file went from days down to hours simply by specifying `-m 10G` rather than the default of 768M.

A more complete solution might be to use the [DNAnexus fork](https://github.com/dnanexus/samtools) of samtools, but it didn't compile out of the box on OS X so I haven't actually tried it (though the accompanying [blog post](http://devblog.dnanexus.com/faster-bam-sorting-with-samtools-and-rocksdb/) was helpful).

Also, remember that newer versions of samtools sort can be parallelized to some extent using the `-@` option, eg `-@4` to specify 4 threads. Note that the memory allocation above is per thread, so in our example, we'd need about 4 x 5 = 20 GB of RAM.