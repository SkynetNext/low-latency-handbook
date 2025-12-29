# CPU & Memory Latency

Detailed latency data for CPU operations, cache hierarchy, and memory access patterns.

---

## Instruction & Operation Costs

**Source**: [ithare.com](http://ithare.com/) - "Not all CPU operations are created equal"

Sorted by latency (ascending):

| Operation | Cycles | Nanoseconds @ 3GHz | Category | Description |
|:----------|:-------|:-------------------|:---------|:------------|
| **━━━ < 10 Cycles ━━━** | | | | |
| **Reg-to-Reg** (ADD, OR, etc.) | **< 1** | **< 0.33ns** | CPU | Basic ALU operations |
| **Memory Write** | **~1** | **~0.33ns** | Memory | To store buffer |
| **Branch Prediction Hit** | **1-2** | **0.33-0.67ns** | Branch | Correct prediction |
| **Int/FP/Vector Add** | **1-3** | **0.33-1ns** | CPU | SIMD operations |
| **Int/FP/Vector Multiply** | **1-7** | **0.33-2.3ns** | CPU | Slower than addition |
| **L1 Cache Read** | **4** | **~1.3ns** | Cache | Private core cache |
| **━━━ 10-100 Cycles ━━━** | | | | |
| **Branch Prediction Miss** | **10-20** | **3.3-6.7ns** | Branch | Pipeline flush required |
| **Floating Point Divide** | **10-40** | **3.3-13ns** | CPU | Significantly slower than mult |
| **128-bit Vector Divide** | **10-70** | **3.3-23ns** | CPU | SIMD division costs |
| **L2 Cache Read** | **10-12** | **3.3-4ns** | Cache | Private core cache |
| **Integer Divide** | **15-40** | **5-13ns** | CPU | Slower than FP in some cases |
| **C Function Call (Direct)** | **15-30** | **5-10ns** | Function | Non-virtual, optimizable |
| **C Function Call (Indirect)** | **20-50** | **6.7-17ns** | Function | Via function pointer |
| **C++ Virtual Call** | **30-60** | **10-20ns** | Function | vtable lookup overhead |
| **L3 Cache Read** | **30-70** | **10-23ns** | Cache | Shared, variable latency |
| **━━━ 100-1000 Cycles ━━━** | | | | |
| **Main Memory Read** (Local) | **100-150** | **33-50ns** | Memory | DDR4 random access |
| **NUMA: Remote L3 Read** | **100-300** | **33-100ns** | Memory | Cross-socket L3 access |
| **Malloc/Free Pair** | **200-500** | **67-167ns** | Memory | Small object allocation |
| **NUMA: Remote Memory Read** | **300-500** | **100-167ns** | Memory | Cross-socket main memory |
| **━━━ 1000-10000 Cycles ━━━** | | | | |
| **Syscall Overhead** | **1,000-1,500** | **0.33-0.5μs** | System | User-to-kernel transition |
| **Context Switch (Direct)** | **~2,000** | **~0.67μs** | System | Context saving only |
| **C++ Exception (Throw/Catch)** | **5,000-10,000** | **1.7-3.3μs** | C++ | Avoid in hot paths |
| **━━━ > 10000 Cycles ━━━** | | | | |
| **Context Switch (Total)** | **10k - 1M** | **3.3μs - 333μs** | System | Including cache pollution |

---

## Architecture Spotlight: Intel Sandy Bridge

**Source**: [Martin Thompson's Mechanical Sympathy](https://mechanical-sympathy.blogspot.com/)

### Core Memory Hierarchy

| Component | Capacity | Latency (Cycles) | Latency (ns @ 3GHz) | Notes |
|:----------|:---------|:-----------------|:--------------------|:------|
| **Registers** | 160 Int + 144 FP | **1** | **~0.33ns** | Single cycle access |
| **L1 Data Cache** | 32KB | **3** | **~1ns** | Private per core |
| **L1 Instruction Cache** | 32KB | **3** | **~1ns** | Private per core |
| **L2 Cache** | 256KB | **12** | **~4ns** | Private, Unified |
| **L3 Cache** | 2-20MB | **Up to 38** | **Up to 12.7ns** | Shared, ring-bus connected |
| **Main Memory (Local)** | - | **~195** | **~65ns** | Average case |
| **Main Memory (Remote)** | - | **~315** | **~105ns** | Cross-socket via QPI |

### Subsystem Details

*   **Memory Ordering Buffer (MOB)**: 64-entry load + 36-entry store buffers.
*   **Store Buffer**: 36 entries, fully associative.
*   **L3 Structure**: 2MB slices, hashed addressing to improve throughput.
*   **Memory Channels**: 4 channels per socket.
*   **QPI Bus**: 8.0 GT/s, supports prefetch forwarding.
