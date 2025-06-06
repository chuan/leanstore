# LeanStore

[LeanStore](https://db.in.tum.de/~leis/papers/leanstore.pdf) is a high-performance OLTP storage engine optimized for many-core CPUs and NVMe SSDs. Our goal is to achieve performance comparable to in-memory systems when the data set fits into RAM while being able to fully exploit the bandwidth of fast NVMe SSDs for large data sets. While LeanStore is currently a research prototype, we hope to make it usable in production in the future.

## Compiling

Install dependencies:

`sudo apt-get install cmake libtbb2-dev libaio-dev libsnappy-dev zlib1g-dev libbz2-dev liblz4-dev libzstd-dev librocksdb-dev liblmdb-dev libwiredtiger-dev liburing-dev`

`mkdir build && cd build && cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo .. && make -j`

## Benchmark Examples
### TPC-C

`build/frontend/tpcc --tpcc_warehouse_count=100 --notpcc_warehouse_affinity --ssd_path=./ssd_block_device_or_file --worker_threads=120 --pp_threads=4 --dram_gib=240 --csv_path=./log --free_pct=1 --contention_split --xmerge --print_tx_console --run_for_seconds=60 --isolation_level=si`

Check `build/frontend/tpcc --help` for other options.

### YCSB
`build/frontend/ycsb --ycsb_read_ratio=50 --target_gib=10 --ssd_path=./ssd_block_device_or_file --worker_threads=120 --pp_threads=4 --dram_gib=5 --csv_path=./log --free_pct=1 --contention_split --xmerge --print_tx_console --run_for_seconds=60 --isolation_level=si`
check `build/frontend/ycsb --help` for other options

## Implement Your Workload

LeanStore offers a flexible transactional Key/Value interface similar to WiredTiger and RocksDB.
A table is a B-Tree index where keys and values are stored in a normalized format, i.e., lexicographically ordered strings.
For convenience, frontend/shared offers templates that take care of (un)folding common types.
The best starting points are frontend/minimal-example and frontend/ycsb.
The required parameters at runtime are: `--ssd_path=/block_device/or/filesystem --dram_gib=fixed_in_gib`.
The default transaction isolation level is `--isolation_level=si`. You can lower it to Read Committed or Read Uncommitted by replacing `si` with `rc` or `ru`, respectively.
You can set the transaction isolation level using `--isolation_level=si` and enable the B-Tree techniques from CIDR202 with `--contention_split --xmerge`.

### Metrics Reporting
LeanStore emits several metrics per second in CSV files: `log_bm.csv, log_configs.csv, log_cpu.csv, log_cr.csv, log_dt.csv`.
Each row has a c_hash value, which is calculated by chaining and hashing all the configurations that you passed to the binary at runtime.
This gives you an easy way to identify your run and join all relevant information from the different CSV files using SQLite, for example."

## Implemented Features

- [x] Lightweight buffer manager with pointer swizzling [ICDE'18]
- [x] Optimstic Lock Coupling with Hybrid Page Guard to synchronize paged data structures [IEEE'19]
- [x] Contention and Space Management in B-Trees [CIDR'21]
- [x] Variable-length key/values B-Tree with prefix compression and hints  [BTW'23]
- [x] Scalable and robust out-of-memory Snapshot Isolation (OSIC protocol, Graveyard, and FatTuple) [VLDB'23]
- [x] Distributed Logging with remote flush avoidance [SIGMOD'20, BTW'23]
- [ ] Recovery [SIGMOD'20]
- [ ] What Modern NVMe Storage Can Do, And How To Exploit It: High-Performance {I/O} for High-Performance Storage Engines [VLDB'23] [branch](https://github.com/leanstore/leanstore/tree/io)
- [ ] Why Files If You Have a DBMS? [ICDE'24] [branch](https://github.com/leanstore/leanstore/tree/blob)
- [ ] Moving on From Group Commit: Autonomous Commit Enables High Throughput and Low Latency on NVMe SSDs [SIGMOD'25] [branch](https://github.com/leanstore/leanstore/tree/latency)

## Cite

LeanStore was originally implemented using Pointer Swizzling ([paper](https://15721.courses.cs.cmu.edu/spring2020/papers/23-largethanmemory/leis-icde2018.pdf)).
More recently, LeanStore was also completely rewritten from scratch, replacing Pointer Swizzling with Virtual-Memory Assisted Buffer Pool ([Paper](https://www.cs.cit.tum.de/fileadmin/w00cfj/dis/_my_direct_uploads/vmcache.pdf)).
You can find the citations for different versions (which include different features) in the following.

### VMCache version

[Branch `blob`](https://github.com/leanstore/leanstore/tree/blob) covers the high-performance variable-sized objects, which we used for our ICDE 2024 paper.

```BibTeX
@inproceedings{DBLP:conf/icde/NguyenL24,
  author       = {Lam{-}Duy Nguyen and
                  Viktor Leis},
  title        = {Why Files If You Have a DBMS?},
  booktitle    = {{ICDE}},
  pages        = {3878--3892},
  publisher    = {{IEEE}},
  year         = {2024}
}
```

[Branch `latency`](https://github.com/leanstore/leanstore/tree/latency) covers the autonomous commit technique that delivers high-throughput, low-latency commits.

```BibTeX
@inproceedings{DBLP:journals/pacmmod/NguyenL25,
  author       = {Lam{-}Duy Nguyen and
                  Adnan Alhomssi and
                  Tobias Ziegler and
                  Viktor Leis},
  title        = {Moving on From Group Commit: Autonomous Commit Enables High Throughput and Low Latency on NVMe SSDs},
  journal      = {Proc. {ACM} Manag. Data},
  volume       = {3},
  year         = {2025}
}
```

### Pointer Swizzling version

The code we used for our VLDB 2023 that covers alternative SI commit protocols is in a different [branch](https://github.com/leanstore/leanstore/tree/mvcc).

```BibTeX
@inproceedings{alhomssi23,
    author    = {Adnan Alhomssi and Viktor Leis},
    title     = {Scalable and Robust Snapshot Isolation for High-Performance Storage Engines},
    booktitle = {VLDB},
    year      = {2023}
}
```

The code we used for our VLDB 2023 that covers the fast I/O implementation is in a different [branch](https://github.com/leanstore/leanstore/tree/io).

```BibTeX
@article{haas23,
  author       = {Gabriel Haas and Viktor Leis},
  title        = {What Modern NVMe Storage Can Do, And How To Exploit It: High-Performance {I/O} for High-Performance Storage Engines},
  journal      = {Proc. {VLDB} Endow.},
  year         = {2023}
}
```

BTW 2023 [branch](https://github.com/leanstore/leanstore/tree/btw) that covers alternative dependency tracking.

```BibTeX
@inproceedings{leanstore23,
    author    = {Adnan Alhomssi, Michael Haubenschild and Viktor Leis},
    title     = {The Evolution of LeanStore},
    booktitle = {BTW},
    year      = {2023}
}
```

CIDR 2021 [branch](https://github.com/leanstore/leanstore/tree/cidr) (outdated).

```BibTeX
@inproceedings{alhomssi21,
    author    = {Adnan Alhomssi and Viktor Leis},
    title     = {Contention and Space Management in B-Trees},
    booktitle = {CIDR},
    year      = {2021}
}
```
