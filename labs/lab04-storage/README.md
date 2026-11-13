# Lab 4 — Partition, mirror, fail, recover

Pair: Zhanat Nurtas + Ibraeva Nursaya · Date: 13.11.2026 · Hypervisor: VirtualBox

## Windows
```
PS C:\Users\sonaril> Get-Disk | Format-Table Number, FriendlyName, PartitionStyle, Size

Number FriendlyName  PartitionStyle        Size
------ ------------  --------------        ----
     0 VBOX HARDDISK GPT            68719476736
     1 VBOX HARDDISK GPT             2147483648

PS C:\Users\sonaril> Get-Partition -DiskNumber 1 | Format-Table PartitionNumber, DriveLetter, Type, Size

PartitionNumber DriveLetter Type           Size
--------------- ----------- ----           ----
              1             Reserved   16759808
              2           E Basic    2130640896

PS C:\Users\sonaril> Get-Volume -DriveLetter E

DriveLetter FriendlyName FileSystemType DriveType HealthStatus OperationalStatus
----------- ------------ -------------- --------- ------------ -----------------
E           LabDisk      NTFS           Fixed     Healthy      OK
```
GPT. MBR is limited to 2 TB and four primary partitions, GPT keeps a backup of its partition table at the end of the disk, and Windows 11 itself boots from GPT/UEFI (disk 0 is GPT too). Partition 1 is the 16 MB Reserved (MSR) partition that Windows creates automatically on every GPT disk, which is why my NTFS partition starts at 16 MB. I have no reason to use MBR on a new virtual disk.

Full output: [evidence/windows-disk.txt](evidence/windows-disk.txt)

## Linux — layout before and after
### Before
```
$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0                       7:0    0 66.8M  1 loop /snap/core24/1643
loop1                       7:1    0 66.8M  1 loop /snap/core24/2124
loop2                       7:2    0  1.5M  1 loop /snap/hwctl/123
loop3                       7:3    0  1.5M  1 loop /snap/hwctl/131
loop4                       7:4    0 50.1M  1 loop /snap/snapd/27710
sda                         8:0    0   25G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   23G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   23G  0 lvm  /
sdb                         8:16   0    2G  0 disk
sdc                         8:32   0    2G  0 disk
sdd                         8:48   0    2G  0 disk
sr0                        11:0    1 1024M  1 rom
```
Full output: [evidence/lsblk-before.txt](evidence/lsblk-before.txt)

### After
```
$ lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS

NAME                       SIZE TYPE  FSTYPE            MOUNTPOINTS
loop0                     66.8M loop  squashfs          /snap/core24/2124
loop1                     66.8M loop  squashfs          /snap/core24/1643
loop2                      1.5M loop  squashfs          /snap/hwctl/123
loop3                      1.5M loop  squashfs          /snap/hwctl/131
loop4                     50.1M loop  squashfs          /snap/snapd/27710
sda                         25G disk
├─sda1                       1M part
├─sda2                       2G part  ext4              /boot
└─sda3                      23G part  LVM2_member
  └─ubuntu--vg-ubuntu--lv   23G lvm   ext4              /
sdb                          2G disk
└─sdb1                       2G part  linux_raid_member
  └─md0                      2G raid1 ext4
sdc                          2G disk
└─sdc1                       2G part  linux_raid_member
sdd                          2G disk
└─sdd1                       2G part  linux_raid_member
  └─md0                      2G raid1 ext4
sr0                       1024M rom
```

## What I did
Creating the disk partitions (the same for sdb, sdc and sdd, in a for loop)
```
sudo apt install -y mdadm
for d in sdb sdc sdd; do sudo parted -s /dev/$d mklabel gpt mkpart primary 1MiB 100% set 1 raid on; done
```

Building the mirror from sdb1 and sdc1
```
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1
cat /proc/mdstat
```

