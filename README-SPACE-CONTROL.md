# Space-Control

This document lists down the steps to reproduce each Space-Control experiment from the paper.
The paper can be found [here](https://arch.cs.ucdavis.edu/security/memory/cxl/2026/03/06/space-control.html).

## Creating the Disk Image

This repository uses a modified version of GAPBS that allows graphs to be shared across multiple gem5 hosts.
```sh
git clone git@github.com:kaustav-goswami/gem5-resources
cd gem5-resources
git checkout disaggregated
cd src
cd shared-gapbs
./build-x86.sh 24.04
# artifacts will be created in the disk-images folder
```

## Compiling the kernel

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

## Generating Baseline

This simulates 8 systems running GAPBS and sharing a graph.
Host 0 allocates the graph and initializes all the sharing metadata.
The rest of sixe systems are running a GAPBS kernel (bfs, bc, cc, cc\_sv, pr, tc).
Host 7 sits idle.

To generate the baseline CXL results:
```sh
python3 disaggregated_memory/unified_run_space_control.py --count=8 --exp-name=no-permissions-8sys-1s --joblist=disaggregated_memory/joblist/space-control/base-v2.json
```
This generates results for a cluster system that has no permission checks, resembling a typical CXL 3.0 system.

### Space-Control results

#### Without caches

To generate the best and worst-case results for Space-Control:
```sh
# has exactly one permission entry across the entire memory region
python3 disaggregated_memory/unified_run_space_control.py \
        --count=8 \
        --exp-name=space-control-1e-8sys-1s \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-1e.json
# has exactly maximum number of permission entries across the entire memory region (size / 4 KiB)
python3 disaggregated_memory/unified_run_space_control.py \
        --count=8 \
        --exp-name=space-control-wc-8sys-1s \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-wc.json
```

The rest of the results combines different benchmarks.
Here are all the best-base results for all these combinations.
```sh
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
```

For 2 system simulation:
```sh
# To simulate 2 systems with bfs and bc
python3 disaggregated_memory/unified_run_space_control.py \
        --count=2 \
        --exp-name=space-control-gapbs-2sysA-2s-1e \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-2-A-1e.json
# To simulate 2 systems with pr and tc
python3 disaggregated_memory/unified_run_space_control.py \
        --count=2 \
        --exp-name=space-control-gapbs-2sysB-2s-1e \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-2-B-1e.json
```

For 4 system simulation:
```sh
To simulate 4 systems with bfs, bc, pr and tc
python3 disaggregated_memory/unified_run_space_control.py \
        --count=4 \
        --exp-name=space-control-gapbs-4sysA-4s-1e \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-4-A-1e.json
```

Here are all the worst-case results for all these combinations.
```sh
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
```

For 2 system simulation:
```sh
# To simulate 2 systems with bfs and bc
python3 disaggregated_memory/unified_run_space_control.py \
        --count=2 \
        --exp-name=space-control-gapbs-2sysA-2s-wc \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-2-A.json
# To simulate 2 systems with pr and tc
python3 disaggregated_memory/unified_run_space_control.py \
        --count=2 \
        --exp-name=space-control-gapbs-2sysB-2s-wc \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-2-B.json
```

For 4 system simulation:
```sh
To simulate 4 systems with bfs, bc, pr and tc
python3 disaggregated_memory/unified_run_space_control.py \
        --count=4 \
        --exp-name=space-control-gapbs-4sysA-4s-wc \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-4-A-wc.json
```
#### With caches

We only do permission caching for the worst-case number of permission entries.

```sh
# Vary the size of the permission cache
python3 disaggregated_memory/unified_run_space_control.py \
        --count=8 \
        --exp-name=space-control-gapbs-8sys-1s-cache-32-wc \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-cache-32.json
python3 disaggregated_memory/unified_run_space_control.py \
        --count=8 \
        --exp-name=space-control-gapbs-8sys-1s-cache-64-wc \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-cache-64.json
python3 disaggregated_memory/unified_run_space_control.py \
        --count=8 \
        --exp-name=space-control-gapbs-8sys-1s-cache-128-wc \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-cache-128.json
python3 disaggregated_memory/unified_run_space_control.py \
        --count=8 \
        --exp-name=space-control-gapbs-8sys-1s-cache-256-wc \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-cache-256.json
python3 disaggregated_memory/unified_run_space_control.py \
        --count=8 \
        --exp-name=space-control-gapbs-8sys-1s-cache-512-wc \
        --joblist=disaggregated_memory/joblist/space-control/space-control-gapbs-cache-512.json
```

### Comparison results

```sh
# modified to accommodate 32 GiB of memory across 8 hosts. Requires two permission lookup (see the paper).
python3 disaggregated_memory/unified_run_space_control.py \
        --count=8 \
        --exp-name=deact-8sys-1s \
        --joblist=disaggregated_memory/joblist/space-control/deact-gapbs.json
# modified to accommodate 32 GiB of memory.
python3 disaggregated_memory/unified_run_space_control.py \
        --count=8 \
        --exp-name=flat-tables-8sys-1s \
        --joblist=disaggregated_memory/joblist/space-control/flat-tables-gapbs.json
# has exactly two permission entries (for local and remote).
python3 disaggregated_memory/unified_run_space_control.py \
        --count=8 \
        --exp-name=mondrian-8sys-1s \
        --joblist=disaggregated_memory/joblist/space-control/mondrian-gapbs.json
```

## Contact

If you have any questions or find any bugs/issues, please email me at [kggoswami@ucdavis.edu](mailto:kggoswami@ucdavis.edu).

