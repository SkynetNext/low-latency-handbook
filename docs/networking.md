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

---

## Aeron Performance Reference

**Source**: [Aeron Google Cloud Performance Testing](https://aeron.io/other/aeron-google-cloud-performance-testing)  
**Test conditions**: 288-byte messages, 100K msg/s, Google Cloud C3 instances, same-zone deployment.

| Component | Version | RTT (P99) | Cycles @ 3GHz | One-way | Throughput |
|:----------|:--------|:----------|:--------------|:--------|:-----------|
| **Aeron Transport** | Open-source | **57μs** | **~171K** | ~28μs | 800K msg/s |
| **Aeron Transport** | Premium (DPDK) | **18μs** | **~54K** | ~9μs | 4.7M msg/s |
| **Aeron Cluster** | Open-source | **109μs** | **~327K** | ~55μs | 250K msg/s |
| **Aeron Cluster** | Premium (DPDK) | **36μs** | **~108K** | ~18μs | 2.2M msg/s |

---

## Receive Path → Ring Buffer Latency Comparison

One-way latency from **packet receive** to **delivery into a ring buffer** (excluding application write/copy). Ordered from highest to lowest latency:

**epoll (10–50μs) > io_uring (3–15μs) > DPDK (0.5–5μs) > FPGA (20–500ns)**

### Comparison by Approach

| Approach | Typical (one-way) | Heavily optimized (one-way) | Pure forward path (one-way) | Main bottlenecks | Notes |
|:---------|:------------------|:----------------------------|:----------------------------|:-----------------|:------|
| **epoll** | **10–50μs** | 5–20μs | — | Kernel stack, syscalls, context switches, scheduling jitter | Hard to stay below 5μs; sub-1μs is effectively unattainable |
| **io_uring** | **3–15μs** | 2–8μs | — | Kernel involvement, memory copy (zero-copy helps) | Fewer syscalls than epoll; sub-1μs remains very difficult |
| **DPDK** | **0.5–5μs** | 0.3–2μs | 0.5–2μs | PCIe DMA, userspace instruction cost | Kernel bypass, UIO/PMD, huge pages, core pinning; ~1μs is a practical floor in many setups. Data = one-way receive delay |
| **FPGA** | **20–500ns** | 20–100ns (on-chip) | 100–300ns (PCIe DMA) | Clock cycles, PCIe link | No software stack; highly deterministic. PCIe DMA round-trip ~500–800ns (Gen3×8) |

### Sources & Verification

| Claim | Source |
|:------|:------|
| **epoll 10–50μs**, kernel stack overhead, “sub-1μs nearly impossible” | [Kernel Bypass Networking: DPDK, SPDK, io_uring](https://anshadameenza.com/blog/technology/2025-01-15-kernel-bypass-networking-dpdk-spdk-io_uring/); [Cloudflare: How to receive a million packets](https://blog.cloudflare.com/how-to-receive-a-million-packets/); [LWN: Improving Linux networking performance](https://lwn.net/Articles/629155/) |
| **io_uring between kernel and DPDK**, better than epoll | [LiU: DPDK vs io_uring vs Linux network stack](https://liu.diva-portal.org/smash/record.jsf?pid=diva2%3A1789103); [io_uring vs epoll](https://ryanseipp.com/post/iouring-vs-epoll/) |
| **DPDK sub-μs to low μs**, L3fwd/testpmd latency | [FD.io CSIT DPDK Packet Latency](https://docs.fd.io/csit/master/report/dpdk_performance_tests/packet_latency/); [DPDK perf reports](https://core.dpdk.org/perf-reports/); [Intel NIC performance reports](https://fast.dpdk.org/doc/perf/) |
| **FPGA on-NIC ns-scale**, 10× lower than CPU | [hXDP (OSDI’20)](https://www.usenix.org/system/files/osdi20-brunella.pdf); [Cisco Nexus FPGA SmartNIC](https://www.cisco.com/c/en/us/products/collateral/interfaces-modules/nexus-smartnic/datasheet-c78-743825.html) |
| **PCIe Gen3 DMA round-trip** | [PLX: PCIe packet latency](https://www.mindshare.com/files/resources/PLX_PCIe_Packet_Latency_Matters.pdf); vendor/FPGA literature |

> **Note**: Ranges are industry-typical. Actual values depend on NIC, CPU, load, and tuning. “Pure forward path” = minimal processing (e.g. receive → enqueue only).
