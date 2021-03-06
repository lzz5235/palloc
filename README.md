PALLOC is a kernel-level memory allocator that exploits page-based virtual-to-physical memory translation to selectively allocate memory pages of each application to the desired DRAM banks. The goal of PALLOC is to control applications' memory locations in a way to minimize memory performance unpredictability in multicore systems by eliminating bank sharing among applications executing in parallel. PALLOC is a software based solution, which is fully compatible with existing COTS hardware platforms and transparent to applications (i.e., no need to modify application code.)

Source code
============
The source code is available in the following [location:](https://github.com/heechul/linux)

    repository: git@github.com:heechul/linux.git
    branch: palloc-3.6

To build the kernel with PALLOC enabled, the following option must be enabled.

    CONFIG_CGROUP_PALLOC=y

Detecting DRAM mappings
======================
See https://github.com/heechul/misc/blob/devel/README-map-detector.md

Usage
=====

    # mount -t cgroup xxx /sys/fs/cgroup

    < DRAM map setting for Non-XOR map systems>
    # echo 0x00183000 > /sys/kernel/debug/palloc/palloc_mask
      --> bank bits: 12, 13, 19, 20

    < DRAM map setting for XOR map systems > 
    # echo 0x0001e000 > /sys/kernel/debug/palloc/palloc_mask
      --> select bit 13, 14, 15, 16
    # echo xor 13 17 > /sys/kernel/debug/palloc/control
    # echo xor 14 18 > /sys/kernel/debug/palloc/control
    # echo xor 15 19 > /sys/kernel/debug/palloc/control
    # echo xor 16 20 > /sys/kernel/debug/palloc/control
    # echo 1 > /sys/kernel/debug/palloc/use_mc_xor
      -->  bank bits: (13 XOR 17), (14 XOR 18), (15 XOR 19), and (16 XOR 20).


    < General >
    # echo 1 > /sys/kernel/debug/palloc/use_palloc
      --> enable palloc (owise the default buddy allocator will be used)

    # mkdir /sys/fs/cgroup/part1
    # echo 0 > /sys/fs/cgroup/part1/cpuset.cpus
    # echo 0 > /sys/fs/cgroup/part1/cpuset.mems
    # echo 0-3 > /sys/fs/cgroup/part1/palloc.bins
      --> DRAM bank 0,1,2,3 are assigned to part1 CGROUP.
    # echo $$ > /sys/fs/cgroup/part1/tasks
      --> from now on, all processes invoked from the shell use pages from bank 0,1,2,3 only.

Papers
============
* Heechul Yun, Renato, Zheng-Pei Wu, Rodolfo Pellizzoni. "PALLOC: DRAM Bank-Aware Memory Allocator for Performance Isolation on Multicore Platforms," _IEEE Intl. Conference on Real-Time and Embedded Technology and Applications Symposium (RTAS)_, 2014. (to appear) ([pdf](http://www.ittc.ku.edu/~heechul/papers/palloc-rtas2014.pdf))
