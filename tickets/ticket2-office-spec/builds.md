# Workstation Builds & Trade-off Justifications

Grand total across all four workstations: **2,893,752 KZT** (within the **3,000,000 KZT** limit, leaving **106,248 KZT** unspent). Every machine meets the mandatory baseline: at least 16 GB RAM, a 1 TB SSD, a licensed OS, and a monitor.

---

### Build 1 — Accountant

- **Load:** Accounting software (e.g., 1C / tax portals), spreadsheet workbooks, PDF documents, and browser tabs over an 8-hour workday.
- **Bottleneck:** Single-core responsiveness during spreadsheet recalculations and native compatibility with local accounting software rather than heavy parallel rendering or multi-GPU compute.
- **Trade-off:** Chose an Intel Core i3-12100 with integrated UHD 730 graphics and 16 GB DDR4 instead of a 32 GB RAM kit or discrete GPU, keeping memory at the 16 GB floor to direct budget toward higher-RAM and GPU workloads in other roles.
- **Total:** **537,156 KZT** (`budget.csv`, rows 2–10)

---

### Build 2 — Sales Manager

- **Load:** Browser-based CRM with 30–40 open tabs, spreadsheets, messaging apps, and several hours of video calls during the workday.
- **Bottleneck:** Memory capacity (RAM). Heavy browser tab counts combined with video calls push memory usage past 16 GB; on an under-provisioned system, the OS swaps to disk during calls and degrades responsiveness.
- **Trade-off:** Selected an AMD Ryzen 5 5600G on an AM4/DDR4 board with integrated graphics instead of a higher-tier CPU or DDR5 platform, putting the budget toward **32 GB DDR4** so browser and call workloads stay in RAM.
- **Total:** **769,093 KZT** (`budget.csv`, rows 12–20)

---

### Build 3 — Web Developer

- **Load:** Local development servers, containers/virtual machines, code editors/IDEs, local databases, and multi-tab browser testing across an 8-hour day.
- **Bottleneck:** Multi-core CPU throughput and memory capacity under concurrent local services, containers, and editor processes, plus reliable 1 TB SSD storage.
- **Trade-off:** Used an Intel Core i5-12400 (with integrated graphics) and **Ubuntu 26.04.1 LTS** (free open-source LTS license) rather than a paid Windows license or discrete GPU, allowing 32 GB DDR4 and a 1 TB NVMe SSD while keeping costs controlled.
- **Total:** **561,171 KZT** (`budget.csv`, rows 22–30)

---

### Build 4 — Designer

- **Load:** Visual design tools, multi-layer assets, high-resolution exports, and display-intensive layout work over an 8-hour workday.
- **Bottleneck:** Graphics acceleration/VRAM for rendering and canvas operations, display size/clarity (27" QHD IPS), and 32 GB memory with 1 TB NVMe storage.
- **Trade-off:** Kept the CPU at an Intel Core i5-12400 rather than stepping up to a higher-tier processor or DDR5 memory so the budget could fund an **NVIDIA GeForce RTX 3060 12 GB**, a **600W PSU**, a **27" QHD IPS monitor**, and **32 GB DDR4** in a roomy mATX case capable of fitting the 3-fan GPU.
- **Total:** **1,026,332 KZT** (`budget.csv`, rows 32–41)
