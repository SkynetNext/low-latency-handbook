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

**Sources**:
- [NVIDIA ConnectX-7 Product Page](https://www.nvidia.com/en-us/networking/ethernet/connectx-7/)
- [NVIDIA InfiniBand NDR Overview](https://www.nvidia.com/en-us/networking/products/infiniband/)
- [Intel Ethernet 800 Series (E810) Datasheet](https://www.intel.com/content/www/us/en/products/details/ethernet/800-network-adapters.html)

| Network Type | Hardware | Latency (One-way) | Cycles @ 3GHz | Notes |
|:-------------|:---------|:------------------|:--------------|:------|
| **InfiniBand NDR** | NVIDIA ConnectX-7 | **< 0.6μs** | **< 1.8K** | 400Gbps, GPUDirect RDMA |
| **InfiniBand HDR** | NVIDIA ConnectX-6 | **~0.6μs** | **~1.8K** | 200Gbps |
| **RoCEv2** | NVIDIA ConnectX-7 | **< 0.6μs** | **< 1.8K** | Ethernet-based RDMA |
| **RoCEv2** | Intel E810 | **~1μs** | **~3K** | 100Gbps, iWARP also supported |
| **RoCEv2** | Broadcom BCM57608 | **< 1μs** | **< 3K** | 400Gbps, low power (30W) |
| **Standard TCP/IP** | - | **100-200μs** | **300K-600K** | Kernel stack overhead |

**Key Observations**:
- Modern RDMA NICs (ConnectX-7, BlueField-3) achieve sub-microsecond latency
- RDMA latency is **100-200x lower** than standard TCP/IP in same-DC scenarios
- RoCEv2 now matches InfiniBand latency on high-end hardware

---

## Loopback & Intra-node Communication

**Sources**:
- [Man Group: High Performance Execution System on Aeron](https://www.man.com/technology/special-fx-execution-system-on-aeron)
- Linux kernel network stack behavior (sockperf, netperf benchmarks)

| Transport | Scenario | Latency | Cycles @ 3GHz | Notes |
|:----------|:---------|:--------|:--------------|:------|
| **UDP (Loopback)** | Round Trip (RTT) | **5-10μs** | **15K-30K** | OS stack overhead only |
| **TCP (Loopback)** | Round Trip (RTT) | **10-20μs** | **30K-60K** | + Connection management |
| **UDP (Same DC)** | Round Trip (RTT) | **50-150μs** | **150K-450K** | + Physical NIC & switches |
| **TCP (Same DC)** | Round Trip (RTT) | **100-200μs** | **300K-600K** | + Protocol overhead |

