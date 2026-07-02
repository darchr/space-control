# Space-Control

This document lists down the steps to reproduce each Space-Control experiment from the paper.
The paper can be found [here](https://arch.cs.ucdavis.edu/security/memory/cxl/2026/03/06/space-control.html).

## Roadmap

[ ] Create python scripts to automatically generate csv with the results.
[ ] Clean the source files.

## Creating resources

### Disk Image

This repository uses a modified version of GAPBS that allows graphs to be shared across multiple gem5 hosts.
```sh
git clone git@github.com:kaustav-goswami/gem5-resources
cd gem5-resources
git checkout disaggregated
cd src
cd shared-gapbs
./build-x86.sh 22.04
# artifacts will be created in the disk-images folder
```

### Kernel

The specific kernel used in Space-Control

```sh
git clone git@github.com:kaustav-goswami/gem5-resources
cd gem5-resources
cd src/kernels
# see README.md on how to build the kernel
git clone https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git
cd linux
git checkout v6.9.9 # maybe
cp ../linux-configs/config.x86.6.9.9 .config
make -j`nproc`
```

Make sure to replace the local paths to these resources in the joblist JSONs from `disaggregated_memory/joblist/space-control/*.json`.
```json
..
            "disk": "<path/to/gem5-resources/src/shared-gapbs/x86-disk-image-24-04/x86-ubuntu>",
            "kernel": "<path/to/gem5-resources/src/kernels/linux/vmlinux>",
..
```

## Reproducing the results

### Setup information

The codebase is based on [CXL-ClusterSim](https://arch.cs.ucdavis.edu/simulation/memory/cxl/2026/05/26/cxl-clustersim.html).
It allows multiple gem5 hosts to share memory in SST.

The setup simulates up to 8 systems running GAPBS and sharing a graph.
The exact implemention of this version of GAPBS can be found [here](https://github.com/darchr/shared-gapbs). 

In most cases, host 0 allocates the graph and initializes all the sharing metadata.
The rest of sixe systems are running a GAPBS kernel (bfs, bc, cc, cc\_sv, pr, tc).
Host 7 sits idle.

When simulating less than 8 systems, we randomly pair up different workloads to make sure we capture scalability effects.
Here, the allocator is run on host0 (always) but is followed up by a graph kernel.
SST only loads the data in the shared memory region from host0 only.

The structure of GAPBS prevent weighted graphs to be shared with unweighted ones.
That is the reason SSSP is missing from our evaluation.
Several hosts running SSSP on the same graph can still be shown using the same infrastructure.

### Generating baseline results

To generate the baseline CXL results:
```sh
# This generates the baseline numbers.
python3 disaggregated_memory/unified_run_space_control.py \
        --count=8 \
        --exp-name=no-permissions-8sys-1s \
        --joblist=disaggregated_memory/joblist/space-control/base-v2.json
```
This generates results for a cluster system that has no permission checks, resembling a typical CXL 3.0 system.

### Space-Control results

A separate section is created to generate experimental results from a given subsection of the paper.

#### Baseline and scalability

This section sets up the baseline case (CXL) and then shows scalability of Space-Control.
Figure 7(a) in the paper shows the CPI of 4 graph kernels: pr, bfs, bc and tc scaled from 1 system to 8 systems.
There are at most one permission entry (1e) in Figure 7.
Note that the allocator in these experiments (< 8 hosts) is executed on host0 before running the graph kernel.

The baseline (CXL) numbers are taken from [the baseline section](#generating-baseline-results).
Space-Control results with a single entry can be generated via:
```sh
# ----------------------------------------------------------------------------
# 1sys
# ----------------------------------------------------------------------------
#
# Note that the paper presents pr, bfs, bc and tc. These results are generated as bfs, bc, pr and tc.
# To simulate 1 system bfs
python3 disaggregated_memory/unified_run_space_control.py \
        --count=1 \
        --exp-name=space-control-gapbs-1sysA-1s-1e \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-1-A-1e.json
# To simulate 1 system bc
python3 disaggregated_memory/unified_run_space_control.py \
        --count=1 \
        --exp-name=space-control-gapbs-1sysB-1s-1e \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-1-B-1e.json
# To simulate 1 system pr
python3 disaggregated_memory/unified_run_space_control.py \
        --count=1 \
        --exp-name=space-control-gapbs-1sysC-1s-1e \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-1-C-1e.json
# To simulate 1 system tc
python3 disaggregated_memory/unified_run_space_control.py \
        --count=1 \
        --exp-name=space-control-gapbs-1sysD-1s-1e \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-1-D-1e.json
# ----------------------------------------------------------------------------
# 2sys
# ----------------------------------------------------------------------------
# 
# To simulate 2 systems running bfs and bc
python3 disaggregated_memory/unified_run_space_control.py \
        --count=2 \
        --exp-name=space-control-gapbs-2sysA-1s-1e \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-2-A-1e.json
# To simulate 2 systems running pr and tc
python3 disaggregated_memory/unified_run_space_control.py \
        --count=2 \
        --exp-name=space-control-gapbs-2sysB-1s-1e \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-2-B-1e.json
# ----------------------------------------------------------------------------
# 4sys
# ----------------------------------------------------------------------------
# 
# To simulate 2 systems running bfs and bc
python3 disaggregated_memory/unified_run_space_control.py \
        --count=4 \
        --exp-name=space-control-gapbs-4sysA-1s-1e \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-4-1e.json
```

Next we show the CPI comparison in Figure 7 (b) across all the graph kernels.
This is generated by running the following command:
```sh
# has exactly one permission entry across the entire memory region
python3 disaggregated_memory/unified_run_space_control.py \
        --count=8 \
        --exp-name=space-control-gapbs-8sys-1s-1e \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-1e.json
```

#### Sensitivity to Permission Table Fragmentation

We repeat the experiments from the previous section to understand the effects of permission table fragmentation.
We call this the worst-case (wc) Space-Control results.
To generate Figure 8(a), follow the following commands:
```sh
# ----------------------------------------------------------------------------
# 1sys
# ----------------------------------------------------------------------------
#
# Note that the paper presents pr, bfs, bc and tc. These results are generated as bfs, bc, pr and tc.
# To simulate 1 system bfs
python3 disaggregated_memory/unified_run_space_control.py \
        --count=1 \
        --exp-name=space-control-gapbs-1sysA-1s-wc \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-1-A.json
# To simulate 1 system bc
python3 disaggregated_memory/unified_run_space_control.py \
        --count=1 \
        --exp-name=space-control-gapbs-1sysB-1s-wc \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-1-B.json
# To simulate 1 system pr
python3 disaggregated_memory/unified_run_space_control.py \
        --count=1 \
        --exp-name=space-control-gapbs-1sysC-1s-wc \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-1-C.json
# To simulate 1 system tc
python3 disaggregated_memory/unified_run_space_control.py \
        --count=1 \
        --exp-name=space-control-gapbs-1sysD-1s-wc \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-1-D.json
# ----------------------------------------------------------------------------
# 2sys
# ----------------------------------------------------------------------------
# 
# To simulate 2 systems running bfs and bc
python3 disaggregated_memory/unified_run_space_control.py \
        --count=2 \
        --exp-name=space-control-gapbs-2sysA-1s-wc \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-2-A.json
# To simulate 2 systems running pr and tc
python3 disaggregated_memory/unified_run_space_control.py \
        --count=2 \
        --exp-name=space-control-gapbs-2sysB-1s-wc \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-2-B.json
# ----------------------------------------------------------------------------
# 4sys
# ----------------------------------------------------------------------------
# 
# To simulate 2 systems running bfs and bc
python3 disaggregated_memory/unified_run_space_control.py \
        --count=4 \
        --exp-name=space-control-gapbs-4sysA-1s-wc \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-4.json
```

Instead of simply comparing CPI between the baseline and space-control-wc, we compare the number of permission lookups per kilo instructions between the best and the worst case.
This is shown in Figure 8(b).
To generate these results, first run the experiment:
```sh
# has exactly maximum number of permission entries across the entire memory region (size / 4 KiB)
python3 disaggregated_memory/unified_run_space_control.py \
        --count=8 \
        --exp-name=space-control-8sys-1s-wc \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-wc.json
```

gem5 automatically generates the Probability Density Function (PDF) and counts per bin of the number of permission lookups in stats.txt.
Figure 9 is simply generated from the stats file from `space-control-8sys-1s-wc/` directory (per host/kernel).
The bins are shown as a histogram.

```txt
# stat to look for
board.permission_table.binarySearchAttempts::0-3
board.permission_table.binarySearchAttempts::4-7
board.permission_table.binarySearchAttempts::8-11
board.permission_table.binarySearchAttempts::12-15
board.permission_table.binarySearchAttempts::16-19
board.permission_table.binarySearchAttempts::20-23
board.permission_table.binarySearchAttempts::24-27
board.permission_table.binarySearchAttempts::28-31
board.permission_table.binarySearchAttempts::32-35
board.permission_table.binarySearchAttempts::36-39
```

#### Memory-Access Split and Memory Bandwidth

Based on the existing experiments, the memory access-split can be quicky generated.
Figures 10(a) can be generated from `space-control-8sys-1s-1e/` and `space-control-8sys-1s-wc/`.

```txt
# stat to look for
# board.permission_table.numOutgoingPermissionPackets gives the exact number of permission packets.
# board.permission_table.numOutgoingMemSidePackets - board.permission_table.numOutgoingPermissionPackets gives the exact number of data requests.
# Each of these packets are exactly 64 Bytes (cache line) size long.
```

Figure 10(b) can be generated from `no-permissions-8sys-1s/`, `space-control-8sys-1s-1e/` and `space-control-8sys-1s-wc/`.

```txt
# stat to look for
board.remote_memory.outgoing_request_bridge.sizeOutgoingPackets
board.remote_memory.outgoing_request_bridge.sizeIncomingPackets
```

#### Performance Enforcement Latency

Based on existing experiments, the performance latencies can be generated.
We have used the worst-case experiment here `space-control-8sys-1s-wc`.
```txt
# stat to look for
board.permission_table.stallTime::mean
```
This generates Figure 11(a).

To generate Figure 11(b), we use the following:
1. Packet creation latency: setup via the SimObject (`permission_table.creation_latency = Int`)
2. Comparison and encryption latency: setup at compile time (TODO) (`Tick comparison_latency`). Since comparison and encryption can be done in parallel, we assume it to be 1 cycle (similar to SGX-MEE designs).
3. Average stall latency: during simulation (`board.permission_table.stallTime::mean` in stats.txt)

To generate the histograms in Figure 12:

```txt
# stat to look for
# histogram data from 
board.permission_table.stallTime
```

#### Performance Cache Efficacy

Finally, we add permission caches to the design.
We vary the cache size, compare the miss ration and compare the CPI of all these different systems.
To generate the miss ratio figure (Figure 13(a)):

```sh
# Vary the size of the permission cache
python3 disaggregated_memory/unified_run_space_control.py \
        --count=8 \
        --exp-name=space-control-gapbs-8sys-1s-cache-0.5KiB-wc \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-cache-8.json
python3 disaggregated_memory/unified_run_space_control.py \
        --count=8 \
        --exp-name=space-control-gapbs-8sys-1s-cache-1KiB-wc \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-cache-16.json
python3 disaggregated_memory/unified_run_space_control.py \
        --count=8 \
        --exp-name=space-control-gapbs-8sys-1s-cache-2KiB-wc \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-cache-32.json
python3 disaggregated_memory/unified_run_space_control.py \
        --count=8 \
        --exp-name=space-control-gapbs-8sys-1s-cache-4KiB-wc \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-cache-64.json
python3 disaggregated_memory/unified_run_space_control.py \
        --count=8 \
        --exp-name=space-control-gapbs-8sys-1s-cache-8KiB-wc \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-cache-128.json
python3 disaggregated_memory/unified_run_space_control.py \
        --count=8 \
        --exp-name=space-control-gapbs-8sys-1s-cache-16KiB-wc \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-cache-256.json
python3 disaggregated_memory/unified_run_space_control.py \
        --count=8 \
        --exp-name=space-control-gapbs-8sys-1s-cache-32KiB-wc \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-cache-512.json
python3 disaggregated_memory/unified_run_space_control.py \
        --count=8 \
        --exp-name=space-control-gapbs-8sys-1s-cache-64KiB-wc \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-cache-1024.json
```

Stat to look for:
```txt
# stat to look for
board.permission_table.numPermissionTableCacheMisses
# and
board.permission_table.numPermissionTableCacheAccesses
```

Figure 13 (b) (cache CPI) uses the results from `space-control-8sys-1s-wc`.

#### Comparison experiments

Each of the different prior approaches can be evaluated via:
```sh
# modified to accommodate 16 GiB of memory across 8 hosts. Requires two permission lookup (see the paper).
python3 disaggregated_memory/unified_run_space_control.py \
        --count=8 \
        --exp-name=deact-8sys-1s \
        --joblist=disaggregated_memory/joblist/space-control/deact-gapbs.json
# modified to accommodate 32 GiB of memory.
python3 disaggregated_memory/unified_run_space_control.py \
        --count=8 \
        --exp-name=flat-tables-8sys-1s \
        --joblist=disaggregated_memory/joblist/space-control/flat-tables-gapbs.json
# has lookups for both local and remote addresses
python3 disaggregated_memory/unified_run_space_control.py \
        --count=8 \
        --exp-name=mondrian-8sys-1s \
        --joblist=disaggregated_memory/joblist/space-control/mondrian-gapbs.json
```

## Contact

If you have any questions or find any bugs/issues, please email me at [kggoswami@ucdavis.edu](mailto:kggoswami@ucdavis.edu).

