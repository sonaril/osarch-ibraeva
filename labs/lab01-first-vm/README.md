# Lab 1 — Your first virtual machine

Pair: Ibraeva Nursaya
Machine: Lenovo IdeaPad Slim 5 16IRL8, Windows 11 Home 25H2; guest Ubuntu Server 26.04.1 LTS
Date: 25.10.2026

## What I did
1. Installed VirtualBox 7.2.20 r175154 (Qt6.8.0 on windows). Licence: https://www.virtualbox.org/wiki/Licensing_FAQ
2. Created VM lab01-ubuntu: 2 vCPU, 4 GB RAM, 25 GB dynamic disk, NAT.
3. Installed Ubuntu Server 26.04.1 LTS (Use an entire disk, OpenSSH server, no snaps), set the server name, grew the root volume, then took snapshot clean-install with the VM powered off:
   ```
   sudo hostnamectl set-hostname lab01-ibraeva
   sudo lvextend -r -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
   sudo poweroff
   ```
4. Broke it (delete /etc/hostname):
   ```
   cat /etc/hostname
   hostname
   sudo rm /etc/hostname
   cat /etc/hostname
   ```
5. Rolled back to clean-install and proved it is fixed:
   ```
   cat /etc/hostname
   hostname
   ```

## Resource justification
I gave the VM 2 vCPU of my 16 logical processors (10 cores of an Intel Core i7-13620H), 4 GB of my 16 GB RAM and a 25 GB dynamic disk out of about 228 GB free on the volume that holds my VMs. Two vCPU is far below half of the host and enough for a console-only server. 4 GB leaves about 12 GB for Windows and my other programs, and I need room for a second guest, Windows 11, in Lab 2. The disk is dynamically allocated, so it grows only as needed and does not take 25 GB up front.

## Result
The VM boots and runs Ubuntu Server 26.04.1 LTS with 2 vCPU, about 3.3 GB visible RAM and a 23 GB root volume. The break-and-rollback cycle is shown in the evidence below: the hostname file was deleted, then restored by reverting to clean-install.

## What did not work the first time
- The root volume was only about half the disk after installation (`df -h /` showed 11.21 GB). I ran `sudo lvextend -r -l +100%FREE/dev/mapper/ubuntu--vg-ubuntu--lv`, which failed with "Specified %FREE/dev/mapper/ubuntu--vg-ubuntu--lv is unknown" because I left out the space before the path. With the space it worked: the volume grew from 11.5 to 23 GiB.
- When saving command output over SSH, the fingerprint prompt was cut off and my `yes` went to PowerShell ("yes is not recognized"). Running the command again accepted the key.
- After deleting /etc/hostname and rebooting, `hostname` returned `sonaril` (the name from the installer) and `cat /etc/hostname` failed. Reverting to clean-install brought back `lab01-ibraeva`.

## Evidence
- evidence/vm-settings.png — CPU and memory as configured
- evidence/guest-release.png — Ubuntu Server is running in the guest
- evidence/broken.png — the guest in its broken state
- evidence/snapshot-list.png — the snapshot, after the rollback
- evidence/after-rollback.png - after roll back
