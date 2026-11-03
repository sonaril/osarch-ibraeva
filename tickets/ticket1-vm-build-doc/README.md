# Ticket 1 — New developer starts Monday, needs a test machine

## 1. What was asked

> From: Aigerim T. (Office Manager)
> Subject: test machine for new dev
>
> Hi, we have someone starting on Monday and he needs a test computer
> set up. Nothing special, just so he can try things without breaking
> anything. Can you write down how to do it so I can do the next one
> myself? Thanks

## 2. What was actually needed

| Missing from the request | What I would have asked | What I assumed |
|---|---|---|
| Which operating system(s) | Does he need Linux, Windows, or both? What will he test? | One Linux and one Windows guest, because a developer's test machine usually has to cover both: Ubuntu Server 26.04.1 LTS and Windows 11 Enterprise (evaluation). |
| What "a test computer" is | A real second computer, or a virtual one? | A **virtual** machine on his existing computer. A real second computer costs money and cannot be rolled back; a VM can. |
| The size of his computer | How much memory and disk does it have? | 16 GB RAM and about 100 GB free disk. With less, two guests do not fit (see section 3). |
| Internet and contact between machines | Does the test machine need the internet? Do the two guests need to talk to each other? | Yes to both, so both guests share one NAT Network. |
| Licensing | Is there a Windows licence? | No. I used the free evaluation edition, which works for about 90 days. |
| Who follows the document | How technical is the person who will do "the next one"? | Someone who has never opened a hypervisor. So every step has a screenshot, a check and a reason. |

**Where the request is wrong.**
"Nothing special" is not accurate: a machine that can be experimented on safely needs a **snapshot** taken before the experiments. Without it, "without breaking anything" is only a hope, and it is the one thing the job cannot do without that the request does not mention. Also, "write down how to do it" is not "set it up for me": the real deliverable is a document that someone else can follow alone. That is why the week-5 peer review matters, and its result is in `review/`.

## 3. What I did

1. Prepared the host (memory, disk, virtualisation) and installed VirtualBox 7.2.20.
2. Created a NAT Network for both guests.
3. Built `lab01-ubuntu` (Ubuntu Server 26.04.1 LTS), grew its root volume, and took the snapshot `clean-install`.
4. Built `lab02-windows` (Windows 11 Enterprise evaluation) and took its snapshot `clean-install`.
5. Tested ping in both directions and the internet from both guests.
6. Broke the Linux guest on purpose and rolled it back to the snapshot.
7. Wrote [BUILD.md](BUILD.md) from these steps.

**Settings that differ from the default, and why**

| Setting | Value | Why |
|---|---|---|
| Ubuntu memory / CPUs | 4096 MB / 2 | A console-only server needs little; leaves room for Windows. |
| Ubuntu disk | 25 GB, dynamic | Grows only as used. |
| Ubuntu install | Use an entire disk, OpenSSH server, no snaps | The virtual disk is empty; SSH lets me copy output out; snaps are not needed. |
| Ubuntu root volume | Extended with `lvextend -r -l +100%FREE` | The installer used only about half the disk (11 GB of 25 GB). |
| Ubuntu firmware | BIOS (UEFI off), no TPM | Ubuntu Server does not need UEFI, Secure Boot or a TPM; only Windows 11 does. |
| Windows memory / CPUs | 4096 MB / 2 | 4 GB is the Windows 11 minimum. |
| Windows disk | 64 GB, dynamic | 64 GB is the Windows 11 minimum disk size. |
| Windows firmware | EFI + Secure Boot + virtual TPM | Windows 11 will not install without them. |
| Windows graphics controller | VMSVGA | Fixed the black 640x480 screen on first start. |
| Network | NAT Network `osarch-net`, both guests | The guests see each other and keep internet access. |
| Windows firewall | Inbound ICMPv4 echo allowed | Otherwise Linux to Windows ping times out. |

**Alternatives I considered**
- *Plain NAT* instead of NAT Network: both guests would reach the internet, but not each other, so the ping test between them could not pass.
- *Bridged* instead of NAT Network: the guest becomes a separate device on the real network, which many office and campus networks refuse.
- *Fixed-size* disks instead of dynamic: a little faster, but they take all 25 GB and 64 GB at once. With about 228 GB free I kept the space free for my own files and accepted the small speed cost.

