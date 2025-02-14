---
tags:
  - Distributed-Systems
---

# Overview

| Operation                          | Latency (ns) | Latency (µs/ms)     | Comparison                               |
| ---------------------------------- | ------------ | ------------------- | ---------------------------------------- |
| L1 cache reference                 | 0.5          |                     |                                          |
| Branch mispredict                  | 5            |                     |                                          |
| L2 cache reference                 | 7            |                     | 14x L1 cache                             |
| Mutex lock/unlock                  | 25           |                     |                                          |
| Main memory reference              | 100          |                     | 14x L2 cache, 200x L1 cache              |
| Compress 1K bytes with Snappy      | 3,000        | 3 µs                |                                          |
| Read 1 MB sequentially from memory | 20,000       | 20 µs               | ~50GB/sec DDR5                           |
| Read 1 MB sequentially from NVMe   | 100,000      | 100 µs              | ~10GB/sec NVMe, 5x memory                |
| Round trip within same datacenter  | 500,000      | 500 µs              |                                          |
| Read 1 MB sequentially from SSD    | 2,000,000    | 2,000 µs (2 ms)     | ~0.5GB/sec SSD, 100x memory, 20x NVMe    |
| Read 1 MB sequentially from HDD    | 6,000,000    | 6,000 µs (6 ms)     | ~150MB/sec 300x memory, 60x NVMe, 3x SSD |
| Send 1 MB over 1 Gbps network      | 10,000,000   | 10,000 µs (10 ms)   |                                          |
| Disk seek                          | 10,000,000   | 10,000 µs (10 ms)   | 20x datacenter roundtrip                 |
| Send packet CA->Netherlands->CA    | 150,000,000  | 150,000 µs (150 ms) |                                          |

# References

- [Numbers everyone (programmer) should know (2023) · GitHub](https://gist.github.com/BlackHC/2d0a3a21542b524a7cf2f8eac977481e)
