# Software & Framework Latency

Internal latency overhead of high-performance messaging and concurrency frameworks.

---

## LMAX Disruptor

**Source**: [LMAX Disruptor Official Docs](https://lmax-exchange.github.io/disruptor/)

### Disruptor vs. ArrayBlockingQueue (3-Stage Pipeline)

| Metric | ArrayBlockingQueue | Disruptor | Improvement | Notes |
|:-------|:-------------------|:----------|:------------|:------|
| **Min Latency** | **145ns** | **29ns** | **5.0x** | Single hop minimum |
| **Mean Latency** | **32,757ns** | **52ns** | **630x** | Average case |
| **99% Latency** | **2,097,152ns** | **128ns** | **16,384x** | P99 tail latency |
| **99.99% Latency** | **4,194,304ns** | **8,192ns** | **512x** | P99.99 tail latency |
| **Max Latency** | **5,069,086ns** | **175,567ns** | **29x** | Worst case |

---

## Aeron Messaging

**Source**: [Man Group: Special FX Execution System on Aeron](https://www.man.com/technology/special-fx-execution-system-on-aeron) (November 2020)

Benchmark methodology: JMH (Java Microbenchmark Harness) to minimize measurement overhead.

| Transport | Scenario | Latency | Cycles @ 3GHz | Notes |
|:----------|:---------|:--------|:--------------|:------|
| **Aeron IPC** | Round Trip (RTT, 100 bytes) | **0.25μs** | **~750** | Shared memory IPC, same box |
| **Aeron UDP** | Round Trip (RTT) | **~10μs** | **~30K** | Reliable UDP protocol over network |
| **Tibco** | Round Trip (RTT) | **~200μs** | **~600K** | Alternative reliable UDP solution |
| **TCP-based Messaging** | Round Trip (RTT) | **≥10ms** | **≥30M** | gRPC/HTTP2, KryoNet, etc. |

**Key Findings from Man Group**:
- Aeron IPC (0.25μs) is **faster than ConcurrentLinkedQueue** (0.3μs) between threads in the same process
- Aeron maintains low latency up to **99.999% percentile** under ping-pong workloads
- No noticeable latency increase under heavy batch load (unlike gRPC/HTTP2 and KryoNet)
- Production deployment (2019): IPC latency reduced by **at least an order of magnitude** at every percentile
