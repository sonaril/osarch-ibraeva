# Lab 3 — Teardown, and the same machine from the command line

Pair: Nursaya Ibraeva   Driver first half: Nursaya
Machine: <host: Lenovo IdeaPad Slim 5 16IRL8, Windows 11>, <guest: Ubuntu Server 26.04.1 LTS in VirtualBox (VM `lab01-ubuntu`)>
Date: 08.11.2026

---

## Part 1 — The ten components

### Processors

**AMD Ryzen 5 7600** — [maker page]( <img width="500" height="494" alt="image" src="https://github.com/user-attachments/assets/6b260c63-2656-48b0-b471-0acc022b4a9c" /> ) · photo: `evidence/photos/ryzen-5-7600.png`
**What it is:** desktop CPU, 6 cores / 12 threads, Zen 4, 65 W TDP, with integrated Radeon graphics.
**Connects to:** socket AM5 (LGA1718), dual-channel DDR5, PCIe 5.0.
**Limit:** AM5 boards and DDR5 only; it will not fit AM4 boards and cannot use DDR4. Ships with the AMD Wraith Stealth cooler.
Source: <https://www.amd.com/en/products/processors/desktops/ryzen/7000-series/amd-ryzen-5-7600.html>

**Intel Core i5-13400** — [Intel ARK](<img width="600" height="720" alt="image" src="https://github.com/user-attachments/assets/a5afe3a1-5e57-427e-964d-3ff637feeaa6" />
) · photo: `evidence/photos/i5-13400.png`
10 cores (6 P + 4 E) / 16 threads, Raptor Lake, 65 W base power, integrated UHD Graphics 730.
Connects to: socket **LGA1700**, 600- and 700-series chipsets.
Limits: supports DDR5-4800 *or* DDR4-3200 (decided by the board, never both), dual channel, 128 GB max; the memory type is fixed by the motherboard slots.

### Motherboards

**ASUS PRIME B650M-A** — [maker page](<img width="450" height="450" alt="image" src="https://github.com/user-attachments/assets/50691f34-c2f1-469d-b9cc-bd98c3d12133" />
) · photo: `evidence/photos/asus-b650m-a.png`
Micro-ATX, socket **AM5**, chipset **AMD B650**, memory **DDR5** (4 DIMM slots).
CPU generations: Ryzen **7000, 8000 and 9000** desktop series (ASUS CPU support list; ⚠ older BIOS may be needed for newer generations).
Connects to: PCIe 4.0 x16 slot for a graphics card, M.2 slots for NVMe SSDs, SATA ports for 2.5" drives.
Limits: DDR5 only; AM5 only, so no Intel CPUs and no DDR4 modules.

**MSI PRO B760M-A WIFI DDR4** — [maker page](<img width="450" height="450" alt="image" src="https://github.com/user-attachments/assets/12ce454d-25e7-47a1-8139-d5d2d6ca9248" />
) · photo: `evidence/photos/msi-b760m-a.png`
Micro-ATX, socket **LGA1700**, chipset **Intel B760**, memory **DDR4** (4 DIMM slots, up to 128 GB).
CPU generations: Intel Core **12th, 13th and 14th** gen, Pentium Gold and Celeron (MSI CPU support list).
Connects to: PCIe 4.0 x16 slot, two M.2 slots, 4 SATA 6 Gb/s ports.
Limits: DDR4 only (a DDR5 module does not fit the slot); LGA1700 only, so no AMD CPUs.

### Memory

**Kingston FURY Beast DDR5-5600, 16 GB (KF556C40BB-16)** — [maker page](<img width="550" height="550" alt="image" src="https://github.com/user-attachments/assets/26d83d56-99b7-40c6-a6d5-0b1f353275a4" />
) · photo: `evidence/photos/kingston-fury-beast.png`
DDR5 DIMM, 16 GB, 288 pins, CL40, 1.25 V at the 5600 profile.
Connects to: a DIMM slot on a DDR5 board (AM5 or LGA1700 DDR5 boards).
Limits: not compatible with DDR4 boards (different notch position and pin layout). The SPD default is DDR5-4800; 5600 is an Intel XMP 3.0 profile, so on AMD it may run at 4800 until a profile is enabled in BIOS ⚠ confirm.

