# Storage Latency

Random access and sequential operation latency across various storage media.

---

## Device Latency Comparison

**Source**: Samsung SSD Whitepapers, SNIA Guidelines, Seagate, Intel

| Device Type | Operation | Latency | Cycles @ 3GHz | Notes |
|:------------|:----------|:--------|:--------------|:------|
| **HDD (7200 RPM)** | Random Read | **8-15ms** | **24M-45M** | Seek + Rotation |
| **HDD (15000 RPM)** | Random Read | **~2ms** | **~6M** | Enterprise grade |
| **SATA SSD** | Random Read | **0.1-0.5ms** | **300K-1.5M** | Typical consumer SSD |
| **NVMe SSD** | Random Read | **20-100μs** | **60K-300K** | PCIe 3.0/4.0 |
| **NVMe SSD (High-end)** | Random Read | **~20μs** | **~60K** | Low-latency NAND/Optane |

### Calculation Notes
*   **HDD 7200 RPM**: Avg. rotation latency = 60s / (7200/2) = 4.17ms. Total includes ~9ms seek time.
*   **SSD**: Latency varies by controller, NAND type (TLC/QLC), and interface (SATA/NVMe).
