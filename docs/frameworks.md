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

**Source**: [Man Group - High Performance Execution on Aeron](https://www.man.com/special-fx-execution-system-on-aeron)

| Transport | Scenario | Latency | Cycles @ 3GHz | Notes |
|:----------|:---------|:--------|:--------------|:------|
| **Aeron IPC** | Round Trip (RTT) | **~0.25μs** | **~750** | Shared memory IPC |
| **Aeron UDP (Loopback)** | Round Trip (RTT) | **~10μs** | **~30K** | Optimized UDP stack |