**Crucial DDR4-3200, 16 GB DIMM** — [maker page](<img width="1500" height="645" alt="image" src="https://github.com/user-attachments/assets/914e888e-88b1-4d8d-aa8b-0f16805dce1f" />
) · photo: `evidence/photos/crucial-ddr4.png`
DDR4 desktop DIMM, 16 GB, 288 pins, 3200 MT/s, 1.2 V ⚠ (confirm part number and CL).
Connects to: a DIMM slot on a DDR4 board.
Limits: DDR4 boards only; 3200 MT/s matches the i5-13400's official DDR4 limit, so no overclocking is needed.

### SSDs

**Samsung 990 PRO 1 TB** — [maker page](<img width="300" height="300" alt="image" src="https://github.com/user-attachments/assets/db47bbd4-41b6-4c15-b317-7a3b8bf9d12c" />
) · photo: `evidence/photos/samsung-990-pro.png`
M.2 2280 NVMe SSD, PCIe 4.0 x4, up to about 7,450 MB/s read ⚠ (confirm on Samsung page).
Connects to: an M.2 (M-key) slot, directly to CPU or chipset PCIe lanes.
Limits: needs an M.2 slot with PCIe/NVMe; both boards have PCIe 4.0 M.2 slots, so the full speed is available on either.

**Samsung 870 EVO 1 TB** — [maker page](<img width="1400" height="1400" alt="image" src="https://github.com/user-attachments/assets/5a84b2d5-9c7a-4668-a789-a56604550004" />
) · photo: `evidence/photos/samsung-870-evo.png`
2.5" SATA SSD, SATA III 6 Gb/s, up to about 560 MB/s read ⚠.
Connects to: a SATA port (data cable) and the PSU's SATA power connector, with a drive bay in the case.
Limits: capped by the SATA interface at about 600 MB/s regardless of the NAND speed.

### Power and cooling

**Corsair RM750e (750 W)** — [maker page](<img width="300" height="223" alt="image" src="https://github.com/user-attachments/assets/f95fbfba-58bf-4db4-aaf2-88a3561ad8ed" />
) · photo: `evidence/photos/corsair-rm750e.png`
ATX power supply, 750 W, 80 PLUS Gold, fully modular ⚠ (confirm revision, ATX 3.0 / 3.1).
Connects to: 24-pin motherboard connector, EPS CPU connector(s), PCIe power for a GPU, SATA power for drives.
Limits: 750 W total; it powers one build here (more than enough for either build, which draw well under 300 W without a discrete GPU).

**Stock cooler: AMD Wraith Stealth** — [maker page](<img width="734" height="706" alt="image" src="https://github.com/user-attachments/assets/7ea8db1f-23a4-4096-ab85-4ad2754a23b5" />
) · photo: `evidence/photos/wraith-stealth.png`
Low-profile air cooler included in the box of the Ryzen 5 7600.
Connects to: the AM5 retention bracket and the board's CPU fan header.
Limits: designed for 65 W CPUs and sufficient for stock operation, but loud and marginal under sustained load. The i5-13400 box also contains an Intel cooler ⚠ (confirm on ARK / Intel page).

---

## Part 2 — Motherboard compatibility

| | ASUS PRIME B650M-A | MSI PRO B760M-A WIFI DDR4 |
|---|---|---|
| Socket | AM5 | LGA1700 |
| Chipset | AMD B650 | Intel B760 |
| Memory type | DDR5 | DDR4 |
| CPU generations | Ryzen 7000 / 8000 / 9000 | Core 12th / 13th / 14th gen |
| Accepts Ryzen 5 7600 | Yes | No (wrong socket) |
| Accepts Core i5-13400 | No (wrong socket) | Yes |

## Part 3 — Two computers on paper

### Build A: Ryzen 5 7600 (AM5, DDR5)