**Resources, calculated from my host** (Lenovo IdeaPad Slim 5 16IRL8, Intel Core i7-13620H, 16 logical processors, 16 GB RAM, about 228 GB free on the drive that holds the VMs):
- Memory: 4 + 4 = **8 GB of 16 GB**, exactly half, leaving 8 GB for Windows on the host.
- CPUs: 2 + 2 = 4 virtual processors of 16 logical ones.
- Disk: 25 + 64 = **89 GB** at most, of about 228 GB free.
- Why not more memory: in a test with 10 GB + 10 GB on a 16 GB host, memory in use rose from 8.5 GB to 12.7 GB, free memory fell from 7.2 GB to 3 GB, about 2 GB went to swap, and both guests lagged too much to use.

**One thing that went wrong, and how I fixed it.**
After installation the Ubuntu root volume was only about 11 GB. I ran `sudo lvextend -r -l +100%FREE/dev/mapper/ubuntu--vg-ubuntu--lv`, and it failed with `Specified %FREE/dev/mapper/ubuntu--vg-ubuntu--lv is unknown`, because I left out the space before the path. With the space it worked and the volume grew to 23 GB. The warning is now written into the build document at that step.

## 4. Evidence

Every file starts with a line `# Proves: ...`. Command output is kept as text; screenshots are used only for settings screens.

| File | What it proves |
|---|---|
| [evidence/01-new-vm-wizard.png](evidence/01-new-vm-wizard.png) | The wizard's hardware page: 4096 MB, 2 CPUs, "Use EFI" unticked. Taken in a wizard that was then cancelled, to show the settings without creating a third VM. |
| [evidence/01b-new-vm-wizard-disk.png](evidence/01b-new-vm-wizard-disk.png) | The wizard's disk page: 25 GB VDI with "allocate full size" unticked, i.e. a dynamically allocated disk. |
| [evidence/02-disk-settings.png](evidence/02-disk-settings.png) | The Linux VM `lab01-ubuntu` has 4096 MB, 2 CPUs and a 25 GB disk, and its adapter is on the NAT Network `osarch-net`. |
| [evidence/03-network-adapter.png](evidence/03-network-adapter.png) | Adapter 1 of the Linux VM is attached to the NAT Network `osarch-net`; its MAC address `0800272EDA61` matches the `link/ether` line in `05`. |
| [evidence/04-guest-uname.txt](evidence/04-guest-uname.txt) | The guest is Ubuntu Server 26.04.1 LTS (`lsb_release -a`, `uname -a`), its hostname is `lab01-ibraeva` (`hostname`), and its root volume is 23 GB (`df -h /`). Text transcribed from the console. |
| [evidence/04-guest-console.png](evidence/04-guest-console.png) | The console screenshot that `04-guest-uname.txt` was transcribed from. |
| [evidence/05-ping-between-guests.txt](evidence/05-ping-between-guests.txt) | Both guests' addresses (`10.0.2.5` and `10.0.2.3`), ping in both directions, and each guest reaching the internet (`1.1.1.1`). |
| [evidence/06-snapshot-list.png](evidence/06-snapshot-list.png) | The Linux VM has the snapshot `clean-install`, shown after the rollback. |
| [evidence/06b-snapshot-list-windows.png](evidence/06b-snapshot-list-windows.png) | The Windows VM has the snapshot `clean-install` (taken 03.11.2026). |
| [evidence/07-windows-vm-settings.png](evidence/07-windows-vm-settings.png) | The Windows VM `lab02-windows` has 4096 MB, UEFI, Secure Boot and TPM 2.0 turned on. |
| [evidence/08-after-rollback.png](evidence/08-after-rollback.png) | After restoring the snapshot, the hostname is `lab01-ibraeva` again. |
| [evidence/09-ubuntu-firmware.png](evidence/09-ubuntu-firmware.png) | The Linux VM uses plain BIOS: UEFI, Secure Boot and TPM are off. |
| [review/](review/) | A second person followed BUILD.md (added in week 5). |

## 5. What I would do differently

- Take the screenshots **while** building, step by step. In the labs I only photographed results, and I had to repeat the installer steps to document them.
- Create the NAT Network first. Lab 1 used plain NAT and Lab 2 needed a NAT Network, so the first guest had to be reconfigured.
- Never delete a VM while it is running. It left a broken entry and a lost disk file, and I had to recreate the Windows VM.

- Take the screenshots **while** building, step by step. In the labs I only photographed results, and I had to repeat the installer steps to document them.
- Create the NAT Network first. Lab 1 used plain NAT and Lab 2 needed a NAT Network, so the first guest had to be reconfigured.
- Never delete a VM while it is running. It left a broken entry and a lost disk file, and I had to recreate the Windows VM.

