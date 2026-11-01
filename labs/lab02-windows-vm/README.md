# Lab 2 — A Windows guest, and a network between them

Pair: Nursaya / Nurtas   Driver first half: Nursaya
Machine: Asus, Windows 11, 16GB, 8 cores, guests Ubuntu Server + Windows 11 Enterprise eval
Date: 01.11.2026

## What I did
1. Downloaded Windows 11 Enterprise evaluation. Licence: (https://www.microsoft.com/en-us/evalcenter/evaluate-windows-11-enterprise)
2. Created lab02-windows: 2 vCPU, 4 GB, 64 GB thin, UEFI + Secure Boot, vTPM.
3. Network mode: Nat Network, both guests. Why: Systems can interact and internet connection is preserved
4. Ping Ubuntu → Windows timed out because <why>. Fixed with: 
New-NetFirewallRule -DisplayName "Allow ping in (ICMPv4)" -Protocol ICMPv4 -IcmpType 8 -Direction Inbound -Action Allow

5. Overcommitment: set 10 GB + 10 GB on a 16 GB host.

## Network diagram
              Internet
                 |
      Host laptop 192.168.1.207
                 |
     [Virtualbox NAT network]
          /                        \
lab01-ubuntu                  lab02-windows
10.0.2.5                     10.0.2.3

## Result
| | Before | After |
|---|---|---|
| Host RAM in use | 8,5GB | 12,7GB |
| Available | 7,2GB | 3GB |
| Committed / Swap used | 157KB | 2GB |
| What I noticed | — | Both machines lag a lot. Impossible to do anything|

## What did not work the first time
NAT didn't work at first. Fixed by setting up a NAT Network

## Evidence
- [evidence/ipconfig-windows.txt](evidence/ipconfig-windows.txt) — Windows address
- [evidence/ip-addr-linux.txt](evidence/ip-addr-linux.txt) — Ubuntu address
- [evidence/ping-both-ways.txt](evidence/ping-both-ways.txt) — both directions and the internet
- [evidence/host-memory-before.png](evidence/host-memory-before.png) — host memory, guests off
- [evidence/host-memory-after.png](evidence/host-memory-after.png) — host memory, both guests overcommitted