| Part | Why it is here |
|---|---|
| Ryzen 5 7600 | AM5 CPU, 6 cores, 65 W, has its own cooler and integrated graphics |
| ASUS PRIME B650M-A | Only board in the list with socket AM5 |
| Kingston FURY Beast DDR5-5600 | AM5 is DDR5-only, so the DDR4 Crucial stick would not fit |
| Samsung 990 PRO 1 TB | The board has PCIe 4.0 M.2 slots, so the NVMe drive runs at full speed |
| Wraith Stealth | Included with the 7600, no extra purchase needed |
| Corsair RM750e | 750 W covers CPU, SSD and a future GPU with headroom |

Still needed: micro-ATX case, operating system, fans if the case has none, and a graphics card only if integrated graphics is not enough.

### Build B: Core i5-13400 (LGA1700, DDR4)

| Part | Why it is here |
|---|---|
| Core i5-13400 | LGA1700 CPU, fits the B760 board, supports DDR4-3200 |
| MSI PRO B760M-A WIFI DDR4 | The only LGA1700 board in the list, and it takes DDR4 |
| Crucial DDR4-3200 | DDR4 board, and 3200 MT/s equals the CPU's official DDR4 limit |
| Samsung 870 EVO 1 TB | The board has 4 SATA ports; a cheaper, simpler drive for a budget build |
| CPU cooler | Intel box cooler ⚠ if confirmed, otherwise a separate cooler is needed |

Still needed: micro-ATX case, **a power supply** (the single Corsair RM750e is assigned to Build A), operating system, and a graphics card only if integrated UHD 730 is not enough.

Mismatches that were checked and rejected: Kingston DDR5 in Build B (board has DDR4 slots), Crucial DDR4 in Build A (AM5 has no DDR4), Ryzen 7600 on the MSI board (wrong socket).

---

## Part 4 — The same facts from the command line

Windows output is from the host laptop. Linux output is from the Ubuntu Server VM `lab01-ubuntu`, so it describes the **virtual machine**, not the physical hardware (for example, `dmidecode` shows the hypervisor's emulated devices). No Mac is available in the pair, so `macos-specs.txt` is not provided.

| Fact | Windows (PowerShell) | Linux (Ubuntu Server) | macOS (not available) |
|---|---|---|---|
| CPU model | `Get-CimInstance Win32_Processor \| Select Name` | `lscpu \| grep "Model name"` | `sysctl -n machdep.cpu.brand_string` |
| Cores / threads | `Get-CimInstance Win32_Processor \| Select NumberOfCores,NumberOfLogicalProcessors` | `lscpu` (Core(s) per socket, Thread(s) per core) | `sysctl hw.physicalcpu hw.logicalcpu` |
| Total memory | `Get-CimInstance Win32_ComputerSystem \| Select TotalPhysicalMemory` | `free -h` | `sysctl hw.memsize` |
| Memory modules and speed | `Get-CimInstance Win32_PhysicalMemory \| Select PartNumber,Capacity,Speed` | `sudo dmidecode -t memory` | `system_profiler SPMemoryDataType` |
| Disk model, type, size | `Get-PhysicalDisk \| Select FriendlyName,MediaType,BusType,Size` | `lsblk -d -o NAME,MODEL,SIZE,ROTA,TRAN` | `system_profiler SPStorageDataType` |
| Free space on volumes | `Get-Volume` | `df -h` | `df -h /` |

How the names differ: Windows reports "logical processors" where Linux says "threads"; Windows `MediaType` says SSD/HDD where Linux shows `ROTA` (1 = rotational, 0 = not); in the VM, `lsblk` shows a virtual disk model, not the real SSD.

Raw outputs: [evidence/windows-specs.txt](evidence/windows-specs.txt), [evidence/linux-specs.txt](evidence/linux-specs.txt).

---

## What did not work the first time

<Write at least one real thing. What you saw, what it turned out to be, what you did. For example: a shop page was found first instead of the maker's page; `dmidecode` in the VM showed virtual devices instead of the real RAM; a board's CPU support list differed between the manual and the website, and you used the website.>

## Evidence

- [evidence/photos/](evidence/photos/) — the makers' photos (screenshots) of all ten components
- [evidence/windows-specs.txt](evidence/windows-specs.txt) — raw PowerShell output from the host
- [evidence/linux-specs.txt](evidence/linux-specs.txt) — raw Linux output from the VM












