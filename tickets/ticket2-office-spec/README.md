# Ticket 2 — Kitting out the new office

## 1. What was asked

> From: Daniyar K. (Director)
> Subject: computers for the new office
>
> We are opening the second office and need four computers — accountant, sales, web developer, designer. Budget 3,000,000 tenge for everything. Everyone says we should just buy the same four machines, is that right? Make sure they last a few years. Monitors included.

## 2. What was actually needed

The director's assumption that four identical machines is best is wrong. An accountant, a sales manager, a web developer, and a designer have different daily workloads and therefore hit different bottlenecks (RAM capacity for multi-tab/video workflows, CPU/storage for local dev environments, or GPU/display for design work). Buying four identical systems either wastes budget on features one role does not use or leaves another role short of what it needs within the 3,000,000 KZT limit.

The request left out several key details: the exact lifespan in years ("a few years"), whether peripherals beyond monitors (keyboards, mice, cables, surge protection) are included, what specific software each role runs (especially accounting software requiring Windows or developer tooling), and whether pre-built vs. custom-built systems are preferred.

I would have asked Daniyar:
- How many years should these machines serve before replacement, and how important is future upgradability?
- Which specific applications does each role rely on (for example, Windows-only accounting tools, containers/VMs, or graphics/3D software)?
- Are keyboards, mice, network cables, and power protection already on site or part of this 3,000,000 KZT budget?

Instead of waiting for answers, I assumed a multi-year service life with practical component choices, desktop tower/mATX form factors with monitors and licensed OSes included, Windows 11 Pro for the roles needing Windows compatibility, and Ubuntu 26.04.1 LTS for the web developer.

## 3. What I did

I mapped each role to its expected workload and bottleneck, selected compatible components meeting the >= 16 GB RAM, 1 TB SSD, licensed OS, and monitor requirements, and recorded KZT prices with dates from Kazakhstan vendors in [`budget.csv`](budget.csv) alongside build justifications in [`builds.md`](builds.md). Total spend came to **2,893,752 KZT**, staying under the **3,000,000 KZT** budget.

**1. Which of the four machines will you upgrade first, eighteen months from now, and why?**
I would upgrade the **Accountant's** machine first — specifically adding/upgrading **RAM** (moving beyond 16 GB if spreadsheet sizes and resident background processes grow over 18 months), because it starts at the 16 GB baseline while the sales, developer, and designer machines already have 32 GB.

**2. Price the same four as pre-built machines from a vendor. What does the company gain and what does it lose by not building them?**
As a reference point from Kazakhstan retail quotes on 2026-11-22, a pre-built tower such as the Tehnicom Core i3-12100 system is listed at **400,676 KZT** (with 64 GB RAM and a 128 GB SSD, which also falls short of the 1 TB SSD requirement on that listing, plus monitor cost on top). Buying pre-built machines gives the company **gained** convenience: single-vendor warranty/support, less assembly time, and ready-to-deploy units. What the company **loses** is tailored component selection for each bottleneck, clear visibility into every individual part, and straightforward per-part upgradeability.

**3. One of these four people will complain about performance within a month. Which one, and about what?**
The **Designer** is the most likely to complain within a month if their design work involves heavy video rendering or large 3D/scene workloads, because the budget trade-off kept the CPU at an i5-12400 and 32 GB RAM rather than stepping up to a higher-tier CPU or workstation class.

## 4. Evidence

