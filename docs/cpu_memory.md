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
| **L2 Cache Read** | **10-12** | **3.3-4ns** | Cache | Private core cache |
| **Branch Prediction Miss** | **10-20** | **3.3-6.7ns** | Branch | Pipeline flush required |
| **C Function Call (Direct)** | **15-30** | **5-10ns** | Function | Non-virtual, optimizable |
| **Atomic Inc/Dec** (Uncontended) | **20-50** | **7-17ns** | Sync | `lock xadd` without contention |
| **CAS Operation** (Uncontended) | **20-50** | **7-17ns** | Sync | `lock cmpxchg` |
| **C Function Call (Indirect)** | **20-50** | **6.7-17ns** | Function | Via function pointer |
| **Floating Point Divide** | **10-40** | **3.3-13ns** | CPU | Significantly slower than mult |
| **Integer Divide** | **15-40** | **5-13ns** | CPU | Slower than FP in some cases |
| **C++ Virtual Call** | **30-60** | **10-20ns** | Function | vtable lookup overhead |
| **L3 Cache Read** | **30-70** | **10-23ns** | Cache | Shared, variable latency |
| **128-bit Vector Divide** | **10-70** | **3.3-23ns** | CPU | SIMD division costs |
| **Mutex Lock/Unlock** (Uncontended) | **50-100** | **17-33ns** | Sync | User-space fast-path (futex) |
| **━━━ 100-1000 Cycles ━━━** | | | | |
| **Main Memory Read** (Local) | **100-150** | **33-50ns** | Memory | DDR4 random access |
| **Atomic (Contended)** | **100-300** | **33-100ns** | Sync | Same socket, cache line bouncing |
| **NUMA: Remote L3 Read** | **100-300** | **33-100ns** | Memory | Cross-socket L3 access |
| **Atomic (Cross-Socket)** | **300-800** | **100-267ns** | Sync | NUMA node/Socket bouncing |
| **Malloc/Free Pair** | **200-500** | **67-167ns** | Memory | Small object allocation |
| **NUMA: Remote Memory Read** | **300-500** | **100-167ns** | Memory | Cross-socket main memory |
| **━━━ 1000-10000 Cycles ━━━** | | | | |
| **Syscall Overhead** | **1,000-1,500** | **0.33-0.5μs** | System | User-to-kernel transition |
| **Mutex Lock/Unlock** (Contended) | **> 1,000** | **> 333ns** | Sync | Triggers syscall & context switch |
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

---

## Branch Prediction

Modern CPUs use deep pipelines (10-20+ stages) to maximize instruction throughput. A branch misprediction forces a complete pipeline flush, discarding all speculatively executed instructions.

### Misprediction Penalty by Architecture

**Sources**: 
- [Agner Fog's Microarchitecture Manual](https://www.agner.org/optimize/microarchitecture.pdf)
- [Intel 64 and IA-32 Architectures Optimization Reference Manual](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html)
- [Chips and Cheese](https://chipsandcheese.com/)

| CPU Architecture | Pipeline Depth | Misprediction Penalty | Notes |
|:-----------------|:---------------|:----------------------|:------|
| **Intel Haswell/Broadwell** | 14 stages | **16-20 cycles** | Agner Fog measured |
| **Intel Skylake** | 14-16 stages | **14-20 cycles** | 16.5 avg (μop cache hit), 19-20 (miss) |
| **Intel Alder Lake P-cores** | 12 stages | **~12-15 cycles** | Golden Cove |
| **Intel Alder Lake E-cores** | 20 stages | **~20 cycles** | Gracemont |
| **Intel Lion Cove (2024)** | 10 stages | **< 14 cycles** | Reduced pipeline depth |
| **AMD Zen 3/4** | ~19 stages | **~15-20 cycles** | Two-level branch predictor |

> **Note**: Actual penalty varies from **10 to 100+ cycles** depending on out-of-order execution state and whether the misprediction triggers cache misses.

### Why Misprediction Is Expensive

1. **Pipeline Flush**: All in-flight instructions on the wrong path must be discarded
2. **Wasted Work**: Deeper pipelines = more speculative instructions wasted
3. **Frontend Stall**: CPU must wait for correct-path instructions to refill the pipeline
4. **Cascading Effects**: Misprediction during cache miss can compound to 100+ cycles

### Optimization Techniques

| Technique | Example | Benefit |
|:----------|:--------|:--------|
| **Branchless code** | `x = (a > b) * a + (a <= b) * b` | Eliminates branch entirely |
| **CMOV instruction** | `cmovg` conditional move | No pipeline flush |
| **likely/unlikely hints** | `[[likely]]`, `__builtin_expect` | Helps predictor training |
| **Loop unrolling** | Reduce branch frequency | Fewer predictions needed |
| **Sorted data** | Sort before conditional processing | Improves prediction accuracy |

### Real-World Impact

A tight loop with ~50% branch prediction accuracy (worst case) can see **2-3x slowdown** compared to predictable patterns.

Classic example from [Stack Overflow](https://stackoverflow.com/questions/11227809):
- **Unsorted array**: ~11 seconds (frequent mispredictions)
- **Sorted array**: ~1.9 seconds (predictable pattern)
