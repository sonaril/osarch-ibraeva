# Lab 4 — Partition, mirror, fail, recover

Pair: Ibraeva / Zhanat 
Machine: laptop with Windows 11 host, VirtualBox; guests lab02-windows (Windows 11) and lab01-ubuntu (Ubuntu Server 26.04)
Date: 12.11.2026 (Windows part), 13.11.2026 (Linux part)

## What I did

### Windows (lab02-windows): partition, format, letter
Attached a 2 GB virtual disk (disk 1) to the Windows guest and ran in an elevated PowerShell:

```powershell
Get-Disk
Initialize-Disk -Number 1 -PartitionStyle GPT
New-Partition -DiskNumber 1 -UseMaximumSize -AssignDriveLetter
Format-Volume -DriveLetter E -FileSystem NTFS -NewFileSystemLabel LabDisk
Get-Disk | Format-Table Number, FriendlyName, PartitionStyle, Size
Get-Partition -DiskNumber 1 | Format-Table PartitionNumber, DriveLetter, Type, Size
```

The disk is **GPT**. I chose GPT because MBR is limited to 2 TB and four primary partitions,
GPT has a backup table at the end of the disk, and Windows 11 itself boots from GPT/UEFI.
Windows automatically created a 16 MB Reserved (MSR) partition 1, so my NTFS partition 2 starts at offset 16 MB.

### Linux (lab01-ubuntu): partition, mirror, fail, rebuild
Attached three 2 GB virtual disks (sdb, sdc, sdd).

```bash
lsblk | tee evidence/lsblk-before.txt
sudo apt install -y mdadm
for d in sdb sdc sdd; do sudo parted -s /dev/$d mklabel gpt mkpart primary 1MiB 100% set 1 raid on; done

sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1
sudo mkfs.ext4 /dev/md0
sudo mkdir -p /mnt/mirror && sudo mount /dev/md0 /mnt/mirror
sudo mdadm --detail /dev/md0 | tee evidence/array-healthy.txt

sudo dd if=/dev/urandom of=/mnt/mirror/testfile bs=1M count=500
sha256sum /mnt/mirror/testfile | tee evidence/checksum-before.txt

sudo mdadm /dev/md0 --fail /dev/sdc1
sudo mdadm /dev/md0 --remove /dev/sdc1
sudo mdadm --detail /dev/md0 | tee evidence/array-degraded.txt
sha256sum /mnt/mirror/testfile | tee evidence/checksum-after-failure.txt

sudo mdadm /dev/md0 --add /dev/sdd1
sudo mdadm --detail /dev/md0 | tee evidence/array-rebuilt.txt
```

## Result

Layout before: sda (system), sdb / sdc / sdd, 2 GB each, no partitions.
Layout after: md0 (RAID 1) = sdb1 + sdd1, mounted on /mnt/mirror; sdc1 was removed as the "failed" disk.

| State | Array | Key lines | Evidence |
|---|---|---|---|
| Healthy | sdb1 + sdc1 | State: clean, 2 active sync, `[UU]` | evidence/array-healthy.txt |
| Degraded | sdb1 only | State: clean, degraded, 1 active, sdc1 removed, `[U_]` | evidence/array-degraded.txt |
| Rebuilt | sdb1 + sdd1 | State: clean, 2 active sync, `[UU]` | evidence/array-rebuilt.txt |

Checksum of the 500 MiB test file is `b4063ad1f40bdd84d2d9a394cef3f781f15722846c2581a9962ed8c7156de64d`
before the failure, after the failure and after the rebuild. The files match, so the data survived the loss of one disk.

**Question: a colleague deleted the wrong file an hour ago. Does RAID 1 help?**
No. RAID 1 copies every write to both disks immediately, including the deletion, so the file disappeared from both copies at once.
RAID protects against a disk failure, not against human error or corruption.
For that you need a backup or a snapshot taken before the mistake.

## What did not work the first time

- `tee evidence/lsblk-before.txt` failed with "No such file or directory" because the folder did not exist yet. I created it and reran it.
- `mkdir -p ~ /lab04/evidence` gave "Permission denied". The space after `~` made bash treat `/lab04/evidence` as a path in the root directory. Without the space it worked.
- `parted ... mkpart primary 1 MiB 100%` gave "Invalid number" for the same reason, a stray space. `1MiB` worked.
- `apt update` hung: `ping 10.0.2.2` worked but `ping ubuntu.com` failed, so it was DNS. I set `DNS=8.8.8.8 1.1.1.1` in `/etc/systemd/resolved.conf.d/dns.conf` and restarted systemd-resolved.
- `mdadm --create` asked two questions and I answered only the first. The second defaulted to "no" and the creation was aborted. The second run with `y` to both worked.
- The rebuild onto sdd1 finished before I could catch the `recovery` percentage: the disks are virtual and tiny, so the sync took seconds.
- The VM repeatedly printed `soft lockup` / `rcu stall` after being idle or paused, and once needed a reboot. The array reassembled on its own, `[UU]`.

## Evidence

- [evidence/windows-disk.txt](evidence/windows-disk.txt) — disk 1 is GPT, partition 1 Reserved (MSR), partition 2 is E:
- [evidence/lsblk-before.txt](evidence/lsblk-before.txt) — three empty 2 GB disks before partitioning
- [evidence/array-healthy.txt](evidence/array-healthy.txt) — RAID 1 clean with two active disks
- [evidence/array-degraded.txt](evidence/array-degraded.txt) — array degraded, sdc1 removed
- [evidence/array-rebuilt.txt](evidence/array-rebuilt.txt) — array rebuilt with sdd1, `[UU]`
- [evidence/checksum-before.txt](evidence/checksum-before.txt) — SHA-256 of the test file before the failure
- [evidence/checksum-after-failure.txt](evidence/checksum-after-failure.txt) — same hash with one disk missing
- [evidence/checksum-after-rebuild.txt](evidence/checksum-after-rebuild.txt) — same hash after the rebuild