Creating the file system and mounting it
```
sudo mkfs.ext4 /dev/md0
sudo mkdir -p /mnt/mirror
sudo mount /dev/md0 /mnt/mirror
```

Saving the healthy state
```
sudo mdadm --detail /dev/md0 | tee evidence/array-healthy.txt
cat /proc/mdstat | tee -a evidence/array-healthy.txt
```

Writing a test file and recording its checksum
```
sudo dd if=/dev/urandom of=/mnt/mirror/testfile bs=1M count=500
sha256sum /mnt/mirror/testfile | tee evidence/checksum-before.txt
```

Breaking one disk and getting the evidence
```
sudo mdadm /dev/md0 --fail /dev/sdc1
sudo mdadm /dev/md0 --remove /dev/sdc1
sudo mdadm --detail /dev/md0 | tee evidence/array-degraded.txt
cat /proc/mdstat | tee -a evidence/array-degraded.txt
ls -l /mnt/mirror
sha256sum /mnt/mirror/testfile | tee evidence/checksum-after-failure.txt
diff evidence/checksum-before.txt evidence/checksum-after-failure.txt
```

Adding the third disk and rebuilding
```
sudo mdadm /dev/md0 --add /dev/sdd1
cat /proc/mdstat
sudo mdadm --detail /dev/md0 | tee evidence/array-rebuilt.txt
cat /proc/mdstat | tee -a evidence/array-rebuilt.txt
sha256sum /mnt/mirror/testfile | tee evidence/checksum-after-rebuild.txt
```

Getting the final layout
```
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS | tee evidence/lsblk-after.txt
```

## Three states
| State | What /proc/mdstat showed | File |
| --- | --- | --- |
| Healthy | [2/2] [UU], sdc1[1] sdb1[0] | evidence/array-healthy.txt |
| Degraded | [2/1] [U_], sdb1[0] only; mdadm --detail: clean, degraded, sdc1 removed | evidence/array-degraded.txt |
| Rebuilt | [2/2] [UU], sdd1[2] sdb1[0]; sdd1 replaced sdc1 | evidence/array-rebuilt.txt |

Checksums before and after the failure: match (`b4063ad1f40bdd84d2d9a394cef3f781f15722846c2581a9962ed8c7156de64d`). They also match after the rebuild.

Files: [before](evidence/checksum-before.txt), [after failure](evidence/checksum-after-failure.txt), [after rebuild](evidence/checksum-after-rebuild.txt)

## RAID is not a backup
Your colleague deleted the wrong file from the mirror an hour ago. Does RAID 1 help? Why?

No, RAID 1 does not help. It writes every change to both disks at once, so the deletion was applied to the second disk immediately and no untouched copy exists. RAID protects against a physical disk failure, not against human error; for that you need a backup or a snapshot taken before the mistake.

## What did not work the first time
- `tee evidence/lsblk-before.txt` failed with "No such file or directory": the folder did not exist yet. I created it and ran the command again.
- `mkdir -p ~ /lab04/evidence` gave "Permission denied". The space after `~` made bash treat `/lab04/evidence` as a path in `/`, where I have no rights. Without the space it worked. `parted ... mkpart primary 1 MiB 100%` failed with "Invalid number" for the same reason (`1MiB` has to be one word).
- `apt update` hung. `ping 10.0.2.2` worked but `ping ubuntu.com` did not resolve, so the problem was DNS, not the network. I set `DNS=8.8.8.8 1.1.1.1` in `/etc/systemd/resolved.conf.d/dns.conf` and restarted systemd-resolved.
- `mdadm --create` asks two questions (bitmap, continue). I answered only the first one, the second defaulted to "no" and the creation was aborted. The second try with `y` to both worked.
- The rebuild onto sdd1 finished before I could see the `recovery` percentage: the disks are small and virtual, so it took seconds.
- The VM repeatedly printed `soft lockup` / `rcu stall` after being idle and stopped responding, so I had to reboot it. The array reassembled by itself as `[UU]` and the checksum was unchanged.