- [`evidence/pcpartpicker-links.md`](evidence/pcpartpicker-links.md) — Lists the four PCPartPicker build links used to check socket, memory, M.2, and power compatibility (PCPartPicker does not price in KZT, so it is used strictly for compatibility verification).
- [`evidence/vendor-quotes/accountant-cpu.png`](evidence/vendor-quotes/accountant-cpu.png) — Proves the 66,989 KZT price and date on Kaspi.kz for the Accountant's Intel Core i3-12100 OEM CPU.
- [`evidence/vendor-quotes/accountant-cooler.png`](evidence/vendor-quotes/accountant-cooler.png) — Proves the 4,360 KZT price and date on DNS for the ID-COOLING SE-802-SD V3 CPU cooler.
- [`evidence/vendor-quotes/accountant-motherboard.png`](evidence/vendor-quotes/accountant-motherboard.png) — Proves the 31,884 KZT price and date on Kaspi.kz for the GIGABYTE H610M K DDR4 motherboard.
- [`evidence/vendor-quotes/accountant-memory.png`](evidence/vendor-quotes/accountant-memory.png) — Proves the 99,900 KZT price and date on Kaspi.kz for the 16 GB Kingston Fury Beast DDR4 kit.
- [`evidence/vendor-quotes/accountant-storage.png`](evidence/vendor-quotes/accountant-storage.png) — Proves the 141,150 KZT price and date on DNS for the 1000 GB Samsung 990 PRO M.2 NVMe SSD.
- [`evidence/vendor-quotes/accountant-psu.png`](evidence/vendor-quotes/accountant-psu.png) — Proves the 29,999 KZT price and date on Kaspi.kz for the Deepcool PF450 450W power supply.
- [`evidence/vendor-quotes/accountant-case.png`](evidence/vendor-quotes/accountant-case.png) — Proves the 69,999 KZT price and date on Kaspi.kz for the LIAN LI A3-mATX-WD case.
- [`evidence/vendor-quotes/accountant-monitor.png`](evidence/vendor-quotes/accountant-monitor.png) — Proves the 45,990 KZT price and date on Kaspi.kz for the 23.8" Xiaomi Monitor A24i.
- [`evidence/vendor-quotes/accountant-os.png`](evidence/vendor-quotes/accountant-os.png) — Proves the 46,885 KZT price and date on Kaspi.kz for Microsoft Windows 11 Professional 64-bit.
- [`evidence/vendor-quotes/sales-cpu.png`](evidence/vendor-quotes/sales-cpu.png) — Proves the 99,990 KZT price and date on Kaspi.kz for the Sales Manager's AMD Ryzen 5 5600G OEM CPU.
- [`evidence/vendor-quotes/sales-motherboard.png`](evidence/vendor-quotes/sales-motherboard.png) — Proves the 34,990 KZT price and date on DNS for the MSI A520M-A PRO motherboard.
- [`evidence/vendor-quotes/sales-memory.png`](evidence/vendor-quotes/sales-memory.png) — Proves the 295,730 KZT price and date on Kaspi.kz for the 32 GB Kingston Fury Beast DDR4 kit.
- [`evidence/vendor-quotes/web-developer-cpu.png`](evidence/vendor-quotes/web-developer-cpu.png) — Proves the 78,378 KZT price and date on itmag.kz for the Web Developer's Intel Core i5-12400 OEM CPU.
- [`evidence/vendor-quotes/web-developer-storage.png`](evidence/vendor-quotes/web-developer-storage.png) — Proves the 60,870 KZT price and date on DNS for the 1000 GB ADATA LEGEND 710 M.2 NVMe SSD.
- [`evidence/vendor-quotes/web-developer-case.png`](evidence/vendor-quotes/web-developer-case.png) — Proves the 13,960 KZT price and date on DNS for the DEEPCOOL MATREXX 30 case.
- [`evidence/vendor-quotes/web-developer-os.png`](evidence/vendor-quotes/web-developer-os.png) — Proves availability and 0 KZT open-source licensing for Ubuntu 26.04.1 LTS Desktop on ubuntu.com.
- [`evidence/vendor-quotes/designer-gpu.png`](evidence/vendor-quotes/designer-gpu.png) — Proves the 234,000 KZT price and date on DNS for the GIGABYTE GeForce RTX 3060 GAMING OC 12GB GPU.
- [`evidence/vendor-quotes/designer-psu.png`](evidence/vendor-quotes/designer-psu.png) — Proves the 22,200 KZT price and date on DNS for the DEEPCOOL PF600 600W power supply.
- [`evidence/vendor-quotes/designer-monitor.png`](evidence/vendor-quotes/designer-monitor.png) — Proves the 101,746 KZT price and date on itmag.kz for the Dell Pro 27 QHD Monitor E2726DS.

## 5. What I would do differently

I would confirm the exact software list and monitor color/resolution requirements with each of the four roles (especially the designer and web developer) before buying parts, so I could verify whether a color-calibrated design monitor or different RAM/SSD split across the four machines fits better than my initial assumptions.
## 5. What I would do differently

I would confirm the exact software list and monitor color/resolution requirements with each of the four roles (especially the designer and web developer) before buying parts, so I could verify whether a color-calibrated design monitor or different RAM/SSD split across the four machines fits better than my initial assumptions[cite: 13, 14, 15].
