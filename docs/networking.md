# Networking Latency

Network stack, virtualization, and high-performance interconnect latency.

---

## Cloud & Virtualization (eBPF)

**Source**: [Cilium CNI Benchmark](https://cilium.io/blog/2021/05/11/cni-benchmark/)

| Scenario | Latency | Cycles @ 3GHz | Notes |
|:---------|:--------|:--------------|:------|
| **Cilium eBPF Forwarding** (Same Node) | **5-50μs** | **15K-150K** | Bypasses kernel netstack |
| **Cilium eBPF Forwarding** (Cross-AZ) | **~0.55ms RTT** | **~1.65M** | Round Trip Time |

---

## RDMA Interconnects

**Source**: China Mobile Research Institute, Academic Benchmarks

| Network Type | Scenario | Latency | Cycles @ 3GHz | Notes |
|:-------------|:---------|:--------|:--------------|:------|
| **InfiniBand (IB)** | RTT (4KB msg) | **~2.3μs** | **~6.9K** | HPC focus |
| **InfiniBand (IB)** | 10 Concurrent Flows | **~2.8μs** | **~8.4K** | Scalability test |
| **RoCE (RDMA over Ethernet)** | RTT (4KB msg) | **~3.1μs** | **~9.3K** | Ethernet-based RDMA |
| **High-end NIC RDMA** | One-way (No congestion) | **< 1μs** | **< 3K** | Best-case scenario |
| **Standard TCP/IP** | Same DC / Rack | **100-200μs** | **300K-600K** | Kernel stack overhead |
| **Standard TCP/IP** | Cross-Network | **200-500μs** | **600K-1.5M** | Routing & Congestion |

---

## Loopback & Intra-node Communication

**Source**: Man Group, UNSW

| Transport | Scenario | Latency | Cycles @ 3GHz | Notes |
|:----------|:---------|:--------|:--------------|:------|
| **UDP (Loopback)** | Round Trip (RTT) | **5-10μs** | **15K-30K** | OS stack overhead |
| **TCP (Loopback)** | Round Trip (RTT) | **10-20μs** | **30K-60K** | Conn management |
| **UDP (Same DC)** | Round Trip (RTT) | **50-150μs** | **150K-450K** | Physical switches |
| **TCP (Same DC)** | Round Trip (RTT) | **100-200μs** | **300K-600K** | Protocol overhead |
