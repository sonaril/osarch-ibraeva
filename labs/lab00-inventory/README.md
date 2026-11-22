# Lab 00 — Inventory your own machine

| What | Value | Where I got it |
| :--- | :--- | :--- |
| CPU model | 13th Gen Intel(R) Core (TM) i7-13620H | `Get-WmiObject Win32_Processor` |
| Core count | 10 | `(Get-WmiObject Win32_Processor).NumberOfCores` |
| Thread count | 16 | `(Get-WmiObject Win32_Processor).NumberOfLogicalProcessors` |
| Total memory | 16 GB (2 x 8,589,934,592 bytes) | `Get-CimInstance Win32_PhysicalMemory \| Select-Object Capacity` |
| Memory modules | 2 (Samsung) | `(Get-CimInstance Win32_PhysicalMemory \| Measure-Object).Count` |
| Memory speed | 6400 MT/s (Configured: 5200 MT/s) | `Get-CimInstance Win32_PhysicalMemory \| Select-Object Speed, ConfiguredClockSpeed` |
| Disk model | SKHynix_HFS512GEJ4X112N | `Get-PhysicalDisk \| Select-Object FriendlyName` |
| Disk type | NVMe SSD | `Get-PhysicalDisk \| Select-Object MediaType, BusType` |
| Free space on VM volume (C:) | 330,136,182,784 bytes (~307 GiB / 330 GB) | `Get-Volume -DriveLetter C \| Select-Object SizeRemaining` |
| Firmware type | <UEFI or BIOS> | `(Get-ComputerInfo).BiosFirmwareType` |
| BIOS version & date | LENOVO LACN41WW, 9/23/2025 | `Get-CimInstance Win32_BIOS \| Select-Object Manufacturer, SMBIOSBIOSVersion, ReleaseDate` |
| Hardware virtualization | Present: True; Firmware enabled: <True or False> | `(Get-CimInstance Win32_ComputerSystem).HypervisorPresent`; `(Get-CimInstance Win32_Processor).VirtualizationFirmwareEnabled` |

## What did not work the first time

* In the initial submission, the inventory tables were pasted as screenshots instead of plain Markdown text, and `msinfo32` (a GUI utility) was referenced instead of a CLI command for the firmware type.
* Checking only `HypervisorPresent` showed that the OS hypervisor was active, so I added `(Get-CimInstance Win32_Processor).VirtualizationFirmwareEnabled` and `(Get-ComputerInfo).BiosFirmwareType` to verify hardware virtualization and firmware mode directly from PowerShell.

## Evidence

* `evidence/cpu.txt` — Raw PowerShell output for CPU model, cores, and logical processors.
* `evidence/memory.txt` — Raw PowerShell output for physical memory modules, capacity, and speed.
* `evidence/diskmodel.txt` — Raw PowerShell output for physical disk model, bus type, and volume free space.
* `evidence/firmwaretype.txt` — Raw PowerShell output for firmware type (`UEFI`/`BIOS`) and BIOS version/release date.
* `evidence/virtualization.txt` — Raw PowerShell output for hardware virtualization status.

* Initially, tables were pasted as images rather than Markdown text tables, and firmware type / firmware virtualization checks were missing from the CLI summary.
* Raw command outputs are stored in `evidence/` (`cpu.txt`, `memory.txt`, `diskmodel.txt`, `firmwaretype.txt`, `virtualization.txt`)[cite: 13].
