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
| Free space on VM volume (C:) | 330,136,182,784 bytes (~307 GB / 330 GB) | `Get-Volume -DriveLetter C \| Select-Object SizeRemaining` |
| Firmware type | Uefi | `(Get-ComputerInfo).BiosFirmwareType` |
| BIOS version & date | LENOVO LACN41WW, 9/23/2025 5:00:00 AM | `Get-CimInstance Win32_BIOS \| Select-Object Manufacturer, SMBIOSBIOSVersion, ReleaseDate` |
| Hypervisor present | True | `(Get-CimInstance Win32_ComputerSystem).HypervisorPresent` |
| Virtualization in firmware | <True or False — paste output from PowerShell> | `(Get-CimInstance Win32_Processor).VirtualizationFirmwareEnabled` |

## What did not work the first time

* At first, the inventory tables were pasted as images rather than text tables, and `msinfo32` was used instead of a command-line check for the firmware type.
* I also needed `(Get-CimInstance Win32_Processor).VirtualizationFirmwareEnabled` and `(Get-ComputerInfo).BiosFirmwareType` to verify firmware virtualization and `Uefi` mode directly in PowerShell rather than relying on GUI tools or `HypervisorPresent` alone.

## Evidence

* `evidence/cpu.txt` — Raw CLI output for CPU model, cores, and logical processors.
* `evidence/memory.txt` — Raw CLI output for physical memory modules, capacity, and speed.
* `evidence/diskmodel.txt` — Raw CLI output for disk model, NVMe SSD type, and volume free space.
* `evidence/firmwaretype.txt` — Raw CLI output (`Uefi`) from `(Get-ComputerInfo).BiosFirmwareType`.
* `evidence/virtualization.txt` — Raw CLI output from `(Get-CimInstance Win32_Processor).VirtualizationFirmwareEnabled`.
