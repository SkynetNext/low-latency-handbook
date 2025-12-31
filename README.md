# Low Latency Handbook

A comprehensive collection of hardware and software latency data for high-performance system design and optimization.

---

## ⚡ Latency Hierarchy (3GHz CPU)

This table ranks operations from lowest to highest latency. Cycle counts are based on a **3GHz CPU** (1 cycle ≈ 0.333ns).

| Operation / Component | Typical Latency | Cycles | Category | Detailed Docs |
|:----------------------|:----------------|:-------|:---------|:--------------|
| **Register Operation** | < 0.33ns | < 1 | CPU | [CPU & Memory](docs/cpu_memory.md) |
| **L1 Cache Hit** | ~1ns | ~3 | Cache | [CPU & Memory](docs/cpu_memory.md) |
| **L2 Cache Hit** | ~4ns | ~12 | Cache | [CPU & Memory](docs/cpu_memory.md) |
| **Atomic (Uncontended)** | **~12ns** | **~36** | Sync | [CPU & Memory](docs/cpu_memory.md) |
| **L3 Cache Hit** | ~12.7ns | ~38 | Cache | [CPU & Memory](docs/cpu_memory.md) |
| **Mutex (Uncontended)** | **~25ns** | **~75** | Sync | [CPU & Memory](docs/cpu_memory.md) |
| **Disruptor (Min)** | **29ns** | **~87** | Software | [Frameworks](docs/frameworks.md) |
| **Main Memory Access** | ~65ns | ~195 | Memory | [CPU & Memory](docs/cpu_memory.md) |
| **Aeron IPC** | **0.25μs** | **~750** | Software | [Frameworks](docs/frameworks.md) |
| **RDMA (High-end NIC)** | < 1μs | < 3K | Network | [Networking](docs/networking.md) |
| **Context Switch** | ~0.67μs | ~2K | System | [CPU & Memory](docs/cpu_memory.md) |
| **Mutex (Contended)** | **> 0.33μs** | **> 1K** | Sync | [CPU & Memory](docs/cpu_memory.md) |
| **C++ Exception** | **1.7-3.3μs** | **5K-10K** | Software | [CPU & Memory](docs/cpu_memory.md) |
| **UDP (Loopback)** | 5-10μs | 15K-30K | Network | [Networking](docs/networking.md) |
| **NVMe SSD Read** | **~20μs** | **~60K** | Storage | [Storage](docs/storage.md) |
| **TCP (Loopback)** | 10-20μs | 30K-60K | Network | [Networking](docs/networking.md) |
| **TCP/IP (Same DC)** | 100-200μs | 300K-600K | Network | [Networking](docs/networking.md) |
| **SATA SSD Read** | 0.1-0.5ms | 300K-1.5M | Storage | [Storage](docs/storage.md) |
| **HDD Random Read** | **8-15ms** | **24M-45M** | Storage | [Storage](docs/storage.md) |

---

## 📖 Deep Dives

- [**CPU & Memory**](docs/cpu_memory.md): Registers, instruction costs, cache hierarchy, NUMA.
- [**Networking**](docs/networking.md): TCP/UDP, RDMA, eBPF, Cross-AZ latency.
- [**Storage**](docs/storage.md): HDD, SATA SSD, NVMe SSD random access.
- [**Software Frameworks**](docs/frameworks.md): LMAX Disruptor, Aeron IPC, etc.

---

## 🛠️ Calculation Baseline (3GHz CPU)

- **Frequency**: 3,000,000,000 Hz
- **1 Clock Cycle**: 1 / 3,000,000,000 seconds = **0.333 nanoseconds**

---

## 💡 Notes & Priority

### Data Source Priority
We prioritize data based on the reliability and age of the source:
**Arch Manuals > Peer-reviewed Benchmarks > Engineering Blogs > Vendor Whitepapers**

### Terminology
*   **AZ (Availability Zone)**: Isolated locations within data center regions. Cross-AZ latency is significantly higher than intra-AZ.
*   **Contention**: Multiple threads/cores competing for the same resource (cache line, mutex), leading to performance degradation.

### Environment Variance
Actual latency varies based on hardware configuration, OS tuning, network load, and message size.

---

## 🤝 Contributing

Data is sourced from LMAX, Martin Thompson, Samsung, and other reputable reports. Pull Requests for updates or new data points are welcome.

**Last Updated**: 2025-12-30  
**Maintainers**: Exchange-CPP Team
